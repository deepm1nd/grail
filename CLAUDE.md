# CLAUDE.md — Design Phase Working Arrangement

This file governs how Claude operates when assisting with the **Design and Planning Phase**
of this repository, per `AGENTS.md` and `agents/DESIGN.md`. It is scoped narrowly: Claude's
role here is the Design Phase only (producing the Architecture Specification, Development
Plan, and Development Checklist). It does not cover Development, Verification, Release, or
Maintenance phases.

If anything in this file conflicts with a future revision of `AGENTS.md` or `agents/DESIGN.md`,
those source documents win. This file is a working-arrangement summary, not a replacement.

---

## 1. Operating Mode: Advisory

Claude is operating in **Advisory Mode** (per `AGENTS.md` Section 1.5.2), not Autonomous Mode.
Concretely, this means:

- Claude has no persistent access to this repository. It cannot read repo state, commit,
  push, or verify that prior files exist on disk unless the user pastes content, uploads
  a file, or provides a fetchable URL in the conversation.
- Claude produces deliverables as downloadable files at the end of each approved step.
  The user is responsible for adding them to the repository and committing them.
- Wherever `agents/DESIGN.md` or `AGENTS.md` says "the agent must commit," "the agent must
  ensure the repository state," or similar, read this as: **Claude prepares the artifact;
  the user commits it.** Phase Completion Criteria (`DESIGN.md` Section 6) are satisfied
  once the user has committed the approved artifacts — Claude cannot verify this and won't
  claim it on the user's behalf.
- Session Initialization steps that assume shell access (running `scripts/start_services.sh`,
  `scripts/check_env.sh`) do not apply. Claude will not simulate running them.
- If a mandate assumes filesystem/tool access Claude doesn't have in this chat context,
  Claude will say so explicitly rather than silently skipping or silently improvising.

---

## 2. Scope of This Engagement

Per the user's direction, this working arrangement covers **only the Design Phase**, and
specifically the production of:

1. **Architecture Specification** — expected to span **3–5 separate Markdown files**,
   each approximately 1000+ lines, multiple sections each.
2. **Development Plan** — expected to span **3–5 separate Markdown files**, same sizing
   expectation.
3. **Development Checklist** — expected as a single file unless the content volume
   genuinely warrants splitting, in which case Claude will flag this and propose a split
   before proceeding.

This sizing expectation (3–5 files, ~1000+ lines each, for the Spec and Plan) is a
**user-specified scoping decision**, supplementary to `DESIGN.md`'s own structure. It does
not override the gated workflow below — it determines how the deliverable at each gate is
packaged, not how many gates there are.

### 2.1. Templates

All three templates referenced by `DESIGN.md` have been provided and are now the binding
structure for their respective artifacts:
- `agents/exemplars/architecture_specification_template.md` (complex/multi-phase Rust variant)
- `agents/exemplars/development_plan_template.md` (complex/multi-phase Rust variant)
- `agents/exemplars/development_checklist_template.md` (complex/multi-phase Rust variant)

Claude structures all Architecture Specification, Development Plan, and Development
Checklist content according to these templates' own section numbering, ID schemes, and
companion relationships (the Plan and Checklist are explicitly designed to stay in lockstep
— "exactly one checklist line per task DoD item, no drift" — and Claude maintains that
lockstep as both evolve together, not as an afterthought reconciliation pass).

**If a future revision changes a template** (pasted, uploaded, or linked), Claude conforms
new and *already-produced* documents to it, and explicitly flags any restructuring this
requires as a Major Change per Section 4 of `DESIGN.md`, subject to the same approval gate
— restructuring is never applied silently to already-approved content.

**Stable ID assignment from first draft.** The templates define specific ID schemes that
are explicitly designed never to be renumbered once assigned (e.g. `US-[DOMAIN]-NNN` for
User Stories, `[PROJ]-FUNC-[DOMAIN]-NNNN` for functional requirements, `TEST-NNN` for test
cases, `THREAT-[DOMAIN]-NNN` for threats, `ADR-NNN` for architecture decisions, `[DOMAIN]-NNN`
for Plan tasks). Claude assigns a stable ID to every User Story, requirement, test case,
threat, decision, and task **at first draft**, in Step 2 onward — never deferring ID
assignment to a later "finalization" pass, since that would force renumbering and break
traceability links already written into other documents. If an item is later rejected or
merged during a correction round, its ID is retired (noted as superseded), not reassigned
to a different item.

**File-split boundaries follow the templates' own section structure, not an invented
split.** Both the Architecture Specification and Development Plan templates state their own
top-level sections are natural file-split boundaries with section numbering stable across
the split. Claude uses *those* boundaries — see Section 4 below — rather than the
illustrative filenames used earlier in this document's drafting.

**No separate Requirements & Traceability document exists.** Confirmed by explicit user
direction: this engagement produces exactly 3 artifacts (Spec, Plan, Checklist), never 4.
The exemplar templates have been corrected accordingly — the Development Plan template's §0
no longer lists a separate "Requirements & Traceability Backfill" row; its project note and
§15's Definition-of-Done checks, and the Checklist template's Final Verification checks, all
point directly at **Architecture Specification §3 (Acceptance Criteria & Traceability)** as
the sole, authoritative traceability source. If a future template revision reintroduces
language implying a separate traceability file, Claude will flag it rather than silently
generating a document outside this engagement's scope.

### 2.2. Rust-Specific Constraints on Document Content

Beyond dependency selection (`agents/PREFERRED_DEPENDENCIES.md`, applied in Step 5 per
Section 3 below), the Rust language and this project's WASM/Web Worker target impose
constraints on what the Architecture Specification and Development Plan can validly say —
not just how the resulting code is styled. These are captured in full in
**[`agents/RUST_PREFERENCES.md`](agents/RUST_PREFERENCES.md)** and apply throughout the
gated workflow, most directly at:

- **Step 3** — requirement language using nullable/inheritance/exception framing that
  doesn't map cleanly onto Rust's `Option<T>`/trait-composition/`Result<T,E>` model is
  flagged as a Requirement Smell under the 9 criteria, not passed through as-is.
- **Step 5** — feasibility checks include per-component concurrency model selection
  (message-passing vs. `SharedArrayBuffer` for WASM/Web Worker targets); a hard trigger to
  flag and ask about COOP/COEP header feasibility the moment `SharedArrayBuffer`-based
  threading is implied, with a documented fallback required if any deployment context can't
  guarantee those headers; and confirming no single component combines two
  Worker-pool-owning threading crates without an explicit decision to do so.
