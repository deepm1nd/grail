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
This guide outlines the Development Phase, executed by a Jules agent session (as distinct
from the Claude-based Design Phase — see `README.md` for the overall human-facing workflow).
All session-level rules are defined in `AGENTS.md` and all script and command rules are in
`agents/SCRIPT_RULES.md`. Both MUST be adhered to at all times.

## 2. Goal
The goal of this phase is to write, test, and build the software, following the `Development Plan` and checklist to create a feature-complete product.

## 3. Session Prompt
**MANDATE:** At the end of Design Phase Step 8, a re-invocable kickoff prompt file,
`[projectname]_dev_prompt.md` (per `agents/exemplars/dev_prompt_template.md`),
is produced once and reused verbatim at the start of every Development-Phase session. This
prompt guides the agent through the development plan, using the checklist and Phase Summary
files to track progress. Per `AGENTS.md` §2.7, the agent uses this prompt file and the
checklist file in place — it never copies, renames, or otherwise reproduces a one-off version
of either.

## 4. Phase-Specific Mandates
- **Error-Free Builds:** All code submitted to the repository MUST build without errors. Code that fails to build is considered a critical failure and must be rectified immediately. Strive to eliminate warnings as well. **The sole exception is a WIP Checkpoint** (`AGENTS.md` §2.1) — a `submit` explicitly not claiming task DoD, prefixed `[WIP-CHECKPOINT]`, used only as a crash-safety net during a long/risky or not-yet-converging task. This exception is scoped to WIP checkpoints alone and never extends to a task-complete or Code-only submit, both of which must still build and (per their own DoD) pass verification.
- **Surgical Changes:** The agent MUST touch only what is necessary.
    - **No "Drive-by" Improvements:** Do not "improve" adjacent code, comments, or formatting that is unrelated to the task.
    - **Style Matching:** Match the existing style of the file, even if it differs from the agent's preference.
    - **Orphan Cleanup:** Remove imports, variables, or functions that the agent's changes made unused. Do not remove pre-existing dead code unless explicitly asked.
- **Agent Tool Tiers and Submit Mechanics:** All platform tool usage (submit, file
  deletion/reset/restore, PR-comment tools) follows `agents/AGENT_TOOL_POLICY.md`. Code
  review invocation follows `AGENTS.md` §2.2's Code Review Policy (bypassed by default).
- **Ask-on-Uncertainty, Scoped Below Escalation Severity:** `AGENTS.md` §2.3/§2.4.1's
  standing mandate — ask well-formed questions rather than assume, surface confusion rather
  than silently guess — applies here too, but is explicitly two-tiered for a Development
  Phase session:
  - **Task-level implementation detail** (a minor unspecified choice the current task's own
    DoD/Design Refs don't resolve — e.g. which of two equivalent helper-function names,
    formatting a log line, an ordering choice with no behavioral consequence): ask directly,
    in-session, via `message_user`/`request_user_input` (`agents/AGENT_TOOL_POLICY.md`
    Allowed tier — no approval needed to ask); the user's answer resolves it and the same
    session continues the task. This is the ordinary, low-stakes case the generic mandate is
    for.
  - **Anything rising to Plan/Spec-level ambiguity** — a genuinely unclear requirement, an
    unverifiable DoR/Entry Criteria, a task whose DoD can't be satisfied as written, or any
    other item matching `agents/exemplars/development_plan_template.md` §13's Escalation
    Triggers — is **never** resolved by asking a question and continuing. Asking here would
    substitute the agent's own framing of the ambiguity for a proper Plan/Spec fix; §13
    governs instead: stop the entire session, write the Phase Summary, and let a Design
    Phase session resolve and restructure. **When genuinely unsure which tier an uncertainty
    falls into, treat it as the Escalation Trigger tier** — the safer default, since a wrongly
    continued session risks building on an unresolved ambiguity, while a wrongly stopped one
    only costs a session restart.
- **No Reproducing Shared Working Documents (`AGENTS.md` §2.7):** The Development Checklist
  (`[projectname]_dev_checklist.md`) and the Dev Prompt (`[projectname]_dev_prompt.md`)
  are edited or read in place, directly in the repository, by every session. The agent is
  explicitly forbidden from creating a copy, a renamed version, a "v2," a session-specific
  duplicate, or any other one-off reproduction of either file. If a session believes either
  file genuinely needs restructuring (not just checkbox updates), that is a Plan-Change
  Escalation (Development Plan §13/§14) — proposed and approved, then edited in place — never
  silently forked into a new file.
