# svc-sync simulator, 50 seeds (1..50)

Run `015716-integration-762b939`, 2026-11-25. Local developer machine (WSL2 Linux, Go 1.27.1, Node 22, Docker 29.1). Services are local processes; Postgres 17 is the infra dev compose stack under its own project name and port; nothing outside this machine is contacted.
Command: `simulate -from 1 -runs 50` (svc-sync `762b939`, SQLite gateway store). Duration 66 s.
Result: **50 of 50 runs converged, 0 divergences.**

```

50 runs in 1m6s, 0 with divergences []
ops=11108 bills=681 held=982 voided=181 flagged=696 injected=965 | crashes=967 hub-restarts=346 hub-wipes=166 cloud-rollbacks=50 key-losses=50 partitions=1423
device syncs: via hub=3995, hub failed=2748, fell back to cloud=1950
network: calls=31794 lost=536 lost-acks=490 duplicates=782 stale-replays=560
```
Artifact: `sim-50.log`.
