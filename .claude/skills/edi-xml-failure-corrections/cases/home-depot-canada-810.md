# Home Depot Canada (`820ALLRENINHOL1`) — 810 — Case Log

Covers 810 (invoice) documents only. Other document types for this customer
(850, 856, 846, ...) get their own `cases/home-depot-canada-<doctype>.md`
file — see `cases/README.md`.

Filenames for this trading partner use two labels — "OS" on the `850` POs
and "Special" on the `810` invoices — but both carry `TradingPartnerId
820ALLRENINHOL1`, so they're treated as one customer slug (`home-depot-canada`),
not split into separate customers.

## Partner notes

- **`AllowChrgIndicator = A` on the header `ChargesAllowances` block reduces
  `TotalAmount`.** Balance due reconciles as
  `TotalNetSalesAmount + TaxAmount - Σ(AllowChrgAmt where indicator = A)`.
  Same convention confirmed for [Home Depot.CA Hub](home-depot-ca-hub-810.md) —
  likely a shared convention across all Home Depot trading partners, not
  proven for other customers.
- **The `850` (PO) carries the authoritative unit price and allowance
  amounts for the corresponding `810`.** Both cases below matched the PO on
  `BuyerPartNumber` via the invoice's `PurchaseOrderNumber` field, and the
  `810`'s own totals footed correctly in both cases — the defect was the
  810 drifting from the 850's figures, not an internal math error. Compare
  against the paired 850 before assuming a drifting field is "wrong" in
  isolation.
- **Unconfirmed:** why the `810` drifts from the `850` on some fields but not
  others (only `E210` drifted on one case; `UnitPrice`, `E210`, and `D240`
  all drifted on the other, in different directions each time) — no pattern
  yet in the rounding/computation that would explain it. Root cause not
  traced in NAV. Flag for a NAV/pricing engine review if this recurs.

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

### Case 02 — 810 — price-mismatch — 2026-08-25
- **Document type:** 810
- **Error type:** `price-mismatch`
- **Reported by:** not a partner rejection — found proactively while auditing unit price / invoice price / tax across a batch of 4 Home Depot Canada files (2 `850` POs + 2 `810` invoices) at the user's request. Rithum did not reject either invoice; both foot correctly on their own totals.
- **Error message:** n/a — no rejection notice, this was a self-initiated discrepancy check.
- **Source file:** `4155785_Home Depot Canada Special 810.xml` (invoice PSI1320586, PO 537765507, BuyerPartNumber 129986, ship-to Kanata-7108) — untouched
- **Reference file:** `4106797_Home Depot Canada OS 850.xml` (the paired PO for this invoice, matched via `PurchaseOrderNumber`)
- **Reference ID:** PO 537765507, invoice PSI1320586
- **Document defect:** `UnitPrice`/`TotalNetSalesAmount` matched the PO exactly (196.47). Only the `E210` allowance amount drifted: PO had `4.26`... (see note) — for this case `E210` was `6.97` on the PO vs `7.07` on the invoice, a $0.10 overstatement of the deduction. `D240`, `C300`, `I170` all matched the PO. `TaxAmount` (22.44) and `TotalAmount` (195.02) were internally consistent with the invoice's own (wrong) `E210`, so the document didn't fail validation — it just didn't match the PO.
- **Upstream root cause:** unconfirmed — not traced in NAV. The `AllowChrgPercent` fields on the 850 don't cleanly reproduce the 850's own `AllowChrgAmt` via simple percent-of-unit-price math either, so the discrepancy may originate in how HD's OS-PO pricing engine computes `E210` versus how it posts on the Special-810 invoice, rather than a Renin-side data entry error.
- **Fix (applied in _v2):**
  - `ChargesAllowances[E210]/AllowChrgAmt`: 7.07 → 6.97 (matches PO)
  - `Summary/Tax/TaxAmount`: 22.44 → 22.45 (recomputed: 13% × (196.47 − 0.49 − 2.46 − 6.97 − 13.87) = 13% × 172.68 = 22.4484 → 22.45)
  - `Summary/Totals/TotalAmount`: 195.02 → 195.13 (172.68 + 22.45)
  - `UnitPrice`, `ExtendedItemTotal`, `TotalNetSalesAmount` unchanged (196.47 — already matched the PO)
  - `PaymentTerms/TermsDiscountAmount` (4.38) left unchanged — out of scope; its computation base doesn't match any obvious combination of fields on this invoice (not 2% of `TotalAmount`, `TotalNetSalesAmount`, or any allowance-adjusted variant tried), so it wasn't touched per the "fix only what the diagnosis evidences" rule.
