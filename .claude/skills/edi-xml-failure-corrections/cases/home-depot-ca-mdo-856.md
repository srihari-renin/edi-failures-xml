# Home Depot.CA MDO (`ADJALLRENINHOLD`) — 856 — Case Log

Covers 856 (ASN) documents only. Other document types for this customer
(810, 850, ...) get their own `cases/home-depot-ca-mdo-<doctype>.md` file —
see `cases/README.md`.

Ships to DFC Bolton - 7340 (Caledon ON) from Renin Mississauga
(`RENMISSI-1`); buyer location `C00000368`.

## Partner notes

- **Shipping from the sales order instead of a warehouse shipment strips all
  pack data from the ASN.** NAV sources the entire `<Pack>` block and the
  `<ItemLevel>` block from the warehouse shipment's package records. Post the
  sales order directly and the ASN still generates — with zeroed weights and
  dimensions, a wrong `MarksAndNumbersQualifier1`, no SSCC, and **no line
  items at all**. See Case 01. If a warehouse shipment must be abandoned,
  capture its package and Shipping FastTab data *before* deleting it.
- **Carton count never appears in the 856.** The ASN models the *package*
  (pallet), not the cartons inside it. Confirmed across packages holding 1, 2
  and 6 units: each emitted exactly one `<Pack>` loop and one SSCC.
  - `ShipmentLadingQuantity` / `OrderLadingQuantity` count **packages** — they
    are `1` for a single-pallet shipment regardless of units or cartons.
  - Unit count lands only in `PackSize`, `OrderQty` and `ShipQty`.
  - NAV's warehouse-shipment header "Number of Pallets" / "Number of Cartons"
    fields are `0` even when packages exist, and feed nothing.
- **`ShipmentWeight` = `OrderWeight` = `PackWeight`** on every reference
  (42/42/42, 132/132/132). All three take the NAV package weight.
  **But the paired 810's `TotalWeight` differs** — 54.00 where the 856 said
  42.00 on invoice PSI1291553. Different source in NAV; do not copy the 856
  weight into the 810.
- **The SSCC (`MarksAndNumbers1`) encodes the NAV package number**, so it and
  `MarksAndNumbers2` are derivable from each other:
  `00` + `1` + `0043044` (Renin GS1 prefix) + package number padded to 9
  digits + GS1 mod-10 check digit. Verified on `PK896811`, `PK929766`,
  `PK930291`.
- **`PartDescription1` comes from the NAV item description, *not* the 850.**
  The 850 carries Home Depot's consumer-facing text
  (`36  X 80 1/2  WHTE FRM MIR SLD DR`); the 856 carries Renin's internal
  description (`120-3680-WT.R SLIDING MIRROR DOOR`). Every other line-item
  field — `BuyerPartNumber`, `ConsumerPackageCode`, `UnitPrice`, `OrderQty` —
  *does* come from the 850. Note these NAV strings can contain double spaces
  (`108-7280-BW.C ASHBURTON 2 PNL DOOR  ERIAS`), so copy them from the item
  card rather than transcribing from a screenshot.
- **`BillOfLadingNumber` differs by document type:** the 856 carries the
  **warehouse shipment number** (`WSH213195`, `WSH219461`); the paired 810
  carries the **posted shipment number** (`SS1299095`). When NAV generates the
  ASN from a sales order it has no WSH number and falls back to the `SS`
  number — i.e. it silently applies the 810 convention to an 856.
- **`CarrierProNumber` spacing:** NAV stores the MDO ID with a space
  (`MDO ID 6100977517`); the warehouse-shipment path strips it, the
  sales-order path does not. Home Depot has only ever accepted the unspaced
  form — emit `MDO ID6100977517`.
- Constants across every reference: `ASNStructureCode` `0001`,
  `TsetPurposeCode` `00`, `FOBPayCode` `CC`, `OrderStatusCode` `CC`,
  `OrderQtyPackingCode` `CTN`, `ItemStatusCode` `AC`, `OuterPack`/`InnerPack`
  `1`/`1`, `MR` reference `X2C`, `PackLevelType` `P`,
  `PackagingDescriptionCode` `1`, and the seven addresses (`RI`, `ST`, `SF`,
  `BT`, `VN`, `BY`, `ZZ` — `ZZ` empty). Pallet dimensions are `40 × 32 × 96`
  on every package seen so far, regardless of fill.
- `CurrentScheduledDeliveryDate` has been ship date **+ 2 days** on all
  references to DFC Bolton.
- Files from this partner use **CRLF** line endings and no trailing newline.

## Cases

Newest first.

### Case 01 — 856 — missing-segment — 2026-08-28
- **Document type:** 856
- **Error type:** `missing-segment` (secondary: `invalid-code` on
  `MarksAndNumbersQualifier1`, and zeroed quantity/weight/dimension values)
- **Reported by:** not a partner rejection — the ASN was inspected before
  sending, after the user flagged that the normal warehouse-shipment route had
  been abandoned. Caught proactively.
- **Error message:** n/a for the EDI document. The upstream NAV error, on
  attempting to post warehouse shipment WSH219483, was: *"The transaction
  cannot be completed because it will cause inconsistencies in the G/L Entry
  table. Check where and how the CONSISTENT function is used in the
  transaction to find the reason for the error."*
