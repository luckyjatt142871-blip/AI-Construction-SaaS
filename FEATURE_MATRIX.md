# Feature matrix — 2026-10-08

| Feature | Status | Evidence / limitations |
|---|---|---|
| Authentication and MFA | IMPLEMENTED — UNVERIFIED | Full suite blocked by missing `pyotp`; PostgreSQL/Redis unavailable |
| RBAC / tenant isolation | IMPLEMENTED — UNVERIFIED | Policy unit tests pass; live RLS and cross-tenant API tests not run |
| Project management | IMPLEMENTED — UNVERIFIED | Tenant-scoped persistence/API and Next.js page; no live integration |
| PDF upload, raster preview | IMPLEMENTED — UNVERIFIED | Unit raster tests pass; private local dev storage only |
| Page-specific PDF calibration | IMPLEMENTED — UNVERIFIED | Immutable calibration history per drawing/revision/page, measurement FK; unit tests pass; live migration not run |
| Native-coordinate 2D takeoff | IMPLEMENTED — UNVERIFIED | Geometry unit tests pass; browser/real-world accuracy not benchmarked |
| Quantity saving and editing | IMPLEMENTED — UNVERIFIED | DB model/API; no live PostgreSQL test |
| Rate library / preliminary estimate / BOQ API | IMPLEMENTED — UNVERIFIED | Deterministic arithmetic unit tests; no approvals, revision workflow or exports |
| S3 private object storage | NOT STARTED | Local private development adapter only |
| IFC/BIM/3D | NOT STARTED | No functional BIM workflow |
| DDC/CWICR | NOT STARTED | Licensing review only; no restricted data imported |
| 4D scheduling / 5D cost | NOT STARTED | Not implemented |
| AI agents / client portal / billing | NOT STARTED | Not implemented |
| Cloud deployment | BLOCKED | No live environment, integration/security verification missing |

## 2026-10-08 calibration integrity update
- Per-page/per-revision calibration: **IMPLEMENTED — UNVERIFIED against PostgreSQL**; additional `0006` DB trigger checks calibration provenance and prevents mutation.
- Measurement revision provenance: **IMPLEMENTED — UNVERIFIED against PostgreSQL**.
- Targeted calibration/geometry tests: **TESTED (35 passed)**; full API/RLS suite remains **BLOCKED**.
- Private object storage, complete estimating workflow, BIM, 4D/5D and other product modules: **NOT COMPLETE**.

## 2026-10-08 estimating increment
- Estimate approval, immutable approved snapshot via draft revision copy: IMPLEMENTED — UNVERIFIED (live API/database)
- Approved BOQ Excel/PDF export: IMPLEMENTED — targeted unit tests PASSED; live API/browser UNVERIFIED
- Rate date provenance: IMPLEMENTED — migration UNVERIFIED; legacy null dates require user remediation
- CI PostgreSQL/Redis and frontend verification: CONFIGURED — NOT EXECUTED
- Full pyotp-backed pytest: BLOCKED in local environment; CI installation workflow configured
- Complete end-to-end estimating flow: IN PROGRESS — NOT ACCEPTED
- IFC/BIM, 4D/5D, AI agents, client portal, billing: NOT STARTED / NOT VERIFIED

### 2026-10-08 migration hardening
- Deterministic initial Alembic schema: IMPLEMENTED — UNVERIFIED AGAINST POSTGRESQL.
- Deterministic historical workspace migration: IMPLEMENTED — UNVERIFIED AGAINST POSTGRESQL.
- Disposable PostgreSQL historical upgrade CI test: IMPLEMENTED — NOT EXECUTED.
- Real PostgreSQL tenant/RLS and trigger tests: BLOCKED / NOT VERIFIED.
- Full pytest / frontend production build: BLOCKED / NOT VERIFIED.

## Latest increment — scheduling engine
| Feature | Implementation | Tests | Integration status |
|---|---|---|---|
| 4D deterministic CPM calculation | `backend/app/engines/scheduling.py` | 9 unit tests passed | Engine only; no persistence/API/frontend; NOT COMPLETE |
| PostgreSQL 16 migration and tenant isolation | Existing migration/CI assets | Not run live | BLOCKED — Docker/PostgreSQL unavailable |
| Full backend authentication suite | Existing code | Collection fails: missing `pyotp` | BLOCKED |
| Next.js production build | Existing frontend | Not run; dependencies missing | BLOCKED |

### Scheduling vertical increment (2026-10-08)
- Schedule activities and dependencies: IMPLEMENTED — UNVERIFIED (models, migration 0008, tenant-scoped API, Next.js UI; 16 targeted tests including existing CPM tests passed).
- PostgreSQL migration 0008, trigger, tenant RLS: IMPLEMENTED — UNVERIFIED (no PostgreSQL service).
- 4D calendar, baselines, BIM links, browser E2E: NOT IMPLEMENTED / NOT VERIFIED.
- Full backend test: BLOCKED by missing local pyotp; reproducible venv install script added, not executed successfully.

