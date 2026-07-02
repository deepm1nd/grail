# Development Phase Guide

See `CHANGELOG.md` for full version history.

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Goal](#2-goal)
- [3. Session Prompt](#3-session-prompt)
- [4. Phase-Specific Mandates](#4-phase-specific-mandates)
- [5. Development Workflow](#5-development-workflow)
  - [5.1. Core Development Cycle](#51-core-development-cycle)
  - [5.2. Phase-Driven Workflow](#52-phase-driven-workflow)
  - [5.3. Completion of Development](#53-completion-of-development)
  - [5.4. Post-Development Remediation Cycle](#54-post-development-remediation-cycle)

---

## 1. Introduction
This guide outlines the Development Phase. All session-level rules are defined in `AGENTS.md` and all script and command rules are in `agents/SCRIPT_RULES.md`. Both MUST be adhered to at all times.

## 2. Goal
The goal of this phase is to write, test, and build the software, following the `Development Plan` and checklist to create a feature-complete product.

## 3. Session Prompt
**MANDATE:** At the end of Design Phase Step 8, a re-invocable kickoff prompt file,
`[projectname]_dev_prompt.md` (per `agents/exemplars/dev_agent_kickoff_prompt_template.md`),
is produced once and reused verbatim at the start of every Development-Phase session. This
prompt guides the agent through the development plan, using the checklist and Phase Summary
files to track progress. Per `AGENTS.md` §2.7, the agent uses this prompt file and the
checklist file in place — it never copies, renames, or otherwise reproduces a one-off version
of either.

## 4. Phase-Specific Mandates
- **Error-Free Builds:** All code submitted to the repository MUST build without errors. Code that fails to build is considered a critical failure and must be rectified immediately. Strive to eliminate warnings as well.
- **Surgical Changes:** The agent MUST touch only what is necessary.
    - **No "Drive-by" Improvements:** Do not "improve" adjacent code, comments, or formatting that is unrelated to the task.
    - **Style Matching:** Match the existing style of the file, even if it differs from the agent's preference.
    - **Orphan Cleanup:** Remove imports, variables, or functions that the agent's changes made unused. Do not remove pre-existing dead code unless explicitly asked.
- **No Reproducing Shared Working Documents (`AGENTS.md` §2.7):** The Development Checklist
  (`[projectname]_dev_checklist.md`) and the Kickoff Prompt (`[projectname]_dev_prompt.md`)
  are edited or read in place, directly in the repository, by every session. The agent is
  explicitly forbidden from creating a copy, a renamed version, a "v2," a session-specific
  duplicate, or any other one-off reproduction of either file. If a session believes either
  file genuinely needs restructuring (not just checkbox updates), that is a Plan-Change
  Escalation (Development Plan §13/§14) — proposed and approved, then edited in place — never
  silently forked into a new file.
- **Missing Tool / Prerequisite Protocol (`AGENTS.md` §2.6):** When the agent encounters a
  missing tool or prerequisite — a different development agent's environment may not match
  what a prior session assumed — it MUST NOT simply halt and report. It attempts to install
  the missing tool for the current session (per `agents/PREFERRED_TOOLS.md` and
  `agents/PREFERRED_SERVICES.md`), **appends the install command idempotently to
  `scripts/setup_env.sh` (Linux/bash, the assumed default environment) and
  `scripts/setup_env.bat` (Windows)**, creating either file if absent, and only then informs
  the user that the prerequisite was missing and has been installed — Escalation Trigger #4
  (Development Plan §13) still fires; a successful self-install auto-resolves it rather than
  suppressing it. If no known-good install path exists, the agent escalates without guessing
  at a workaround.
- **One Phase Per Session (`AGENTS.md` §2.8):** A Development Phase session completes **at
  most one** Development Plan phase. Even with session capacity remaining after a phase's
  Exit Criteria is satisfied and its Phase Summary is written, the agent stops and awaits a
  new session rather than beginning the next phase. See §5.2 below for the concrete
  workflow this constrains.
- **Infrastructure Services via Docker:** Any infrastructure service the project depends on
  (databases, object storage, message brokers — see `agents/PREFERRED_SERVICES.md`) is run
  via Docker/`docker-compose`, with service definitions in `deploy/`, rather than installed
  directly into the development environment. If the project itself is packaged as one of
  several interrelated containerized services, its own service definition also lives in
  `deploy/`.

## 5. Development Workflow
The development process is iterative and checklist-driven.

### 5.1. Core Development Cycle: The Evidence Loop
For every individual task, which corresponds to a "software unit" as defined in the `agents/DESIGN.md` guide, the agent MUST follow the iterative build-fix loop defined in `agents/SCRIPT_RULES.md`.

**MANDATE: Goal-Driven Execution**
The agent MUST transform every task into a verifiable goal.
- **Add Validation:** Write tests for invalid inputs, then make them pass.
- **Fix Bug:** Write a test that reproduces the bug, then make it pass.
- **Refactor:** Ensure all existing tests pass before and after the change.

**MANDATE: Task-Level Evidence Capture**
Task completion is not determined by the agent's subjective assessment. For every task, the agent MUST:
1.  **Produce Required Artifacts:** Capture the specific logs, screenshots, or data outputs defined in the task's "Definition of Done (DoD)."
2.  **Verify Against DoD:** Meticulously compare the captured artifacts against the task's exit criteria.
3.  **Present Evidence & Obtain Approval:** Present the artifacts (especially screenshots) to the user and obtain explicit approval before marking the task as complete.

**Exception for Batched Tasks:** If several tasks are tightly interrelated and would be more efficient to implement at once, the agent MUST request permission from the user to batch these tasks into a single build cycle.

### 5.2. Phase-Driven Workflow
The workflow for a single Development Phase **session** is as follows. Per §4's One Phase
Per Session mandate, this workflow covers exactly one phase per session — a session never
advances into a second phase even if time/capacity remains.

1.  **Run the Session-Start Sequence:** Per the Kickoff Prompt (`[projectname]_dev_prompt.md`): review all relevant `docs\` files, run the environment check (including the pre-flight version sanity check per `agents/PREFERRED_TOOLS.md`, self-installing and recording any missing prerequisite per §4 above), and verify repository build/test state before touching any code.
2.  **Identify Current Phase:** Determine the first phase in `[projectname]_dev_checklist.md` whose Exit Criteria is not yet checked. Verify its Entry Criteria are actually true against the current repository state, not assumed from the checklist alone.
3.  **Implement All Tasks in Phase:** For each task within the current phase, in order (respecting stated dependencies), the agent must follow the **Core Development Cycle** (§5.1). Once a task is implemented and its DoD is fully satisfied, the agent updates the checklist **continuously, in place** — not batched until end of phase, and never via a copy of the checklist (§4).
4.  **Phase Integration and System Test:** After all tasks in the phase are implemented, build the system and test it using the project's actual build/test commands (Development Plan §2/§4). The agent must perform a mandatory log inspection before concluding the test outcome.
5.  **Pre-Commit Verification & Quality Assurance:**
    -   **Documentation:** Verify that all documentation is up-to-date per the **Mandate for Pre-Commit Documentation Integrity**. For the first phase of the project, this includes scaffolding the project's root `README.md` (see §5.2.1 below); for the final phase, this includes a final README review.
    -   **Assurance Review:** Perform a final, active review of all code and changes in the current phase. Ensure that all planned tasks are fully implemented and that NO partial, incomplete, or stubbed work exists.
6.  **Write the Phase Summary:** Per `agents/exemplars/development_plan_template.md` §11.3, write `[projectname]_phaseN_summary.md`.
7.  **Commit Phase Changes:** After all tests have passed and the **Phase-End Quality Assurance** is complete, the agent MUST commit (finalize and submit) all changes. This is a mandatory checkpoint to prevent work loss from session crashes.
8.  **Stop. Do Not Proceed to the Next Phase.** Per §4's One Phase Per Session mandate, the session ends here regardless of remaining capacity. Notify the user the phase is complete and await a new session to begin the next phase.

#### 5.2.1. Project README

The project's root `README.md` is scaffolded as an early task in the first Development Phase
session — derived from the already-approved Architecture Specification (overview, stack,
roadmap), not written from scratch — and is revisited for a final accuracy/completeness
review during the last phase, once the actual built system may have diverged in minor ways
from the original Spec's framing. It is not otherwise required to be touched every phase,
though any phase that materially changes how the project is built, run, or used should update
it as part of that phase's normal documentation-integrity check (§5.2 step 5).

### 5.3. Completion of Development
After all phases in the `Development Plan` are complete, the agent must notify the user and await instruction on next steps, which may include a post-development remediation cycle.

### 5.4. Post-Development Remediation Cycle
This cycle begins only when the user explicitly requests it by saying "perform an audit".
1.  **Audit and Create Remediation Plan:** The agent will perform a feature audit and create the necessary planning documents.
    -   **COMMIT POINT:** After creating all documents, the agent MUST commit them together and await further user instruction.
2.  **Execute Remediation Checklist:** The agent will execute the remediation checklist using the phase-by-phase workflow in §5.2 (including the One Phase Per Session mandate).
    -   **COMMIT POINT:** After each phase of the remediation checklist is complete and verified, the agent MUST commit the changes.
    -   Upon completion, the agent MUST notify the user and await further instructions.

## 6. Phase Completion Criteria
This phase is complete when all tasks in the `Development Plan` have been implemented, built, and committed.

---

## Appendix
See `CHANGELOG.md` for this file's full version history.
