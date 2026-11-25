# ADR-0014 R11 trading-day soak: seed 1 settle

Run `051502-e2e-1b14369`, 2026-11-25. Local developer machine (WSL2 Linux), full Phase 2 stack as local processes, Postgres 17 and NATS 2.15.3 in Docker under their own compose project and ports; nothing outside this machine is contacted.
Variant: **svc-billing AS-IS (as-is billing, unpatched)**.
Command: `soak/scripts/run-one.sh <work> asis 1 settle 0`. Wall time 161 s (the simulated day 16 s, reconnect convergence 2.6 s).
Result: **5 claims passed, 9 failed, 2 not run** (status `fail`). Seed 1, `number_at=settle`, business date 2026-11-24.

## Repositories (SHAs)

| repo | branch | SHA |
|---|---|---|
| infra | feat/phase2-local-dev | `34eeec7` |
| svc-identity | feat/phase2-* | `e28e23c` |
| svc-tenancy | feat/phase2-* | `a5b3183` |
| svc-admin | feat/phase2-* | `24444d3` |
| svc-catalog | feat/phase2-* | `38e7bc1` |
| svc-orders | feat/phase2-* | `50f041f` |
| svc-billing | feat/phase2-* | `6f5cb60` (as pinned, no patch) |
| svc-sync | feat/phase2-* | `e7c883e` (HOSTRY_SYNC_DIRECTORY=stub; its packages run inside soakgw, cmd/gateway stopped) |
| client-core | feat/phase2-integration | `2db7e1e` |
| app-restaurant | feat/phase2-app | `c370225` |
| go-common | feat/phase2-libs | `a170f64` |
| go-auth | feat/phase2-auth | `94f1970` |
| api-specs-p2 | feat/phase2-specs | `690d4d3` (services pin b153c62; head moved during the run; NATS accounts and digest entity metadata were read from this checkout) |

## Claims

| claim | result | first failing check |
|---|---|---|
| T1 Convergence | FAIL | state hashes differ: [["A07","0009b1b4"],["A08","29c469ef"],["A09","77ea8f71"],["B3K","77ea8f71"],["B4K","77ea8f71"],["B5K","77ea8f71"],["K9K","ebc25350"],["K8K","380ee3ea"]] |
| T2 Money | FAIL | billing has no closed business day with totals for the date |
| T3 Numbers | FAIL | billing register AAA/2627: undefined vs devices count 31 last 31 |
| T4 Tax | FAIL | the seeded wrong-tax bill is missing from billing: it was rejected |
| T5 Kitchen | PASS |  |
| T6 Printing | PASS |  |
| T7 Nothing silent | FAIL | no line_on_two_bills exception in billing (kinds: ) |
| T8 Day close | PASS |  |
| T9 Projections | FAIL | orders: state_hash differs; 1 entity differ: order_line_modifier: device has 7 rows, service lists none |
| T10 Trading is never gated | NOT_RUN | not exercisable on this build: app-restaurant has no entitlement snapshot or subscription check (grep: none); the nearest condition, an expired offline pack, only refuses new PIN sign-ins in StubIdentity and was not run |
| T11 Business day | FAIL | billing revisions 0 vs device 0: the late bill after close (AAE/2627/000002) produced no Z revision at billing within the wait |
| T12 Numbering trigger | FAIL | numbers consumed 0 != documents 53 |
| T13 Blackout | PASS |  |
| X1 No duplicate ops after lost acks | PASS |  |
| X3 Cloud restore epoch | NOT_RUN | no restore epoch in this seed |
| X2 Erasure of a customer's details | FAIL | devices still holding the details: A07, A08, A09, B3K, B4K, B5K, K9K, K8K (fields: check.customer_gstin, bill.customer_gstin) |

## Evidence and failures per claim

