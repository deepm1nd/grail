# Verification Phase Guide
v.0.0.01

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Goal](#2-goal)
- [3. The Verification Lifecycle](#3-the-verification-lifecycle)
  - [3.1. Stage 1: Verification Planning](#31-stage-1-verification-planning)
  - [3.2. Stage 2: Verification Script Development](#32-stage-2-verification-script-development)
  - [3.3. Stage 3: Test Execution](#33-stage-3-test-execution)
- [Appendix R - Revision History](#appendix-r---revision-history)

---

## 1. Introduction
This guide outlines the Verification Phase, a critical step that follows the Development Phase. This phase ensures that the implemented software meets the precise acceptance criteria laid out in the `Architecture Specification`.

## 2. Goal
The goal of this phase is to systematically verify every functional and non-functional requirement through automated tests and user-approved evidence. Successful completion of this phase provides high confidence that the system is ready for the Release Phase.

## 3. Session Prompt
**MANDATE:** At the beginning of this phase, the `p_verification.md` prompt must be created. This prompt will serve as the entry point for the verification work and will reference the `Architecture Specification`, `verification_plan.md`, and `verification_checklist.md`.

## 4. The Verification Lifecycle
The Verification Phase is divided into four distinct stages.

### 4.1. Stage 1: Preparation
-   **Feature Audit:** A comprehensive audit of the codebase is performed to ensure all planned features from the preceding phases have been implemented correctly.
-   **Deployment Tooling:** Any tools or packages needed for the final deployment are prepared.

### 4.2. Stage 2: Verification Planning
The first stage is to create a comprehensive plan for the verification process.
1.  **Create Verification Plan:** The agent must create a `verification_plan.md` document. This plan will outline the scope, strategy, and schedule for verifying the requirements.
2.  **Create Verification Checklist:** The agent must create a `verification_checklist.md` document. This checklist will list every Requirement ID from the `Architecture Specification` and will be used to track the verification status of each item.
3.  **Create Verification Prompt:** The agent must create a `p_verification.md` prompt. This prompt will serve as the entry point for the verification work and will reference the `Architecture Specification`, `verification_plan.md`, and `verification_checklist.md`.

### 4.3. Stage 3: Verification Script Development
This stage focuses on creating the automated scripts necessary to execute the verification tests.
1.  **Create Test Runner:** The agent MUST create a primary test runner script at `scripts/verification/run_verification_test.sh`.
    -   **Parameterized Execution:** This script MUST be designed to accept a Requirement ID (e.g., `PROJ-FUNC-REQ-0001`) as a command-line parameter.
    -   **Orchestration:** The script will use this parameter to call the specific verification script corresponding to that requirement.
2.  **Develop Individual Verification Scripts:** For each requirement, the agent will develop a dedicated verification script.
    -   These scripts will use appropriate tools (e.g., Playwright for UI tests, custom scripts for API tests) to execute the test and capture evidence.
    -   Evidence can be a screenshot, a log file, or other console output that proves the acceptance criterion has been met.

### 4.4. Stage 4: Test Execution
This stage is an iterative cycle focused on producing a stable, high-quality, and user-approved product.
-   **Workflow:** The workflow is **change -> system build -> system test**. The agent MUST adhere to the **SCRIPT EXCLUSIVITY MANDATE** at all times.
-   **Test Execution:** The agent will run the test runner for a specific Requirement ID: `scripts/verification/run_verification_test.sh [REQUIREMENT_ID]`.
-   **Log Inspection:** Before presenting results, the agent MUST meticulously inspect all relevant logs to confirm the test outcome.
-   **Evidence Presentation:**
    -   **MANDATE: The agent is explicitly forbidden from marking any test, acceptance criterion, or requirement as complete.** This decision rests solely with the user.
    -   **MANDATE: Visual Verification.** For any user-facing feature, the agent MUST provide visual evidence using Playwright-generated screenshots or screen recordings. Use `frontend_verification_complete(screenshot_path: "path/to/screenshot.png")`.
    -   **MANDATE: Log Capture.** For back-end or logic-heavy features, the agent MUST provide filtered log captures or structured data outputs that clearly demonstrate the requirement being met.
    -   For textual evidence, the agent MUST use `message_user(message: str, continue_working: bool)`.
-   **User Approval and Iteration:**
    -   The agent must wait for the user to explicitly approve the evidence.
    -   If the user rejects the evidence, the agent must enter a `test -> verify -> fix` cycle until the user provides approval.
    -   Upon approval, the agent updates the `verification_checklist.md`.
-   **UAT Promotion:** User acceptance of all verified requirements promotes the build to the next stage (e.g., `beta`, `rc`), triggering User Acceptance Testing (UAT).

## 5. Phase Completion Criteria
This phase is complete when all requirements listed in the `verification_checklist.md` have been successfully verified and approved by the user, and the project is ready to proceed to the Release Phase.

---

## Appendix R - Revision History
| Version | Date       | Author      | Changes                               |
|---------|------------|-------------|---------------------------------------|
| 0.0.01  | 2025-10-10 | Jules       | Initial creation of the guide.        |