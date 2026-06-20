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
    easy reference.
  - The user reviews the batch in bulk: striking stories that don't apply, editing ones that
    are close but wrong, confirming ones that are right, and adding any Claude missed.
  - After incorporating the user's corrections, Claude explicitly asks whether anything is
    still missing — preserving the spirit of `DESIGN.md`'s "any more to capture?" check —
    and treats the **user's confirmation, not Claude's own sense of coverage, as the actual
    completeness signal.** Claude does not declare the set "sufficient" on its own judgment.
  - This loop (propose batch → user corrects → confirm completeness) may repeat more than
    once if the user's corrections reveal a substantially different shape to the problem
    than the first batch assumed.
  - This replaces strict one-story-at-a-time dictation, but preserves the underlying
    guarantee `DESIGN.md` is protecting: every story is individually visible and
    individually confirmed or rejected by the user before the set is treated as final.
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
