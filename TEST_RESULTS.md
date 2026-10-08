# Test results — latest execution (2026-10-08)
- PASS: `cd backend && python -m pytest -q tests/test_issues_contract.py tests/test_scheduling.py tests/test_cost_summary.py tests/test_boq_exports.py` — 15 passed in 0.20s.
- PASS: `cd backend && python -m compileall -q app alembic/versions`.
- BLOCKED: full backend pytest due missing pyotp; attempted `python -m pip install pyotp==2.9.0 --retries 0 --timeout 4` and no version was available from configured index.
- BLOCKED: `npm ci --offline` — frontend package-lock.json absent.
- NOT EXECUTED: PostgreSQL 16 migrations, RLS, Redis worker tests, Next.js build, browser E2E and PDF→BOQ acceptance workflow.

## 2026-10-08 runtime hardening
- `cd backend && python -m pytest -q tests/test_scheduling.py tests/test_cost_summary.py tests/test_boq_exports.py tests/test_migration_graph.py tests/test_migration_history_static.py` — **16 passed in 0.22s**.
- `python -m compileall -q backend/app backend/alembic backend/scripts` — **passed**.
- `cd backend && python -m pytest -q` — **exit 2**, collection error: `ModuleNotFoundError: No module named 'pyotp'`.
- `cd frontend && npm run build` — **exit 127**, `next: not found`.
- `cd frontend && npm install --package-lock-only --ignore-scripts --no-audit --no-fund --fetch-retries=0 --fetch-timeout=12000` — **failed**, npm registry `EAI_AGAIN`.
- PostgreSQL 16, Redis 7, browser E2E: **not executed**.

## 2026-10-08
- `python -m pytest -q backend/tests/test_runtime_contracts.py backend/tests/test_schedule_integration_static.py`: 7 passed.
- `python -m compileall -q backend/app backend/alembic backend/scripts backend/tests`: passed.
- `python -m pytest -q` from backend: collection error ModuleNotFoundError: pyotp (not a passing suite).
- `npm install --package-lock-only --offline --ignore-scripts --no-audit --no-fund`: ENOTCACHED for @types/node.
- `curl -I https://registry.npmjs.org/next`: DNS resolution failed.
- PostgreSQL, Redis, Next.js production build and browser E2E: not executed.

## 2026-10-08 executed checks
- `cd backend && python -m pytest -q tests/test_estimate_integrity.py tests/test_scheduling.py tests/test_cost_summary.py tests/test_workspace_calculations.py`: **32 passed in 0.07s**.
- `cd backend && python -m pytest -q`: **exit 2**; collection error `ModuleNotFoundError: No module named 'pyotp'`.
- `cd frontend && npm run build`: **exit 127**; `next: not found`.
- `python -m compileall -q backend/app backend/tests`: **exit 0**.
- npm registry lockfile request: **EAI_AGAIN**; pyotp pip install: **failed**.
- PostgreSQL 16 / Redis 7 / browser end-to-end: **not executed**.

2026-10-08 local execution (Python environment without pyotp, PostgreSQL or Docker):
- `cd backend && pytest -q tests/test_approved_estimate_guard_migration.py tests/test_approval_workflow_static.py tests/test_estimate_integrity.py tests/test_scheduling.py tests/test_cost_summary.py tests/test_boq_exports.py`: **30 passed**.
- `cd backend && python -m compileall -q app alembic tests`: **passed**.
- `cd frontend && npm view next version --fetch-retries=0 --fetch-timeout=3000`: **failed**, EAI_AGAIN DNS registry.npmjs.org.
- Full pytest, live PG16 migrations, Redis, frontend build and browser E2E: **not executed**.

## 2026-10-08 continuation (local Python 3.13; no PostgreSQL, Docker or installed frontend dependencies)
- `cd backend && pytest -q tests/test_rfi_overview_contract.py tests/test_scheduling.py tests/test_workspace_calculations.py tests/test_calibration_integrity.py tests/test_migration_graph.py tests/test_migration_history_static.py`: **33 passed in 0.08s**. These are pure/static tests, not live DB/API/browser checks.
- `python -m compileall -q backend/app backend/alembic backend/tests`: **passed**.
- `cd backend && pytest -q`: **exit 2**; `ModuleNotFoundError: No module named 'pyotp'` while collecting `tests/test_core.py`.
- `cd frontend && npm run build`: **exit 127**, `sh: next: not found`.
- `python -m pip install pyotp==2.9.0 --disable-pip-version-check --retries 0 --timeout 4 -q`: **failed** (package index unreachable/unavailable).
- `npm view next version --fetch-retries=0 --fetch-timeout=4000`: **failed** (EAI_AGAIN DNS).
- PostgreSQL migrations, restricted-role RLS, Redis, authenticated API/browser acceptance: **NOT EXECUTED**.


## 2026-10-08 maximum execution continuation
See session execution output; only actually run commands may be reported as passed.
