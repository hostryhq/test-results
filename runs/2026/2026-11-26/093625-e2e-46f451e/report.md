# ADR-0014 R11 trading-day soak: soak seed 2 settle (run 6)

Run `093625-e2e-46f451e`, 2026-11-26. Local developer machine (WSL2 Linux), full Phase 2 stack as local processes, Postgres 17 and NATS 2.15.3 in Docker under their own compose project and ports; nothing outside this machine is contacted.
Variant: **cloud restore (X3); real cmd/hub; hub dev clock**.
Command: `soak/scripts/run-all.sh r2:2:settle:1 (work dir /tmp/hostry-soak/w6)`. Wall time 153 s (the simulated day 39 s, reconnect convergence 9.4 s).
Result: **22 claims passed, 0 failed, 1 not run** (status `partial`). Seed 2, `number_at=settle`, business date 2026-11-26.

## Repositories (SHAs)

| repo | branch | SHA |
|---|---|---|
| svc-identity | feat/phase2-identity | `d51e743` |
| svc-tenancy | feat/phase2-tenancy | `33e2db4` |
| svc-billing | feat/phase2-billing | `9b8d6fa` |
| svc-orders | feat/phase2-orders | `6bfa33d` |
| svc-catalog | feat/phase2-catalog | `fcf4b93` |
| svc-admin | feat/phase2-admin | `24444d3` |
| svc-sync | feat/phase2-sync | `c0499fa` |
| client-core | feat/phase2-integration | `a74d521` |
| app-restaurant | feat/phase2-app | `3b80cb6` |
| api-specs-p2 | feat/phase2-specs | `17de74f` |
| go-common | feat/phase2-libs | `88a6ae9` |
| infra | feat/phase2-local-dev | `1eb6fb3` |
| e2e-tests | feat/phase2-soak | `46f451e` |

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
| T9 Projections | PASS |  |
| T10 Trading is never gated | PASS |  |
| T11 Business day | PASS |  |
| T12 Numbering trigger | PASS |  |
| T13 Blackout | PASS |  |
| X1 No duplicate ops after lost acks | PASS |  |
| X3 Cloud restore epoch | PASS |  |
| X2 Erasure of a customer's details | PASS |  |
| S1 Payment arrives before its bill | PASS |  |
| S2 Two tills credit the same bill | PASS |  |
| S3 Late bill arrives before the day close | PASS |  |
| S4 Fast-clock device, Z coverage by vector | PASS |  |
| S5 Duplicate invoice number under two bill ids | PASS |  |
| S6 Override citing a non-existent approval | NOT_RUN |  |
| S7 Re-bill of a paid bill: cashier refused, manager with another approver's approval accepted | PASS |  |

## Evidence and failures per claim

### T1 Convergence: PASS
- pass: kds plain hash equals the others': [["K9K",true],["K8K",true]] (informative: the commitment-only hash is meant to be equal on a kds too)
- pass: 8 devices hold one acceptedStateHash (1 ops held on A09:1) 28f7456df2a1c3a9…
- pass: plain stateHash groups: 2; accepted groups: 1; held per device [["A07",0],["A08",0],["A09",1],["B3K",0],["B4K",0],["B5K",0],["K9K",0],["K8K",0]]
- pass: vector key order differs between devices (JSON text): true
- pass: every device holds the same version vector (origin to seq, order-independent)
- pass: pii placement: A07(pos)=values A08(pos)=values A09(pos)=values B3K(waiter)=values B4K(waiter)=values B5K(waiter)=values K9K(kds)=none K8K(kds)=none
- pass: the real hub holds 4 op_pii row(s) and the customer's name is in the clear in none of its files (column encryption)
- pass: the cloud's outlet vector equals the devices' (origins compared)
- pass: converged in 3 rounds of 5 s virtual time (9352 ms wall) after the internet returned: within 60 s
- pass: scope: the nodes compared are the 8 headless devices (device state hash), the hub and cloud by vector; the real entity registry (client-core restaurantRegistry) is in use

### T2 Money: PASS
- pass: device Z-report printed offline: Z/2026-11-26/AAA total 1352100 paise over 55 bills
- pass: billing close total (1352100) equals the device Z total
- pass: billing close bill_count 55 equals the device Z bill count
- pass: billing holds 57 bills = device 57
- pass: bill totals equal to the paisa: 1385000
- pass: payment totals per tender equal: {"cash":712200,"upi":428650,"card":207850}
- pass: refunds equal (1)
- pass: credit notes equal (6)
- pass: day summary cgst = 32246
- pass: day summary sgst = 32246
- pass: day summary discounts = 38650
- pass: day summary round_off = 1158
- pass: day summary bill_count = 57
- pass: day summary total = 1385000

