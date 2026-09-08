# Home Care TimbrMart (`5V9ALLRENINHOLD`) — 810 — Case Log

Covers regular 810 sales invoices (`InvoiceTypeCode=10`). Credit memos
(`InvoiceTypeCode=CR`) live in `cases/home-care-timbrmart-810-credit-memo.md`
— the sign convention documented there does **not** apply to this file.

## Partner notes

All of the following were confirmed to the cent against reference invoice
**PSI1247205** (ship-to 8089, Saint-Victor-de-Beauce QC, freight on the
document, accepted by LBMX) — not inferred from another customer.

- **Quebec tax structure:** two `Summary/Tax` records — `GS` at 5% (GST) and
  **`PS`** at 9.975% (QST). Both carry `TaxID` `1225162693TQ0001` (the QST
  registration), including the GST record. Do **not** "correct" the QST code
  to `SP` — Canac uses `SP` for QST, but that is a different partner's
  mapping and `PS` is what TimbrMart's own accepted invoices carry.
- **Freight is inside the tax base.** A header `ChargesAllowances` with
  `AllowChrgIndicator` = `C`, `AllowChrgCode` `D240` ("Freight") is taxed:
  base = `TotalNetSalesAmount` + the charge. On PSI1247205, base 416.38 =
  266.38 + 150.00 gives GST 20.82 and QST 41.53 exactly as issued; taxing net
  alone would have given 13.32 / 26.57. This is the only partner in the
  project so far with a real charge (`C`) rather than an allowance (`A`).
- **`TaxPercent` is the *effective* rate, not the statutory one** —
  `round(TaxAmount ÷ base × 100, 2)`. QST therefore prints as `9.97` on most
  invoices even though the rate is 9.975%; it prints `9.98` when the rounding
  happens to land there. Write the rate you actually computed, don't hardcode.
- **`TermsDiscountAmount` is 2% of the PRE-TAX total** (net + charges), not
  the tax-inclusive total. PSI1247205: 2% × 416.38 = 8.33 as issued; 2% of the
  tax-inclusive 478.73 would have been 9.57. **This is the opposite of Canac**,
  where the same field is 2% of the tax-inclusive total — see the General
  notes in `SKILL.md`. A practical consequence: on an invoice where tax is
  missing, `TermsDiscountAmount` is usually still *correct*, because it never
  depended on the tax. Check before recomputing it.
- **`TotalWeight` of `0.00` is normal for this partner** — the accepted
  reference invoice carries it too. Not a defect, don't chase it.
- **`TotalNetSalesAmount` is net of line-level allowances.** PSI1247205 has a
  line `C310` "Discount" allowance of 360.40 against a 626.78 unit price, and
  `TotalNetSalesAmount` is the resulting 266.38.

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

### Case 01 — 810 — `tax-missing` — 2026-09-08
- **Document type:** 810 (`InvoiceTypeCode=10`)
- **Error type:** `tax-missing`
- **Reported by:** TIM-BR-MART, by email via LBMX, after their system deleted the invoice
- **Error message:** "The attached invoice has the following warnings: 1. QST is not charged on this invoice. 2. There is no tax charged on the attached invoice. All Quebec dealers must be charged QST. This invoice has been deleted from our system. Please resubmit the invoice with QST."
- **Source file:** `4166843_Home Care TimbrMart 810.xml` (invoice PSI1323681, PO 19711, BOL SS1331495) — untouched
- **Reference file:** `3892509_Home Care TimbrMart 810 - Reference.xml` (invoice PSI1247205, ship-to 8089 Saint-Victor-de-Beauce QC) — supplied by Sri Hari on request. Same customer, same doc type, same province, **and the same $150.00 `D240` freight charge with `AllowChrgIndicator` = `C`** — the attribute that mattered, and the one no other document in the project had.
- **Reference ID:** PSI1323681 (PO 19711, ship-to 8262 YVON DUCHESNE & FILS INC, St-Urbain-de-Charlevoix QC)
- **Document defect:** both `Summary/Tax` records were present and structurally correct — right codes (`GS`, `PS`), right `TaxID` — but `TaxAmount` and `TaxPercent` were `0.00` on each, and `TotalAmount` (397.42) equalled net + freight with no tax added. So the mapping emitted the right shape for a Quebec ship-to and the ERP simply calculated zero.
- **Upstream root cause:** **unconfirmed at time of fix.** Strongly suspected to be the same NAV defect as Home Hardware Case 01 — blank **Tax Area Code** / **Tax Liable** on the ship-to record (8262). Sri Hari asked to check ship-to 8262 in NAV and to run a blast-radius check for other invoices to 8262 and to other Quebec ship-tos added around the same time. Until that is confirmed, the corrected XML resends this one invoice but does not stop recurrence, and the NAV invoice behind it is still posted with zero tax — so the GL and the QST return understate until that document is corrected too.
- **How the reference settled it (all exact to the cent, no inference):**
  - Freight in the tax base — base 266.38 + 150.00 = 416.38 → GST 20.82 ✓, QST 41.53 ✓, total 478.73 ✓. Taxing net alone would have given 13.32 / 26.57 ✗.
  - `TaxPercent` as effective rate — 41.53 ÷ 416.38 = 9.97 ✓.
  - Discount basis — 2% × pre-tax 416.38 = 8.33 ✓; 2% × tax-inclusive 478.73 = 9.57 ✗.
  - QST code — TimbrMart's own accepted invoice carries `PS`, so `PS` is correct and Canac's `SP` was a red herring.
- **Fix (applied in _v2):**
  - `GS` record: `TaxAmount` 0.00 → **19.87**, `TaxPercent` 0.00 → **5.00** (397.42 × 5%)
  - `PS` record: `TaxAmount` 0.00 → **39.64**, `TaxPercent` 0.00 → **9.97** (397.42 × 9.975%; 39.64 ÷ 397.42 = 9.97%)
  - `TotalAmount`: 397.42 → **456.93** (397.42 + 19.87 + 39.64)
  - Tax base = `TotalNetSalesAmount` 247.42 + `D240` freight charge 150.00 = 397.42.
  - **`TermsDiscountAmount` left at 7.95 — verified correct, not merely left alone.** 2% × pre-tax 397.42 = 7.95, which is the basis the reference confirms. It never depended on the tax, so the missing tax did not corrupt it.
  - **`TaxID`, `TaxTypeCode`, `TotalNetSalesAmount`, `TotalWeight` all unchanged** — each confirmed correct against the reference rather than assumed.
  - Nothing else changed; the diff is five lines.
- **Files:** `Resolved/4166843_Home Care TimbrMart 810.xml` → `Resolved/4166843_Home Care TimbrMart 810_v2.xml` (moved to `Resolved/` once the corrected invoice was confirmed sent); reference `Resolved/3892509_Home Care TimbrMart 810 - Reference.xml` moved alongside it
- **Status:** resolved. Corrected invoice sent to TIM-BR-MART, confirmed by the user 2026-09-08. **Upstream NAV check on ship-to 8262 (Tax Area Code / Tax Liable) and the blast-radius check on other Quebec ship-tos remain open** — the document is fixed, the cause is not, so a repeat on the next order to this dealer is still possible.
