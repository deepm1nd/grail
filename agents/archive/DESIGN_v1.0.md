# Design and Planning Phase Guide
v.0.0.02

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Goal](#2-goal)
- [3. Core Design Principles](#3-core-design-principles)
- [4. Agent Responsibilities](#4-agent-responsibilities)
- [5. The Design and Planning Workflow](#5-the-design-and-planning-workflow)
  - [5.1. Step 1: Elicit and Define High-Quality Requirements](#51-step-1-elicit-and-define-high-quality-requirements)
  - [5.2. Step 2: Create the Architecture Specification](#52-step-2-create-the-architecture-specification)
  - [5.3. Step 3: Create the Development Plan](#53-step-3-create-the-development-plan)
- [Appendix R - Revision History](#appendix-r---revision-history)

---

## 1. Introduction
This guide outlines the unified Design and Planning Phase. This is the most critical phase for ensuring a project's success. All session-level rules are defined in `AGENTS.md` and all script and command rules are in `agents/SCRIPT_RULES.md`. Both MUST be adhered to at all times.

**MANDATE: The agent MUST create the Architecture Specification in compliance with the principles and practices outlined in Sections 1-8 of the Jama Software Requirements Management Guide, which can be found at https://www.jamasoftware.com/requirements-management-guide/**

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
-   **Elicit Detail:** If the user's initial input is insufficient to meet the criteria defined in this guide, the agent MUST provide guidance and feedback to the user to elicit more details. The agent may ask questions like, "Please provide a use case for XYZ feature," or "Can you clarify the expected performance under these conditions?"
-   **No Deficient Specs:** The agent MUST NOT create an architecture specification or development plan that is deficient in any of the core principles or requirements criteria. It must provide feedback to the user indicating any insufficiency and work with the user to resolve it.
-   **Sole Responsibility:** It is the agent's sole responsibility to ensure that once an architecture specification is complete and accepted, the development plan created from it meets the detail, precision, and technical depth required to satisfy the core principles.

## 5. The Design and Planning Workflow

### 5.1. Step 1: Elicit and Define High-Quality Requirements
The foundation of any successful project is a set of high-quality requirements. The agent must work with the user to define both Functional and Non-Functional Requirements.

**MANDATE:** Each requirement the agent defines MUST be reviewed against the following quality criteria before being presented to the user:
-   **Necessary:** The requirement is essential for the system to meet its goals.
-   **Atomic:** The requirement is a single, complete statement. It cannot be broken down further.
-   **Unambiguous:** The requirement has only one possible interpretation.
-   **Verifiable:** It is possible to determine, through objective means (testing, inspection), whether the requirement has been met.
-   **Feasible:** The requirement can be implemented with the available resources and technology.
-   **Complete:** The requirement fully describes the necessary functionality.
-   **Consistent:** The requirement does not conflict with any other requirement.
-   **Design-independent:** The requirement describes *what* the system must do, not *how* it should do it.
-   **Traceable:** The requirement can be linked to its origin and to the design, implementation, and test elements that satisfy it.

### 5.2. Step 2: Create the Architecture Specification
Using the high-quality requirements as input, the agent will create the `*_architecture_specification.md` document. The agent MUST use the template located at `agents/exemplars/architecture_specification_template.md`. The specification MUST include extensive use of diagrams, including SysML diagram types where appropriate (e.g., Use Case, Block Definition, Sequence diagrams).

### 5.3. Step 3: Create the Development Plan
Once the Architecture Specification is complete, the agent will create the `*_development_plan.md` document using the template at `agents/exemplars/development_plan_template.md`.

**MANDATE: Unit-Level Decomposition**
The development plan MUST decompose the architecture and its requirements down to the **unit** level.
-   **Definition of a Software Unit:** A software unit is the smallest, indivisible component of a software program, defined according to a specific standard or architecture. A unit is a finite group of operations that together perform a function or task. In no case should a unit be larger than a single source code file.
-   **Unit Requirements:** Each unit defined in the development plan must have its own specific, traceable requirements derived from the main architecture specification.

## 6. Phase Completion Criteria
This phase is complete when:
1.  The `Architecture Specification` has been created and approved by the user.
2.  The `Development Plan` has been created and approved by the user.
3.  The `Development Checklist` has been generated from the development plan.

---

## Appendix R - Revision History
| Version | Date       | Author      | Changes                               |
|---------|------------|-------------|---------------------------------------|
| 0.0.01  | 2025-10-10 | Jules       | Initial creation of the unified guide. |
| 0.0.02  | 2025-10-10 | Jules       | Complete overhaul to enforce rigor.   |