# EDI Failiures XML — working instructions

This folder is where EDI XML transaction failures get diagnosed and
corrected, one failure at a time. Follow this exact process every time.

## Workflow

1. **User provides the failure.** In chat, the user pastes the EDI error
   message and places the source file in this folder. That file is the
   "source file" — the original, broken document as it came from the system.
2. **Check for a matching case first.** Before diagnosing from scratch, check
   `SKILL.md`'s Quick index (and the relevant `cases/<slug>-<doctype>.md`
   file) for a case on the same customer + document type + error type, or
   the same error type on a different customer/doc type — see `SKILL.md`'s
   own matching-order rules. If a similar case is already logged, use it as
   the starting point.
4. **Diagnose together.** Read the error message against the source file and
   discuss what's actually wrong before touching anything. Do not jump
   straight to editing.
5. **Never edit the source file.** The original file placed in this folder is
   permanent and untouched — it's the record of what the failure looked like.
6. **Create a version 2 file.** Once the fix is agreed, create a corrected
   copy in this same folder named `<original-filename>_v2.<ext>` (e.g.
   `PO4521.xml` → `PO4521_v2.xml`). All corrections go into this new file.
7. **Update the skill — automatically, no need to ask first.** After each
   fix, append a case — error signature, root cause, and what changed — to
   the correct file inside the `edi-xml-failure-corrections` skill
   **package** (see below), and add the Quick index row. Do this as a normal
   part of finishing the fix, without pausing to confirm it first — it's
   expected every time, not a judgment call. This is one growing skill, not
   a new skill per error type or trading partner, so it accumulates every
   pattern we've solved and future failures can be matched against past
   ones.

## Skill package structure

At volume (hundreds/thousands of invoices) a single `SKILL.md` holding every
case becomes too large to load or scan efficiently. To prevent that,
`edi-xml-failure-corrections` is a **package**, not one file:

- `.claude/skills/edi-xml-failure-corrections/SKILL.md` — stays small on
  purpose: workflow, error-type vocabulary, matching rules, cross-cutting
  notes, and a **Quick index** table (one row per case: customer, doc type,
  error type, status, link). Never write full case detail here.
- `.claude/skills/edi-xml-failure-corrections/cases/<customer-slug>-<doctype>.md`
  — one file per customer **and document type** (e.g.
  `home-hardware-colonial-810.md`), holding that combination's full case
  entries and partner-specific notes. Split by doc type from the first case
  for that customer, not just at volume — different doc types rarely share
  fields or error types anyway. Slug = the customer name as it appears in the
  invoice filename, lowercased, hyphenated (e.g. `Home Depot.CA Hub` →
  `home-depot-ca-hub`). Create the file the first time that customer+doctype
  combination gets a case.
- When a single customer+doctype file passes **~50 cases**, split it further
  — by year is the default (`<slug>-810-2027.md`) — and update the Quick
  index links. Do this proactively at the threshold, not once the file is
  already unwieldy.
- Cross-cutting rules that aren't specific to one customer go in `SKILL.md`'s
  General notes section, not in a customer file.
- After every fix: append the case to the relevant `cases/<slug>-<doctype>.md`,
  then add one row to `SKILL.md`'s Quick index pointing at it. Both steps,
  every time — a case that's only in one place is effectively lost.
- See `cases/README.md` inside the skill folder for the full layout and the
  reasoning behind sharding by customer.

## Git workflow

