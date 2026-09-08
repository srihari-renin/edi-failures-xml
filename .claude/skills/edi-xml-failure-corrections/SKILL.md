---
name: edi-xml-failure-corrections
description: 'Diagnose and correct EDI XML failures reported by error message or partner email, working in the "EDI Failiures XML" folder. Use when the user pastes an EDI error message or a customer complaint (e.g. missing/incorrect tax, missing/invalid segment, failed transaction, ASN/PO/850/856 validation error) and provides or references a source XML file in that folder. Also use when asked to "log this fix", "add this case to the skill", or "update the EDI failure skill" after a correction is made. This is a skill package: this file stays small (workflow, vocabulary, index) and full case history lives in cases/<customer>-<doctype>.md, loaded only for the customer and document type in play.'
---

# EDI XML Failure Corrections

Skill **package** — not a single file. This file is the router: workflow,
matching rules, error-type vocabulary, cross-cutting notes, and a quick index.
It must stay small. Full case history lives one file per customer under
`cases/`, loaded only when that customer is in play. See `cases/README.md`
for the package layout and sharding rule.

## Workflow (see the folder's `CLAUDE.md` for the authoritative process)

1. User pastes the error message or partner email in chat and places the source
   file in the folder. That file is the "before" record — never edited.
2. **Match against past cases before diagnosing anything.** Customer + document
   type + error type first, then document type + error type across customers —
   see *How to match a new failure to a past case* below. A prior case is a
   starting point, not just background: if one matches, work from its fix.
3. **Triage the shape of the fix** — this decides how you proceed, and getting
   it wrong is the most expensive mistake available here. See *Triage* below.
4. **For recalculation / structural fixes, get a reference document before you
   ask the user to decide anything.** See *Reference-first* below.
5. Diagnose against the source and the reference together with the user.
   Separate the **document defect** (what is wrong in this XML) from the
   **upstream root cause** (what in the ERP produced it). Fixing the XML
   resends this one document; only the upstream fix stops it recurring.
6. Once the fix is agreed, create `<original>_v2.<ext>` in the same folder. If a
   v2 does not resolve it, create `_v3`, `_v4` — never edit a prior version.
   **Fix only the defect the error names or that the diagnosis actually
   evidences.** A reference file is for *diagnosis* — spotting what's wrong and
   confirming the correct value for the field in question — not license to
   "correct" other fields that merely look different from the reference.
   Two documents can legitimately differ (different store, different order,
   coincidental math) without either being wrong. If a reference invoice
   suggests another field might also be off, say so as an open question in the
   case entry rather than changing it — let the customer's complaint or a
   second confirmed data point drive that fix, not inference from one other
   document.
7. Append the case to `cases/<customer-slug>-<doctype>.md` (create the file
   from the template there if this customer+doc-type combination has no
   cases yet), then add one row to the **Quick index** below. Every version
   gets its own entry, including failed attempts. Never write full case
   detail into this file.

### Triage: substitution, or recalculation?

The error type does *not* decide this — the **shape of the fix** does. The same
`tax-missing` error is a substitution when the user hands you the amount and a
recalculation when you have to derive it.

- **Substitution** — the fix replaces a literal value with the correct one.
  No arithmetic, and no other field has to move as a consequence: a carrier
  code, a placeholder string, an address, a qualifier code. Propose the
  correction with your reasoning and implement it once the user agrees. This
  is a proposal to confirm, not a decision to adjudicate — don't spend a
  weigh-the-options question on a field with one plausible value.
- **Recalculation or structural** — the fix computes a number, or changing one
  field forces others to move (tax → total → discount), or a record/segment is
  added or removed. Go to *Reference-first* before asking the user anything
  substantive.

Why the split matters: under a substitution there is one defensible answer, so
asking the user to choose wastes their time. Under a recalculation there are
usually several defensible arithmetics — which base, which rounding, whether a
charge is inside the tax base — and choosing between them from first principles
is guessing dressed up as reasoning. A known-good document from the same partner
turns that guess into an observed fact, so it is worth one round-trip to get one.

### Reference-first

For recalculation and structural fixes, in this order:

1. **Work out what would actually settle it.** Name the distinguishing
   attributes concretely — not "a similar invoice" but "same customer, same
   document type, ship-to in the same tax jurisdiction, and a freight charge on
   the document". You can only ask a useful question once you know what the
   reference has to contain.
2. **Search what the project already holds** before asking: the working folder,
   `Resolved/`, and the **`Reference file:` and `Files:` lines of past case
   entries** — those are effectively an index of every known-good document this
   project has accumulated.
3. **Ask the user, informed — never cold.** State what you found, what it fails
   to cover, and exactly what you need. For example: *"Closest I have is Canac
   PSI1310799 — Quebec, taxed, same ERP, but no freight line and a different
   partner's mapping. Do you have a TimbrMart invoice to a QC dealer with a
   freight charge on it?"* Asking cold risks them handing back a document you
   already had, or one that misses the attribute that mattered.
