# Maintenance — Jules Execution Session Prompt

## Branch — use this exact name, verbatim

**Branch:** `[EXACT_BRANCH_NAME — copied from the item's own "Target branch" field in the
maintenance batch file, character-for-character]`

Check out this exact branch. Do not append a numeric hash, timestamp, or session
identifier. Do not create a different branch, and do not work on the project's default
branch. If this placeholder wasn't filled in, stop and ask rather than guessing.

## What to implement

**Item `[BF-####]`** in **`docs/[project_name]_maintenance_[batch_identifier].md`**. Read
that item's full entry before doing anything else — root cause/design, Requirement
ID(s), regression scope, and "Instructions to Jules" are all there. **Do not re-derive
root cause yourself; implement against what's briefed.** If the entry seems incomplete or
internally inconsistent, stop and report — do not fill gaps with your own judgment on a
Tier 2/3 item (Tier 1 items are small enough that reasonable local judgment on pure
implementation detail is fine; root cause and scope are still not yours to redefine).

## Context — attached or must be requested, not assumed

- **Architecture Specification** (as-built, all numbered files) — for the Requirement
  ID(s) this item cites, if any.
- **The maintenance batch file itself** (see above) — do not proceed without it.
- Any other project file the batch item's own entry references. **Do not assume any file's
  existence or content beyond what's actually attached** — if something the entry
  references wasn't provided, ask.

## Constraints

- Fix genuinely broken things you find while implementing the item's stated scope — that's
  in scope. Do **not** expand scope beyond what the item's entry states; if the real fix
  needs a larger regression scope or touches more than the entry anticipated, stop and
  report back for a scope revision rather than silently proceeding wider.
- Do not edit the Architecture Specification, the maintenance batch file, `CHANGELOG.md`,
  or any Appendix B/F row yourself — flag anything that looks like it needs one of those
  updated; a later Claude-side Verification session (`agents/MAINTENANCE.md` §9) makes
  those edits.
- A license/security-scan finding is a stop-and-report, never a silent fix or a new
  `deny.toml` exception of your own judgment.
- **Do not submit.** Leave changes as local, uncommitted edits (or committed-but-unmerged,
  per whatever this project's own submit convention is) until the user specifically
  directs a submit.

## Completion

Run the regression scope stated in the item's entry. Report back a summary: what was
implemented, what was fixed along the way, and the result of the regression scope run.
Do not update the batch file or any Appendix B/F row yourself — that happens in
Verification, back in a Claude session.
