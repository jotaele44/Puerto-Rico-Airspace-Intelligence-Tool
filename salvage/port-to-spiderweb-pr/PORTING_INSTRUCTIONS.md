# Porting Instructions: airspace-producer-contract → `jotaele44/spiderweb-pr`

This directory contains everything needed to land the 12 producer-contract artifacts in the upstream `jotaele44/spiderweb-pr` repository. Both deliverables in this directory have been dry-run applied against a fresh clone of `jotaele44/spiderweb-pr@main` — they apply with **zero conflicts**.

## Choose one of two paths

| | Path A — single squashed change | Path B — preserve history |
|---|---|---|
| Deliverable | `airspace-producer-contract.patch` (combined unified diff, 26K) | `airspace-producer-contract.mbox` (12 per-commit patches in mbox format, 31K) |
| Apply with | `git apply` + a single new commit | `git am` (replays 12 commits with original authorship + messages) |
| Result | One commit titled "Add airspace producer export contract" | 12 commits: "Add upstream component reuse matrix" → "Add validator tests" |
| Use when | Upstream wants a single reviewable diff | Upstream wants the design progression preserved |

Both produce **identical file trees**. Pick whichever the upstream maintainer prefers.

## Prerequisites

- Local clone of `jotaele44/spiderweb-pr` with push access (or fork access if proposing via PR)
- Python ≥3.10 for the verification step (`jsonschema` and `pytest` are already installed by spiderweb-pr's CI config; no new deps)

## Path A — single squashed commit

```bash
cd /path/to/spiderweb-pr
git checkout main && git pull
git checkout -b feat/airspace-producer-contract

# Dry run first
git apply --check /path/to/airspace-producer-contract.patch

# Apply
git apply /path/to/airspace-producer-contract.patch
git add -A
git commit -m "Add airspace producer export contract

Salvaged from jorgegonzalez44/Puerto-Rico-Airspace-Intelligence-Tool@7ec53b0.
Defines the PRIIS producer export contract: observation schema, export
manifest schema, validator with synthetic-in-production fail-closed,
canonical synthetic example package, and two design docs."
```

## Path B — preserve 12-commit history

```bash
cd /path/to/spiderweb-pr
git checkout main && git pull
git checkout -b feat/airspace-producer-contract

git am /path/to/airspace-producer-contract.mbox
```

The 12 commits replayed (in order):

```
fc4e692  Add upstream component reuse matrix
d05ea08  Define airspace producer export target
e60b063  Add airspace observation schema
eea84a9  Add airspace export manifest schema
1fe1cdc  Add synthetic airspace export manifest
2531849  Add synthetic observations GeoJSON
e4d547f  Add synthetic observations CSV
b50a3ab  Add synthetic source manifest
e25ed30  Add synthetic lineage manifest
2744615  Add synthetic confidence manifest
fb8f55e  Add airspace export validator
7ec53b0  Add validator tests
```

If `git am` aborts for any reason: `git am --abort`, then fall back to Path A.

## Verification

Run from the spiderweb-pr root after applying:

```bash
# Validator behaviour
python scripts/validate_airspace_export.py exports/examples/synthetic_airspace_package --mode test
# expect: "VALIDATION PASSED" (exit 0)

python scripts/validate_airspace_export.py exports/examples/synthetic_airspace_package --mode production
# expect: "VALIDATION FAILED" with synthetic-row errors (exit 1)

# New test (auto-discovered by pytest via existing pyproject.toml testpaths)
python -m pytest tests/test_validate_airspace_export.py -v
# expect: 2 passed

# No regression in spiderweb-pr's existing suite
python -m pytest tests/ -q --tb=short --ignore=tests/test_io.py --ignore=tests/test_terrain.py
# expect: same pass count as pre-patch baseline
```

## Files added (verified against current `jotaele44/spiderweb-pr@main` — no collisions)

```
docs/contracts/AIRSPACE_PRODUCER_EXPORT_TARGET.md     (alongside existing SATELLITE_SOURCE_MANIFEST.md)
docs/contracts/UPSTREAM_COMPONENT_REUSE_MATRIX.md
schemas/airspace_observation.schema.json              (new file)
schemas/airspace_export_manifest.schema.json          (distinct from existing export_manifest.schema.json — different $id, different required fields)
scripts/validate_airspace_export.py                   (alongside existing validate_jsonl.py)
tests/test_validate_airspace_export.py                (auto-discovered)
exports/examples/synthetic_airspace_package/manifest.json
exports/examples/synthetic_airspace_package/observations.geojson
exports/examples/synthetic_airspace_package/observations.csv
exports/examples/synthetic_airspace_package/sources.json
exports/examples/synthetic_airspace_package/lineage.json
exports/examples/synthetic_airspace_package/confidence.json
```

`exports/` is virgin territory in spiderweb-pr — this patch creates it.

## Optional follow-ups (not in the patch)

Pre-staged as small snippets for the upstream maintainer to apply or reject independently.

### Optional 1 — README cross-link

Add this subsection to spiderweb-pr's `README.md` under the existing **Integration outputs** section so the producer contract is discoverable:

```markdown
### Producer contracts

Schemas and validators that define what downstream/producer repositories must hand to this hub:

- [`docs/contracts/AIRSPACE_PRODUCER_EXPORT_TARGET.md`](docs/contracts/AIRSPACE_PRODUCER_EXPORT_TARGET.md) — PRIIS airspace producer export contract
- [`docs/contracts/UPSTREAM_COMPONENT_REUSE_MATRIX.md`](docs/contracts/UPSTREAM_COMPONENT_REUSE_MATRIX.md) — Component reuse decisions (ADOPT/ADAPT/REFERENCE/BLOCK)
- [`docs/contracts/SATELLITE_SOURCE_MANIFEST.md`](docs/contracts/SATELLITE_SOURCE_MANIFEST.md) — Satellite source manifest contract

Validate a producer package locally:

    python scripts/validate_airspace_export.py exports/examples/synthetic_airspace_package --mode test
```

### Optional 2 — CI smoke step

Add this step to `.github/workflows/ci.yml` in the existing `test` job, just before the pytest invocation, mirroring the existing "Self-tests and CLI smoke" pattern:

```yaml
      - name: Airspace export validator smoke
        run: |
          python scripts/validate_airspace_export.py exports/examples/synthetic_airspace_package --mode test
          ! python scripts/validate_airspace_export.py exports/examples/synthetic_airspace_package --mode production
```

(The `!` inverts the expected non-zero exit — fail-closed in production mode is the desired behaviour.)

No new deps required; `jsonschema>=4.17` is already pre-installed by the workflow.

## Ready-to-paste upstream PR description

```
Title: Add airspace producer export contract

## Summary
Adds the PRIIS airspace producer export contract — schemas, validator, synthetic
example package, and two design docs — defining what a Puerto Rico airspace
producer must hand to this hub. Net 12 files / 632 lines; no edits to existing
code or schemas.

Salvaged from jorgegonzalez44/Puerto-Rico-Airspace-Intelligence-Tool@7ec53b0
before that repo is deleted.

## Layout
- `docs/contracts/` — 2 new design docs (export target + component reuse matrix)
- `schemas/` — 2 new JSON Schemas (observation + export manifest); distinct from
  existing `export_manifest.schema.json` (different $id, different required fields)
- `scripts/validate_airspace_export.py` — dependency-light validator with
  test/production modes; fails closed on synthetic rows in production
- `tests/test_validate_airspace_export.py` — auto-discovered round-trip tests
- `exports/examples/synthetic_airspace_package/` — canonical example + test fixture

## Verification
- `python scripts/validate_airspace_export.py exports/examples/synthetic_airspace_package --mode test` → exit 0
- `python scripts/validate_airspace_export.py exports/examples/synthetic_airspace_package --mode production` → exit 1
- `python -m pytest tests/test_validate_airspace_export.py -v` → 2 passed
- Existing test suite unchanged
```

## Provenance

The history-preserving mbox keeps original authorship and timestamps from the source repo's `airspace-analysis-design` branch (commits `fc4e692`..`7ec53b0`). The combined patch contains the same final tree as a single diff.

For full original-repo history (all branches, all commits, including the salvage merge), see `salvage/puerto-rico-airspace-intelligence-tool.bundle` one directory up.
