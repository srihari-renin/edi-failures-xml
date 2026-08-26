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
2. Identify the failing file by invoice / PO / document number, and identify a
   **reference file**: a known-good document from the same customer and document
   type, ideally with the same distinguishing attribute (same province/state,
   same terms, same item type). Diff the two before theorising.
3. Diagnose against the source and reference together with the user. Separate
   the **document defect** (what is wrong in this XML) from the **upstream root
   cause** (what in the ERP produced it). Fixing the XML resends this one
   document; only the upstream fix stops it recurring.
4. Once the fix is agreed, create `<original>_v2.<ext>` in the same folder. If a
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
5. Append the case to `cases/<customer-slug>-<doctype>.md` (create the file
   from the template there if this customer+doc-type combination has no
   cases yet), then add one row to the **Quick index** below. Every version
   gets its own entry, including failed attempts. Never write full case
   detail into this file.

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

## Quick index

Newest first. One row per case *version* (a failed `_v2` and its resolving
`_v3` each get a row). Link goes to `cases/<slug>-<doctype>.md#case-NN`.

| Case | Customer | Doc | Error type | Status | File |
|---|---|---|---|---|---|
| 01 | True Value | 856 | `missing-segment` | resolved | [cases/true-value-856.md](cases/true-value-856.md#case-01--856--missing-segment--2026-08-26) |
| 01 | Do It Best Hardware | 810 | `missing-segment` | resolved | [cases/do-it-best-hardware-810.md](cases/do-it-best-hardware-810.md#case-01--810--missing-segment--2026-08-26) |
| 01 | Do It Best Hardware | 856 | `missing-segment` | resolved | [cases/do-it-best-hardware-856.md](cases/do-it-best-hardware-856.md#case-01--856--missing-segment--2026-08-26) |
| 02 | Home Depot Canada | 810 | `price-mismatch` | resolved | [cases/home-depot-canada-810.md](cases/home-depot-canada-810.md#case-02--810--price-mismatch--2026-08-25) |
| 01 | Home Depot Canada | 810 | `price-mismatch` | resolved | [cases/home-depot-canada-810.md](cases/home-depot-canada-810.md#case-01--810--price-mismatch--2026-08-25) |
| 01 | Home Care TimbrMart | 810 Credit Memo | `credit-sign-convention` | resolved | [cases/home-care-timbrmart-810-credit-memo.md](cases/home-care-timbrmart-810-credit-memo.md#case-01--810-credit-memo--credit-sign-convention--2026-08-21) |
| 01 | Home Depot.CA Hub | 810 | `totals-mismatch` | resolved | [cases/home-depot-ca-hub-810.md](cases/home-depot-ca-hub-810.md#case-01--810--totals-mismatch--2026-08-21) |
| 01 | Home Hardware Colonial | 810 | `tax-missing` | resolved | [cases/home-hardware-colonial-810.md](cases/home-hardware-colonial-810.md#case-01--810--tax-missing--2026-08-20) |
