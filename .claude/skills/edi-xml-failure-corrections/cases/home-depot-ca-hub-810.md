# Home Depot.CA Hub (`ADJALLRENINHOLD`) — 810 — Case Log

Covers 810 (invoice) documents only. Other document types for this customer
(850, 856, 846, ...) get their own `cases/home-depot-ca-hub-<doctype>.md`
file — see `cases/README.md`.

## Partner notes

- **`AllowChrgIndicator = A` on the header `ChargesAllowances` block reduces
  `TotalAmount`.** Balance due reconciles as
  `TotalNetSalesAmount + TaxAmount - Σ(AllowChrgAmt where indicator = A)`.
  Confirmed on Case 01 (two allowance lines, codes I170 and C300). No charge
  (`C`) indicator seen yet for this customer to confirm the sign flips the
  other way — treat that as unconfirmed until seen.
- **The `850` carries the authoritative `UnitPrice` and allowance amounts** —
  same convention as [Home Depot.CA MDO](home-depot-ca-mdo-810.md) and
  [Home Depot Canada](home-depot-canada-810.md). An 810 that drifts from its
  PO still foots on its own numbers and passes validation; it is only caught
  by matching to the 850 via `PurchaseOrderNumber`. Case 02 was $1.00/unit
  under the PO with every derived figure consistent.
- **Summary formulas, confirmed to the cent on PSI1303944 (accepted) and
  PSI1327965 `_v2`:**
  - `I170` = 0.25% and `C300` = 1.25% of `TotalNetSalesAmount` (both are on
    the 850 with `AllowChrgPercent`, so they double as a price cross-check)
  - taxable base = `TotalNetSalesAmount` − Σ`AllowChrgAmt`
  - `TotalAmount` = `TotalNetSalesAmount` + Σ`TaxAmount` − Σ`AllowChrgAmt`
  - `TermsDiscountAmount` = 2% × (`TotalNetSalesAmount` + Σ`TaxAmount`)
    — excludes allowances, same as MDO.
- **Quebec ship-to:** two `Tax` records — `CG` (GST 5%) and **`ST`** (QST
  9.975%, `TaxPercent` shown as `9.97` or `9.98` depending on NAV rounding;
  both accepted). Both lines carry the QST registration `1225162693TQ0001`
  as `TaxID`, not the GST number `835391830` used on non-QC invoices. All
  confirmed by accepted invoice PSI1303944 (store 7147, St-Jérôme).
- **Order programs:** `MR` reference `D2C` / `4C` = `C` is home delivery to
  the consumer (PSI1317282); `D2S` / `4C` = `S` is ship-to-store for pickup
  — the `ST` address is the **Home Depot store** (`93` + store number) with
  the *consumer's* name in `AddressName`. The `850` carries the same value
  in `TicketingCodeReference` and its `MR` reference.
- **`TotalWeight` / `TotalVolume` are `0.00`** on every accepted Hub invoice
  seen (PSI1317282, PSI1303944) — not a defect for this program.
- Files from this partner use **CRLF** line endings and no trailing newline.

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

### Case 02 — 810 — price-mismatch — 2026-09-17
- **Document type:** 810
- **Error type:** `price-mismatch`
- **Reported by:** caught before sending — the user suspected "some
  information might be missing" on the ASN/invoice pair for PO 538826239 and
  asked for a comparison against previously accepted documents.
- **Error message:** n/a — no partner rejection; proactive check.
- **Source file:** `4182998_Home Depot.CA Hub 810.xml` (invoice PSI1327965,
  PO 538826239, SO1333498, shipment SS1335829, store 7185 St-Constant QC) —
  untouched
- **Reference files:** `4094391_Home Depot.CA Hub 810 - Reference.xml`
  (PSI1303944, PO 537321519 — accepted; **same item
  `BY0114BWPRM060080`, same D2S program, same carrier SDCR_DS, QC ship-to,
  qty 1**, so it settled every derived figure); `4177760_Home Depot.CA
  850.xml` (the inbound PO 538826239 itself — settled the price);
  `4094342_Home Depot.CA 856 - Reference.xml` (the ASN paired with
  PSI1303944; see the 856 log).
