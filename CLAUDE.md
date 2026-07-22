# CLAUDE.md — Design & Maintenance Phase Working Arrangement

See `CHANGELOG.md` for full version history (human reference only; agents skip it).

Governs Claude's operation of the **Design Phase** of this repository, per `AGENTS.md` and
`agents/DESIGN.md`, and, for **§1 (Operating Mode: Advisory) only**, the Claude-side
sessions of **Maintenance Phase** (`agents/MAINTENANCE.md` M1–M3) — both are Claude
sessions with no persistent repo access, governed by the same Advisory Mode contract.
Scope for Design Phase: producing the Architecture Specification, Development Plan, and
Development Checklist only — not Development Phase execution. §§2–6 below are Design-Phase-
specific and do not apply to Maintenance sessions; `agents/MAINTENANCE.md` governs
Maintenance-specific content directly. Where this file conflicts with a future revision of
`AGENTS.md`/`agents/DESIGN.md`/`agents/MAINTENANCE.md`, those win; this is a
working-arrangement summary, not a replacement.

---

## 1. Operating Mode: Advisory

Per `AGENTS.md` §1.5.2, not Autonomous:

- No persistent repo access. Claude can't read repo state, commit, or verify prior files
  exist unless pasted, uploaded, or linked in-conversation.
- Claude produces deliverables as downloadable files; **the user commits them.** Wherever
  `AGENTS.md`/`agents/DESIGN.md` says "the agent must commit"/"ensure repository state,"
  read: Claude prepares, the user commits. `agents/DESIGN.md` §6 Phase Completion is
  satisfied once the user has actually committed — Claude can't verify this and won't claim it.
- Session-Initialization steps assuming shell access (`scripts/start_services.sh`, etc.)
  don't apply; Claude won't simulate running them.
- If a mandate assumes access Claude doesn't have, Claude says so rather than skipping or
  improvising silently.
- **`AGENTS.md` §2.1's Selective Reading Mandate and No Broad Repository Scan Mandate do not
  apply to this session.** Both are scoped explicitly to Development Phase (Jules) sessions.
  Claude reads Architecture Spec, Plan, and related files as broadly as a given Step's
  synthesis genuinely requires — whole-file reads are expected and normal here, not a
  violation. If a session catches itself narrating compliance with either mandate (e.g.
  reaching for targeted `grep`/`sed` extraction "per the Selective Reading Mandate"), that
  narration itself is the signal something is misapplied — stop and read normally.
- **Every Step, and every repeated run of a Step (user-requested per §3.9, or automatic,
  e.g. a Step 7 re-audit per §3.11), runs in its own separate chat session — a hard rule.**
  Step 2 (User Story Elicitation) is one session, not split into named passes (§3.4). Nothing
  "earlier in this conversation" can be referred back to once a new session starts; whatever a
  new session needs from a prior Step must be re-provided directly (uploaded/pasted/linked),
  same as any other file dependency in Advisory Mode. Consequence: **every Step Approval Gate
  (§3.4) is also a session-boundary packaging point** — presenting a step's output for approval
  always means packaging every file and handoff note the next session could need, not just the
  step's headline deliverable. See §3.10 for what this requires as files (not chat text), §3.11
  for Step 7 re-runs specifically (Step 9's backtrack mechanics, `agents/DESIGN.md` §5.9, mirror
  §3.11), and §3.12 for the open-items review every gate must also carry forward.

---

## 2. Scope, Templates, and Project-Specific Constraints

**Artifacts (3 total, per user direction — never a 4th):** Architecture Specification spans
**8 files, one per originating Design Step 1–6** (§4 for the mapping/rationale — supersedes
any earlier 3–5-file grouping). Development Plan spans 3–5 files, ~1000+ lines each.
Development Checklist is one file unless content genuinely warrants a split (flag and propose
before splitting). This sizing is a user scoping choice, not an `agents/DESIGN.md`
requirement — it determines packaging, not gate count. The Spec's finer 8-way split vs. the
Plan's coarser 3–5 is deliberate: each Spec file tracks exactly one Design Step's own
session-versioning history (§4), which only works cleanly at one-file-per-Step granularity;
the Plan has no equivalent per-step authorship history to preserve.

**Templates (all four provided, binding):**
`agents/exemplars/architecture_specification_template.md`,
`agents/exemplars/development_plan_template.md`,
`agents/exemplars/development_checklist_template.md`,
`agents/exemplars/dev_prompt_template.md` (complex/multi-phase Rust variant).
Plan and Checklist stay in lockstep (one checklist line per task DoD item). A later template
revision means Claude conforms existing documents to it and flags the restructuring as a
Major Change (§3.6) — never silently.

**Stable IDs**, assigned at first draft, never renumbered (`US-`, `REQ-`, `TEST-`,
`THREAT-`, `ADR-`, task IDs per template schemas). A rejected/merged item's ID is retired, not
reassigned — this is what keeps cross-document traceability links from breaking.

**No separate Requirements & Traceability document.** Every such reference (template or
conversation) resolves to the Architecture Spec's Traceability content (template §3 — see §4
for which physical file currently holds it). Templates are corrected accordingly.

**File-split boundaries follow each template's own top-level sections**, grouped toward the
file-count target in force for that artifact — see §4 for the Architecture Spec's concrete,
Step-aligned grouping and naming convention.

### 2.1. Rust/WASM-Specific Constraints

Full detail: `agents/RUST_PREFERENCES.md` (naming/keywords, type-system/ownership mapping,
WASM/Web Worker concurrency, edition and MSRV policy), `agents/PREFERRED_DEPENDENCIES.md`
(Rust library policy, checked at Step 5), `agents/PREFERRED_TOOLS.md` (dev tool policy,
including MSRV-determination mechanics), `agents/PREFERRED_SERVICES.md` (infra service
policy — Docker deployment, `deploy/` conventions). This project is a Rust web framework: a
design goal is **multithreading wherever possible, native and WASM**, via Web Workers — not a
constraint to design around. Requirement language using null/inheritance/exception framing
that doesn't map onto `Option<T>`/traits/`Result` is a Requirement Smell (§3, Step 3) needing
translation, not pass-through. `agents/PREFERRED_TOOLS.md` is relevant here (dev tool/MSRV
policy); `agents/SCRIPT_RULES.md` is not (Advisory Mode runs no scripts). **For any project
with an ESP32/ESP-IDF embedded component,** `agents/ESP32_ESPIDF_RUST_BUILD_GUIDE.md` is the
binding toolchain/build reference (target triples, `.cargo/config.toml`, `tokio`-on-ESP-IDF
configuration) — consult it at Step 5 alongside `agents/PREFERRED_DEPENDENCIES.md`'s and
`agents/PREFERRED_TOOLS.md`'s Embedded sections.

### 2.2. Visual & Media Asset Input