4. **If they supply one**, use it for every field it genuinely settles and raise
   a decision question only for the gap. A partial match is still evidence — a
   reference that covers the tax rate but not the freight treatment has done
   real work. Record in the case entry which fields the reference settled and
   which you inferred, so a future reader can tell the difference.
5. **If they say they have none**, say so plainly, then run your own diagnosis
   and ask the direct decision questions using the folder `CLAUDE.md`'s format.

**Boundary with step 6.** A reference settles how a field is *derived* — rate,
basis, sign, code, which records exist — for the fields the error implicates. It
is never authority to change a field the error doesn't implicate. Home Hardware
Case 01 is the precedent: the reference settled the 5% tax rate and was
explicitly refused as authority over `TermsDiscountAmount` on a different store.

## How to match a new failure to a past case

Match on this order of specificity:

1. **Customer + document type + error type** — exact match: open
   `cases/<slug>-<doctype>.md` and jump straight to the case.
2. **Document type + error type** across customers — the mechanic usually
   transfers even when the partner differs. Check the Quick index for other
   customers with the same doc type + error type before opening any file.
3. **Error type** alone — the weakest match; re-diagnose, but skim the Quick
   index first for the shape of past problems with that error type.

### Error type vocabulary

Use one of these values so matching stays reliable. Add a new value only when
nothing fits, and add it to this list in the same edit.

| Error type | Means |
|---|---|
| `tax-missing` | Tax should have been charged, document shows zero or no tax |
| `tax-rate-mismatch` | Tax present but wrong rate or wrong amount |
| `tax-jurisdiction` | Wrong tax type for the ship-to jurisdiction (e.g. GST where HST due) |
| `totals-mismatch` | Summary totals do not reconcile to the line items |
| `missing-segment` | A required element/segment is absent |
| `invalid-code` | A qualifier or code value not accepted by the partner |
| `address-error` | Wrong, missing or unmapped address / location code |
| `qty-mismatch` | Ordered / shipped / invoiced quantities disagree |
| `price-mismatch` | Unit or extended price disagrees with the PO |
| `duplicate-document` | Same document number transmitted more than once |
| `credit-sign-convention` | Credit memo (`InvoiceTypeCode=CR`) rejected/mis-posted because a field's sign didn't match the partner's expected convention for credits (e.g. tax sign, price sign) |

## General notes

Cross-cutting rules that apply regardless of customer. Customer-specific
rules live in that customer's `cases/<slug>.md` instead — see each file's own
Partner notes section.

- **Any partner, `tax-missing` on a 810:** in NAV, check the ship-to record's
  **Tax Area Code** and **Tax Liable** fields first — a newly added store /
  ship-to can go live with both blank, which silently zeroes tax on every
  invoice to that address until caught. This was the root cause of
  Home Hardware Case 01 (store `5233-2`, Glenboro MB) and is likely to recur
  whenever a new ship-to is added for any customer, not just Home Hardware.
- **Any partner: after a "penny adjustment" in NAV, always correct the
  invoice price back to the PO.** A penny adjustment on the sales line is the
  established way to clear certain NAV posting errors — it is a deliberate
  remedy, not a mistake. Its known side effect is that the line price no
  longer matches the PO, and the resulting 810 inherits the adjusted figure,
  with tax, totals **and every percentage-based allowance** recomputed
  consistently around it. The invoice therefore foots correctly and passes
  validation with nothing to signal the defect — it is visible only by
  diffing the 810 against its 850. Confirmed on Home Depot.CA MDO 810 Case 01
  (3 cents/unit, $0.18 under-billed). **Treat "correct the invoice afterwards"
  as the second half of the penny-adjustment procedure**, and remember the
  allowances drift with the price, not just the price itself.
- **Any partner, missing pack/line data on an 856:** check *how the order was
  shipped in NAV* before treating it as a mapping defect. NAV sources the
  `<Pack>` and `<ItemLevel>` blocks from a **warehouse shipment's package
  records**. Ship and invoice straight from the sales order — which is the
  usual fallback when a warehouse shipment won't post — and the ASN still
  generates, but with zeroed weights/dimensions/quantities, no SSCC, and **no
  line items at all**. Proven on Home Depot.CA MDO Case 01; the mechanism is
  NAV-side, not partner-side, so expect it for any trading partner. If a
  warehouse shipment has to be abandoned, capture its package records **and**
  its Shipping FastTab (carrier, tracking/PRO number, seal, total weight)
  before deleting it — that data is unrecoverable afterwards.