## 2026-10-08 5D vertical increment
| Feature | Backend/API | DB migration | Connected frontend | Verification | Status |
|---|---|---|---|---|---|
| 5D project budget | PUT/GET via costs endpoints | 0009, tenant RLS | costs page | pure/static tests only | IMPLEMENTED — UNVERIFIED |
| Actual/committed cost ledger | POST/GET via costs endpoints, source/date/currency validation | 0009, tenant RLS and project trigger | costs page | pure/static tests only | IMPLEMENTED — UNVERIFIED |
| Cost summary | deterministic Decimal calculation | budget + ledger records | summary UI | unit test passed | UNIT TESTED, INTEGRATION UNVERIFIED |
| 5D forecasting, cost-loaded schedule, change orders | not implemented | none | none | none | NOT IMPLEMENTED |
| Complete migration chain 0001–0009 | configured CI | pending live execution | n/a | not executed | BLOCKED |
| Full PDF→BOQ E2E | prior partial workflow | prior migrations | prior pages | not executed | BLOCKED |

## 2026-10-08 increment
| Feature | Status | Evidence / remaining work |
|---|---|---|
| Project issue register | IMPLEMENTED — UNVERIFIED | `app/issue_models.py`, `app/issue_api.py`, migration `0010`, Next.js `/projects/[companyId]/[projectId]/issues`; 3 static contract tests; live database/API/browser checks pending |
| Dependency installation | BLOCKED | `pyotp` declared but unavailable from local package index; frontend has no lockfile and packages unavailable |
| Full estimating acceptance workflow | BLOCKED | Requires live PostgreSQL, API and browser verification |
| PostgreSQL 0001–0010 / RLS | BLOCKED | CI configured, not executed |

## Runtime hardening increment (2026-10-08)
| Capability | Implementation | Verification |
|---|---|---|
| Docker backend migration image | Alembic files and scripts copied into image | Static/compile only; Docker unavailable |
| PostgreSQL restricted-role RLS regression | CI role provisioning + rollback-only runtime test | Implemented, not executed against PostgreSQL |
| Frontend deterministic npm installation | `npm ci` configured in CI and Docker | BLOCKED: no package-lock; registry DNS unavailable |
| Backend `pyotp` | Declared in requirements | BLOCKED: not installed locally |
| PDF-to-BOQ end-to-end | Existing APIs and export modules retained | NOT VERIFIED |

## Verification update 2026-10-08
| Requirement | Status | Evidence / remaining work |
|---|---|---|
| npm dependency bootstrap | IMPLEMENTED, NOT VERIFIED ONLINE | frontend/install-locked.sh and CI/Docker fallback; no lockfile committed because registry inaccessible |
| PostgreSQL 0001-0010 migration checks | TEST HARNESS EXTENDED, NOT EXECUTED | scripts/test_migration_chain.py checks tenant RLS and scope triggers |
| Full backend pytest / pyotp | BLOCKED | missing local pyotp, external package index unreachable |
| PDF-to-BOQ browser integration | NOT VERIFIED | requires running PostgreSQL/Redis/frontend and browser |

## 2026-10-08 verification delta
| Requirement | Implementation | Verification |
|---|---|---|
| Estimate approval source freshness | `backend/app/estimate_integrity.py` integrated into `workspace_api.py` | Targeted tests passed; database integration unverified |
| Rate effective-date calendar validation | `RateInput` field validator + approval-time validation | Targeted tests passed |
| Calibration company/project/drawing/revision/page validation at approval | Existing approval API + integrity module | Targeted tests passed; PostgreSQL triggers unverified |
| Dependency installation / lockfile | Existing manifests and bootstrap script | Blocked: npm EAI_AGAIN, pyotp missing; no fabricated lockfile |
| Full product acceptance | Existing application plus modules | Not verified; incomplete |

### Continuation 2026-10-08
| Feature | Implementation | Verification |
|---|---|---|
| Approved estimate DB immutability | Additive migration 0011, header and line guards | Static regression tested; PostgreSQL NOT executed |
| Company cost rate creation UI | Next.js estimating page wired to existing authenticated rate POST API | Code present; Next.js build/browser NOT executed |
| Migration smoke guard assertions | PostgreSQL script checks approval triggers | Script compile only; PostgreSQL NOT executed |

### 2026-10-08 additional feature traceability
| Feature | Backend/API | Migration | Frontend | Verification | Status |
|---|---|---|---|---|---|
| Project RFI register and lifecycle | `app/rfi_api.py`, scoped GET/POST/answer/close, audit | `0012_project_rfis.py` (RLS + project trigger) | `/projects/[companyId]/[projectId]/rfis` | Static contracts only; no DB/browser | Implemented, unverified |
| Cross-module project overview | `app/project_overview_api.py`, tenant-scoped persisted metrics | Uses existing + 0012 | `/projects/[companyId]/[projectId]/overview` | Static contracts only | Implemented, unverified |
| Unified workspace navigation | Existing takeoff page linked to overview, estimating, scheduling, costs, issues, RFIs | None | Takeoff + overview | Build blocked | Partial |
| IFC/3D, DDC, AI agents, billing, client portal | Not complete | Not complete | Not complete | Not verified | Missing/partial |


## 2026-10-08 maximum execution continuation
5D forecast revision: IMPLEMENTED BUT UNVERIFIED (model, migration, API, UI, pure arithmetic tests). Full 5D: PARTIAL. IFC/BIM, AI agents, billing: MISSING/PARTIAL.
