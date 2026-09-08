# EDI Failiures XML — working instructions

This folder is where EDI XML transaction failures get diagnosed and corrected,
one failure at a time.

## The process lives in the skill, not here

**Use the `edi-xml-failure-corrections` skill for every EDI failure in this
folder** — a pasted error message, a partner rejection email, a customer
complaint about tax/totals/segments, or a request to log a fix. It is the
authoritative process, and it holds:

- `.claude/skills/edi-xml-failure-corrections/SKILL.md` — the workflow
  (including the triage and reference-first steps), how to match a new failure
  to a past case, the error-type vocabulary, the non-negotiable rules,
  cross-cutting notes, and the Quick index of every case.
- `cases/<customer-slug>-<doctype>.md` — full case history and partner notes,
  one file per customer + document type. `cases/README.md` explains the layout.
- `references/git-and-tracking.md` — branching, PR and worktree cleanup, the
  project-root tracking files (`task-list.md`, `gaps.md`,
  `open-questions.md`), and when a case's files move into `Resolved/`.

This file deliberately does not restate any of that, so there's exactly one
place each rule can be wrong.

## Invariants — these hold whether or not the skill has loaded

The skill triggers on EDI failures. Plenty of work in this folder isn't one —
cleaning up worktrees, moving files into `Resolved/`, answering a question
about a past case. These four apply in every session regardless, because
getting them wrong is unrecoverable rather than merely inconvenient:

- **Originals are never modified or deleted.** The file the user drops in is
  the permanent record of what the failure looked like. Corrections go in a new
  file; a failed correction gets `_v3`, not an edit to `_v2`. Moving a file into
  `Resolved/` is fine — that rule is about content, not location.
- **Corrections are named `<original>_v2.<ext>`** — same folder, same
  extension, incrementing `_v3`, `_v4` for later attempts. No other pattern.
- **Never commit case work directly to `master`.** It's the backup and restore
  point for this project and should only ever hold resolved or explicitly
  logged state. Branch first; merge via PR.
- **Source files arrive in the main checkout** (the project root), even when the
  session is working inside a `.claude/worktrees/...` worktree. If you're in a
  worktree, copy the file in from the main checkout before starting.