Where a project has a human-facing UI, the user supplies reference **assets** — HTML,
images, and optionally audio/video — tracked as first-class, filename-stable artifacts across
both Design (this file) and Development, not consumed once and discarded. Full mechanism:
§3.7. Physical tracking record: the **Asset Manifest**, a table inside the Architecture
Specification (§3.7 states which file currently owns it under §4's Step-aligned split).

---

## 3. Core Mechanisms and the Gated Workflow

`agents/DESIGN.md` §5 defines a **9-step gated lifecycle** — no skipping, no combining, no
proceeding past a gate without explicit approval. §§3.1–3.2 define the single, fixed
procedure (RCD/RATS) shared across every step but 7 and 9; §3.4 lists the steps, stating
only what's specific to each.

### 3.1. The Standard Procedure: Research → Draft, then RATS

**Every step except 7 and 9 follows one fixed procedure — no toggle, no mode, no
Tailoring.** There is no longer a mechanical/substantive split at the batch level: *all*
core step content generation is research-backed by default; only genuine open
items/assumptions/questions run through individual resolution.

**Step 1 — RCD (Research, Competitive-research, Draft).** For the step's core
deliverable (User Stories, requirements, test cases, viewpoints, etc.):
1. **Research:** general web research on the subject matter.
2. **Competitive research:** how comparable/competitive systems solve the same problem.
3. **Draft:** generate the full batch in one pass, informed by 1–2 — not drafted first
   and researched after. For large sets, sub-batch into groups of **~50 items**; each
   group completes this full procedure (RCD → bulk-accept → RATS, below) before the next
   group starts.

**Step 2 — Bulk-accept pass.** Present the batch as a whole. The user accepts in bulk in
one reply. This is expected to close the great majority of the batch — RATS below applies
only to what bulk-accept didn't settle: genuine assumptions, flagged items, and open
questions the agent could not simply decide while drafting.

**Step 3 — RATS (Research, Analysis, recommendation, Table, Selectable-options), per
residual item.** For each item left after bulk-accept:
1. Research (general + competitive), same as RCD's research step, scoped to that item.
2. Full prose analysis and recommendation — uncompressed, complete reasoning.
3. The same content in table form: **Item — Choices — Recommendation**.
4. A selectable-options prompt (§3.2) for that item, with short recognizable per-choice
   text (the prompt UI truncates; the analysis and table above already carry the full
   content — the prompt is purely the input mechanism).

**Step 4 — Terminal outcome, every item, no exceptions.** Each item resolves to exactly
one of:
- **Resolved** — settled now.
- **Deferred to Step N** — N must be ≥ the current step; dormant, not "open," until Step N
  is reached (§3.12). If the right target step isn't yet obvious, defer to the nearest
  step that plausibly governs the decision type and flag that placement as provisional.
- **Future Feature** — recorded per Spec §2.6 (originating ID, description, deferral
  reason, dependency if known).
- **Rejected** — recorded in the Open Items Register (§3.12), tagged `Rejected`, ID
  retired per the stable-ID rule — considered, not silently dropped.

**Research is mandatory by default for every substantive item**, not gated behind a
narrow trigger list. As a guide to what's substantive (a genuine decision) versus
mechanical (no real ambiguity — an ID-uniqueness check, a keyword-collision check): a
purely mechanical check that has no real decision content is not RATS'd. The four
historical trigger categories below remain useful *examples* of what commonly needs
research, but are not an exhaustive gate — **anything not cleanly matching them is still
researched by default, not skipped for lack of a matching category:**
  - **A — checkable/time-sensitive factual claims:** a specific tool/crate/library/version,
    or a "current best practice" claim that could be stale.
  - **B — under-determined design parameters with no objectively correct value:** research
    grounds the choice in what comparable systems converged on.
  - **C — option-space completeness:** is Claude aware of the full option set, or might a
    search surface a newer/domain-specific approach it would otherwise miss.
  - **D — user-facing capability/feature-surface decisions:** opportunity-seeking —
    whether a best-in-class/competitive option exists that wouldn't otherwise get proposed.

**Recommendation first, labeled.** Option #1 in any table/prompt is Claude's
recommendation, explicitly labeled — position never silently implies endorsement.

### 3.2. The Selectable-Options Convention

Applies wherever Claude uses the selectable-options tool for a RATS item.

- **Three layers, every time:** (1) full prose research/analysis; (2) the full Item —
  Choices — Recommendation table; (3) only then the selectable-options prompt — the input
  mechanism for a decision the user is already fully informed about.
- **Standing "Defer" option** — resolves to Step 4's Deferred-to-Step-N outcome, never
  silently dropped.
- **Tool-capacity ceiling: max 4 options.** Defer is 1 standing slot; a
  "provide analysis" option is included only if room remains and the analysis wasn't
  already shown in full above (normally it was, so this is rarely needed). With 3+
  substantive options, state analysis inline in framing text rather than adding more
  slots; with 4+ genuinely distinct options, first try to consolidate to ≤3 (usually
  possible — 4+ often means the question is underspecified) before falling back to plain
  text with Defer noted as available by reply.
- **3+ surviving options is expected to be rare** — flag it when it happens, since real
  analysis should usually collapse apparent 3-way choices to 2.

---

### 3.4. The 9 Steps

**Every step below except 7 and 9 runs the standard procedure (§3.1): RCD-drafted batch →
bulk-accept → RATS on residuals → terminal outcome per item.** No toggle, no autonomy
preset — this is fixed. The table states only what's specific to each step.

| # | Step | Output | Specifics |
|---|---|---|---|
| 1 | Concept Intake & Context Mapping | Architecture file `_01_introduction` (Draft) | Highest-cost step for an unflagged wrong guess — every later step inherits it. Deep web + competitive research is mandatory, aiming at best-in-class/competitive-advantage/novel-capability framing; an honest "no meaningful competitive landscape, here's why" is acceptable, a fabricated comparison is not. **Actively solicits screen mockups, reference HTML, brand/image assets, and (if relevant) audio/video assets, as early as possible** (§3.7/§2.2). |
| 2 | User Story Elicitation | Architecture file `_02_user_stories` | Always runs, in full, regardless of project size — no skip path. Single session; the story list and each story's Interaction Sequence are drafted and closed together. |
| 3 | Requirement Decomposition | Architecture file `_03_requirements` | 3-pass decomposition (Functional → Logical → Detailed) plus the 9-criteria/Requirement-Smell check (`agents/DESIGN.md` §4.5) as part of core-batch generation. |
| 4 | Test Identification | Architecture file `_04_test_strategy` | Test cases derived per requirement; the 9-criteria table and Rust Requirement Smell catalog checked as part of core-batch generation. |
| 5 | Verification Feasibility | Architecture file `_05_verified_traceability` | Rust dependencies checked against `agents/PREFERRED_DEPENDENCIES.md` (preferred used directly; Forbidden never proposed; Requires-Approval/unlisted is a RATS item); dev tools/`agents/PREFERRED_TOOLS.md`, infra services/`agents/PREFERRED_SERVICES.md`; ESP32/ESP-IDF components cross-checked against `agents/ESP32_ESPIDF_RUST_BUILD_GUIDE.md`; **sets the workspace MSRV** (`agents/RUST_PREFERENCES.md` §0) — flagged placeholder here, relocated to `_07_interfaces_and_stack` §6 at Step 6. Claude never certifies "technical sufficiency" unilaterally. |
| 6 | Final Architecture Synthesis (ISO 42010) | Architecture files `_06_viewpoints`, `_07_interfaces_and_stack`, `_08_constraints_and_roadmap` (new); `_05_verified_traceability` (finalized) | Mostly recombination of already-approved content — less new drafting. One live judgment call: formal notation vs. prose per element, stated briefly so the user can override without re-litigating. Covers all 4 mandatory viewpoints. Asset Manifest migrates to permanent home in `_06`'s §4.13. |
| 7 | Spec Audit & Phase-End QA | Final Deficiency Audit Report | **No RCD/RATS here — by design.** Adversarial independence from the rest of the process, including Claude's own prior work. Checks, on Claude's own analysis: every User Story maps to a requirement; every requirement atomic; no elided/summarized content; no gap forcing a stub downstream; every Asset Manifest entry referenced with a prose description and, for mockups, a
logged Authority Level; every filename conforms to §4; **no file carries a front-of-file metadata box (§4.3) — a
finding here is Trivial per §3.11's tiering**; **the Open Items Register contains only valid, not-yet-reached Deferred items — any Resolved/Future Feature/Rejected entry still sitting on the Register, or any item with no terminal outcome at all, is itself a finding** (§3.12). **Every finding tagged Trivial or Substantive (§3.11) — all-Trivial fixes in place, same session, no backtrack; any Substantive finding → Step 7 Backtrack Workflow (§3.11).** Zero findings → proceed to Step 8. |
| 8 | Development Plan & Checklist | Development Plan + Checklist + Dev Prompt + draft README + draft `.gitignore` + draft `ci.yml` + draft `THIRD_PARTY_LICENSES.md` | Environment/config facts (toolchain, CI, local setup) elicited directly; unspecified items with a reasonable default proposed once per Plan via RATS, not once per phase. ****Phase Sizing:** `Score = (task_count × 1) + (new_public_interfaces × 2) + (cross_file_tasks × 2) + (cross_task_dependencies × 1.5)`, default ceiling **≤15** (a Code/Verify-split task counts as 2 tasks). **Shown as its own computed Complexity Score column in the Phase Index (§6.1), never left blank or only implied by Task Count**; over-ceiling requires a recorded override note in the same cell. **Frontend targeted interleaving** where a UI exists: each screen/component's frontend task sits in the same phase as its real backend/data dependency. **Per-task Design Refs, Submit Points, and per-phase Session Unit are populated at drafting time** (`agents/exemplars/development_plan_template.md` §6.1/§8), not left as stubs — Design Refs cite the specific Spec file/section/item each task derives from; the mandatory Code/Verify split is derived mechanically from each task's Verification Method. **Drafts the project README** (overview/stack/roadmap) — Development Phase's Phase 0 task reviews/confirms/enhances it. **Generates `ci.yml`** from `agents/CI.md`'s skeleton, using Step 5's CI Stage Applicability findings, and **initial `THIRD_PARTY_LICENSES.md` content** from Step 5's License Disclosure Artifact finding — both reviewed/confirmed at Development Phase 0 (`agents/DEVELOPMENT.md` §5.2.2/§5.2.3). **Deliverable:** `agents/exemplars/dev_prompt_template.md` → `[projectname]_dev_prompt.md`, produced once, reused every Development-Phase session. |
| 9 | Plan & Checklist Audit | Plan & Checklist Audit Report | **No RCD/RATS here either — mirrors Step 7's independence.** Runs `agents/exemplars/development_plan_template.md` §15 directly as an audit checklist: every Core requirement traced, no orphan citations, every phase has Entry/Exit Criteria, every task has a Verification Method and DoD, Phase Dependency Graph acyclic, Checklist/Plan lockstep, every filename conforms to §4, Build Order fidelity, Frontend Targeted Interleaving where a UI exists, **every phase's Complexity Score recomputed from its own listed tasks and checked against the §6 ceiling (any mismatch, or any over-ceiling phase with no recorded override, is a finding)**, and **the Open Items Register contains only valid, not-yet-reached Deferred items — same check as Step 7 (§3.12), re-verified here in case anything slipped through since.** **Zero findings closes the Design Phase. Any finding classifies as (A) Plan/Checklist-only** — reopen Step 8 alone — **or (B) Spec-originating** — reopen the relevant Spec step, re-clear Step 7, then return to Step 8. Repeats until clean. See `agents/DESIGN.md` §5.9. |

Every step ends with **STOP, present output, await explicit `APPROVED`** — never combined,
never skipped. Per §1, every step (and
every repeated run) is its own session — a step's gate output is always a complete file
package for the next session (§3.10), and always includes the §3.12 accumulated open-items
review.

---

### 3.5. Scope: a Three-Way Question, Not Binary

"Is this in scope" has a real third answer: wanted, just not in this build. Claude never
collapses that into a plain exclusion.

- **Default case — ask which is meant:** not wanted at all (Spec §2.5, Out-of-Scope) vs.
  wanted but deferred (§2.6, Future Features) — materially different downstream handling, so
  Claude doesn't guess. Once "deferred" is confirmed, record it in §2.6 per that section's
  mandate (originating ID if any, sufficient description, deferral reason, known
  dependency/precondition) — not just a conversational mention. If the user already states
  which is meant in the same message, don't ask redundantly (§3.2's early-resolution
  exception).
