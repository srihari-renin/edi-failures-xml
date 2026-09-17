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
| Home Depot.CA Hub PO 538826239 — invoice PSI1327965 (810) and ASN SS1335829 (856), both `price-mismatch` | `case/home-depot-ca-hub/psi1327965` | 2026-09-17 | `UnitPrice` 171.20 → 172.20 per the 850; 810 derived figures (C300, GST, QST, total, discount) settled by accepted reference PSI1303944. Both `_v2` written; awaiting resend to Home Depot and confirmation before moving to `Resolved/`. NAV price source on SO1333498 unconfirmed. |
| DIB PO DIB300124 carrier fields — ASN SS1326194 (856) and invoice PSI1318473 (810), both `missing-segment` | `claude/dib-asn-invoice-errors-fe4453` | 2026-08-26 | Both `_v2` fixes applied (CarrierAlphaCode/CarrierRouting = ODFL / Old Dominion). Awaiting resend to DIB and partner confirmation before moving to Resolved/. |
