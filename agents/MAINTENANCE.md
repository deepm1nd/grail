# Maintenance Phase Process

**MANDATE: Adherence to this guide is not optional. It is a strict requirement.**

Governs corrective and adaptive/perfective maintenance on a Rust project **after** its
Design and Development Phases are otherwise complete — bug fixes, enhancements, and new
features. Distinct from `DESIGN.md` (initial architecture/planning) and `DEVELOPMENT.md`
(initial implementation), which this file assumes have already run at least once. See
`CHANGELOG.md` for this file's own version history.

Scope is corrective and adaptive/perfective maintenance only (ISO/IEC/IEEE 14764's
categories) — a change with no bearing on correctness or capability isn't routed through
this file.

**Dedicated templates, Development's own files untouched.** `maintenance_batch_template.md`,
`maintenance_checklist_template.md`, and `maintenance_prompt_template.md` are Maintenance's
own files — `development_plan_template.md`, `development_checklist_template.md`, and
`dev_prompt_template.md` are never modified or shared to serve this process, to keep the
proven Development pipeline's own files and outputs unaffected.

---

## 1. Relationship to Design and Development

Reuses existing mechanisms rather than inventing parallel ones: branch-per-unit-of-work
with exact-name-verbatim discipline; Claude-drafts/Jules-executes file-relay hand-off, no
direct API/credential integration between sessions (`AGENT_TOOL_POLICY.md`); append-only,
edited-in-place logging (Appendix B/F, same convention as `CHANGELOG.md`); the Complexity
Score formula and Task Template shape (`development_plan_template.md` §8), reused not
reinvented.

**Claude-side Maintenance sessions (M1–M3 below) operate in Advisory Mode** — the same
operating contract `CLAUDE.md` §1 defines for Design Phase (no persistent repo access,
Claude produces downloadable files, the user commits, session-boundary file-delivery
discipline applies). `CLAUDE.md` §1 governs both phases directly; its remaining sections
(§§2–6) stay Design-Phase-specific.

## 2. Starting and Resuming a Maintenance Session

**Starting a new item** — mirrors Design's own opening ("user states a goal or concept,
Claude helps expand"), no tier/type asserted up front since that's decided at M3:

```
Start Maintenance for [projectname]: [one-line description of the bug or idea]
```

**Resuming an already-open item:**

```
Resume Maintenance for [projectname], item [ID]
```

Same file-attachment discipline as any other session boundary — nothing carries over from
chat memory; the resuming session needs the current batch/checklist/prompt files
re-provided.

## 3. Two Entry Points — Never Interleaved

**Already-completed project** (Final Verification passed): run once, before any item is
worked:
1. **Phase Final+1a — Readiness Audit** (read-only): run the full Productization
   Readiness Checklist (`DEVELOPMENT.md` §5.2.4, all nine items) against current state.
   Output is a gap report.
2. **Phase Final+1b — Remediation**: close every gap as a normal task list. **Item 1
   (Regression Scaffold) is a hard gate** — nothing else is worked until it closes. Every
   other gap becomes an ordinary item in the currently open Maintenance Batch, classified
   like anything else at M3.

A project built under `DEVELOPMENT.md` §5.2.4 (v0.9.4+) should find this Audit clean —
a finding here signals a gap in that checklist, not just a sloppy project.

**In-flight project** (Development Plan not yet at Final Verification): finishes its own
Plan first — an urgent bug uses the existing Escalation Trigger mechanism
(`development_plan_template.md` §13), not early Maintenance entry. If it predates v0.9.4,
apply the (one-off, untracked) retrofit trio before its own Final Phase.

## 4. Two Tracks

| | Track A — Corrective | Track B — Substantive |
|---|---|---|
| **Covers** | Tier 1/2 fixes | Tier 3 fixes, features, enhancements |
| **Terse log (once released)** | Appendix B | Appendix F |
| **Touches a Requirement ID?** | Rarely (Tier 2 only) | Usually |
| **Gate before Jules** | Tier 1: none beyond the item entry. Tier 2: user approves the entry. | Always: user approves the entry, including its Requirement ID change. |

Track and Tier are both determined at **M3**, not at intake — an item's type is a working
hypothesis until then.

## 5. Intake

