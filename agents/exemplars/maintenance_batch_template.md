# Maintenance Batch: [projectname]_[type]_v[N.NN.NN]

> Companion to `agents/MAINTENANCE.md`. One file per release batch, holding the
> Spec-equivalent content (M0–M3) for every item headed to this release — full detail,
> not a terse log; Appendix B/F in the Architecture Specification hold only a pointer row
> per item, once released. **Batch composition is fixed at open time (`MAINTENANCE.md`
> §5) — no drip-feed — so this file's own filename already carries its real target
> version from the moment the batch opens** (`agents/MAINTENANCE.md` §8/§11): no `_open`
> placeholder stage, no rename step at release. The sole exception is a rare, escalation-
> driven mid-batch correction (`agents/MAINTENANCE.md` §8's exception), which is a genuine
> version change, not routine drift. Task content (M4, full-path items only) lives in the
> paired `maintenance_checklist_template.md`-generated file, not here.
>
> Section numbers below (§1–§4, §11) are chosen to align with
> `development_plan_template.md`'s equivalent content, so a citation resolves the same way
> regardless of which document is governing a given session.

**Batch status:** Open | Released (v[N.NN.NN], YYYY-MM-DD, type: fix | feature | enhancement)
**Provisional SemVer bump (computed once at batch-open, `MAINTENANCE.md` §8/§11):** patch | minor | major

---

## Item [ID] — [short title]

- **Path (decided at M1, mechanical — not a working hypothesis):** Lightweight | Full
- **Status:** Open (M0) → Triaged (M1) → [Full path only: Classified (M3) →] Briefed →
  Fixed → Verified

### §0 — Elicitation (M0)
- Request as presented by the user:
- Discussion/fleshed-out scope (Claude + user, interactive):
- **User confirmation — completeness of scope:** Y/N
- **User confirmation — approval of discussed result:** Y/N
> No file is presented at this stage — this section is filled in retroactively once M1
> exits, as a record of what M0 established, not drafted live during the discussion.

### §1 — Impact Triage (M1)
Five mechanical yes/no questions (`agents/MAINTENANCE.md` §6, M1), read against §0 above:

| # | Question | Answer |
|---|---|---|
| 1 | Adds/modifies/removes a Requirement ID? | Y/N |
| 2 | New external interface, data-model/schema field, or component/module boundary? | Y/N |
| 3 | Cross-cutting — touches more than one existing architectural component? | Y/N |
| 4 | Changes the public API / consumer contract (Spec §10)? | Y/N |
| 5 | Requires a new dependency/infra service not already preferred/approved? | Y/N |

- **Outcome:** All-no → **Lightweight path** (provisional SemVer bump stated below,
  proceed straight to Jules artifacts). Any-yes → **Full path** (User Story generated
  below, proceed to §2).
- **Lightweight path — provisional SemVer bump:** patch | minor | major
- **Full path — User Story ID(s) generated this step:** `US-[DOMAIN]-[NNN]` (appended to
  Architecture Spec `02_user_stories`, real project numbering — not a separate
  Maintenance-specific scheme)

*(§2–§3 apply to full-path items only. A lightweight-path item's record ends at §1 above
and proceeds directly to the Jules Checklist/Prompt.)*

### §2 — Requirements, Test, Verification (M2) — *full path only*
- Requirement ID(s) touched: existing (cite) | modified (old→new) | new
- Test/regression coverage identified:
- **Self-check (9 Requirement Quality Criteria / Requirement Smell catalog, `DESIGN.md`
  §4.5)** — applied to any Requirement text this item touched: pass | issue found, resolved

### §3 — Classification, Architecture Synthesis, Impact Assessment (M3) — *full path only*
> **This is the earliest point any file content in this batch is presented to the user**
> (`agents/MAINTENANCE.md` §6, M3) — nothing above this section for a full-path item, and
> nothing at all for a lightweight-path item, is shown as a drafted file before this point.
- **Tier:** 1 — Trivial | 2 — Standard | 3 — Architectural/Feature
- **Type:** fix | feature | enhancement
- **Architecture Synthesis** (only whichever of `06_viewpoints`/`07_interfaces_and_stack`/
  `08_constraints_and_roadmap` the M1 question(s) that fired actually implicate):
- **Impact assessment:** [what changed, concretely] → **Recommended bump:** patch | minor | major
  **Rationale:**
- **Asset Manifest** (feature/enhancement items with visual/media content only — mirrors
  `CLAUDE.md` §3.7):

  | Filename | Type | Repository Target Path | Authoritative/Informative For | Authority Level | Provided At |
  |---|---|---|---|---|---|
  | | | | | | |

### §4 — Task Decomposition (M4) — *full path only*
Tasks for this item live in `[projectname]_v[N.NN.NN]_checklist.md` (this item's own
`## Phase [ID]` block) — not duplicated here. **Self-check:** every task has a
Verification Method and DoD (confirmed in the Checklist file itself). *(A lightweight-path
item's Checklist Phase is generated directly from §1's exit, with no separate M4 pass.)*

### Verification (filled in after Jules reports back)
- Matches briefed content (§0–§3, or §0–§1 for a lightweight item): Y/N — [note if N]
- Regression scope actually run: Y/N — [note if N]
- **Escape valve fired?** (`agents/MAINTENANCE.md` §9/§12) Y/N — if Y, note whether this
  was a lightweight-path item reclassified after Jules found real impact
- Appendix B/F row added: Y/N — reference: `[B|F].<version><letter>`

---

*(Repeat one `## Item [ID]` block per item in this batch — sequential ID within this file
(`BF-0001`, `BF-0002`, ...), reset per batch file. Do not compress or summarize multiple
items into one block; each item gets its own, however small — including lightweight-path
items, which still get a full block even though most of it is a fast "N/A, lightweight
path" pass.)*

---

## §11 — Escalation

See `agents/MAINTENANCE.md` §12 — stop, summarize, wait on genuinely ambiguous
classification (at M1 or M3), scope creep discovered during implementation (including a
lightweight-path item found to have real impact — the escape valve, §9), or a standing
safety concern. Not restated here.

## Release Checklist

See `agents/MAINTENANCE.md` §11 — the patch-baseline/minor/major checklist is defined
once there, not duplicated per batch file. Complete it at release time, referencing this
file's items. Since this file's own filename already carries its real target version
(assigned at batch-open), release is a confirmation pass, not a rename pass — absent
`MAINTENANCE.md` §8's sole mid-batch-correction exception.
