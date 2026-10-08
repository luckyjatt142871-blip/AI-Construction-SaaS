# Production readiness checklist — 2026-10-08

- [ ] Install and pin backend dependencies in network-enabled CI; full pytest green.
- [ ] Generate genuine `frontend/package-lock.json` with registry; commit it and verify `npm ci`, typecheck, build.
- [ ] PostgreSQL 16 clean migrations 0001–0012 and existing-revision upgrade checks.
- [ ] Restricted-role cross-company, cross-project, cross-page RLS and calibration trigger adversarial tests.
- [ ] Authenticated API and browser PDF → takeoff → estimate → approval → BOQ PDF/XLSX E2E.
- [ ] Verify RFI, scheduling, cost, issue, project overview persistence and authorization in PostgreSQL.
- [ ] Verify Redis workers, retry/idempotency, private storage and scanning.
- [ ] Implement and verify IFC/3D, 4D/5D advanced integrations, DDC licensing, AI agents, client portal and sandbox billing.
- [ ] Threat model, backup/restore rehearsal, observability, secrets, performance, operational runbook.

Not production-ready. Do not deploy with unverified migrations, RLS or billing.
