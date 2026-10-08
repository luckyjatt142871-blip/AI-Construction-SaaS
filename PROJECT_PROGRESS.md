# Project progress — 2026-10-08

## Existing baseline
Continues the previously delivered `ai-construction-saas-audited-phase1.zip` source. No original Git repository/commit metadata was available. No deployment performed.

## Implemented — verified by targeted unit/static tests
- `backend/app/authorization.py`: explicit, deny-by-default project-read, project-create and audit-read permissions for all nine organization roles. Cross-company IDs rejected at the policy layer. Wired to project creation and company audit endpoints.
- `backend/app/engines/quantities.py`: deterministic Decimal-based unit conversion, rectangle area/volume, simple polygon area with intersection checks, count, waste and extended cost, source-trace validation. No automated drawing detection or database persistence; **not a complete 2D takeoff module**.
- MFA recovery-code rotation now requires password + fresh TOTP with a locked user row, revokes old codes and records an event. Recovery-code login now locks the user row before code consumption. Requires real PostgreSQL concurrency testing.
- Removed usable default development database credentials from the backend settings fallback.

## Tests actually run
- 48 targeted tests passed (`test_quantities.py`, `test_authorization_policy.py`, `test_security_static.py`, `test_auth_security.py`, `test_phase1_regressions.py`). Note: many legacy tests are source/static checks, not live API tests.
- Full pytest collection **FAILED**: `ModuleNotFoundError: No module named 'pyotp'` from `test_core.py`.
- Python compilation passed.
- `pip install pyotp redis rq` **FAILED**: package index DNS unavailable.
- Docker executable not installed; PostgreSQL, Redis, migrations, RLS, frontend build, workers, SMTP, private storage, cross-tenant API integration, production verification **NOT VERIFIED**.

## Known critical blockers
1. Full dependency installation and complete pytest suite must pass in a networked Docker environment.
2. PostgreSQL RLS policies and migration/runtime role separation must be independently verified and hardened.
3. Full API integration tests for authentication, tenant isolation, MFA, CSRF, file access and IDOR remain missing.
4. Private storage and secure document ingestion not implemented.
5. Full frontend integration and production build not verified.
6. All major downstream SaaS modules remain incomplete.
7. MFA recovery-code rotation and replay prevention need concurrent integration tests; MFA enrollment and session step-up policies need further review.

## Next exact task
On Windows with Docker Desktop: run Compose infrastructure, install backend requirements, migrate database, execute full pytest; repair any migration/privilege errors. Then add PostgreSQL-backed two-tenant API integration tests before expanding downstream modules.

## Windows PowerShell
```powershell
Expand-Archive .\ai-construction-saas-continuation.zip -DestinationPath .\saas-work
cd .\saas-work\ai-construction-saas
Copy-Item .env.example .env
notepad .env
# Set unique POSTGRES_PASSWORD, APP_DB_PASSWORD and MFA_ENCRYPTION_KEY.
docker compose config
docker compose up -d db redis
docker compose --profile tools run --rm migrate
docker compose up --build -d api worker frontend
docker compose exec api python -m pytest -q
docker compose exec frontend npm run build
docker compose logs --tail=100
```
Existing database volumes require migration planning; don't delete them without backup. These commands were not executed here.

## Acceptance
**NOT APPROVED / NOT PRODUCTION-READY**. The user authorized later phases, but critical Phase 1 security verification remains incomplete.

## Session continuation — 2026-10-08 (project workspace increment)

