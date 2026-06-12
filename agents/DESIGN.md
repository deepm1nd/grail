# Design and Planning Phase Guide
v.0.0.03

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Goal](#2-goal)
- [3. Core Design Principles](#3-core-design-principles)
- [4. Agent Responsibilities](#4-agent-responsibilities)
- [5. The Design and Planning Workflow](#5-the-design-and-planning-workflow)
- [Appendix R - Revision History](#appendix-r---revision-history)

---

## 1. Introduction
This guide outlines the unified Design and Planning Phase. This is the most critical phase for ensuring a project's success. All session-level rules are defined in `AGENTS.md` and all script and command rules are in `agents/SCRIPT_RULES.md`. Both MUST be adhered to at all times.

**MANDATE:** The agent MUST create the Architecture Specification in compliance with the inlined quality criteria defined in Section 5. Reference to external guides (e.g., Jama Software Requirements Management Guide) is supplementary.

## 2. Goal
The goal of this phase is to produce two key artifacts, derived from user input:
1.  A comprehensive **Architecture Specification**.
2.  A detailed **Development Plan** and corresponding **Development Checklist**.

These documents serve as the single source of truth for the system's design and are the primary input for the Development Phase.

## 3. Core Design Principles
The final documentation set (Architecture Specification and Development Plan) MUST adhere to the following principles:
-   **Sufficiency:** The documents must be standalone. Any reasonably trained engineer (for the architecture spec) or developer (for the development plan) must be able to create the intended project with a high likelihood of success without needing additional information.
-   **Repeatability & Reproducibility:** The documents must be precise and detailed enough for any two engineers or developers to arrive at a highly similar end product.
-   **Testability:** The documents must provide enough detail about how to test, test criteria, and exit criteria such that any tester would come to the same conclusion when testing a given implementation.
-   **Traceability:** The documents must provide enough detail and granularity that each unit, its functionality, and its testability can be traced through the development plan back to the architecture document and the specific requirement that specifies the unit's functions.

## 4. Agent Responsibilities
-   **Sole Responsibility:** The agent is solely responsible for the technical quality, completeness, and rigor of its work. If information is insufficient, the agent MUST ask the user well-formed questions to elicit the required detail.
-   **Elicit Detail:** If the user's initial input is insufficient to meet the criteria defined in this guide, the agent MUST provide guidance and feedback to the user to elicit more details.
-   **Major Change Notification:** Throughout the design process, the agent MUST notify the user if any input or iteration causes a "Major Change" to the architecture. The agent has the discretion to determine what constitutes a major change.
-   **No Deficient Specs:** The agent MUST NOT create an architecture specification or development plan that is deficient in any of the core principles or requirements criteria.

## 5. The Design and Planning Workflow
The design process is a highly iterative, evidence-based lifecycle.

### 5.1. Phase Initiation & Concept Intake
1.  **Concept Statement:** The user starts the session by providing a concept statement or pointing to existing concept documents in the repository.
2.  **Architecture Pass 1 (Draft):** The agent creates the first draft of the `Architecture Specification` using the template at `agents/exemplars/architecture_specification_template.md`. This pass focuses on high-level structure and system boundaries.

### 5.2. User Story Elicitation Loop
1.  **Elicit Stories:** The agent MUST ask the user for several **User Stories**.
2.  **Interaction Sequences:** For each story, the agent MUST elicit detailed interaction sequences (happy paths, edge cases, error conditions).
3.  **Architecture Pass 2 (Refinement):** The agent performs a second pass on the `Architecture Specification`, incorporating the user stories into Section 2.2 and making any structural modifications (additions, removals, changes) implied by the stories.

### 5.3. Progressive Elaboration & Decomposition Loop
The agent is solely responsible for ensuring requirements reach atomic stability.
1.  **Three-Pass Decomposition:** The agent breaks high-level goals into:
    -   **Pass 1: Functional Scope.**
    -   **Pass 2: Logical Decomposition.**
    -   **Pass 3: Detailed Specification.** (May be relaxed for simple projects).
2.  **Requirement Smell Detection:** Scan for subjective terms (e.g., "fast," "easy") and ask questions to replace them with quantified metrics.
3.  **Iteration & Questioning:** The agent repeatedly asks the user questions to refine functional/non-functional requirements and to identify specific **Test Cases**, **Definitions of Done (DoD)**, and **Verification Criteria**.
4.  **The Decomposition Gate:** The agent MUST NOT proceed until requirements are atomic and verifiable, unless the user explicitly overrides this rigor.

**MANDATE: Requirement Quality Criteria**
Every requirement MUST be: Necessary, Atomic, Unambiguous, Verifiable, Feasible, Complete, Consistent, Design-independent, and Traceable.

### 5.4. Final Architecture & Verification Review
1.  **Architecture Pass 3 (Final):** The agent performs a final pass of updates to the `Architecture Specification`, ensuring all atomic requirements and test cases are fully mapped.
2.  **Verification Feasibility Study:** For every requirement, the agent must identify the tool (Playwright, `grep`) and artifact (screenshot, log segment) that will prove completion.
3.  **Deficiency Audit:** The agent performs a final review of the spec looking for gaps in testing, validation, or structural integrity.

### 5.5. Step 3: Create the Development Plan
Once the Architecture Specification is complete, the agent will create the `*_development_plan.md` document using the template at `agents/exemplars/development_plan_template.md`.

**MANDATE: Unit-Level Decomposition**
The development plan MUST decompose the architecture into **units** based on logical responsibility and testability. Each unit must have specific, traceable requirements.

## 6. Phase Completion Criteria
This phase is complete when the approved Architecture Specification, Development Plan, and Development Checklist are committed to the repository.

---

## Appendix R - Revision History
| Version | Date       | Author      | Changes                               |
|---------|------------|-------------|---------------------------------------|
| 0.0.01  | 2025-10-10 | Jules       | Initial creation of the unified guide. |
| 0.0.02  | 2025-10-10 | Jules       | Complete overhaul to enforce rigor.   |
| 0.0.03  | 2025-10-12 | Jules       | Integrated iterative 6-step workflow. |