### T1 Convergence: FAIL
- pass: the cloud's outlet vector equals the devices' (origins compared)
- pass: converged in 4 rounds of 5 s virtual time (2586 ms wall) after the internet returned: within 60 s
- pass: scope: the nodes compared are the 8 headless devices (device state hash), the hub and cloud by vector; the real entity registry (client-core restaurantRegistry) is in use
- **fail:** state hashes differ: [["A07","0009b1b4"],["A08","29c469ef"],["A09","77ea8f71"],["B3K","77ea8f71"],["B4K","77ea8f71"],["B5K","77ea8f71"],["K9K","ebc25350"],["K8K","380ee3ea"]]
- **fail:** version vectors differ between devices

### T2 Money: FAIL
- pass: device Z-report printed offline: Z/2026-11-24/A07 total 1129100 paise over 49 bills
- **fail:** billing has no closed business day with totals for the date
- **fail:** billing holds 0 bills, device 51
- **fail:** bill total device 1162000 vs billing 0
- **fail:** tender totals device {"cash":736650,"card":215650,"upi":203200} vs billing {}
- **fail:** refunds device 1 vs billing 0
- **fail:** credit notes device 3/41200 vs billing 0/0
- **fail:** day summary cgst: device 27150 vs billing undefined
- **fail:** day summary sgst: device 27150 vs billing undefined
- **fail:** day summary discounts: device 8600 vs billing undefined
- **fail:** day summary round_off: device 800 vs billing undefined
- **fail:** day summary bill_count: device 51 vs billing undefined
- **fail:** day summary total: device 1162000 vs billing undefined

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
- pass: billing reports no gap in the document register
- pass: no gap alert at the gateway
- pass: device day_close rows on A07: 2 (z numbers Z/2026-11-24/A07, Z/2026-11-24/A08); the contract has no series section on day_close, so the section-equals-register check is the register comparison above
- **fail:** billing register AAA/2627: undefined vs devices count 31 last 31
- **fail:** billing register AAC/2627: undefined vs devices count 17 last 17
- **fail:** billing register AAE/2627: undefined vs devices count 2 last 2
- **fail:** billing register AAB/2627: undefined vs devices count 3 last 3

### T4 Tax: FAIL
- pass: 0 bills verified by billing (all but the seeded one)
- pass: 0 credit notes verified
- pass: tax_mismatch exceptions in billing: 0: 
- pass: every tax mismatch is the seeded one
- **fail:** the seeded wrong-tax bill is missing from billing: it was rejected

### T5 Kitchen: PASS
- pass: KITCHEN screen held every one of its 60 KOTs' tickets with the internet cut (none missing)
- pass: KITCHEN: every KOT line exactly once
- pass: BAR screen held every one of its 60 KOTs' tickets with the internet cut (none missing)
- pass: BAR: every KOT line exactly once
- pass: order to KDS on the LAN: 12 probes, p50 32 ms, p95 34 ms (≤ 1000 ms; each probe = push to hub + pull, driven back to back)
- pass: K9K holds all 47 KITCHEN KOTs after 3 hub stops (one with a wiped disk)
- pass: K8K holds all 15 BAR KOTs after 3 hub stops (one with a wiped disk)

### T6 Printing: PASS
- pass: each of 50 bills has exactly one original print job
- pass: 1 reprint(s) carry the duplicate banner
- pass: 62 KOTs: each printed once as an original on its routed printer (stations that only show on the KDS print none; the handed-over and the stale blackout ticket print none)
- pass: scope: the fake printers capture the ESC/POS bytes of the bill printer and the kitchen printer per device; counts are of decoded text matches on the document number

### T7 Nothing silent: FAIL
- pass: every node shows the same 1 detector finding(s): overpayment:01a20cd7-82a0-70
- pass: seeded double payment: made; overpayment finding still open after the refund: 1; net paid after refund 15800
- pass: the double payment was seeded
- pass: the race is closed by a credit note op
- pass: the double payment is closed by a refund op
- pass: ops shown as needing attention equal the quarantined or voided: A07 1=1, A08 1=1, K9K 19=19, K8K 3=3
- **fail:** no line_on_two_bills exception in billing (kinds: )
- **fail:** the seeded double payment does not appear in billing exceptions (no payment-related exception kind)