### Implemented — NOT VERIFIED end-to-end
- Added tenant-owned SQLAlchemy models for drawings, measurements, company rate items, estimates and estimate lines (`backend/app/workspace_models.py`).
- Added Alembic revision `0003_workspace.py` with RLS enabled and forced on these new tenant tables. Updated `0002_auth_hardening.py` to avoid duplicate-column/table failures caused by the historical `0001` metadata-based migration. **Migration execution remains NOT VERIFIED.**
- Added authenticated, tenant-scoped APIs for project listing and renaming; drawing upload/list/private retrieval and manual calibration; measurement create/list/update; rate creation/list; estimate creation, line creation and computed BOQ retrieval.
- Added deterministic quantity and money calculations with validation (`workspace_calculations.py`). Rates explicitly require provenance and currency. Estimate lines snapshot unit rate, quantity and amount. No automatic pricing feeds or external restricted data were imported.
- Added Next.js company project listing/creation page and a drawing workspace page with upload, image point picking, calibration, measurement save and list. PDF is embedded as a preview only; **interactive PDF takeoff is NOT implemented**. Image coordinate system uses a fixed 1000×700 canvas and is **NOT yet safe for calibrated engineering measurement on arbitrary image dimensions**.
- Added basic upload magic-byte/size validation and per-request authenticated local private-file retrieval. **No antivirus, complete PDF parser validation, durable S3 adapter, or safe content processing**; local adapter is development-only.
- Added `python-multipart` dependency.

### Tests actually executed this session
- Previous targeted suite: **48 passed**.
- New pure calculation and document validation tests: **8 passed**.
- Full pytest: **FAILED at collection**, `ModuleNotFoundError: No module named 'pyotp'`; declared in requirements, but unavailable in current environment.
- New FastAPI endpoints, PostgreSQL migration, live RLS, Docker, Redis, Next.js build and workers: **NOT VERIFIED**.

### New critical blockers / known defects
1. Historical migration `0001` uses live `Base.metadata.create_all()` instead of frozen schema. Migration chain requires rework and real PostgreSQL verification before use on existing production data.
2. Tenant RLS SECURITY DEFINER membership function and least-privilege database role need live verification and adversarial tests.
3. Image viewer currently scales to a fixed 1000×700 coordinate system. Do not rely on takeoff measurements for real engineering quantities until image pixel coordinates, aspect ratio, PDF rendering and calibration are corrected and tested.
4. Private storage requires S3 adapter, malware scanning/quarantine, signed downloads where appropriate, size limits at proxy, and threat testing.
5. Authentication suite cannot collect without `pyotp` and the complete dependency set. Do not treat pure unit test results as API integration tests.
6. New estimate/BOQ endpoints are preliminary. No approval/revision/export or full financial workflow yet.
7. Frontend fetch/session/CORS and secure-cookie behavior must be validated in a browser with live services.

### Next exact implementation task
**First** fix the drawing image coordinate/calibration model to use original image pixel dimensions, implement authenticated PDF page rasterization, and add browser/API tests. **In parallel** run Docker Compose with PostgreSQL, Redis, migrate `0001`–`0003`, and execute full pytest with dependencies installed. Fix any migration/RLS failures before trusting tenant persistence.

## Session continuation — 2026-10-08 (native drawing coordinates + PDF measurement)

### Implemented — integration NOT VERIFIED
- Replaced the fixed 1000×700 stretched drawing canvas with an SVG viewBox based on the actual image or PDF page raster dimensions. Pointer positions use inverse SVG screen transformation, so CSS resizing, letterboxing, and zoom no longer directly distort persisted pixel coordinates. Added zoom buttons, reset and pointer-based pan mode.
- Added authenticated per-page PDF raster and metadata endpoints. Pages render at 144 DPI through PyMuPDF and are selectable in the Next.js takeoff workspace. Drawing points now persist their source `page_number` through Alembic revision `0004_drawing_pages.py`.
- Uploads validate image structure and PDF page dimensions before saving. Rendering has page-count and pixel-count limits, but **must be moved into a resource-restricted sandbox/worker for production**.
- Calibration now uses two points picked on the drawing rather than a manually typed pixel distance. PDF page 1 may be calibrated; other PDF pages are deliberately rejected for measurement until per-page calibration records are implemented. The UI currently displays the drawing-level scale on all pages, so it MUST NOT be used for engineering measurement on pages other than page 1.
- Added pure geometry and actual PDF rasterization tests. No claim of engineering accuracy benchmark validation.