**Bug or idea:** add an item to the currently open Maintenance Batch file — created fresh
if none is currently open, reused if one already is. **A Future Feature idea not yet
approved for active work** is instead logged in the Architecture Specification's own
**§2.6** (Future Features / Deferred Scope) — the same section Design Phase already uses.
**Promotion** pulls a §2.6 item into the open batch (carrying forward its Originating ID,
if reserved, into M2's Requirement ID field) and removes it from §2.6 at that point, per
§2.6's own existing rule. A rejected (not promoted) idea moves to §2.5 (Out-of-Scope
Features) — the same register Design Phase uses, regardless of which phase rejected it.

## 6. The M1–M4 Process

Every item runs all four steps — **depth and content scale by what M3 finds**, not by a
tier asserted in advance. M1–M3 are Claude-side, Advisory Mode (§1); M4's output is
executed by Jules.

### M1 — Elicitation
Bug reproduction (environment, steps, expected/actual), or a feature/enhancement's goal
and story. Scales from one paragraph (an obvious one-line bug) to a fuller elicitation
pass for a genuinely new capability.

### M2 — Requirements, Test, Verification
Which Requirement ID(s) this item touches (existing, modified, or new), what test/
regression coverage applies. **Self-check after M2:** the 9 Requirement Quality
Criteria / Requirement Smell catalog (`DESIGN.md` §4.5), applied only to whatever
Requirement text this item actually touched — mechanical, not a gated audit.

### M3 — Classification, Architecture Synthesis, Impact Assessment
**The load-bearing step — determines Tier, Track, Type, and SemVer recommendation for
every item, light or heavy alike.**
- **Tier/Track:** per §4's table.
- **Type:** `fix` | `feature` | `enhancement` — descriptive, not a separate process lane;
  `feature` and `enhancement` share Track B's rigor once Tier 3.
- **Architecture Synthesis:** real ISO-42010-style content (interface/data-model/viewpoint
  impact) only where the item actually warrants it — a Tier 1/2 fix's root-cause
  statement already *is* its architecture-level analysis; skip the rest.
- **Impact Assessment → SemVer recommendation**, stated explicitly per item:
  ```
  Impact assessment: [what changed, concretely] → Recommended bump: patch | minor | major
  Rationale: [why this is/isn't user-perceptible enough to warrant more than patch]
  ```
  This is a judgment call the user confirms or overrides, not a mechanical formula — a
  10% performance gain and a 2x one are the same *kind* of change with different
  *magnitude*, and magnitude — not category — drives the recommendation.
- **Asset Manifest** (feature/enhancement items with visual/media content only): same
  mechanics as `CLAUDE.md` §3.7 — table (Filename, Type, Repository Target Path,
  Authoritative/Informative For, Authority Level, Provided At), fixed paths
  (`assets/html|images|audio|video/`), never re-attached at a session boundary, only
  referenced by filename.

