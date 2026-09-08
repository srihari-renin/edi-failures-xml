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

### TimbrMart PSI1323681 — do you have a QC-with-freight reference invoice? — asked 2026-09-08
**Question:** Reference-first step 3. Closest known-good documents in the project are the Canac Quebec invoices (PSI1310799, PSI1301959, PSI1310963) — right province, right ERP, tax correct, but **no freight charge on any of them** and a different partner's mapping (`SP` vs `PS` for QST). Is there a **Home Care TimbrMart** invoice, shipped to a **QC** dealer, with a **freight charge on the document**, that went through cleanly?
**Why it matters / what's blocked:** It would settle all three questions below as observed fact instead of inference — whether freight sits inside the tax base, whether `TermsDiscountAmount` is computed on the tax-inclusive total, and whether `PS` is TimbrMart's correct QST code. The three questions below are **deferred behind this one** and should only be asked if the answer is "no such invoice exists."
**Answered 2026-09-08:** Sri Hari confirmed such an invoice exists and is dropping it into the project root. Awaiting the file; the three questions below stay deferred until it has been read.

### TimbrMart PSI1323681 — is the $150 freight charge inside the GST/QST base? — asked 2026-09-08
**Question:** Invoice PSI1323681 (Home Care TimbrMart, 810, Quebec ship-to 8262) carries a header `ChargesAllowances` record with `AllowChrgIndicator` = `C`, `AllowChrgCode` = `D240` (Freight), `AllowChrgAmt` = 150.00. Does GST/QST apply to net sales + freight (397.42) or to net sales only (247.42)?
**Why it matters / what's blocked:** It sets every number written into `_v2` — tax 19.87 + 39.64 and `TotalAmount` 456.93 on the freight-taxable reading, versus 12.37 + 24.68 and 434.47 on the goods-only reading. `_v2` cannot be written until this is settled.

### TimbrMart PSI1323681 — recompute `TermsDiscountAmount` once tax is added? — asked 2026-09-08
**Question:** Source `TermsDiscountAmount` is 7.95 = 2% x 397.42, i.e. 2% of the (currently tax-free) `TotalAmount`. Canac's Quebec reference PSI1310799 computes the same field as 2% of the **tax-inclusive** total. Recompute to 9.14 (2% x 456.93) or leave 7.95 untouched as with Home Hardware Case 01?
**Why it matters / what's blocked:** One field in `_v2`. It does not affect `TotalAmount`, but it is what TimberMart would short-pay against if they take the discount.

### TimbrMart PSI1323681 — is `TaxTypeCode` `PS` the right QST code for this partner? — asked 2026-09-08
**Question:** This invoice emits `GS` + `PS`. Canac (also Quebec, same ERP, different partner mapping) emits `GS` + `SP`; TimbrMart's own Ontario credit memo PSCM032060 emitted `OH`. Left as `PS` per the skill's "fix only the evidenced defect" rule — but if LBMX rejects again on the code rather than the amount, `SP` is the next thing to try.
**Why it matters / what's blocked:** Nothing blocked; noted so a second rejection is diagnosed in one step rather than re-derived.

## Answered

_(Empty.)_
