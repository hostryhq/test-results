# Test results index

One line per run, newest first. Append-only: a wrong record is corrected by a new run entry that references it.

- 2026-11-25 051055-e2e-1b14369 | e2e | soak seed 4 print (svc-billing patched with one line (go build overlay)) | fail | pass 6 fail 8 skip 2 | R11 soak T1-T13 | runs/2026/2026-11-25/051055-e2e-1b14369/report.md
- 2026-11-25 050803-e2e-1b14369 | e2e | soak seed 3 settle (svc-billing patched with one line (go build overlay)) | fail | pass 7 fail 7 skip 2 | R11 soak T1-T13 | runs/2026/2026-11-25/050803-e2e-1b14369/report.md
- 2026-11-25 050451-e2e-1b14369 | e2e | soak seed 2 settle with cloud restore (svc-billing patched with one line (go build overlay)) | fail | pass 8 fail 7 skip 1 | R11 soak T1-T13 | runs/2026/2026-11-25/050451-e2e-1b14369/report.md
- 2026-11-25 050139-e2e-1b14369 | e2e | soak seed 1 settle (svc-billing patched with one line (go build overlay)) | fail | pass 7 fail 7 skip 2 | R11 soak T1-T13 | runs/2026/2026-11-25/050139-e2e-1b14369/report.md
- 2026-11-25 021243-unit-f6bb266 | unit | client-core (sync, worktree client-core-sync) | pass | pass 115 fail 0 skip 1 | client-core-sync unit | runs/2026/2026-11-25/021243-unit-f6bb266/report.md
- 2026-11-25 021023-unit-762b939 | unit | svc-sync | pass | pass 6 fail 0 skip 0 | sync-unit-sim | runs/2026/2026-11-25/021023-unit-762b939/report.md
- 2026-11-25 021008-unit-67a42d3 | unit | svc-admin | pass | pass 6 fail 0 skip 0 | admin-unit-contract | runs/2026/2026-11-25/021008-unit-67a42d3/report.md
- 2026-11-25 020954-integration-0084d7f | integration | svc-tenancy | pass | pass 6 fail 0 skip 0 | tenancy-integration | runs/2026/2026-11-25/020954-integration-0084d7f/report.md
- 2026-11-25 020947-unit-0084d7f | unit | svc-tenancy | pass | pass 4 fail 0 skip 0 | tenancy-unit-contract | runs/2026/2026-11-25/020947-unit-0084d7f/report.md
- 2026-11-25 020808-integration-7ae4f6b | integration | svc-identity | pass | pass 2 fail 0 skip 0 | identity-integration | runs/2026/2026-11-25/020808-integration-7ae4f6b/report.md
- 2026-11-25 020706-unit-7ae4f6b | unit | svc-identity | pass | pass 5 fail 0 skip 0 | identity-unit-contract | runs/2026/2026-11-25/020706-unit-7ae4f6b/report.md
- 2026-11-25 015822-integration-762b939 | integration | svc-sync simulator soak 30m | pass | pass 1458 fail 0 skip 0 | sync soak 30m, 1458 seeds | runs/2026/2026-11-25/015822-integration-762b939/report.md
- 2026-11-25 015716-integration-762b939 | integration | svc-sync simulator | pass | pass 50 fail 0 skip 0 | sync simulator 50 seeds (1..50) | runs/2026/2026-11-25/015716-integration-762b939/report.md
- 2026-11-25 015715-integration-762b939 | integration | svc-sync simulator | pass | pass 1 fail 0 skip 0 | sync simulator seed 1 | runs/2026/2026-11-25/015715-integration-762b939/report.md
- 2026-11-25 015707-e2e-f6bb266 | e2e | client-core-sync <-> svc-sync devstack | pass | pass 1 fail 0 skip 0 | sync client e2e | runs/2026/2026-11-25/015707-e2e-f6bb266/report.md
- 2026-11-25 015449-load-be964ce | load | svc-identity+svc-tenancy+svc-admin (local) | partial | pass 8 fail 0 skip 0 | NEW local baseline | runs/2026/2026-11-25/015449-load-be964ce/report.md
- 2026-11-25 015449-e2e-be964ce | e2e | svc-identity+svc-tenancy+svc-admin (cross-service) | pass | pass 31 fail 0 skip 0 | scenarios 1-3 | runs/2026/2026-11-25/015449-e2e-be964ce/report.md