### Tests executed
- `pytest -q tests/test_drawing_geometry.py tests/test_workspace_calculations.py tests/test_quantities.py tests/test_authorization_policy.py`: **52 passed**.
- Full `pytest -q`: **FAILED during collection** (`pyotp` missing). The project declares `pyotp>=2.9,<3`; this environment cannot install all dependencies. No attempt to fake or vendor pyotp.
- Python compilation: passed.
- PostgreSQL, Alembic live migrations, RLS, Redis, Docker, frontend build and browser E2E: **NOT VERIFIED**.

### Critical remaining blockers / exact next tasks
1. Implement **per-PDF-page calibration** with a tenant-owned scale table or per-page calibration column and migration; reject any measurement without calibration for its own page. Add page bounds validation and image rotation tests.
2. Install backend dependencies in network-enabled Docker, run complete pytest; fix actual failures and run live PostgreSQL migrations and adversarial RLS/cross-tenant API tests.
3. Run `npm ci`/`npm install` and `npm run build`, then browser E2E for image/PDF pointer mapping, pan and zoom.
4. Replace synchronous PDF processing with sandboxed asynchronous conversion and quarantine/malware scanning. Add private S3-compatible storage.
5. Continue full estimating approval/revision/BOQ export workflows, IFC/BIM and remaining modules. None are production-ready.

**Acceptance: NOT APPROVED.**

## Session continuation — 2026-10-08 (page-specific PDF calibration)

### Source changes — implemented, live integration NOT VERIFIED
- Added `drawing_calibrations` immutable versioned history, scoped to company, project, drawing, drawing revision and page; migrations in `0005_page_calibrations.py`.
- Measurements now retain `calibration_id` foreign key; existing historical measurements remain NULL/unverified rather than inheriting an incorrect page scale.
- Replaced the drawing-global calibration endpoint's write with per-page calibration history; row-locks the drawing during calibration creation, and returns calibration ID, version, page and drawing revision.
- Measurement create/update reject missing page-specific calibration, out-of-range PDF pages, and points outside native page dimensions. Saved measurements bind to the exact calibration record used.
- Next.js takeoff workspace fetches the current page's calibration independently, sends the page on calibration, disables saving uncalibrated pages, and displays calibration provenance.
- Corrected `0004_drawing_pages` down_revision from nonexistent `0003_workspace` to actual `0003`. Added static migration graph regression test.
- Added pure invariant tests for cross-page/drawing/revision calibration rejection and native resolution-independent quantity calculations.
- Added `FEATURE_MATRIX.md` reflecting implementation vs verification.

### Tests actually executed
- Targeted pytest: **64 passed** (`test_page_calibrations`, `test_migration_graph`, `test_drawing_geometry`, `test_workspace_calculations`, `test_quantities`, `test_authorization_policy`).
- Full backend pytest: **FAILED collection** (`ModuleNotFoundError: pyotp`). `pyotp` remains declared in `backend/requirements.txt`; it is not installed here. No claim of reproducible dependency install.
- Next.js `npm run build`: **FAILED**, `next: not found` (node_modules unavailable).
- Python compilation: passed. Migration graph checked statically, **NOT** against PostgreSQL.
- Docker, PostgreSQL migrations/RLS, Redis, browser E2E and live tenant API integration: **NOT VERIFIED — environment limitation**.

### Known limitations and next exact task
1. In a network-enabled Docker environment, install dependencies and run the complete backend pytest suite, then `alembic upgrade head` against a disposable PostgreSQL database and verify RLS with the restricted runtime role. Fix any actual failures.
2. Add PostgreSQL-backed cross-tenant tests for page calibration, measurement IDOR, revocation and concurrent recalibration. Check DB foreign keys enforce matching tenant/project/drawing across references (current single-column FKs do **not** enforce all of these composite invariants).
3. Run Next.js production build and browser tests for page switching, zoom/pan and correct scale display.
4. Implement revision lifecycle: current drawing revision is initialized to 1; there is no version-upload endpoint yet. New uploads receive new drawing IDs. Never assume same drawing ID implies a new revision without explicit implementation.
5. Implement private S3 storage, upload quarantine/malware scanning and isolated PDF rendering before production use.
6. Continue approvals, financial provenance/revisions, BOQ exports, IFC/BIM and remaining product modules in dependency order.