### T3 Numbers: PASS
- pass: A07: no gap or duplicate in its series register
- pass: A08: no gap or duplicate in its series register
- pass: A09: no gap or duplicate in its series register
- pass: B3K: no gap or duplicate in its series register
- pass: B4K: no gap or duplicate in its series register
- pass: B5K: no gap or duplicate in its series register
- pass: K9K: no gap or duplicate in its series register
- pass: K8K: no gap or duplicate in its series register
- pass: 63 document numbers, all unique across the outlet
- pass: series AAA/2627: 1..36 gapless
- pass: series AAC/2627: 1..18 gapless
- pass: series AAE/2627: 1..2 gapless
- pass: series AAB/2627: 1..5 gapless
- pass: series AAD/2627: 1..1 gapless
- pass: billing register AAA/2627: count 36, last 36, gaps 0
- pass: billing register AAC/2627: count 18, last 18, gaps 0
- pass: billing register AAE/2627: count 2, last 2, gaps 0
- pass: billing register AAB/2627: count 5, last 5, gaps 0
- pass: billing register AAD/2627: count 1, last 1, gaps 0
- pass: billing reports no gap in the document register
- pass: no gap alert at the gateway
- pass: device day_close rows on A07: 2 (z numbers Z/2026-11-26/AAA, Z/2026-11-26/AAC); the contract has no series section on day_close, so the section-equals-register check is the register comparison above

