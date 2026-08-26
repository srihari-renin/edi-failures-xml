# Completed Tasks

Archive of finished work, moved here from [`task-list.md`](../task-list.md)
so the open list stays short enough to scan. Newest first.

| Task | Branch | Started | Completed | Notes |
|---|---|---|---|---|
| True Value PO 08062302W2700 carrier fields — ASN SS1326195 (856), `missing-segment` | `claude/true-value-error-validation-b67e71` | 2026-08-26 | 2026-08-26 | `_v2` fix applied (CarrierAlphaCode/CarrierRouting = ODFL / Old Dominion Freight Line, confirmed by user). Sent to True Value and confirmed; files moved to `Resolved/`. See [cases/true-value-856.md](../.claude/skills/edi-xml-failure-corrections/cases/true-value-856.md) Case 01. |
| Home Depot Canada 810 price-mismatch, invoices PSI1320440 & PSI1320586 | `claude/home-depot-price-discrepancies-1c0f5c` | 2026-08-25 | 2026-08-25 | Caught before either original was sent. Corrected `_v2` sent to Home Depot instead and confirmed by user; files moved to `Resolved/`. See [cases/home-depot-canada-810.md](../.claude/skills/edi-xml-failure-corrections/cases/home-depot-canada-810.md) Case 01 & 02. |
| TimberMart 810CM tax sign fix, invoice PSCM032060 | `case/home-care-timbrmart/pscm032060` | 2026-08-21 | 2026-08-21 | `TaxAmount` → `-23.28` in `_v2`. Sent to TimberMart and confirmed; files moved to `Resolved/`. |
