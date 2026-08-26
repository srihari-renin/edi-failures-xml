# True Value (`GZWALLRENINHOLD`) — 856 — Case Log

Covers 856 (ASN) documents only. Other document types for this customer
(810, 850, ...) get their own `cases/true-value-<doctype>.md` file — see
`cases/README.md`.

## Partner notes

- **Carrier fields are all-or-nothing** — same mechanic as
  [Do It Best Hardware 856 Case 01](do-it-best-hardware-856.md#case-01--856--missing-segment--2026-08-26).
  If `ShipmentHeader` carries any carrier info at all (e.g.
  `CarrierTransMethodCode`), True Value also requires `CarrierAlphaCode` and
  `CarrierRouting` to be present — omitting either one fails validation. See
  Case 01.
- The actual carrier for a given shipment is **not derivable from the ASN
  itself or from a reference file** — it must come from the BOL, carrier
  invoice, or TMS/shipping record for that specific shipment, or user
  confirmation. Don't assume a reference document's carrier applies to the
  shipment being fixed unless the user confirms it's the same carrier.

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
- **Reported by:** True Value trading-partner portal, automated failure notice, two separate "Required field missing from your document" errors referencing the same line/column
- **Error message:** "Your document is missing CarrierAlphaCode, a field this trading partner requires before they'll accept it. The document references it at line 95, column 30." / "Your document is missing CarrierRouting, a field this trading partner requires before they'll accept it. The document references it at line 95, column 30."
- **Source file:** `4148685_True Value 856.xml` (ASN SS1326195, PO 08062302W2700, ship-to True Value Corporation - Warehouse, Chicago IL) — untouched
- **Reference file:** `4098118_True Value 856 - Reference.xml` (ASN SS1312551, PO 06252295W2700) — used only to confirm where/how the two fields should appear in `ShipmentHeader`, not as the source of the actual carrier value
- **Reference ID:** ASN SS1326195, PO 08062302W2700, Document ID SS1326195
- **Document defect:** `ShipmentHeader` had `CarrierTransMethodCode` (`M`) but no `CarrierAlphaCode` or `CarrierRouting` elements at all — True Value requires all three together once any carrier info is present.
- **Upstream root cause:** unconfirmed — not yet traced in the shipping/ERP system that generates the ASN.
- **Fix (applied in _v2):**
  - Added `CarrierAlphaCode` = `ODFL` and `CarrierRouting` = `OLD DOMINION FREIGHT LINE` to `ShipmentHeader`, in the same position as the reference document.
  - User confirmed Old Dominion Freight Line was the actual carrier for this shipment (BOL WSH218632, Carrier Pro 04116241151) — not inferred from the reference alone.
  - Nothing else changed.
- **Files:** `Resolved/4148685_True Value 856.xml` → `Resolved/4148685_True Value 856_v2.xml`
- **Status:** resolved. Corrected ASN sent to True Value, confirmed by the user 2026-08-26. Files moved to `Resolved/`.
