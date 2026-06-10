# hashhog-ci

Public CI for the [hashhog](https://github.com/hashhog) fleet — 10 independent
Bitcoin full-node implementations, each in a different language, all validated
for Bitcoin Core consensus parity.

This repo holds **only** GitHub Actions workflows. It clones the public impl
repos + [`hashhog/test-suite`](https://github.com/hashhog/test-suite) + a pinned
Bitcoin Core, builds them on hosted runners, and runs the cross-implementation
differential regression harnesses. Nothing private lives here — the meta repo
(ops, server config, runbooks) stays private; everything CI needs is public, so
sweeps run on free Actions minutes with no token.

## Differential sweep

`differential-sweep.yml` (`workflow_dispatch`) runs one harness **family**
against any subset of the 10 impls — one impl per matrix job, fully isolated.

```
gh workflow run differential-sweep.yml --repo hashhog/hashhog-ci \
  --ref master -f family=watchonly -f impls=rustoshi,ouroboros
gh workflow run differential-sweep.yml --repo hashhog/hashhog-ci \
  --ref master -f family=gettxout -f impls=all
```

Families: `chaintxstats`, `coinstatsindex`, `getblockfilter`, `getblockheader`,
`getindexinfo`, `getnodeaddresses`, `getrawtransaction`, `gettxout`,
`gettxoutproof`, `gettxoutsetinfo`, `history`, `import`, `policy`, `rbf`,
`recovery`, `scanblocks`, `scantxoutset`, `spend`, `watchonly`.

Each job builds only its impl (+ the Core oracle for non-wallet families),
runs `test-suite/run-<family>-regression.sh` constrained to that impl, and
uploads a per-impl log + JSON. A final aggregate job emits `gap-map.json`.

## Layout

```
.github/
  actions/fleet-checkout/   clone public impls + test-suite + pinned Core
  actions/build-fleet/      build each (per-language caches)
  workflows/differential-sweep.yml
```

The Core oracle is pinned to a specific commit (`fleet-checkout` `core-ref`)
for reproducible ground truth; bump it deliberately.
