# Salvage Manifest

## What this is

The repo `jorgegonzalez44/Puerto-Rico-Airspace-Intelligence-Tool` is being deleted. Before deletion, every artifact judged worth keeping was consolidated onto this branch (`claude/airspace-intelligence-spiderweb-gaps-ekw5p`) by merging the only branch that held real work, `airspace-analysis-design`. The full git history of the repo (every branch, every commit) is preserved as a portable bundle at `salvage/puerto-rico-airspace-intelligence-tool.bundle`. This file is the index, the porting map, and the deletion checklist.

The salvaged work defines this repo's role as a **PRIIS producer** whose downstream/upstream hub is `jotaele44/spiderweb-pr`. The artifacts are intentionally a thin contract layer — schemas, a validator, a synthetic example, and two design notes — *not* a port of the spiderweb-pr codebase.

## Provenance

Salvaged from branch `airspace-analysis-design` (13 commits, tip `7ec53b0`):

| Commit | Subject |
|---|---|
| `fc4e692` | Add upstream component reuse matrix |
| `d05ea08` | Define airspace producer export target |
| `e60b063` | Add airspace observation schema |
| `eea84a9` | Add airspace export manifest schema |
| `1fe1cdc` | Add synthetic airspace export manifest |
| `2531849` | Add synthetic observations GeoJSON |
| `e4d547f` | Add synthetic observations CSV |
| `b50a3ab` | Add synthetic source manifest |
| `e25ed30` | Add synthetic lineage manifest |
| `2744615` | Add synthetic confidence manifest |
| `fb8f55e` | Add airspace export validator |
| `7ec53b0` | Add validator tests |

## Inventory

| # | File | Lines | Purpose |
|---:|---|---:|---|
| 1 | `docs/AIRSPACE_PRODUCER_EXPORT_TARGET.md` | 103 | Producer→hub export contract: package layout, observation field table, evidence tiers T1–T4, validation rules, production-mode synthetic fail-closed rule, future extension queue |
| 2 | `docs/UPSTREAM_COMPONENT_REUSE_MATRIX.md` | 58 | ADOPT/ADAPT/REFERENCE/BLOCK decisions for Contract-Sweeper, spiderweb-pr, PR.INT, PRIIS — explains what to lift and what to leave |
| 3 | `schemas/airspace_observation.schema.json` | 46 | JSON Schema 2020-12 for one observation; enums for `source_type`, `evidence_tier`, `geometry_status`, `temporal_status`; required `synthetic` flag |
| 4 | `schemas/airspace_export_manifest.schema.json` | 47 | JSON Schema for the export manifest; producer pinned to `Puerto-Rico-Airspace-Intelligence-Tool`; `mode` ∈ {test, production}; file map; record counts |
| 5 | `scripts/validate_airspace_export.py` | 221 | Dependency-light validator: file presence, manifest fields, observation field/range/enum checks, sources/lineage/confidence cross-refs, synthetic-in-production fail-closed |
| 6 | `tests/test_validate_airspace_export.py` | 16 | Round-trip tests: synthetic package passes in test mode, fails in production mode |
| 7 | `exports/examples/synthetic_airspace_package/manifest.json` | 21 | Canonical example package — also the test fixture |
| 8 | `exports/examples/synthetic_airspace_package/observations.geojson` | 55 | |
| 9 | `exports/examples/synthetic_airspace_package/observations.csv` | 3 | |
| 10 | `exports/examples/synthetic_airspace_package/sources.json` | 18 | |
| 11 | `exports/examples/synthetic_airspace_package/lineage.json` | 20 | |
| 12 | `exports/examples/synthetic_airspace_package/confidence.json` | 24 | |

**Discarded as not worth keeping:** `main`, the working branch's pre-merge state, and the README — all held only the placeholder line `# Puerto-Rico-Airspace-Intelligence-Tool`. No issues, no PRs, no other branches existed.

## Porting map → `jotaele44/spiderweb-pr`

