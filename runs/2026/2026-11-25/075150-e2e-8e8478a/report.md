# ADR-0014 R11 trading-day soak: soak seed 2 settle with a cloud restore

Run `075150-e2e-8e8478a`, 2026-11-25. Local developer machine (WSL2 Linux), full Phase 2 stack as local processes, Postgres 17 and NATS 2.15.3 in Docker under their own compose project and ports; nothing outside this machine is contacted.
Variant: **real identity directory, harness additions listed in the report**.
Command: `soak/scripts/run-all.sh <work> f2:2:settle:0 (fresh stack: stack.sh down, up with SOAK_SYNC_PATCH=1; run-one.sh: soak-onboard, soakgw, vitest src/soak.soak.ts)`. Wall time 86 s (the simulated day 17 s, reconnect convergence 3.5 s).
Result: **11 claims passed, 4 failed, 1 not run** (status `fail`). Seed 2, `number_at=settle`, business date 2026-11-25.

## Repositories (SHAs)

| repo | branch | SHA |
|---|---|---|
| infra | feat/phase2-local-dev | `5f784f4` (up-full.sh sets the real-directory environment of sync, adds stop|start|restart|rebuild) |
| svc-identity | feat/phase2-* | `2ba5adc` |
| svc-tenancy | feat/phase2-* | `3792f07` |
| svc-admin | feat/phase2-* | `24444d3` |
| svc-catalog | feat/phase2-* | `38e7bc1` |
| svc-orders | feat/phase2-* | `809fffb` |
| svc-billing | feat/phase2-* | `26676cf` (as pinned, no overlay) |
| svc-sync | feat/phase2-sync | `f013d8b` (cmd/gateway with SYNC_DIRECTORY=identity, built with soak/patches/svc-sync-credential-brand.patch through go build -overlay (checkout untouched); the hub runs in soakgw) |
| client-core | feat/phase2-integration | `413b5b4` |
| app-restaurant | feat/phase2-app | `09bf2db` |
| go-common | feat/phase2-libs | `a170f64` |
| go-auth | feat/phase2-auth | `94f1970` |
| api-specs-p2 | feat/phase2-specs | `6e63014` |

## Claims

