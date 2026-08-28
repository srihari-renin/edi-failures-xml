# WSH219483 — package capture before deletion

**Captured:** 2026-08-28
**Why:** Warehouse shipment WSH219483 cannot post (NAV G/L Entry inconsistency
error). Plan is to delete the WSH and ship/invoice directly from the sales
order instead — which produces an 856/810 with **no pack-level data**. This
file preserves the package detail so it can be re-applied by hand to the ASN.

**Reference documents** (known-good Home Depot.CA MDO, same trading partner,
same ship-to, used to map NAV fields → EDI fields):

| File | Shipment | Invoice | WSH | Package | Qty | Weight |
|---|---|---|---|---|---|---|
| `4050970_Home Depot.CA MDO 856.xml` | SS1299095 | PSI1291553 | WSH213195 | PK896811 | 1 | 42.00 |
| `4160057_Home Depot.CA MDO 856.xml` | SS1329635 | PSI1321832 | WSH219461 | PK929766 | 2 | 132.00 |
| `4050986_Home Depot.CA MDO 810.xml` | — | PSI1291553 | — | — | 1 | 54.00 † |

† 810 `TotalWeight`, which does **not** match the 856 `ShipmentWeight` of 42.00.

---

## 1. Raw NAV capture

### Warehouse shipment header — WSH219483

| Field | Value |
|---|---|
| No. | WSH219483 |
| Location Code | RENMISSI-1 |
| Zone Code / Bin Code | (blank) |
| Document Status | (blank) |
| Status | Open |
| Posting Date | 2026-08-28 |
| Assigned User ID | RENINCORP\FRANK.PERSAUD |
| Assignment Date / Time | 2026-08-26 / 3:30:26 PM |
| Sorting Method | (blank) |
| Hide Shipment No. and Date on Customer Portal | unchecked |
| Port of Arrival | (blank) |
| Shipping Freight Amount | 0 |
| PAPS | (blank) |

### Warehouse shipment — Shipping FastTab

**This tab is also lost on deletion** and is the source of several 856 fields.

| Field | Value | Feeds |
|---|---|---|
| External Document No. | (blank) | — |
| Shipment Date | 2026-08-28 | `CurrentScheduledShipDate` |
| Shipping Agent Code | `ROF` | `CarrierAlphaCode` |
| Shipping Agent Service Code | (blank) | — |
| Shipment Method Code | `COLLECT` | `FOBPayCode` = `CC` |
| **Tracking No.** | **`MDO ID 6100977517`** | **`CarrierProNumber`** |
| **TMS Load ID** | **`MDO ID 6100977517`** | (same value) |
| Trailer Length | (blank) | — |
| Dock No. | (blank) | — |
| Seal No. | `7600675` | not mapped in reference ASNs |
| Number of Pallets | 0 | not mapped — see note |
| Number of Cartons | 0 | not mapped — see note |
| Total Weight | 210 | `ShipmentWeight` / `OrderWeight` |
| Commercial Invoice Weight (LBS) | (blank) | — |
| Commercial Invoice Weight (KG) | 0 | — |

**Note on Number of Pallets / Cartons:** both are `0` at the header even though
PK930291 holds 6 cartons. Neither feeds `ShipmentLadingQuantity` (which is `1`
on both references) — further confirmation that `LadingQuantity` is derived
from the package count, not from these fields.

**Note on the MDO ID space:** NAV stores `MDO ID 6100977517` *with* a space,
but both reference ASNs emit `MDO ID6100897604` / `MDO ID6100977518` *without*
one. Match the reference output format — write `MDO ID6100977517` in the ASN.

**Cross-check:** the sibling shipment WSH219461, which posted normally the same
day, carried `MDO ID6100977518` — adjacent to ours, confirming both the format
and that Home Depot issues these IDs in a batch.

### Warehouse shipment line

