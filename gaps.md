# Gaps, Caveats & Known Limitations

Running knowledge log for things discovered while resolving EDI failures
that don't belong in one case's file — unresolved errors, open risks,
limitations in our own process or in a partner's data, things worth knowing
next time even though nothing was "fixed." This is not a task list; nothing
here necessarily needs action, it needs to be *known*.

Append an entry any time:
- an error can't be fully resolved and you need to record why, so it isn't
  silently dropped;
- you notice a limitation in how the skill or process handles something;
- you find a caveat that isn't specific enough to one case to live in that
  case's `cases/<slug>-<doctype>.md` file (put it there instead if it is).

<!--
Entry template:

### <short title> — <YYYY-MM-DD>
- **Context:** what were we doing when this came up (case #, file, or general)
- **Gap / limitation:** what's actually incomplete, uncertain, or unresolved
- **Impact:** what this means in practice if left as-is
- **Related case(s):** link to case entries if applicable, or "n/a"
-->

### NAV→SPS map passes signed credit-memo charge lines through — Home Hardware CR — 2026-09-14
- **Context:** Home Hardware Colonial 810 Credit Memo PSCM031734 — [Case 01](.claude/skills/edi-xml-failure-corrections/cases/home-hardware-colonial-810-credit-memo.md#case-01--810-credit-memo--credit-sign-convention--2026-09-14)
- **Gap / limitation:** a restocking charge on a NAV credit memo is a negative G/L line (42010, return reason B7), and the 810 map copies that signed amount straight into `AllowChrgAmt` (`C` / `-60.46`). Home Hardware takes the amount as sent, so every credit memo with a negative charge/allowance line will be rejected out of balance by 2× the amount. Not fixed at source; corrected on the document only.
- **Impact:** recurs on any Home Hardware credit memo carrying a restocking charge, charge-back or unapplied discount. Until the map emits `abs(amount)` with the indicator carrying the direction, check the sign of every `AllowChrgAmt` on outbound Home Hardware credits before they go. Also not yet checked whether other recent PSCM* documents to 833ALLRENINHOLD have the same defect.
- **Related case(s):** [home-hardware-colonial-810-credit-memo.md#case-01](.claude/skills/edi-xml-failure-corrections/cases/home-hardware-colonial-810-credit-memo.md#case-01--810-credit-memo--credit-sign-convention--2026-09-14)

### NAV G/L inconsistency on WSH219483 — root cause untraced — 2026-08-28
- **Context:** Home Depot.CA MDO 856, SO1327572 / warehouse shipment WSH219483 — [Case 01](.claude/skills/edi-xml-failure-corrections/cases/home-depot-ca-mdo-856.md#case-01--856--missing-segment--2026-08-28)
- **Gap / limitation:** WSH219483 refused to post with *"The transaction cannot be completed because it will cause inconsistencies in the G/L Entry table."* The standard remedies were tried and all failed: removing the line from the package, removing it from the warehouse shipment line, applying a penny adjustment on the sales order, then re-adding and re-building cartons. The shipment was ultimately deleted and the order shipped/invoiced straight from the sales order. **Why SO1327572 specifically triggered the inconsistency was never established** — the other four sales orders on the same shipment (SO1327637, SO1327656, SO1327577, SO1327612) posted normally, as did the sibling shipment WSH219461 the same day.
- **Impact:** The workaround is known and now documented, but the trigger is not, so it can recur without warning on any order. Each recurrence costs a full manual ASN rebuild, because the sales-order route drops all pack and line data (see the case). Worth a NAV review of what distinguishes SO1327572 from the four that posted.
- **Follow-up 2026-08-28:** the penny adjustment was a deliberate step — per the user it is the only way to clear this error in NAV — and its price side effect is known and accepted, not accidental. It moved the sales line 3 cents off the PO price, which invoice PSI1321830 then inherited (see [810 Case 01](.claude/skills/edi-xml-failure-corrections/cases/home-depot-ca-mdo-810.md#case-01--810--price-mismatch--2026-08-28)) and which was corrected in `_v2`. The point to carry forward is not "avoid the adjustment" but "**the invoice correction is the second half of the procedure**" — and that the percentage-based allowances drift with the price, so they need correcting too. Now recorded as a cross-cutting rule in `SKILL.md`'s General notes.
- **Related case(s):** [home-depot-ca-mdo-856.md#case-01](.claude/skills/edi-xml-failure-corrections/cases/home-depot-ca-mdo-856.md#case-01--856--missing-segment--2026-08-28), [home-depot-ca-mdo-810.md#case-01](.claude/skills/edi-xml-failure-corrections/cases/home-depot-ca-mdo-810.md#case-01--810--price-mismatch--2026-08-28)

### Home Depot.CA MDO 810 `ProductProcessDescription` — meaning unknown — 2026-08-28
- **Context:** Home Depot.CA MDO 810, invoice PSI1321830 — [810 Case 01](.claude/skills/edi-xml-failure-corrections/cases/home-depot-ca-mdo-810.md#case-01--810--price-mismatch--2026-08-28)
- **Gap / limitation:** The invoice carries `ProductProcessDescription` = `ALLITEMS`; the only known-good MDO 810 (PSI1291553) carries `040`. With one reference it's impossible to tell whether `040` is item- or order-specific, or whether `ALLITEMS` is a generic fallback produced by invoicing from the sales order rather than a warehouse shipment. Left unchanged in `_v2` per the "fix only what's evidenced" rule. What the field actually drives on Home Depot's side is also unknown.
- **Impact:** If `ALLITEMS` turns out to be wrong, PSI1321830 will need a `_v3`. Low risk — nothing suggests it is validated — but unresolved.
- **Related case(s):** [home-depot-ca-mdo-810.md#case-01](.claude/skills/edi-xml-failure-corrections/cases/home-depot-ca-mdo-810.md#case-01--810--price-mismatch--2026-08-28)

### TimberMart CR tax sign — upstream cause unconfirmed — 2026-08-21
- **Context:** Home Care TimbrMart 810 Credit Memo, invoice PSCM032060 (PO 106193) — [Case 01](.claude/skills/edi-xml-failure-corrections/cases/home-care-timbrmart-810-credit-memo.md#case-01--810-credit-memo--credit-sign-convention--2026-08-21)
- **Gap / limitation:** The credit memo shipped with `TaxAmount` positive when TimberMart requires it negative. Haven't traced why the source ERP/EDI mapping defaults credit memo tax to positive — could be a per-document-type template setting (would recur on every future TimberMart CR, and possibly other CR-type trading partners with the same convention) or a one-off. Also haven't checked whether any other open/pending TimberMart credit memos have the same defect.
- **Impact:** Until traced, every new TimberMart 810 Credit Memo should be checked for this before sending, not assumed fixed at the source.
- **Related case(s):** [home-care-timbrmart-810-credit-memo.md#case-01](.claude/skills/edi-xml-failure-corrections/cases/home-care-timbrmart-810-credit-memo.md#case-01--810-credit-memo--credit-sign-convention--2026-08-21)
