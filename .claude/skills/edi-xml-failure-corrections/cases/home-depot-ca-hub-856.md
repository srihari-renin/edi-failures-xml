# Home Depot.CA Hub (`ADJALLRENINHOLD`) — 856 — Case Log

Covers 856 (ASN) documents only. The paired invoice log is
[home-depot-ca-hub-810.md](home-depot-ca-hub-810.md); see `cases/README.md`.

These ASNs arrive named `NNNNNNN_Home Depot.CA 856.xml` — **without** "Hub"
in the filename, unlike the paired `Home Depot.CA Hub 810`. They are filed
here rather than under a separate `home-depot-ca-856` slug because the
`MR` reference (`D2S` / `D2C`) and the shared PO make them the same program
as the Hub invoice. Don't confuse them with `Home Depot.CA MDO 856` (pallet
shipments to DFC Bolton), which has its own log.

## Partner notes

- **A SameDay parcel ASN is legitimately sparse.** Confirmed by accepted
  SS1311495 (`4094342_Home Depot.CA 856 - Reference.xml`): `ShipmentWeight`
  and `PackWeight` are `0.00`; there is no `PackSize`, no pack dimensions, no
  `PackagingDescriptionCode`, no `SM`/`MarksAndNumbers2` package number, no
  `OrderWeight`, `Vendor`, `InvoiceNumber`/`InvoiceDate` or `DeliveryDate` in
  `OrderHeader`, and no `CurrentScheduledShipDate`. None of that is a defect
  for this program — do not "complete" it from the MDO layout.
- **Tracking number appears twice:** `CarrierProNumber` in the header and a
  `Miscellaneous` block (`RecordType` `ML`, `Qualifier1` `ZZ`,
  `Description1` = the same SameDay number, format `A########`).
- **Carrier:** `CarrierAlphaCode` `SDCR_DS`, `CarrierTransMethodCode` `M`,
  `CarrierRouting` `SAMEDAY WORLDWIDE-Threshold Service`. The paired 810
  carries `UNSP` in `CarrierRouting` instead.
- **`BillOfLadingNumber` = the posted shipment number (`SS…`)** on this
  program — unlike MDO, where the 856 carries the `WSH…` number.
- **SSCC (`MarksAndNumbers1`) encodes the shipment number**, not a package
  number: `00` + extension `3` + `0043044` + `SS` number padded to 9 digits +
  GS1 mod-10 check digit (`00300430440013114956`, `00300430440013358299`).
  MDO uses extension `1` and the `PK` package number.
- `UnitPrice` on the `ShipmentLine` follows the sales order, so a price
  defect on the 810 shows up here too (Case 01).
- Constants across both documents seen: `TsetTypeCode` `SA`,
  `ASNStructureCode` `0001`, `ShipmentLadingQuantity` /
  `OrderLadingQuantity` `1`, `FOBPayCode` `CC`, `OrderStatusCode` `CC`,
  `ItemStatusCode` `AC`, `PackLevelType` `P`, `MarksAndNumbersQualifier1`
  `GM`, `OuterPack`/`InnerPack` `1`/`1`, six addresses (`RI` `92` no location
  number, `ST` `93` + store, `SF` `91` `RENMISSI-1`, `BT`, `VN` `91`
  `70001340`, `BY` `91` `C12000292` with `Country` `CA` despite the Atlanta
  address — accepted as-is). No `ZZ` address.
- `CurrentScheduledDeliveryDate` is not derived from the ship date — the
  reference had it two days *before* `ShipmentDate` and was still accepted.
- Files use **CRLF** line endings and no trailing newline.

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

### Case 01 — 856 — price-mismatch — 2026-09-17
- **Document type:** 856
- **Error type:** `price-mismatch`
- **Reported by:** caught before sending — checked alongside the paired
  invoice PSI1327965 after the user suspected missing information.
- **Error message:** n/a — no partner rejection; proactive check.
- **Source file:** `4182997_Home Depot.CA 856.xml` (shipment SS1335829,
  SO1333498, PO 538826239, store 7185 St-Constant QC) — untouched
- **Reference files:** `4094342_Home Depot.CA 856 - Reference.xml`
  (SS1311495 — accepted; same item, same D2S program, same carrier);
  `4177760_Home Depot.CA 850.xml` (PO 538826239 — settled the price).
- **Reference ID:** shipment SS1335829, PO 538826239, SO1333498
- **Document defect:** `ShipmentLine/UnitPrice` `171.20` where the 850 says
  `172.2`. **Nothing was missing**: the element structure was identical to
  the accepted reference, including all the zero weights and absent pack
  fields that looked suspicious next to the MDO layout (see Partner notes).
  SSCC check digit verified.
- **Upstream root cause:** same as the 810 — the NAV sales line price on
  SO1333498 was $1.00 below the PO; unconfirmed why.
- **Fix (applied in _v2):** `UnitPrice`: 171.20 → 172.20. One-line diff,
  CRLF preserved, no trailing newline.
- **Files:** `4182997_Home Depot.CA 856.xml` →
  `4182997_Home Depot.CA 856_v2.xml`
- **Status:** fixed, awaiting resend and partner confirmation. Paired invoice:
  [home-depot-ca-hub-810.md](home-depot-ca-hub-810.md) Case 02.
