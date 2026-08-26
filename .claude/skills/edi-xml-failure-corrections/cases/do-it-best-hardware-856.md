# Do It Best Hardware (`562ALLRENINHOLD`) — 856 — Case Log

Covers 856 (ASN) documents only. Other document types for this customer
(810, 850, ...) get their own `cases/do-it-best-hardware-<doctype>.md` file
— see `cases/README.md`.

## Partner notes

- **Carrier fields are all-or-nothing.** If `ShipmentHeader` carries any
  carrier info at all (e.g. `CarrierTransMethodCode`), the partner also
  requires `CarrierAlphaCode` and `CarrierRouting` to be present — omitting
  either one fails validation. See Case 01.
- The actual carrier for a given shipment is **not derivable from the ASN
  itself or from a reference file** — it must come from the BOL, carrier
  invoice, or TMS/shipping record for that specific shipment. Don't assume a
  reference document's carrier applies to the shipment being fixed unless
  the user confirms it's the same carrier.

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

### Case 01 — 856 — missing-segment — 2026-08-26
- **Document type:** 856
- **Error type:** `missing-segment`
- **Reported by:** Do It Best Hardware trading-partner portal, automated failure notice, two separate "Required field missing from your document" errors referencing the same line/column
- **Error message:** "Your document is missing CarrierAlphaCode, a field this trading partner requires before they'll accept it. The document references it at line 26, column 22." / "Your document is missing CarrierRouting, a field this trading partner requires before they'll accept it. The document references it at line 26, column 22."
- **Source file:** `4148684_Do It Best Hardware 856.xml` (ASN SS1326194, PO DIB300124, ship-to Do It Best Corp. - Mesquite) — untouched
- **Reference file:** `4134309_Do It Best Hardware 856 (Reference).xml` (ASN SS1322542, PO DIB299402, ship-to Do It Best Corp. - Dixon) — used only to confirm where/how the two fields should appear in `ShipmentHeader`, not as the source of the actual carrier value
- **Reference ID:** ASN SS1326194, PO DIB300124
- **Document defect:** `ShipmentHeader` had `CarrierTransMethodCode` (`M`) but no `CarrierAlphaCode` or `CarrierRouting` elements at all — DIB requires all three together once any carrier info is present.
- **Upstream root cause:** unconfirmed — not yet traced in the shipping/ERP system that generates the ASN.
- **Fix (applied in _v2):**
  - Added `CarrierAlphaCode` = `ODFL` and `CarrierRouting` = `OLD DOMINION FREIGHT LINE` to `ShipmentHeader`, in the same position as the reference document.
  - User confirmed Old Dominion Freight Line was the actual carrier for this shipment (same carrier as the reference document) — not inferred from the reference alone.
  - Nothing else changed.
- **Files:** `4148684_Do It Best Hardware 856.xml` → `4148684_Do It Best Hardware 856_v2.xml`
- **Status:** resolved (fix applied; awaiting resend/confirmation from partner).
