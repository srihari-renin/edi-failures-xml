# Home Depot.CA MDO (`ADJALLRENINHOLD`) — 810 — Case Log

Covers 810 (invoice) documents only. Other document types for this customer
get their own `cases/home-depot-ca-mdo-<doctype>.md` file — see the 856 log at
[home-depot-ca-mdo-856.md](home-depot-ca-mdo-856.md) and `cases/README.md`.

## Partner notes

- **The `850` carries the authoritative `UnitPrice` and allowance amounts.**
  Same convention already proven on
  [Home Depot Canada](home-depot-canada-810.md) — match the invoice to its PO
  via `PurchaseOrderNumber` and compare before assuming the invoice is right.
  An 810 that drifts from its PO still **foots correctly on its own numbers**
  and passes validation, so this is only ever caught by comparing documents.
- **All three summary formulas are confirmed** (verified on PSI1291553 and
  PSI1321830, and used to correct Case 01):
  - taxable base = `TotalNetSalesAmount` − Σ`AllowChrgAmt` (all indicator `A`)
  - `TaxAmount` = 13% × taxable base
  - `TotalAmount` = taxable base + `TaxAmount`
  - `TermsDiscountAmount` = 2% × (`TotalNetSalesAmount` + `TaxAmount`)
    — note this base **excludes** allowances, unlike the tax base.
  The `TermsDiscountAmount` formula is worth re-testing on
  [Home Depot Canada](home-depot-canada-810.md), whose Cases 01/02 record it
  as not reconciling to any obvious base and left it untouched.
- **Allowance percentages:** `I170` = 0.25% and `C300` = 1.25% of the extended
  line total. Both appear on the 850 with explicit `AllowChrgPercent`, so they
  can be recomputed independently as a cross-check on `UnitPrice` × `OrderQty`.
- **Field conventions that differ from the 856 for the same shipment** — do
  not copy values across:
  - `BillOfLadingNumber`: 810 uses the **posted shipment number** (`SS…`); the
    856 uses the **warehouse shipment number** (`WSH…`).
  - `CarrierRouting`: 810 uses `UNSP`; the 856 uses
    `Retailer owned/operated fleet`.
  - `BY` address: 810 uses `LocationCodeQualifier` `93` + store `7340`; the
    856 uses `91` + `C00000368`.
  - `RI` address: 810 uses qualifier `92`; the 856 carries no qualifier.
  - `TotalWeight`: **not** the pallet weight. PSI1291553 shows `54.00` where
    its own ASN reported `42.00`. Source unconfirmed — likely item-card gross
    weight × quantity rather than actual pack weight.
- **`CarrierProNumber` spacing:** same defect as the 856 — NAV stores
  `MDO ID 6100977517` with a space; every accepted document has it unspaced.
  Emit `MDO ID6100977517`.
- Files from this partner use **CRLF** line endings and no trailing newline.
  `PartDescription1` may carry trailing spaces straight from NAV; that is
  source data, not a defect.

## Cases

Newest first.