- **Feature-level stackable/incremental choices (e.g. MVP-floor decisions) default to Future
  Features**, stated explicitly ("the richer level goes to Future Features unless you'd rather
  drop it"), overridable in the same reply. This default is for genuine feature/product-scope
  choices only.
- **Implementation-level choices are a distinct category** — an algorithm/strategy choice for
  an already-in-scope requirement isn't a scope decision (the requirement is in scope
  regardless; what's decided is *how*). These route to Spec **§4.15** (deferred, noted only —
  no commitment) or **§4.16** (deferred, with an extension point Claude must actually specify
  in §4.4/§4.9, not just index) — Claude asks which is meant when unclear, sub-grouped by
  originating ID either way.
  - **Claude actively weighs "ship simple + commit an extension point" within the mandatory
    pre-analysis itself (§3.3)** — not only after the user has chosen — especially where an
    option carries real scope/effort impact, since the retrofit-later-vs.-cheap-extension-now
    cost is itself part of what the analysis must surface.

### 3.6. Backtracking & Major Changes

Per `agents/DESIGN.md`, the user may return to any previous step, most often when a later
step reveals an earlier step's content — flagged or not — was wrong. For normal forward
iteration (i.e. *not* a Step 7 or Step 9 finding — §3.11 for Step 7's mandatory case,
`agents/DESIGN.md` §5.9 for Step 9's analogous one), Claude: (1) names the originating step
and specific content at issue, rather than patching around it; (2) presents — doesn't
unilaterally decide — the choice between a local patch vs. reopening the earlier step, since
that determines how much upstream work needs re-approval; (3) if reopened, preserves all
already-approved later content, revisiting only as needed once the earlier step is
re-approved (additive-only, no wholesale discard); (4) flags it as a Major Change whenever it
materially changes scope/requirements/architecture — not just when convenient. **Major
Changes generally** are flagged explicitly, by name, the moment Claude recognizes one — never
buried in a diff or folded silently into the next deliverable.

#### 3.6.1. Post-Audit Fix Pass

An explicit, per-instance opt-out from the normal multi-session backtrack — available for
**both** an ordinary backtrack above **and** a Step 7/Step 9 finding (§3.11,
`agents/DESIGN.md` §5.9), which otherwise always requires the full workflow. (Formerly
"Surgical Fix Override" — renamed because "surgical" invited exactly the wrong instinct:
treating the fix as narrowly scoped to the one finding that triggered it. It is not. See
point 5 below.)

1. Claude flags the Major Change/backtrack requirement as normal, naming the originating
   step and content.
2. The user may request a fix pass in plain language (e.g. "let's do a post-audit fix").
3. **Claude gives a real recommendation, not a formality** — either agreeing with reasoning,
   or recommending against with a thorough explanation of what the full multi-session
   backtrack catches that this shortcut doesn't (separate sessions and individual gate
   approvals per intervening step), plus a best-effort, in-session-only check of the
   immediate blast radius (not the guaranteed full sweep a real backtrack performs).
4. The user decides with one of two **exact** phrases:
   - **`BACKTRACK APPROVED`** — normal full multi-session backtrack workflow, no override.
   - **`POST AUDIT FIX`** — proceeds as below.