- **Reference ID:** invoice PSI1327965, PO 538826239, SO1333498
- **Document defect:** `UnitPrice` `171.20` where the 850 says `172.2`.
  Every dependent figure was computed consistently around the wrong price
  (C300 2.14, GST 8.43, QST 16.82, `TotalAmount` 193.88, discount 3.93), so
  the invoice footed and would have passed Rithum's balance check — the
  defect is only visible against the PO. Net effect: $1.14 under-billed.
  **Nothing was structurally missing**: element-for-element the file matched
  accepted PSI1303944; the only other differences were order-specific values.
- **Upstream root cause:** unconfirmed. $1.00/unit is not a penny adjustment;
  most likely the NAV sales line took a price-list / item-card price instead
  of the 850 price on SO1333498. Worth checking where SO1333498's unit price
  came from before the next Hub order for this item.
- **Fix (applied in _v2)** — all values settled by reference PSI1303944, which
  carries identical figures for the same item at 172.20 / qty 1:
  - `UnitPrice`, `ExtendedItemTotal`, `TotalNetSalesAmount`: 171.20 → 172.20
  - `C300` `AllowChrgAmt`: 2.14 → 2.15 (1.25% × 172.20; `I170` stays 0.43)
  - `CG` `TaxAmount`: 8.43 → 8.48 (5% × 169.62)
  - `ST` `TaxAmount`: 16.82 → 16.92 (9.975% × 169.62)
  - `TotalAmount`: 193.88 → 195.02
  - `TermsDiscountAmount`: 3.93 → 3.95 (2% × 197.60)
  - Left alone: `ST` `TaxPercent` `9.98` (reference shows `9.97`; both are
    NAV roundings of 9.975 and both have been accepted — not evidenced as a
    defect). Eight-line diff, CRLF preserved, no trailing newline.
- **Files:** `4182998_Home Depot.CA Hub 810.xml` →
  `4182998_Home Depot.CA Hub 810_v2.xml`
- **Status:** fixed, awaiting resend and partner confirmation. Paired ASN
  corrected in the same session — see
  [home-depot-ca-hub-856.md](home-depot-ca-hub-856.md) Case 01.

### Case 01 — 810 — totals-mismatch — 2026-08-21
- **Document type:** 810
- **Error type:** `totals-mismatch`
- **Reported by:** Rithum, automated failure notice ("Dear Rithum Partner...") emailed after the file was received
- **Error message:** "Header level balance due does not match calculated value: balance_due = balance_due_calculated with values 47.56, 47.58 ... From file (ISA number or file name): 100048242 Received: 8/14/26 3:47:23 PM EDT ... PO #:538177850 Line #:10 QTY:1 Action:v_invoice ... Status:Shipped"
- **Source file:** `4143542_Home Depot.CA Hub 810.xml` (invoice PSI1317282, PO 538177850, line 10) — untouched
- **Reference file:** n/a — resolved by reconciling the document's own line/tax/allowance figures, no comparison document needed
- **Reference ID:** PO 538177850, invoice PSI1317282
- **Document defect:** `Summary/Totals/TotalAmount` was `47.56`, a 2¢ mismatch against the reconciled balance due. The line item (`TotalNetSalesAmount` 46.00), tax (`TaxAmount` 2.26), and both header `ChargesAllowances` amounts (I170 = 0.11, C300 = 0.57, both `AllowChrgIndicator = A`) were all internally consistent with each other; only the header total was wrong: 46.00 + 2.26 − 0.11 − 0.57 = 47.58, matching Rithum's `balance_due_calculated`.
- **Upstream root cause:** unconfirmed — not yet traced in NAV/ERP. Likely a rounding or truncation slip when the header total was calculated at invoice creation, since every contributing figure (net sales, tax, both allowances) checks out individually.
- **Fix (applied in _v2):**
  - `TotalAmount`: 47.56 → 47.58 (46.00 + 2.26 − 0.11 − 0.57)
  - Nothing else changed — line item, tax, and allowance amounts were all already correct.
- **Files:** `Resolved/4143542_Home Depot.CA Hub 810.xml` → `Resolved/4143542_Home Depot.CA Hub 810_v2.xml` (moved to `Resolved/` once the corrected invoice was confirmed sent)
- **Status:** resolved. Document corrected; upstream root cause in NAV not yet investigated — worth a follow-up if this recurs for this customer.
