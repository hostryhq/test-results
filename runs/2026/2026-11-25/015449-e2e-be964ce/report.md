# E2E scenarios 1-3: real services on the local dev stack

Run `015449-e2e-be964ce`, 2026-11-25. Environment: Local developer machine (WSL2 Linux, Go 1.27.1, Node 22, Docker 29.1). Services are local processes; Postgres 17 is the infra dev compose stack under its own project name and port; nothing outside this machine is contacted.
Command: `scripts/run-all.sh` step `scenarios` = `go test -tags e2e -count=1 -v ./e2e` in e2e-tests. Duration 135 s (includes the load test, recorded separately).

## What ran
The suite builds `svc-identity`, `svc-tenancy` and `svc-admin` binaries from the commits below, starts them against Postgres 17 from `infra/dev/compose.yaml`
(project `e2e`, port 55500; identity 18101, tenancy 18102, admin 18103), runs the scenarios, and removes the compose project.

| Scenario | Result |
|---|---|
| --- PASS: TestScenario1_VendorBootstrapToOfflinePack (4.25s) | pass |
| --- PASS: TestScenario1b_GSTINVerifyDeviceAndOfflinePack (0.10s) | pass |
| --- PASS: TestScenario2_CrossTenantIsolation (0.17s) | pass |
| --- PASS: TestScenario3_TwoPersonApproval (28.72s) | pass |

Result: **31 passed, 0 failed, 0 skipped** (tests and subtests). No failures.

1. Vendor bootstrap by `svc-identity vendor bootstrap` (two platform_admin accounts), setup, sign-in through svc-admin (cookie + CSRF + Origin; a mutation without CSRF is refused 403), organisation + outlet + owner invite via `createOrganisation` (tenancy then identity, one Idempotency-Key; replay returns the same organisation), owner OTP login (invitation accepted), GSTIN verified by a vendor user (tenant sees `verified`), device registered with `billing_capability: active`, offline pack carries the series that tenancy's registry holds.
2. Tenant B token gets 404 on every A resource it can name (tenancy: brand, outlet read/update, tax registration read/update/close; identity: device read/revoke/pack, member, role), lists never contain A's ids, controls on B's own resources succeed, tenant tokens are refused on vendor routes, A is unchanged afterwards. `deleteRole` is idempotent by contract (204 for a role not visible); the check is that A's role survives.
3. Two young accounts cannot form a pair (filing refused 409 `second_approver_unavailable`); after seeding admin2's `cred_at` 8 days back, a request is filed, the requester cannot self-approve, the older approver approves with a fresh TOTP, the vendor audit log shows the approval.

## Workarounds and gaps found (open issues, not failures)
- svc-identity's test SMS sender (`cmd/svc-identity/config.go` `testSMS`) sends and records nothing, so no black-box test can read the OTP. The test recovers it from the challenge's HMAC with this run's throw-away MAC key (10^6 candidates). Suggested fix for the owner: a dev-only recording sender, or an echo endpoint behind the dev flag.
- The real binary has no injectable clock; the 7-day approver age is seeded in the identity database (`global_records` `vendor_user.cred_at`).
- Vendor users created through the API cannot finish setup: the mail interface discards the link and the CLI refuses a second bootstrap once an admin is active. The test creates both admins with two bootstraps before either activates.
- The OTP challenge endpoint's limits (per-device 20/day, per-IP 40/day) are not configurable, so no meaningful OTP load rate exists.
- Not run, with reason: sync gateway behind the real identity service (svc-sync still uses `identitystub`; emit API not built); NATS relay of the outbox (bus not started in this stack); approval grant verification in tenancy (go-auth has no verifier yet, so `org_suspend` was approved but not executed).

## Commits under test
- svc-identity: `7ae4f6b01153c93734a9cf81a4358f81d06cfba5`
- svc-tenancy: `0084d7fbdcf0d8ce469852fb49b71cc2ae40bea6`
- svc-admin: `67a42d35cfc07972318aa4af48cf07b3d75ae10f`
- e2e-tests: `be964ce443e168e164214b39289c717d0241b07e`
- infra: `c0a26f6d2977db1c259d3eefbbf57e74ed0be6cd`
- go-common: `95088b54b6edd966e610bd5826d3f3009a2b4c77`
- go-auth: `b19b5407e3f206ad1f79a7a764a1650c65d8fcc7`
- api-specs: `c92ca77ecb4e8334b440256cecdbbb843049f6a0`

svc-tenancy was built from `git archive` of its commit (its working tree carried an unrelated uncommitted edit that breaks the build). svc-identity and svc-admin were built in place from clean trees with their sibling libraries via `go.work`; go-common and go-auth were at the SHAs shown when the binaries were built (their later commits were written after the build).
e2e-tests: the tree was clean at `be964ce` when the run started; later commits only add README and a service-suite script.

## Artifacts
`artifacts/scenarios.log` (go test -v), `build-info.txt`, `steps.tsv`, service request logs (`identity.log.gz`, `tenancy.log`, `admin.log.gz`; no secrets, method and path only).