| Field | Value |
|---|---|
| No. | WSH219483 |
| Source Document | Sales Order |
| Source No. | SO1327572 |
| Item No. | BY0120BWCLJ036080 |
| Description | 120-3680-WT.R SLIDING MIRROR… *(truncated on screen)* |
| Description 2 | (blank) |
| Location Code | RENMISSI-1 |
| Quantity | 6 |
| Qty. to Ship | 6 |
| Pick Qty. | 0 |
| Qty. Picked | 0 |

### Package — PK930291

| Field | Value |
|---|---|
| No. | PK930291 |
| Package Date | 2026-08-27 |
| Packing Station No. | (blank) |
| Destination Type | Customer |
| Destination No. | C00000368 |
| Ship-to Name | DFC Bolton - 7340 |
| Ship-to Address | 100 Pillsworth Road |
| Location Code | RENMISSI-1 |
| Tracking No. | (blank) |
| Packaging Type | 1 - Pallet or container larger than a carton |
| Carton Type | Re-pack |
| Carton Type Code | PALLET1 |
| No. of Cartons | 6 |
| Height | 96 |
| Length | 40 |
| Width | 32 |
| Cubage | 122,880 |
| Weight | 210 |
| Status | Packed |
| Comment | (blank) |
| UCC128 | 00100430440009302912 |

### Package contents — PK930291

| Field | Value |
|---|---|
| Item No. | BY0120BWCLJ036080 |
| UPC Code | *(blank on screen — see gaps)* |
| Description | 120-3680-WT.R SLIDING MIRROR… *(truncated)* |
| Unit of Measure | EA |
| Qty. Packed | 6 |
| Qty. Outstanding | 6 |
| Qty. Shipped | (blank) |
| Assignment No. | WSH219483 |
| Assignment Line No. | 200000 |
| Whse. Source Document | Sales Order |
| Whse. Source No. | SO1327572 |
| Whse. Source Line No. | 12000 |

---

## 2. How cartons map to the 856 — **they don't**

Confirmed across two reference ASNs with different unit counts:

| | PK896811 | PK929766 | PK930291 (ours) |
|---|---|---|---|
| Units in package | 1 | 2 | 6 |
| `PackSize` | `1.00` | `2.00` | **`6.00`** |
| `<Pack>` loops emitted | 1 | 1 | **1** |
| SSCCs | 1 | 1 | **1** |
| `ShipmentLadingQuantity` | `1` | `1` | **`1`** |
| `OrderLadingQuantity` | `1` | `1` | **`1`** |
| `OuterPack` / `InnerPack` | 1 / 1 | 1 / 1 | **1 / 1** |

**NAV's "No. of Cartons" never appears in the 856.** The ASN models the
*package* (pallet), not the cartons inside it. So:

- `LadingQuantity` counts **packages**, not cartons and not units — it stayed
  `1` on the 2-unit shipment, so it is `1` for our 6-unit shipment too.
- One NAV package → exactly **one** `<Pack>` loop and **one** SSCC, whatever
  the carton count. No per-carton labels are needed.
- Unit count lands only in `PackSize`, `OrderQty` and `ShipQty`.

Weights also confirm: `ShipmentWeight` = `OrderWeight` = `PackWeight` on both
references (42/42/42 and 132/132/132), so all three are **210.00** for ours.

### SSCC ↔ package number relationship (verified)

The UCC128 encodes the NAV package number. Validated all three check digits:

```
PK896811 → 00 1 0043044 000896811 9   ✔
PK929766 → 00 1 0043044 000929766 9   ✔
PK930291 → 00 1 0043044 000930291 2   ✔
           AI ext prefix   serial  chk
```

`0043044` is the Renin GS1 company prefix; serial = package number padded to
9 digits; standard GS1 mod-10 check digit. So `MarksAndNumbers1` and
`MarksAndNumbers2` are each derivable from the other.

---

## 3. Draft 856 field values

Everything below is settled unless marked **NEEDED**.

### ShipmentHeader

