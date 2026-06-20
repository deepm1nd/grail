# Design and Planning Phase Guide
v.0.3.01

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Goal](#2-goal)
  - [2.1. Multi-File Packaging (Scale-Dependent)](#21-multi-file-packaging-scale-dependent)
- [3. Core Design Principles](#3-core-design-principles)
- [4. Agent Responsibilities](#4-agent-responsibilities)
- [5. The Integrated Design & Planning Workflow](#5-the-integrated-design--planning-workflow)
- [Appendix R - Revision History](#appendix-r---revision-history)

---

## 1. Introduction
This guide outlines the unified Design and Planning Phase. This is the most critical phase for ensuring a project's success. All session-level rules are defined in `AGENTS.md` and all script and command rules are in `agents/SCRIPT_RULES.md`. Both MUST be adhered to at all times.

**Scope:** per `AGENTS.md`, this guide governs Rust projects only. The exemplar templates
referenced throughout Section 5 (`agents/exemplars/architecture_specification_template.md`,
`agents/exemplars/development_plan_template.md`, `agents/exemplars/development_checklist_template.md`)
are written for Rust projects and assume the conventions in `agents/PREFERRED_DEPENDENCIES.md`
and (where present) a Rust-specific constraints guide such as `agents/RUST_PREFERENCES.md`.

**MANDATE:** The agent MUST create the Architecture Specification in compliance with the requirement quality criteria defined in Section 4.5. Reference to external guides (e.g., Jama Software Requirements Management Guide) is supplementary.

## 2. Goal
The goal of this phase is to produce three key artifacts, derived from user input:
1.  A comprehensive **Architecture Specification**.
2.  A detailed **Development Plan**.
3.  A granular **Development Checklist**.

### 2.1. Multi-File Packaging (Scale-Dependent)
For a project substantial enough to warrant it (multiple components, a large requirement set, or any case where a single file would be unwieldy to review or diff), the Architecture Specification and/or Development Plan MAY be split across multiple physical files rather than produced as one document each — consult `agents/exemplars/architecture_specification_template.md` and `agents/exemplars/development_plan_template.md` for whether the project's chosen template variant states its own file-split guidance. Where a split is used:
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
-   **Major Change Notification:** The agent MUST notify the user if any iteration causes a "Major Change" to the architecture. Where this arises from returning to a previously approved step (rather than from forward iteration within the current step), the explicit backtracking protocol in `AGENTS.md` §2.1 governs how the originating step is identified and how much prior work is reopened.
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
The Design Phase is a gated, 8-step lifecycle. **MANDATE: The agent MUST STOP and request User Approval after EVERY step.** The user may explicitly approve a step or request to return to any previous step to refine the design.

### 5.1. Step 1: Concept Intake & Context Mapping
- **Input:** User provides a concept statement or references repository documents.
- **Process:** The agent maps the "Problem Space" and identifies system boundaries. Restructuring the concept statement into a coherent problem-space description, and drafting a first-pass boundary mapping, is mechanical drafting work. Deciding what is actually *in scope* and where the system's real boundaries sit is not — a concept statement is necessarily a compressed signal, and the agent filling gaps plausibly is not the same as filling them correctly. Per `AGENTS.md` §3.3, the agent MUST explicitly flag every boundary, scope inclusion/exclusion, or actor it had to infer rather than derive from something the user actually stated, and treat the user's confirmation of those flagged items — not its own judgment — as settling them. An unflagged scope assumption here is the most expensive place in the whole workflow for one to go unnoticed, since every later step builds on it.
- **Output:** **Architecture Pass 1 (Draft)** – The agent MUST create the first draft of the `Architecture Specification` using the template at `agents/exemplars/architecture_specification_template.md`.
- **GATE: STOP and Present Architecture Pass 1 for User Approval.**

### 5.2. Step 2: The User Story Elicitation Loop
- **Process:** Iterative elicitation of User Stories, per the elicitation mechanism defined in `AGENTS.md` §3.3 for the agent's current operating mode (strict one-by-one in Autonomous Mode; propose-with-flagged-assumptions batching in Advisory Mode). In either mode, stories are organized by actor/persona and by theme (happy path, edge case, error/failure state, admin/operational) as they accumulate, and each carries a stable User Story ID assigned at first draft (`US-[DOMAIN]-NNN`), never renumbered.
- **Iteration Mandate:** The loop continues — regardless of mode — only until the user explicitly confirms no more User Stories remain; the agent's own sense that the set is sufficient in coverage, variety, or range never substitutes for that confirmation. In Autonomous Mode this confirmation is sought after each individual story ("Are there any more User Stories to capture?"); in Advisory Mode it is sought once per batch round.
- **Output:** **Architecture Pass 2 (Behavioral)** – The agent MUST present a refined spec incorporating interaction sequences into Section 2.2.
- **GATE: STOP and Present refined Architecture Pass 2 for User Approval.**

### 5.3. Step 3: Recursive Requirement Decomposition (The Three-Pass Loop)
- **Process:** Agent decomposes high-level goals into atomic units using a three-pass approach: 1. Functional Scope; 2. Logical Decomposition; 3. Detailed Specification.
- **Iterative Decomposition Mandate:** The agent MUST review each requirement against the 9 criteria in Section 4.5. It MUST iteratively decompose any requirements that do not meet all criteria until they are atomic and stable. Checking a requirement against these 9 criteria is a mechanical quality bar the agent applies unsupervised, per `AGENTS.md` §3.3's mechanical-vs-substantive delegation principle — but where satisfying a criterion (most often Atomic or Complete) requires the agent to invent a detail the source User Story did not actually specify, that invented detail MUST be flagged as an assumption for user confirmation, not folded silently into the "finished" requirement. Whether a requirement is *necessary*, *correct*, or *feasible* is never settled by the 9-criteria check alone and remains the user's call.
- **Mandate:** Scan for and eliminate "Requirement Smells."
- **Output:** Draft Requirements List, with each requirement carrying a stable Requirement ID assigned at first draft, never renumbered (per `AGENTS.md` §2.1).
- **GATE: STOP and Present Draft Requirements List for User Approval.**

### 5.4. Step 4: Iterative Questioning & Test Identification
- **Process:** Agent repeatedly asks well-formed questions to identify Test Cases, Definitions of Done (DoD), and Verification Criteria for every requirement.
- **Loop Mandate:** The agent must work through requirements systematically, presenting its findings for validation. Deriving plausible test cases from an already-atomic, already-verifiable requirement is largely mechanical; deciding what bar a Definition of Done must actually clear is not — "Verifiable" (per Section 4.5) only guarantees some objective test exists, not what that test must prove. The agent MUST explicitly flag anywhere it had to assume what "done" means rather than derive it from something the requirement or its originating User Story already stated.
- **Output:** Test Strategy Mapping, with each Test Case carrying a stable Test Case ID assigned at first draft, never renumbered.
- **GATE: STOP and Present Test Strategy Mapping for User Approval.**

### 5.5. Step 5: The Decomposition Gate & Verification Feasibility
- **Feasibility:** Identify specific tools (Playwright, etc.) and artifacts for every requirement. Matching a verification need to a *category* of tool is reasonably mechanical; whether a *specific* tool or dependency is actually feasible — available infrastructure, budget, license terms, team familiarity, fit with `agents/PREFERRED_DEPENDENCIES.md`'s preferred/forbidden/requires-approval lists — depends on facts about the user's real environment the agent cannot see or verify on its own. The agent MUST explicitly flag any tool/dependency choice that is a best guess rather than confirmed against the actual environment or `agents/PREFERRED_DEPENDENCIES.md`, and MUST treat any dependency outside that file's preferred list as requiring the user's explicit approval (or amendment of that file) before it is treated as part of the plan.
- **Mandate:** Agent determines technical sufficiency for development. This determination is presented as a proposal pending the user's confirmation, not delivered as a settled fact — the agent is not authorized to certify sufficiency unilaterally where real-environment facts it cannot verify are involved.
- **Output:** Verified Requirements & V&V Protocol.
- **GATE: STOP and Present Verified V&V Protocol for User Approval.**

### 5.6. Step 6: Final Architecture Synthesis (ISO 42010 Viewpoints)
- **Process:** Final update to the Architecture Specification using **SysML and formal engineering notation** where appropriate. Mandatory viewpoints: Functional, Information (Data Dictionary), Deployment, and Interface Control (ICD). This step is primarily *recombination* of already-approved content from Steps 1-5 rather than new elicitation, so there is less judgment to delegate than in earlier steps — but deciding what counts as "appropriate" for formal notation versus prose remains a real judgment call. The agent MUST briefly state why a given element received a formal diagram versus a prose description, so the choice is visible and the user can override it for any specific element without re-litigating the whole viewpoint.
- **Output:** Final Architecture Specification.
- **GATE: STOP and Present Final Architecture Specification for User Approval.**

### 5.7. Step 7: Spec Audit & Phase-End Quality Assurance
- **This step is a real, independent audit — not a formality, and not a propose-with-flagged-assumptions step like Steps 1-5.** Steps 1 through 5 delegate mechanical drafting to the agent while requiring user confirmation on substantive judgment calls; that pattern does not apply here. An audit's value depends on it being adversarial toward the artifact being audited, including toward the agent's own prior work in Steps 1-6 — weakening the agent's independence from the rest of the process at this step costs more than any efficiency it could buy, since this is the last systematic check before the Development Plan is built on top of whatever the Spec actually says. The agent performs this audit on its own analysis, not by asking the user to confirm assumptions, and presents findings as findings, not as proposals awaiting correction.
- **Process:** "Blind spot" review and comprehensive Assurance Review. Ensure all User Stories are mapped to requirements and all requirements are atomic.
- **Content Continuity Check:** Verify that no previously approved technical detail, diagrams, or requirements were accidentally removed, elided, or summarized during the iteration passes. The iteration process MUST be strictly additive unless deletion was explicitly requested. Ensure that no sections have been replaced with references to "previous versions."
- **Anti-Stub Mandate:** Specifically verify that no logical gaps exist that could lead to stubbed or partial implementations during development.
- **Notification:** Notify user of any "Major Changes" resulting from iterations.
- **Output:** Final Deficiency Audit Report.
- **GATE: STOP and Present Audit Report for User Approval.**

### 5.8. Step 8: Development Plan & Checklist Generation
- **Plan:** The agent MUST create the `*_development_plan.md` using the template at `agents/exemplars/development_plan_template.md`.
- **Environment/Configuration Elicitation:** Before drafting any Plan content describing the actual development/test environment (toolchain prerequisites, local configuration, CI setup, or equivalent, per whatever the Plan template's own sections require), the agent MUST elicit the concrete facts those sections need directly from the user rather than inferring plausible-sounding defaults — this content describes *where and how the system gets built*, which Steps 1-7 do not address, since those steps are about *what the system does*. Where the user has not specified something and a reasonable default exists, the agent MAY propose the default but MUST flag it as an assumption requiring confirmation, per the same flagged-assumption principle as Steps 1-5. This elicitation happens once per Plan, not once per phase.
- **Phase Sizing Mandate:** The Development Plan MUST be subdivided into phases where each phase represents a volume of work that can be comfortably completed within a single agent session. The agent MUST include a sufficient margin (buffer) in its estimation to account for unexpected technical complications or debugging cycles.
- **Checklist:** The agent MUST generate a task-level checklist using the style and format defined in `agents/exemplars/development_checklist_template.md`.
- **Output:** Development Plan and Checklist.
- **GATE: STOP and Await Final Approval of all design artifacts.**

## 6. Phase Completion Criteria
Phase is complete ONLY when all approved artifacts (Spec, Plan, Checklist) are committed to the repository.

---

## Appendix R - Revision History
| Version | Date       | Author      | Changes                               |
|---------|------------|-------------|---------------------------------------|
| 0.0.01  | 2025-10-10 | Jules       | Initial creation of the unified guide. |
| 0.0.02  | 2025-10-10 | Jules       | Complete overhaul to enforce rigor.   |
| 0.0.03  | 2025-10-12 | Jules       | Integrated iterative 6-step workflow. |
| 0.2.00  | 2025-10-12 | Jules       | DESIGN v2.0: 8-Step Gated Protocol.   |
| 0.2.01  | 2025-10-13 | Jules       | Explicit 'STOP and Ask' mandates.     |
| 0.3.00  | 2026-06-20 | Claude      | Generalized the propose-with-flagged-assumptions elicitation pattern into Steps 1, 3, 4, and 5 (mechanical drafting/checking delegated to the agent; scope, correctness, necessity, and feasibility judgments require explicit user confirmation, with every such assumption flagged rather than silently folded in); added Step 2's mode-conditional elicitation cross-reference to `AGENTS.md` §3.3; added notation-judgment flagging to Step 6; added an environment/configuration elicitation sub-step to Step 8; added §2.1 Multi-File Packaging as a scale-dependent, template-driven mechanism; cross-referenced `AGENTS.md`'s new stable-ID and explicit-backtracking rules from §4. Originated from a working arrangement (`CLAUDE.md`) built for one specific Rust project's Design Phase engagement; generalized here for the Rust-project methodology as a whole. |
| 0.3.01  | 2026-06-20 | Claude      | Added explicit Step 7 independence framing: stated directly that Steps 1-5's propose-with-flagged-assumptions pattern does not apply to the Spec Audit, since an audit's value depends on adversarial independence from the rest of the process, including the agent's own prior work. Added a Rust-only scope statement to §1 Introduction, consistent with `AGENTS.md`'s own scope statement. Tightened Step 5's dependency-feasibility language to reference `agents/PREFERRED_DEPENDENCIES.md` directly rather than hedging on whether the file exists, now that Rust-only scope is confirmed. |
