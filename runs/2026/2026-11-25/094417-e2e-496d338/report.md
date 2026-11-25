# ADR-0014 R11 trading-day soak: soak seed 3 settle

Run `094417-e2e-496d338`, 2026-11-25. Local developer machine (WSL2 Linux), full Phase 2 stack as local processes, Postgres 17 and NATS 2.15.3 in Docker under their own compose project and ports; nothing outside this machine is contacted.
Variant: **run 3: products do session, approval, brand, known_tenants; no harness workaround for them**.
Command: `soak/scripts/run-all.sh <work> r3:3:settle:0`. Wall time 72 s (the simulated day 19 s, reconnect convergence 4.6 s).
Result: **12 claims passed, 2 failed, 2 not run** (status `fail`). Seed 3, `number_at=settle`, business date 2026-11-25.

## Repositories (SHAs)

| repo | branch | SHA |
|---|---|---|
| infra | feat/phase2-local-dev | `1eb6fb3` (up-full.sh with the real-directory environment of sync; unchanged this run) |
| svc-identity | feat/phase2-identity | `6f0c495` |
| svc-tenancy | feat/phase2-tenancy | `f5629bc` |
| svc-admin | feat/phase2-admin | `24444d3` |
| svc-catalog | feat/phase2-catalog | `b319d77` |
| svc-orders | feat/phase2-orders | `03d75b9` |
| svc-billing | feat/phase2-billing | `0db7222` (as pinned, no overlay) |
| svc-sync | feat/phase2-sync | `569c19b` (cmd/gateway with SYNC_DIRECTORY=identity, no overlay; the LAN hub runs in soakgw) |
| client-core | feat/phase2-integration | `1e26251` |
| app-restaurant | feat/phase2-app | `e7cc713` |
| go-common | feat/phase2-libs | `88a6ae9` |
| go-auth | feat/phase2-auth | `94f1970` |
| api-specs-p2 | feat/phase2-specs | `1c94c67` |

## Claims

| claim | result | first failing check |
|---|---|---|
| T1 Convergence | PASS |  |
| T2 Money | PASS |  |
| T3 Numbers | PASS |  |
| T4 Tax | PASS |  |
| T5 Kitchen | PASS |  |
| T6 Printing | PASS |  |
| T7 Nothing silent | PASS |  |
| T8 Day close | PASS |  |
| T9 Projections | FAIL | orders: state_hash differs; 1 entity differ: check: 49 rows, content hash differs |
| T10 Trading is never gated | NOT_RUN | not exercisable on this build: app-restaurant has no entitlement snapshot or subscription check (grep: none); the nearest condition, an expired offline pack, only refuses new PIN sign-ins in StubIdentity and was not run |
| T11 Business day | FAIL | original Z 1165300 + revisions 15800 != all bills 1198200 (late bills on A09 total 32900) |
| T12 Numbering trigger | PASS |  |
| T13 Blackout | PASS |  |
| X1 No duplicate ops after lost acks | PASS |  |
| X3 Cloud restore epoch | NOT_RUN | no restore epoch in this seed |
| X2 Erasure of a customer's details | PASS |  |

## Evidence and failures per claim

### T1 Convergence: PASS
- pass: kds plain hash equals the others': [["K9K",true],["K8K",true]] (informative: the commitment-only hash is meant to be equal on a kds too)
- pass: 8 devices hold one plain stateHash (no op held) 6b1ebd285470a510…
- pass: plain stateHash groups: 1; accepted groups: 1; held per device [["A07",0],["A08",0],["A09",0],["B3K",0],["B4K",0],["B5K",0],["K9K",0],["K8K",0]]
- pass: vector key order differs between devices (JSON text): true
- pass: every device holds the same version vector (origin to seq, order-independent)
- pass: pii placement: A07(pos)=values A08(pos)=values A09(pos)=values B3K(waiter)=values B4K(waiter)=values B5K(waiter)=values K9K(kds)=none K8K(kds)=none
- pass: the hub holds the customer's name (2 of 3 op_pii rows)
- pass: the cloud's outlet vector equals the devices' (origins compared)
- pass: converged in 4 rounds of 5 s virtual time (4575 ms wall) after the internet returned: within 60 s
- pass: scope: the nodes compared are the 8 headless devices (device state hash), the hub and cloud by vector; the real entity registry (client-core restaurantRegistry) is in use

