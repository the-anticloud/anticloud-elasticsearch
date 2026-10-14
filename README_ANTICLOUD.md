# ELASTICSEARCH (Anticloud overlay)

![license](https://img.shields.io/badge/license-Apache-2.0-blue) ![offline-first](https://img.shields.io/badge/offline-first-air-gap-green) ![audit](https://img.shields.io/badge/audit-SHA3-256-orange)

**Upstream:** https://github.com/lmenezes/elasticsearch @ b0fee6c01bf60fab9283148401e52353847932e1

**Category:** ONLINE_RETAIL  **Vendor:** upstream (see provenance)

## Architecture

```mermaid
graph LR
    U[ELASTICSEARCH<br/>upstream] --> S[Anticloud overlay]
    S --> B[BENCH.json + sbom.cdx.json]
    B --> A[AIOSS ledger<br/>SHA3-256 chained]
```

## Benchmarks (measured, with provenance)

| Check | Value | Source |
|---|---|---|
| files_scanned | 0 | BENCH.json metrics, run 2026-10-06T06:33:47.665935+00:00 |
| full metrics | see anticloud/08_BENCHMARK_MAPPING.md | BENCH.json |
| SBOM | sbom.cdx.json | bench engine |

No other benchmark number is claimed here. Anything not listed above is See BENCH.json for this project.

## Contents

- `UPSTREAM_CLONE/` (untouched upstream source)
- `anticloud/` (overlay: CRDT, provenance, licence, security, deps, perf, CLI, bench map, SBOM, compliance, hygiene, rebrand)
- `BENCH.json`, `sbom.cdx.json`

## Provenance

- Upstream SHA: b0fee6c01bf60fab9283148401e52353847932e1
- Clone tree SHA3-256: see anticloud/02_PROVENANCE.json

## Contact

Anticloud overlay worker (group6). Upstream issues go to the upstream repository.
