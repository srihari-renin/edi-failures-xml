# Open Tasks

Live coordination board across parallel worktrees/sessions. When you start
work on something — a case, a chore, an investigation — add a row here.
When it's done, **cut the row and paste it into
[`Resolved/task-list-completed.md`](Resolved/task-list-completed.md)** —
don't leave finished tasks in this file. Keeping this short is the entire
point: it's what lets someone (or another session) see everything currently
in flight, across every worktree, at a glance.

Multiple worktrees may edit this file at the same time. If two branches
both add a row and that produces a merge conflict on push, resolve it by
keeping both new rows — same rule as the Quick index table in the skill.

| Task | Branch | Started | Notes |
|---|---|---|---|
| DIB PO DIB300124 carrier fields — ASN SS1326194 (856) and invoice PSI1318473 (810), both `missing-segment` | `claude/dib-asn-invoice-errors-fe4453` | 2026-08-26 | Both `_v2` fixes applied (CarrierAlphaCode/CarrierRouting = ODFL / Old Dominion). Awaiting resend to DIB and partner confirmation before moving to Resolved/. |
| Home Care TimbrMart QC invoice PSI1323681 (810) — `tax-missing` (GST + QST both 0.00 on a Quebec ship-to) | `claude/quebec-qst-tax-validation-603f21` | 2026-09-08 | `_v2` written and logged (Case 01): GST 19.87, QST 39.64, TotalAmount 456.93, all confirmed against reference PSI1247205. Awaiting resend to TIM-BR-MART **and** the NAV check on ship-to 8262 (Tax Area Code / Tax Liable) plus a blast-radius check on other QC ship-tos. |
