# Case file layout

This skill is built to hold hundreds or thousands of cases without the root
`SKILL.md` becoming unreadable or expensive to load. It does that by never
putting case detail in `SKILL.md` — only a workflow, vocabulary, and a
one-row-per-case index.

## Sharding rule

- One file per customer **and document type**: `cases/<customer-slug>-<doctype>.md`
  (e.g. `cases/home-hardware-colonial-810.md`,
  `cases/home-hardware-colonial-850.md`). Split by document type from the
  first case for that customer — don't wait for volume, since 810/850/856/846
  cases for the same customer rarely share fields, error types, or partner
  notes anyway, so combining them buys nothing and just gets split apart
  later.
  - Customer slug = the customer name as it appears in the invoice filename,
    lowercased, spaces to hyphens, punctuation dropped (e.g. `Home Depot.CA
    Hub` → `home-depot-ca-hub`, `Orgill US Stores` → `orgill-us-stores`).
  - Doc type = the transaction set number as used in the filename/EDI type
    (`810`, `850`, `856`, `846`, ...).
- Create the file the first time that customer+doc-type combination gets a
  case, using the template block inside any existing `cases/*.md` file
  (Partner notes section + Cases section with the entry template comment).
- **If a single customer+doc-type file still exceeds ~50 case entries**
  (high-volume customer), split further — by year is the default
  (`<customer-slug>-810-2027.md`) unless a clearer split emerges from the
  actual case mix (e.g. by error type). Do this proactively when the
  threshold is hit, not once the file is already unwieldy. Update `SKILL.md`'s
  Quick index links when you split.
- A partner note that's genuinely specific to one document type stays in
  that doc type's file. A note that applies across a customer's document
  types (e.g. a shared ship-to numbering quirk) — duplicate it into each
  relevant file rather than inventing a third shared location; the goal is
  that opening any one case file gives the full picture for that lookup
  without cross-referencing.
- Cross-cutting rules that aren't specific to one customer belong in
  `SKILL.md`'s **General notes** section instead.

## Why customer + doc type, not customer alone

Matching a new failure starts with "which customer and which document type is
this," so that pair is the shard that lets a lookup open exactly one small,
relevant file instead of scanning everything — including everything else that
customer sends, most of which (a different doc type's fields, error types,
and partner quirks) is irrelevant to the failure in hand. Error type is the
next-most-specific match (see `SKILL.md` → *How to match a new failure to a
past case*) and is served by the Quick index table, which stays small and
searchable even at large scale because it holds one line per case, not full
case bodies.
