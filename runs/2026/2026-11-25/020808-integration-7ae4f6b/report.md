# svc-identity: identity-integration

Run `020808-integration-7ae4f6b`, 2026-11-25. Local developer machine (WSL2 Linux, Go 1.27.1, Node 22, Docker 29.1). Services are local processes; Postgres 17 is the infra dev compose stack under its own project name and port; nothing outside this machine is contacted. Duration 99 s.
Command: `HOSTRY_PG_DSN=... go test -race -count=1 -tags integration ./integration/... ./contract/...  (real Postgres 17)` in a `git archive` snapshot of the pinned SHAs (scripts/run-service-suites.sh, e2e-tests `be964ce` or later), exit code 0.
Result: **2 packages ok, 0 failed** (packages without test files are not counted).
Commits: svc-identity `7ae4f6b`, go-common `95088b5`, go-auth `b19b540`, api-specs `c92ca77`.
Artifact: `identity-integration.log`.
