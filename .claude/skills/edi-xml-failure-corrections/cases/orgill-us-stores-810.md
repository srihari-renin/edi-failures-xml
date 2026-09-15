# Orgill US Stores (`570ALLRENINHOLD`) — 810 — Case Log

Covers 810 (invoice) documents only. Other document types for this customer
(850, 856, ...) get their own `cases/orgill-us-stores-<doctype>.md` file —
see `cases/README.md`.

## Partner notes

- **D2C order PO number format, CONFIRMED:** 6-digit Business Central Member
  No. + hyphen + 4-digit Authorization No. suffix (e.g. `109942-3709`), **11
  characters including the dash** — not 10 as Orgill's own rejection message
  literally states. Confirmed against accepted/paid reference invoice
  PSI1222579 (PO `069914-9067`). Treat Orgill's rejection-message wording as
  an approximate description, not a literal character-count spec. —
  [Case 01](#case-01--810--po-number-format--2026-09-11)
- **D2C order also requires the same value in `LetterOfCredit`.** This feed
  reuses the standard EDI `LetterOfCredit` tag to carry the authorization
  number for Orgill D2C orders specifically — confirmed by the reference
  invoice, where `LetterOfCredit` = `PurchaseOrderNumber` = `069914-9067`. On
  a failing D2C invoice, expect `LetterOfCredit` to hold a placeholder (`NR`
  on Case 01) instead of the real value. — [Case 01](#case-01--810--po-number-format--2026-09-11)
- Both fields source from the sales order's Business Central **External
  Document No.** and **Authorization No.** fields, which display the same
  dash-separated value on the SO screen.
- **`LetterOfCredit` isn't always the broken field — check it first.** On
  Case 02, `LetterOfCredit` already held the correct `<member>-<auth>` value;
  only `PurchaseOrderNumber` had the stale internal PO number instead of that
  same value. Before assuming both fields need fixing (as on Case 01), check
  whether `LetterOfCredit` is already correct — if so, the fix is a single
  substitution: copy it into `PurchaseOrderNumber`. — [Case 02](#case-02--810--po-number-format--2026-09-11)
- **CONFIRMED: an 8-character original `PurchaseOrderNumber` also triggers a
  `BuyerPartNumber`-required check, and fixing the D2C PO format clears it as
  a side effect.** On Case 03, Orgill's rejection additionally flagged
  `BuyerPartNumber` missing on every line item that lacked one, with the
  stated condition "required when PO number is 8 characters long." The
  original PO (`42794261`) was exactly 8 characters; Cases 01 and 02's
  original POs weren't (5 and 11 characters respectively), and neither got
  this second error. `_v2` corrected the PO to the 11-character D2C format
  without touching `BuyerPartNumber`, and Orgill accepted it — **confirming
  the requirement is genuinely conditional on the PO being 8 characters, not
  an unconditional per-line requirement.** If a future case has an original
  PO that happens to be 8 characters, expect this second rejection line and
  don't populate `BuyerPartNumber` reactively — the D2C PO-format fix alone
  should clear it, same as here. See
  [Case 03](#case-03--810--po-number-format--2026-09-15).

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

### Case 03 — 810 — po-number-format — 2026-09-15
- **Document type:** 810
- **Error type:** `po-number-format`
- **Reported by:** Orgill SPS rejection notice, relayed by Sri Hari in chat (source filename `IN4071464 Orgill US Stores 810.C40004077`)
- **Error message:** "InvoiceNumber:PSI1297591 Missing mandatory data: /Invoice/LineItem/InvoiceLine/BuyerPartNumber; BuyerPartNumber is required when PO number is 8 characters long" (reported twice — once per affected line item) followed by "InvoiceNumber:PSI1297591 The Ship to address location number sent (N104 when N101=ST) indicates this order is for a direct to customer order, however the purchase order number (BIG04) is not valid. If this invoice is for a stock order, update the Ship To location number (N104 when N101=ST) to match the warehouse code from the purchase order's N104 ST field, then re-send the invoice. If this invoice is for a direct to customer order, please update the purchase order number (BIG04) to be the 6-character customer account number followed by the 4-character credit authorization number, resulting in a 10-character purchase order number and re-send the invoice." — **three rejection lines on one invoice, two distinct defects.** See the open flag below the fix.
- **Source file:** `4071464_Orgill US Stores 810.xml` (invoice PSI1297591, PO 42794261, ship-to Schoeneman's - Harrisburg, Harrisburg SD) — untouched
- **Reference file:** n/a — matched directly to Case 01/02 by customer + doc type + error type
- **Reference ID:** invoice PSI1297591, SO1290820
- **Document defect:** `PurchaseOrderNumber` (BIG04) = `42794261` and `LetterOfCredit` = `NR` — same shape as Case 01 (both fields wrong, `LetterOfCredit` a placeholder, not a usable value already in the document). Ship-to `125005` correctly signals D2C.
- **Upstream root cause:** Same mechanism as Case 01/02 — D2C order's PO/authorization fields never populated from the SO. Not independently reconfirmed in Business Central for this order beyond the value Sri Hari supplied.
- **Fix (applied in _v2):**
  - `PurchaseOrderNumber`: `42794261` → `125005-7176`
  - `LetterOfCredit`: `NR` → `125005-7176`
  - Value supplied directly by Sri Hari from Business Central (SO1290820's External Document No./Authorization No.), consistent with the ship-to number (`125005`) matching the value's first 6 digits, same pattern as Case 01/02.
  - Nothing else changed.
  - **Not fixed in this `_v2`, flagged instead — now CONFIRMED correct:** the rejection also carried two `BuyerPartNumber` missing-data errors, on `LineSequenceNumber` 2 (`VendorPartNumber` 201240) and 3 (201414) — line 1 (671-7839) already has one. Orgill's wording ties the requirement to PO length: "required when PO number is 8 characters long," and the *original* `PurchaseOrderNumber` (`42794261`) was exactly 8 characters — the only one of Cases 01–03 where that was true. The theory was that fixing the PO to the 11-character D2C format (`125005-7176`) would also remove this requirement as a side effect. **Confirmed: Orgill accepted `_v2` as sent, with `BuyerPartNumber` still absent on lines 2 and 3** — the 8-character-PO condition was the actual trigger, not an unconditional requirement. See Partner notes.
- **Files:** `Resolved/4071464_Orgill US Stores 810.xml` → `Resolved/4071464_Orgill US Stores 810_v2.xml`
- **Status:** resolved. Corrected invoice sent to Orgill, confirmed by the user 2026-09-15 — accepted with `BuyerPartNumber` still absent, confirming the 8-character-PO theory above. Files moved to `Resolved/`.

### Case 02 — 810 — po-number-format — 2026-09-11
- **Document type:** 810
- **Error type:** `po-number-format`
- **Reported by:** Orgill SPS rejection notice ("User Data Invalid"), relayed by Sri Hari in chat
- **Error message:** "Data Error: InvoiceNumber:PSI1321287 The Ship to address location number sent (N104 when N101=ST) indicates this order is for a direct to customer order, however the purchase order number (BIG04) is not valid. If this invoice is for a stock order, please update the Ship to location number (N104 when N101=ST) to be the exact warehouse code sent in the N104 "ST" field of the purchase order and re-send the invoice. If this invoice is for a direct to customer order, please update the purchase order number (BIG04) to be the 6-digit customer account number and the 4-digit credit authorization number, resulting in a 10-digit purchase order number and re-send the invoice."
- **Source file:** `4158334_Orgill US Stores 810.xml` (invoice PSI1321287, PO 2607-58535R, ship-to Helliesen Lumber, Yakima WA) — untouched
- **Reference file:** n/a — matched directly to Case 01 (same customer, same doc type, same error type); no new reference document needed
- **Reference ID:** invoice PSI1321287, SO1319846
- **Document defect:** `PurchaseOrderNumber` (BIG04) = `2607-58535R` — the order's internal PO number, not the D2C format Orgill requires. Ship-to (N104 when N101=ST) = `343962`, a real customer (Helliesen Lumber), correctly signaling D2C. Unlike Case 01, `LetterOfCredit` was **already correct** at `343962-2065` — matching the ship-to number as its first 6 digits — so only one field was actually broken.
- **Upstream root cause:** Not separately investigated — matches Case 01's pattern (D2C order's `PurchaseOrderNumber` never populated from the SO's authorization value), except this order's `LetterOfCredit` was populated correctly while `PurchaseOrderNumber` wasn't. Same upstream mechanism as Case 01 is the working assumption; not reconfirmed against Business Central for this specific order since the fix was self-evident from `LetterOfCredit` already present on the same document.
- **Fix (applied in _v2):**
  - `PurchaseOrderNumber`: `2607-58535R` → `343962-2065` (copied directly from this document's own `LetterOfCredit`, not derived externally)
  - `LetterOfCredit`: unchanged — already correct.
  - Nothing else changed.
- **Files:** `Resolved/4158334_Orgill US Stores 810.xml` → `Resolved/4158334_Orgill US Stores 810_v2.xml`
- **Status:** resolved. Corrected invoice sent to Orgill, confirmed by the user 2026-09-15. Files moved to `Resolved/`.

### Case 01 — 810 — po-number-format — 2026-09-11
- **Document type:** 810
- **Error type:** `po-number-format`
- **Reported by:** Katherine (Orgill), via SPS rejection notice, relayed by Sri Hari in chat
- **Error message:** "Data Error: InvoiceNumber:PSI1285562 The Ship to address location number sent (N104 when N101=ST) indicates this order is for a direct to customer order, however the purchase order number (BIG04) is not valid. If this invoice is for a stock order, update the Ship To location number (N104 when N101=ST) to match the warehouse code from the purchase order's N104 ST field, then re-send the invoice. If this invoice is for a direct to customer order, please update the purchase order number (BIG04) to be the 6-character customer account number followed by the 4-character credit authorization number, resulting in a 10-character purchase order number and re-send the invoice."
- **Source file:** `4028691_Orgill US Stores 810.xml` (invoice PSI1285562, PO 84837, ship-to C.C. Allis & Sons Inc, Wyalusing PA) — untouched
- **Reference file:** `3805032_Orgill US Stores 810 - Reference.xml` (invoice PSI1222579, accepted/paid, PO `069914-9067`, ship-to T H Rogers Lumber, Pea Ridge AR) — same customer, same doc type, a confirmed D2C order that was accepted; used to settle the exact PO number format (dash vs. no dash) and to discover the `LetterOfCredit` duplication
- **Reference ID:** invoice PSI1285562, SO1269579
- **Document defect:** `PurchaseOrderNumber` (BIG04) = `84837` — doesn't match the D2C format Orgill requires, and `LetterOfCredit` = `NR` — a placeholder, not the required authorization value. Ship-to (N104 when N101=ST) = `NR`, a non-warehouse value, correctly signals D2C per Orgill's own rule; the mismatch was entirely in the two PO/authorization fields, not the ship-to.
- **Upstream root cause:** SO1269579 was a manual drop-ship order for an Orgill dealer (confirmed by Diego Arizaga, Retail Account Manager, by email: "this was a manual drop ship order for an orgill dealer... not an orgill warehouse"). The invoice's `PurchaseOrderNumber` carried the order's internal PO number instead of the D2C-specific identifier, and `LetterOfCredit` was left at its default placeholder. Business Central's own SO1269579 record already has the correct value in both its **External Document No.** and **Authorization No.** fields (`109942-3709` — Member No. `109942` + Authorization No. suffix `3709`), confirmed via a Business Central screenshot supplied by Sri Hari. The outbound 810 simply never populated `PurchaseOrderNumber`/`LetterOfCredit` from those SO fields for this order.
- **Fix (applied in _v2):**
  - `PurchaseOrderNumber`: `84837` → `109942-3709`
  - `LetterOfCredit`: `NR` → `109942-3709`
  - Format (dash included, 11 characters) confirmed against reference invoice PSI1222579, not from Katherine's message alone — her "10-character" wording undercounts the dash. See Partner notes.
  - Nothing else changed. Ship-to N104 (`NR`) left as-is — the error only implicated the PO/authorization fields, not the address.
- **Files:** `Resolved/4028691_Orgill US Stores 810.xml` → `Resolved/4028691_Orgill US Stores 810_v2.xml`; reference `Resolved/3805032_Orgill US Stores 810 - Reference.xml`
- **Status:** resolved. Corrected invoice sent to Orgill, confirmed by the user 2026-09-11. Files moved to `Resolved/`.
