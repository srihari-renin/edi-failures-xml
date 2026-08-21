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

### TimberMart CR tax sign — upstream cause unconfirmed — 2026-08-21
- **Context:** Home Care TimbrMart 810 Credit Memo, invoice PSCM032060 (PO 106193) — [Case 01](.claude/skills/edi-xml-failure-corrections/cases/home-care-timbrmart-810-credit-memo.md#case-01--810-credit-memo--credit-sign-convention--2026-08-21)
- **Gap / limitation:** The credit memo shipped with `TaxAmount` positive when TimberMart requires it negative. Haven't traced why the source ERP/EDI mapping defaults credit memo tax to positive — could be a per-document-type template setting (would recur on every future TimberMart CR, and possibly other CR-type trading partners with the same convention) or a one-off. Also haven't checked whether any other open/pending TimberMart credit memos have the same defect.
- **Impact:** Until traced, every new TimberMart 810 Credit Memo should be checked for this before sending, not assumed fixed at the source.
- **Related case(s):** [home-care-timbrmart-810-credit-memo.md#case-01](.claude/skills/edi-xml-failure-corrections/cases/home-care-timbrmart-810-credit-memo.md#case-01--810-credit-memo--credit-sign-convention--2026-08-21)
