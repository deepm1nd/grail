# Maintenance Batch: [projectname]_maintenance_open

> Companion to `agents/MAINTENANCE.md`. One file per release batch, holding the
> Spec-equivalent content (M1–M3) for every item headed to this release — full detail,
> not a terse log; Appendix B/F in the Architecture Specification hold only a pointer row
> per item, once released. Renamed to `[projectname]_[type]_v[N.NN.NN].md` at release
> (`agents/MAINTENANCE.md` §8/§11). Task content (M4) lives in the paired
> `maintenance_checklist_template.md`-generated file, not here.
>
> Items not resolved by the time a release is cut carry over unforced into the next batch
> file. Section numbers below (§1–§4, §11) are chosen to align with
> `development_plan_template.md`'s equivalent content, so a citation resolves the same way
> regardless of which document is governing a given session.

**Batch status:** Open | Released (v[N.NN.NN], YYYY-MM-DD, type: fix | feature | enhancement)

---

## Item [ID] — [short title]

- **Kind (working hypothesis until §3):** Bug | Feature/Enhancement idea
- **Status:** Open → Classified → Briefed → Fixed → Verified

### §1 — Elicitation (M1)
**For a bug:**
- Environment:
- Steps to reproduce:
- Expected behavior:
- Actual behavior:

**For a feature/enhancement:**
- Scope/description:
- Rationale:

### §2 — Requirements, Test, Verification (M2)
- Requirement ID(s) touched: existing (cite) | modified (old→new) | new
- Test/regression coverage identified:
- **Self-check (9 Requirement Quality Criteria / Requirement Smell catalog, `DESIGN.md`
  §4.5)** — applied to any Requirement text this item touched: pass | issue found, resolved

### §3 — Classification, Architecture Synthesis, Impact Assessment (M3)
- **Tier:** 1 — Trivial | 2 — Standard | 3 — Architectural/Feature
- **Track:** A — Corrective | B — Substantive
- **Type:** fix | feature | enhancement
- **Architecture Synthesis** (only where the item warrants it — interface/data-model/
  viewpoint impact):
- **Impact assessment:** [what changed, concretely] → **Recommended bump:** patch | minor | major
  **Rationale:**
- **Asset Manifest** (feature/enhancement items with visual/media content only — mirrors
  `CLAUDE.md` §3.7):

  | Filename | Type | Repository Target Path | Authoritative/Informative For | Authority Level | Provided At |
  |---|---|---|---|---|---|
  | | | | | | |

### §4 — Task Decomposition (M4)
Tasks for this item live in `[projectname]_maintenance_checklist_open.md` (this item's own
`## Phase [ID]` block) — not duplicated here. **Self-check:** every task has a
Verification Method and DoD (confirmed in the Checklist file itself).

### Verification (filled in after Jules reports back)
- Matches §1–§3 content: Y/N — [note if N]
- Regression scope actually run: Y/N — [note if N]
- Appendix B/F row added: Y/N — reference: `[B|F].<version><letter>`

---

*(Repeat one `## Item [ID]` block per item in this batch — sequential ID within this file
(`BF-0001`, `BF-0002`, ...), reset per batch file. Do not compress or summarize multiple
items into one block; each item gets its own, however small.)*

---

## §11 — Escalation

See `agents/MAINTENANCE.md` §12 — stop, summarize, wait on genuinely ambiguous
classification, scope creep discovered during implementation, or a standing safety
concern. Not restated here.

## Release Checklist

See `agents/MAINTENANCE.md` §11 — the patch-baseline/minor/major checklist is defined
once there, not duplicated per batch file. Complete it at release time, referencing this
file's items.