- **Files:** `Resolved/4106797_Home Depot Canada OS 850.xml` (reference, PO) / `Resolved/4155785_Home Depot Canada Special 810.xml` → `Resolved/4155785_Home Depot Canada Special 810_v2.xml`
- **Status:** resolved. Caught before the original PSI1320586 was ever sent — `_v2` was sent to Home Depot instead, confirmed by the user 2026-08-25. Files moved to `Resolved/`.

### Case 01 — 810 — price-mismatch — 2026-08-25
- **Document type:** 810
- **Error type:** `price-mismatch`
- **Reported by:** not a partner rejection — found proactively in the same batch audit as Case 02 above.
- **Error message:** n/a — no rejection notice.
- **Source file:** `4155759_Home Depot Canada Special 810.xml` (invoice PSI1320440, PO 538321703, BuyerPartNumber 172674, ship-to Ellesmere-7001) — untouched
- **Reference file:** `4144401_Home Depot Canada OS 850.xml` (the paired PO for this invoice, matched via `PurchaseOrderNumber`)
- **Reference ID:** PO 538321703, invoice PSI1320440
- **Document defect:** Three fields drifted from the PO, in different directions: `UnitPrice`/`TotalNetSalesAmount` was `120.00` on the invoice vs `120.24` on the PO (−$0.24, short); `E210` was `4.32` on the invoice vs `4.26` on the PO (+$0.06 overstated deduction); `D240` was `8.47` on the invoice vs `8.49` on the PO (−$0.02 understated deduction). `C300` and `I170` matched. The invoice's own totals were internally consistent with its (wrong) figures, so it passed validation despite disagreeing with the PO on three separate fields.
- **Upstream root cause:** unconfirmed — not traced in NAV. Three fields drifting in different directions on the same document argues against a single simple cause (e.g. one wrong lookup value); more likely several independent rounding/mapping steps between the 850 and 810 generation. Worth a NAV/pricing engine review if this recurs — see Case 02 and the Partner notes above.
- **Fix (applied in _v2):**
  - `InvoiceLine/UnitPrice`: 120.00 → 120.24 (matches PO)
  - `InvoiceLine/ExtendedItemTotal`: 120.00 → 120.24 (1 × corrected unit price)
  - `ChargesAllowances[E210]/AllowChrgAmt`: 4.32 → 4.26 (matches PO)
  - `ChargesAllowances[D240]/AllowChrgAmt`: 8.47 → 8.49 (matches PO)
  - `Summary/Totals/TotalNetSalesAmount`: 120.00 → 120.24
  - `Summary/Tax/TaxAmount`: 13.70 → 13.74 (recomputed: 13% × (120.24 − 0.30 − 1.50 − 4.26 − 8.49) = 13% × 105.69 = 13.7397 → 13.74)
  - `Summary/Totals/TotalAmount`: 119.11 → 119.43 (105.69 + 13.74)
  - `PaymentTerms/TermsDiscountAmount` (2.67) left unchanged — same "out of scope, base doesn't reconcile to any obvious formula" reasoning as Case 02.
- **Files:** `Resolved/4144401_Home Depot Canada OS 850.xml` (reference, PO) / `Resolved/4155759_Home Depot Canada Special 810.xml` → `Resolved/4155759_Home Depot Canada Special 810_v2.xml`
- **Status:** resolved. Caught before the original PSI1320440 was ever sent — `_v2` was sent to Home Depot instead, confirmed by the user 2026-08-25. Files moved to `Resolved/`.