**Acceptance: NOT APPROVED — NOT PRODUCTION READY.**

## Session continuation — calibration integrity hardening (2026-10-08)

### Implemented — PostgreSQL integration NOT VERIFIED
- Added Alembic `0006_calibration_integrity.py`: measurement `drawing_revision` source column; PostgreSQL trigger rejects new measurements without a calibration and rejects mismatched calibration tenant/project/drawing/revision/page; calibration history records are immutable. Historical NULL-calibration measurements remain unverified and cannot be remeasured. Migration has NOT been run against PostgreSQL.
- Measurement create/update now validates exact calibration identity via `calibrated_quantity`, native page bounds, and stores drawing revision with the measurement. Measurement listing exposes drawing revision.
- Pure coordinate validation now rejects NaN/infinity and malformed point data.
- Added `test_calibration_integrity.py` with page/revision separation, invalid coordinate cases, and migration structural assertions.

### Tests actually run
- Targeted pytest: **35 passed** (`test_calibration_integrity.py`, `test_page_calibrations.py`, `test_drawing_geometry.py`, `test_workspace_calculations.py`). These are pure/static tests, not live API/PostgreSQL integration tests.
- Python compileall: **PASS**.
- Full backend pytest: **FAILED during collection** (`ModuleNotFoundError: pyotp`); this environment lacks required dependencies. Docker and psql commands are unavailable.
- PostgreSQL migration/trigger execution, RLS, Redis, browser E2E, Next.js production build: **NOT VERIFIED**.

### Exact next task
Install full backend dependencies in a network-enabled environment and execute complete pytest. Then run Alembic `upgrade head` on a disposable PostgreSQL database and add adversarial SQL/API tests proving that the `0006` trigger blocks cross-page, cross-revision, cross-project and cross-tenant calibration swaps; fix any live migration or RLS defects. Subsequently implement private object storage and end-to-end estimating approvals/BOQ exports.

**Acceptance: NOT APPROVED — NOT PRODUCTION-READY.**

## Session continuation — estimating approvals, exports and CI (2026-10-08)

### Implemented (integration UNVERIFIED)
- Added estimate approval API with database row locking, validation of nonempty lines, verified measurement calibrations, rate provenance dates, unit/currency consistency and deterministic line arithmetic. Approval stores approver and UTC timestamp; new revisions copy lines into a new DRAFT estimate while leaving approved records intact.
- Added tenant-scoped estimate list, revise, approve, and approved-only PDF/Excel BOQ export endpoints. Added `backend/app/boq_exports.py` with openpyxl and ReportLab.
- Added `rate_date` to cost rate input, storage and listing, with nullable historical migration for existing rates. Historical rates without dates cannot be used for newly approved estimates.
- Added migration `0007_estimate_approvals.py` and connected Next.js estimating/BOQ page at `/projects/[companyId]/[projectId]/estimates`.
- Added `.github/workflows/backend-integration.yml` with PostgreSQL 16 and Redis 7 service containers, full backend dependency installation, Alembic migration, full pytest, RLS metadata verification and frontend build job. Fixed Docker Compose migration working directory `/app` -> `/srv`.
- `pyotp>=2.9,<3` remains correctly declared in backend requirements; CI installs it via pip. Local environment still cannot import pyotp. **CI has not run and must not be represented as passing.**

### Tests run in this session
- Targeted tests for exports, approval structural checks, workspace arithmetic and calibration: **20 passed**.
- Full pytest: **FAILED during collection** because pyotp is absent in local environment.
- Live PostgreSQL migration, trigger behavior, adversarial RLS, Redis worker, API integration, Next.js build and browser E2E: **NOT VERIFIED**.

