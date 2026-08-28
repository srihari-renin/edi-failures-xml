# Open Questions

Questions asked of Sri Hari that didn't get answered right away — logged
here so they don't get lost when he jumps to a different task. Whenever you
ask a decision question and don't get an answer in the same turn, add it
here before moving on. When picking work back up, check this file first and
re-ask anything still pending.

When a question gets answered: either delete its entry, or move it under
**Answered** with a one-line note of the answer — whichever fits.

## Pending

<!--
Entry template:

### <short question title> — asked <YYYY-MM-DD>
**Question:** <the actual question, or enough of it to re-ask cold>
**Why it matters / what's blocked:** <what can't proceed until this is answered>
-->

_(All questions below were answered — `_v2` is written. Kept only as the audit
trail for Case 01; delete once the ASN is confirmed accepted.)_

### ~~WSH219483 / PK930291 — data still needed to hand-build the Home Depot MDO 856~~ — asked and fully answered 2026-08-28

**Context:** WSH219483 won't post (NAV G/L Entry inconsistency). Plan is to
delete the WSH and ship/invoice from SO1327572 directly, which produces an
856 with no pack data. Package detail is captured in
[`WSH219483-package-capture.md`](WSH219483-package-capture.md); these are the
remaining gaps.

**Questions:**

1. ~~**Before deleting — do the other four sales orders originally on
   WSH219483 (SO1327637, SO1327656, SO1327577, SO1327612) still need their
   packages captured?**~~ **Answered 2026-08-28: all four have posted.**
   Nothing further to capture — WSH219483 is safe to delete.
2. **`BillOfLadingNumber`** — both reference 856s used the warehouse shipment
   number (`WSH213195`, `WSH219461`). WSH219483 is being deleted. Still use
   `WSH219483`, or substitute something else?
3. **Item identifiers** for BY0120BWCLJ036080: BuyerPartNumber (Home Depot
   SKU, 10-digit), UPC / ConsumerPackageCode (blank on the Package Contents
   screen), full untruncated description, unit price.
4. ~~**Order identifiers** for SO1327572~~ **Answered 2026-08-28** from the
   generated ASN `4160649`: PO `538506961`, PO date `20260826`,
   CustomerOrderNumber `0242886527`, `X9` `3789974279`.
5. ~~**`CarrierProNumber`**~~ **Answered 2026-08-28:** WSH219483 Shipping tab,
   Tracking No. / TMS Load ID = `MDO ID 6100977517` → emit as
   `MDO ID6100977517`.
6. **Does SO1327572 have lines beyond BY0120BWCLJ036080?** Invoicing from the
   SO ships the whole order, so any extra line would appear on the 856
   without captured pack data.
7. ~~**Posted document numbers**~~ **Answered 2026-08-28:** shipment
   `SS1329633`, invoice `PSI1321830`.

**Why it matters / what's blocked:** only Q2 (BOL decision) and Q3 (four
item-level fields) remain, and both block writing
`4160649_Home Depot.CA MDO 856_v2.xml`. Q6 is a sanity check, not a blocker.

**Resolved by reference ASN `4160057_Home Depot.CA MDO 856.xml` (SS1329635):**
carton count never appears in the 856; one NAV package → one `<Pack>` loop and
one SSCC regardless of cartons; `ShipmentLadingQuantity` /
`OrderLadingQuantity` count packages, so both are `1`; carrier is `ROF` /
`M` / `Retailer owned/operated fleet`. See
[`WSH219483-package-capture.md`](WSH219483-package-capture.md) §2.

## Answered

_(Empty.)_
