# ADR-0014 R11 trading-day soak: soak seed 4 print (run 6)

Run `094227-e2e-46f451e`, 2026-11-26. Local developer machine (WSL2 Linux), full Phase 2 stack as local processes, Postgres 17 and NATS 2.15.3 in Docker under their own compose project and ports; nothing outside this machine is contacted.
Variant: **number_at=print (M5); real cmd/hub; hub dev clock**.
Command: `soak/scripts/run-all.sh r4:4:print:0 (work dir /tmp/hostry-soak/w6)`. Wall time 115 s (the simulated day 40 s, reconnect convergence 12.4 s).
Result: **21 claims passed, 0 failed, 2 not run** (status `partial`). Seed 4, `number_at=print`, business date 2026-11-26.

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
| X3 Cloud restore epoch | NOT_RUN | no restore epoch in this seed |
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
- pass: 8 devices hold one acceptedStateHash (1 ops held on A09:1) 5c8bfea754975719…
- pass: plain stateHash groups: 2; accepted groups: 1; held per device [["A07",0],["A08",0],["A09",1],["B3K",0],["B4K",0],["B5K",0],["K9K",0],["K8K",0]]
- pass: vector key order differs between devices (JSON text): true
- pass: every device holds the same version vector (origin to seq, order-independent)
- pass: pii placement: A07(pos)=values A08(pos)=values A09(pos)=values B3K(waiter)=values B4K(waiter)=values B5K(waiter)=values K9K(kds)=none K8K(kds)=none
- pass: the real hub holds 4 op_pii row(s) and the customer's name is in the clear in none of its files (column encryption)
- pass: the cloud's outlet vector equals the devices' (origins compared)
- pass: converged in 4 rounds of 5 s virtual time (12376 ms wall) after the internet returned: within 60 s
- pass: scope: the nodes compared are the 8 headless devices (device state hash), the hub and cloud by vector; the real entity registry (client-core restaurantRegistry) is in use