### Exact next task
Run the GitHub Actions integration workflow in a repository with network access. Fix actual migration, RLS, and API failures; add two-tenant API tests that verify approval/export IDOR protection. Complete user-facing rate creation and quantity review; secure object storage and worker isolation before deployment. Then continue IFC/BIM and downstream modules. The entire end-to-end estimating workflow is **NOT yet verified**.

**Acceptance: NOT APPROVED / NOT PRODUCTION READY.**

## 2026-10-08 — Deterministic Alembic historical-schema increment

- Replaced unsafe `Base.metadata.create_all()` in `0001` with explicit historical DDL. Historical 0001 excludes columns introduced in later revisions.
- Replaced live-ORM table creation in `0003` with explicit historical DDL for drawings, measurements, rate items, estimates and estimate lines.
- Kept revision IDs unchanged. Existing databases stamped at earlier revisions are not reset or dropped; later revisions inspect existing columns/tables where necessary to accommodate legacy `create_all` deployments.
- Added migration-history static regression tests and a **disposable PostgreSQL-only** 0003→head upgrade test script. CI now provisions an additional PostgreSQL database for this script.
- **Tested locally:** 10 targeted migration/security tests passed; Python compilation passed.
- **Blocked/unverified:** PostgreSQL executable and Docker absent locally, so fresh and existing live migrations, RLS isolation, trigger behavior and full backend integration remain unverified. Complete pytest still blocked by unavailable pyotp. Next.js dependencies not installed; frontend build not verified.
- **Caution:** historical deployments stamped at 0001 may contain later-version objects created by the old dynamic migration. Upgrade compatibility has been improved but requires testing against snapshots of those databases before deployment. Do not run destructive migration tests against real customer data.
- **Exact next task:** Execute GitHub Actions PostgreSQL integration workflow, repair observed migration errors, test legacy database snapshots and runtime non-owner tenant role RLS. Then run full API/browser estimating workflow and add adversarial calibration/approval tests.
- **Overall status:** IN PROGRESS; NOT PRODUCTION READY.

## Continuation — 2026-10-08 (critical-path calculation increment)

### Implemented and tested
- Added `backend/app/engines/scheduling.py`: deterministic, pure critical-path method (CPM) with topological cycle detection, finish-to-start predecessors, earliest/latest starts and finishes, total float and critical-path flags. Durations are integer day offsets; calendar date handling and resource leveling are intentionally excluded.
- Added `backend/tests/test_scheduling.py` with nine executable tests covering branching dependencies, float, milestones, empty schedules and invalid graph inputs.
- `python -m pytest -q tests/test_scheduling.py`: **9 passed** in this environment.

### Unverified and blockers
- Full backend `python -m pytest -q`: **FAILED during collection**, missing `pyotp` package. Existing `backend/requirements.txt` declares `pyotp>=2.9,<3`, but the local interpreter has no installed package. Do not fake the dependency or call full suite passing.
- Frontend dependencies (`node_modules`) absent; frontend build not executed.
- PostgreSQL, Redis and Docker executables unavailable; live migration, RLS, trigger, integration and browser tests not executed. Existing CI configuration is not evidence of passing tests.
- Scheduling calculation is a standalone verified engine only; it is **NOT** a persisted 4D scheduling module. No API, migration, Gantt UI or BIM link was added in this increment.

### Exact next implementation task
1. Run the existing GitHub Actions PostgreSQL 16/Redis 7 workflow and repair real migration errors; do not alter deployed databases without backups and compatibility validation.
2. Install backend requirements in a network-enabled environment, run the complete suite, fix all failures; run frontend `npm install && npm run build`.
3. Implement tenant-owned schedule/activity/dependency models, Alembic migration and RLS; expose authenticated CRUD and critical-path API using the tested calculation engine; connect a Next.js Gantt page and add PostgreSQL-backed cross-tenant tests.
4. Finish and integration-test the existing project → PDF → takeoff → estimate approval → BOQ exports workflow before claiming it works end to end.

