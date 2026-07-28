# Release Plan: [projectname]_release_v[N.NN.NN]_plan

> Companion to `RELEASE.md`. Lives at `dev/release/v[N.NN.NN]/[projectname]_release_v[N.NN.NN]_plan.md`
> — the version folder plus the filename's own `_v[N.NN.NN]` both carry the version
> (redundant on purpose: the filename retains full provenance if the file is ever copied
> out of its folder). Named from the moment this release's RELEASE work opens (mirrors
> `MAINTENANCE.md`'s own batch-open convention: no `_open` placeholder, no rename at
> publish time, except a rare escalation-driven correction). Edited in place across this
> release's RELEASE work, never copied/renamed/"v2'd" (`AGENTS.md` §2.7).
>
> Section numbers below align with `development_plan_template.md`'s equivalent content
> where a parallel exists, so a citation resolves consistently regardless of which document
> governs a given session.

**Release status:** Open | Published (v[N.NN.NN], YYYY-MM-DD)
**Gated on (`RELEASE.md` §6.7):** `MAINTENANCE.md` §11 Release Checklist state for this
version | `DEVELOPMENT.md` Final Verification (first release only)
**SemVer tier this release (drives §7's versioning behavior):** patch | minor | major

---

## §1 — Scope (Step 1: Elicitation output)

- **In-scope Diátaxis quadrants this release:** Tutorials | How-To Guides | Reference |
  Explanation (strike any omitted, per `RELEASE.md` §4)
- **`web/site/` content types this release** (per `RELEASE.md` §5's checklist): [list
  selected — Home, Blog, Community, Playground, Showcase, Comparison, Sponsors, Roadmap,
  Changelog-page, or none beyond Docs]
- **Audience/persona summary:** [distinct from Architecture Spec §2.1's product personas —
  who is this release's documentation actually written for]
- **Seed material consulted:**
  - User Stories tagged with a Diátaxis Destination (Architecture Spec §2.2): [list IDs]
  - Requirements with a User-Facing Description (Architecture Spec §2.3): [list IDs]
  - This release's `CHANGELOG.md` entries flagged as needing a doc update: [list]

## §2 — Step Index

| Step | Title | Entry Criteria | Exit Criteria | Gated On |
|---|---|---|---|---|
| 1 | Elicitation | — (first Step) | §1 above complete, user-confirmed | — |
| 2 | Scaffold | Step 1 Exit | Skeleton builds, matches §1's selections | Step 1 |
| 3 | Terminology/Concept Spine | Step 2 Exit | Glossary/outline settled, user-confirmed | Step 2 |
| 4 | Tutorials | Step 3 Exit | Draft complete for all Tutorial-tagged stories | Step 3 (order-independent w/ 5, 6) |
| 5 | How-To Guides | Step 3 Exit | Draft complete for all How-To-tagged stories | Step 3 (order-independent w/ 4, 6) |
| 6 | Explanation | Step 3 Exit | Draft complete per §1's Explanation scope | Step 3 (order-independent w/ 4, 5) |
| 7 | Reference-Sync | Steps 4–6 Exit **and** `MAINTENANCE.md` §11 / Final Verification | Reference quadrant complete, as-built-accurate | See gating note above |
| 8 | Finalize | Step 7 Exit | Draft content reconciled against Reference, no known inaccuracies | Step 7 |
| 9 | Consistency Audit | Step 8 Exit | No orphaned/drifted terminology, style-guide adherent | Step 8 |

**No compressed formats** (`AGENTS.md` §2.3) — a real generated Plan expands every Step's
own Entry/Exit detail in §3 below, in full; this Step Index is a summary table, not a
substitute for it.

## §3 — Per-Step Detail

*(One `### Step N: <Title>` sub-section per row in §2, each restating that Step's Process
and Output from `RELEASE.md` §6, plus this release's specific instantiation — e.g. Step 1's
actual scope statement, Step 3's actual glossary entries, Steps 4–6's actual story/guide
list. Fully written out per real release, never abbreviated or left as a placeholder per
`AGENTS.md` §2.3.)*

### Step 1: Elicitation
- **Process:** [`RELEASE.md` §6.1, instantiated — this release's actual scope/audience work]
- **Output:** [this release's actual scope statement, cross-referenced to §1 above]

*(...Steps 2–9 follow the same shape, fully expanded for the real release...)*

## §4 — Escalation

See `RELEASE.md` §11 — not restated here. Any ambiguous Step 1 scope call, any discovery
that Step 7's extraction assumption doesn't hold for this project, any Step 9 finding
implying a Step 1/3 revisit, or any hosting/compliance concern per `RELEASE.md` §8 is an
Escalation Trigger — stop, summarize, wait, same as `development_plan_template.md` §13.