### T8 Day close: PASS
- pass: day close was refused while an order line was unresolved (day_blocked)
- pass: day close succeeded offline after resolution: Z/2026-11-24/A07
- pass: orders exceptions around close: [["unresolved_line_at_close/resolved",2]]

### T9 Projections: FAIL
- pass: the cloud rollback was not part of this run (seed without a restore epoch)
- pass: the bus reset was not run in any seed: no bus-reset tool exists in go-common or infra (the outbox documentation names hostry-bus-reset)
- **fail:** orders: state_hash differs; 1 entity differ: order_line_modifier: device has 7 rows, service lists none
- **fail:** billing: state_hash differs; 12 entities differ: bill: device has 51 rows, service lists none | bill_line: device has 76 rows, service lists none | bill_reprint: device has 1 rows, service lists none | bill_tax_line: device has 54 rows, service lists none | business_day: 1 rows, content hash differs | credit_note: device has 3 rows, service lists none | credit_note_line: device has 3 rows, service lists none | credit_note_tax_line: device has 3 rows, service lists none | day_open: rows device 2 vs service 1 | payment: device has 63 rows, service lists none | refund: device has 1 rows, service lists none | shift: 1 rows, content hash differs

### T10 Trading is never gated: NOT_RUN
- **fail:** not exercisable on this build: app-restaurant has no entitlement snapshot or subscription check (grep: none); the nearest condition, an expired offline pack, only refuses new PIN sign-ins in StubIdentity and was not run

### T11 Business day: FAIL
- pass: A07: one business_day for 2026-11-24
- pass: A08: one business_day for 2026-11-24
- pass: A09: one business_day for 2026-11-24
- pass: B3K: one business_day for 2026-11-24
- pass: B4K: one business_day for 2026-11-24
- pass: B5K: one business_day for 2026-11-24
- pass: K9K: one business_day for 2026-11-24
- pass: K8K: one business_day for 2026-11-24
- pass: the original Z totals on the device are unchanged after the late bills arrived
- pass: revision numbers equal on every node: []
- pass: A08 closed the same date while partitioned from A07: Z numbers Z/2026-11-24/A07 and Z/2026-11-24/A08; billing day status open, late_document_count 0, revision_count 0
- **fail:** billing revisions 0 vs device 0: the late bill after close (AAE/2627/000002) produced no Z revision at billing within the wait
- **fail:** original Z 1129100 + revisions 0 != all bills 1162000 (late bills on A09 total 32900)

### T12 Numbering trigger: FAIL
- pass: 2 check(s) were presented (pre-bill) during the day
- pass: every presented check ended billed, cancelled with approval or carried over
- **fail:** numbers consumed 0 != documents 53

### T13 Blackout: PASS
- pass: the KDS station lists the ticket handed over by hand and the stale one as late tickets: ["handover","stale"]
- pass: the recent blackout ticket printed once on the station
- pass: no handed-over and no stale KOT printed as an original after power returned
- pass: bills finalised on the battery counter in the blackout keep their series (AAC: gapless per T3) and their original print exists (T6)
- pass: nodes converged 2 rounds (5 s virtual each, 754 ms wall) after the LAN returned: within 60 s
- pass: the paper page typed twice (2 entries) is 1 bill: MB1/2627/000007
- pass: the handover is recorded and listed on the KDS side