**Overall product status: NOT PRODUCTION-READY.**

## 2026-10-08 — Project scheduling vertical integration

**Implemented — integration unverified:** `backend/app/schedule_models.py`, `backend/app/schedule_api.py`, `backend/alembic/versions/0008_project_scheduling.py`, `frontend/app/projects/[companyId]/[projectId]/schedule/page.tsx`. Tenant-scoped activities, progress, zero-duration milestones, finish-to-start dependencies, create/edit/delete APIs, CPM calculations, Gantt day-offset view, PostgreSQL RLS and scope trigger. Added scheduling link from estimates page. The RLS policy and trigger have **not** been exercised against PostgreSQL. Schedule is not calendar-aware, has no baselines or BIM links, and therefore is not the complete 4D specification.

**Dependency setup:** `backend/requirements.txt` already declares `pyotp>=2.9,<3`. Added `backend/scripts/install_and_verify.sh` to create a venv, install all dependencies and run pytest in a network-enabled environment. Local package installation remains blocked/unperformed; no vendored fake pyotp.

**Executed:** `cd backend && python -m pytest -q tests/test_scheduling.py tests/test_schedule_integration_static.py tests/test_migration_graph.py tests/test_migration_history_static.py` — **16 passed**. `python -m pytest -q` — **FAILED collecting test_core.py** (`ModuleNotFoundError: pyotp`). `cd frontend && npm run build` — **FAILED** (`next: not found`). Docker and psql executables absent. No live PostgreSQL, Redis, browser, tenant isolation, RLS or complete estimating workflow tests passed or were claimed to pass.

**Critical caveat:** The new 0008 migration requires PostgreSQL verification including upgrade from 0007 and scope-trigger tests; don't deploy without this. Existing legacy migration compatibility remains unverified.

**Exact next task:** Provision PostgreSQL 16/Redis 7 via existing CI; install backend/frontend dependencies; execute fresh and 0007→0008 migrations; add live API cross-tenant scheduling tests; fix observed failures; run full project→PDF→takeoff→estimate→BOQ browser test. Then extend calendar-aware Gantt, IFC/BIM and other modules.

**Acceptance:** NOT APPROVED / NOT PRODUCTION READY.

## Session continuation — 2026-10-08 (5D connected cost ledger)

### Implemented (NOT live integration verified)
- Added 5D project budget and actual/committed cost ledger (`backend/app/cost_models.py`, `backend/app/cost_api.py`, `backend/app/engines/cost_ledger.py`). Each record is company/project scoped; currency must match the project budget, and every cost entry requires a source reference and date. Budget arithmetic uses Decimal and does not invent rates or forecasts.
- Added Alembic `0009_project_costs.py` with tenant RLS, FORCE RLS, indexes, and a project-company scope trigger. The migration is not yet executed on PostgreSQL.
- Added API-connected Next.js `/projects/[companyId]/[projectId]/costs` budget/ledger page, including forms, error states, and a summary.
- Extended PostgreSQL RLS metadata checker to include schedule and 5D tables.
- Existing GitHub Actions workflow installs all declared backend requirements including pyotp, PostgreSQL 16 and Redis 7, runs migrations and pytest, and runs frontend npm install/build; workflow NOT executed here.

### Actual tests
- `cd backend && python -m pytest -q tests/test_scheduling.py tests/test_cost_summary.py tests/test_cost_migration_static.py tests/test_workspace_calculations.py tests/test_migration_graph.py`: **21 passed** (pure/static tests; not live API or PostgreSQL tests).
- `cd backend && python -m pytest -q`: **FAILED during collection** because pyotp is missing from this environment. It is already declared in requirements.txt. Do not vendor an insecure replacement.
- `cd frontend && npm run build`: **FAILED**, `next: not found` because node_modules is absent.
- `python -m compileall -q app alembic tests`: passed.
- Docker and PostgreSQL clients absent; live migration, RLS, tenant isolation, PDF-to-BOQ browser workflow and Redis tests remain **NOT EXECUTED**.

