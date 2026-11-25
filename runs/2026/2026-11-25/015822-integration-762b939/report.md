# svc-sync simulator, 30-minute soak

Run `015822-integration-762b939`, 2026-11-25. Local developer machine (WSL2 Linux, Go 1.27.1, Node 22, Docker 29.1). Services are local processes; Postgres 17 is the infra dev compose stack under its own project name and port; nothing outside this machine is contacted.
Command: `simulate -soak 30m` (svc-sync `762b939`). It ran for 1800 s wall clock; the machine also ran the service suites in parallel for part of that time, so the seed count is not a throughput figure.
Result: **1458 of 1458 seeds converged, 0 divergences** (30m0s).

```

1458 runs in 30m0s, 0 with divergences []
ops=323858 bills=19825 held=29274 voided=5117 flagged=18933 injected=27099 | crashes=29171 hub-restarts=9573 hub-wipes=5033 cloud-rollbacks=1458 key-losses=1458 partitions=43496
device syncs: via hub=111985, hub failed=84767, fell back to cloud=59158
network: calls=924555 lost=14844 lost-acks=14090 duplicates=21604 stale-replays=14207
```
Artifact: `sim-soak.log` (compressed if large).