- **`[projectname]_dev_checklist.md` is the only `docs`/planning file a development-agent
  session is permitted to edit.** Every other file in the documentation set — the
  Architecture Specification files, the Development Plan files, the Dev Prompt, any
  handoff note or Phase Summary from a prior Design or Development session — is
  **read-only** to a development-agent session; none of them are ever modified, even to fix
  an apparent typo or inconsistency (that's an Escalation Trigger, Development Plan §13, not
  a same-session edit). Within the one file it may touch, a session's edits are further
  restricted to **marking DoD sub-items and tasks complete (`[ ]` → `[x]`, or `[D]` per the
  Checklist template's deferred-task convention) strictly within its own current, identified
  Phase (§5.2 step 2)** — appending its own Session Log row (checklist template's Session Log
  table) is likewise permitted. A session never checks, unchecks, or otherwise alters a mark
  belonging to any phase other than the one it is actively executing, and never edits the
  checklist's structural content (phase/task text, Entry/Exit Criteria wording, DoD item
  wording) — a genuine need to change checklist *content* (not just check a box) is the
  Plan-Change Escalation path above, never a same-session direct edit.
- **Escalation Model — Stop, Summarize, Wait:** Follows `agents/exemplars/
  development_plan_template.md` §13 exactly. A missing tool/prerequisite that installs
  cleanly (`AGENTS.md` §2.6) is the sole case that continues without stopping.
  **Everything else the agent cannot resolve itself — a package/version conflict, an
  install that fails, a persistent test failure, an ambiguous spec question, an
  unverifiable Entry Criteria/DoR — stops the entire session immediately**: halt all task
  work, write the Phase Summary with full diagnostic detail, leave the repository in its
  last clean committed state, and stop — no PR, no partial continuation, no further
  troubleshooting. See Plan §13 for the full trigger list and its §13.1 A/B
  (Replanning/Re-architecting) classification for Design-Phase re-engagement.
- **One Session Unit Per Session (`AGENTS.md` §2.8):** A Development Phase session completes
  **at most one** declared Session Unit — a full Phase (default), a single Task, or one
  Code/Verify sub-task, per the Plan's per-phase declaration
  (`agents/exemplars/development_plan_template.md` §6.1). Even with session capacity
  remaining after the unit's own exit condition is satisfied and its Phase Summary is
  written (or updated, for a Task/Code+Verify unit within an in-progress phase), the agent
  stops and awaits a new session rather than beginning the next unit. See §5.2 below for the
  concrete workflow this constrains.
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

**Mandatory Code/Verify Split.** Any task whose Verification Method
(`agents/exemplars/development_plan_template.md` §8) is **Build+Test** or **Hybrid** —
i.e., any task where a real test run is part of DoD — is split into two sub-tasks, ID
suffixed `a` (Code) and `b` (Verify), per that template's §8 convention. A task whose
Verification Method is pure **Visual/Behavioral** (no build-test-debug cycle) is exempt and
stays a single task. This is mechanical, derived from the Verification Method already
stated in the Plan — never a separate judgment call at execution time.
- **`a` (Code):** implement the code; DoD is satisfied by a clean build alone
  (`cargo build` or equivalent) — no test execution required to close it. Ends in its own
  Submit Point (task-complete submit for the `a` half).
- **`b` (Verify):** DoR is `a` complete and submitted. This is where the build-test-debug
  loop lives — isolated in its own session/context per the Session Unit in force
  (`AGENTS.md` §2.8). DoD is the task's original DoD (tests pass, artifacts captured).
  Ends in its own Submit Point.
- A split task counts as **two** tasks against the Phase Sizing complexity formula
  (`CLAUDE.md` §3.4 Step 8) — stated explicitly so Step 8 sizing doesn't silently overrun.