5. **What the pass actually compresses is session count and gate friction, not
   thoroughness or scope.** Claude still fixes the originating content and works forward
   through every step between it and Step 6 (or, for a mid-step trigger, through whatever
   cascading effects exist) — same real work as a normal backtrack, just in one session
   instead of several, without an intermediate approval per step. **Every finding in the
   triggering audit report is fixed — all of them, not just the one named when the pass was
   requested.** If, while fixing an authorized finding, Claude discovers an additional defect
   in the same document(s) — regardless of whether it predates this session, and regardless
   of whether it relates to the finding that triggered the pass — that defect is fixed in the
   same pass too, not flagged for a future auditor to verify independently. "I found X but
   left it alone because it wasn't the finding I was authorized to fix" is never a valid
   outcome of a Post-Audit Fix Pass — everything discovered is in scope for the same reason
   the original findings are: the document is being brought into conformance, not
   incrementally patched. The only two valid outcomes for any discovered issue are: fixed in
   this pass, or raised to the user *before* the pass concludes because it genuinely requires
   a judgment call Claude cannot make alone (see §3.11's Trivial/Substantive split for what
   does and doesn't rise to that bar).
6. **Mid-step trigger:** documented as a brief section in this session's own handoff note,
   carried forward unmodified through every subsequent handoff note until the next Step
   7/9 audit actually runs — no new file created for this.
7. **Audit-originated trigger (a Step 7 or Step 9 finding):** the fix starts at the earliest
   originating step and works forward through every intervening step's own file, through
   Step 6, resolving cascading effects, every finding from the same audit report, and any
   incidentally discovered defect per point 5 — all in one session. At session end, package
   every touched file exactly as at a normal post-Step-6 handoff, plus a handoff note
   describing the fix work performed under this pass, as though Step 6 had just completed
   normally. This hands off into a **new Step 7 (or Step 9) session** using the existing
   `pass2`/`pass3` naming convention (no new variant) — **mandatory, never waivable**,
   re-auditing from scratch independent of the fix session's own account.
8. **Major Change Notification still fires regardless of which path is chosen.**

---

> **The former "Autonomy Toggle" section is removed.** There is no interactive/autonomous
> mode choice and no Tailored/Full-Autonomous preset — §3.1's standard RCD/RATS procedure is
> the single, fixed workflow for every step but 7 and 9; Step 2 always runs in full
> regardless of project size. All subsequent sections in this document are renumbered down
> by one accordingly.

### 3.7. Visual & Media Asset Input: Discovery Source and Tracked Artifact

The user may provide **HTML, images, and (where relevant) audio/video** showing or
constituting how the system's UI should look, sound, and behave. This plays two roles that
must both be honored: a **discovery source** (revealing data-model concepts, missing
entities, or requirements no verbal description surfaced), and a **tracked artifact** carried
by filename through Design and into Development, landing at fixed repository paths for the
development agent to consume directly.

**Authoritativeness differs by asset type — deliberate, non-uniform:**
- **HTML is presumptively authoritative for structure and behavior.** Per the user's own
  statement that HTML input is "highly representative of the desired final appearance,"
  Claude treats a provided HTML file as the default source of truth for a screen/component's
  markup structure, layout, conditional visibility, validation behavior, and exact copy
  text — not merely informative styling guidance to reinterpret freely. Deviating from what an
  HTML asset specifies is itself a flagged item (§3.1), same as any other departure from
  stated input.
- **Images are informative, not structurally binding**, treated as a **design-token source**:
  branding, iconography, typography, color palette, pattern/texture tiles, plus any other
  graphical info they carry. Feeds the Spec's Design System content (§4.13) rather than
  dictating layout structure the way HTML does.
- **Audio/video assets** are logged in the Asset Manifest (below) and their intended usage is
  captured in requirements/UX content as normal, but Claude does not "consume" their content
  directly the way it reads HTML/images — tracked for the development agent, not analyzed by
  Claude for discovery.

**Persistent ask, from Step 1 until satisfied — UI mockup.** The moment a UI/frontend
component is identified (Step 1 at the latest, but possibly a later step), a standing
obligation activates: **every step thereafter explicitly asks for at least one mockup**
(any form — hand sketch, wireframe, reference HTML, competitor screenshot) until the user
has supplied one, not just once at Step 1. **Never a gate** — no step's approval waits on
this; the ask repeats but never blocks. Satisfied the moment ≥1 mockup exists; from then on
the agent may still request *updates* as new features/capabilities emerge (Proactive
UI-Impact Flagging below), but the mandatory per-step ask stops.

**Authority Level — recorded, never inferred.** Whatever the user supplies, they state one
of: **Authoritative** (build to this) · **Authoritative-with-changes** (build to this, noted
deltas) · **Conceptual** (mood/direction only) · **Firm-layout-content-forthcoming**
(structure locked, copy/data pending) · or another label the user states. Logged as its own
Asset Manifest column (below) — the type-based Authoritativeness rule above (HTML
presumptively authoritative, images informative) is the *default absent a stated label*; a
user-stated label always wins over that default.

**Proactive UI-Impact Flagging.** Independent of the ask above: whenever a later step's own
work (new requirement, capability, or architectural decision) implies a UI change — a new
page, view, setting, or structural change not visible in the current mockup — the agent
flags it explicitly at that step, same as any other gap (§3.1), rather than silently
absorbing it into the mockup's existing shape. Mirror case of the completeness check below
(mockup → spec gap); this direction is spec/feature → mockup gap.

**Auth/Authorization addendum.** Where an authentication or authorization component exists,
the same persistent per-step ask applies to a **sign-in/sign-up mockup** specifically. Where
the component is *authorization* (roles/permissions, not just identity), the Spec must
additionally define **where and how roles are assigned, permissioned, and revoked** — the
actual page/setting/admin surface, not merely that roles exist (§4.8 Security Architecture
and §4.13 UX are the two homes for this).

**Delivery channel:** same as any other Advisory Mode input (§1) — attached to a message or
made available via the project space — distinct from, and preceding, the fixed repository
paths below, which govern where the *same* files land for the development agent once
Development Phase begins.

**The Asset Manifest — mandatory tracking record.** The moment any asset is provided, at any
step, Claude logs it: a table with columns `Filename | Type (html / image / audio / video) |
Repository Target Path | What It's Authoritative/Informative For | Authority Level (mockups
only, per above) | Provided At (Step)`. Filenames are **immutable once logged** — Claude
never proposes a rename; a collision or ambiguity is a flagged item for user resolution,
never silently disambiguated by Claude picking a new name. Repository target paths are
fixed: `assets/html/`, `assets/images/`, `assets/audio/`, `assets/video/` — the same
paths/filenames the user will populate in the repo for the development agent, so any asset
referenced anywhere in the Spec or Plan resolves unambiguously with zero translation between
Design and Development. Until Step 6 relocates it to its permanent home in `_06_viewpoints`
§4.13, the Manifest accumulates inside whichever file is current at the step the asset was
provided (§4's Step-aligned mapping) — the same flagged-placeholder-then-relocate pattern
used for the Step 5 MSRV decision.

**The Manifest row is never the only place a mockup lives.** Distinct from the file-handoff
exclusion below: every logged mockup gets a **prose description** in the Spec (§4.13) and,
where it drives task decomposition, the Plan — information architecture, key flows, and
notable structural details a filename alone can't convey — not merely a manifest row and a
"see attached." A one-line placeholder ("per mockup X") without that description is itself a
flagged gap.

**Triggers a completeness check whenever provided, at any point** — not only at Step 1.
Claude checks already-approved Data Dictionary/requirements content against what the visual
input implies. Any gap is a flagged item run through RATS (§3.1) like any other residual
item — never silently patched into already-approved content, however obvious the gap seems.

**Both image and HTML are reviewed when both are provided for the same screen/component —
neither substitutes for the other:** HTML exposes behavior/state/content an image can't show;
an image shows visual layout/design tokens raw markup doesn't make obvious. Each is checked
for what only it reveals.

**Frontend framework/crate implications.** Where HTML input implies a specific Rust/WASM UI
framework capability, that choice still goes through Step 5's normal
`agents/PREFERRED_DEPENDENCIES.md`/`agents/PREFERRED_TOOLS.md` feasibility check like any
other dependency — visual input motivates the need but does not bypass approval.

**Assets are never carried forward as files in a handoff note (§3.10) — a deliberate
exception to §3.10's general "anything crossing a session boundary is a file" rule.** During
Design, a provided asset is reasonably authoritative only for informing UI vision as it bears
on whichever step is currently using it — not a durable input the next session needs
re-attached. By Development, the actual asset will likely be refined/modified somewhat from
whatever was shown during Design, so re-bundling the Design-time copy at every handoff would
just propagate a stale version. Concretely:
- A step's own session may request the asset directly from the user (per the delivery channel
  above) if that step's work is affected by it — a live input to that session, not something
  inherited from a prior handoff.
- The handoff note and Architecture Specification content **reference** the asset (filename,
  Asset Manifest entry, what it's authoritative/informative for) — never re-attach or
  re-package the asset file itself.
- The Architecture Specification and Development Plan state, wherever a UI element depends on
  an asset, that the (possibly-updated) asset **will be available in the repository's
  `assets/` directories at Development Phase** (fixed paths above) — the actual file is a
  Development-Phase input, not a Design-Phase handoff artifact.
- The Asset Manifest itself (the tracking table) *is* carried forward normally, as ordinary
  Architecture Specification content (§4.1) — only the underlying asset files are excluded
  from handoff packaging.

### 3.8. No Agent-Side Certification of Technical Sufficiency

Claude cannot certify technical sufficiency unilaterally at any step (§3.4 Step 5 states this
explicitly; it holds everywhere). Research (§3.3, §3.4 Step 1) narrows the substantive gap
between Claude's proposal and the right answer — it never closes it, since real
infrastructure, budget, and team constraints remain unverifiable from inside this chat. More
rigor produces a better-informed proposal, never a certified one.

### 3.9. Requesting More Depth ("Not Enough — Another Pass")

After any Steps 1, 2, 3, 4, 5, 6, or 8 gate (not 7 or 9 — see §3.4, §3.11, `agents/DESIGN.md`
§5.9; both audit steps use their own backtrack workflow instead), the user may request a
deeper pass rather than a correction — e.g. "not detailed enough, another pass," "double the
number of stories," "more detail in the interface specification." Distinct from the Grouped
Closing Protocol (§3.2), which handles *correctness*; this handles *insufficiency*. The user
may scope narrowly (a viewpoint, a phase) or broadly (the whole step), and may give a concrete
metric — Claude treats a given metric as the actual target, not a vague cue to add a little
more. Each repeated run is its own new session (§1), named `pass2`, `pass3`, etc. in its
handoff-note filename (§4) — the re-touched Architecture file keeps its own independent
version history, simply bumped again at that session's end (§4's versioning rule).

### 3.10. File Delivery for Anything Crossing a Session Boundary

Per §1's session-per-Step reality: anything a *later* session needs to resume, review, or
approve must be an actual downloadable file — never left as chat text to copy manually.
Covers, at minimum: the handoff note itself (below); the Architecture Specification file(s)
current as of session end — per §4, each Step owns and directly edits one numbered file, so
"deliver the Step's output" and "deliver the current file version" are the same action. In
short — **anything presented at a Step Approval Gate is a file.** Content that only matters
within the single session that produced it (e.g. back-and-forth resolving a Grouped Closing
Protocol stage before it's finalized) may stay as chat text — the next session only needs the
*settled* result. **Exception: user-supplied UI/media assets (HTML/image/audio/video) are
never re-packaged into a handoff, per §3.7 — only referenced by filename.**

**Handoff notes are themselves a separate `.md` file** — never inline chat text — named:

```
[projectname]_design_handoff_stepN_passX.md
```

`N` = step number, `X` = pass identifier: `1` for a step's first run, incrementing
(`pass2`, `pass3`, ...) for a repeated run (user-requested per §3.9, or automatic, e.g. a
Step 7 re-audit per §3.11). Step 2, now a single session (§1/§3.4), uses this same scheme —
there is no `passA`/`passB` variant anywhere. Examples:
`myproject_design_handoff_step1_pass1.md`, `myproject_design_handoff_step7_pass2.md` (a
second Step 7 audit after a backtrack remediation).

A handoff note is load-bearing, not a courtesy summary: since the next session has no memory
of the current one, starting it requires *actually providing* the handoff note file plus the
current version of whatever Architecture file(s) it hands off — announcing "continue from
Step 3" without attaching `_03_requirements` at its current version and the handoff note
gives the new session nothing to resume from.

**Every handoff note MUST open with an explicit, unambiguous Next Action directive.** A
session that receives files but no imperative instruction has, in practice, reported "no
explicit ask" and offered the obvious next step as a question instead of starting it — an
observed failure that wastes a round-trip. The directive is the note's first line, exact form:

```
## Next Action
Start Step N[, Pass X].
```

(e.g. `Start Step 4.`, `Start Step 7, pass 2.`) — a command to execute immediately, not a
suggestion; the receiving session opens with the Step's own work (§3.4's row for that Step),
not "what would you like me to do?" A note without this line is incomplete, corrected before
handoff, same as a missing file-list entry.

**The handoff note MUST explicitly enumerate every file the next session needs** — a complete
list, not just this session's own output. Includes earlier-session files still relevant — e.g.
Step 4's note names `_01_introduction`, `_02_user_stories`, `_03_requirements` (current
versions) alongside Step 4's own `_04_test_strategy`, since Step 4 reads all of them — so the
user knows what to gather without reconstructing the dependency chain. A note that only names
this session's own output, silently omitting earlier-session files still needed, doesn't
satisfy this. **This same file list is also stated directly in the chat response**, same turn
as the note — not only inside the file — so the user can see what to gather without opening it.