| claim | result | first failing check |
|---|---|---|
| T1 Convergence | FAIL | stateHash (no op held) differ: [["A07","dcd41b11"],["A08","f8b957bb"],["A09","f8b957bb"],["B3K","f8b957bb"],["B4K","f8b957bb"],["B5K","f8b957bb"],["K9K","f8b957bb"],["K8K","f8b957bb"]] |
| T2 Money | PASS |  |
| T3 Numbers | FAIL | billing reports gaps: [{"detected_at":"2026-11-25T07:52:30.992Z","financial_year":"2627","kind":"manual_book","missing_from":1,"missing_to":6,"series_code":"MB1","status":"open","tax_registration_id":"01a211b0-a938-71fc- |
| T4 Tax | FAIL | bills not verified: {"verified":49,"manual_entry":1} |
| T5 Kitchen | PASS |  |
| T6 Printing | PASS |  |
| T7 Nothing silent | PASS |  |
| T8 Day close | PASS |  |
| T9 Projections | PASS |  |
| T10 Trading is never gated | NOT_RUN | not exercisable on this build: app-restaurant has no entitlement snapshot or subscription check (grep: none); the nearest condition, an expired offline pack, only refuses new PIN sign-ins in StubIdentity and was not run |
| T11 Business day | FAIL | original Z 1267400 + revisions 0 != all bills 1300300 (late bills on A09 total 32900) |
| T12 Numbering trigger | PASS |  |
| T13 Blackout | PASS |  |
| X1 No duplicate ops after lost acks | PASS |  |
| X3 Cloud restore epoch | PASS |  |
| X2 Erasure of a customer's details | PASS |  |

## Evidence and failures per claim

### T1 Convergence: FAIL
- pass: A07 differs from A08 in: bill (51 rows vs 51 on A08; fields customer_name,customer_gstin), check (49 rows vs 49 on A08; fields customer_name,customer_gstin)
- pass: rows compared with the customer_* values left out: all devices equal
- pass: plain stateHash groups: 2; accepted groups: 1; held per device [["A07",0],["A08",0],["A09",0],["B3K",0],["B4K",0],["B5K",0],["K9K",0],["K8K",0]]
- pass: the cloud's outlet vector equals the devices' (origins compared)
- pass: converged in 4 rounds of 5 s virtual time (3520 ms wall) after the internet returned: within 60 s
- pass: scope: the nodes compared are the 8 headless devices (device state hash), the hub and cloud by vector; the real entity registry (client-core restaurantRegistry) is in use
- **fail:** stateHash (no op held) differ: [["A07","dcd41b11"],["A08","f8b957bb"],["A09","f8b957bb"],["B3K","f8b957bb"],["B4K","f8b957bb"],["B5K","f8b957bb"],["K9K","f8b957bb"],["K8K","f8b957bb"]]
- **fail:** version vectors differ between devices

### T2 Money: PASS
- pass: device Z-report printed offline: Z/2026-11-25/AAA total 1267400 paise over 49 bills
- pass: billing close total (1267400) equals the device Z total
- pass: billing close bill_count 49 equals the device Z bill count
- pass: billing holds 51 bills = device 51
- pass: bill totals equal to the paisa: 1300300
- pass: payment totals per tender equal: {"cash":681950,"upi":404200,"card":199950}
- pass: refunds equal (1)
- pass: credit notes equal (3)
- pass: day summary cgst = 30233
- pass: day summary sgst = 30233
- pass: day summary discounts = 22650
- pass: day summary round_off = 984
- pass: day summary bill_count = 51
- pass: day summary total = 1300300

### T3 Numbers: FAIL
- pass: A07: no gap or duplicate in its series register
- pass: A08: no gap or duplicate in its series register
- pass: A09: no gap or duplicate in its series register
- pass: B3K: no gap or duplicate in its series register
- pass: B4K: no gap or duplicate in its series register
- pass: B5K: no gap or duplicate in its series register
- pass: K9K: no gap or duplicate in its series register
- pass: K8K: no gap or duplicate in its series register
- pass: 54 document numbers, all unique across the outlet
- pass: series AAA/2627: 1..31 gapless
- pass: series AAC/2627: 1..17 gapless
- pass: series AAE/2627: 1..2 gapless
- pass: series AAB/2627: 1..3 gapless
- pass: billing register AAA/2627: count 31, last 31, gaps 0
- pass: billing register AAC/2627: count 17, last 17, gaps 0
- pass: billing register AAE/2627: count 2, last 2, gaps 0
- pass: billing register AAB/2627: count 3, last 3, gaps 0
- pass: no gap alert at the gateway
- pass: device day_close rows on A07: 2 (z numbers Z/2026-11-25/AAA, Z/2026-11-25/AAC); the contract has no series section on day_close, so the section-equals-register check is the register comparison above
- **fail:** billing reports gaps: [{"detected_at":"2026-11-25T07:52:30.992Z","financial_year":"2627","kind":"manual_book","missing_from":1,"missing_to":6,"series_code":"MB1","status":"open","tax_registration_id":"01a211b0-a938-71fc-85

### T4 Tax: FAIL
- pass: 3 credit notes verified
- pass: seeded wrong-tax bill AAE/2627/000001 is flagged: verification {"calc_v":1,"checked_at":"2026-11-25T07:52:32.002Z","difference":{"amount_paise":300,"currency":"INR"},"ruleset_id":"rs-1","status":"mismatch"}
- pass: and billing kept it unchanged (total equals the device's)
- pass: tax_mismatch exceptions in billing: 1: bill:01a212c3 diff=300
- pass: every tax mismatch is the seeded one
- **fail:** bills not verified: {"verified":49,"manual_entry":1}

### T5 Kitchen: PASS
- pass: KITCHEN screen held every one of its 62 KOTs' tickets with the internet cut (none missing)
- pass: KITCHEN: every KOT line exactly once
- pass: BAR screen held every one of its 62 KOTs' tickets with the internet cut (none missing)
- pass: BAR: every KOT line exactly once
- pass: order to KDS on the LAN: 12 probes, p50 35 ms, p95 41 ms (≤ 1000 ms; each probe = push to hub + pull, driven back to back)
- pass: K9K holds all 46 KITCHEN KOTs after 3 hub stops (one with a wiped disk)
- pass: K8K holds all 18 BAR KOTs after 3 hub stops (one with a wiped disk)

### T6 Printing: PASS
- pass: each of 50 bills has exactly one original print job
- pass: 7 reprint(s) carry the duplicate banner
- pass: 64 KOTs: each printed once as an original on its routed printer (stations that only show on the KDS print none; the handed-over and the stale blackout ticket print none)
- pass: scope: the fake printers capture the ESC/POS bytes of the bill printer and the kitchen printer per device; counts are of decoded text matches on the document number

### T7 Nothing silent: PASS
- pass: every node shows the same 0 detector finding(s): 
- pass: seeded double payment: made; overpayment finding still open after the refund: 0; net paid after refund 15800
- pass: the double payment was seeded
- pass: line-on-two-bills race appears in billing exceptions: line_on_two_bills, two_bills_one_check
- pass: the double payment appears in billing exceptions: payments_exceed_total
- pass: the race is closed by a credit note op
- pass: the double payment is closed by a refund op
- pass: attention made of: [{"dev":"A07","held":0,"refused":0,"conflicts":[],"clock_held":[]},{"dev":"A08","held":0,"refused":0,"conflicts":[],"clock_held":[]}] (same on every device: true)
- pass: ops shown as needing attention equal the quarantined or voided: none quarantined

### T8 Day close: PASS
- pass: day close was refused while an order line was unresolved (day_blocked)
- pass: day close succeeded offline after resolution: Z/2026-11-25/AAA
- pass: orders exceptions around close: [["unresolved_line_at_close/resolved",2]]

### T9 Projections: PASS
- pass: orders: service state_hash equals the device's (5eeeb8f8fb1cc1b8…)
- pass: billing: service state_hash equals the device's (949f1fd0f45f5ed6…)
- pass: before the cloud rollback, orders: equal to the device digest; after it the service state_hash is 5eeeb8f8fb1c…
- pass: before the cloud rollback, billing: equal to the device digest; after it the service state_hash is 949f1fd0f45f…
- pass: the bus reset was not run in any seed: no bus-reset tool exists in go-common or infra (the outbox documentation names hostry-bus-reset)

### T10 Trading is never gated: NOT_RUN
- **fail:** not exercisable on this build: app-restaurant has no entitlement snapshot or subscription check (grep: none); the nearest condition, an expired offline pack, only refuses new PIN sign-ins in StubIdentity and was not run

### T11 Business day: FAIL
- pass: A07: one business_day for 2026-11-25
- pass: A08: one business_day for 2026-11-25
- pass: A09: one business_day for 2026-11-25
- pass: B3K: one business_day for 2026-11-25
- pass: B4K: one business_day for 2026-11-25
- pass: B5K: one business_day for 2026-11-25
- pass: K9K: one business_day for 2026-11-25
- pass: K8K: one business_day for 2026-11-25
- pass: the original Z totals on the device are unchanged after the late bills arrived
- pass: revision numbers equal on every node: ["f5787f57-01b0-524f-9bf3-cd6d9c3b5824#1"]
- pass: billing holds 1 Z revision(s), equal to the devices'
- pass: A08 closed the same date while partitioned from A07: Z numbers Z/2026-11-25/AAA and Z/2026-11-25/AAC; billing day status closed, late_document_count 1, revision_count 1
- **fail:** original Z 1267400 + revisions 0 != all bills 1300300 (late bills on A09 total 32900)

### T12 Numbering trigger: PASS
- pass: 2 check(s) were presented (pre-bill) during the day
- pass: numbers consumed (53) equal documents that exist (53): presenting consumed none
- pass: every presented check ended billed, cancelled with approval or carried over

### T13 Blackout: PASS
- pass: the KDS station lists the ticket handed over by hand and the stale one as late tickets: ["handover","stale"]
- pass: the recent blackout ticket printed once on the station
- pass: no handed-over and no stale KOT printed as an original after power returned
- pass: bills finalised on the battery counter in the blackout keep their series (AAC: gapless per T3) and their original print exists (T6)
- pass: nodes converged 2 rounds (5 s virtual each, 771 ms wall) after the LAN returned: within 60 s
- pass: the paper page typed twice (2 entries) is 1 bill: MB1/2627/000007
- pass: the handover is recorded and listed on the KDS side

### X1 No duplicate ops after lost acks: PASS
- pass: A07: 203 own ops, gateway stores 203, contiguous from 1
- pass: A08: 125 own ops, gateway stores 125, contiguous from 1
- pass: A09: 13 own ops, gateway stores 13, contiguous from 1
- pass: B3K: 33 own ops, gateway stores 33, contiguous from 1
- pass: B4K: 27 own ops, gateway stores 27, contiguous from 1
- pass: B5K: 17 own ops, gateway stores 17, contiguous from 1
- pass: K9K: 20 own ops, gateway stores 20, contiguous from 1
- pass: K8K: 7 own ops, gateway stores 7, contiguous from 1
- pass: 1 push answer(s) were lost after the server applied them; 3 hub stop(s) incl. 1 disk wipe; 1 device restart(s)
- pass: 454 outbox events, all distinct ids

### X3 Cloud restore epoch: PASS
- pass: cloud ops: 442 at backup, 454 before the restore, 442 right after it, 454 after the nodes re-offered
- pass: the restore really rolled the cloud back (it lost ops)
- pass: the cloud holds every op again after 3 rounds (1663 ms wall)
- pass: the hub's round reported the epoch change
- pass: no device's state changed through the rollback and the re-offer (state hash equal before and after, per device)

### X2 Erasure of a customer's details: PASS
- pass: erasable side tables (sync_pii) holding the values before: billing 1, orders 1
- pass: before erasure: details present in billing false, orders false, device A07 true, gateway op_pii rows 3
- pass: customer details were present somewhere before the erasure (the test is meaningful)
- pass: gateway erasure issued for 1 op(s) (POST /internal/v1/pii/erase as billing)
- pass: billing no longer returns the name or GSTIN
- pass: orders no longer returns the name or GSTIN
- pass: no device holds the details any more
- pass: billing and orders side tables no longer hold the values (1/1 before, 0/0 after)
- pass: device A07 holds 0 pii_erase record(s); ["cloud-genesis","cloud-soak-restore-2"] cloud origins in its vector
- pass: the gateway's op_pii rows of those ops are gone
- pass: the copy on the internal queue ages out within 7 days (ADR-0014 A2.1 item 7): not waited for, stated in the erasure promise

## Run facts
- Script steps: 106 run, 0 failed: []
- Hub stops 3 (disk wipes 1), device restarts ['B4K'], lost push answers 1, approvals 2, bills by device {"A07": 28, "A08": 16, "A09": 2}
- Offline Z: {"zNumber": "Z/2026-11-25/AAA", "totalPaise": 1267400, "billCount": 49}; second close on the partitioned counter: {"zNumber": "Z/2026-11-25/AAC", "totalPaise": 1267400, "billCount": 49}
- Gateway ops by outcome: [{"cloud_origin": false, "status": "accepted", "reason": "", "n": 450}, {"cloud_origin": true, "status": "accepted", "reason": "", "n": 7}]; parked events billing/orders: {"billing": "0", "orders": "0"}
- Billing exceptions: ["late_document/info/open/bill", "tax_mismatch/high/open/bill", "late_document/info/open/business_day", "line_on_two_bills/high/open/bill", "two_bills_one_check/warning/open/check", "payments_exceed_total/high/resolved/bill", "manual_entry/warning/open/bill", "manual_book_missing_number/high/open/series"]; orders exceptions: {"unresolved_line_at_close/resolved": 2, "late_ticket/open": 1}
- Cloud restore: {"ops_at_backup": 442, "ops_before_restore": 454, "hashes_before": {"A07": "dcd41b1101fd4962f06983b1593e23e3abc34c4e5a9a9e848f09e8f303e3e6bb", "A08": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b", "A09": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b", "B3K": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b", "B4K": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b", "B5K": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b", "K9K": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b", "K8K": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b"}, "ops_after_restore": 442, "ops_after_reoffer": 454, "hashes_after": {"A07": "dcd41b1101fd4962f06983b1593e23e3abc34c4e5a9a9e848f09e8f303e3e6bb", "A08": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b", "A09": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b", "B3K": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b", "B4K": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b", "B5K": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b", "K9K": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b", "K8K": "f8b957bb661bbae280d1dab0ed977516f13a4d91dcb536f6e4fa1858bf97801b"}, "settle_rounds": 3, "reoffer_wall_ms": 1663, "hub_epoch_changed": true}

## Timeline
- 07:52:00Z cast up: 3 counters, 3 captains, 2 KDS, one hub; morning: internet up for the configuration push
- 07:52:20Z internet cut for the whole trading day
- 07:52:20Z configuration published: 7/7 device intents, 3/3 cloud emits
- 08:13:20Z first bill seen
- 10:21:20Z BLACKOUT: LAN, hub and the A07 counter are off
- 10:52:20Z POWER RETURNS
- 12:52:40Z A09 isolated; A07 and A08 partitioned from each other for the close
- 12:52:40Z wrong-tax bill AAE/2627/000001 recorded; its print failed under the harness's doctored view (not_found: bill 01a212c3-ef73-7006-9c39-45f1ae8468d7); printing it now

Findings drawn from this and the other runs: `docs/execution/phase-2/exit-checks.md (section 2026-11-25 rerun after the fix wave)`.

## Artifacts

- `artifacts/vitest.out`
- `artifacts/gw.log`
- `artifacts/run.result.json`
- `artifacts/art-f2-onboard.json`
- `artifacts/art-f2-sync.log`
- `artifacts/art-f2-identity.log`
- `artifacts/art-f2-billing.log`
- `artifacts/art-f2-orders.log`
- `artifacts/art-f2-tenancy.log`
- `artifacts/art-f2-catalog.log`
