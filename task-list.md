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
| Home Hardware Colonial PSCM031734 (PO 81901) — credit memo out of balance by -120.92, `credit-sign-convention` (810 Credit Memo) | `claude/renin-invoice-balance-a8fd3b` | 2026-09-14 | `_v2` applied (`AllowChrgAmt` `-60.46` → `60.46`, indicator stays `C`), settled by archived reference PSCM014940. Awaiting resend to Home Hardware via SPS and partner confirmation before moving to `Resolved/`. NAV→SPS map root cause logged in `gaps.md`. |
| DIB PO DIB300124 carrier fields — ASN SS1326194 (856) and invoice PSI1318473 (810), both `missing-segment` | `claude/dib-asn-invoice-errors-fe4453` | 2026-08-26 | Both `_v2` fixes applied (CarrierAlphaCode/CarrierRouting = ODFL / Old Dominion). Awaiting resend to DIB and partner confirmation before moving to Resolved/. |