- **Any partner: an early-payment discount basis is per-partner — confirm it,
  never carry it across.** `TermsDiscountAmount` is 2% of the **tax-inclusive**
  total for Canac (PSI1310799: 2% × 2039.92 = 40.80) and 2% of the **pre-tax**
  total for Home Care TimbrMart (PSI1247205: 2% × 416.38 = 8.33, where
  tax-inclusive would have been 9.57). Two partners, same ERP, opposite bases.
  Home Hardware Case 01 declined to generalise this field from a single other
  invoice and was right to — treat the basis as unknown until you have a
  confirmed document from *that* partner. Practical upshot on a `tax-missing`
  failure: if the partner's basis is pre-tax, `TermsDiscountAmount` is usually
  already correct on the failing invoice, because it never depended on the tax.
- **Any partner: a freight charge is inside the tax base.** A header
  `ChargesAllowances` with `AllowChrgIndicator` = **`C`** (a charge, not the
  far more common `A` allowance) and `AllowChrgCode` `D240` is taxable — tax
  base = `TotalNetSalesAmount` + the charge, and `TotalAmount` = that base +
  tax. Confirmed to the cent on Home Care TimbrMart PSI1247205. This mirrors
  how allowances behave everywhere else in the project (they reduce the base
  *and* the total), and matches Canadian GST/QST treatment of vendor-charged
  freight on a taxable supply. Watch for it: charges are rare here — nearly
  every `ChargesAllowances` record in the project is an `A`.
- **`CarrierProNumber` format `TST-CF 701 ######` → carrier is TST Overland
  Express** (`CarrierAlphaCode` `OVLD`, `CarrierRouting` `TST Overland
  Express`). Confirmed on two different customers with this exact pro-number
  prefix (Canac reference invoice PSI1310799, and Home Hardware Colonial
  Case 02). Still confirm against the BOL/carrier record or the user when
  possible — this is a strong pattern match, not a guarantee for every
  shipment.

## Quick index

Newest first. One row per case *version* (a failed `_v2` and its resolving
`_v3` each get a row). Link goes to `cases/<slug>-<doctype>.md#case-NN`.

| Case | Customer | Doc | Error type | Status | File |
|---|---|---|---|---|---|
| 01 | Home Care TimbrMart | 810 | `tax-missing` | corrected, awaiting confirmation | [cases/home-care-timbrmart-810.md](cases/home-care-timbrmart-810.md#case-01--810--tax-missing--2026-09-08) |
| 01 | Home Depot.CA MDO | 810 | `price-mismatch` | resolved | [cases/home-depot-ca-mdo-810.md](cases/home-depot-ca-mdo-810.md#case-01--810--price-mismatch--2026-08-28) |
| 01 | Home Depot.CA MDO | 856 | `missing-segment` | resolved | [cases/home-depot-ca-mdo-856.md](cases/home-depot-ca-mdo-856.md#case-01--856--missing-segment--2026-08-28) |
| 02 | Home Hardware Colonial | 810 | `invalid-code` | resolved | [cases/home-hardware-colonial-810.md](cases/home-hardware-colonial-810.md#case-02--810--invalid-code--2026-08-28) |
| 01 | True Value | 856 | `missing-segment` | resolved | [cases/true-value-856.md](cases/true-value-856.md#case-01--856--missing-segment--2026-08-26) |
| 01 | Do It Best Hardware | 810 | `missing-segment` | resolved | [cases/do-it-best-hardware-810.md](cases/do-it-best-hardware-810.md#case-01--810--missing-segment--2026-08-26) |
| 01 | Do It Best Hardware | 856 | `missing-segment` | resolved | [cases/do-it-best-hardware-856.md](cases/do-it-best-hardware-856.md#case-01--856--missing-segment--2026-08-26) |
| 02 | Home Depot Canada | 810 | `price-mismatch` | resolved | [cases/home-depot-canada-810.md](cases/home-depot-canada-810.md#case-02--810--price-mismatch--2026-08-25) |
| 01 | Home Depot Canada | 810 | `price-mismatch` | resolved | [cases/home-depot-canada-810.md](cases/home-depot-canada-810.md#case-01--810--price-mismatch--2026-08-25) |
| 01 | Home Care TimbrMart | 810 Credit Memo | `credit-sign-convention` | resolved | [cases/home-care-timbrmart-810-credit-memo.md](cases/home-care-timbrmart-810-credit-memo.md#case-01--810-credit-memo--credit-sign-convention--2026-08-21) |
| 01 | Home Depot.CA Hub | 810 | `totals-mismatch` | resolved | [cases/home-depot-ca-hub-810.md](cases/home-depot-ca-hub-810.md#case-01--810--totals-mismatch--2026-08-21) |
| 01 | Home Hardware Colonial | 810 | `tax-missing` | resolved | [cases/home-hardware-colonial-810.md](cases/home-hardware-colonial-810.md#case-01--810--tax-missing--2026-08-20) |