### T2 Money: PASS
- pass: device Z-report printed offline: Z/2026-11-26/AAA total 1363100 paise over 55 bills
- pass: billing close total (1363100) equals the device Z total
- pass: billing close bill_count 55 equals the device Z bill count
- pass: billing holds 57 bills = device 57
- pass: bill totals equal to the paisa: 1396000
- pass: payment totals per tender equal: {"cash":656350,"upi":522200,"card":188250}
- pass: refunds equal (1)
- pass: credit notes equal (6)
- pass: day summary cgst = 32648
- pass: day summary sgst = 32648
- pass: day summary discounts = 35100
- pass: day summary round_off = 1304
- pass: day summary bill_count = 57
- pass: day summary total = 1396000

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
- pass: seeded wrong-tax bill AAE/2627/000001 is flagged: verification {"calc_v":1,"checked_at":"2026-11-26T09:43:21.194Z","difference":{"amount_paise":300,"currency":"INR"},"ruleset_id":"rs-1","status":"mismatch"}
- pass: and billing kept it unchanged (total equals the device's)
- pass: tax_mismatch exceptions in billing: 1: bill:01a21741 diff=300
- pass: every tax mismatch is the seeded one

### T5 Kitchen: PASS
- pass: KITCHEN screen held every one of its 79 KOTs' tickets with the internet cut (none missing)
- pass: KITCHEN: every KOT line exactly once
- pass: BAR screen held every one of its 79 KOTs' tickets with the internet cut (none missing)
- pass: BAR: every KOT line exactly once
- pass: order to KDS on the LAN: 12 probes, p50 27 ms, p95 31 ms (≤ 1000 ms; each probe = push to hub + pull, driven back to back)
- pass: K9K holds all 55 KITCHEN KOTs after 3 hub stops (one with a wiped disk)
- pass: K8K holds all 28 BAR KOTs after 3 hub stops (one with a wiped disk)

### T6 Printing: PASS
- pass: each of 56 bills has exactly one original print job
- pass: 1 reprint(s) carry the duplicate banner
- pass: 83 KOTs: each printed once as an original on its routed printer (stations that only show on the KDS print none; the handed-over and the stale blackout ticket print none)
- pass: scope: the fake printers capture the ESC/POS bytes of the bill printer and the kitchen printer per device; counts are of decoded text matches on the document number

### T7 Nothing silent: PASS
- pass: every node shows the same 0 detector finding(s): 
- pass: seeded double payment: made; overpayment finding still open after the refund: 0; net paid after refund 15800
- pass: the double payment was seeded
- pass: line-on-two-bills race appears in billing exceptions: line_on_two_bills, two_bills_one_check
- pass: the double payment appears in billing exceptions: payments_exceed_total, payment_unmatched, payment_unmatched, payment_unmatched, payment_unmatched
- pass: the race is closed by a credit note op
- pass: the double payment is closed by a refund op
- pass: attention made of: [{"dev":"A07","held":0,"refused":0,"conflicts":[],"clock_held":[]},{"dev":"A08","held":0,"refused":0,"conflicts":[],"clock_held":[]}] (same on every device: false)
- pass: ops shown as needing attention equal the quarantined or voided: A09 1=1

### T8 Day close: PASS
- pass: day close was refused while an order line was unresolved (day_blocked)
- pass: day close succeeded offline after resolution: Z/2026-11-26/AAA
- pass: orders exceptions around close: [["unresolved_line_at_close/resolved",45]]

### T9 Projections: PASS
- pass: orders: service state_hash equals the device's (bf3595c58889a6b3…)
- pass: billing: service state_hash equals the device's (48afba9fb7188b67…)
- pass: the cloud rollback was not part of this run (seed without a restore epoch)
- pass: the bus reset was not run in any seed: no bus-reset tool exists in go-common or infra (the outbox documentation names hostry-bus-reset)

### T10 Trading is never gated: PASS
- pass: all 8 devices held the pack's snapshot from the morning (v 3, plan soak_full, active, restaurant.kds and restaurant.captain granted)
- pass: the subscription was overdue (a charge for a long-past period was open) for the whole trading day
- pass: all 115 steps of the trading day succeeded on devices that held an overdue subscription and a kitchen-screen plan; 55 bills made
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
- pass: revision numbers equal on every node: ["7af80824-025a-51ec-80e3-5f05152dfc6a#1"]
- pass: billing holds 1 Z revision(s), equal to the devices'
- pass: original Z plus revisions equals all bills (1396000)
- pass: A08 closed the same date while partitioned from A07: Z numbers Z/2026-11-26/AAA and Z/2026-11-26/AAC; billing day status closed, late_document_count 5, revision_count 1

### T12 Numbering trigger: PASS
- pass: print mode: 56 invoices were issued and no pre-bill row exists (Present finalised, it never recorded an unnumbered pre-bill)
- pass: numbers consumed (62) equal documents that exist (62)
- pass: no check is left open without a bill or a cancellation
- pass: 37 Present(s) were pressed in print mode
- pass: every one of the 37 payment flows pressed Present first
- pass: Present on AAA/2627/000013 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000013 recorded no payment op
- pass: no payment existed on AAA/2627/000013 right after Present
- pass: AAA/2627/000013 is the one bill of its check
- pass: Present on AAC/2627/000001 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000001 recorded no payment op
- pass: no payment existed on AAC/2627/000001 right after Present
- pass: AAC/2627/000001 is the one bill of its check
- pass: Present on AAA/2627/000014 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000014 recorded no payment op
- pass: no payment existed on AAA/2627/000014 right after Present
- pass: AAA/2627/000014 is the one bill of its check
- pass: Present on AAC/2627/000002 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000002 recorded no payment op
- pass: no payment existed on AAC/2627/000002 right after Present
- pass: AAC/2627/000002 is the one bill of its check
- pass: Present on AAA/2627/000015 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000015 recorded no payment op
- pass: no payment existed on AAA/2627/000015 right after Present
- pass: AAA/2627/000015 is the one bill of its check
- pass: Present on AAC/2627/000003 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000003 recorded no payment op
- pass: no payment existed on AAC/2627/000003 right after Present
- pass: AAC/2627/000003 is the one bill of its check
- pass: Present on AAA/2627/000016 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000016 recorded no payment op
- pass: no payment existed on AAA/2627/000016 right after Present
- pass: AAA/2627/000016 is the one bill of its check
- pass: Present on AAC/2627/000004 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000004 recorded no payment op
- pass: no payment existed on AAC/2627/000004 right after Present
- pass: AAC/2627/000004 is the one bill of its check
- pass: Present on AAA/2627/000017 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000017 recorded no payment op
- pass: no payment existed on AAA/2627/000017 right after Present
- pass: AAA/2627/000017 is the one bill of its check
- pass: Present on AAC/2627/000005 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000005 recorded no payment op
- pass: no payment existed on AAC/2627/000005 right after Present
- pass: AAC/2627/000005 is the one bill of its check
- pass: Present on AAA/2627/000018 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000018 recorded no payment op
- pass: no payment existed on AAA/2627/000018 right after Present
- pass: AAA/2627/000018 is the one bill of its check
- pass: Present on AAC/2627/000006 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000006 recorded no payment op
- pass: no payment existed on AAC/2627/000006 right after Present
- pass: AAC/2627/000006 is the one bill of its check
- pass: Present on AAA/2627/000019 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000019 recorded no payment op
- pass: no payment existed on AAA/2627/000019 right after Present
- pass: AAA/2627/000019 is the one bill of its check
- pass: Present on AAC/2627/000007 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000007 recorded no payment op
- pass: no payment existed on AAC/2627/000007 right after Present
- pass: AAC/2627/000007 is the one bill of its check
- pass: Present on AAA/2627/000020 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000020 recorded no payment op
- pass: no payment existed on AAA/2627/000020 right after Present
- pass: AAA/2627/000020 is the one bill of its check
- pass: Present on AAC/2627/000008 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000008 recorded no payment op
- pass: no payment existed on AAC/2627/000008 right after Present
- pass: AAC/2627/000008 is the one bill of its check
- pass: Present on AAA/2627/000021 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000021 recorded no payment op
- pass: no payment existed on AAA/2627/000021 right after Present
- pass: AAA/2627/000021 is the one bill of its check
- pass: Present on AAC/2627/000009 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000009 recorded no payment op
- pass: no payment existed on AAC/2627/000009 right after Present
- pass: AAC/2627/000009 is the one bill of its check
- pass: Present on AAA/2627/000022 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000022 recorded no payment op
- pass: no payment existed on AAA/2627/000022 right after Present
- pass: AAA/2627/000022 is the one bill of its check
- pass: Present on AAC/2627/000010 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000010 recorded no payment op
- pass: no payment existed on AAC/2627/000010 right after Present
- pass: AAC/2627/000010 is the one bill of its check
- pass: Present on AAA/2627/000023 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000023 recorded no payment op
- pass: no payment existed on AAA/2627/000023 right after Present
- pass: AAA/2627/000023 is the one bill of its check
- pass: Present on AAC/2627/000011 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000011 recorded no payment op
- pass: no payment existed on AAC/2627/000011 right after Present
- pass: AAC/2627/000011 is the one bill of its check
- pass: Present on AAA/2627/000024 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000024 recorded no payment op
- pass: no payment existed on AAA/2627/000024 right after Present
- pass: AAA/2627/000024 is the one bill of its check
- pass: Present on AAC/2627/000012 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000012 recorded no payment op
- pass: no payment existed on AAC/2627/000012 right after Present
- pass: AAC/2627/000012 is the one bill of its check
- pass: Present on AAA/2627/000025 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000025 recorded no payment op
- pass: no payment existed on AAA/2627/000025 right after Present
- pass: AAA/2627/000025 is the one bill of its check
- pass: Present on AAC/2627/000013 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000013 recorded no payment op
- pass: no payment existed on AAC/2627/000013 right after Present
- pass: AAC/2627/000013 is the one bill of its check
- pass: Present on AAA/2627/000026 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000026 recorded no payment op
- pass: no payment existed on AAA/2627/000026 right after Present
- pass: AAA/2627/000026 is the one bill of its check
- pass: Present on AAC/2627/000014 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000014 recorded no payment op
- pass: no payment existed on AAC/2627/000014 right after Present
- pass: AAC/2627/000014 is the one bill of its check
- pass: Present on AAC/2627/000015 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000015 recorded no payment op
- pass: no payment existed on AAC/2627/000015 right after Present
- pass: AAC/2627/000015 is the one bill of its check
- pass: Present on AAA/2627/000032 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000032 recorded no payment op
- pass: no payment existed on AAA/2627/000032 right after Present
- pass: AAA/2627/000032 is the one bill of its check
- pass: Present on AAA/2627/000033 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000033 recorded no payment op
- pass: no payment existed on AAA/2627/000033 right after Present
- pass: AAA/2627/000033 is the one bill of its check
- pass: Present on AAA/2627/000034 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000034 recorded no payment op
- pass: no payment existed on AAA/2627/000034 right after Present
- pass: AAA/2627/000034 is the one bill of its check
- pass: Present on AAC/2627/000017 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000017 recorded no payment op
- pass: no payment existed on AAC/2627/000017 right after Present
- pass: AAC/2627/000017 is the one bill of its check
- pass: Present on AAA/2627/000036 recorded exactly one bill.finalize
- pass: Present on AAA/2627/000036 recorded no payment op
- pass: no payment existed on AAA/2627/000036 right after Present
- pass: AAA/2627/000036 is the one bill of its check
- pass: Present on AAC/2627/000018 recorded exactly one bill.finalize
- pass: Present on AAC/2627/000018 recorded no payment op
- pass: no payment existed on AAC/2627/000018 right after Present
- pass: AAC/2627/000018 is the one bill of its check
- pass: Present on AAE/2627/000002 recorded exactly one bill.finalize
- pass: Present on AAE/2627/000002 recorded no payment op
- pass: no payment existed on AAE/2627/000002 right after Present
- pass: AAE/2627/000002 is the one bill of its check
- pass: Present on AAE/2627/000002 recorded exactly one bill.finalize
- pass: Present on AAE/2627/000002 recorded no payment op
- pass: no payment existed on AAE/2627/000002 right after Present
- pass: AAE/2627/000002 is the one bill of its check
- pass: 38 staff-screen total(s) were taken
- pass: taking a total on a staff screen wrote no op and no bill

### T13 Blackout: PASS
- pass: the KDS station lists the ticket handed over by hand and the stale one as late tickets: ["handover","stale"]
- pass: the recent blackout ticket printed once on the station
- pass: no handed-over and no stale KOT printed as an original after power returned
- pass: bills finalised on the battery counter in the blackout keep their series (AAC: gapless per T3) and their original print exists (T6)
- pass: nodes converged 2 rounds (5 s virtual each, 871 ms wall) after the LAN returned: within 60 s
- pass: the paper page typed twice (2 entries) is 1 bill: MB1/2627/000007
- pass: the handover is recorded and listed on the KDS side
- pass: the stale ticket was 30 min old at the station and the recent one 5 min
- pass: the hub's clock ran 960 s ahead and back to 0: 8 devices synced in both states, no error

### X1 No duplicate ops after lost acks: PASS
- pass: A07: 270 own ops, gateway stores 270, contiguous from 1
- pass: A08: 148 own ops, gateway stores 148, contiguous from 1
- pass: A09: 22 own ops, gateway stores 22, contiguous from 1
- pass: B3K: 33 own ops, gateway stores 33, contiguous from 1
- pass: B4K: 24 own ops, gateway stores 24, contiguous from 1
- pass: B5K: 16 own ops, gateway stores 16, contiguous from 1
- pass: K9K: 20 own ops, gateway stores 20, contiguous from 1
- pass: K8K: 9 own ops, gateway stores 9, contiguous from 1
- pass: 1 push answer(s) were lost after the server applied them; 3 hub stop(s) incl. 1 disk wipe; 1 device restart(s)
- pass: 551 outbox events, all distinct ids

### X3 Cloud restore epoch: NOT_RUN
- **fail:** no restore epoch in this seed

### X2 Erasure of a customer's details: PASS
- pass: erasable side tables (sync_pii) holding the values before: billing 1, orders 1
- pass: before erasure: details present in billing false, orders false, device A07 true, gateway op_pii rows 4
- pass: customer details were present somewhere before the erasure (the test is meaningful)
- pass: gateway erasure issued for 1 op(s) (POST /internal/v1/pii/erase as billing)
- pass: billing no longer returns the name or GSTIN
- pass: orders no longer returns the name or GSTIN
- pass: no device holds the details any more
- pass: billing and orders side tables no longer hold the values (1/1 before, 0/0 after)
- pass: device A07 holds 0 pii_erase record(s); ["cloud-genesis"] cloud origins in its vector
- pass: the gateway's op_pii rows of those ops are gone
- pass: the copy on the internal queue ages out within 7 days (ADR-0014 A2.1 item 7): not waited for, stated in the erasure promise

### S1 Payment arrives before its bill: PASS
- pass: the script made a payment on a device other than the bill's issuer
- pass: billing projected the payment while its bill was still absent (arrival order forced)
- pass: tender card: 188250 paise in 14 payment(s) on both sides
- pass: tender cash: 656350 paise in 42 payment(s) on both sides
- pass: tender upi: 522200 paise in 15 payment(s) on both sides
- pass: the bill's payments total 12600 paise on both sides
- pass: the payment's payment_unmatched (1 seen) is cleared now that its bill arrived
- pass: no other payment_unmatched is open (only the quarantined duplicate's, asserted by S5)

### S2 Two tills credit the same bill: PASS
- pass: two tills each credited the same bill
- pass: bill 01a21741 (9500 paise) was credited 19000 paise by 2 notes
- pass: the over-credit is flagged: credit_exceeds_bill
- pass: output tax reduced once: credit tax 4052 paise (expected 4052, tolerance 6)

### S3 Late bill arrives before the day close: PASS
- pass: the late bill(s) were projected before any close arrived (2)
- pass: 2 finalize op(s) of the isolated device are outside the Z
- pass: late bill 01a21741 (op 7) is in exactly one revision
- pass: late bill 01a21741 (op 13) is in exactly one revision
- pass: arrival: {"planned":["A09","A08","A07"],"pushed":[{"device":"A09","ok":true},{"device":"A08","ok":true},{"device":"A07","ok":true}],"lateBeforeClose":{"reached":true,"billIds":["01a21741-83d3-7006-a1d9-0eaccb2ae208","01a21741-863f-7006-9244-987cd61165da"],"dayClosedAtThatTime":false},"paymentFirst":{"reached":true,"billId":"01a21741-5ea2-7006-8033-392d78fe677d","paymentId":"01a21747-c8db-7000-83e5-198780d3dc63"}}; billing day closed, late_document_count 5, revisions 1

### S4 Fast-clock device, Z coverage by vector: PASS
- pass: a bill was issued on the fast-clock device
- pass: the closing device held that bill's op before it closed
- pass: the bill's own time is 7 min after the close time of the closing device
- pass: no Z revision covers the fast-clock bill (the Z holds it, by vector)
- pass: original Z plus revisions equals all bills (1396000)

### S5 Duplicate invoice number under two bill ids: PASS
- pass: the same number was issued under two bill ids
- pass: the gateway refused the second op as duplicate_document (quarantined)
- pass: billing kept the number with the first bill only
- pass: 2 payment(s) were recorded on the second bill
- pass: payment 01a21741 of the quarantined bill carries one open payment_unmatched (warning)
- pass: payment 01a21741 of the quarantined bill carries one open payment_unmatched (warning)

### S6 Override citing a non-existent approval: NOT_RUN
- pass: not waited in this seed (SOAK_APPROVAL_WINDOW_MS=0); at the end of the run the gateway shows the op as accepted [approval_pending]

### S7 Re-bill of a paid bill: cashier refused, manager with another approver's approval accepted: PASS
- pass: a cashier attempted a re-bill of a paid bill without approval
- pass: the bill carried 2 payment(s)
- pass: the cashier was refused on the device (approval_needed)
- pass: a manager re-billed it citing Anand's approval
- pass: the approval came from another approver than the requester (Anand for Meena)
- pass: billing holds the credit note and 1 replacement bill(s) of the re-bill
- pass: billing raised no rebill_after_payment for the approved re-bill

## Run facts
- Script steps: 115 run, 0 failed: []
- Hub stops 3 (disk wipes 1), device restarts ['B4K'], lost push answers 1, approvals 2, bills by device {"A07": 33, "A08": 18, "A09": 4}
- Offline Z: {"zNumber": "Z/2026-11-26/AAA", "totalPaise": 1363100, "billCount": 55}; second close on the partitioned counter: {"zNumber": "Z/2026-11-26/AAC", "totalPaise": 1363100, "billCount": 55}
- Gateway ops by outcome: [{"cloud_origin": false, "status": "accepted", "reason": "", "n": 546}, {"cloud_origin": false, "status": "quarantined", "reason": "duplicate_document", "n": 1}, {"cloud_origin": true, "status": "accepted", "reason": "", "n": 8}]; parked events billing/orders: {"billing": "0", "orders": "0"}
- Billing exceptions: ["late_document/info/open/business_day", "credit_exceeds_bill/high/open/bill", "line_on_two_bills/high/open/bill", "two_bills_one_check/warning/open/check", "payments_exceed_total/high/resolved/bill", "late_document/info/open/payment", "late_document/info/open/bill", "late_document/info/open/bill", "late_document/info/open/payment", "late_document/info/open/payment", "manual_entry/warning/open/bill", "payment_unmatched/warning/resolved/payment", "payment_unmatched/warning/resolved/payment", "payment_unmatched/warning/open/payment", "payment_unmatched/warning/open/payment", "tax_mismatch/high/open/bill"]; orders exceptions: {"late_ticket/open": 1, "unresolved_line_at_close/resolved": 45}

## Timeline
- 09:42:29Z cast up: 3 counters, 3 captains, 2 KDS, one hub; morning: internet up for the configuration push
- 09:43:53Z internet cut for the whole trading day
- 09:43:53Z configuration published: 7/7 device intents, 3/3 cloud emits
- 09:48:13Z BLACKOUT: LAN, hub and the A07 counter are off
- 09:48:13Z POWER RETURNS
- 09:48:18Z A09 isolated; A07 and A08 partitioned from each other for the close
- 09:48:19Z wrong-tax bill AAE/2627/000001 recorded; its print failed under the harness's doctored view (not_found: bill 01a21741-83d3-7006-a1d9-0eaccb2ae208); printing it now

Findings drawn from this and the other runs: `docs/execution/phase-2/exit-checks.md (Run 6)`.

## Artifacts

- `artifacts/vitest.out`
- `artifacts/gw.log`
- `artifacts/r4.hub.log`
- `artifacts/onboard.sanitised.json`
- `artifacts/provision.log`
- `artifacts/run.result.json`