Repo: [github.com/srihari-renin/edi-failures-xml](https://github.com/srihari-renin/edi-failures-xml)
(private). `master` is the backup / source of truth — it should always
reflect only resolved (or explicitly logged) case state, so it stays safe to
clone or restore from at any time.

1. **Never commit case work directly to `master`** — even solo. Before
   starting a new failure, branch off an up-to-date `master`:
   ```
   git checkout master && git pull && git checkout -b case/<customer-slug>/<reference-id>
   ```
   `<customer-slug>` matches the `cases/<slug>.md` naming from the skill
   package above; `<reference-id>` is the invoice/PO/control number in
   lowercase (e.g. `case/home-hardware-colonial/psi1312154`). For work that
   isn't tied to one invoice (skill or process changes), use
   `chore/<short-description>` instead (e.g. `chore/branching-conventions`).
2. Do all of that case's work on its branch: the source file, every `_v2` /
   `_v3` / ... attempt, the case entry in `cases/<slug>.md`, and the Quick
   index row in `SKILL.md`. Commit as you go.
3. When the case reaches a stopping point (resolved, or a failed attempt
   that's fully logged), push the branch and open a PR into `master`:
   ```
   git push -u origin case/<customer-slug>/<reference-id>
   gh pr create --fill
   ```
   Self-merge is fine — no required review for this project — but always via
   a PR, not a local `git merge`, so GitHub keeps a record of what changed
   per case.
4. Merge, then delete the branch (`gh pr merge --squash --delete-branch` or
   the equivalent from the GitHub UI). Squash so `master` gets one clean
   commit per resolved case.
5. **Working two cases at once** (you and someone else, or two parallel
   Claude sessions): each just branches from `master` and works
   independently — no coordination needed until merge time. The only file
   likely to conflict is `SKILL.md`'s Quick index table, since every case
   adds a row there; if both branches add a row in the same place, Git will
   flag a merge conflict — resolve it by keeping both new rows, nothing more
   is needed.
6. **Running several cases in parallel** (e.g. `git worktree add` off
   different branches so more than one session can work at once without
   switching branches in the same checkout): each worktree still follows
   steps 1–5 independently on its own `case/`/`chore/` branch. Use the task
   tracking files below so it's visible what every worktree is doing.

## Task tracking files

Three files at the project root, separate from the skill package, track
work-in-progress and knowledge that doesn't belong in a case entry:

- **[`task-list.md`](task-list.md)** — open tasks only, one row per task
  currently being worked (across any worktree/session). Add a row when you
  start something. When it's done, **move the row** into
  `Resolved/task-list-completed.md` — cut from one file, paste into the
  other, don't leave it in both and don't leave finished work in the open
  file. This is a live coordination board, not a history — keeping it short
  is the point.
- **[`Resolved/task-list-completed.md`](Resolved/task-list-completed.md)** —
  archive of finished tasks, moved here from `task-list.md`. Newest first.
- **[`gaps.md`](gaps.md)** — caveats, known limitations, and errors that
  couldn't be fully resolved, logged for future reference. Not a task list —
  nothing here necessarily needs action. Append whenever an error can't be
  fully resolved and you need to record why, or you notice a limitation
  that isn't specific enough to one case to belong in that case's
  `cases/<slug>-<doctype>.md` file.
- **[`open-questions.md`](open-questions.md)** — any decision question
  asked in chat that doesn't get answered in the same turn. Log it here
  before moving on to other work so it isn't lost; when picking work back
  up, check this file and re-ask anything still pending. Remove or mark
  answered once resolved.

`Resolved/` isn't only for the completed task list — once a case's
corrected invoice is confirmed **sent to the partner** (not just fixed
locally), move that case's source file and every `_v2`/`_v3`/... attempt
into `Resolved/`, together with any reference file used only for that case's
diagnosis. This keeps the working folder showing only invoices still in
play. Moving doesn't count as editing — the "originals are never modified"
rule (below) is about content, not location — but update the case's
**Files** line in `cases/<slug>-<doctype>.md` to point at the new
`Resolved/...` path so the record stays accurate.

## Rules

- One skill only: `edi-xml-failure-corrections`. Every corrected failure adds
  a case to it — don't create a separate skill per error type or partner.
- File naming for corrections is always `_v2` suffix, same folder as the
  original, same extension. No other naming pattern.
- Originals are never modified or deleted. If a fix doesn't fully resolve the
  error, keep incrementing: `_v2`, `_v3`, `_v4`, etc., in the same folder,
  never editing a prior version — each attempt is its own file until the
  error is actually resolved.
- Every new version gets its own Case Log entry in the
  `edi-xml-failure-corrections` skill, not just the first attempt — including
  what was tried, whether it worked, and if not, why, so failed attempts are
  as visible as successful ones.
- Logging a case and updating the skill package is not a decision to check in
  on — do it automatically every time a fix (or a "no fix needed" outcome) is
  reached. Before diagnosing a new failure, check the Quick index / relevant
  `cases/<slug>-<doctype>.md` file for a matching case first and reuse it as
  the starting point instead of re-diagnosing from scratch.
