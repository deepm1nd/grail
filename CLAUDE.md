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

`DESIGN.md` references three template files that do not yet exist in this repo:
- `agents/exemplars/architecture_specification_template.md`
- `agents/exemplars/development_plan_template.md`
- `agents/exemplars/development_checklist_template.md`

The user is creating these. Until a template is provided:
- Claude will structure documents directly from `DESIGN.md`'s own mandates (ISO 42010
  viewpoints, the 9 requirement-quality criteria, V&V protocol structure, etc.) and state
  plainly that it is doing so in lieu of a template.
- Once the user provides a template (pasted, uploaded, or linked), Claude will conform new
  and *already-produced* documents to it, and will explicitly flag any restructuring this
  requires as a Major Change per Section 4 of `DESIGN.md`, subject to the same approval gate.

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

- **One gate at a time.** Claude will not produce Step 2's output until Step 1 is explicitly
  approved, etc. This also means Claude will not pre-draft "the whole spec" up front and
  walk it back into steps — each step is genuinely built in sequence.
- **Step 2 is a real loop.** User Stories are elicited **one at a time**. After each story,
  Claude asks "Are there any more User Stories to capture?" and continues until the user
  confirms there are none. Claude will not ask for "all your user stories" in one shot.
- **Step 3 enforces the 9 requirement-quality criteria** (Necessary, Atomic, Unambiguous,
  Verifiable, Feasible, Complete, Consistent, Design-independent, Traceable) on every
  requirement, and will iteratively decompose any requirement that fails one of them —
  this can mean multiple internal passes before a requirement is presented as final.
- **Step 4 works through requirements systematically**, not in bulk — Claude will present
  test-identification findings per requirement (or in clearly delineated batches it
  proposes and the user accepts), not as one undifferentiated wall of test cases.
- **Step 6 uses formal notation where appropriate** (SysML-style where it clarifies, not
  decorative) and **must cover all four mandatory viewpoints**: Functional, Information
  (Data Dictionary), Deployment, and Interface Control (ICD).
- **Step 7 is a real audit, not a formality.** Claude will explicitly check: every User
  Story maps to a requirement; every requirement is atomic; no previously approved content
  was elided, summarized, or replaced with a "see previous version" reference; no logical
  gaps exist that could lead to stubbed/partial implementation downstream.
- **Clarification ≠ approval.** If the user's reply to a gate is a question or a request
  for a change, Claude treats that as feedback to incorporate, then re-presents the revised
  output for approval again — it does not treat the reply itself as a green light to proceed.
- **Major Changes are flagged explicitly**, by name, the moment Claude recognizes one
  resulting from an iteration — not buried in a diff or folded silently into the next
  deliverable.

---

## 4. Multi-File Output & Cross-Linking Convention

For both the Architecture Specification and the Development Plan, once a deliverable is
ready for a given gate and spans multiple files:

- **All files for one artifact are assumed to live in the same flat directory.**
  No subfolder nesting is assumed in the links below unless the user changes this.
- **File naming** uses a numeric prefix + short slug, e.g.:
  - `01_architecture_overview.md`
  - `02_functional_viewpoint.md`
  - `03_information_viewpoint_data_dictionary.md`
  - `04_deployment_viewpoint.md`
  - `05_interface_control_document.md`
  (Actual names will be finalized once the real section breakdown is known — this is
  illustrative, not committed.)
- **The first file in each set acts as an index/entry point.** It contains a short overview
  and a linked table of contents to every other file in the set, using relative Markdown
  links (e.g. `[Functional Viewpoint](./02_functional_viewpoint.md)`), since all files are
  assumed co-located.
- **Every other file links back to the index file** near the top (e.g. a breadcrumb line:
  `← [Back to Architecture Specification Index](./01_architecture_overview.md)`), and
  **links forward/sideways to adjacent or referenced files** wherever it makes a substantive
  cross-reference (e.g. the Interface Control file linking to the specific Data Dictionary
  entries it depends on, not just to the index).
- **Cross-links are based on actual content dependency, not just sequence.** If
  Section 4 of the ICD references a type defined in the Data Dictionary, that's a direct
  link to that file (and ideally that section/anchor), not just "see index."
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
- Naming/dependency/tooling mandates from `AGENTS.md` Section 2.2/2.5 that govern *code* —
  not directly relevant to producing Markdown design documents, though the underscore-not-
  hyphen convention will be used for any identifiers Claude introduces in examples or
  filenames for consistency.

---

## Appendix — Revision History

| Version | Date       | Changes |
|---------|------------|---------|
| 0.1.0   | 2026-06-19 | Initial creation, based on `AGENTS.md` and `agents/DESIGN.md` v0.2.01, plus user-specified scope (Design Phase only, 3–5 files per artifact, cross-linked, same-directory assumption). |
