# svc-tenancy: tenancy-integration

Run `020954-integration-0084d7f`, 2026-11-25. Local developer machine (WSL2 Linux, Go 1.27.1, Node 22, Docker 29.1). Services are local processes; Postgres 17 is the infra dev compose stack under its own project name and port; nothing outside this machine is contacted. Duration 14 s.
Command: `GOWORK=off TENANCY_TEST_PG_ADMIN_DSN=... go test -race -count=1 -tags integration ./...  (real Postgres 17: contract, isolation, roles, registry)` in a `git archive` snapshot of the pinned SHAs (scripts/run-service-suites.sh, e2e-tests `be964ce` or later), exit code 0.
Result: **6 packages ok, 0 failed** (packages without test files are not counted).
Commits: svc-tenancy `0084d7f`.
Artifact: `tenancy-integration.log`.
