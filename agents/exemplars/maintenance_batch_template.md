# Maintenance Batch: [v0.<x>.<y> — or "Unreleased" while still open]

> Companion to `agents/MAINTENANCE.md`. **One file per release batch.** Created when the
> user begins assembling the next release (or kept standing as `..._maintenance_open.md`
> between releases), renamed/archived to reflect the actual version once cut. Contains
> **full detail** — reproduction steps, environment, root cause, regression scope — for
> every item being worked toward this release. Appendix B/F in the Architecture
> Specification hold only a terse pointer row per item, referencing this file and an item
> ID below — full detail lives here, once, not duplicated in both places.
>
> Items not resolved by the time a release is cut **carry over unforced** into the next
> batch file — being open here doesn't force inclusion in an imminent release.

**Batch status:** Open | Released (v0.<x>.<y>, YYYY-MM-DD)

---

## Item BF-0001 — [short title]

- **Kind:** Bug | Feature
- **Track:** A | B
- **Tier:** 1 | 2 | 3 (`agents/MAINTENANCE.md` §5)
- **Status:** Reported/Proposed → Triaged/Approved → Briefed → Fixed → Verified
- **Reported/Proposed:** YYYY-MM-DD, by whom

**For a bug:**
- **Environment:**
- **Steps to reproduce:**
- **Expected behavior:**
- **Actual behavior:**
- **Severity / Priority:** *(kept as two separate fields, never conflated)*

**For a feature:**
- **Scope/description:** *(what this actually does, once past `Proposed`)*
- **Rationale:**

**Filled in at Triage/Briefing (both kinds):**
- **Root cause / feature design:**
- **Requirement ID(s) affected:** added / modified / superseded — old text → new text
  where applicable
- **Regression scope:** *(Claude + user, jointly — `agents/MAINTENANCE.md` §7)*
- **Target branch:** `[exact name — verbatim, per `dev_prompt_template.md` §3]`
- **Instructions to Jules:** *(concrete enough that Jules doesn't re-derive root cause —
  see `maintenance_prompt_jules_template.md` for the hand-off itself)*

**Filled in at Verification:**
- **Matches root cause/scope:** Y/N — [note if N]
- **Regression scope actually run:** Y/N — [note if N]
- **Appendix B/F row added:** Y/N — reference: `[B|F].<version><letter>`

---

*(Repeat one `## Item BF-####` block per item in this batch — sequential ID within this
file, reset per batch file. Do not compress or summarize multiple items into one block;
each item gets its own, however small.)*

---

## Batch Release Checklist (filled in when cutting this release)

- [ ] Every item above is `Status: Verified`, or explicitly carried over to the next batch
      (noted, not silently dropped)
- [ ] SemVer bump determined — highest bump any single item in this batch requires
      (`agents/MAINTENANCE.md` §10)
- [ ] Appendix B/F terse rows added to the Architecture Specification for every item,
      referencing this file by name and item ID
- [ ] `CHANGELOG.md` entry added; `[Unreleased]` section emptied
- [ ] Repo tagged; this file renamed/archived to its released version's filename
