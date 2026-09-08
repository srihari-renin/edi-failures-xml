# Git workflow and task tracking

Read this when starting a case, when a case reaches a stopping point, or when
you need to know where work-in-progress and unresolved caveats get recorded.
The diagnosis workflow itself is in `SKILL.md`.

## Git workflow

Repo: [github.com/srihari-renin/edi-failures-xml](https://github.com/srihari-renin/edi-failures-xml)
(private). `master` is the backup / source of truth — it should always reflect
only resolved (or explicitly logged) case state, so it stays safe to clone or
restore from at any time. That property is the reason for every rule below.

1. **Never commit case work directly to `master`** — even solo. Before starting
   a new failure, branch off an up-to-date `master`:
   ```
   git checkout master && git pull && git checkout -b case/<customer-slug>/<reference-id>
   ```
   `<customer-slug>` matches the `cases/<slug>-<doctype>.md` naming;
   `<reference-id>` is the invoice/PO/control number in lowercase (e.g.
   `case/home-hardware-colonial/psi1312154`). For work not tied to one invoice
   (skill or process changes), use `chore/<short-description>` instead (e.g.
   `chore/branching-conventions`).

   If the session is already sitting on a branch created for it by the harness
   (a `claude/...` worktree branch), leave the name alone rather than renaming
   it — the harness tracks that name for its own cleanup. The PR title and the
   case entry carry the identifying information anyway.
2. Do all of that case's work on its branch: the source file, every `_v2` /
   `_v3` / ... attempt, the case entry in `cases/<slug>-<doctype>.md`, and the
   Quick index row in `SKILL.md`. Commit as you go.
3. When the case reaches a stopping point (resolved, or a failed attempt that's
   fully logged), push the branch and open a PR into `master`:
   ```
   git push -u origin <branch>
   gh pr create --fill
   ```
   Self-merge is fine — no required review for this project — but always via a
   PR, not a local `git merge`, so GitHub keeps a record of what changed per
   case. Opening the PR is part of the documented workflow and doesn't need
   confirmation; **merging into `master` is worth a beat** when the case is
   still awaiting partner confirmation or an unconfirmed upstream root cause —
   ask rather than assume.
4. Merge, then delete the branch (`gh pr merge --squash --delete-branch` or the
   equivalent from the GitHub UI). Squash so `master` gets one clean commit per
   resolved case. This deletes the remote branch and, if you're sitting on a
   normal checkout of that branch, the local one too.
5. **Clean up the worktree after merge, every time.** If the case's work
   happened in a `git worktree add` checkout (per step 7), merging the PR does
   not remove the worktree or its local branch — do both explicitly right after
   the merge:
   ```
   git worktree remove <worktree-path>
   git branch -d <branch>
   ```
   Run these from the main checkout, not from inside the worktree being
   removed. If `git branch -d` refuses because the squash-merge commit isn't
   detected as merged, confirm the PR actually merged into `master` and use
   `-D` — don't skip cleanup and leave the stale branch/worktree around.
6. **Working two cases at once** (you and someone else, or two parallel Claude
   sessions): each just branches from `master` and works independently — no
   coordination needed until merge time. The only files likely to conflict are
   `SKILL.md`'s Quick index table and `task-list.md`, since every case adds a
   row to both; if two branches add a row in the same place Git will flag a
   conflict — resolve it by keeping both new rows, nothing more is needed.
7. **Running several cases in parallel** (e.g. `git worktree add` off different
   branches so more than one session can work at once without switching
   branches in the same checkout): each worktree still follows steps 1–5
   independently on its own branch, and step 5's cleanup once that worktree's
   PR merges. Use the tracking files below so it's visible what every worktree
   is doing.

   Note that OneDrive can hold locks on `.git/worktrees/...`, so git may print
   `Permission denied` while pruning stale worktree metadata for *other*
   worktrees during an unrelated commit. That's noise, not a failed commit —
   check `git log` before treating it as an error.

## Task tracking files

Three files at the project root, separate from the skill package, track
work-in-progress and knowledge that doesn't belong in a case entry:

- **`task-list.md`** — open tasks only, one row per task currently being worked
  (across any worktree/session). Add a row when you start something. When it's
  done, **move the row** into `Resolved/task-list-completed.md` — cut from one
  file, paste into the other, don't leave it in both and don't leave finished
  work in the open file. This is a live coordination board, not a history —
  keeping it short is the point.
- **`Resolved/task-list-completed.md`** — archive of finished tasks, moved here
  from `task-list.md`. Newest first.
- **`gaps.md`** — caveats, known limitations, and errors that couldn't be fully
  resolved, logged for future reference. Not a task list — nothing here
  necessarily needs action. Append whenever an error can't be fully resolved
  and you need to record why, or you notice a limitation that isn't specific
  enough to one case to belong in that case's `cases/<slug>-<doctype>.md` file.
- **`open-questions.md`** — any decision question asked in chat that doesn't get
  answered in the same turn. Log it here before moving on to other work so it
  isn't lost; when picking work back up, check this file first and re-ask
  anything still pending. Remove it, or move it under **Answered** with a
  one-line note, once resolved.

## When files move to `Resolved/`

`Resolved/` isn't only for the completed task list — once a case's corrected
invoice is confirmed **sent to the partner** (not just fixed locally), move that
case's source file and every `_v2`/`_v3`/... attempt into `Resolved/`, together
with any reference file used only for that case's diagnosis. This keeps the
working folder showing only invoices still in play.

Moving doesn't count as editing — the "originals are never modified" rule is
about content, not location — but update the case's **Files** line in
`cases/<slug>-<doctype>.md` to point at the new `Resolved/...` path so the
record stays accurate.
