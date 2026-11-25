# LOCAL load baseline (new baseline, not a pass)

Run `015449-load-be964ce`, 2026-11-25. **Local dev machine, not production shape.** local dev machine (linux/amd64, 8 logical CPUs), services as local processes, Postgres 17 in Docker, load generator on the same host. NOT production shape.
Target: svc-identity, svc-tenancy, svc-admin as local processes against one Postgres 17 container; generator is the Go test `e2e/zz_load_test.go` on the same host.
Stated targets: none exist yet (no SLO is defined for Phase 1); this run only fixes the baseline. Baseline compared: **none stored, so this run is recorded as the baseline** (`baselines/load.json`). Status is `partial` for that reason.

Each scenario runs 15 s at fixed concurrency against warm services (after the scenarios have created the data).

| Scenario | Conc. | Requests | Errors | Req/s | p50 ms | p95 ms | p99 ms | Note |
|---|---|---|---|---|---|---|---|---|
| tenancy GET /v1/outlets/{id} | 16 | 37432 | 0 | 2495 | 5.46 | 13.12 | 18.19 |  |
| tenancy GET /v1/document-series | 16 | 29073 | 0 | 1937 | 6.62 | 18.97 | 31.32 |  |
| identity GET /v1/members/{id} | 16 | 10639 | 0 | 708 | 19.81 | 48.22 | 67.89 |  |
| identity GET offline pack | 8 | 2801 | 0 | 186 | 39.70 | 79.14 | 98.63 | signs and builds a pack per request |
| admin GET /v1/sessions/current | 16 | 132390 | 0 | 8825 | 0.70 | 6.92 | 11.86 |  |
| admin GET /v1/organisations | 16 | 7823 | 0 | 520 | 26.15 | 61.92 | 81.15 | svc-admin to tenancy and identity |
| identity POST /v1/tokens/refresh | 1 | 200 | 0 | 84 | 10.87 | 19.35 | 25.57 | one rotating chain, sequential |
| identity POST /v1/otp-challenges | 1 | 12 | 0 | 85 | 10.79 | 15.39 | 15.39 | sample of 12: per-device/per-IP daily limits make a rate meaningless |

Total request errors: 0.

## Not measured, and why
- Sync push/pull: no standalone signed-op client exists. The only drivers are the in-process simulator and the single-device TypeScript e2e; the svc-sync gateway also still uses `identitystub`. Simulator throughput is in the simulator run entries, labelled as such. Not run: needs a Go or TS load client for `/sync/v1/push` and `/pull`.
- OTP challenge as a rate: limited by design (per-device 20/day, per-IP 40/day, per-number 3 per 15 min); a 12-request sample of the accepted path is shown.
- Session refresh: refresh tokens rotate, so this is one sequential chain (concurrency 1), not a throughput test.

## Artifacts
`artifacts/load.json`.
