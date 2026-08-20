# Home Hardware Colonial (`833ALLRENINHOLD`) — 810 — Case Log

Covers 810 (invoice) documents only. Other document types for this customer
(850, 856, 846, ...) get their own `cases/home-hardware-colonial-<doctype>.md`
file — see `cases/README.md`.

## Partner notes

- **Discount basis, UNCONFIRMED / per-store, not per-partner:** on store
  `5320-7` (Winkler), `TermsDiscountAmount` matches 2% of the
  tax-**inclusive** `TotalAmount`. Do **not** generalize this to other
  stores — on store `5233-2` (Glenboro) the same math was ambiguous (tax was
  zero, so net-basis and total-basis calculations coincided) and we
  deliberately left the original discount amount untouched rather than
  guess. Treat discount basis as per-store until confirmed on a second store
  with nonzero tax. — [Case 01](#case-01--810--tax-missing--2026-08-20)

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

### Case 01 — 810 — tax-missing — 2026-08-20
- **Document type:** 810
- **Error type:** `tax-missing`
- **Reported by:** Stacey Koebel (Home Hardware AP), by email, after receiving the invoice
- **Error message:** "I received the invoice today with the taxes missing... would you kindly review your invoice PSI1312154 and advise if taxes should have been applied?" — customer attached a second Home Hardware invoice for the same state/province as a reference.
- **Source file:** `4146318_Home Hardware Colonial 810.xml` (invoice PSI1312154, ship-to Glenboro, MB, store 5233-2) — untouched
- **Reference file:** `4110662_Home Hardware Colonial 810 (Reference).xml` (invoice PSI1308344, ship-to Winkler, MB, store 5320-7) — same customer, same doc type, same province, supplied by the customer
- **Reference ID:** PSI1312154 (PO 24-666409, BOL SS1319799)
- **Document defect:** `Summary/Tax/TaxPercent` and `TaxAmount` were both `0.00` despite a valid `TaxTypeCode` (GS) and correct `TaxID` (835391830) — same tax ID as the reference, which charged 5%. `TotalAmount` was set equal to `TotalNetSalesAmount` (717.64) with no tax added.
- **Root-cause check:** initially inferred the discount basis from the reference invoice — in PSI1308344, `TermsDiscountAmount` 14.08 = 2% × `TotalAmount` 703.86 (tax-inclusive) — and recomputed `TermsDiscountAmount` to 15.07 on that basis in a first pass. **Reverted on review**: the reference is a different store (`5320-7`, Winkler) from the failing invoice's store (`5233-2`, Glenboro), and the original 14.35 on the source document can't actually distinguish "discount on net" from "discount on tax-inclusive total" — tax was zero at the time it was calculated, so both bases give the same figure by coincidence. Decided not to generalize a single other-store invoice into a rule for this store. Discount basis for `5233-2` specifically is **unconfirmed** — left as originally issued rather than guessed.
- **Upstream root cause:** confirmed in NAV — ship-to `5233-2` (6756906 Manitoba Ltd, Glenboro, MB) had **Tax Area Code** and **Tax Liable** both blank on the customer/ship-to record, so no tax calculated on any invoice to that address. Fixed by populating both fields on the ship-to record; new orders for this store will now calculate tax going forward. `5320-7` (Winkler, MB) already had both fields set correctly, which is why the reference invoice taxed properly.
- **Blast radius check:** no open sales orders exist for store `5233-2` at time of fix, and PSI1312154 is the only invoice ever issued to this ship-to — confirmed by checking order history. No other documents need correcting.
- **Fix (applied in _v2):**
  - `TaxPercent`: 0.00 → 5.00
  - `TaxAmount`: 0.00 → 35.88 (717.64 × 5%)
  - `TotalAmount`: 717.64 → 753.52 (717.64 + 35.88)
  - `TermsDiscountAmount`: **left unchanged at 14.35** — deliberately not recalculated. Only the actual defect (missing tax) was corrected; the discount amount was left exactly as the source document had it, since the correct basis for this specific store is unconfirmed (see Root-cause check above). `TermsDiscountAmount` does not feed `TotalAmount` in this format, so leaving it untouched does not affect the total.
  - `TotalNetSalesAmount` (717.64) left unchanged — it is correctly pre-tax on both documents.
  - Not changed: `TotalWeight` is 0.00 on the source (reference shows 210.00) — left as-is, out of scope for this fix; real weight not available. Worth a separate check.
- **Files:** `4146318_Home Hardware Colonial 810.xml` → `4146318_Home Hardware Colonial 810_v2.xml`
- **Status:** resolved. Document corrected, sent to customer, upstream NAV setup fixed, and confirmed no other affected documents exist for this ship-to.