### T4 Tax: PASS
- pass: 55 bills verified by billing (all but the seeded one and 1 typed paper-book entry)
- pass: the 1 typed paper-book entry is recorded manual_entry
- pass: 6 credit notes verified
- pass: seeded wrong-tax bill AAE/2627/000001 is flagged: verification {"calc_v":1,"checked_at":"2026-11-26T09:37:19.888Z","difference":{"amount_paise":300,"currency":"INR"},"ruleset_id":"rs-1","status":"mismatch"}
- pass: and billing kept it unchanged (total equals the device's)
- pass: tax_mismatch exceptions in billing: 1: bill:01a2173c diff=300
- pass: every tax mismatch is the seeded one

### T5 Kitchen: PASS
- pass: KITCHEN screen held every one of its 67 KOTs' tickets with the internet cut (none missing)
- pass: KITCHEN: every KOT line exactly once
- pass: BAR screen held every one of its 67 KOTs' tickets with the internet cut (none missing)
- pass: BAR: every KOT line exactly once
- pass: order to KDS on the LAN: 12 probes, p50 27 ms, p95 30 ms (≤ 1000 ms; each probe = push to hub + pull, driven back to back)
- pass: K9K holds all 51 KITCHEN KOTs after 3 hub stops (one with a wiped disk)
- pass: K8K holds all 20 BAR KOTs after 3 hub stops (one with a wiped disk)

### T6 Printing: PASS
- pass: each of 56 bills has exactly one original print job
- pass: 7 reprint(s) carry the duplicate banner
- pass: 71 KOTs: each printed once as an original on its routed printer (stations that only show on the KDS print none; the handed-over and the stale blackout ticket print none)
- pass: scope: the fake printers capture the ESC/POS bytes of the bill printer and the kitchen printer per device; counts are of decoded text matches on the document number

### T7 Nothing silent: PASS
- pass: every node shows the same 0 detector finding(s): 
- pass: seeded double payment: made; overpayment finding still open after the refund: 0; net paid after refund 15800
- pass: the double payment was seeded
- pass: line-on-two-bills race appears in billing exceptions: line_on_two_bills, two_bills_one_check
- pass: the double payment appears in billing exceptions: payments_exceed_total, payment_unmatched, payment_unmatched
- pass: the race is closed by a credit note op
- pass: the double payment is closed by a refund op
- pass: attention made of: [{"dev":"A07","held":0,"refused":0,"conflicts":[],"clock_held":[]},{"dev":"A08","held":0,"refused":0,"conflicts":[],"clock_held":[]}] (same on every device: false)
- pass: ops shown as needing attention equal the quarantined or voided: A09 1=1

### T8 Day close: PASS
- pass: day close was refused while an order line was unresolved (day_blocked)
- pass: day close succeeded offline after resolution: Z/2026-11-26/AAA
- pass: orders exceptions around close: [["unresolved_line_at_close/resolved",35]]

### T9 Projections: PASS
- pass: orders: service state_hash equals the device's (8a30e27113c64359…)
- pass: billing: service state_hash equals the device's (75d77c474f15e052…)
- pass: before the cloud rollback, orders: equal to the device digest; after it the service state_hash is 8a30e27113c6…
- pass: before the cloud rollback, billing: equal to the device digest; after it the service state_hash is 75d77c474f15…
- pass: the bus reset was not run in any seed: no bus-reset tool exists in go-common or infra (the outbox documentation names hostry-bus-reset)

### T10 Trading is never gated: PASS
- pass: all 8 devices held the pack's snapshot from the morning (v 3, plan soak_full, active, restaurant.kds and restaurant.captain granted)
- pass: the subscription was overdue (a charge for a long-past period was open) for the whole trading day
- pass: all 114 steps of the trading day succeeded on devices that held an overdue subscription and a kitchen-screen plan; 55 bills made
- pass: mid-day, internet cut: the vendor changed the plan to one without restaurant.kds (soak_lite overdue=true); the subscription was still overdue
- pass: while offline no device changed what it held (same version, features and status as the morning)
- pass: on the pack refresh every device adopted the higher snapshot (v3 to v4)
- pass: after adoption restaurant.kds is off and restaurant.captain stays on, on every device
- pass: an old pack (the morning's version) offered again does not lower the version
- pass: a malformed snapshot is ignored
- pass: with the internet cut again both kitchen screens find restaurant.kds absent from the held snapshot: kitchen mode is unavailable offline for a plan without it
- pass: synthetic status suspended, day in progress: trading goes on and the day is marked as the last (ADR-0014 A1.2)
- pass: synthetic status suspended or closed, the real day closed: no new trading starts after the day
- pass: no snapshot at all: trading continues
- pass: suspension and closure statuses are synthetic: the vendor suspension needs an approval and is refused while a day is open (admin suspendOrganisation), so the status was set on the adopted snapshot and the app's own tradingStanding decided against the device's real day rows

### T11 Business day: PASS
- pass: A07: one business_day for 2026-11-26
- pass: A08: one business_day for 2026-11-26
- pass: A09: one business_day for 2026-11-26
- pass: B3K: one business_day for 2026-11-26
- pass: B4K: one business_day for 2026-11-26
- pass: B5K: one business_day for 2026-11-26
- pass: K9K: one business_day for 2026-11-26
- pass: K8K: one business_day for 2026-11-26
- pass: the original Z totals on the device are unchanged after the late bills arrived
- pass: revision numbers equal on every node: ["be57a364-3e4a-5764-b843-22e38c56281e#1"]
- pass: billing holds 1 Z revision(s), equal to the devices'
- pass: original Z plus revisions equals all bills (1385000)
- pass: A08 closed the same date while partitioned from A07: Z numbers Z/2026-11-26/AAA and Z/2026-11-26/AAC; billing day status closed, late_document_count 5, revision_count 1

### T12 Numbering trigger: PASS
- pass: 2 check(s) were presented (pre-bill) during the day
- pass: numbers consumed (62) equal documents that exist (62): presenting consumed none
- pass: every presented check ended billed, cancelled with approval or carried over

### T13 Blackout: PASS
- pass: the KDS station lists the ticket handed over by hand and the stale one as late tickets: ["handover","stale"]
- pass: the recent blackout ticket printed once on the station
- pass: no handed-over and no stale KOT printed as an original after power returned
- pass: bills finalised on the battery counter in the blackout keep their series (AAC: gapless per T3) and their original print exists (T6)
- pass: nodes converged 2 rounds (5 s virtual each, 846 ms wall) after the LAN returned: within 60 s
- pass: the paper page typed twice (2 entries) is 1 bill: MB1/2627/000007
- pass: the handover is recorded and listed on the KDS side
- pass: the stale ticket was 30 min old at the station and the recent one 5 min
- pass: the hub's clock ran 960 s ahead and back to 0: 8 devices synced in both states, no error

### X1 No duplicate ops after lost acks: PASS
- pass: A07: 244 own ops, gateway stores 244, contiguous from 1
- pass: A08: 136 own ops, gateway stores 136, contiguous from 1
- pass: A09: 19 own ops, gateway stores 19, contiguous from 1
- pass: B3K: 33 own ops, gateway stores 33, contiguous from 1
- pass: B4K: 27 own ops, gateway stores 27, contiguous from 1
- pass: B5K: 17 own ops, gateway stores 17, contiguous from 1
- pass: K9K: 20 own ops, gateway stores 20, contiguous from 1
- pass: K8K: 7 own ops, gateway stores 7, contiguous from 1
- pass: 1 push answer(s) were lost after the server applied them; 3 hub stop(s) incl. 1 disk wipe; 1 device restart(s)
- pass: 512 outbox events, all distinct ids

### X3 Cloud restore epoch: PASS
- pass: cloud ops: 500 at backup, 512 before the restore, 500 right after it, 512 after the nodes re-offered
- pass: the restore really rolled the cloud back (it lost ops)
- pass: the cloud holds every op again after 3 rounds (9123 ms wall)
- pass: the real hub's own epoch report is not observable to the harness (it runs its rounds itself); what is checked is the effect: every device converges to the restated cloud and no state hash changes
- pass: no device's state changed through the rollback and the re-offer (state hash equal before and after, per device)

### X2 Erasure of a customer's details: PASS
- pass: erasable side tables (sync_pii) holding the values before: billing 1, orders 1
- pass: before erasure: details present in billing false, orders false, device A07 true, gateway op_pii rows 4
- pass: customer details were present somewhere before the erasure (the test is meaningful)
- pass: gateway erasure issued for 1 op(s) (POST /internal/v1/pii/erase as billing)
- pass: billing no longer returns the name or GSTIN
- pass: orders no longer returns the name or GSTIN
- pass: no device holds the details any more
- pass: billing and orders side tables no longer hold the values (1/1 before, 0/0 after)
- pass: device A07 holds 0 pii_erase record(s); ["cloud-genesis","cloud-soak-restore-2"] cloud origins in its vector
- pass: the gateway's op_pii rows of those ops are gone
- pass: the copy on the internal queue ages out within 7 days (ADR-0014 A2.1 item 7): not waited for, stated in the erasure promise

### S1 Payment arrives before its bill: PASS
- pass: the script made a payment on a device other than the bill's issuer
- pass: billing projected the payment while its bill was still absent (arrival order forced)
- pass: tender card: 207850 paise in 12 payment(s) on both sides
- pass: tender cash: 712200 paise in 43 payment(s) on both sides
- pass: tender upi: 428650 paise in 14 payment(s) on both sides
- pass: the bill's payments total 13100 paise on both sides
- pass: the payment's payment_unmatched (1 seen) is cleared now that its bill arrived
- pass: no other payment_unmatched is open (only the quarantined duplicate's, asserted by S5)

### S2 Two tills credit the same bill: PASS
- pass: two tills each credited the same bill
- pass: bill 01a2173b (22100 paise) was credited 44200 paise by 2 notes
- pass: the over-credit is flagged: credit_exceeds_bill
- pass: output tax reduced once: credit tax 3526 paise (expected 3526, tolerance 6)

### S3 Late bill arrives before the day close: PASS
- pass: the late bill(s) were projected before any close arrived (2)
- pass: 2 finalize op(s) of the isolated device are outside the Z
- pass: late bill 01a2173c (op 7) is in exactly one revision
- pass: late bill 01a2173c (op 13) is in exactly one revision
- pass: arrival: {"planned":["A09","A08","A07"],"pushed":[{"device":"A09","ok":true},{"device":"A08","ok":true},{"device":"A07","ok":true}],"lateBeforeClose":{"reached":true,"billIds":["01a2173c-02fc-7006-a91b-440fb2dce2d5","01a2173c-054e-7006-b4d2-726f0ada580b"],"dayClosedAtThatTime":false},"paymentFirst":{"reached":true,"billId":"01a2173b-deac-7009-ad79-6c8dbf5d7216","paymentId":"01a21742-48d2-7000-a111-c810889e6bf9"}}; billing day closed, late_document_count 5, revisions 1

### S4 Fast-clock device, Z coverage by vector: PASS
- pass: a bill was issued on the fast-clock device
- pass: the closing device held that bill's op before it closed
- pass: the bill's own time is 7 min after the close time of the closing device
- pass: no Z revision covers the fast-clock bill (the Z holds it, by vector)
- pass: original Z plus revisions equals all bills (1385000)

### S5 Duplicate invoice number under two bill ids: PASS
- pass: the same number was issued under two bill ids
- pass: the gateway refused the second op as duplicate_document (quarantined)
- pass: billing kept the number with the first bill only
- pass: settle mode: the 0 payment(s) of the quarantined bill travelled inside its finalize op; billing holds none of them (no orphan to flag)

### S6 Override citing a non-existent approval: NOT_RUN
- pass: not waited in this seed (SOAK_APPROVAL_WINDOW_MS=0); at the end of the run the gateway shows the op as accepted [approval_pending]

### S7 Re-bill of a paid bill: cashier refused, manager with another approver's approval accepted: PASS
- pass: a cashier attempted a re-bill of a paid bill without approval
- pass: the bill carried 1 payment(s)
- pass: the cashier was refused on the device (approval_needed)
- pass: a manager re-billed it citing Anand's approval
- pass: the approval came from another approver than the requester (Anand for Meena)
- pass: billing holds the credit note and 1 replacement bill(s) of the re-bill
- pass: billing raised no rebill_after_payment for the approved re-bill

## Run facts
- Script steps: 114 run, 0 failed: []
- Hub stops 3 (disk wipes 1), device restarts ['B4K'], lost push answers 1, approvals 2, bills by device {"A07": 33, "A08": 18, "A09": 4}
- Offline Z: {"zNumber": "Z/2026-11-26/AAA", "totalPaise": 1352100, "billCount": 55}; second close on the partitioned counter: {"zNumber": "Z/2026-11-26/AAC", "totalPaise": 1352100, "billCount": 55}
- Gateway ops by outcome: [{"cloud_origin": false, "status": "accepted", "reason": "", "n": 507}, {"cloud_origin": false, "status": "quarantined", "reason": "duplicate_document", "n": 1}, {"cloud_origin": true, "status": "accepted", "reason": "", "n": 8}]; parked events billing/orders: {"billing": "0", "orders": "0"}
- Billing exceptions: ["late_document/info/open/business_day", "credit_exceeds_bill/high/open/bill", "line_on_two_bills/high/open/bill", "two_bills_one_check/warning/open/check", "payments_exceed_total/high/resolved/bill", "late_document/info/open/payment", "late_document/info/open/bill", "late_document/info/open/payment", "late_document/info/open/bill", "late_document/info/open/payment", "manual_entry/warning/open/bill", "payment_unmatched/warning/resolved/payment", "payment_unmatched/warning/resolved/payment", "tax_mismatch/high/open/bill"]; orders exceptions: {"late_ticket/open": 1, "unresolved_line_at_close/resolved": 35}
- Cloud restore: {"ops_at_backup": 500, "ops_before_restore": 512, "hashes_before": {"A07": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2", "A08": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2", "A09": "73be72ea879346e48a310283459570838fd9875a3948ed67f6adff02249bb35c", "B3K": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2", "B4K": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2", "B5K": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2", "K9K": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2", "K8K": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2"}, "ops_after_restore": 500, "ops_after_reoffer": 512, "hashes_after": {"A07": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2", "A08": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2", "A09": "73be72ea879346e48a310283459570838fd9875a3948ed67f6adff02249bb35c", "B3K": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2", "B4K": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2", "B5K": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2", "K9K": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2", "K8K": "28f7456df2a1c3a9410b999ecbbeaa4142ed10d3f4cbeb96725ab58266c9add2"}, "settle_rounds": 3, "reoffer_wall_ms": 9123, "hub_epoch_changed": null}

## Timeline
- 09:36:29Z cast up: 3 counters, 3 captains, 2 KDS, one hub; morning: internet up for the configuration push
- 09:37:53Z internet cut for the whole trading day
- 09:37:53Z configuration published: 7/7 device intents, 3/3 cloud emits
- 09:42:00Z first bill seen
- 09:42:12Z BLACKOUT: LAN, hub and the A07 counter are off
- 09:42:13Z POWER RETURNS
- 09:42:18Z A09 isolated; A07 and A08 partitioned from each other for the close
- 09:42:18Z wrong-tax bill AAE/2627/000001 recorded; its print failed under the harness's doctored view (not_found: bill 01a2173c-02fc-7006-a91b-440fb2dce2d5); printing it now

Findings drawn from this and the other runs: `docs/execution/phase-2/exit-checks.md (Run 6)`.

## Artifacts

- `artifacts/vitest.out`
- `artifacts/gw.log`
- `artifacts/r2.hub.log`
- `artifacts/onboard.sanitised.json`
- `artifacts/provision.log`
- `artifacts/run.result.json`
