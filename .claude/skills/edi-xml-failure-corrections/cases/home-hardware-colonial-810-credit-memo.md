# Home Hardware Colonial (`833ALLRENINHOLD`) — 810 Credit Memo — Case Log

Covers 810 Credit Memo (`InvoiceTypeCode=CR`) documents only. Regular 810
invoices for this customer are in `cases/home-hardware-colonial-810.md` —
see `cases/README.md`.

## Partner notes

- **Home Hardware CR balance rule (settled by reference PSCM014940):** Home
  Hardware negates the *lines* and the *taxes* of a credit memo itself, and
  takes every `ChargesAllowances/AllowChrgAmt` **exactly as sent, sign
  included**. Its balance check is
  `−Σlines − ΣA + ΣC − Σtax = −TotalAmount`. Checked to the cent on the
  accepted PSCM014940 (`−0 − 36.25 + 1.09 − 4.57 = −39.73`) and on the
  rejection figures for PSCM031734 (`−241.87 + (−60.46) − 9.07 − 18.10 =
  −329.50`, the exact "Calculated total" in the rejection).
  Practical upshot — **everything on a Home Hardware credit memo is sent
  positive**: `UnitPrice`, `TaxAmount`, `TotalAmount`, and every
  `AllowChrgAmt`. Direction lives in the indicator only: `A` = money credited
  to the customer, `C` = a deduction from the credit (restocking charge,
  unapplied discount, charge-back). A **negative `AllowChrgAmt` is always
  wrong** for this partner and throws the balance off by exactly 2× the
  amount. — [Case 01](#case-01--810-credit-memo--credit-sign-convention--2026-09-14)
- **This is the opposite of TimberMart** (`cases/home-care-timbrmart-810-credit-memo.md`),
  which does *not* negate tax and therefore needs `TaxAmount` sent negative.
  Never carry a credit-memo sign convention across partners — each one is
  its own rule until a reference from that partner proves it.
- **Known-good credit memo shapes for this partner** live in
  `I:\EDI Info\Moved to Sharepoint\EDI Info XML Archives\`:
  `2027841_Home Hardware Colonial 810 Credit Memo To Compare.xml`
  (PSCM014940 — `A` allowance + `C` charge, ON HST; copied into the project
  as `2027841_Home Hardware Colonial 810 Credit Memo - Reference.xml`),
  `2065107_… To Compare.xml` (PSCM018133 — single `A`, AB GST) and
  `2065535_… To Compare - Stores.xml` (PSCM018724 — `A`, QC GST+QST with
  `SP`). All three carry the credit body as a positive `A` allowance with
  `TotalNetSalesAmount` 0.00; PSCM014940 also shows the four-attempt history
  (`v1`–`v4`) that produced that shape.
- **`D240` on a restocking charge:** NAV maps G/L 42010 "Shipping and
  Allowances" to `AllowChrgCode` `D240` (Freight) regardless of the return
  reason. Home Hardware's rejection was balance-only and said nothing about
  the code, and the accepted PSCM014940 used `ZZZZ` — so the code is not known
  to be validated. Left alone; revisit only if Home Hardware ever rejects on it.

## Cases

Newest first.

<!--
Entry template — copy this block for every new case AND every new version:

### Case NN — <doc type> — <error type> — <YYYY-MM-DD>
- **Document type:** <810 / 850 / 856 / …>
- **Error type:** <value from the vocabulary in the root SKILL.md>
- **Reported by:** <person / system / portal, and how it arrived>
- **Error message:** <verbatim text of the complaint or rejection>
- **Source file:** <original filename — kept untouched>
- **Reference file:** <known-good filename used for comparison, or n/a>
- **Reference ID:** <invoice no. / PO no. / control no. that identifies this failure>
- **Document defect:** <what is factually wrong in the source XML>
- **Upstream root cause:** <what in the ERP / master data produced it, or "unconfirmed">
- **Fix:** <exactly which elements changed, old → new>
- **Files:** <original.xml> → <original_v2.xml>
- **Status:** <resolved / resent, awaiting confirmation / failed — superseded by _v3>
-->

### Case 01 — 810 Credit Memo — `credit-sign-convention` — 2026-09-14
- **Document type:** 810 Credit Memo (`InvoiceTypeCode=CR`)
- **Error type:** `credit-sign-convention`
- **Reported by:** Home Hardware AP (EDI ID 9056699335, AP number 9056699335500106800), by email, forwarded by Sri Hari in chat
- **Error message:** "Invoice out of balance. - Please correct and resend the invoice. Invoice #: PSCM031734 Reference #: 81901. Out of balance by -120.92. Line total: -241.87. GST/HST: -9.07. QST/PST: -18.10. Allowances/Charges: -60.46. Calculated total: -329.50. Invoice total: -208.58"
- **Source file:** `4075401_Home Hardware Colonial 810 Credit Memo.xml` (credit PSCM031734, PO 81901, ship-to store 2575-9 Roberge et Fils, La Sarre QC) — untouched. Arrived in `I:\EDI Info\` rather than the project root; copied in.
- **Reference file:** `2027841_Home Hardware Colonial 810 Credit Memo - Reference.xml` (PSCM014940, PO 23-459129, store 1321-5 Timmins ON) — same customer, same doc type, the final `v4` of a 2022 credit that Home Hardware accepted, kept in the SharePoint archive as the "To Compare" copy. Carries both an `A` allowance (36.25) and a `C` charge (1.09), which is exactly what was needed to see how the partner signs a deduction on a credit. Found by searching `I:\EDI Info` after the project and the user's own files turned up nothing.
- **Reference ID:** PSCM031734 (PO 81901); NAV credit memo against invoice PSI1273707 / shipment SS1281605
- **Document defect:** `Header/ChargesAllowances/AllowChrgAmt` = `-60.46` (indicator `C`, code `D240`). Home Hardware negates lines and taxes itself but passes the charge amount through with its sign, so the negative charge landed as `−60.46` when it needed to land as `+60.46` — out by exactly 2 × 60.46 = 120.92. Every other figure was already right: 19 PR × 12.73 = 241.87; restocking deduction 60.46 gives base 181.41; GST 5% = 9.07; QST 9.975% = 18.10; `TotalAmount` 208.58.
- **What the reference settled vs. what was inferred:** the reference settled the whole fix — a positive `C` amount is how Home Hardware represents a deduction from a credit (its `C`/1.09 "Unapplied discount" reduced that credit the same way the restocking charge reduces this one), and the derived balance rule reproduces both the accepted document and the rejection figures to the cent. Nothing inferred. `A`/`-60.46` would also have balanced arithmetically but no accepted Home Hardware document has ever carried a negative amount, and X12 SAC05 is unsigned by definition, so it was not used.
- **Upstream root cause:** NAV credit memo PSCM031734 has the restocking charge as a G/L line (42010 "Shipping and Allowances", return reason B7 Restocking Charge) with amount **−60.46**, and the NAV→SPS 810 map copies the line's signed amount straight into `AllowChrgAmt`. Any Home Hardware credit memo with a negative G/L line (restocking, charge-back, unapplied discount) will fail the same way until the map emits `abs(amount)` and flips the indicator on sign. Not fixed at source in this session — logged in `gaps.md`.
- **Blast radius check:** not run — no check yet for other open Home Hardware credit memos carrying a negative charge line. Recommend a quick look at recent PSCM* documents to 833ALLRENINHOLD before assuming this is isolated.
- **Fix (applied in _v2):**
  - `Header/ChargesAllowances/AllowChrgAmt`: `-60.46` → `60.46`
  - `AllowChrgIndicator` stays `C`; `AllowChrgCode` stays `D240` (see Partner notes); nothing else changed. One line differs between the two files.
  - Balance under Home Hardware's rule: `−241.87 + 60.46 − 9.07 − 18.10 = −208.58` = −`TotalAmount` ✓
- **Not changed, noted only:** NAV's ship-to on the credit memo is Renin Mississauga (return-to address) while the XML `ST` address is the La Sarre store — Home Hardware did not object, and the error doesn't implicate it. `TermsNetDueDate` 20260423 predates `InvoiceDate` 20260513 — same reasoning.
- **Files:** `4075401_Home Hardware Colonial 810 Credit Memo.xml` → `4075401_Home Hardware Colonial 810 Credit Memo_v2.xml`; reference `2027841_Home Hardware Colonial 810 Credit Memo - Reference.xml`
- **Status:** `_v2` created, awaiting resend to Home Hardware via SPS and partner confirmation. Move all three files to `Resolved/` once confirmed.
