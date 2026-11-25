# client-core-sync: unit and simulation tests

Run `021243-unit-f6bb266`, 2026-11-25. Local developer machine (WSL2 Linux, Go 1.27.1, Node 22, Docker 29.1). Services are local processes; Postgres 17 is the infra dev compose stack under its own project name and port; nothing outside this machine is contacted. Duration 13 s. Command `pnpm vitest run`, tree clean at `f6bb266`.
Result: **115 passed, 0 failed, 1 skipped** (the skipped test is the Go-devstack e2e, which needs SYNC_E2E_FIXTURE and is recorded in its own run).
Artifact: `client-core-sync-unit.log`.
