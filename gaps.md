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

### NAV G/L inconsistency on WSH219483 — root cause untraced — 2026-08-28
- **Context:** Home Depot.CA MDO 856, SO1327572 / warehouse shipment WSH219483 — [Case 01](.claude/skills/edi-xml-failure-corrections/cases/home-depot-ca-mdo-856.md#case-01--856--missing-segment--2026-08-28)
- **Gap / limitation:** WSH219483 refused to post with *"The transaction cannot be completed because it will cause inconsistencies in the G/L Entry table."* The standard remedies were tried and all failed: removing the line from the package, removing it from the warehouse shipment line, applying a penny adjustment on the sales order, then re-adding and re-building cartons. The shipment was ultimately deleted and the order shipped/invoiced straight from the sales order. **Why SO1327572 specifically triggered the inconsistency was never established** — the other four sales orders on the same shipment (SO1327637, SO1327656, SO1327577, SO1327612) posted normally, as did the sibling shipment WSH219461 the same day.
- **Impact:** The workaround is known and now documented, but the trigger is not, so it can recur without warning on any order. Each recurrence costs a full manual ASN rebuild, because the sales-order route drops all pack and line data (see the case). Worth a NAV review of what distinguishes SO1327572 from the four that posted — the penny adjustment applied during troubleshooting is itself a candidate contributor and should be ruled in or out.
- **Related case(s):** [home-depot-ca-mdo-856.md#case-01](.claude/skills/edi-xml-failure-corrections/cases/home-depot-ca-mdo-856.md#case-01--856--missing-segment--2026-08-28)

### TimberMart CR tax sign — upstream cause unconfirmed — 2026-08-21
- **Context:** Home Care TimbrMart 810 Credit Memo, invoice PSCM032060 (PO 106193) — [Case 01](.claude/skills/edi-xml-failure-corrections/cases/home-care-timbrmart-810-credit-memo.md#case-01--810-credit-memo--credit-sign-convention--2026-08-21)
- **Gap / limitation:** The credit memo shipped with `TaxAmount` positive when TimberMart requires it negative. Haven't traced why the source ERP/EDI mapping defaults credit memo tax to positive — could be a per-document-type template setting (would recur on every future TimberMart CR, and possibly other CR-type trading partners with the same convention) or a one-off. Also haven't checked whether any other open/pending TimberMart credit memos have the same defect.
- **Impact:** Until traced, every new TimberMart 810 Credit Memo should be checked for this before sending, not assumed fixed at the source.
- **Related case(s):** [home-care-timbrmart-810-credit-memo.md#case-01](.claude/skills/edi-xml-failure-corrections/cases/home-care-timbrmart-810-credit-memo.md#case-01--810-credit-memo--credit-sign-convention--2026-08-21)
