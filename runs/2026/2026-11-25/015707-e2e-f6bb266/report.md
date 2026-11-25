# client-core sync e2e against the svc-sync devstack

Run `015707-e2e-f6bb266`, 2026-11-25. Local developer machine (WSL2 Linux, Go 1.27.1, Node 22, Docker 29.1). Services are local processes; Postgres 17 is the infra dev compose stack under its own project name and port; nothing outside this machine is contacted.
The `cmd/devstack` binary (svc-sync `762b939`, built in place from a tree that was clean at build time; later commits and edits in that repo came after) serves a gateway and a hub on 127.0.0.1:18111/18112 with the identity stub and a test token issuer; the TypeScript client (client-core-sync `f6bb266`, clean) runs `src/sync/e2e.go.test.ts` against it.

Result: **1 test passed, 0 failed** (1 test file), 8 s.

Known gap, not a failure: the gateway authenticates through `identitystub`, not the real svc-identity, so this is not a three-service flow.

Artifacts: `sync-client-e2e.log`, `sync-devstack.log`.