Verified targets — none of these files exist in `spiderweb-pr` today (schemas/, scripts/, docs/ trees were each listed and checked).

| Source path (this repo) | Suggested target in `spiderweb-pr` | Notes |
|---|---|---|
| `docs/AIRSPACE_PRODUCER_EXPORT_TARGET.md` | `docs/contracts/AIRSPACE_PRODUCER_EXPORT_TARGET.md` | `docs/contracts/` already exists in spiderweb-pr — natural home for producer contracts |
| `docs/UPSTREAM_COMPONENT_REUSE_MATRIX.md` | `docs/contracts/UPSTREAM_COMPONENT_REUSE_MATRIX.md` | Same folder; matrix's subject *is* spiderweb-pr |
| `schemas/airspace_observation.schema.json` | `schemas/airspace_observation.schema.json` | Net-new; sits alongside `flight_event.schema.json`, `track_point.schema.json` |
| `schemas/airspace_export_manifest.schema.json` | `schemas/airspace_export_manifest.schema.json` | Net-new and **distinct** from spiderweb-pr's existing generic `export_manifest.schema.json` — keep both names so they don't collide |
| `scripts/validate_airspace_export.py` | `scripts/validate_airspace_export.py` | Net-new; complements existing `validate_jsonl.py` |
| `tests/test_validate_airspace_export.py` | `tests/test_validate_airspace_export.py` | Imports `scripts.validate_airspace_export`; path-compatible as-is |
| `exports/examples/synthetic_airspace_package/` (6 files) | `exports/examples/synthetic_airspace_package/` | Net-new directory; required fixture for the test |

### Acceptance check after porting

```bash
python scripts/validate_airspace_export.py exports/examples/synthetic_airspace_package --mode test        # exits 0, prints "VALIDATION PASSED"
python scripts/validate_airspace_export.py exports/examples/synthetic_airspace_package --mode production  # exits 1, prints synthetic-in-production failure
pytest tests/test_validate_airspace_export.py -q                                                          # 2 passed
```

Both validator invocations were already exercised on this branch — see "Local verification" below.

## Local verification done before push

```text
$ python3 scripts/validate_airspace_export.py exports/examples/synthetic_airspace_package --mode test
VALIDATION PASSED  (exit 0)

$ python3 scripts/validate_airspace_export.py exports/examples/synthetic_airspace_package --mode production
VALIDATION FAILED
- syn-air-0001: synthetic rows are not allowed in production mode
- syn-air-0002: synthetic rows are not allowed in production mode
(exit 1)

$ git bundle verify salvage/puerto-rico-airspace-intelligence-tool.bundle
salvage/puerto-rico-airspace-intelligence-tool.bundle is okay
The bundle records a complete history.
```

`pytest` is not installed in the salvage environment, but the test calls `validate_package(...)` directly, which is the same code path exercised by the two invocations above.

## Portable archive

`salvage/puerto-rico-airspace-intelligence-tool.bundle` is a git bundle containing every branch (`main`, the working branch, `origin/airspace-analysis-design`) and the full commit history. To restore anywhere — including after the GitHub repo is deleted:

```bash
git clone salvage/puerto-rico-airspace-intelligence-tool.bundle restored-repo
cd restored-repo
git branch -a
```

## Deletion checklist

Before clicking **Delete this repository** on `https://github.com/jorgegonzalez44/Puerto-Rico-Airspace-Intelligence-Tool/settings`:

- [ ] You have downloaded `salvage/puerto-rico-airspace-intelligence-tool.bundle` (sent to you as a session file artifact)
- [ ] The draft PR titled "Salvage airspace-analysis-design before repo deletion" shows the 12 salvaged files + this manifest + the bundle in its diff
- [ ] `git bundle verify` on the downloaded bundle reports `complete history`
- [ ] You have either copied the porting-map files into `jotaele44/spiderweb-pr` *or* retained the bundle as the long-term archive
- [ ] You have confirmed there are no issues, comments, or other GitHub-side artifacts you wanted to keep (verified empty at salvage time, but worth a final pass)
