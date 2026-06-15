# Design and Planning Phase Guide
v.0.2.00

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Goal](#2-goal)
- [3. Core Design Principles](#3-core-design-principles)
- [4. Agent Responsibilities](#4-agent-responsibilities)
- [5. The Integrated Design & Planning Workflow](#5-the-integrated-design--planning-workflow)
- [Appendix R - Revision History](#appendix-r---revision-history)

---

## 1. Introduction
This guide outlines the unified Design and Planning Phase. This is the most critical phase for ensuring a project's success. All session-level rules are defined in `AGENTS.md` and all script and command rules are in `agents/SCRIPT_RULES.md`. Both MUST be adhered to at all times.

**MANDATE:** The agent MUST create the Architecture Specification in compliance with the inlined quality criteria defined in Section 5. Reference to external guides (e.g., Jama Software Requirements Management Guide) is supplementary.

## 2. Goal
The goal of this phase is to produce three key artifacts, derived from user input:
1.  A comprehensive **Architecture Specification**.
2.  A detailed **Development Plan**.
3.  A granular **Development Checklist**.

## 3. Core Design Principles
The final documentation set MUST adhere to the following principles:
-   **Sufficiency:** Standalone documents that enable any trained engineer/developer to succeed without additional info.
-   **Repeatability & Reproducibility:** Precise enough for different teams to arrive at highly similar end product.
-   **Testability:** Clear test criteria and exit criteria ensuring objective pass/fail assessment.
-   **Traceability:** Every unit and test must be traceable back to a specific requirement ID.

## 4. Agent Responsibilities
-   **Sole Responsibility:** The agent is solely responsible for the technical quality, completeness, and rigor of the design.
-   **Proactive Elicitation:** If information is insufficient, the agent MUST ask well-formed questions to elicit the required detail.
-   **Major Change Notification:** The agent MUST notify the user if any iteration causes a "Major Change" to the architecture.
-   **Gated Iteration:** The agent must not exit a workflow step without explicit user approval.

## 5. The Integrated Design & Planning Workflow
The Design Phase is a gated, 8-step lifecycle. **MANDATE: User Approval is required after every step.** The user may explicitly approve a step or request to return to any previous step to refine the design.

### 5.1. Step 1: Concept Intake & Context Mapping
- **Input:** User provides a concept statement or references repository documents.
- **Process:** The agent maps the "Problem Space" and identifies system boundaries.
- **Output:** **Architecture Pass 1 (Draft)** – The agent MUST create the first draft of the `Architecture Specification` using the template at `agents/exemplars/architecture_specification_template.md`.
- **GATE:** Await User Approval or request to return.

### 5.2. Step 2: The User Story Elicitation Loop
- **Process:** Iterative elicitation of User Stories (happy paths, edge cases, failure modes).
- **Mandate:** Agent determines sufficiency for requirements mapping.
- **Output:** **Architecture Pass 2 (Behavioral)** – Interaction sequences in Section 2.2.
- **GATE:** Await User Approval or request to return.

### 5.3. Step 3: Recursive Requirement Decomposition (The Three-Pass Loop)
- **Process:** Agent decomposes high-level goals into atomic, verifiable units.
- **Mandate:** Scan for "Requirement Smells" (subjective terms) and replace with quantified metrics.
- **Three Passes:** 1. Functional Scope; 2. Logical Decomposition; 3. Detailed Specification.
- **Output:** Draft Requirements List.
- **GATE:** Await User Approval or request to return.

### 5.4. Step 4: Iterative Questioning & Test Identification
- **Process:** Agent asks well-formed questions to identify Test Cases, Definitions of Done (DoD), and Verification Criteria for every requirement.
- **Output:** Test Strategy Mapping.
- **GATE:** Await User Approval or request to return.

### 5.5. Step 5: The Decomposition Gate & Verification Feasibility
- **The Gate:** Agent verifies "Atomic Stability."
- **Feasibility:** Identify specific tools (Playwright, etc.) and artifacts for every requirement.
- **Output:** Verified Requirements & V&V Protocol.
- **GATE:** Await User Approval or request to return.

### 5.6. Step 6: Final Architecture Synthesis (ISO 42010 Viewpoints)
- **Process:** Final update to the Architecture Specification with mandatory viewpoints:
    - **Functional View:** Responsibilities and sequences.
    - **Information View:** **Data Dictionary** and schemas.
    - **Deployment View:** Service boundaries and environment mapping.
    - **Interface Control (ICD):** Formalized contracts for all module interactions.
- **Output:** Final Architecture Specification.
- **GATE:** Await User Approval or request to return.

### 5.7. Step 7: Spec Audit & Major Change Notification
- **Process:** "Blind spot" review. Notify user of any "Major Changes" resulting from iterations.
- **Output:** Audit Report/Final Spec.
- **GATE:** Await User Approval or request to return.

### 5.8. Step 8: Development Plan & Checklist Generation
- **Plan:** The agent MUST create the `*_development_plan.md` using the template at `agents/exemplars/development_plan_template.md`. Decompose into logical Units linked to Requirement IDs. Specify Required Artifacts (Logs/Screenshots) for every task.
- **Checklist:** The agent MUST generate a task-level checklist using the style and format defined in `agents/exemplars/development_checklist_template.md`.
- **GATE:** Await Final Approval or request to return.

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