| Element | Value |
|---|---|
| `TradingPartnerId` | `ADJALLRENINHOLD` |
| `ShipmentIdentification` | **NEEDED** — `SS…` posted shipment no. |
| `RecordType` | `HS` |
| `ShipmentDate` | `20260828` |
| `Vendor` | `70001340` |
| `TsetPurposeCode` | `00` |
| `ShipNoticeDate` / `ShipNoticeTime` | `20260828` / time of generation |
| `ASNStructureCode` | `0001` |
| `ShipmentLadingQuantity` | `1` |
| `ShipmentWeight` / `UOM` | `210.00` / `LB` |
| `CarrierAlphaCode` | `ROF` |
| `CarrierTransMethodCode` | `M` |
| `CarrierRouting` | `Retailer owned/operated fleet` |
| `BillOfLadingNumber` | **NEEDED (decision)** — both refs used the WSH no.; ours is being deleted |
| `CarrierProNumber` | `MDO ID6100977517` (NAV Tracking No., space removed) |
| `FOBPayCode` | `CC` |
| `CurrentScheduledDeliveryDate` | `20260830` (ship + 2 days on both refs) |
| `CurrentScheduledShipDate` | `20260828` |

### References

| Qual | Value |
|---|---|
| `MR` | `X2C` — constant on both refs |
| `X9` | **NEEDED** — 10-digit Home Depot ref |

### Addresses — all fixed, no lookup needed

`RI` Renin Canada Corporation · `ST` qual 93 / `7340` DFC Bolton - 7340 ·
`SF` qual 91 / `RENMISSI-1` Renin Mississauga · `BT` HOME DEPOT CAN ECOMMERCE ·
`VN` Renin Canada Corporation · `BY` qual 91 / `C00000368` HOME DEPOT.CA (MDO) ·
`ZZ` empty.

NAV confirms the match: package Destination No. `C00000368` = `BY`, Ship-to
`DFC Bolton - 7340` = `ST`, Location `RENMISSI-1` = `SF`.

### OrderHeader

| Element | Value |
|---|---|
| `OrderNumber` | `SO1327572` |
| `InvoiceNumber` | **NEEDED** — `PSI…` posted invoice no. |
| `InvoiceDate` | `20260828` |
| `PurchaseOrderNumber` | **NEEDED** |
| `PurchaseOrderDate` | **NEEDED** |
| `OrderQtyPackingCode` | `CTN` |
| `OrderLadingQuantity` | `1` |
| `OrderWeight` / `UOM` | `210.00` / `LB` |
| `Vendor` | `70001340` |
| `CustomerOrderNumber` | **NEEDED** — 10-digit, `02…` |
| `OrderStatusCode` | `CC` |
| `DeliveryDate` | `20260830` |

### Pack — complete

| Element | Value |
|---|---|
| `RecordType` | `PO` |
| `PackLevelType` | `P` |
| `PackSize` / `PackUOM` | `6.00` / `EA` |
| `PackWeight` / `UOM` | `210.00` / `LB` |
| `PackLength` / `PackWidth` / `PackHeight` | `40.00` / `32.00` / `96.00` |
| `PackagingDescriptionCode` | `1` |
| `MarksAndNumbersQualifier1` / `MarksAndNumbers1` | `GM` / `00100430440009302912` |
| `MarksAndNumbersQualifier2` / `MarksAndNumbers2` | `SM` / `PK930291` |

### ShipmentLine

| Element | Value |
|---|---|
| `LineSequenceNumber` | `10` |
| `BuyerPartNumber` | **NEEDED** — HD SKU, 10-digit `10017…`/`10018…` |
| `VendorPartNumber` | `BY0120BWCLJ036080` |
| `ConsumerPackageCode` | **NEEDED** — UPC, `043044######` |
| `PartDescription1` | **NEEDED** — full text, starts `120-3680-WT.R SLIDING MIRROR` |
| `OrderQty` / `OrderQtyUOM` | `6` / `EA` |
| `UnitPrice` | **NEEDED** |
| `OuterPack` / `PackUOM` / `InnerPack` | `1` / `EA` / `1` |
| `ShipQty` / `ShipQtyUOM` | `6.00` / `EA` |
| `ItemStatusCode` | `AC` |