### Next exact task
Run `.github/workflows/backend-integration.yml` in a networked GitHub Actions runner. Repair failures in migration chain 0001–0009, PostgreSQL scope triggers and real tenant authorization, then add and run a multi-tenant FastAPI integration test for 5D budgets and PDF→BOQ. Complete project cost baselines, schedule cost-loading, IFC/BIM and remaining master-spec modules. No production deployment.

## 2026-10-08 — project issue register vertical increment
- **IMPLEMENTED — INTEGRATION UNVERIFIED:** Added tenant/project-owned issue records with reference, title, description, priority and status; migration `0010_project_issues.py`; authorized list/create/update FastAPI endpoints; and connected Next.js project issue register page.
- Scope protection: company membership and project ownership checked on each endpoint, resource queries include company_id and project_id; PostgreSQL migration enables FORCE RLS and project-scope trigger. This has NOT been exercised on live PostgreSQL.
- Tests actually executed: `cd backend && python -m pytest -q tests/test_issues_contract.py tests/test_scheduling.py tests/test_cost_summary.py tests/test_boq_exports.py` => **15 passed** (unit/static; no live API/database tests). `python -m compileall -q app alembic/versions` => passed.
- Backend `pyotp` install attempt failed in this environment (package index unavailable). `npm ci --offline` failed because no `package-lock.json` exists. Node 22 and npm ARE available, but dependencies are not installed. Docker and PostgreSQL executables are unavailable; no migrations, RLS or browser E2E were run. CI workflow exists but has NOT been executed here.
- **BLOCKED:** complete backend pytest, migrations 0001–0010, tenant RLS, Redis, browser build, end-to-end estimating workflow. This increment does not resolve them.
- **NEXT EXACT TASK:** In network-enabled GitHub Actions run backend-integration.yml; repair failing Alembic migrations and tenant tests; generate/commit a valid frontend lockfile using npm install; run npm ci, typecheck and build; then add actual FastAPI+PostgreSQL integration tests for issue register and the PDF-to-BOQ acceptance workflow. Do not deploy until verified.

## 2026-10-08 dependency and runtime integration hardening
- IMPLEMENTED: Backend Docker image now includes Alembic configuration/revisions, scripts, tests and pytest configuration, enabling its migration command to find the source files.
- IMPLEMENTED: GitHub Actions provisions a restricted PostgreSQL runtime login and invokes `backend/scripts/test_rls_runtime.py` after migrations. The test verifies deny-by-default and cross-company project SELECT/INSERT under a non-owner role in a rollback-only transaction.
- IMPLEMENTED: Frontend Docker/CI switched to `npm ci` to require a lockfile; frontend `.npmrc` added. **BLOCKED:** npm registry DNS EAI_AGAIN prevented generating a valid package-lock.json in this environment; `npm ci` cannot pass until lockfile generation in a network-enabled environment. No lockfile has been fabricated.
- TESTED: 16 targeted pytest cases passed; Python compileall passed. Full pytest collection fails on missing pyotp. Next.js build fails because `next` is not installed.
- NOT VERIFIED: PostgreSQL 16 migration chain 0001-0010, runtime RLS test, calibration triggers, end-to-end PDF/BOQ and browser E2E. Docker and psql are not installed locally.
- NEXT: In network-enabled CI, run `cd frontend && npm install --package-lock-only && npm ci`, commit the generated lockfile; run backend CI with PostgreSQL 16 and Redis 7, fix observed failures and add browser E2E tests.