### Case 01 — 810 — price-mismatch — 2026-08-28
- **Document type:** 810
- **Error type:** `price-mismatch`
- **Reported by:** not a partner rejection — found proactively while checking
  the invoice paired with the ASN rebuilt in
  [856 Case 01](home-depot-ca-mdo-856.md#case-01--856--missing-segment--2026-08-28).
- **Error message:** n/a — no rejection. The invoice validates cleanly.
- **Source file:** `4160650_Home Depot.CA MDO 810.xml` (invoice PSI1321830,
  PO 538506961, SO1327572, shipment SS1329633, ship-to DFC Bolton - 7340) —
  untouched
- **Reference files:** `4157486_Home Depot.CA MDO 850.xml` (PO 538506961 — the
  paired PO, matched via `PurchaseOrderNumber`; authoritative on price),
  `4050986_Home Depot.CA MDO 810.xml` (invoice PSI1291553 — known-good, used
  to verify the tax/total/terms formulas)
- **Reference ID:** invoice PSI1321830, PO 538506961
- **Document defect:** `UnitPrice` was `104.99` against the PO's `105.02` — 3
  cents per unit, $0.18 across 6 units. The two allowances drifted one cent
  each in step (`I170` `1.57` vs PO `1.58`; `C300` `7.87` vs PO `7.88`),
  because NAV computed them as percentages of the *understated* extended
  total. `TaxAmount`, `TotalAmount` and `TermsDiscountAmount` were all
  internally consistent with the understated figures, so the invoice footed
  correctly and would not have been rejected — Renin would simply have
  under-billed by $0.18.
- **Upstream root cause:** confirmed with high confidence. A **penny
  adjustment was applied to SO1327572 during troubleshooting** of the NAV G/L
  inconsistency that blocked warehouse shipment WSH219483 (see
  [856 Case 01](home-depot-ca-mdo-856.md#case-01--856--missing-segment--2026-08-28)
  and `gaps.md`). That adjustment moved the sales line price off the PO price,
  and the invoice inherited it. The wider lesson: **a penny adjustment used to
  clear a posting error silently corrupts the invoiced price**, and the
  resulting invoice gives no signal that anything is wrong.
- **Fix (applied in _v2):** all values taken from the 850.
  - `InvoiceLine/UnitPrice`: `104.99` → `105.02`
  - `InvoiceLine/ExtendedItemTotal`: `629.94` → `630.12` (6 × 105.02)
  - `Summary/Totals/TotalNetSalesAmount`: `629.94` → `630.12`
  - `ChargesAllowances[I170]/AllowChrgAmt`: `1.57` → `1.58`
  - `ChargesAllowances[C300]/AllowChrgAmt`: `7.87` → `7.88`
  - `Summary/Tax/TaxAmount`: `80.67` → `80.69`
    (13% × (630.12 − 1.58 − 7.88) = 13% × 620.66 = 80.6858)
  - `Summary/Totals/TotalAmount`: `701.17` → `701.35` (620.66 + 80.69)
  - `PaymentTerms/TermsDiscountAmount`: `14.21` → `14.22`
    (2% × (630.12 + 80.69) = 14.2162)
  - `InvoiceHeader/CarrierProNumber`: `MDO ID 6100977517` →
    `MDO ID6100977517` (space removed, matching every accepted document)
  - Net effect: invoiced total rises from $701.17 to **$701.35**.
- **Deliberately not changed** — differences from the reference that the
  diagnosis does not evidence, per the "fix only what's evidenced" rule:
  - `TotalWeight` `0.00` (reference PSI1291553 shows `54.00`). Zeroed by the
    same root cause as the ASN's weights, but the 810's weight is **not** the
    pallet weight, and the true source is unconfirmed — so there is no
    authoritative value to substitute. User confirmed 2026-08-28 that it has
    no downstream effect; left as `0.00`.
  - `ProductProcessDescription` `ALLITEMS` (reference shows `040`). Only one
    reference 810 exists, so it is unknown whether `040` is item-specific or
    whether `ALLITEMS` is a generic fallback produced by the sales-order
    route. **Open question** — resolve on the next MDO 810 and correct here if
    it turns out to be wrong.
  - `PartDescription1` trailing spaces — source data from NAV, not a defect.
- **Validation:** `_v2` parses as well-formed XML; diff against the original is
  exactly the 9 intended lines and nothing else; CRLF endings preserved. Every
  total re-derives from the corrected figures: 105.02 × 6 = 630.12 =
  `ExtendedItemTotal` = `TotalNetSalesAmount`; base 620.66; tax 80.69; total
  701.35; terms 14.22. `UnitPrice`, `I170` and `C300` all match the 850.
- **Files:** `4160650_Home Depot.CA MDO 810.xml` →
  `4160650_Home Depot.CA MDO 810_v2.xml`
- **Status:** corrected, awaiting send to Home Depot and partner confirmation.