### Summary

`RecordType` `ST`, `TotalLineItems` `1` — assuming SO1327572 has no other lines.

---

## 4. Consistency checks performed

- Cubage reconciles: 40 × 32 × 96 = 122,880 ✔
- All three SSCC check digits validate under GS1 mod-10 ✔
- Pallet dimensions are **40 × 32 × 96 on all three packages**, holding 1, 2
  and 6 units. NAV writes the pallet footprint regardless of fill.
- Weight per unit: 42 (1 unit), 66 (2 units), 35 (6 units) — varies by item,
  as expected for different door sizes.
- ⚠ 856 `ShipmentWeight` (42.00) ≠ 810 `TotalWeight` (54.00) on the one
  matched pair. Different sources in NAV — do **not** assume 210 goes into
  both documents.
- WSH Status = Open, Qty. Shipped blank, Qty. Outstanding 6 → nothing posted;
  this package data is live and is lost on delete.

---

## 5. Remaining gaps

The other four sales orders originally on the shipment (SO1327637, SO1327656,
SO1327577, SO1327612) have all posted — confirmed 2026-08-28 — so PK930291
was the only package left to capture. Both the package and the Shipping
FastTab are now recorded above, so **WSH219483 can be deleted.**

Before deleting, worth a glance at the sales order: if SO1327572 carries its
own Shipping Agent Code / Shipment Method / package tracking fields, those
values survive deletion and can be read back later. If they are blank on the
SO, this file is the only remaining record of `ROF`, `COLLECT`,
`MDO ID6100977517`, seal `7600675` and the 210 lb total.

See [`open-questions.md`](open-questions.md).

---

## 6. The generated ASN — `4160649_Home Depot.CA MDO 856.xml`

Produced 2026-08-28 by shipping/invoicing SO1327572 directly from the sales
order after WSH219483 was deleted. Shipment **SS1329633**, invoice
**PSI1321830**. Diffed against `4160057` (SS1329635) as the known-good.

### Values NAV got right

`ASNStructureCode` `0001`, `CarrierAlphaCode` `ROF`, `CarrierTransMethodCode`
`M`, `CarrierRouting`, `FOBPayCode` `CC`, `CurrentScheduledShipDate`
`20260828`, `CurrentScheduledDeliveryDate` `20260830` (ship + 2, as predicted),
`MR`/`X2C`, and all seven addresses. Order-level identifiers all populated.

### Newly captured — previously listed as gaps

| Element | Value |
|---|---|
| `ShipmentIdentification` | `SS1329633` |
| `InvoiceNumber` | `PSI1321830` |
| `PurchaseOrderNumber` | `538506961` |
| `PurchaseOrderDate` | `20260826` |
| `CustomerOrderNumber` | `0242886527` |
| `X9` `ReferenceID` | `3789974279` |
| `ShipNoticeTime` | `1334` |

### Defects to correct in `_v2`

**A. Missing structure**

| # | Defect | Fix |
|---|---|---|
| 1 | Entire `<ItemLevel>` block absent — no `LineItem` / `ShipmentLine` at all | Rebuild the block |
| 2 | `<Pack>` missing `TradingPartnerId`, `ShipmentIdentification`, `RecordType`>`PO` | Add all three |
| 3 | `MarksAndNumbers1` element absent | Add `00100430440009302912` |
| 4 | `MarksAndNumbers2` element absent (qualifier `SM` present, value missing) | Add `PK930291` |

**B. Zeroed values**

| # | Element | Is | Should be |
|---|---|---|---|
| 5 | `ShipmentLadingQuantity` | `0` | `1` |
| 6 | `ShipmentWeight` | `0.00` | `210.00` |
| 7 | `OrderLadingQuantity` | `0` | `1` |
| 8 | `OrderWeight` | `0.00` | `210.00` |
| 9 | `PackSize` | `0.00` | `6.00` |
| 10 | `PackWeight` | `0.00` | `210.00` |
| 11 | `PackLength` / `PackWidth` / `PackHeight` | `0.00` | `40.00` / `32.00` / `96.00` |
| 12 | `PackagingDescriptionCode` | `0` | `1` |
| 13 | `TotalLineItems` | `0` | `1` |

