# Case file layout

This skill is built to hold hundreds or thousands of cases without the root
`SKILL.md` becoming unreadable or expensive to load. It does that by never
putting case detail in `SKILL.md` — only a workflow, vocabulary, and a
one-row-per-case index.

## Sharding rule

- One file per customer/trading partner: `cases/<customer-slug>.md`.
  Slug = the customer name as it appears in the invoice filename, lowercased,
  spaces to hyphens, punctuation dropped (e.g. `Home Depot.CA Hub` →
  `home-depot-ca-hub`, `Orgill US Stores` → `orgill-us-stores`).
- Create the file the first time that customer gets a case, using the
  template block inside any existing `cases/*.md` file (Partner notes section
  + Cases section with the entry template comment).
- **When a single customer file exceeds ~50 case entries, split it further by
  document type**: `<customer-slug>-810.md`, `<customer-slug>-856.md`, etc.
  Do this proactively when the threshold is hit, not only once the file
  becomes unwieldy to read. Move that customer's Partner notes into whichever
  new file makes most sense, or duplicate a note across files if it applies
  to more than one doc type — update `SKILL.md`'s Quick index links when you
  split.
- Cross-cutting rules that aren't specific to one customer belong in
  `SKILL.md`'s **General notes** section, not in a customer file.

## Why customer, not date or error type

Matching a new failure starts with "which customer/document is this," so
customer is the shard that lets a lookup open exactly one small file instead
of scanning everything. Error type is the second-most-specific match (see
`SKILL.md` → *How to match a new failure to a past case*) and is served by
the Quick index table, which stays small and searchable even at large scale
because it holds one line per case, not full case bodies.