**Submit Points.** Per `agents/exemplars/development_plan_template.md` §8, every task (or
sub-task, for a split task) states its own Submit Point at drafting time. The agent submits
at minimum at every declared Submit Point — never batching multiple tasks' completions into
one end-of-phase submit. Where a task is flagged long/risky at drafting time, or where its
build-test-debug cycle is visibly not converging, the agent additionally issues a WIP
Checkpoint submit mid-task per `AGENTS.md` §2.1. **After every `submit` call, the session
stops; the user says "Continue" or "Proceed" to resume** (`agents/AGENT_TOOL_POLICY.md` §2)
— this is expected, normal flow at every declared Submit Point, not a stopping condition or
an Escalation Trigger in itself.

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

1.  **Run the Session-Start Sequence:** Per the Dev Prompt (`[projectname]_dev_prompt.md`): read the checklist, prior phase summaries, the current phase's plan section (§6.1 Phase Index + §8 current-phase tasks only), and the protocols file — in that order, using targeted extraction for the plan sections (`sed`/`grep`, never whole-file reads of large documents). Do **not** read all Architecture Specification files or all Development Plan files upfront; do **not** perform a broad repository scan. Architecture Specification and remaining plan sections are referenced on demand only, when a specific uncertainty arises during task work, using the reference table in the Dev Prompt. **Check out the current phase's Branch Name** (`agents/exemplars/development_plan_template.md` §6.1) — creating it from the default branch if it doesn't yet exist, or resuming it if a prior session already started the phase — before touching any code; never work the phase's tasks on the default branch. **Use the declared Branch Name exactly as written in Plan §6.1 — do not append, prepend, or otherwise modify it in any way**, including appending a numeric hash, timestamp, or session identifier (observed failure pattern: checking out `phase11b_hwaccel_benchmark_ladder-1251787334605801970` or `phase12b-4334437132416834814` instead of the Plan's actual declared name). `git checkout -b <exact_branch_name>` (or `git checkout <exact_branch_name>` to resume) verbatim, character-for-character. **Re-confirm this branch is still checked out before starting each subsequent task in the session**, not only at session start; a session that finds itself on the wrong branch mid-task stops and corrects it before any further code changes. Then run the environment check (pre-flight version sanity check per `agents/PREFERRED_TOOLS.md`, self-installing and recording any missing prerequisite per §4 above) and verify repository build/test state before touching any code.
2.  **Identify Current Phase and Task State:** Determine the first phase in `[projectname]_dev_checklist.md` whose Exit Criteria is not yet checked. Verify its Entry Criteria are actually true against the current repository state, not assumed from the checklist alone. **Then, for the current phase's tasks, run the three-way task-state check** (mirrored in `[projectname]_dev_prompt.md`'s "find next unit" step) against each declared Submit Point, not raw git-log archaeology:
    - **No submit exists for this task/sub-task** → not started; begin fresh.
    - **A WIP Checkpoint submit exists, no task-complete submit** → resume-in-place: check
      out the WIP state as the actual starting point (never redo from scratch, never
      discard it), and continue from exactly what its `[WIP-CHECKPOINT]` description states
      was attempted, confirmed working, and known incomplete (`AGENTS.md` §2.1). A WIP
      checkpoint whose description doesn't support this — too vague to resume from — is
      corrected before further work continues, not worked around.
    - **A task-complete submit exists** → done; move to the next task/sub-task.
    A checklist box checked with no corresponding submit, or a submit with no corresponding
    checklist entry, is an inconsistency — an Escalation Trigger (§13's model below), never
    silently patched over.
3.  **Implement the Current Session Unit** (`AGENTS.md` §2.8 — `Phase`, `Task`, or
    `Code+Verify`, as declared for this phase): for each task within scope of the session's
    declared unit, in order (respecting stated dependencies), the agent must follow the
    **Core Development Cycle** (§5.1), including the mandatory Code/Verify split and Submit
    Point cadence. **Tasks within the unit are worked in the exact order the Checklist lists
    them — no reordering, no skipping ahead to a later task, and no leaving a DoD sub-item
    unchecked-but-passed-over — without the user's explicit permission given in that
    session.** An unnecessary-seeming, already-satisfied, or blocked task/item is a
    task-level question (§4's Ask-on-Uncertainty) or an Escalation Trigger, never a silent
    skip. Once a task (or sub-task) is implemented and its DoD is fully satisfied,
    the agent, in this order: (a) appends that task's entry to
    `test/[projectname]_phase_[N]_verification.md` and drops any screenshots/clips into
    `test/phase_[N]/` per `agents/exemplars/development_plan_template.md` §11.4 (skip for
    tasks with no Verification Method beyond human review/approval); (b) updates the
    checklist **continuously, in place** — not batched until end of phase, and never via a
    copy of the checklist (§4); (c) submits at that task's declared Submit Point
    immediately, not deferred to end-of-phase, and before starting the next task.
4.  **Phase Integration and System Test:** After all tasks in the phase are implemented, build the system and test it using the project's actual build/test commands (Development Plan §2/§4). The agent must perform a mandatory log inspection before concluding the test outcome. **In addition to this build/test pass, every phase's Exit Criteria include running the full local CI-equivalent sequence** — Lint & Format, Build, Test, Coverage, Security Scan (`agents/CI.md`'s stage skeleton), the same sequence already mandated per-task at Submit Points (step 5, below) — **once more at the phase level, and resolving/fixing any bug or issue this run surfaces before the phase's Exit Criteria can be checked off.** A phase is not exited on the strength of its individual tasks' local Submit Point checks alone; the full sequence is re-run integrated, across the whole phase's combined changes, and any finding is fixed in this same phase, not deferred to the next one or left for CI's async pass to catch later.
5.  **Pre-Commit Verification & Quality Assurance:**
    -   **Mandatory Pre-Submit Local Verification** (`agents/DESIGN.md` §5.8): before any
        task-complete Submit Point, run the full local CI-equivalent sequence — Lint &
        Format, Build, Test, Coverage, Security Scan (`agents/CI.md`'s stage skeleton) —
        and confirm all stages clean. **The Security Scan's `cargo deny check`/
        `cargo audit` run in this context is read-only.** A license or advisory finding is
        reported to the user immediately as a stop-and-alert, the same as any other
        Escalation Trigger — it is not resolved by this session regenerating,
        reconciling, or otherwise touching `THIRD_PARTY_LICENSES.md`. That file has
        exactly one legitimate writer: CI's own Stage 5 regeneration step (`agents/CI.md`),
        reviewed and committed by a human. A Development Phase session's role on a license
        finding is to report it and, per the project's own stated default, propose
        swapping the offending dependency — never to edit the disclosure file itself.
    -   **Documentation:** Verify that all documentation is up-to-date per the **Mandate for Pre-Commit Documentation Integrity**. For the first phase of the project, this includes scaffolding the project's root `README.md` (see §5.2.1 below), reviewing/extending `ci.yml` (§5.2.2), creating `deny.toml` and reconciling `THIRD_PARTY_LICENSES.md` against the first real `cargo deny check licenses` run (§5.2.3); for the final phase, this includes a final README review and the full Productization Readiness Checklist (§5.2.4). **README badges are static and CI-written** (`agents/exemplars/README_template.md`'s Metrics & Badges section, `agents/CI.md` Stage 6) — there is no per-phase Branch-Name substitution for a Development session to perform; CI's Metrics Commit step rewrites the badge values on every push, from whichever branch it ran on. No Documentation-integrity DoD item exists for this any more.
    -   **Assurance Review:** Perform a final, active review of all code and changes in the current phase. Ensure that all planned tasks are fully implemented and that NO partial, incomplete, or stubbed work exists — checked continuously during the phase, not only here (`AGENTS.md` §2.3's Maximal Implementation mandate); this review is a final backstop, not the primary enforcement point.
6.  **Write the Phase Summary:** Per `agents/exemplars/development_plan_template.md` §11.3, write `[projectname]_phaseN_summary.md`.
7.  **Final Wrap-Up Submit:** Individual tasks are already submitted at their own declared Submit Points per step 3 — this step is the phase-level wrap-up only: docs, README updates, and the Phase Summary itself, submitted together after the **Phase-End Quality Assurance** is complete. This is not the sole checkpoint for the phase's work (per-task submits already provide that); it closes out anything not itself task-scoped.
8.  **Stop. Do Not Proceed to the Next Session Unit.** Per `AGENTS.md` §2.8, the session ends here regardless of remaining capacity, whether the declared Session Unit was a full Phase, a single Task, or one Code/Verify sub-task. Notify the user the unit is complete and await a new session to begin the next one. **Do not check the next Session Unit's Entry Criteria either** — a session verifies only its own Exit Criteria; the next Session Unit is always opened fresh, in a different session, which checks its own Entry Criteria itself at that time (same rule as `agents/MAINTENANCE.md` §10's Phase-boundary scope rule — reaching forward, even to glance, is out of this session's scope).

#### 5.2.1. Project README

**The project's root `README.md` is now drafted during Design Phase, at Step 8**, alongside
the Development Plan/Checklist/Dev Prompt (`agents/DESIGN.md` §5.8) — not authored from
scratch by the Development agent. The first Development Phase session's task
(Phase 0, per the Checklist template) is **review, confirmation, and enhancement** of that
already-drafted README against the actual repository as it starts to take shape — not
initial scaffolding. It is revisited for a final accuracy/completeness review during the
last phase, once the built system may have diverged in minor ways from the Design-time
draft. Any phase that materially changes how the project is built, run, or used should
update it as part of that phase's documentation-integrity check (§5.2 step 5).

#### 5.2.2. CI Workflow

**`.github/workflows/ci.yml` is drafted during Design Phase, at Step 8**, from
`agents/CI.md`'s stage skeleton, using Step 5's CI Stage Applicability findings
(`agents/DESIGN.md` §5.5/§5.8) — not authored from scratch here. Phase 0's task is
**review, confirmation, and extension** of that draft against the repository as it starts
to take shape, mirroring the README pattern above — most commonly confirming that the
conditional stages Step 5 identified (WASM, Playwright/E2E, ESP32, infra services) are
correctly present or correctly absent once the actual codebase makes that visible, and that
Stage 0's `scripts/setup_env.sh` step matches whatever `setup_env.sh` content Phase 0
itself produces or extends.

#### 5.2.3. Third-Party License Disclosure

**`deny.toml` (with its `[licenses]` allow-list, `agents/PREFERRED_TOOLS.md`) is created
during Phase 0**, not before — it has no content prerequisite of its own, but naturally
belongs alongside the workspace's `Cargo.toml`/`Cargo.lock`, which are themselves products
of Phase 0's scaffolding, not something Design Phase produces. Create it as part of
scaffolding, before running the check below.

**`THIRD_PARTY_LICENSES.md` is drafted during Design Phase, at Step 8**, from the
dependency set finalized in the Architecture Specification (`agents/DESIGN.md` §5.8) — but
unlike README/`ci.yml`, that draft is necessarily provisional: no `Cargo.lock` exists until
the workspace is actually scaffolded, so Design Step 5's own license check
(`agents/PREFERRED_DEPENDENCIES.md`'s License Compatibility Criterion) is limited to
**direct** dependencies only. **Format, both at this draft stage and at every later
regeneration: a one-line description followed by a dependency/license table (crate name,
version, license identifier) — no header/footer prose, no full license text bodies**
(`agents/CI.md` Stage 5). Phase 0's reconciliation below is a table expansion to the full
resolved tree, not a restructuring.

**Phase 0 runs the first real `cargo deny check licenses`** against the actual resolved
`Cargo.lock` — the earliest point a transitive-dependency license violation (a dependency's
own dependency carrying an incompatible license, invisible at Design time) is genuinely
catchable — and reconciles `THIRD_PARTY_LICENSES.md` against that result. Any violation
found here is an Escalation Trigger (`agents/exemplars/development_plan_template.md` §13),
not a silent fix: the agent cannot itself decide to swap a dependency or add a `[patch]`
exception, per `PREFERRED_DEPENDENCIES.md`'s No Local Patching mandate. From Phase 0
onward, `agents/CI.md` Stage 5 re-runs this check on every push and fails (does not
auto-commit) on any drift between the committed `THIRD_PARTY_LICENSES.md` and what the
current dependency tree would actually produce.

**`LICENSE.md`** (drafted at Step 8, static text) needs no reconciliation the way
`THIRD_PARTY_LICENSES.md` does — Phase 0's review is a simple accuracy check (correct
license text, correct copyright holder/year), same tier as `.gitignore`'s review.

#### 5.2.4. Productization Readiness Checklist

**Every item below is a mandatory Exit Criterion of the Final Phase** (Development Plan
§15), checked once, at the end of the project, in addition to that phase's own task-level
DoD items — not a substitute for them. This checklist is the single canonical definition
referenced by both the Final Phase's Exit Criteria and, for Maintenance Phase work on an
existing project, `agents/MAINTENANCE.md`'s Readiness Audit — defined once, here, not
restated in either place. Applicability of items 7–9 (marked *conditional*, below) is
determined at Design Step 5 (`agents/DESIGN.md` §5.5's Productization Applicability
finding) and drives which of `PROD-007`–`PROD-009` (`agents/exemplars/
development_plan_template.md` §8) are drafted into the Final Phase at Step 8.

1. **Regression Scaffold:** every Core-status Requirement ID (Spec §3) traces to ≥1
   independently re-runnable test — re-runnable by name/tag, not merely "covered somewhere
   in the Phase N verification run." The Requirement-to-Test traceability table exists as
   a committed artifact, `test/[projectname]_requirement_traceability.md`, not something
   reconstructed ad hoc from memory or grep. This table is drafted as a skeleton (one row
   per Core Requirement ID, empty test column) at Design Step 8, then updated
   incrementally every Development phase as that phase's own tasks add tests — it is a
   living artifact maintained continuously, not assembled for the first time here. **The
   concrete naming convention connecting a Test ID to an actual invocable test
   (`test_<nnnn>__description` for Rust, resolved via `cargo nextest run <id> --exact`; a
   `TEST-<nnnn>: description` title for Playwright, resolved via `-g`), and the
   verification script that confirms this mechanically rather than by assumption, are
   defined once, canonically, in `agents/MAINTENANCE.md` §6a — not restated here.**
2. **Versioning:** the repository is tagged at the current version; `CHANGELOG.md`'s
   `[Unreleased]` section is empty (every change either has a released version entry or is
   explicitly deferred and logged as such); the tag matches `CHANGELOG.md`'s latest entry.
3. **No Incomplete/Stubbed Work:** the Maximal Implementation / Anti-Stub mandate
   (`AGENTS.md` §2.3) is re-checked against the actual as-built repository at this point —
   not merely trusted from each task's own self-reported DoD — and the Open Items Register
   contains no item that should already have been reached and resolved by this phase.
4. **User-Facing Docs:** `README.md` reflects the as-built system (this item is satisfied
   by, not separate from, the existing `DOC-FINAL` task, `agents/exemplars/
   development_plan_template.md` §8).
5. **Developer-Facing Docs:** the Architecture Specification has no known undocumented
   divergence from the as-built system — any found divergence is either fixed or logged as
   an Appendix F Spec Amendment (`agents/exemplars/architecture_specification_template.md`
   Appendix F); `docs/[project_name]_dev_risks.md` is current (every Open risk
   re-evaluated against its own Re-evaluation Trigger, not left stale).
6. **License/Dependency Drift:** `cargo deny check licenses` is clean against the current
   `Cargo.lock`; `THIRD_PARTY_LICENSES.md` matches its actual output with no undisclosed
   drift.
7. **Rollback Procedure** *(conditional — only if the project has a release/deploy step)*:
   a documented, step-by-step procedure for reverting to the previous tagged version is
   actually executed once, in a non-production environment, not merely written; if the
   project owns a database schema, migration reversibility is confirmed (backward-compatible
   migrations only).
8. **Operational Runbook** *(conditional — only if the project has a running/deployed
   service component)*: documented start/stop/restart procedure, common failure symptoms
   and their remediation, and a backup/restore procedure if the service holds persistent
   data.
9. **Monitoring/Observability Baseline** *(conditional — only if the project has a
   running/deployed service component)*: structured logging (`tracing`,
   `agents/PREFERRED_DEPENDENCIES.md`) is confirmed actually emitted at runtime and
   includes a version identifier, so a future bug report can be correlated to a specific
   build. No SLO/error-budget/alerting apparatus is required unless a specific project's
   own Architecture Specification explicitly scopes one in.

A gap found in any of these nine items (or the applicable subset) at Final Phase is treated
exactly like any other unmet Exit Criterion — it blocks Phase completion, it is not
deferred past this project's own Final Verification.

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