**C. Wrong values**

| # | Element | Is | Should be |
|---|---|---|---|
| 14 | `MarksAndNumbersQualifier1` | `CP` | `GM` |
| 15 | `CarrierProNumber` | `MDO ID 6100977517` | `MDO ID6100977517` |

Defect 15 is new information: the WSH path emitted the MDO ID **without** the
space on both references, but the sales-order path emits it **with** the space
straight from the NAV field. Home Depot has only ever accepted the unspaced
form, so strip it.

### `BillOfLadingNumber` — open question now has a default

NAV populated it with `SS1329633`, the posted shipment number — i.e. it fell
back to the **810 convention**, not the 856 convention. Both reference ASNs
used a WSH number (`WSH213195`, `WSH219461`). Decision needed: leave NAV's
`SS1329633`, or force `WSH219483`.

### Item level — resolved from the 850

Source: `4157486_Home Depot.CA MDO 850.xml` (PO 538506961), matched to the ASN
via `PurchaseOrderNumber`. Per the Home Depot partner note in
[`home-depot-canada-810.md`](.claude/skills/edi-xml-failure-corrections/cases/home-depot-canada-810.md),
**the 850 carries the authoritative unit price**.

| `ShipmentLine` element | Value | Source |
|---|---|---|
| `LineSequenceNumber` | `10` | 850 |
| `BuyerPartNumber` | `1000127692` | 850 |
| `VendorPartNumber` | `BY0120BWCLJ036080` | 850 / NAV — agree |
| `ConsumerPackageCode` | `043044995468` | 850 |
| `PartDescription1` | **see gap below** | NAV item description |
| `OrderQty` / `OrderQtyUOM` | `6` / `EA` | 850 / NAV — agree |
| `UnitPrice` | `105.02` | 850 |
| `OuterPack` / `PackUOM` / `InnerPack` | `1` / `EA` / `1` | reference convention |
| `ShipQty` / `ShipQtyUOM` | `6.00` / `EA` | NAV |
| `ItemStatusCode` | `AC` | reference convention |

**Price cross-check:** 6 × 105.02 = 630.12. The 850's own allowances reconcile
against that exactly — `I170` @ 0.25% = 1.58 ✔ and `C300` @ 1.25% = 7.88 ✔ —
confirming both the unit price and the quantity.

**`TotalLineItems`:** the 850's `TotalLineItemNumber` is `1`, confirming
SO1327572 has a single line. Earlier open question closed.

### ⚠ Remaining gap — `PartDescription1` does **not** come from the 850

The 850's `PartDescription1` is `36  X 80 1/2  WHTE FRM MIR SLD DR` — Home
Depot's consumer-facing description. Both reference ASNs instead carry
**Renin's NAV item description**, in a completely different style:

| File | `VendorPartNumber` | 856 `PartDescription1` |
|---|---|---|
| `4050970` | `BY0108BWPRC072080` | `108-7280-BW.C ASHBURTON 2 PNL DOOR  ERIAS` |
| `4160057` | `BY0100BWWTC048096` | `100-4896-WT.E FLUSH PANEL DOOR` |

So the ASN must use the NAV description, not the 850's. NAV showed it as
`120-3680-WT.R SLIDING MIRROR DOOR` on the WSH219483 line — but that was read
off a screenshot, and the reference above proves these strings can contain
**double spaces** (`2 PNL DOOR  ERIAS`), which a screenshot cannot show
reliably. **Confirm the exact string from the item card before writing `_v2`.**

### `BillOfLadingNumber` — decided

Use `WSH219483`, matching both reference ASNs' convention (user decision,
2026-08-28). NAV's generated value `SS1329633` is overridden.
