# svc-tenancy: tenancy-unit-contract

Run `020947-unit-0084d7f`, 2026-11-25. Local developer machine (WSL2 Linux, Go 1.27.1, Node 22, Docker 29.1). Services are local processes; Postgres 17 is the infra dev compose stack under its own project name and port; nothing outside this machine is contacted. Duration 7 s.
Command: `GOWORK=off go test -race -count=1 ./...  (module's own pins)` in a `git archive` snapshot of the pinned SHAs (scripts/run-service-suites.sh, e2e-tests `be964ce` or later), exit code 0.
Result: **4 packages ok, 0 failed** (packages without test files are not counted).
Commits: svc-tenancy `0084d7f`.
Artifact: `tenancy-unit-contract.log`.
