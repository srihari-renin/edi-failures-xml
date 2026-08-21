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
- **Files:** `4143542_Home Depot.CA Hub 810.xml` → `4143542_Home Depot.CA Hub 810_v2.xml`
- **Status:** resolved. Document corrected; upstream root cause in NAV not yet investigated — worth a follow-up if this recurs for this customer.
