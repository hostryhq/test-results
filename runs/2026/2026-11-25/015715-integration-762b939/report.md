# svc-sync simulator, seed 1

Run `015715-integration-762b939`, 2026-11-25. Local developer machine (WSL2 Linux, Go 1.27.1, Node 22, Docker 29.1). Services are local processes; Postgres 17 is the infra dev compose stack under its own project name and port; nothing outside this machine is contacted.
Command: `simulate -seed 1 -v` (svc-sync `762b939`, SQLite gateway store). Duration 1 s.
Result: **1 of 1 runs converged, 0 divergences.**

```

1 runs in 1s, 0 with divergences []
ops=208 bills=8 held=21 voided=3 flagged=14 injected=20 | crashes=20 hub-restarts=12 hub-wipes=4 cloud-rollbacks=1 key-losses=1 partitions=23
device syncs: via hub=66, hub failed=80, fell back to cloud=50
network: calls=659 lost=3 lost-acks=11 duplicates=10 stale-replays=11
```
Artifact: `sim-seed1.log`.
