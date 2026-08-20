# EDI Failiures XML — working instructions

This folder is where EDI XML transaction failures get diagnosed and
corrected, one failure at a time. Follow this exact process every time.

## Workflow

1. **User provides the failure.** In chat, the user pastes the EDI error
   message and places the source file in this folder. That file is the
   "source file" — the original, broken document as it came from the system.
2. **Diagnose together.** Read the error message against the source file and
   discuss what's actually wrong before touching anything. Do not jump
   straight to editing.
3. **Never edit the source file.** The original file placed in this folder is
   permanent and untouched — it's the record of what the failure looked like.
4. **Create a version 2 file.** Once the fix is agreed, create a corrected
   copy in this same folder named `<original-filename>_v2.<ext>` (e.g.
   `PO4521.xml` → `PO4521_v2.xml`). All corrections go into this new file.
5. **Update the skill.** After each fix, append a case — error signature,
   root cause, and what changed — to the correct file inside the
   `edi-xml-failure-corrections` skill **package** (see below). This is one
   growing skill, not a new skill per error type or trading partner, so it
   accumulates every pattern we've solved and future failures can be matched
   against past ones.

## Skill package structure

At volume (hundreds/thousands of invoices) a single `SKILL.md` holding every
case becomes too large to load or scan efficiently. To prevent that,
`edi-xml-failure-corrections` is a **package**, not one file:

- `.claude/skills/edi-xml-failure-corrections/SKILL.md` — stays small on
  purpose: workflow, error-type vocabulary, matching rules, cross-cutting
  notes, and a **Quick index** table (one row per case: customer, doc type,
  error type, status, link). Never write full case detail here.
- `.claude/skills/edi-xml-failure-corrections/cases/<customer-slug>.md` —
  one file per customer/trading partner, holding that customer's full case
  entries and partner-specific notes. Slug = the customer name as it appears
  in the invoice filename, lowercased, hyphenated (e.g. `Home Depot.CA Hub`
  → `home-depot-ca-hub`). Create the file the first time that customer gets
  a case.
- When a single customer file passes **~50 cases**, split it further by
  document type (`<slug>-810.md`, `<slug>-856.md`, ...) and update the
  Quick index links. Do this proactively at the threshold, not once the
  file is already unwieldy.
- Cross-cutting rules that aren't specific to one customer go in `SKILL.md`'s
  General notes section, not in a customer file.
- After every fix: append the case to the relevant `cases/<slug>.md`, then
  add one row to `SKILL.md`'s Quick index pointing at it. Both steps, every
  time — a case that's only in one place is effectively lost.
- See `cases/README.md` inside the skill folder for the full layout and the
  reasoning behind sharding by customer.

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
