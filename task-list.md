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
| Confirm resend/credit channel for corrected Home Depot Canada invoices PSI1320440 & PSI1320586, then send | claude/home-depot-price-discrepancies-1c0f5c | 2026-08-25 | Both `_v2` invoices are corrected (see [cases/home-depot-canada-810.md](.claude/skills/edi-xml-failure-corrections/cases/home-depot-canada-810.md) Case 01 & 02) but neither original was rejected by Rithum, so a plain resend may bounce as `duplicate-document`. Need to confirm the right channel (credit/debit vs. resend) before transmitting; move source files to `Resolved/` once sent. |