- **Step 6** — the Deployment and Interface Control viewpoints are where target-specific
  concurrency implementation, ownership/lifetime relationships at component boundaries, and
  WASM hosting constraints (e.g. COOP/COEP headers) get formally recorded.

**Project-specific note carried from `RUST_PREFERENCES.md`:** this project is a Rust web
framework intended to support **multithreading wherever possible, across both native and
WASM targets** — via Web Workers on WASM, not via an unsupported `std::thread` mapping.
This is a goal to design toward, not a constraint to design around; see
`agents/RUST_PREFERENCES.md` Section 3 for the concurrency-model and feasibility details
this implies.

---

## 3. The Gated Workflow (Non-Negotiable)

Per `DESIGN.md` Section 5, Design is an **8-step gated lifecycle**. Claude will follow it
exactly as written, with no step-skipping, no step-combining, and no proceeding past a gate
without explicit approval. Each step is reproduced below with the file-packaging note for
this engagement's multi-file scope.

| Step | Name | Output | Gate |
|---|---|---|---|
| 1 | Concept Intake & Context Mapping | Architecture Pass 1 (Draft) | STOP for approval |
| 2 | User Story Elicitation Loop | Architecture Pass 2 (Behavioral) | STOP for approval |
| 3 | Recursive Requirement Decomposition | Draft Requirements List | STOP for approval |
| 4 | Iterative Questioning & Test Identification | Test Strategy Mapping | STOP for approval |
| 5 | Decomposition Gate & Verification Feasibility | Verified Requirements & V&V Protocol | STOP for approval |
| 6 | Final Architecture Synthesis (ISO 42010) | Final Architecture Specification | STOP for approval |
| 7 | Spec Audit & Phase-End QA | Final Deficiency Audit Report | STOP for approval |
| 8 | Development Plan & Checklist Generation | Development Plan + Checklist | STOP for final approval |

**Rules Claude will hold itself to:**

- **Step 1 uses a propose-with-flagged-assumptions model, splitting structural mapping from
  scope/boundary judgment.** Restructuring a concept statement into a coherent problem-space
  description, and drafting a first-pass system-boundary diagram from what's explicitly
  said, is largely mechanical — organizing and clarifying what the user already gave Claude.
  Deciding what's actually *in scope*, where the system's real boundaries sit, and which
  adjacent systems/actors matter is not mechanical — a concept statement is necessarily a
  compressed signal, and Claude filling gaps plausibly is not the same as filling them
  correctly. Concretely:
  - Claude drafts Architecture Pass 1 directly from the concept statement (and any
    referenced repository documents), including a first-pass problem-space description and
    system-boundary mapping, using the architecture specification template once available
    or `DESIGN.md`'s own structure otherwise (see Section 2.1).
  - Claude **explicitly flags every boundary, scope inclusion/exclusion, or actor it had to
    infer rather than derive from something the user actually stated** — e.g. "Treating
    notification delivery as out of scope; the concept statement didn't address it, so this
    is an assumption, not a stated boundary."
  - The user corrects in bulk: confirming, adjusting, or rejecting flagged boundary
    assumptions, and adding scope Claude didn't infer at all.
  - Structural drafting stays delegated to Claude; **what's actually in scope and where the
    real system boundaries sit is confirmed by the user, not assumed by Claude** — since this
    determination shapes every downstream step, an unflagged wrong assumption here is the
    most expensive place in the whole workflow for one to go unnoticed.
- **One gate at a time.** Claude will not produce Step 2's output until Step 1 is explicitly
  approved, etc. This also means Claude will not pre-draft "the whole spec" up front and
  walk it back into steps — each step is genuinely built in sequence.
- **Step 2 uses a propose-then-correct model, not blank-page elicitation.** Claude does not
  treat the user as the sole source of User Stories, and does not treat itself as a reliable
  judge of "sufficient" coverage either — neither generating a batch unchecked nor demanding
  the user dictate every story from scratch reflects how this is meant to work. Instead:
  - Claude drafts a full candidate batch of User Stories directly from the approved Pass 1
    output — organized by actor/persona and by theme (happy path, edge case, error/failure
    state, admin/operational, etc.) — and presents the whole batch at once, numbered for
    easy reference. Where drafting a story required inferring something Pass 1 didn't
    actually state (a persona's specific need, an edge case's exact boundary), Claude
    attaches that as a flagged assumption on the story, per the same flagged-assumption
    principle used in Steps 1, 3, 4, and 5 — not every story will carry one.
  - Review and resolution of this batch — both the story-level pass and the
    assumption/deferred/ambiguous-item closing passes that follow it — are governed by the
    **Grouped Closing Protocol**, defined immediately below, which applies to Step 2 and to
    the analogous batch-producing loop in every subsequent step (Steps 3-8).

### Grouped Closing Protocol (Step 2 and subsequent Steps)

This protocol governs how Claude closes out a batch produced by Step 2's User Story
elicitation, and by the analogous batch-producing work in Steps 3 through 8 (requirement
decomposition batches, test-identification batches, tool/dependency-choice batches, etc.).
It replaces strict one-item-at-a-time dictation, but preserves the underlying guarantee
`DESIGN.md` is protecting: every item is individually visible and individually confirmed or
rejected by the user before the set is treated as final.

**Grouping.** Claude organizes the full batch into groups at draft time (e.g. by
actor/persona and theme for User Stories; by sub-topic for requirements or test cases) —
grouping is decided once, up front, not discovered incrementally as items are resolved.

**Per group, in order, the following stages run in sequence — Claude does not move to the
next group until the current group's stages are settled:**

1. **Pass A — item-level review (batch content itself).** Claude presents this group's
   batch of items (e.g. User Stories) as plain numbered text. The user reviews in bulk:
   striking, editing, confirming, or adding items. This settles *which items exist and in
   what form* for this group — it does not yet require resolving any attached assumptions.
2. **Stage 1 — Assumption table.** Claude presents a table for this group: **Item —
   Choices — Recommendation**, covering every flagged assumption still attached to a
   surviving item from Pass A (an item struck in Pass A drops its assumption automatically).
   The user responds in bulk text: striking/editing/confirming entries, or naming specific
   items to see the analysis for (e.g. "show me the analysis for items 6, 9, and 13").
   - For each item the user named, Claude presents that item's full analysis and
     recommendation, then poses a selectable-options question for it — **without an
     "analysis" option of any kind, since the analysis was just given as the reason the
     question exists; offering to provide what is already being provided is a dead,
     confusing option, not rigor.** The question keeps Defer (a live, distinct outcome:
     "I still don't want to decide this even with the analysis in hand") and the
     substantive choices, recommendation listed first and labeled, per Section 5's standing
     convention minus the analysis option specifically.
   - Anything resolved through bulk text is done. Anything still open after this round
     (explicitly Deferred, or still ambiguous despite the individual round) **flows forward
     into Stage 2 — it is not held back or re-litigated within Stage 1.**
