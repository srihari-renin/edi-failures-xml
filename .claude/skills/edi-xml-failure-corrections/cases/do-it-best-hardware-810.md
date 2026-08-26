# Do It Best Hardware (`562ALLRENINHOLD`) — 810 — Case Log

Covers 810 (invoice) documents only. Other document types for this customer
(850, 856, ...) get their own `cases/do-it-best-hardware-<doctype>.md` file
— see `cases/README.md`.

## Partner notes

- **Carrier fields are all-or-nothing**, same rule as the 856 for this
  customer (see `cases/do-it-best-hardware-856.md`): if `InvoiceHeader`
  carries any carrier info (e.g. `CarrierProNumber`, `BillOfLadingNumber`),
  the partner also requires `CarrierAlphaCode` and `CarrierRouting` —
  omitting either fails validation with "Carrier Routing is required when
  sending any Carrier information."
- When an invoice references a shipment (`BillOfLadingNumber` /
  `ReferenceQual=PK`) whose 856 ASN was already fixed for the same carrier
  defect, reuse that confirmed carrier rather than re-asking — it's the same
  shipment, not a new inference. See Case 01.

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

### Case 01 — 810 — missing-segment — 2026-08-26
- **Document type:** 810
- **Error type:** `missing-segment`
- **Reported by:** Do It Best Hardware trading-partner portal, automated failure notice ("User Data Invalid")
- **Error message:** "Data Error: InvoiceNumber:PSI1318473 Carrier Routing is required when sending any Carrier information."
- **Source file:** `4148721_Do It Best Hardware 810 Sales Invoice.xml` (invoice PSI1318473, PO DIB300124, BOL SS1326194) — untouched
- **Reference file:** `4134439_Do It Best Hardware 810 Sales Invoice (Reference).xml` (invoice PSI1314855, PO DIB299402, BOL SS1322542) — used to confirm where the two fields belong in `InvoiceHeader`
- **Reference ID:** invoice PSI1318473, PO DIB300124, BOL SS1326194
- **Document defect:** `InvoiceHeader` had `CarrierProNumber` and `BillOfLadingNumber` but no `CarrierAlphaCode` or `CarrierRouting` — same all-or-nothing carrier rule as the 856 for this customer. This invoice's BOL (`SS1326194`) is the same shipment fixed in the 856 case (`cases/do-it-best-hardware-856.md`, Case 01), so the carrier was already confirmed rather than re-derived from the reference.
- **Upstream root cause:** unconfirmed — same likely upstream source as the 856 (shipping/ERP system not populating carrier fields for this shipment).
- **Fix (applied in _v2):**
  - Added `CarrierAlphaCode` = `ODFL` and `CarrierRouting` = `OLD DOMINION FREIGHT LINE` to `InvoiceHeader`, in the same position as the reference document.
  - Nothing else changed.
- **Files:** `Resolved/4148721_Do It Best Hardware 810 Sales Invoice.xml` → `Resolved/4148721_Do It Best Hardware 810 Sales Invoice_v2.xml`
- **Status:** resolved. Corrected invoice sent to Do It Best, confirmed by the user 2026-08-26. Files moved to `Resolved/`.
