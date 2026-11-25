# svc-sync: sync-unit-sim

Run `021023-unit-762b939`, 2026-11-25. Local developer machine (WSL2 Linux, Go 1.27.1, Node 22, Docker 29.1). Services are local processes; Postgres 17 is the infra dev compose stack under its own project name and port; nothing outside this machine is contacted. Duration 140 s.
Command: `go test -race -count=1 ./...  (unit, hub, simulation property tests; the Postgres integration test needs its own DSN and was not enabled)` in a `git archive` snapshot of the pinned SHAs (scripts/run-service-suites.sh, e2e-tests `be964ce` or later), exit code 0.
Result: **6 packages ok, 0 failed** (packages without test files are not counted).
Commits: svc-sync `762b939`, go-common `95088b5`, go-auth `b19b540`.
Artifact: `sync-unit-sim.log`.