### M4 — Task Decomposition
Generates this item's tasks, following the standard Task Template shape (Design Refs,
Verification Method, DoD, Submit Point — `development_plan_template.md` §8's shape,
reused directly). **Self-check after M4:** every task has a Verification Method and DoD —
mechanical completeness check, not a gated audit. **Complexity Score
(`development_plan_template.md` §8's formula, reused unmodified) applied across the
batch's full task list** determines Phase count in the generated Checklist — one batch is
at least one Phase, more if the ceiling requires a split, exactly as an over-ceiling
Development phase already gets split.

**One item = one Phase** in the generated `maintenance_checklist_template.md` output —
Entry Criteria from M1/M2's preconditions, Exit Criteria = Verified (§10 below), tasks
from M4, identical Task/DoD/Submit-Point/Session-Log shape as Development's own Checklist.

## 7. Regression Scope by Project Type

A `retest-all` default is always safe but often wasteful; pure selective scope risks
missing cross-cutting impact. Starting points (combine as a project's actual shape
requires):

| Project type | Regression scope starting point |
|---|---|
| Web client/server | Contract tests for touched endpoint(s); E2E only if user-facing surface changed |
| CLI | Unit/integration for touched command(s); output-format stability check if formatted output changed |
| Embedded/IoT | HIL or simulator run for touched subsystem; full retest-all if shared toolchain/HAL touched |
| Multi-container services | Cross-service contract tests for every consumer of the touched interface |
| Headless server + remote client | Backend regression, plus screenshot-diff if rendering is affected |
| Library/SDK | API/ABI compatibility check in addition to the crate's own suite |
| Mobile/desktop/browser extension | Full retest-all often justified — store-review latency makes a missed regression expensive |
| Batch/data pipeline | Re-run against fixture datasets; diff actual vs. expected output |
| Serverless/on-prem/vendor-installed | Check the project's own Spec for deployment-model constraints not listed here |

Regression scope is determined jointly by Claude and the user at M2/M3 — never by Jules
alone, never inferred from the diff after the fact. Jules verifies the stated scope; it
does not independently decide scope during implementation.

## 8. Generated Artifacts and Filenames

Three files, all placeholder-named while the batch is open, renamed together at release
(§11):

| Content | Template | Open (placeholder) name | Released name |
|---|---|---|---|
| Spec-equivalent (M1–M3, per item) | `maintenance_batch_template.md` | `[projectname]_maintenance_open.md` | `[projectname]_[type]_v[N.NN.NN].md` |
| Checklist (M4 output) | `maintenance_checklist_template.md` | `[projectname]_maintenance_checklist_open.md` | `[projectname]_v[N.NN.NN]_checklist.md` |
| Jules hand-off prompt | `maintenance_prompt_template.md` | `[projectname]_maintenance_prompt_open.md` | `[projectname]_v[N.NN.NN]_prompt.md` |

`[type]` is `fix` | `feature` | `enhancement` — assigned at release, reflecting whichever
specific category actually drove the batch's SemVer bump (§11), not assigned at item
creation, since a batch's dominant content isn't known until it's about to ship.

## 9. Hand-off to Jules

The generated Prompt file (§8) points Jules at the generated Checklist and the batch's
M1–M3 content for the item(s) it names — same file-relay pattern as Design→Development, no
direct integration, no shared credentials. Jules implements per the Checklist's tasks,
runs the stated regression scope, and reports back (PR/diff/summary) for the user to relay
into a Verification session.

## 10. Verification

A Claude session reviews Jules's result against the item's M1–M3 content and Checklist:
does it match root cause/scope, was the stated regression scope actually run, does
anything suggest the scope was too narrow. Marks the Phase's Exit Criteria met in the
Checklist. Flags — does not silently fix — anything inconsistent. Does not submit/merge/
tag; that stays user-directed.

## 11. Release Checklist and Versioning

- **Batching at user discretion** — items accumulate in the currently open batch until the
  user decides to cut a release.
- **SemVer bump = highest confirmed recommendation across every item in the batch** (§6
  M3) — an all-Track-A batch is typically patch; any item recommending minor or higher
  makes the whole release at least that; a breaking change is major.
- **Fix-forward only, one supported line** — no concurrent maintenance of older release
  branches at this time. Every Appendix B/F row records its shipping version, which is
  what a future multi-line model would need — adding one later is additive.

**Every release (patch baseline):**
- [ ] Every item's Checklist Phase is Exit-Criteria-met (Verified), or explicitly carried
      over to the next batch (noted, not silently dropped)
- [ ] SemVer bump determined, with rationale rolled up from each item's M3 assessment
- [ ] Appendix B/F rows added for every item, referencing the released batch file + item ID
- [ ] `CHANGELOG.md` entry added; `[Unreleased]` section emptied
- [ ] Repo tagged; batch/Checklist/Prompt files renamed to their released filenames (§8)

**Additional, MINOR or higher:**
- [ ] New/modified Requirement ID(s) reflected in Appendix F, old→new text
- [ ] Any Asset Manifest entries (§6 M3) confirmed logged with fixed repo paths present
- [ ] README updated if the new capability changes Quick Start/Tech Stack/feature list
- [ ] Aggregate regression scope re-confirmed adequate for the whole batch, not just
      per-item

**Additional, MAJOR only:**
- [ ] Breaking change(s) explicitly called out in `CHANGELOG.md` with migration guidance
- [ ] Deprecation/removal notices added where applicable
- [ ] Consumer-facing contract (Architecture Spec §10) reviewed and updated if affected

## 12. Escalation

Stop, summarize, wait (`development_plan_template.md` §13) when: an item's M3
classification is genuinely ambiguous between two SemVer recommendations with materially
different consequences; Jules's implementation reaches further than the Checklist's
stated scope (return to Claude for a scope revision, not unilateral expansion); or
anything that would itself raise a standing safety concern outside this framework's
scope, handled as it would be in any other context.
