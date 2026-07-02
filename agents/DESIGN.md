# Design and Planning Phase Guide

See `CHANGELOG.md` for full version history.

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Goal](#2-goal)
  - [2.1. Multi-File Packaging (Scale-Dependent)](#21-multi-file-packaging-scale-dependent)
- [3. Core Design Principles](#3-core-design-principles)
- [4. Agent Responsibilities](#4-agent-responsibilities)
- [5. The Integrated Design & Planning Workflow](#5-the-integrated-design--planning-workflow)
- [6. Phase Completion Criteria](#6-phase-completion-criteria)

---

## 1. Introduction
This guide outlines the unified Design and Planning Phase. This is the most critical phase for ensuring a project's success. All session-level rules are defined in `AGENTS.md` and all script and command rules are in `agents/SCRIPT_RULES.md`. Both MUST be adhered to at all times.

**Scope:** per `AGENTS.md`, this guide governs Rust projects only. The exemplar templates
referenced throughout Section 5 (`agents/exemplars/architecture_specification_template.md`,
`agents/exemplars/development_plan_template.md`, `agents/exemplars/development_checklist_template.md`,
`agents/exemplars/dev_agent_kickoff_prompt_template.md`) are written for Rust projects and
assume the conventions in `agents/PREFERRED_DEPENDENCIES.md`, `agents/PREFERRED_TOOLS.md`,
`agents/PREFERRED_SERVICES.md`, and `agents/RUST_PREFERENCES.md`.

**MANDATE:** The agent MUST create the Architecture Specification in compliance with the requirement quality criteria defined in Section 4.5. Reference to external guides (e.g., Jama Software Requirements Management Guide) is supplementary.

## 2. Goal
The goal of this phase is to produce three key artifacts, derived from user input:
1.  A comprehensive **Architecture Specification**.
2.  A detailed **Development Plan**.
3.  A granular **Development Checklist**.

These three artifacts are only considered complete once independently audited: the
Architecture Specification by Step 7 (Spec Audit), and the Development Plan and Checklist —
specifically their consistency against the finalized Architecture Specification — by Step 9
(Plan & Checklist Audit). See §5.9 for Step 9's detail.

### 2.1. Multi-File Packaging (Scale-Dependent)
For a project substantial enough to warrant it (multiple components, a large requirement set, or any case where a single file would be unwieldy to review or diff), the Architecture Specification and/or Development Plan MAY be split across multiple physical files rather than produced as one document each — consult `agents/exemplars/architecture_specification_template.md` and `agents/exemplars/development_plan_template.md` for whether the project's chosen template variant states its own file-split guidance, and `CLAUDE.md` §4 for the binding file naming convention. Where a split is used:
-   **File-split boundaries follow the template's own top-level sections**, not an invented scheme — a template that states its sections are natural file-split boundaries with section numbering stable across a split should be split along exactly those boundaries, grouped as needed to keep the number of files reasonable rather than one file per section.
-   **The first file in each artifact's set acts as an index**, containing an overview and a linked table of contents to every other file in the set (relative Markdown links, assuming co-located files unless the user states otherwise).
-   **Every other file links back to the index** and links forward/sideways to any file it has a substantive content dependency on — cross-links should follow actual dependency, not just file order, and should point at the specific section (via heading-derived anchor) rather than only the top of the target file.
-   **No single document's required deliverables are removed or reduced by adopting a split** — the split changes packaging, not content; this section does not relax any other mandate in this guide.

## 3. Core Design Principles
The final documentation set MUST adhere to the following principles:
-   **Sufficiency:** Standalone documents that enable any trained engineer/developer to succeed without additional info.
-   **Repeatability & Reproducibility:** Precise enough for different teams to arrive at highly similar end product.
-   **Testability:** Clear test criteria and exit criteria ensuring objective pass/fail assessment.
-   **Traceability:** Every unit and test must be traceable back to a specific requirement ID.

## 4. Agent Responsibilities
-   **Sole Responsibility:** The agent is solely responsible for the technical quality, completeness, and rigor of the design.
-   **Proactive Elicitation:** If information is insufficient, the agent MUST ask well-formed questions to elicit the required detail.
-   **Major Change Notification:** The agent MUST notify the user if any iteration causes a "Major Change" to the architecture. Where this arises from returning to a previously approved step (rather than from forward iteration within the current step), the explicit backtracking protocol in `AGENTS.md` §2.1 governs how the originating step is identified and how much prior work is reopened — except for findings produced by Step 7 (Spec Audit), which always follow the dedicated Step 7 Backtrack Workflow in `CLAUDE.md` §3.12 rather than the general patch-vs-reopen choice.
-   **Strict Gated Execution:** The agent is explicitly forbidden from combining steps or bypassing gates. Every step's output must be presented for approval.

### 4.5. Requirement Quality Criteria
**MANDATE: The agent MUST review each requirement against these 9 criteria.**
-   **Necessary:** The requirement is essential for the system to meet its goals.
-   **Atomic:** The requirement is a single, complete statement. It cannot be broken down further.
-   **Unambiguous:** The requirement has only one possible interpretation.
-   **Verifiable:** It is possible to determine, through objective means (testing, inspection), whether the requirement has been met.
-   **Feasible:** The requirement can be implemented with the available resources and technology.
-   **Complete:** The requirement fully describes the necessary functionality.
-   **Consistent:** The requirement does not conflict with any other requirement.
-   **Design-independent:** The requirement describes *what* the system must do, not *how* it should do it.
-   **Traceable:** The requirement can be linked to its origin and to the design, implementation, and test elements that satisfy it.

## 5. The Integrated Design & Planning Workflow
The Design Phase is a gated, 9-step lifecycle. **MANDATE: The agent MUST STOP and request User Approval after EVERY step.** The user may explicitly approve a step or request to return to any previous step to refine the design.

**Every step — and every Pass within Step 2, and every repeated run of a step (whether
user-requested per §5's "another pass" mechanism or an automatically-triggered re-run such
as a Step 7 re-audit) — runs in its own, separate agent session, with no memory of any prior
session.** This is a hard rule, not a default; see `CLAUDE.md` §1 for the Advisory Mode
consequence (every Step Approval Gate packages a complete file set and handoff note for the
next session, not just the step's headline deliverable) and `CLAUDE.md` §3.11 for the
concrete file-delivery mechanics.

**Shared mechanism (Steps 1, 3, 4, 5):** within these steps, mechanical work (checkable
against an explicit, objective standard) is delegated to the agent unsupervised; substantive
work (depending on facts about the user's actual needs/environment the agent cannot verify)
is drafted by the agent but every assumption made is explicitly flagged, never folded in
silently, per `AGENTS.md` §3.3's elicitation mechanism for the agent's operating mode. The
user's explicit confirmation, not the agent's own judgment, settles what's flagged. Each
step below states only what's mechanical and what's substantive *for that step* — this
principle itself is not restated per step.

### 5.1. Step 1: Concept Intake & Context Mapping
- **Input:** User provides a concept statement or references repository documents.
- **Process:** Map the "Problem Space" and identify system boundaries. Mechanical: structuring the concept statement into a problem-space description and boundary map. Substantive: actual scope and real boundaries — the highest-cost place in the workflow for an unflagged wrong assumption, since every later step inherits it; never autonomous, in any mode. Always attempts deep web research and competitive analysis (how comparable systems solve the same problem), aiming at best-in-class/competitive-advantage/novel-capability framing — an honest "no meaningful competitive landscape" is an acceptable reported outcome, not a fabricated comparison. Actively solicits screen mockups, reference HTML, brand/image assets, and (if relevant) audio/video assets, as early as possible — not just passively accepting them if offered, and not deferred to when UI work becomes imminent. See `CLAUDE.md` §3.8 for the full asset-tracking mechanism (Asset Manifest, filename-stable handoff into the development repository's `assets/` directories, HTML treated as presumptively authoritative for structure/behavior).
- **Output:** Architecture Specification file `_01_introduction` (Draft) — see `CLAUDE.md`
  §4.1 for the full 8-file Architecture Specification structure this and every later step
  contributes to.
- **GATE: STOP and Present `_01_introduction` (Draft) for User Approval.**

### 5.2. Step 2: The User Story Elicitation Loop
- **Process:** Iterative elicitation of User Stories, per `AGENTS.md` §3.3's elicitation mechanism for the agent's operating mode. Runs as a **single session** — the story list and each confirmed story's Interaction Sequence detail are drafted and closed together in one Grouped Closing Protocol pass, not split into separate named passes requiring separate sessions. Stories are organized by actor/persona and theme as they accumulate; each carries a stable User Story ID at first draft (`US-[DOMAIN]-NNN`), never renumbered.
- **Iteration Mandate:** Continues, regardless of mode, only until the user explicitly confirms no more stories remain — the agent's own sense of sufficiency never substitutes.
- **Output:** Architecture Specification content incorporating both the User Story list and interaction sequences into Section 2.2.
- **GATE: STOP and Present the refined Architecture Specification content for User Approval.**

### 5.3. Step 3: Recursive Requirement Decomposition (The Three-Pass Loop)
- **Process:** Decompose high-level goals into atomic units via three passes: Functional Scope → Logical Decomposition → Detailed Specification.
- **Iterative Decomposition Mandate:** Review each requirement against the 9 criteria (§4.5); iteratively decompose any that don't meet all criteria until atomic and stable. Mechanical: the 9-criteria check itself. Substantive: an invented detail needed to satisfy a criterion (often Atomic/Complete) is flagged, not folded silently in; necessity/correctness/feasibility are never settled by the check alone.
- **Mandate:** Scan for and eliminate "Requirement Smells," including the Rust-specific smells in `agents/RUST_PREFERENCES.md` §2.
- **Output:** Draft Requirements List; each requirement carries a stable Requirement ID at first draft, never renumbered (`AGENTS.md` §2.1).
- **GATE: STOP and Present Draft Requirements List for User Approval.**

### 5.4. Step 4: Iterative Questioning & Test Identification
- **Process:** Identify Test Cases, Definitions of Done (DoD), and Verification Criteria for every requirement.
- **Loop Mandate:** Work through requirements systematically. Mechanical: deriving test cases from an already-verifiable requirement. Substantive: what bar a DoD must clear — "Verifiable" (§4.5) only guarantees *some* test exists, not which one — flagged where assumed rather than stated.
- **Output:** Test Strategy Mapping; each Test Case carries a stable ID at first draft, never renumbered.
- **GATE: STOP and Present Test Strategy Mapping for User Approval.**

### 5.5. Step 5: The Decomposition Gate & Verification Feasibility
- **Feasibility:** Identify specific tools and artifacts per requirement. Mechanical: matching a need to a tool *category*; checking Rust library dependencies against `agents/PREFERRED_DEPENDENCIES.md` (preferred used without confirmation; **Forbidden** never proposed; **Requires-Approval**/unlisted flagged); checking development tools against `agents/PREFERRED_TOOLS.md`; checking infrastructure services against `agents/PREFERRED_SERVICES.md`; **setting the project's workspace MSRV** per `agents/RUST_PREFERENCES.md` §0. Substantive: real environment feasibility (infra, budget, licensing, team familiarity) for anything not list-checkable — flagged, not asserted.
- **Mandate:** The agent determines technical sufficiency for development — presented as a proposal pending confirmation, never certified unilaterally where unverifiable real-environment facts are involved.
- **Output:** Verified Requirements & V&V Protocol.
- **GATE: STOP and Present Verified V&V Protocol for User Approval.**

### 5.6. Step 6: Final Architecture Synthesis (ISO 42010 Viewpoints)
- **Process:** Final update to the Architecture Specification using **SysML and formal engineering notation** where appropriate. Mandatory viewpoints: Functional, Information (Data Dictionary), Deployment, Interface Control (ICD). Primarily recombination of already-approved content, not new elicitation — less to delegate than earlier steps. One live judgment call: formal notation vs. prose, per element — the agent states briefly why an element got one or the other, so the user can override per-element without re-litigating the viewpoint.
- **Output:** Final Architecture Specification.
- **GATE: STOP and Present Final Architecture Specification for User Approval.**

### 5.7. Step 7: Spec Audit & Phase-End Quality Assurance
- **No propose-with-flagged-assumptions here — by design.** An audit's value depends on adversarial independence from the rest of the process, including the agent's own prior work; this is the last systematic check before anything is built on the Spec. The agent audits on its own analysis, not by asking the user to confirm assumptions, and presents findings as findings.
- **Process:** "Blind spot" review and comprehensive Assurance Review. Ensure all User Stories map to requirements and all requirements are atomic. Verify every deliverable file's name conforms to the naming convention in `CLAUDE.md` §4.
- **Content Continuity Check:** Verify no previously approved technical detail, diagrams, or requirements were elided or summarized; the process MUST be strictly additive unless deletion was explicitly requested; no section replaced with a "previous versions" reference.
- **Anti-Stub Mandate:** Verify no logical gap could force a stubbed or partial implementation.
- **Notification:** Notify the user of any "Major Changes" from iterations.
- **Output:** Final Deficiency Audit Report.
- **GATE behavior depends on the report's findings:**
  - **Zero findings:** the gate behaves normally — STOP, present the clean Audit Report, await User Approval to proceed to Step 8.
  - **Any finding at all (even one):** the normal approval gate does NOT apply. Instead, the
    mandatory **Step 7 Backtrack Workflow** (`CLAUDE.md` §3.12) governs: every finding is
    traced to its originating step (1–6); the agent reopens the *earliest* originating step
    and works forward, fixing each step's findings and any cascading effects on later
    already-approved content, ending with a fresh package of all touched files and a handoff
    note for a brand-new Step 7 session. That new session re-audits the Spec independently
    from scratch — it does not take the backtrack session's own account on faith. This
    repeats until a Step 7 run produces zero findings.

### 5.8. Step 8: Development Plan & Checklist Generation
- **Plan:** Create the dev plan files using `agents/exemplars/development_plan_template.md`.
- **Environment/Configuration Elicitation:** Before drafting environment/config content (toolchain, local setup, CI — not addressed by Steps 1-7, which are about *what*, not *where/how built*): elicit concrete facts directly; for anything unspecified with a reasonable default, propose the default as a flagged assumption. Once per Plan, not once per phase.
- **Phase Sizing Mandate:** Each phase must be completable within a single agent session, sized for an agent less capable than the one performing this Design Phase, with margin for unexpected complications. See `CLAUDE.md` §3.4 (Step 8) for the complexity-scoring formula and current ceiling. **Per `AGENTS.md` §2.8, a Development Phase session completes at most one phase regardless of phase size** — sizing governs how much fits comfortably in a session, not whether multiple phases may be attempted in one.
- **Frontend Targeted Interleaving:** Where the project has a human-facing UI component, phase sequencing does not build the entire backend before any frontend work, nor push all frontend work into a single trailing phase. Instead, each screen/component's frontend implementation task is placed in the same phase as the real (non-mock) backend/data dependency it needs — never earlier (which would force a throwaway stub, contradicting the Anti-Stub Mandate) and never artificially deferred once its real dependency is available. See `agents/exemplars/development_plan_template.md` §6/§9.1 for the concrete sequencing mechanics this principle drives.
- **Checklist:** Generate a task-level checklist using `agents/exemplars/development_checklist_template.md`.
- **Kickoff Prompt:** Generate the reusable development-agent kickoff prompt using `agents/exemplars/dev_agent_kickoff_prompt_template.md`.
- **Output:** Development Plan, Checklist, and Kickoff Prompt.
- **GATE: STOP and Present Plan, Checklist, and Kickoff Prompt for User Approval.**

### 5.9. Step 9: Plan & Checklist Audit
- **No propose-with-flagged-assumptions here — by design, mirroring Step 7's independence.** Step 8's Plan and Checklist are produced by the same propose-with-flagged-assumptions mechanism as every other step; nothing in the workflow so far has independently verified them against the finalized Architecture Specification or against their own internal Definition of Done. Step 9 closes that gap the same way Step 7 closes it for the Spec: an adversarial, independent check before the Plan is handed to a development agent, not a restatement of Step 8's own reasoning.
- **Process:** Execute `agents/exemplars/development_plan_template.md` §15 (Plan-Level Definition of Done) directly as an audit checklist, on Claude's own analysis: every Core-status requirement ID from Architecture Specification §3 traced in at least one Plan task; no orphan requirement citations; every phase has non-empty Entry/Exit Criteria; every task has a non-empty Verification Method and DoD; the Phase Dependency Graph is acyclic and fully reachable; the Development Checklist contains exactly one line per task DoD item (no drift); every deliverable file's name conforms to `CLAUDE.md` §4. Additionally cross-checks phase-to-Build-Order mapping fidelity against Architecture Specification §9.2, and — where a UI component exists — that phase sequencing actually reflects the Frontend Targeted Interleaving principle (§5.8 above) rather than a backend-then-frontend split.
- **Output:** Plan & Checklist Audit Report.
- **GATE behavior depends on the report's findings, mirroring Step 7 (`CLAUDE.md` §3.12's mechanics, applied here):**
  - **Zero findings:** the gate behaves normally — STOP, present the clean report, await User Approval. This closes the Design Phase (§6).
  - **Any finding at all (even one):** the normal approval gate does not apply. Every finding is classified as it's found:
    - **(A) Plan/Checklist-only defect** (bad phase sizing, a task missing a Verification Method, an orphan citation the Plan itself introduced, a Checklist/Plan drift) — reopen Step 8 alone in a new backtrack session, fix, re-package, and proceed to a fresh Step 9 session (`pass2`, `pass3`, ... per `CLAUDE.md` §3.11's naming convention) that re-audits independently from scratch.
    - **(B) Spec-originating defect** (the Plan surfaces a genuinely missing or inconsistent requirement that Step 7 should have caught, or that was only discoverable once Plan-level task decomposition exposed it) — this is equivalent in severity to a re-architecting escalation (`agents/exemplars/development_plan_template.md` §13.1(B)). Reopen the relevant Architecture Specification step (1–6), then **re-run Step 7 to a clean result** before returning to Step 8, rather than patching the Plan around a still-defective Spec.
  - This repeats until a Step 9 run produces zero findings.

### 5.10. Autonomy Toggle and Named Presets

The user may invoke autonomous (silent-process, gate-output-only) handling for eligible
steps (2, 3, 4+5, 6, 8 — never 1, 7, or 9), individually or via two named presets (Tailored
Mode: Step 2 skipped; Full Autonomous Mode: Step 2 runs autonomously) — explicitly invoked,
not a default. Every gate above still applies; only the visibility of process within the
steps it touches changes. Step 9, like Step 7, can never be autonomous in any mode, for the
same independence rationale. See `CLAUDE.md` §3.7 for the concrete per-step/per-preset
behavior and gate output format (this guide doesn't restate it, to avoid two sources of
truth for the same mechanism).

## 6. Phase Completion Criteria
Phase is complete ONLY when all approved artifacts (Spec, Plan, Checklist, Kickoff Prompt)
are committed to the repository **and both Step 7 (Spec Audit) and Step 9 (Plan & Checklist
Audit) have each produced a clean, zero-finding report.**

---

## Appendix
See `CHANGELOG.md` for this file's full version history.