### T2 Money: PASS
- pass: device Z-report printed offline: Z/2026-11-25/AAA total 1165300 paise over 49 bills
- pass: billing close total (1165300) equals the device Z total
- pass: billing close bill_count 49 equals the device Z bill count
- pass: billing holds 51 bills = device 51
- pass: bill totals equal to the paisa: 1198200
- pass: payment totals per tender equal: {"cash":725500,"card":226300,"upi":239600}
- pass: refunds equal (1)
- pass: credit notes equal (3)
- pass: day summary cgst = 28380
- pass: day summary sgst = 28380
- pass: day summary discounts = 9300
- pass: day summary round_off = 740
- pass: day summary bill_count = 51
- pass: day summary total = 1198200

### T3 Numbers: PASS
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
- pass: billing reports no gap in the document register
- pass: no gap alert at the gateway
- pass: device day_close rows on A07: 2 (z numbers Z/2026-11-25/AAA, Z/2026-11-25/AAC); the contract has no series section on day_close, so the section-equals-register check is the register comparison above

### T4 Tax: PASS
- pass: 49 bills verified by billing (all but the seeded one and 1 typed paper-book entry)
- pass: the 1 typed paper-book entry is recorded manual_entry
- pass: 3 credit notes verified
- pass: seeded wrong-tax bill AAE/2627/000001 is flagged: verification {"calc_v":1,"checked_at":"2026-11-25T09:45:06.152Z","difference":{"amount_paise":300,"currency":"INR"},"ruleset_id":"rs-1","status":"mismatch"}
- pass: and billing kept it unchanged (total equals the device's)
- pass: tax_mismatch exceptions in billing: 1: bill:01a2133d diff=300
- pass: every tax mismatch is the seeded one

### T5 Kitchen: PASS
- pass: KITCHEN screen held every one of its 58 KOTs' tickets with the internet cut (none missing)
- pass: KITCHEN: every KOT line exactly once
- pass: BAR screen held every one of its 58 KOTs' tickets with the internet cut (none missing)
- pass: BAR: every KOT line exactly once
- pass: order to KDS on the LAN: 12 probes, p50 37 ms, p95 45 ms (≤ 1000 ms; each probe = push to hub + pull, driven back to back)
- pass: K9K holds all 43 KITCHEN KOTs after 3 hub stops (one with a wiped disk)
- pass: K8K holds all 17 BAR KOTs after 3 hub stops (one with a wiped disk)

### T6 Printing: PASS
- pass: each of 50 bills has exactly one original print job
- pass: 1 reprint(s) carry the duplicate banner
- pass: 60 KOTs: each printed once as an original on its routed printer (stations that only show on the KDS print none; the handed-over and the stale blackout ticket print none)
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

### T9 Projections: FAIL
- pass: orders check 01a2127b: device commitments [~c:customer_gstin=2bee9c7f6c, ~c:customer_name=b7be5f0d90]; orders pii_commitments rows []; projection pii_c ["null"]
- pass: billing: service state_hash equals the device's (7562b4a2eb74886f…)
- pass: the cloud rollback was not part of this run (seed without a restore epoch)
- pass: the bus reset was not run in any seed: no bus-reset tool exists in go-common or infra (the outbox documentation names hostry-bus-reset)
- **fail:** orders: state_hash differs; 1 entity differ: check: 49 rows, content hash differs

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
- pass: revision numbers equal on every node: ["064ca9af-b614-5b95-866e-ad3c35e0b075#1"]
- pass: billing holds 1 Z revision(s), equal to the devices'
- pass: A08 closed the same date while partitioned from A07: Z numbers Z/2026-11-25/AAA and Z/2026-11-25/AAC; billing day status closed, late_document_count 1, revision_count 1
- **fail:** original Z 1165300 + revisions 15800 != all bills 1198200 (late bills on A09 total 32900)

### T12 Numbering trigger: PASS
- pass: 2 check(s) were presented (pre-bill) during the day
- pass: numbers consumed (53) equal documents that exist (53): presenting consumed none
- pass: every presented check ended billed, cancelled with approval or carried over

### T13 Blackout: PASS
- pass: the KDS station lists the ticket handed over by hand and the stale one as late tickets: ["handover","stale"]
- pass: the recent blackout ticket printed once on the station
- pass: no handed-over and no stale KOT printed as an original after power returned
- pass: bills finalised on the battery counter in the blackout keep their series (AAC: gapless per T3) and their original print exists (T6)
- pass: nodes converged 2 rounds (5 s virtual each, 921 ms wall) after the LAN returned: within 60 s
- pass: the paper page typed twice (2 entries) is 1 bill: MB1/2627/000007
- pass: the handover is recorded and listed on the KDS side

### X1 No duplicate ops after lost acks: PASS
- pass: A07: 209 own ops, gateway stores 209, contiguous from 1
- pass: A08: 112 own ops, gateway stores 112, contiguous from 1
- pass: A09: 13 own ops, gateway stores 13, contiguous from 1
- pass: B3K: 35 own ops, gateway stores 35, contiguous from 1
- pass: B4K: 21 own ops, gateway stores 21, contiguous from 1
- pass: B5K: 14 own ops, gateway stores 14, contiguous from 1
- pass: K9K: 18 own ops, gateway stores 18, contiguous from 1
- pass: K8K: 8 own ops, gateway stores 8, contiguous from 1
- pass: 1 push answer(s) were lost after the server applied them; 3 hub stop(s) incl. 1 disk wipe; 1 device restart(s)
- pass: 439 outbox events, all distinct ids

### X3 Cloud restore epoch: NOT_RUN
- **fail:** no restore epoch in this seed

### X2 Erasure of a customer's details: PASS
- pass: erasable side tables (sync_pii) holding the values before: billing 1, orders 1
- pass: before erasure: details present in billing false, orders false, device A07 true, gateway op_pii rows 3
- pass: customer details were present somewhere before the erasure (the test is meaningful)
- pass: gateway erasure issued for 1 op(s) (POST /internal/v1/pii/erase as billing)
- pass: billing no longer returns the name or GSTIN
- pass: orders no longer returns the name or GSTIN
- pass: no device holds the details any more
- pass: billing and orders side tables no longer hold the values (1/1 before, 0/0 after)
- pass: device A07 holds 0 pii_erase record(s); ["cloud-genesis"] cloud origins in its vector
- pass: the gateway's op_pii rows of those ops are gone
- pass: the copy on the internal queue ages out within 7 days (ADR-0014 A2.1 item 7): not waited for, stated in the erasure promise

## Run facts
- Script steps: 105 run, 0 failed: []
- Hub stops 3 (disk wipes 1), device restarts ['B4K'], lost push answers 1, approvals 2, bills by device {"A07": 28, "A08": 16, "A09": 2}
- Offline Z: {"zNumber": "Z/2026-11-25/AAA", "totalPaise": 1165300, "billCount": 49}; second close on the partitioned counter: {"zNumber": "Z/2026-11-25/AAC", "totalPaise": 1165300, "billCount": 49}
- Gateway ops by outcome: [{"cloud_origin": false, "status": "accepted", "reason": "", "n": 435}, {"cloud_origin": true, "status": "accepted", "reason": "", "n": 7}]; parked events billing/orders: {"billing": "0", "orders": "0"}
- Billing exceptions: ["late_document/info/open/bill", "tax_mismatch/high/open/bill", "late_document/info/open/business_day", "line_on_two_bills/high/open/bill", "two_bills_one_check/warning/open/check", "payments_exceed_total/high/resolved/bill", "manual_entry/warning/open/bill"]; orders exceptions: {"unresolved_line_at_close/resolved": 2, "late_ticket/open": 1}

## Timeline
- 09:44:31Z cast up: 3 counters, 3 captains, 2 KDS, one hub; morning: internet up for the configuration push
- 09:44:51Z internet cut for the whole trading day
- 09:44:51Z configuration published: 7/7 device intents, 3/3 cloud emits
- 10:10:51Z first bill seen
- 12:33:51Z BLACKOUT: LAN, hub and the A07 counter are off
- 13:04:51Z POWER RETURNS
- 15:05:11Z A09 isolated; A07 and A08 partitioned from each other for the close
- 15:05:11Z wrong-tax bill AAE/2627/000001 recorded; its print failed under the harness's doctored view (not_found: bill 01a2133d-41f7-7006-9126-1a35ee3673a1); printing it now

Findings drawn from this and the other runs: `docs/execution/phase-2/exit-checks.md, section Run 3`.

## Artifacts

- `artifacts/vitest.out`
- `artifacts/run.result.json`
- `artifacts/gw.log`
- `artifacts/times.json`
- `artifacts/sync.log`
- `artifacts/billing.log`
- `artifacts/orders.log`
