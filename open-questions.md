# Open Questions

Questions asked of Sri Hari that didn't get answered right away — logged
here so they don't get lost when he jumps to a different task. Whenever you
ask a decision question and don't get an answer in the same turn, add it
here before moving on. When picking work back up, check this file first and
re-ask anything still pending.

When a question gets answered: either delete its entry, or move it under
**Answered** with a one-line note of the answer — whichever fits.

## Pending

<!--
Entry template:

### <short question title> — asked <YYYY-MM-DD>
**Question:** <the actual question, or enough of it to re-ask cold>
**Why it matters / what's blocked:** <what can't proceed until this is answered>
-->

_(Empty — fills in as questions come up.)_

## Answered

### Orgill PSI1297591 — does BuyerPartNumber requirement disappear once PO isn't 8 characters? — answered 2026-09-15
Yes. `_v2` (PO `125005-7176`, 11 characters) was sent as-is, with `BuyerPartNumber` still absent
on `LineSequenceNumber` 2 and 3, and Orgill accepted it. Confirms the requirement is conditional
on an 8-character PO, not unconditional — logged as a CONFIRMED Partner note in
`cases/orgill-us-stores-810.md` for future Orgill cases with an 8-character original PO.

### TimbrMart PSI1323681 (810, `tax-missing`) — all four questions settled 2026-09-08
Reference invoice **PSI1247205** (`3892509_Home Care TimbrMart 810 - Reference.xml`) — same
customer, same doc type, QC ship-to, same $150.00 `D240` freight charge — answered every one
to the cent, so none needed adjudicating:
- **QC-with-freight reference exists?** Yes; Sri Hari supplied it.
- **Freight inside the tax base?** Yes. Base = net + charge (266.38 + 150.00 = 416.38 gives the
  issued GST 20.82 / QST 41.53 / total 478.73; net alone gives 13.32 / 26.57).
- **Recompute `TermsDiscountAmount`?** No — TimbrMart's basis is the **pre-tax** total
  (2% × 416.38 = 8.33 as issued; tax-inclusive would be 9.57). The failing invoice's 7.95 was
  already correct. Note this is the *opposite* of Canac; logged as a General note in `SKILL.md`.
- **Is `PS` the right QST code?** Yes — TimbrMart's own accepted invoice carries `PS`. Canac's
  `SP` is a different partner's mapping and was a red herring.

Still open, but not a question for this case log: the **NAV root cause on ship-to 8262** and the
blast-radius check for other Quebec ship-tos. Tracked on the case entry.