## 2026-10-08 dependency/bootstrap and verification hardening
- Added frontend/install-locked.sh: generates an actual npm lockfile when missing (requires registry access), then runs npm ci.
- CI now generates a missing npm lockfile from the registry, uploads it for review, and executes npm ci; Docker build follows the same bootstrap behavior. **No committed lockfile**: registry DNS unavailable here; commit CI-generated lockfile before treating installs as strictly reproducible.
- Extended PostgreSQL migration smoke test to check RLS flags for scheduling, costing and issues tables and scope triggers at revision 0010.
- Added backend/tests/test_runtime_contracts.py; 7 focused tests passed including existing schedule integration static tests; compileall passed.
- Full pytest blocked at collection by missing pyotp; npm offline lock generation failed ENOTCACHED, and registry DNS resolution failed. No live PostgreSQL/Redis, frontend build, or browser E2E tests executed.
- NEXT: run network-enabled CI, commit generated lockfile, execute migrations/RLS, fix observed failures, verify PDF-to-BOQ browser workflow.

## 2026-10-08 continuation — approval integrity
- IMPLEMENTED, targeted TESTED: estimate approval API invokes strict source snapshot validation for calibration identity, quantity/rate freshness, provenance, ISO calendar date, units and currency. Approval rejects stale lines rather than silently approving inconsistent amounts.
- TESTED: 32 targeted tests passed (`test_estimate_integrity.py`, `test_scheduling.py`, `test_cost_summary.py`, `test_workspace_calculations.py`). Python compileall passed.
- BLOCKED: complete pytest collection (pyotp missing), npm registry EAI_AGAIN (no valid package-lock), Next.js build (`next` unavailable), live PostgreSQL 16 / Redis / browser acceptance tests (not available here).
- NOT VERIFIED: actual PostgreSQL migrations, RLS, tenant-scoped APIs and end-to-end PDF/BOQ browser workflow. Do not deploy without these tests.

## 2026-10-08 continuation: approval immutability and rate entry
- Added additive Alembic `0011_approved_estimate_guard` to reject direct database modifications to approved estimate headers and line snapshots. **Implemented, PostgreSQL execution not verified.** Existing approved records are not rewritten.
- Connected the existing Next.js estimating page to the existing authorized company rate-creation endpoint, including material/labour/equipment, currency, source provenance and effective date. **Implemented, frontend build/browser unverified.**
- Expanded disposable migration smoke checks to require the new approval guards.
- Focused pytest: 30 passed; compileall passed. npm registry lookup failed EAI_AGAIN; full backend and live PostgreSQL/browser integration not executed. No genuine npm lockfile was created.
- Still missing: full application acceptance workflows, IFC/BIM, advanced 4D/5D, AI agents, billing, client portal, and production verification.

## 2026-10-08 continuation: integrated RFIs and persisted project overview
- Added `project_rfis` with migration `0012_project_rfis`, forced company RLS, project-consistency trigger, unique project reference and status constraint. **Implemented; live PostgreSQL not verified.**
- Added authorized RFI creation, listing, answer-once, close-after-answer APIs, row locking, actor/time provenance and audit actions. **Implemented; API integration not verified.**
- Added connected Next.js RFI page and project overview page; linked workspaces from drawing takeoff. **Implemented; production build/browser unverified.**
- Added project overview API aggregating persisted drawings, measurements, estimates, scheduling, costs, issues and RFIs. Mixed-currency costs are explicitly withheld from aggregate totals. **Implemented; live DB unverified.**
- Expanded migration and RLS metadata checks for `project_rfis`; these checks have not run against PostgreSQL.
- Targeted pytest: 33 passed. Full pytest: collection error due to missing pyotp. Frontend build: `next` missing. Docker/psql not installed; npm registry DNS fails. **Not production ready.**
- Next priority: network-enabled dependency install, genuine npm lockfile, PostgreSQL 16 migrations 0001–0012 and adversarial RLS, then authenticated browser PDF-to-BOQ workflow. Other master-scope modules remain partial or missing.


## 2026-10-08 maximum execution continuation
Implemented (unverified): persisted versioned 5D forecast assumptions, EAC API and UI, migration 0013; corrected schedule activity ID response. Live database/browser verification pending.