**Interim files — presented for review, never version-bumped, never packaged as handoff.**
Whenever Claude presents a file for the user's review *before* the step is approved (which
may happen several times across a RATS resolution cycle), that file is an **interim
file**: same base name and current version as last approved, plus a **lowercase-letter
suffix, no separator** (`v3a`, `v3b`, `v3c`, ...) appended directly to the version — never a
new integer version, never a handoff-note/package delivery. Interim files may be freely
revised within a session; the letter simply increments per interim revision shown, purely so
the user and Claude can refer to "the one I sent you" unambiguously mid-review. **No
handoff note, no file-list packaging, and no version-integer bump happens until the user
replies with exact `APPROVED`** (`AGENTS.md` §3.1) for the step. This is deliberate: a prior
scheme bumped and packaged files during the *proposal* itself, which left the next session
unable to tell whether a `v4` file represented an approved result or merely the last
proposal shown — interim lowercase suffixes remove that ambiguity structurally.

**On receipt of exact `APPROVED`: staleness check, then version bump, then handoff —
in that order, as the only point any of this happens.** Claude enumerates every
Architecture file touched this session (including files belonging to a step other than the
current one — a backtrack session or any §3.6 revisit touches files outside its "home"
step, and those are subject to this exactly as the current step's own file is) and:
1. Confirms every touched file conforms to the §4 naming convention — corrected now, not
   flagged for later.
2. Bumps each touched file's version by exactly one **integer**, dropping any lowercase
   interim suffix — the approved file is always a plain-integer version (`v4`, never
   `v3c`), regardless of how many interim revisions preceded it.
3. Produces the handoff note and packages every touched file at its new integer version —
   this is the file delivery point (§3.10 above), now confirmed as approved content rather
   than a still-open proposal.
A file listed as touched but hidden from this bump, or a handoff package assembled before
`APPROVED` was received, is a defect. Step 7's audit re-checks naming compliance
project-wide as part of its own independent review.

**The next session verifies its inputs against the handoff note's manifest before
proceeding.** Where a handoff note exists (any session resuming from a prior one — not a
fresh Step 1), Claude checks files actually provided against the note's own enumerated list —
a concrete comparison, not independent judgment of what the step "should" need. If something
named is missing, Claude says so and asks for it directly, rather than proceeding incomplete
or silently assuming. Mirrors `agents/exemplars/development_plan_template.md` §11's Session
Handoff Protocol, applied to Design Phase sessions too.

**Required inputs, minimized: the numbered Architecture Spec files actually needed, the Dev
Plan files (once they exist, post Step 8), and the handoff note — nothing else, by
default.** Two artifacts that used to be separate required files are now embedded instead,
specifically to keep this list short:
- **The Open Items Register lives inside `01_introduction`** (§4.1's Step 1 file, present
  from the first gate onward) — not a standalone file, so it's never separately listed or
  requested; it travels automatically with whichever Architecture files a session already
  needs.
- **An Audit Report (Step 7 or Step 9) is embedded directly in the backtrack handoff note's
  own text when the audit found anything** (§3.11) — the backtrack session needs the
  findings to act on, and folding them into the note it already receives avoids a second
  required file. On a **clean** audit (zero findings), the report is presented in chat at
  the gate and is not a required input to Step 8/the next session — nothing to carry
  forward when there's nothing to act on.

**Any additional file a step genuinely needs beyond this default set is an exception, not a
standing option** — Claude names the specific file and the concrete reason it's needed, and
proceeds only once the user approves adding it. This keeps the ordinary case (numbered Spec
files + Plan files + handoff note) the only thing a user has to gather without being asked,
while still allowing a real edge case to surface explicitly rather than silently expanding
what gets requested every time.

---

### 3.11. The Step 7 Backtrack Workflow

Step 7 (Spec Audit) is, by design (§3.4), a pure **finder**, never a fixer — no RCD/RATS,
never patches content itself. This is what makes its independence meaningful. Consequence:
**a Step 7 finding is never
resolved via the ordinary local-patch-vs-reopen choice in §3.6/`AGENTS.md` §2.1** — every
finding requires reopening the step that should have produced or caught it, working forward
through Step 6, whether via the full multi-session workflow below or the compressed
single-session **Post-Audit Fix Pass** (§3.6.1) — there is no "local patch at Step 7" to
choose between, since Step 7 itself never touches content, and a Post-Audit Fix Pass still
mandatorily ends in a fresh Step 7 session (except the all-Trivial fast path immediately
below, which needs no backtrack at all).

**Every finding is tagged a Severity at the moment it's found — Trivial or Substantive —
never left untagged:**
- **Trivial:** mechanically unambiguous, no judgment call, no risk of masking a deeper issue
  — a duplicate/orphaned section, a broken internal cross-reference, a filename/naming-
  convention violation, a stale version number or inline `File N vM` citation. If fixing it
  could plausibly be wrong two different ways, it is not Trivial.
- **Substantive:** anything else — a genuine ambiguity, a missing/incorrect requirement or
  traceability link, a Content Continuity or Anti-Stub gap, a phase-sizing problem, or
  anything requiring judgment about correctness, scope, or intent.

**All findings Trivial, none Substantive:** no backtrack, no new session, no reopening an
earlier step's approval. Claude fixes every Trivial finding directly, in place, in this same
Step 7 session — editing each finding's owned file — then presents the corrected content
alongside a short fix log (finding → file → what changed) at the gate. The gate proceeds
normally from there: STOP, present, await approval to Step 8. This exists precisely so a
handful of small mechanical defects don't trigger the full re-litigation cycle below.

**Any Substantive finding present (regardless of how many Trivial findings accompany it):**
the normal Step 7 gate does not close as an approval gate. Instead:

1. **Map every finding to its originating step** — Trivial and Substantive alike; Trivial
   findings are not fixed separately from this workflow once a Substantive one is present,
   they're folded into the same pass. For each finding, identify which step
   (1–6) should have produced/caught the content — and, per §4, which numbered Architecture
   file that step owns.
2. **Determine the single earliest originating step across all findings.** If findings trace
   to multiple steps, do **not** reopen each separately or reopen the same step multiple
   times — start at the earliest.
2.5. **Offer the Post-Audit Fix Pass (§3.6.1) at this point, if the user requests it** —
   a compressed single-session version of steps 3–6 below (same work, no intermediate
   sessions/gates), still mandatorily ending in a fresh Step 7 session. Default path is the
   full multi-session workflow below unless the user explicitly invokes the pass.
3. **Open one new "backtrack" session** (per §1; Step 7's handoff note for this session makes
   clear what kind of session it is and what it must accomplish). **Proceeding directly into
   fixing is the default the moment this session opens — no separate "go ahead"/"proceed with
   fixes" instruction is needed beyond the findings themselves having been presented at the
   Step 7 gate.** The user starting this new session (unavoidable, no cross-session memory) is
   itself the go-ahead; Claude does not re-confirm findings or wait for an additional nod
   before beginning to fix them. Working forward from the earliest originating step through to
   Step 7 again:
   - Fix every finding tracing to the current step, Trivial and Substantive alike, editing
     that step's own owned file directly.
   - **Fix anything else discovered incidentally while fixing** — a defect not in the
     original report, found only because the fix required looking closely at the file —
     regardless of whether it predates this session or relates to the finding being fixed.
     Never deferred to "flag for the next auditor"; that is scope-creep in the wrong
     direction (under-fixing, not over-fixing) and is not permitted (§3.6.1 point 5).
   - **Re-check already-approved later-step content for cascading effects of the fix**, per
     §3.6's "revisit only as needed" — discovered by re-checking as each fix is made, not
     pre-identified by Step 7 (which only reports what it found, not downstream consequences
     of a fix that hasn't happened yet).
   - Move to the next step in sequence (finding or not), applying any cascading fix plus any
     finding tracing directly to it.
   - Continue through Step 6, so every originating file and every downstream file is
     reconciled by session end — both original findings and cascading effects.
   - **Do not re-run Step 7 itself within this session.** The job is to get the Spec ready
     for a fresh, independent audit — not to self-certify success.
4. **At session end, package every file exactly as at any other Step Approval Gate** (§1,
   §3.10): every touched Architecture file, version bumped exactly once per §4 (never once per
   finding fixed — one session, one bump, per file actually touched); a handoff note for the
   next session. **The triggering Audit Report's findings are embedded directly in this
   handoff note's own text** (§3.10) — not shipped as a separate file — since this backtrack
   session is the only consumer that needs them.
5. **That handoff note MUST state plainly the next session is a new Step 7 audit** (pass2,
   pass3, ... per §3.10), not a continuation, and MUST summarize: which findings were fixed
   (Trivial and Substantive alike), which incidental defects were found and fixed along the
   way, which steps/files were reopened and in what order, and **explicitly that cascading
   effects were discovered and resolved by re-checking later-step files during the backtrack
   session itself** (not pre-identified by the original report) — so the new Step 7 session
   understands the full scope of what changed and why.
6. **The new Step 7 session re-audits the entire Spec from scratch**, same adversarial
   independence as any Step 7 run (§3.4) — it does not take the backtrack session's own
   account as given; it re-derives findings independently. Clean → proceed to Step 8
   normally. Any finding (even something new) → this workflow (1–6) repeats, or the
   all-Trivial fast path above if every new finding qualifies.
7. **Circuit breaker: two consecutive non-clean Step 7 passes on the same Spec.** If the
   re-audit in step 6 is itself the *second* consecutive Step 7 run to produce any
   Substantive finding (i.e. a backtrack-then-reaudit cycle has now failed to converge once
   already on a genuine judgment-call item — repeated all-Trivial passes don't count toward
   this, since the fast path already resolves them without a full cycle), this workflow does
   **not** silently repeat a third time unchanged. Instead:
   - Claude produces a short **Convergence Diagnostic**: the findings from the prior
     non-clean pass alongside the findings from this pass, side by side, each flagged as
     either (a) recurring / same underlying category as before, or (b) genuinely new and
     unrelated to anything the prior fix touched.
   - Claude presents the user an explicit choice rather than looping again automatically:
     invoke the **Post-Audit Fix Pass** (§3.6.1) to force resolution in one compressed
     session; **accept a specific finding as a documented, deferred known-issue** (normal
     RATS outcome — Deferred-to-a-step or Future Feature — never silently dropped, and
     subject to §3.12's Register rules); or **direct a manually-scoped fix** themselves.
   - **A third identical pass with no change in approach is not permitted.** Something —
     scope, method, or an explicit deferral — must change before a third Step 7 run.

**Major Change Notification still applies** (§3.6 point 4): a backtrack fix that materially
changes scope/requirements/architecture is flagged the same as any other backtracking.

---

### 3.12. End-of-Step Open-Items Review

**Every Step Approval Gate (§3.4), without exception, includes a standing review of every
unresolved item accumulated so far — not only items from the current step.** Distinct from,
and additional to, RATS's own per-item resolution (§3.1), which settles *that step's own*
residual items before its gate; this reviews the **running total across every step so far**,
in an explicit attempt to shrink the open list before it compounds.

**The Register tracks open items only — a resolved item is deleted from it, not archived.**
Per §3.1's terminal-outcome rule, every item that ever entered RATS resolves to exactly one
of four outcomes, but only one of the four leaves anything on the Register:
- **Resolved** — the resolution is folded into the actual content of the relevant Spec
  section (that's where the decision now lives, in context); the Register entry is deleted
  immediately, not kept as a historical record. Nothing about a closed item needs to keep
  taking up space in a list whose only job is tracking what's still outstanding.
- **Deferred to Step N** — the **only outcome that remains on the Register**, tagged with its
  target step, and **dormant**, not "open," until Step N is actually reached. Not re-presented
  at any gate before Step N; at Step N's own gate it re-enters RATS as a normal residual item
  for that step — at which point it resolves to one of the other three outcomes and is
  deleted per this rule.
- **Future Feature** — recorded in Spec §2.6 (its permanent home); Register entry deleted
  immediately. Reopening it later is a fresh decision referencing Spec §2.6 directly, not a
  Register lookup.
- **Rejected** — the ID is retired per the stable-ID rule (recorded as superseded, per
  `AGENTS.md` §2.1's Stable Identifier Assignment); Register entry deleted immediately.

**An item with no terminal outcome yet is the only other kind actively re-surfaced at every
gate** — this should not exist in practice past the step that raised it, since RATS resolves
every residual item to one of the four outcomes before that step's gate closes; if one
persists anyway (e.g. a Deferred target step turned out invalid), it's re-run through RATS at
the very next gate rather than left indefinitely open.

**Mechanism, every gate:**
1. Claude maintains the running **Open Items Register** — genuinely open items only (in
   practice: Deferred-and-not-yet-reached, plus any stray non-terminal item), each tagged
   with its originating step and, if Deferred, its target step. A Resolved, Future Feature, or
   Rejected item is never a Register row — it's deleted the moment RATS reaches that outcome.
2. At every gate, before the step's own headline output, Claude presents: (a) any item whose
   Deferred target step is *this* step — these re-enter RATS now, and are deleted from the
   Register on resolution per point 1; (b) any item still without a terminal outcome — these
   are re-run through RATS immediately, since this state shouldn't persist.
3. **The Register lives inside `01_introduction`** (§4.1) as a dedicated subsection
   (§1.7 of the Architecture Spec template) — not a separate file. Every step's gate touches
   `01_introduction` to append/update/prune it, even when that step's own owned file is a
   different one; this is an explicit, standing case of §4.3's "touched file" bump rule, not
   an exception to it. Pruning a resolved entry is a normal edit here, not a violation of the
   additive-only mandate (`AGENTS.md` §2.1) — the additive-only mandate governs technical
   content and prior decisions, not the bookkeeping list of what's still outstanding; the
   decision itself is preserved (in Spec content, Spec §2.6, or the retired-ID record), only
   the now-redundant Register row is removed.

**This does not relax §3.1's own per-step RATS resolution** — a step's own residual items
still resolve to a terminal outcome before its gate. §3.12 is the additional cumulative
check ensuring a Deferred item is actually picked back up at its target step, and that
nothing lingers in a non-terminal state past the step that raised it.

---

## 4. Multi-File Output & Cross-Linking

Applies once a Spec or Plan deliverable spans multiple files. All files for one artifact live
in the same flat directory (no nesting, unless the user changes this). Names below are **base
names** — Architecture Spec and Plan files (not the Checklist) carry a version suffix per
§4.3. `[projectname]` is the actual project's filesystem-safe slug (underscores, not hyphens,
`AGENTS.md` §2.5), confirmed with the user at Step 1 if not already established.

**Naming convention — `[projectname]` first, then artifact type, then a zero-padded two-digit
sequence starting at `01`, then topic, then version suffix.** Projectname-first keeps every
file belonging to one project's Spec and Plan sorting together, with architecture/dev-plan/
checklist/prompt files interleaving sensibly in an alphabetically sorted listing, rather than
scattering by generic numeric prefix alone.

### 4.1. Architecture Specification — 8 Files, One Per Design Step

**Split into 8 files, one per originating Design Step (1–6; Steps 4 and 5 each get their own
file and always run as separate sessions).** This supersedes any coarser topic-based grouping: smaller projects could once fold multiple
Steps' output into a shared handful of files, but at this template's target scale (50+ User
Stories, 100–200+ requirements are common), a coarser grouping produces multi-thousand-line
files and forces every Step's session to bump a shared file's version even when it touched
none of the file's pre-existing content. One-file-per-Step avoids both: each file's size stays
bounded by what one Step actually produces, and its version history reflects only the
sessions that actually changed it.

Pattern: `[projectname]_architecture_NN_<topic>_v[N].md`, `NN`/`<topic>` fixed per the table,
`[N]` that file's own independent version counter (§4.3):

| File (`NN_topic`) | Template content | Owning Step | Created at |
|---|---|---|---|
| `01_introduction` | §1 Introduction, including §1.7 Open Items Register (§3.12) — updated at every subsequent Step's gate, not just Step 1's own | Step 1 | Step 1 |
| `02_user_stories` | §2.1 Personas, §2.2 User Stories, §2.5 Out-of-Scope, §2.6 Future Features | Step 2 | Step 2 |
| `03_requirements` | §2.3 Core Functional Requirements, §2.4 Non-Functional/Quality Attribute Scenarios | Step 3 | Step 3 |
| `04_test_strategy` | §3.1 9 Criteria (self-check ref), §3.2 Test Case Catalog, §3.3 Rust Requirement Smells | Step 4 | Step 4 |
| `05_verified_traceability` | §3.4 Traceability Matrix, §3.5 Acceptance Criteria Detail; temporarily holds the Step 5 MSRV decision and any pre-Step-6 Asset Manifest entries, both flagged as placeholders pending relocation | Step 5 | Step 5 |
| `06_viewpoints` | §4, all four ISO 42010 viewpoints (Functional, Information, Deployment, Interface Control) plus §4.5–4.16, including §4.13's Asset Manifest (relocated here from `05`) | Step 6 | Step 6 |
| `07_interfaces_and_stack` | §5 External Interfaces & Integrations, §6 Technology Stack & Dependencies (including finalized MSRV, relocated from `05`) | Step 6 | Step 6 |
| `08_constraints_and_roadmap` | §7 Constraints & Assumptions, §8 Risks & Technical Debt, §9 Implementation Roadmap & Build Order, §10 Public API & Framework Consumer Contract, §11 Appendices | Step 6 | Step 6 |

If `06_viewpoints` alone overruns a reasonable size once real content volume is known
(likely, per the template's own note that §4 is its largest section), Claude flags this at
Step 6 and proposes a further split (e.g. `06a_functional_view`, `06b_information_view`, ...)
rather than silently overrunning — the same scale-driven-split principle applies recursively.

**A Step 7 or Step 9 Audit Report is never a standalone numbered file.** On zero findings,
it's presented in chat at the gate — nothing to carry forward. On any finding, its content
is embedded directly in the backtrack handoff note (§3.10, §3.11) rather than delivered
separately. Handoff notes themselves also remain outside this numbered set, per §3.10's
own naming convention.

**Do not conflate the two `_01_...` files.**
`[projectname]_architecture_01_introduction_v1.md` (this section) and
`[projectname]_dev_plan_01_overview_v1.md` (§4.2) are independent artifacts with
independently-chosen naming schemes that happen to both start `_01_` and share the
version-suffix format — neither is stale/superseded relative to the other, and the Plan's
continued use of a topic word (`overview`) isn't evidence the Spec still uses topic-based
names too. A name like `[projectname]_architecture_01_overview_v[N].md` (combining a Spec
`NN` slot with a Plan-style topic word) is invalid under this scheme — almost certainly a
pre-restructure leftover, a naming error, or a hallucination, not evidence the 8-file table
has changed. Verify against this table directly before accepting a cited filename at face
value.

### 4.2. Development Plan (15 sections → 4 files), Development Checklist, Kickoff Prompt

Unchanged from prior structure — the Plan's coarser grouping is intentional and distinct from
the Spec's Step-aligned scheme (see §2's note on why the two split at different
granularities). Pattern `[projectname]_dev_plan_NN_<topic>_v[N].md`:
- `..._01_overview_v[N].md` — §0 Architecture Cross-Reference, §1 Introduction, §2 Technology
  Stack, §3 Project Folder Structure
- `..._02_environment_and_phases_v[N].md` — §4 Environment Setup, §5 Dev/Test Configuration,
  §6 Phases and Milestones (including the frontend targeted-interleaving sequencing
  principle from §3.4's Step 8 row, elaborated per §9.1)
- `..._03_tasks_and_testing_v[N].md` — §7 Risk Management, §8 Task Decomposition, §9 Test
  Strategy, §10 Logging Strategy
- `..._04_protocols_and_dod_v[N].md` — §11 Session Handoff, §12 Abort/Rollback, §13
  Escalation Triggers, §14 Change Control, §15 Plan-Level Definition of Done

**Development Checklist** (single file unless content genuinely warrants a split, §2):
`[projectname]_dev_checklist.md`, base name, no numeric prefix — conventionally one file
covering the whole Plan, not a set member. **No version suffix**, same in-place-editing
reason as the Kickoff Prompt. Per `AGENTS.md` §2.7, edited in place — never copied, renamed,
or otherwise reproduced.

**Development-Agent Kickoff Prompt:** `[projectname]_dev_prompt.md`, base name, no numeric
prefix, no version suffix, produced once at Step 8's end, reused verbatim every
Development-Phase session.

**Phase Summaries** (Development Phase, per `agents/exemplars/development_plan_template.md`
§11.3): `[projectname]_phaseN_summary.md` — carries the `[projectname]` prefix for
consistency, even though it's a Development-, not Design-Phase, artifact.

This grouping is a starting proposal for the Plan (the Spec's 8-file grouping is fixed, tied
directly to Step ownership) — Claude revisits the Plan's grouping once real content volume is
known at Step 8, flagging a different grouping if warranted, rather than forcing content into
a pre-decided shape.

**Linking:** the first file in each numbered set is an index (overview + linked TOC, relative
links, co-located assumed) — for the Spec, `01_introduction`. Every other file links back to
the index and forward/sideways wherever there's a substantive content dependency — based on
actual dependency, not sequence, using heading-derived anchors for a specific section, not
just a file's top. (E.g. `06_viewpoints`'s ICD referencing a Data Dictionary type earlier in
the same file is an in-file anchor; `08_constraints_and_roadmap`'s Roadmap referencing a
requirement in `03_requirements` is cross-file.) The Spec and Plan file sets are independent —
no cross-linking absent a concrete reason (e.g. a Plan phase implementing a specific ICD
interface).

**Intermediate/handoff artifacts** (handoff notes — which, on an audit finding, embed that
audit's report content directly, §3.10/§3.11 — plus working notes, draft batches, anything
mid-step rather than settled Architecture content) use the §3.10 handoff-note convention:
`[projectname]_design_handoff_stepN_passX.md` — distinct from the numbered Architecture/Plan
files, which are organized by content section (and, for the Spec, by owning Step), not by
which session produced a given revision. **The Open Items Register is not in this
category** — it's embedded in `01_introduction` (§4.1, §3.12), not a handoff artifact.

### 4.3. Versioning: Per-File, Bumped Once at Session End

**Version format is a plain integer counter — `v1`, `v2`, `v3`, ... — nothing else.** Fixed,
not a preference: no decimals, no semver triplets (`v1.0.0`, `v1.0.01`), no zero-padding
(`v01`). `[N]` increments by exactly `1` per bump, starting at `v1` at creation. Stated
explicitly because different sessions have independently adopted different schemes — every
file, in every project, uses plain-integer versioning only. A file found using another format
(e.g. `v1.0.01`) is corrected to plain-integer at its *next* bump (continuing from the closest
sensible integer, flagged as a one-time correction) rather than perpetuated.

**Every Architecture Spec and Dev Plan file carries its own independent `_v[N]` suffix,
starting at `v1` at creation.** Two rules govern `[N]`, both material:

- **Independent per file, not shared across a set.** No single project-wide or artifact-wide
  version. `01_introduction` may be at `v4` (revisited across backtrack sessions) while
  `03_requirements` is still `v1` (untouched since Step 3) — normal, not an inconsistency.
  Any example filename elsewhere implying a single shared `[N]` across files (e.g. a
  kickoff-prompt file list at one placeholder version) is illustrative shorthand only — a
  file's real version is always read off its own actual name, never inferred from a sibling.
- **Bumped exactly once, at the end of the session that touched it — never mid-session, never
  more than once per session** regardless of edit/fix count. Within a session, a file may be
  edited many times (RATS resolution rounds, corrections, flagged-item resolution) —
  all under the *same* version number, since the bump exists to prevent download/upload
  collisions *across* sessions, not to track every internal edit. Only at session end, when
  packaged for handoff (§3.10), does the version increase by exactly one. An untouched file
  keeps its version, not redelivered just because siblings in the same batch changed.
  **"Touched" means any file whose content changed this session, full stop — not limited to
  the current step's own file.** A backtrack session (§3.11) or ordinary §3.6 revisit editing
  an earlier step's file bumps it exactly once too, at the same session-end point, via the
  §3.10 enforcement check — stated explicitly because this is the specific case observed
  slipping through: a session correctly bumping its own file while leaving a revisited
  earlier-step file's version unchanged.

This fixes a real, repeated download/upload collision that motivated file versioning in the
first place: the same base filename re-downloaded across sessions under an identical name.
Bumping strictly at session-end, per file actually changed, solves the collision without
punishing every file in a set for one file's edits, and without turning mid-session iteration
into untrackable version churn.

**This has a real, accepted cost: cross-links break on every version bump, and rewriting them
is mandatory.** Per §4's cross-linking convention, files in a set link to each other by name —
a version bump means every *other* linking file now points at a stale name. Whenever Claude
re-presents a versioned file with new content, it MUST, in the same pass: (1) update that
file's own internal links if their targets also changed version, and (2) check every other
file in the current set for links to this file's *old* version name, updating them. This
recurs every bump; skipping it silently reintroduces broken cross-links the same way skipping
filename versioning reintroduced download collisions.

**Inline prose citations to another file's *content* never include a version number — cite
`File N §section` only, never `File N vM`.** This is distinct from (and simpler than) the
filename cross-link rule above, which governs actual download-link targets and has no way
around carrying the version suffix. An inline citation in running prose — "see File 6 §4.2,"
"as defined in File 3 §2.4" — is only ever pointing at *where* to look, not asserting which
version it was when written; a version number there is pure downside with no payoff, since
nothing downstream needs it and it goes stale the moment the target file bumps again. This
was the actual mechanism behind a repeated, multi-session defect: `File N vM`-style prose
citations drifting out of sync across bumps that happened in sessions never touching the
citing file, with each backtrack session only catching whatever drift existed *at that
moment* and falsely asserting an exhaustive re-sweep. Dropping the version number from prose
citations entirely removes the defect at its source — there is no version number left in
prose to go stale, and no sweep is needed to keep one current. If Claude finds an existing
`File N vM` prose citation while touching a file, correct it to `File N §section` form as a
normal edit; this is not a version bump trigger by itself.

**Order of operations, mandatory: bump, then fix, never fix-then-bump.** This is the specific
sequencing failure observed causing repeated Step 7 findings on the same class of issue — a
session fixes cross-links against the *current* (pre-bump) filenames, then bumps the version
at the very end, silently re-staling every reference it just corrected. The correct order,
every session, is:
1. First, increment the version of every touched file (once each, per the rule above).
2. Only *after* every touched file carries its new name, compute and rewrite every cross-link
   project-wide against the **post-bump** filenames — never the pre-bump names.
3. **A final reference-consistency pass runs last, immediately before packaging (§3.10), as
   the literal final step before delivery**: grep every file in the current set for any
   remaining reference to any old version name. If this final pass finds a stale reference,
   fix it in place — this does **not** trigger a further version bump (the file was already
   bumped once this session, per the once-per-session rule above).

**No frontmatter or header box, anywhere, for anything — all file metadata lives in
`Appendix R — Version History`, at the end of the file, never at the front.** This
generalizes past version history specifically: no metadata table or block at the top of
any Architecture Spec or Dev Plan file — no `Status:`, `Owner:`, `Last Updated:`,
`Reviewed By:`, `Approved:`, or any other such field, in any format (table, definition
list, bolded key-value lines). The only content permitted before the first numbered
section is the title line and, where the template calls for one, a brief italicized/
blockquoted usage note (e.g. this doc's own "When to use" guidance) — never file metadata
of any kind. Every Architecture Spec and Dev Plan file ends with this appendix (after its
last content section — e.g. after §11 Appendices for the Spec, after Appendix G for the
Plan): one row per session that bumped *this file*, in order, each row stating the
version reached, the date, and a brief description of what changed in that file this session.
This is deliberately **not** a separate carried-along file — unlike grail's own meta-repo
`CHANGELOG.md` (which tracks grail's own instruction/template files, a different system
entirely), a per-project deliverable's version history travels with the file
itself, so it's automatically present whenever the file is, with nothing extra to request or
enumerate in a handoff note. On a version bump, Claude appends exactly one new row to this
appendix, in the same pass as the bump itself — never rewriting or removing a prior row.
**Any status/ownership fact that would otherwise have lived in a front-of-file header box
has a real home elsewhere, not a new column here:** approval state is the Gate history
itself (a file's latest version was approved at whatever gate produced it — nothing further
to record); document purpose/audience is §1.1; anything else genuinely informational about
the file's own content belongs in the relevant numbered section, never a standalone field.
**Nothing resembling a changelog, revision note, or "what changed" commentary belongs
anywhere else in the file** — not under the title, not in the Introduction, not inline near
whatever content changed. If Claude finds itself about to write a version-related note
anywhere but this appendix, that's the signal to relocate it here instead.

---

## 5. Other Standing Mandates (from `AGENTS.md`)

Pulled forward as directly applicable to document production/revision here: **additive-only
by default** (no removing/"cleaning up" approved content without explicit deletion approval);
**no self-referential elision** (never "see Section X of the previous version" — every
document stays self-contained); **no stubs** (sections written to full depth when presented,
not sketched and deferred); **formal approval protocol** (exact, all-uppercase `APPROVED`
only — no "Continue"/"Proceed"/"Go ahead" substitutes, for a Step Gate, a Minor Change, a
Major Change, or handoff-package preparation; new instructions + `APPROVED` = partial
approval, incorporated then re-presented, `AGENTS.md` §3.1); **`CHAT:` prefix** = pure question, no document changes, no
gate progression, no side effects of any kind (`AGENTS.md` §3.2.1); **no assumptions on
ambiguity** — ask rather than guess and build on the guess; **no reproducing shared working
documents** — the Development Checklist and Dev Prompt are edited/reused in place, never
copied or renamed (`AGENTS.md` §2.7).

---

## 6. Out of Scope for This File

Development Phase execution behavior (`agents/DEVELOPMENT.md`); Maintenance-Phase-specific
mechanics beyond the shared Advisory Mode contract (§1) — the M1–M4 process, trigger
phrases, batch/checklist/prompt filenames, Tier/Track/Type classification, and the Release
Checklist all belong to `agents/MAINTENANCE.md`, not here; `agents/SCRIPT_RULES.md` (no
script execution in Advisory Mode). The concrete task-level mechanics of frontend targeted
interleaving (§3.4's Step 8 row states only the principle Design Phase applies when sizing
phases — the full mechanism belongs in
`agents/exemplars/development_plan_template.md` §6/§9.1 and `agents/DEVELOPMENT.md`).
`AGENTS.md` §2.5's naming convention (underscores, not hyphens) isn't a standalone concern for
these Markdown documents, but is followed anyway for any Rust identifiers/crate names Claude
introduces in examples, for consistency.

---

## Appendix
See `CHANGELOG.md` for this file's full version history. **v0.8.5 batch:** removed the
Autonomy Toggle/Tailored/Full-Autonomous presets and the Grouped Closing
Protocol/propose-with-flagged-assumptions mechanism entirely, replaced by the single fixed
RCD/RATS procedure (§3.1-3.2) for every step but 7 and 9 — Step 2 always runs in full. RATS
items now resolve to exactly one of Resolved/Deferred-to-a-specific-step/Future
Feature/Rejected (§3.1), with Step 7/9 auditing that every Open Items Register entry carries
a terminal outcome. Handoff/version-bump timing reworked: interim files use lowercase-letter
suffixes and are never packaged or version-bumped until exact `APPROVED` is received (§3.10).
`AGENTS.md` §3.1 tightened to exact, all-uppercase `APPROVED` only. Renamed
`dev_agent_prompt_template.md`/`[projectname]_dev_agent_prompt.md` back to
`dev_prompt_template.md`/`[projectname]_dev_prompt.md`. Step 8 now also drafts the project
README. Added ESP32/ESP-IDF build guide cross-reference (§2.1). All prior sections renumbered
down by one following removal of the former §3.7 (Autonomy Toggle). This should be recorded
as a new dated entry in `CHANGELOG.md`'s `CLAUDE.md` section and flagged to the user as a
Major Change per §3.6.
