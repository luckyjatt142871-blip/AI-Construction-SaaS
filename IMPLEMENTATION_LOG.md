# Implementation log
2026-10-08: Continued from cost-ledger ZIP, preserving earlier modules. Added project issue register vertical (model, migration, authorized APIs, Next.js page, static contract tests). No live database/API/browser integration verification performed.

## 2026-10-08
Corrected backend Docker build context to include migrations and test scripts; introduced restricted-role RLS CI test and runtime role provisioning; changed frontend CI/Docker to fail-closed npm ci; added reproducible local verification helper. No unverified integration result is claimed.

## 2026-10-08
- Added frontend/install-locked.sh and CI/Docker real-registry lockfile bootstrap.
- Extended disposable PostgreSQL migration checks to include scheduling, costing and issue RLS and scope triggers.
- Added runtime harness contract tests and recorded exact local blockers.

## 2026-10-08 — estimate approval provenance hardening
- Added `backend/app/estimate_integrity.py`, used by the existing authorized estimate approval API.
- Approval now rejects stale saved quantities, changed rates, impossible rate dates and calibration references that do not match company/project/drawing/revision/page. This preserves the existing estimate/BOQ API contract and does not mutate prior migrations.
- Added `backend/tests/test_estimate_integrity.py` with positive and adversarial regression cases.
- Attempted live dependency resolution; npm registry returned EAI_AGAIN; pip could not resolve pyotp from configured index. Did not generate a fake lockfile.

2026-10-08: Added migration 0011 to guard approved estimate headers and lines at DB level; connected company rate creation in estimating UI; extended migration smoke verification; added focused regression tests. Existing source preserved.

2026-10-08: Added tenant-scoped RFI persistence/API/frontend and project overview API/frontend; migration 0012 and project navigation; expanded migration checks. Ran targeted regression tests and attempted full pytest, npm build and dependency installation. No PostgreSQL/Redis/browser integration was executed.


## 2026-10-08 maximum execution continuation
Added 0013 project_cost_forecasts, authenticated project-scoped forecast endpoints, Next.js forecast form/history, deterministic EAC/variance arithmetic and tests; fixed scheduling activity ID type.
