# Home Care TimbrMart (`5V9ALLRENINHOLD`) — 810 Credit Memo — Case Log

Covers 810 Credit Memo (`InvoiceTypeCode=CR`) documents only. Regular 810
invoices for this customer, if any occur, get their own
`cases/home-care-timbrmart-810.md` file — see `cases/README.md`.

## Partner notes

- **TimberMart CR sign convention (this is the whole ballgame for this doc
  type):** TimberMart's EDI system negates the credit amount for display but
  passes `TaxAmount` through **exactly as sent, unchanged**. Sending
  `TaxAmount` positive (the natural/intuitive value) causes an out-of-balance
  rejection of 2×tax, because their system computes
  `-(gross amount) + (+TaxAmount) ≠ -TotalAmount`. The fix is always the
  same: send `TaxAmount` **negative**; leave the credit/charge amount and
  `TotalAmount` **positive**. See the `timbermart-credit-correction` command
  (`~/.claude/commands/timbermart-credit-correction.md`) for the full
  convention table and known failed attempts (all-positive, all-negative,
  mixed-negative-price all fail differently).
- **Sending a negative price/charge amount is the dangerous failure mode**,
  not just a rejection: TimberMart's system negates it *again* on top
  (`-1 × -X = +X`), so the credit posts as a **positive debit** on the
  customer's account instead of being rejected. If that has already happened,
  it requires a **two-credit fix** (offset credit with a new invoice number,
  then the corrected credit) — see the command doc for sequencing. Always
  verify with the customer whether a debit was already booked before treating
  a rejection as "just resend it."
- **This document type has no `LineItem` records** — the credit amount lives
  in `Header/ChargesAllowances/AllowChrgAmt`, not a per-line `UnitPrice`. The
  same sign convention still applies to `AllowChrgAmt` (positive) and
  `Summary/Tax/TaxAmount` (negative); a `ChargesAllowances`-only structure
  does **not** exempt the document from this rule — it was tried once
  assuming otherwise and still went out of balance.
- **Balance check for this doc type:** `AllowChrgAmt + |TaxAmount| = TotalAmount`.

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

### Case 01 — 810 Credit Memo — `credit-sign-convention` — 2026-08-21
- **Document type:** 810 Credit Memo (`InvoiceTypeCode=CR`)
- **Error type:** `credit-sign-convention`
- **Reported by:** TimberMart AP, by email, after the credit was deleted by their EDI system
- **Error message:** "This credit has been deleted due to out of balance issue. Tax amount of 23.28 should be in negative value. Please resend this credit correctly. PSCM032060"
- **Source file:** `4094325_Home Care TimbrMart 810 Credit Memo.xml` (invoice PSCM032060, PO 106193) — untouched
- **Reference file:** n/a — customer's rejection email named the exact field and required sign; matched directly against the documented TimberMart CR convention (see Partner notes)
- **Reference ID:** PSCM032060 (PO 106193)
- **Document defect:** `Summary/Tax/TaxAmount` was sent positive (`23.28`). TimberMart passes tax through unchanged rather than negating it themselves, so a positive tax on a negated credit throws the balance off by 2×tax, matching the customer's report exactly.
- **Upstream root cause:** unconfirmed — not yet traced in the source ERP/EDI mapping to see why credit memos default to positive tax. Worth checking whether this is a per-document-type template issue (would recur on every future TimberMart CR) or a one-off. Logged in [gaps.md](../../../../gaps.md) as open.
- **Blast radius check:** not yet checked for other open/pending TimberMart credit memos that may have the same defect. Recommend a check before assuming this is isolated.
- **Fix (applied in _v2):**
  - `TaxAmount`: `23.28` → `-23.28`
  - `AllowChrgAmt` (179.04) and `TotalAmount` (202.32) left unchanged — both already positive, matching the required convention.
  - Balance check: `179.04 + 23.28 = 202.32` ✓
- **Files:** `4094325_Home Care TimbrMart 810 Credit Memo.xml` → `4094325_Home Care TimbrMart 810 Credit Memo_v2.xml`
- **Status:** corrected, awaiting send to TimberMart and confirmation it posts as a credit (not yet moved to `Resolved/`).
