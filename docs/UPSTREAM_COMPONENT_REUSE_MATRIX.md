# Upstream Component Reuse Matrix

ACTIVE_VECTOR: Puerto Rico Airspace producer export contract

Purpose: decide what to reuse from existing Puerto Rico intelligence repos before porting code.

Rule: extract patterns first. Do not merge codebases.

## Classification keys

| Status | Meaning |
|---|---|
| ADOPT | Safe to reuse mostly as-is after review |
| ADAPT | Useful but must be refactored for this repo |
| REFERENCE | Use as a design pattern only |
| BLOCK | Do not import for this vector |

## Component matrix

| Source | Component | Decision | Reason |
|---|---|---|---|
| Contract-Sweeper | Step-based pipeline flow | REFERENCE | Strong setup, validation, normalization, coverage sequence |
| Contract-Sweeper | run_all.py orchestration pattern | ADAPT | Useful CLI pattern, but airspace commands differ |
| Contract-Sweeper | download validation pattern | ADAPT | Good gate model for required files and column families |
| Contract-Sweeper | coverage report idea | REFERENCE | Useful for export completeness, not contract-specific logic |
| Contract-Sweeper | federal contract ingestion code | BLOCK | Finance pipeline belongs outside this repo |
| spiderweb-pr | screenshot inventory with hashes | ADAPT | Directly relevant to FR24/screenshot evidence provenance |
| spiderweb-pr | schema validation and review queue pattern | ADAPT | Needed for fail-closed validation and manual review routing |
| spiderweb-pr | PR Intel adapter/export manifest pattern | REFERENCE | Good export discipline; this repo needs a narrower contract |
| spiderweb-pr | route extraction / event export ideas | ADAPT | Useful after schema and fixture validation are locked |
| spiderweb-pr | dashboard files | BLOCK | UI is premature for this foundation vector |
| spiderweb-pr | broad mission inference stack | BLOCK | Must wait until evidence, confidence, and provenance layers pass |
| PR.INT | CRS and geometry validation discipline | REFERENCE | Useful for Puerto Rico spatial exports and map alignment |
| PR.INT | municipal/context joins | REFERENCE | Hub or GIS producer concern unless needed for export enrichment |
| PR.INT | full GIS stack | BLOCK | Too broad for the airspace producer foundation |
| PRIIS design | federation and readiness gates | REFERENCE | Defines hub boundary and producer export requirements |
| PRIIS Analysis dossier | evidence-tier and completeness concepts | REFERENCE | Output belongs in hub Analysis, not this producer repo |

## Immediate adoption set

Implement only:

1. export package contract,
2. observation schema,
3. manifest schema,
4. synthetic fixture package,
5. validator script,
6. tests proving test-mode pass and production-mode fail for synthetic data.

## Deferred queue

| Queue item | Condition to start |
|---|---|
| screenshot inventory port | after export validator passes |
| manual review queue | after validation failure taxonomy is stable |
| route extraction | after screenshot inventory and geometry uncertainty rules exist |
| integration report | after validator produces structured results |
| dashboard/API | blocked until producer contract and hub gates exist |