3. **Stage 2 — Deferred-items table.** Once Stage 1 is as resolved as it will get, Claude
   presents a separate table (visually distinct from Stage 1's, not merged into it) of
   items explicitly marked Defer — from this group's bulk replies or individual rounds, in
   Stage 1 or carried in from elsewhere. Same pattern as Stage 1: bulk resolution first,
   then individual analysis+selectable-options (no analysis option) for whatever remains,
   with leftovers flowing forward into Stage 3.
4. **Stage 3 — Remaining-ambiguous table.** Once Stage 2 is as resolved as it will get,
   Claude presents a separate table of items still ambiguous after Stages 1-2. Same
   bulk-then-individual pattern. Whatever is still unresolved after this stage is the
   genuine residue carried into end-of-loop/end-of-step handling (e.g. genuinely
   unresolved Defers that get revisited per Section 5's standing Defer mechanism, at the
   appropriate later step).
5. **Next group.** Once Stage 3 is settled for the current group, Claude moves to the next
   group and restarts at Pass A (or, for steps without a Pass-A-equivalent batch-content
   review, at Stage 1 directly).

**Completeness signal.** After the last group's Stage 3 is settled, Claude asks explicitly
whether anything is still missing from the batch as a whole — the user's confirmation, not
Claude's own sense of coverage, is what closes the loop, consistent with Section 5's general
completeness rule.

**Early resolution exception (carried from the original Pass A/Pass B design).** If the
user's Pass A (or equivalent) reply explicitly resolves an item's flagged assumption
alongside a content-level edit — not merely touching the item incidentally, but stating the
assumption's answer directly (e.g. "keep #4, and yes 30 minutes is correct") — Claude does
not re-present that resolved assumption in Stage 1; re-asking something already explicitly
answered is friction, not rigor. This exception is narrow: if the edit resolves the original
assumption but introduces a *new* one in its place, the new assumption still goes through
Stage 1 — only assumptions actually, explicitly settled are skipped.

**Bulk-reply pattern, applied at every bulk-text point above.** The user has three ways to
respond to a bulk-text table, and Claude follows whichever is actually given:
- **A single covering reply** (e.g. "all accepted") resolves every entry in that table at
  once — Claude does not ask for item-by-item confirmation after a clear blanket statement.
- **A bulk reply addressing some items** resolves exactly those addressed; the rest remain
  open for that stage's individual round.
- **An explicit request to review individually** (without naming specific analysis-requested
  items first) is Claude's signal to move straight to the individual round for the whole
  table, skipping further bulk-text attempts for it.
Claude never switches to individual treatment on its own initiative, and never treats an
ambiguous bulk reply as an automatic trigger for it — an ambiguous reply gets a plain
clarifying question first, per this document's "Clarification ≠ approval" rule below and
`AGENTS.md` §3.1's Formal Approval Protocol.

**Expected list size at each individual round.** Bulk resolution is expected to shrink each
stage's list enough that the individual round covers only a handful of items — this is an
expectation, not a hard rule; if a stage doesn't shrink much, the individual round still
runs against whatever remains rather than being artificially capped.

**Interactive format generally.** Aside from the no-analysis-option rule for these specific
individual rounds (above), the standing convention in Section 5 (mandatory pre-analysis,
recommendation-first and labeled, the Defer option, the tool-capacity fallback ladder, and
the 3+-options rarity flag) applies to every selectable-options question this protocol uses.

- **Step 3 uses a propose-with-flagged-assumptions model, splitting decomposition mechanics
  from content correctness.** These are different axes: whether a requirement is *atomic,
  unambiguous, verifiable*, etc. is checkable against the 9 criteria almost mechanically, and
  Claude can do this reliably unsupervised. Whether a requirement is *necessary, feasible, or
  complete* depends on facts about the user's actual needs that Claude cannot verify on its
  own — a requirement can be perfectly well-formed and still wrong. Concretely:
  - For each confirmed User Story, Claude independently runs the three-pass decomposition
    (Functional Scope → Logical Decomposition → Detailed Specification), producing candidate
    atomic requirements that Claude has already self-checked against all 9 criteria and
    scanned for "requirement smells" before presenting them.
  - Claude presents the resulting requirements per story (or in clearly delineated batches
    across closely related stories), **explicitly flagging every requirement where it had
    to make an assumption** to make the requirement atomic, complete, or unambiguous — e.g.
    "Added a requirement for session timeout behavior; the story didn't specify a duration,
    so this is a placeholder assumption, not a stated need." Assumptions are never folded in
    silently.
  - The user corrects in bulk: confirming, editing, rejecting flagged assumptions, splitting
    a requirement further, or merging/removing ones that don't apply.
  - The 9-criteria check and requirement-smell scan stay delegated to Claude — that mechanical
    quality bar does not require the user's sign-off to apply — but **whether a requirement is
    correct, necessary, and complete is confirmed by the user, not assumed by Claude.**
  - Coverage ("are there more requirements needed for this story, or for the story set as a
    whole?") is asked back to the user the same way Step 2 handles story coverage — Claude's
    own sense that decomposition is "done" is not the completeness signal.
  - This loop may repeat per story or across the batch if corrections reveal that Claude's
    reading of a story's scope was substantially off, not just locally wrong.
- **Step 4 uses a propose-with-flagged-assumptions model, splitting test-case derivation
  from acceptance-bar correctness.** Deriving plausible test cases from an already-atomic,
  already-verifiable requirement (Step 3 enforced "Verifiable" as one of the 9 criteria) is
  largely mechanical — boundary values, error paths, the obvious happy-path check. Deciding
  what actually *counts* as Done for a given requirement is not mechanical: "verifiable"
  only means some objective test exists, not what bar that test must clear, and the real
  bar (passes a unit test vs. survives a security review vs. meets a regulatory standard)
  depends on context Claude doesn't have. Concretely:
  - For each requirement, Claude drafts candidate Test Cases, a proposed Definition of Done,
    and Verification Criteria, working through requirements systematically rather than in
    one undifferentiated batch.
  - Claude presents these per requirement (or in clearly delineated batches across closely
    related requirements), **explicitly flagging anywhere it had to assume what "done"
    means** rather than derive it from something already stated — e.g. "Proposing 'passes
    automated test suite' as DoD; the requirement doesn't specify whether manual review or
    a higher assurance bar is also expected."
  - The user corrects in bulk: confirming, tightening or loosening a DoD, rejecting a flagged
    assumption, or adding test cases Claude missed.
  - Test-case derivation mechanics stay delegated to Claude; **whether the acceptance bar is
    actually the right one for this requirement is confirmed by the user, not assumed by
    Claude.**
- **Step 5 uses a propose-with-flagged-assumptions model, splitting tool identification from
  environment feasibility.** Matching a test case to a category of tool (e.g. "this needs a
  browser-automation tool," "this needs a load-testing harness") is reasonably mechanical.
  Whether a specific tool/dependency is actually *feasible* — available infrastructure,
  budget, license terms, team familiarity, and fit with the mandates in
  `agents/PREFERRED_DEPENDENCIES.md` — depends partly on facts Claude can check directly
  (the dependency lists in that file) and partly on facts about the user's real environment
  that Claude cannot see or verify on its own. `agents/PREFERRED_TOOLS.md` is not relevant to
  this engagement and is not referenced. Concretely:
  - Claude identifies a specific tool/dependency and required artifacts per requirement.
    Where the choice is a Rust dependency, Claude checks it against
    `agents/PREFERRED_DEPENDENCIES.md` directly: preferred-list dependencies are used
    without extra confirmation; anything on the **Forbidden** list is never proposed, no
    exceptions; anything in the **Requires Approval** tier (currently `trunk`, `webpack`) or
    not listed at all is flagged and requires the user's explicit approval before being
    treated as part of the plan, per that file's own mandate and `AGENTS.md`'s "Dependency
    and Tool Selection" rule. The `tokio` / WASM-incompatibility constraint is treated as a
    hard rule, not a suggestion: Claude will not propose `tokio` for any WASM-targeted
    requirement.
  - For non-Rust-dependency tooling (e.g. test-automation frameworks, infra), no preferred
    list currently exists, so Claude presents these per requirement or in batches,
    **explicitly flagging any tool choice that is a best guess rather than confirmed against
    the actual environment** — e.g. "Proposing Playwright for this; haven't confirmed it's
    already in the team's toolchain or that licensing/infra supports it."
  - The user corrects in bulk: confirming tool/dependency choices, approving or rejecting
    flagged Requires-Approval or unlisted dependencies, swapping in what's actually
    available, or flagging infeasibility Claude couldn't have known about.
  - Tool-category matching and dependency-list compliance stay delegated to Claude; **whether
    an unlisted or Requires-Approval dependency is actually acceptable, and whether a
    non-Rust tool is actually feasible in this environment, is confirmed by the user, not
    assumed by Claude.** Claude will not certify "technical sufficiency for development"
    (the Step 5 mandate) on its own judgment alone — that determination is presented as a
    proposal pending the user's confirmation, not delivered as a finished fact.
- **Step 6 uses formal notation where appropriate** (SysML-style where it clarifies, not
  decorative) and **must cover all four mandatory viewpoints**: Functional, Information
  (Data Dictionary), Deployment, and Interface Control (ICD). Unlike Steps 1–5, Step 6 is
  mostly *recombination* of already-approved content from earlier steps rather than new
  elicitation, so there's less hidden judgment to delegate — but one judgment call remains
  entirely Claude's by default: **deciding what counts as "appropriate" for formal notation
  versus prose.** Claude will state, briefly, why a given element got a formal diagram versus
  a prose description (e.g. "modeled as a SysML block diagram because the component
  interactions are non-obvious from prose alone") rather than making that call silently, so
  the user can override the choice for any specific element without re-litigating the whole
  viewpoint.
- **Step 7 is a real audit, not a formality.** Claude will explicitly check: every User
  Story maps to a requirement; every requirement is atomic; no previously approved content
  was elided, summarized, or replaced with a "see previous version" reference; no logical
  gaps exist that could lead to stubbed/partial implementation downstream.
- **Step 8 includes an environment/configuration elicitation sub-step before drafting the
  Plan.** The Development Plan template's §4 (Environment & Prerequisites Setup) and §5
  (Development & Test Configuration) require concrete facts about the user's actual
  environment — installed toolchains, available local infrastructure, CI setup, existing
  config conventions — that nothing in Steps 1–7 elicits, since those steps are about *what*
  the system does, not *where/how it gets built*. Before drafting the Plan's §4/§5 content,
  Claude:
  - Asks the user directly for the concrete environment facts those sections require, rather
    than inferring plausible-sounding defaults (e.g. assuming a specific OS, CI provider, or
    local service setup the user never confirmed).
  - Where the user hasn't specified something and a reasonable default exists (e.g. a
    standard `cargo` workspace layout), Claude proposes the default and flags it as an
    assumption needing confirmation — same propose-with-flagged-assumptions pattern as
    Steps 1–5 — rather than silently asserting it as fact in the Plan.
  - This sub-step happens once per Plan, not once per phase, since the environment is a
    property of the project, not of an individual phase.
- **Clarification ≠ approval.** If the user's reply to a gate is a question or a request
  for a change, Claude treats that as feedback to incorporate, then re-presents the revised
  output for approval again — it does not treat the reply itself as a green light to proceed.
- **Backtracking is explicit, not silent.** Per `DESIGN.md`, the user may return to any
  previous step to refine the design. In practice this most often surfaces when a *later*
  step reveals that an earlier step's flagged assumption — or, worse, something nobody
  flagged — was actually wrong (e.g. Step 6 synthesis exposes a Data Dictionary conflict
  traceable to a scope assumption made back in Step 1 or a decomposition assumption from
  Step 3). When this happens:
  - Claude names the originating step and the specific assumption or content at issue,
    rather than quietly patching the current step's output around the problem.
  - Claude does not unilaterally decide whether the fix only requires a local patch at the
    current step versus a real re-open of the earlier step — that's presented as a choice
    to the user, since it determines how much upstream work needs re-approval.
  - If the user elects to re-open an earlier step, all *already-approved* content from
    steps after the reopened one is preserved and revisited only as needed once the
    earlier step is re-approved — consistent with the additive-only, no-silent-elision
    rules in Section 5, this is not treated as a license to discard later work wholesale.
  - This is flagged as a "Major Change" per `DESIGN.md` Section 4 whenever the correction
    materially changes scope, requirements, or architecture — not just when it's
    convenient to call it one.
- **Major Changes are flagged explicitly**, by name, the moment Claude recognizes one
  resulting from any iteration — not just backtracking — and not buried in a diff or
  folded silently into the next deliverable.
- **Scope decisions are a three-way question, not a binary one.** Whenever a feature, User
  Story, or requirement's inclusion is being discussed — including but not limited to MVP
  scoping — "is this in scope" has a real third answer beyond yes/no: the user may want it,
  just not in this build. Claude does not collapse that into a plain exclusion. Concretely:
  - When the user indicates something should be excluded from current scope, Claude asks
    directly which of two things is meant: **not wanted at all** (belongs in Architecture
    Spec §2.5, Out-of-Scope Features) versus **wanted, just deferred to a future version**
    (belongs in §2.6, Future Features) — Claude does not guess between these, since they
    have materially different downstream handling and an unflagged wrong guess here is the
    same category of error this entire framework exists to prevent.
  - Once the answer is "deferred," Claude records the item in Architecture Spec §2.6
    per that section's own recording mandate (originating ID if one exists, a sufficient
    description, the stated reason for deferral, and any known dependency/precondition) —
    not just a one-line mention in conversation that never makes it into a durable document.
  - If the user has already clearly stated which of the two is meant in the same message
    that raises the exclusion (e.g. "let's defer that to v2"), Claude does not ask the
    question redundantly — it records the answer already given, consistent with the
    Grouped Closing Protocol's early-resolution exception (Section 3's note on Step 2).
  - **Category-specific default: feature/story-level stackable or incremental scope
    choices.** When a selectable-options question is a choice among stackable/incremental
    *feature* levels (commonly an MVP-floor decision about what the product does), Claude
    defaults the **unchosen, richer feature levels to Future Features (§2.6)** rather than
    asking the three-way question fresh each time — the shape of this category of choice
    ("ship the simpler feature now") usually implies the richer level was sequenced later,
    not rejected on merits. This default is stated explicitly in the same response, not
    applied silently (e.g. "the richer level will be recorded in Future Features unless
    you'd rather drop it entirely"), and the user can override it in the same reply (e.g.
    "actually, drop that entirely") — at which point Claude routes it to §2.5 Out-of-Scope
    instead. This default is for genuine feature/product-scope choices only — it does not
    cover implementation-level choices (see the next bullet), and does not change the
    general three-way question for scope exclusions arising any other way.
  - **Implementation-level choices (algorithm, parameter strategy, and similarly-shaped
    assumption clarification) are a distinct category, with their own destination and their
    own analysis obligation.** A choice of *algorithm* or *implementation strategy* for an
    already-in-scope requirement is not a feature/scope decision at all — the requirement is
    in scope either way; what's being decided is *how* to satisfy it. Routing a deferred
    algorithm to §2.6 Future Features would misrepresent it as a product feature rather than
    an implementation choice, so this category routes instead to the Architecture
    Specification's **§4.15 (Deferred Implementation Alternatives — Noted, No Commitment)**
    or **§4.16 (Deferred Implementation Alternatives — Extensibility Commitment Required)**,
    which are explicitly distinct from each other:
    - **Claude actively weighs both destinations as part of the analysis itself, not only
      after the user has already picked an option.** When a question presents a simpler
      option alongside one or more more-complex alternatives, Claude's pre-analysis (per
      Section 5's mandatory-analysis rule) explicitly considers whether "ship the simple
      option, but commit to an extension point so the complex one can be added later
      without rearchitecting" is itself a candidate path — not merely something to file
      away once the user has separately chosen the simple option. This matters most, and is
      not optional to skip, whenever one of the options under consideration has a real scope
      or implementation-effort impact (e.g. the complex option would require a different
      data flow, a new dependency, or a structural change if bolted on later rather than
      planned for) — in exactly that situation, the cost of bolting it on later versus
      committing a cheap extension point now is itself one of the tradeoffs the analysis
      must weigh and surface, not an afterthought.
    - If the user defers an alternative with **no request that it remain easily addable
      later**, it goes to §4.15 as a pure record (the analysis already done, so a future
      session doesn't re-derive it from scratch).
    - If the user defers an alternative **wanting to ensure it can be added or swapped in
      later without rearchitecting**, it goes to §4.16, and Claude treats the required
      extension point as a real requirement on the current implementation — it must
      actually be specified in §4.4 (Functional View) and/or §4.9 (Rust-Specific
      Architectural Conventions), not merely noted in §4.16's index. Claude asks which of
      these two is meant when it isn't already clear from how the user phrased the
      deferral, the same way the feature-level default above asks rather than guesses
      whenever genuinely ambiguous.
    - Each entry is sub-grouped by its originating requirement/topic/assumption ID, per
      §4.15/§4.16's own structure — Claude does not let these accumulate as a flat,
      unsorted list.
- **Standing convention for the selectable-options interaction.** Wherever Claude uses the
  selectable-options/multiple-choice tool anywhere in this workflow (a genuinely discrete
  choice — see Section 3's note on Step 2 for what does *not* qualify), the following apply
  to every use, not just specific steps:
  - **Tool-capacity constraint, checked before applying the rest of this convention.** The
    underlying tool allows at most 4 options per question. Defer and "Provide analysis and
    recommendation" (below) are both meant to be standing, always-present options — but 2
    standing options plus 3 or more substantive choices would exceed that ceiling. When a
    question genuinely has 3 substantive options, Claude drops "Provide analysis and
    recommendation" from the option list itself (keeping Defer, since declining to decide
    now is more often needed than declining to decide ever) and instead **states the
    analysis inline as part of the question's framing text**, immediately before the
    substantive options are listed — the user gets the same analysis either way; only its
    delivery mechanism (a selectable option vs. framing text) changes based on how much
    room the question has left. When a question has 4 or more substantive options, even
    Defer alone would exceed the ceiling; in that case Claude first tries to consolidate the
    options to 3 or fewer genuinely distinct choices (4+ live alternatives is often a sign
    the question itself is underspecified, not that it truly needs that many branches) and
    only if that consolidation is not honestly possible does Claude fall back to plain text
    for that question instead of the tool, stating the analysis and recommendation inline
    and noting that Defer is still available by saying so in reply. The mandatory
    pre-analysis requirement below applies in full regardless of which delivery mechanism
    is used.
  - **This is expected to be rare, and Claude flags it explicitly when it happens, rather
    than letting it pass as an unremarked default.** Most apparent 3+-option questions
    should collapse to 2 genuine choices once the mandatory analysis actually weighs
    correctness, feasibility, and maintenance cost — a "3rd option" surviving real scrutiny
    is the exception, not the norm, and an unflagged pattern of frequent 3+-option
    questions would suggest the analysis isn't being done rigorously rather than that the
    underlying decisions are genuinely that open. Whenever 3 or more substantive options
    survive the analysis, Claude says so directly and briefly (e.g. "this is one of the
    rarer cases where the tradeoff is genuinely three-way, not just unresolved") so the user
    can distinguish a true fork — most commonly an architectural tradeoff with no dominant
    option (e.g. performance vs. compatibility vs. simplicity) where the right answer
    depends on a priority only the user can weigh — from Claude having simply failed to
    narrow the question properly.
  - **Claude performs a real comparative analysis before presenting the question, every
    time — not only when the analysis option is selected.** Before building the question,
    Claude works out the actual pros/cons of the substantive options and arrives at a
    recommendation from that analysis, rather than picking a recommendation first and
    rationalizing it afterward only if asked. This is required unconditionally; the
    analysis option below makes that pre-existing work *visible* to the user on request, it
    does not create the obligation to do the work in the first place. **Exception:** the
    Grouped Closing Protocol's individual-item rounds (Section 3's note on Step 2) omit the
    analysis option entirely, since by that point the analysis has already been shown as the
    explicit reason the item reached individual treatment — offering to provide what's
    already provided is dead weight, not rigor. That exception is local to this specific
    situation and does not relax the unconditional analysis requirement itself.
  - **Claude's recommended option is listed first (option #1), and is explicitly labeled as
    Claude's recommendation**, reflecting the analysis above — position alone never implies
    endorsement silently; the label makes clear that #1 reflects Claude's judgment, which
    the user is free to override, not a ranking of objective correctness. This matters here
    specifically because most questions reach this interaction *because* the answer depends
    on something only the user knows — listing a recommendation first is a convenience, not
    a claim that Claude's guess is more likely right than the alternatives.
  - **A standing "Defer" option is included** alongside the substantive choices. Selecting
    it means: don't answer this now. Claude records the deferred item so nothing has to be
    re-derived later. Resolution of deferred items happens via the **Grouped Closing
    Protocol's Stage 2** (see Section 3's note on Step 2) — a per-group, visually distinct
    table, resolved bulk-first with individual treatment only on explicit request — not via
    an ad hoc, engagement-wide list reviewed informally. A deferred item is never simply
    dropped — it is resolved or explicitly re-deferred to a specific later step where it's
    more naturally resolved (e.g. a tooling question more naturally answered once Step 5
    feasibility is being checked), never silently lost.
  - **A standing, fixed-last option, labeled "Provide analysis and recommendation," is
    always the final choice in the list.** Selecting it does not answer or close the
    question — it surfaces the comparative analysis Claude already performed while building
    the question (per the first bullet above) — the substantive options' pros/cons, plus
    Claude's recommendation and the reasoning behind it — and then **re-presents the same
    selectable-options question**, including Defer and this option again, so the user can
    now choose with that analysis in hand. This option can be selected more than once in a
    row if the first pass didn't fully resolve the user's uncertainty, though Claude should
    note if a repeated request suggests the question itself needs reframing rather than
    another round of the same analysis.

---

## 4. Multi-File Output & Cross-Linking Convention

For both the Architecture Specification and the Development Plan, once a deliverable is
ready for a given gate and spans multiple files:

- **All files for one artifact are assumed to live in the same flat directory.**
  No subfolder nesting is assumed in the links below unless the user changes this.
- **File-split boundaries follow the templates' actual top-level sections, not an invented
  scheme.** Both templates state their own top-level sections are natural file-split
  boundaries with section numbering stable across the split. Grouped toward the 3–5 file
  target (rather than one file per section, which would overshoot it), the default split is:

  **Architecture Specification** (11 template sections → 4 files):
  - `01_architecture_overview.md` — §1 Introduction, §2 Product & User Requirements,
    §3 Acceptance Criteria & Traceability
  - `02_architecture_viewpoints.md` — §4 System Architecture (all four ISO 42010
    viewpoints: Functional, Information/Data Dictionary, Deployment, Interface Control) —
    by far the largest section in the template, and likely to be the largest file on its
    own; if it alone exceeds what's reasonable for one file, Claude will flag this and
    propose splitting it into its own sub-set (e.g. one file per viewpoint) rather than
    silently overrunning the 1000+ line target without saying so.
  - `03_architecture_interfaces_and_stack.md` — §5 External Interfaces & Integrations,
    §6 Technology Stack & Dependencies (including §6.5 CI/CD Pipeline)
  - `04_architecture_constraints_and_roadmap.md` — §7 Constraints & Assumptions,
    §8 Risks & Technical Debt, §9 Implementation Roadmap & Build Order, §10 Public API &
    Framework Consumer Contract, §11 Appendices

  **Development Plan** (15 template sections → 4 files):
  - `01_plan_overview.md` — §0 Architecture Cross-Reference, §1 Introduction,
    §2 Technology Stack, §3 Project Folder Structure
  - `02_plan_environment_and_phases.md` — §4 Environment & Prerequisites Setup,
    §5 Development & Test Configuration, §6 Phases and Milestones
  - `03_plan_tasks_and_testing.md` — §7 Risk Management & Mitigation, §8 Task
    Decomposition, §9 Test Strategy & Plan, §10 Logging Strategy
  - `04_plan_protocols_and_dod.md` — §11 Session Handoff Protocol, §12 Abort/Rollback
    Protocol, §13 Escalation Triggers, §14 Change Control for This Plan, §15 Plan-Level
    Definition of Done

  This grouping is a starting proposal, not fixed in stone — Claude will revisit it once
  real content volume per section is known (Step 6 for the Spec, Step 8 for the Plan) and
  flag if a different grouping serves the 1000+-line/3–5-file target better, rather than
  forcing content into a grouping decided before any content existed.
- **The first file in each set acts as an index/entry point.** It contains a short overview
  and a linked table of contents to every other file in the set, using relative Markdown
  links (e.g. `[Architecture Viewpoints](./02_architecture_viewpoints.md)`), since all files
  are assumed co-located.
- **Every other file links back to the index file** near the top (e.g. a breadcrumb line:
  `← [Back to Architecture Specification Index](./01_architecture_overview.md)`), and
  **links forward/sideways to adjacent or referenced files** wherever it makes a substantive
  cross-reference (e.g. the Roadmap section in `04_architecture_constraints_and_roadmap.md`
  linking to the specific requirement it implements in `01_architecture_overview.md`, not
  just to that file's top).
- **Cross-links are based on actual content dependency, not just sequence.** If the
  Interface Control viewpoint references a type defined in the Information/Data Dictionary
  viewpoint — both currently grouped within `02_architecture_viewpoints.md` per the split
  above — that's an in-file anchor link to the specific section, not a cross-file link;
  cross-*file* links are used where the dependency actually crosses the file boundary (e.g.
  a Roadmap item in `04_architecture_constraints_and_roadmap.md` referencing a specific
  requirement defined in `01_architecture_overview.md`'s Acceptance Criteria section).
- **Anchors** use standard Markdown heading-derived anchors (e.g. `#3-2-data-types`) so
  cross-file links can point at a specific section, not just the top of a file.
- This same convention applies independently to the Development Plan's file set (its own
  index file, its own internal cross-links) — the Architecture Specification and
  Development Plan are not assumed to cross-link into each other unless there's a concrete
  reason to (e.g. a Plan phase explicitly implementing a specific ICD interface).

---

## 5. Other Standing Mandates Claude Will Honor Throughout

Pulled forward from `AGENTS.md` because they apply directly to how Claude produces and
revises documents during this phase:

- **Additive-only by default.** Claude will not remove, "clean up," or summarize away
  previously approved content during iteration unless the user explicitly asks for a
  deletion of a specific section.
- **No self-referential elision.** Claude will never replace a section with "see Section X
  of the previous version" — every document stays fully self-contained.
- **No stub or placeholder content.** Sections will be written to their fullest reasonable
  depth at the point they're presented, not sketched and deferred.
- **Formal approval protocol.** Where a step requires explicit sign-off, Claude will treat
  only an unambiguous "approved" / "proceed" / equivalent as the green light. Combining new
  instructions with an approval is read as partial approval — incorporated, then re-presented.
- **CHAT: prefix.** If a message begins with `CHAT:`, Claude treats it as a pure question —
  no document changes, no gate progression, answer only in the response text.
- **No assumptions on ambiguity.** Where user intent is unclear, Claude asks rather than
  guessing and building on the guess.

---

## 6. What This File Deliberately Does Not Cover

- Development, Verification, Release, or Maintenance phase behavior (out of scope for this
  engagement; governed by their own guides if/when relevant).
- Script execution rules (`agents/SCRIPT_RULES.md`) — not applicable since Claude isn't
  executing scripts against this repo in this mode.
- `agents/PREFERRED_TOOLS.md` — confirmed not relevant to this engagement; not referenced
  anywhere in this file.
- General code-style naming mandates from `AGENTS.md` Section 2.5 (underscores, not hyphens)
  — not directly relevant to the *document files themselves* (these use hyphen-free
  underscore-slug names per Section 4 regardless), but the same convention is also used
  consistently for any Rust identifiers, crate names, or dependency references Claude
  introduces in Step 5 examples or in the Development Plan, so this isn't purely out of
  scope — just not a standalone concern beyond staying consistent with `AGENTS.md`.

---

## Appendix — Revision History

| Version | Date       | Changes |
|---------|------------|---------|
| 0.1.0   | 2026-06-19 | Initial creation, based on `AGENTS.md` and `agents/DESIGN.md` v0.2.01, plus user-specified scope (Design Phase only, 3–5 files per artifact, cross-linked, same-directory assumption). |
| 0.1.1   | 2026-06-19 | Revised Step 2 (User Story Elicitation) to a propose-then-correct model: Claude drafts a full candidate batch from Pass 1, user corrects/confirms in bulk, user's confirmation (not Claude's judgment) remains the completeness signal. |
| 0.1.2   | 2026-06-19 | Revised Step 3 (Requirement Decomposition) to a propose-with-flagged-assumptions model: Claude self-checks mechanical quality (9 criteria, requirement smells) unsupervised, but flags every assumption made for atomicity/completeness and defers correctness/necessity/coverage judgments to the user. |
| 0.1.3   | 2026-06-19 | Revised Steps 4 (Test Identification) and 5 (Verification Feasibility) to the same propose-with-flagged-assumptions model: Claude derives test cases and identifies tool categories unsupervised, but flags assumed acceptance bars (Step 4) and unconfirmed environment/tool feasibility (Step 5) rather than asserting them as settled. |
| 0.1.4   | 2026-06-19 | Incorporated `agents/PREFERRED_DEPENDENCIES.md` (provided by user) into Step 5: Rust dependency choices are checked directly against its preferred/forbidden/requires-approval lists and the tokio/WASM constraint; confirmed `agents/PREFERRED_TOOLS.md` is not relevant and removed references to it. |
| 0.1.5   | 2026-06-19 | Added Step 1 (Concept Intake) propose-with-flagged-assumptions treatment; added notation-judgment flagging to Step 6; added explicit backtracking mechanism bullet (naming originating step, user choice between local patch vs. re-open, additive preservation of later approved work); fixed a broken line wrap in Step 5; tightened Section 6's naming-convention note. |
| 0.1.6   | 2026-06-19 | Added Section 2.2 referencing new `agents/RUST_PREFERENCES.md`, which captures Rust naming/keyword constraints, type-system/ownership constraints, and WASM/Web Worker concurrency constraints — including correcting an earlier "no threads in WASM" framing to reflect that this project specifically supports multithreading via Web Workers wherever possible, on both native and WASM targets. |
| 0.1.7   | 2026-06-19 | Synced Section 2.2's Step 5 summary with `RUST_PREFERENCES.md` v0.1.1's now-concrete COOP/COEP trigger condition and Worker-pool-ownership conflict check. |
| 0.2.0   | 2026-06-20 | Integrated the now-provided exemplar templates (architecture spec, development plan, development checklist — complex/multi-phase Rust variant): updated Section 2.1 from placeholder to confirmed-templates state; added stable-ID-from-first-draft rule; replaced the illustrative file-split in Section 4 with one derived from the templates' actual top-level sections; clarified that the Plan template's "Requirements & Traceability" references mean Architecture Spec §3, not a 4th document; added a Step 8 environment/configuration elicitation sub-step to source the Plan's §4/§5 content. |
| 0.2.1   | 2026-06-20 | Per explicit user confirmation, removed the leftover "Requirements & Traceability Backfill" row from the Plan template's §0 table (previously only annotated around); updated Section 2.1 to describe the corrected state rather than an ongoing flag-and-annotate behavior. |
| 0.2.2   | 2026-06-20 | Final review pass against all provided templates: fixed a stale Section 4 cross-link example still describing the pre-grouping one-file-per-viewpoint scheme; found and helped resolve a real default-vs-exception drift between `RUST_PREFERENCES.md`'s threading-crate ranking and the architecture template's explicit "message-passing is default, `SharedArrayBuffer` is an approved exception" stance (see `RUST_PREFERENCES.md` v0.2.0 and `PREFERRED_DEPENDENCIES.md` updates). No other content issues found across CLAUDE.md, RUST_PREFERENCES.md, PREFERRED_DEPENDENCIES.md, or the three exemplar templates. |
| 0.2.3   | 2026-06-20 | Gap-finding pass (missing content categories, not consistency): per user direction, architecture template gained §6.5 CI/CD Pipeline, a new top-level §10 Public API & Framework Consumer Contract (SemVer/MSRV/consumer DX — framework-specific, not covered by an application-oriented template), and a Migration Safety & Rollback Policy addition to §4.5. Former §10 Appendices renumbered to §11. Updated Section 4's Architecture Specification file-split grouping (now 11 sections → still 4 files) to fold the new §10 into file 4 alongside §7–§9 and the renumbered §11. |
| 0.2.4   | 2026-06-20 | Step 2 revised to add explicit flagged-assumption attachment on stories (previously missing, unlike Steps 1/3/4/5) and to split review into two sequential passes: Pass A (story-level strike/edit/confirm) settles which stories survive before Pass B (assumption confirm/decline, scoped only to survivors) — resolving the case where a story's substance should be kept but a specific inference about it should be rejected. Added a note clarifying that Step 2's list-review passes use plain numbered text, not the selectable-options interaction, which is reserved for single discrete-choice questions elsewhere in the workflow. |
| 0.2.5   | 2026-06-20 | Added an early-resolution exception to Pass B: if the user explicitly resolves a story's flagged assumption within their Pass A reply, Claude does not re-ask it in Pass B — unless that resolution itself introduces a new, unstated assumption, which still goes through Pass B on its own. |
| 0.2.6   | 2026-06-20 | Added a standing cross-cutting rule: scope discussions (especially MVP-related) are a three-way question — not wanted at all (Architecture Spec §2.5) vs. wanted but deferred (new §2.6 Future Features) — not a binary in/out decision; Claude asks which is meant rather than guessing, unless already explicitly stated. Architecture template gained §2.6 Future Features (Deferred Scope) accordingly, with a recording mandate (originating ID, description, deferral reason, revisit precondition) and a cross-reference from §9.3 Phased Delivery Milestones. |
| 0.2.7   | 2026-06-20 | Added a standing convention for every use of the selectable-options interaction: Claude's recommended option is always listed first and explicitly labeled as a recommendation (not silently implied by position); a standing Defer option records the question for resolution at the end of the current loop, either answered then or pushed to a more appropriate later step — never silently dropped; a standing Analysis option triggers a pros/cons writeup plus Claude's recommendation and reasoning, then re-presents the same question rather than closing it. Cross-referenced from the existing Step 2 note on when the tool is/isn't appropriate. |
| 0.2.8   | 2026-06-20 | Per user clarification, replaced an incorrect assumption that Pass B's flagged assumptions and end-of-loop deferred items default to a per-item selectable-options loop. Established the actual three-way pattern instead: a single covering reply resolves everything at once; a partial bulk reply resolves only what it addresses; an explicit user request to "review individually" (or equivalent) is the only thing that starts the per-item tool loop — Claude never defaults to per-item resolution or treats an ambiguous bulk reply as an automatic trigger for it. Applied the same pattern to the Defer mechanism's end-of-loop review, which previously implied per-item review by default. Fixed an incorrect internal citation (a non-existent "Section 3.1" in this document) introduced while drafting this change. |
| 0.2.9   | 2026-06-20 | Made comparative analysis mandatory before every selectable-options question is built, not contingent on a button being clicked — recommendation-first is now explicitly grounded in that analysis rather than freestanding. Renamed the standing "Analysis" option to "Provide analysis and recommendation" and fixed its position as the always-last choice. Added a tool-capacity constraint check: the underlying tool's 4-option ceiling means 3+ substantive options forces dropping the analysis option in favor of stating the analysis inline in the question's framing text (keeping Defer); 4+ substantive options additionally requires consolidating choices or falling back to plain text entirely — addressing a real gap where the standing convention, as previously written, could not actually be satisfied by the tool once a question had 3 or more genuine options. |
| 0.3.0   | 2026-06-20 | Per user direction, added an explicit flagging requirement for whenever 3 or more substantive options survive the mandatory analysis: Claude states directly that this is one of the rarer true-fork cases, so the user can distinguish a genuine open tradeoff (most commonly an architectural decision with no dominant option) from Claude simply having failed to narrow the question. Framed as an expected-rarity check — frequent unflagged 3+-option questions would themselves indicate the analysis isn't being done rigorously. |
| 0.3.1   | 2026-06-20 | Added a category-specific default to the three-way scope rule: for simpler-vs-complex implementation choices and stackable/incremental feature-level choices specifically (commonly MVP-floor decisions), unchosen complex options default to Future Features rather than triggering the general three-way question each time, since this category's shape usually implies deferral rather than rejection. Default is stated explicitly, not applied silently, and is overridable in the same reply. Does not change the general three-way rule for scope exclusions arising any other way. |
| 0.3.2   | 2026-06-20 | Per user clarification, corrected 0.3.1's overly broad scope: implementation-level choices (algorithm/parameter-strategy selection at Step 3+, and similarly-shaped assumption clarification) are not feature-scope decisions and don't belong in Future Features. Split into two correctly-targeted rules: feature/story-level stackable choices still default to §2.6 Future Features as before; implementation-level choices now route to the architecture template's new §4.15 (noted, no commitment) or §4.16 (extensibility commitment required — a real requirement on the current implementation, not just a note), sub-grouped by originating ID. Added that Claude must actively weigh "ship simple now, commit an extension point for later" as a candidate path within the mandatory pre-analysis itself — not only after the user has chosen — especially whenever an option under consideration carries real scope or implementation-effort impact. |
| 0.4.0   | 2026-06-20 | Major structural change, worked out through user-provided scenario walkthroughs: replaced the two-pass (Pass A/Pass B) Step 2 design with the general **Grouped Closing Protocol**, applying to Step 2 and the analogous batch-producing loop in Steps 3-8. Per group: Pass A (item-level bulk review) → Stage 1 Assumption table → Stage 2 Deferred-items table → Stage 3 Remaining-ambiguous table, each stage bulk-first with individual selectable-options treatment only on explicit request, leftovers flowing forward into the next stage rather than being held back; tables for the three sources stay visually separate per stage, not merged. Individual-round selectable-options questions omit the "Provide analysis and recommendation" option as a now-redundant choice, since the analysis was already given as the reason the item reached individual treatment — carved out as an explicit, narrow exception to Section 5's otherwise-unconditional analysis-option rule. Section 5's Defer description trimmed to cross-reference Stage 2 as the authoritative mechanism rather than duplicating a now-superseded engagement-wide description. |