### X1 No duplicate ops after lost acks: PASS
- pass: A07: 179 own ops, gateway stores 179, contiguous from 1
- pass: A08: 97 own ops, gateway stores 97, contiguous from 1
- pass: A09: 12 own ops, gateway stores 12, contiguous from 1
- pass: B3K: 33 own ops, gateway stores 33, contiguous from 1
- pass: B4K: 19 own ops, gateway stores 19, contiguous from 1
- pass: B5K: 12 own ops, gateway stores 12, contiguous from 1
- pass: K9K: 19 own ops, gateway stores 19, contiguous from 1
- pass: K8K: 3 own ops, gateway stores 3, contiguous from 1
- pass: 1 push answer(s) were lost after the server applied them; 3 hub stop(s) incl. 1 disk wipe; 1 device restart(s)
- pass: 382 outbox events, all distinct ids

### X3 Cloud restore epoch: NOT_RUN
- **fail:** no restore epoch in this seed

### X2 Erasure of a customer's details: FAIL
- pass: erasable side tables (sync_pii) holding the values before: billing 0, orders 1
- pass: before erasure: details present in billing false, orders false, device A07 true, gateway op_pii rows 2
- pass: customer details were present somewhere before the erasure (the test is meaningful)
- pass: gateway erasure issued for 2 op(s) (via Gateway.ErasePII: no HTTP route exists, see findings)
- pass: billing no longer returns the name or GSTIN
- pass: orders no longer returns the name or GSTIN
- pass: device A07 holds 0 pii_erase record(s); ["cloud-genesis"] cloud origins in its vector
- pass: the gateway's op_pii rows of those ops are gone
- pass: the copy on the internal queue ages out within 7 days (ADR-0014 A2.1 item 7): not waited for, stated in the erasure promise
- **fail:** devices still holding the details: A07, A08, A09, B3K, B4K, B5K, K9K, K8K (fields: check.customer_gstin, bill.customer_gstin)
- **fail:** side tables still hold the values after the erasure: billing 0, orders 1

## Run facts
- Script steps: 103 run, 0 failed: []
- Hub stops 3 (disk wipes 1), device restarts ['B4K'], lost push answers 1, approvals 2, bills by device {"A07": 28, "A08": 16, "A09": 2}
- Offline Z: {"zNumber": "Z/2026-11-24/A07", "totalPaise": 1129100, "billCount": 49}; second close on the partitioned counter: {"zNumber": "Z/2026-11-24/A08", "totalPaise": 1129100, "billCount": 49}
- Gateway ops by outcome: [{"cloud_origin": false, "status": "accepted", "reason": "", "n": 355}, {"cloud_origin": false, "status": "quarantined", "reason": "actor_unknown", "n": 22}, {"cloud_origin": false, "status": "quarantined", "reason": "foreign_series", "n": 2}, {"cloud_origin": true, "status": "accepted", "reason": "", "n": 5}]; parked events billing/orders: {"billing": "0", "orders": "0"}
- Billing exceptions: []; orders exceptions: {"unresolved_line_at_close/resolved": 2, "late_ticket/resolved": 42, "late_ticket/open": 18}

## Timeline
- 07:35:00Z cast up: 3 counters, 3 captains, 2 KDS, one hub; morning: internet up for the configuration push
- 07:35:20Z internet cut for the whole trading day
- 07:35:20Z configuration published: 7/7 device intents, 3/3 cloud emits
- 07:59:20Z first bill seen
- 10:16:20Z BLACKOUT: LAN, hub and the A07 counter are off
- 10:47:20Z POWER RETURNS
- 12:47:40Z A09 isolated; A07 and A08 partitioned from each other for the close
- 12:47:40Z wrong-tax bill AAE/2627/000001 recorded; its print failed under the harness's doctored view (not_found: bill 01a20d98-fde0-7006-989e-4ddb275549ed); printing it now

Findings drawn from this and the other runs: `docs/execution/phase-2/exit-checks.md`.

## Artifacts

- `artifacts/vitest.out`
- `artifacts/run.result.json`
- `artifacts/gw.log`
- `artifacts/times.json`
- `artifacts/billing-asis.log`
