# AI Construction Estimating SaaS — Phase 1 (INCOMPLETE)

This is an unfinished security foundation, **not** a production-ready application. Phase 2 has not started.

## Windows (PowerShell + Docker Desktop)
```powershell
Expand-Archive .\ai-construction-saas-phase1-hardening.zip -DestinationPath .\source
cd .\source\ai-construction-saas
Copy-Item .env.example .env
notepad .env
# Set POSTGRES_PASSWORD and a DIFFERENT APP_DB_PASSWORD
docker compose config
docker compose up -d db redis
docker compose --profile tools run --rm migrate
docker compose up --build -d api worker frontend
docker compose ps
docker compose exec api python -m pytest -q
docker compose logs --tail=100 api worker
```
Docker Desktop must be running. The roles initialization script runs **only on a new PostgreSQL volume**. Existing volumes require a carefully reviewed manual migration; never delete customer data to recreate them.

## Verification caveats
Pytest collection currently fails when optional packages are missing in the execution environment; Docker/PostgreSQL/Redis and end-to-end flows are unverified. The MFA recovery codes currently have no login redemption endpoint; email delivery and reset tokens have not been integration tested. Platform-wide admin company listing intentionally returns 501 until a dedicated audited connection is implemented. Do not deploy.

See docs/TESTING.md and docs/SECURITY.md for blockers.

## Latest security audit
See `docs/PHASE1_SECURITY_AUDIT.md` for implemented hardening and critical unresolved blockers. **Phase 1 is NOT APPROVED.** Do not deploy commercially or upload sensitive customer documents.


## 2026-10-08 source audit
See `docs/PHASE0_AUDIT.md`, `docs/FEATURE_MATRIX.md`, `docs/IMPLEMENTATION_ROADMAP.md`, `docs/DDC_INTEGRATION_MATRIX.md` and `THIRD_PARTY_LICENSES.md`. This remains an incomplete Phase 1 foundation, not production-ready.

## 2026-10-08 continuation
See [`PROJECT_PROGRESS.md`](PROJECT_PROGRESS.md) for exact code changes, 48 targeted passing tests, full-suite dependency failure, environment limitations and Windows Docker commands. This is an **incomplete, not production-ready** system. The new quantity functions are a small independently tested calculation engine, **not** a working takeoff workflow.

## Workspace development increment (unverified integration)

New APIs: `GET /api/companies/{company_id}/projects`, `PATCH /api/companies/{company_id}/projects/{project_id}`, drawing upload/list/file/calibration under `/api/companies/{company_id}/projects/{project_id}/drawings`, measurement create/list/update, company rates, project estimates, estimate lines and `/boq`.

The Next.js `/projects` and `/projects/{companyId}/{projectId}` pages use these APIs. **These workflows have not been run against PostgreSQL, Redis or a browser.** The image takeoff canvas is preliminary and its fixed dimensions are not suitable for accurate measurements on arbitrary images. No automatic AI/CAD/IFC functionality is claimed. See `PROJECT_PROGRESS.md` for blockers and validation commands.
