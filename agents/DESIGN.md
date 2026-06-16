# Design and Planning Phase Guide
v.0.2.01

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
-   **Strict Gated Execution:** The agent is explicitly forbidden from combining steps or bypassing gates. Every step's output must be presented for approval.

## 5. The Integrated Design & Planning Workflow
The Design Phase is a gated, 8-step lifecycle. **MANDATE: The agent MUST STOP and request User Approval after EVERY step.** The user may explicitly approve a step or request to return to any previous step to refine the design.

### 5.1. Step 1: Concept Intake & Context Mapping
- **Input:** User provides a concept statement or references repository documents.
- **Process:** The agent maps the "Problem Space" and identifies system boundaries.
- **Output:** **Architecture Pass 1 (Draft)** – The agent MUST create the first draft of the `Architecture Specification` using the template at `agents/exemplars/architecture_specification_template.md`.
- **GATE: STOP and Present Architecture Pass 1 for User Approval.**

### 5.2. Step 2: The User Story Elicitation Loop
- **Process:** Iterative elicitation of User Stories. The agent MUST ask for stories **one by one**.
- **Iteration Mandate:** After each story, the agent MUST ask: "Are there any more User Stories to capture?" The loop continues until the user confirms all stories have been provided.
- **Output:** **Architecture Pass 2 (Behavioral)** – The agent MUST present a refined spec incorporating interaction sequences into Section 2.2.
- **GATE: STOP and Present refined Architecture Pass 2 for User Approval.**

### 5.3. Step 3: Recursive Requirement Decomposition (The Three-Pass Loop)
- **Process:** Agent decomposes high-level goals into atomic units using a three-pass approach: 1. Functional Scope; 2. Logical Decomposition; 3. Detailed Specification.
- **Mandate:** Scan for and eliminate "Requirement Smells."
- **Output:** Draft Requirements List.
- **GATE: STOP and Present Draft Requirements List for User Approval.**

### 5.4. Step 4: Iterative Questioning & Test Identification
- **Process:** Agent repeatedly asks well-formed questions to identify Test Cases, Definitions of Done (DoD), and Verification Criteria for every requirement.
- **Loop Mandate:** The agent must work through requirements systematically, presenting its findings for validation.
- **Output:** Test Strategy Mapping.
- **GATE: STOP and Present Test Strategy Mapping for User Approval.**

### 5.5. Step 5: The Decomposition Gate & Verification Feasibility
- **Feasibility:** Identify specific tools (Playwright, etc.) and artifacts for every requirement.
- **Mandate:** Agent determines technical sufficiency for development.
- **Output:** Verified Requirements & V&V Protocol.
- **GATE: STOP and Present Verified V&V Protocol for User Approval.**

### 5.6. Step 6: Final Architecture Synthesis (ISO 42010 Viewpoints)
- **Process:** Final update to the Architecture Specification using **SysML and formal engineering notation** where appropriate. Mandatory viewpoints: Functional, Information (Data Dictionary), Deployment, and Interface Control (ICD).
- **Output:** Final Architecture Specification.
- **GATE: STOP and Present Final Architecture Specification for User Approval.**

### 5.7. Step 7: Spec Audit & Major Change Notification
- **Process:** "Blind spot" review. Notify user of any "Major Changes" resulting from iterations.
- **Output:** Final Deficiency Audit Report.
- **GATE: STOP and Present Audit Report for User Approval.**

### 5.8. Step 8: Development Plan & Checklist Generation
- **Plan:** The agent MUST create the `*_development_plan.md` using the template at `agents/exemplars/development_plan_template.md`.
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