- **Source file:** `4160649_Home Depot.CA MDO 856.xml` (shipment SS1329633,
  invoice PSI1321830, PO 538506961, SO1327572, ship-to DFC Bolton - 7340) —
  untouched
- **Reference files:** `4160057_Home Depot.CA MDO 856.xml` (SS1329635 — the
  primary known-good, a 2-unit shipment from the same day),
  `4050970_Home Depot.CA MDO 856.xml` (SS1299095 — 1-unit),
  `4050986_Home Depot.CA MDO 810.xml` (the 810 paired with SS1299095),
  `4157486_Home Depot.CA MDO 850.xml` (PO 538506961 — the inbound PO for this
  very order, matched via `PurchaseOrderNumber`)
- **Reference ID:** shipment SS1329633, invoice PSI1321830, PO 538506961
- **Document defect:** 15 defects in three groups.
  - *Missing structure:* the entire `<ItemLevel>` / `<LineItem>` /
    `<ShipmentLine>` block was absent, so the ASN declared no product at all;
    `<Pack>` was missing its `TradingPartnerId`, `ShipmentIdentification` and
    `RecordType`>`PO`; `MarksAndNumbers1` and `MarksAndNumbers2` elements were
    both absent (the `SM` qualifier was present with no value).
  - *Zeroed values:* `ShipmentLadingQuantity`, `OrderLadingQuantity`,
    `TotalLineItems` = `0`; `ShipmentWeight`, `OrderWeight`, `PackWeight`,
    `PackSize`, `PackLength`, `PackWidth`, `PackHeight` = `0.00`;
    `PackagingDescriptionCode` = `0`.
  - *Wrong values:* `MarksAndNumbersQualifier1` = `CP` (should be `GM`);
    `CarrierProNumber` = `MDO ID 6100977517` with a space;
    `BillOfLadingNumber` = `SS1329633` (810 convention on an 856).
- **Upstream root cause:** confirmed. Warehouse shipment WSH219483 could not
  post (NAV G/L Entry inconsistency — see error message above). Attempts to
  clear it by removing the line from the package and the shipment, applying a
  penny adjustment on the sales order, and re-adding/re-packing it did not
  help. The shipment was deleted and SO1327572 was shipped and invoiced
  directly from the sales order instead. NAV sources the `<Pack>` and
  `<ItemLevel>` blocks from the warehouse shipment's package records, so that
  route produces an ASN with no pack or line data. The underlying G/L
  inconsistency on SO1327572 was **not** traced — see `gaps.md`.
- **Fix (applied in _v2):** values recovered from package PK930291 and the
  WSH219483 Shipping FastTab (captured before deletion, see
  `WSH219483-package-capture.md`), from the 850, and from the NAV item card.
  - Added the full `<ItemLevel>` block: `LineSequenceNumber` `10`,
    `BuyerPartNumber` `1000127692`, `VendorPartNumber` `BY0120BWCLJ036080`,
    `ConsumerPackageCode` `043044995468`, `PartDescription1`
    `120-3680-WT.R SLIDING MIRROR DOOR`, `OrderQty` `6` `EA`, `UnitPrice`
    `105.02`, `OuterPack`/`PackUOM`/`InnerPack` `1`/`EA`/`1`, `ShipQty`
    `6.00` `EA`, `ItemStatusCode` `AC`
  - Added to `<Pack>`: `TradingPartnerId`, `ShipmentIdentification`
    `SS1329633`, `RecordType` `PO`, `MarksAndNumbers1`
    `00100430440009302912`, `MarksAndNumbers2` `PK930291`
  - `MarksAndNumbersQualifier1`: `CP` → `GM`
  - `PackSize`: `0.00` → `6.00`; `PackWeight`: `0.00` → `210.00`
  - `PackLength`/`PackWidth`/`PackHeight`: `0.00` → `40.00`/`32.00`/`96.00`
  - `PackagingDescriptionCode`: `0` → `1`
  - `ShipmentLadingQuantity`, `OrderLadingQuantity`: `0` → `1`
  - `ShipmentWeight`, `OrderWeight`: `0.00` → `210.00`
  - `TotalLineItems`: `0` → `1`
  - `BillOfLadingNumber`: `SS1329633` → `WSH219483` (user decision — match the
    856 convention from both references, even though that WSH no longer exists)
  - `CarrierProNumber`: `MDO ID 6100977517` → `MDO ID6100977517`
- **Validation:** `_v2` parses as well-formed XML; its element structure is
  identical to reference `4160057`; CRLF endings and no trailing newline,
  matching the partner's other files. `UnitPrice` × `OrderQty` = 6 × 105.02 =
  630.12, against which the 850's own allowances reconcile exactly
  (`I170` @ 0.25% = 1.58 ✔, `C300` @ 1.25% = 7.88 ✔), confirming both price
  and quantity. The 850's `TotalLineItemNumber` `1` confirms `TotalLineItems`.
- **Files:** `4160649_Home Depot.CA MDO 856.xml` →
  `4160649_Home Depot.CA MDO 856_v2.xml`
- **Status:** corrected, awaiting send to Home Depot and partner confirmation.
