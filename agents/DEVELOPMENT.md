# Development Phase Guide

See `CHANGELOG.md` for full version history.

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Goal](#2-goal)
- [3. Session Prompt](#3-session-prompt)
- [4. Phase-Specific Mandates](#4-phase-specific-mandates)
- [5. Development Workflow](#5-development-workflow)
  - [5.1. Core Development Cycle](#51-core-development-cycle)
  - [5.2. Task-Group-Driven Workflow](#52-task-group-driven-workflow)
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
prompt guides the agent through the development plan, using the checklist and Task Group Summary
files to track progress. Per `AGENTS.md` §2.7, the agent uses this prompt file and the
checklist file in place — it never copies, renames, or otherwise reproduces a one-off version
of either.

## 4. Phase-Specific Mandates
- **Error-Free Builds:** All code submitted to the repository MUST build without errors. Code that fails to build is considered a critical failure and must be rectified immediately. Strive to eliminate warnings as well. **The sole exception is a WIP Checkpoint** (`AGENTS.md` §2.1) — a `submit` explicitly not claiming task DoD, prefixed `[WIP-CHECKPOINT]`, used only as a crash-safety net at a Design-specified checkpoint point within a long/risky task. This exception is scoped to WIP Checkpoints alone and never extends to a Task Group's Final Wrap-Up Submit, which must still build and, per every included task's own DoD, pass verification.
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
    governs instead: stop the entire session, write the Task Group Summary, and let a Design
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
  handoff note or Task Group Summary from a prior Design or Development session — is
  **read-only** to a development-agent session; none of them are ever modified, even to fix
  an apparent typo or inconsistency (that's an Escalation Trigger, Development Plan §13, not
  a same-session edit). Within the one file it may touch, a session's edits are further
  restricted to **marking DoD sub-items and tasks complete (`[ ]` → `[x]`, or `[D]` per the
  Checklist template's deferred-task convention) strictly within its own current, identified
  Task Group (§5.2 step 2)** — appending its own Session Log row (checklist template's Session Log
  table) is likewise permitted. A session never checks, unchecks, or otherwise alters a mark
  belonging to any Task Group other than the one it is actively executing, and never edits the
  checklist's structural content (Task Group/task text, Entry/Exit Criteria wording, DoD item
  wording) — a genuine need to change checklist *content* (not just check a box) is the
  Plan-Change Escalation path above, never a same-session direct edit.
- **Escalation Model — Stop, Summarize, Wait:** Follows `agents/exemplars/
  development_plan_template.md` §13 exactly. A missing tool/prerequisite that installs
  cleanly (`AGENTS.md` §2.6) is the sole case that continues without stopping.
  **Everything else the agent cannot resolve itself — a package/version conflict, an
  install that fails, a persistent test failure, an ambiguous spec question, an
  unverifiable Entry Criteria/DoR — stops the entire session immediately**: halt all task
  work, write the Task Group Summary with full diagnostic detail, leave the repository in its
  last clean committed state, and stop — no PR, no partial continuation, no further
  troubleshooting. See Plan §13 for the full trigger list and its §13.1 A/B
  (Replanning/Re-architecting) classification for Design-Phase re-engagement.
- **One Session Unit Per Session (`AGENTS.md` §2.8):** A Development Phase session completes
  **at most one** declared Session Unit — a full Task Group (default), a single Task, or one
  Code/Verify sub-task, per the Plan's per-Task Group declaration
  (`agents/exemplars/development_plan_template.md` §6.1). Even with session capacity
  remaining after the unit's own exit condition is satisfied and its Task Group Summary is
  written (or updated, for a Task/Code+Verify unit within an in-progress Task Group), the agent
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
  (`cargo build` or equivalent) — no test execution required to close it. No longer has its
  own Submit Point — closing its DoD and flipping its Checklist box is sufficient to move to
  `b`; both sub-tasks' work is saved together at the Task Group's single Final Wrap-Up
  Submit (`AGENTS.md` §2.1).
- **`b` (Verify):** DoR is `a`'s DoD satisfied (no longer "submitted," since there is no
  intermediate submit to wait on). This is where the build-test-debug loop lives — isolated
  in its own session/context per the Session Unit in force (`AGENTS.md` §2.8). DoD is the
  task's original DoD (tests pass, artifacts captured). Like `a`, has no Submit Point of its
  own; if either half is long/risky enough to warrant a mid-task save point, that is a
  Design-specified WIP Checkpoint (`AGENTS.md` §2.1), not a sub-task-boundary submit.
- A split task counts as **two** tasks against the Task Group Sizing complexity formula
  (`CLAUDE.md` §3.4 Step 8) — stated explicitly so Step 8 sizing doesn't silently overrun.

**Submit Points.** Per `AGENTS.md` §2.1, the Session Unit's Submit Point now occurs once, at
the unit's own completion — the Task Group's Final Wrap-Up Submit (§5.2 step 7) for the
default `Task Group` Session Unit, or the equivalent completion point for a `Task` or
`Code+Verify` unit. Individual task/sub-task completions within the unit are tracked via the
Checklist and Verification File, not via an intermediate `submit`. **The only mid-unit save
point is a WIP Checkpoint, and only at the specific location Design specified for that task
at drafting time** (`agents/exemplars/development_plan_template.md` §8, `AGENTS.md` §2.1) —
there is no runtime trigger (e.g. a non-converging build-test-debug cycle) that authorizes an
agent-initiated WIP Checkpoint; that judgment call was deliberately removed rather than
bounded, and a task stuck in an unanticipated way has no save point until its Design-specified
checkpoint (if any) or the unit's own end. **After every `submit` call, the session stops; the
user says "Continue" or "Proceed" to resume** (`agents/AGENT_TOOL_POLICY.md` §2) — this is
expected, normal flow at every Submit Point, not a stopping condition or an Escalation Trigger
in itself.

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

### 5.2. Task-Group-Driven Workflow
The workflow for a single Development Phase **session** is as follows. Per §4's One Task Group
Per Session mandate, this workflow covers exactly one Task Group per session — a session never
advances into a second Task Group even if time/capacity remains.

1.  **Run the Session-Start Sequence:** Per the Dev Prompt (`[projectname]_dev_prompt.md`): read the checklist, prior Task Group summaries, the current Task Group's plan section (§6.1 Task Group Index + §8 current-Task Group tasks only), and the protocols file — in that order, using targeted extraction for the plan sections (`sed`/`grep`, never whole-file reads of large documents). Do **not** read all Architecture Specification files or all Development Plan files upfront; do **not** perform a broad repository scan. Architecture Specification and remaining plan sections are referenced on demand only, when a specific uncertainty arises during task work, using the reference table in the Dev Prompt. **Check out the current Task Group's Branch Name** (`agents/exemplars/development_plan_template.md` §6.1) — creating it from the default branch if it doesn't yet exist, or resuming it if a prior session already started the Task Group — before touching any code; never work the Task Group's tasks on the default branch. **Use the declared Branch Name exactly as written in Plan §6.1 — do not append, prepend, or otherwise modify it in any way**, including appending a numeric hash, timestamp, or session identifier (observed failure pattern: checking out `task_group11b_hwaccel_benchmark_ladder-1251787334605801970` or `task_group12b-4334437132416834814` instead of the Plan's actual declared name). `git checkout -b <exact_branch_name>` (or `git checkout <exact_branch_name>` to resume) verbatim, character-for-character. **Re-confirm this branch is still checked out before starting each subsequent task in the session**, not only at session start; a session that finds itself on the wrong branch mid-task stops and corrects it before any further code changes. Then run the environment check (pre-flight version sanity check per `agents/PREFERRED_TOOLS.md`, self-installing and recording any missing prerequisite per §4 above) and verify repository build/test state before touching any code. **Capture the current resolved `Cargo.lock` dependency set as this Task Group's starting baseline** — this is the diff basis step 4's crate-drift notification compares against at Task Group end.
2.  **Identify Current Task Group and Task State:** Determine the first Task Group in `[projectname]_dev_checklist.md` whose Exit Criteria is not yet checked. Verify its Entry Criteria are actually true against the current repository state, not assumed from the checklist alone. **Then, for the current Task Group's tasks, run the task-state check** (mirrored in `[projectname]_dev_prompt.md`'s "find next unit" step) against the Checklist and any WIP Checkpoint, not raw git-log archaeology — there is no longer a per-task submit to check against, since Submit Points now occur only at Task-Group end (`AGENTS.md` §2.1):
    - **No Checklist box checked for this task/sub-task, no WIP Checkpoint exists** → not
      started; begin fresh.
    - **A WIP Checkpoint submit exists for this task** (only possible where Design specified
      one, §8) → resume-in-place: check out the WIP state as the actual starting point (never
      redo from scratch, never discard it), and continue from exactly what its
      `[WIP-CHECKPOINT]` description states was attempted, confirmed working, and known
      incomplete (`AGENTS.md` §2.1). A WIP checkpoint whose description doesn't support this —
      too vague to resume from — is corrected before further work continues, not worked
      around.
    - **The Checklist box is checked** → done; move to the next task/sub-task. Since no
      per-task submit exists to cross-check against, a checked box's authority is the
      Checklist entry itself plus its corresponding Verification File entry
      (`development_plan_template.md` §11.4) — a checked box with no corresponding
      Verification File entry (where one is required) is an inconsistency — an Escalation
      Trigger (§13's model below), never silently patched over.
3.  **Implement the Current Session Unit** (`AGENTS.md` §2.8 — `Task Group`, `Task`, or
    `Code+Verify`, as declared for this Task Group): for each task within scope of the session's
    declared unit, in order (respecting stated dependencies), the agent must follow the
    **Core Development Cycle** (§5.1), including the mandatory Code/Verify split and the
    Design-specified WIP Checkpoint (if any) for that task. **Tasks within the unit are worked
    in the exact order the Checklist lists them — no reordering, no skipping ahead to a later
    task, and no leaving a DoD sub-item unchecked-but-passed-over — without the user's explicit
    permission given in that session.** An unnecessary-seeming, already-satisfied, or blocked
    task/item is a task-level question (§4's Ask-on-Uncertainty) or an Escalation Trigger,
    never a silent skip. Once a task (or sub-task) is implemented and its DoD is fully
    satisfied, the agent, in this order: (a) appends that task's entry to
    `test/v[N.NN.NN]/[projectname]_task_group_[N]_verification.md` and drops any screenshots/clips into
    `test/v[N.NN.NN]/task_group_[N]/` per `agents/exemplars/development_plan_template.md` §11.4 (skip for
    tasks with no Verification Method beyond human review/approval); (b) updates the
    checklist **continuously, in place** — not batched until end of Task Group, and never via a
    copy of the checklist (§4); (c) proceeds directly to the next task — **there is no
    per-task or per-sub-task `submit` call any more; the session's only `submit` is the
    Task-Group-end Final Wrap-Up Submit (step 7), or a Design-specified WIP Checkpoint if one
    is reached first (`AGENTS.md` §2.1).**
4.  **Task Group Integration and System Test:** After all tasks in the Task Group are implemented, build the system and test it using the project's actual build/test commands (Development Plan §2/§4). The agent must perform a mandatory log inspection before concluding the test outcome. **In addition to this build/test pass, every Task Group's Exit Criteria include running the full local CI-equivalent sequence** — Lint & Format, Build, Test, Coverage, Security Scan (`agents/CI.md`'s stage skeleton), the same sequence the Task Group's own Final Wrap-Up Submit (step 5, below) requires as its Mandatory Pre-Submit Local Verification — **once more at the Task Group level, and resolving/fixing any bug or issue this run surfaces before the Task Group's Exit Criteria can be checked off.** A Task Group is not exited on the strength of any individual task's own narrower Verification Method alone; the full sequence is re-run integrated, across the whole Task Group's combined changes, and any finding is fixed in this same Task Group, not deferred to the next one or left for CI's async pass to catch later.
    - **Coverage is not evidence of integration.** This run's Coverage stage (`cargo-llvm-cov`)
      confirms no regression in what currently exists — line/branch coverage measures
      execution, not validation, and an isolated unit test satisfies both build-green and
      coverage-% identically to a real cross-crate test. Passing this stage is never treated,
      here or in any Task Group Summary, as evidence that an Integration/System/
      Acceptance-type Test Case is actually satisfied — that is established solely by the
      Test-Type-to-Naming Binding and mechanical check described in
      `agents/exemplars/development_plan_template.md` §8/§11.4, never by a coverage
      percentage. `PREFERRED_TOOLS.md`'s `cargo-mutants` CI job (informational, non-blocking)
      is the recommended spot-check for whether a suite's assertions are meaningful, when
      Fix 3's Step 9 sample-audit or this Task Group's own review flags a component as
      suspicious — it is a different claim from coverage-green, not a stronger version of it.
    - **This Task Group's Security Scan run additionally distinguishes two outcomes from
      `cargo deny check licenses`, of different severity:**
      - **A license violation** — same stop-and-alert severity as the per-task rule in step 5
        below: reported to the user immediately as an Escalation Trigger, and the Task
        Group's Exit Criteria cannot be checked off until resolved. Not resolved by this
        session regenerating or reconciling `THIRD_PARTY_LICENSES.md` (step 5's rule on that
        file's sole legitimate writer applies here identically).
      - **A new crate added since this Task Group's starting `Cargo.lock`** (direct or
        transitive, diffed against the dependency set captured at this Task Group's
        session-start sequence, §5.2 step 1) — **notification only**, does not block Exit
        Criteria, no stop. Every newly-added crate is listed inline in the Task Group
        Summary — name, version, license, same table shape as `THIRD_PARTY_LICENSES.md` —
        so the user sees exactly what drifted without a manual `Cargo.lock` diff.
5.  **Pre-Commit Verification & Quality Assurance:**
    -   **Mandatory Pre-Submit Local Verification** (`agents/DESIGN.md` §5.8): before the
        Task Group's Final Wrap-Up Submit (its sole task-complete-equivalent Submit Point,
        `AGENTS.md` §2.1), run the full local CI-equivalent sequence — Lint &
        Format, Build, Test, Coverage, Security Scan (`agents/CI.md`'s stage skeleton) —
        and confirm all stages clean. This is the same run already required at step 4's
        Task Group Integration and System Test; it is not re-run a third time here where
        step 4 already covers it, but the Task Group cannot reach its Final Wrap-Up Submit
        without it having been run clean. **The Security Scan's `cargo deny check`/
        `cargo audit` run in this context is read-only.** A license or advisory finding is
        reported to the user immediately as a stop-and-alert, the same as any other
        Escalation Trigger — it is not resolved by this session regenerating,
        reconciling, or otherwise touching `THIRD_PARTY_LICENSES.md`. That file has
        exactly one legitimate writer: CI's own Stage 5 regeneration step (`agents/CI.md`),
        reviewed and committed by a human. A Development Phase session's role on a license
        finding is to report it and, per the project's own stated default, propose
        swapping the offending dependency — never to edit the disclosure file itself.
    -   **Documentation:** Verify that all documentation is up-to-date per the **Mandate for Pre-Commit Documentation Integrity**. For the first Task Group of the project, this includes scaffolding the project's root `README.md` (see §5.2.1 below), reviewing/extending `ci.yml` (§5.2.2), creating `deny.toml` and reconciling `THIRD_PARTY_LICENSES.md` against the first real `cargo deny check licenses` run (§5.2.3); for the final Task Group, this includes a final README review and the full Productization Readiness Checklist (§5.2.4). **README badges are static and CI-written** (`agents/exemplars/README_template.md`'s Metrics & Badges section, `agents/CI.md` Stage 6) — there is no per-Task Group Branch-Name substitution for a Development session to perform; CI's Metrics Commit step rewrites the badge values on every push, from whichever branch it ran on. No Documentation-integrity DoD item exists for this any more.
    -   **Assurance Review:** Perform a final, active review of all code and changes in the current Task Group. Ensure that all planned tasks are fully implemented and that NO partial, incomplete, or stubbed work exists — checked continuously during the Task Group, not only here (`AGENTS.md` §2.3's Maximal Implementation mandate); this review is a final backstop, not the primary enforcement point.
6.  **Write the Task Group Summary:** Per `agents/exemplars/development_plan_template.md` §11.3, write `dev/plan/[projectname]_task_group_[N]_summary.md`.
7.  **Final Wrap-Up Submit:** This is now the Task Group's **sole `submit` call** (`AGENTS.md`
    §2.1) — every task and sub-task implemented in steps 2–3 above is included in this one
    submit, not previously saved individually. Fires after docs, README updates, the Task
    Group Summary, and the Mandatory Pre-Submit Local Verification (step 5) are all complete.
    This is the single point at which the whole Task Group's work becomes safe from a session
    crash, aside from any Design-specified WIP Checkpoint reached along the way — treat the
    Task Group's work as unsaved, not merely "paused for bookkeeping," until this submit
    actually fires.
8.  **Stop. Do Not Proceed to the Next Session Unit.** Per `AGENTS.md` §2.8, the session ends here regardless of remaining capacity, whether the declared Session Unit was a full Task Group, a single Task, or one Code/Verify sub-task. Notify the user the unit is complete and await a new session to begin the next one. **Do not check the next Session Unit's Entry Criteria either** — a session verifies only its own Exit Criteria; the next Session Unit is always opened fresh, in a different session, which checks its own Entry Criteria itself at that time (same rule as `agents/MAINTENANCE.md` §10's Task-Group-boundary scope rule — reaching forward, even to glance, is out of this session's scope).

#### 5.2.1. Project README

**The project's root `README.md` is now drafted during Design Phase, at Step 8**, alongside
the Development Plan/Checklist/Dev Prompt (`agents/DESIGN.md` §5.8) — not authored from
scratch by the Development agent. The first Development Phase session's task
(Task Group 0, per the Checklist template) is **review, confirmation, and enhancement** of that
already-drafted README against the actual repository as it starts to take shape — not
initial scaffolding. It is revisited for a final accuracy/completeness review during the
last Task Group, once the built system may have diverged in minor ways from the Design-time
draft. Any Task Group that materially changes how the project is built, run, or used should
update it as part of that Task Group's documentation-integrity check (§5.2 step 5).

#### 5.2.2. CI Workflow

**`.github/workflows/ci.yml` is drafted during Design Phase, at Step 8**, from
`agents/CI.md`'s stage skeleton, using Step 5's CI Stage Applicability findings
(`agents/DESIGN.md` §5.5/§5.8) — not authored from scratch here. Task Group 0's task is
**review, confirmation, and extension** of that draft against the repository as it starts
to take shape, mirroring the README pattern above — most commonly confirming that the
conditional stages Step 5 identified (WASM, Playwright/E2E, ESP32, infra services) are
correctly present or correctly absent once the actual codebase makes that visible, and that
Stage 0's `scripts/setup_env.sh` step matches whatever `setup_env.sh` content Task Group 0
itself produces or extends.

#### 5.2.3. Third-Party License Disclosure

**`deny.toml` (with its `[licenses]` allow-list, `agents/PREFERRED_TOOLS.md`) is created
during Task Group 0**, not before — it has no content prerequisite of its own, but naturally
belongs alongside the workspace's `Cargo.toml`/`Cargo.lock`, which are themselves products
of Task Group 0's scaffolding, not something Design Phase produces. Create it as part of
scaffolding, before running the check below.

**`THIRD_PARTY_LICENSES.md` is drafted during Design Phase, at Step 8**, from the
dependency set finalized in the Architecture Specification (`agents/DESIGN.md` §5.8) — but
unlike README/`ci.yml`, that draft is necessarily provisional: no `Cargo.lock` exists until
the workspace is actually scaffolded, so Design Step 5's own license check
(`agents/PREFERRED_DEPENDENCIES.md`'s License Compatibility Criterion) is limited to
**direct** dependencies only. **Format, both at this draft stage and at every later
regeneration: a one-line description followed by a dependency/license table (crate name,
version, license identifier) — no header/footer prose, no full license text bodies**
(`agents/CI.md` Stage 5). Task Group 0's reconciliation below is a table expansion to the full
resolved tree, not a restructuring.

**Task Group 0 runs the first real `cargo deny check licenses`** against the actual resolved
`Cargo.lock` — the earliest point a transitive-dependency license violation (a dependency's
own dependency carrying an incompatible license, invisible at Design time) is genuinely
catchable — and reconciles `THIRD_PARTY_LICENSES.md` against that result. Any violation
found here is an Escalation Trigger (`agents/exemplars/development_plan_template.md` §13),
not a silent fix: the agent cannot itself decide to swap a dependency or add a `[patch]`
exception, per `PREFERRED_DEPENDENCIES.md`'s No Local Patching mandate. From Task Group 0
onward, `agents/CI.md` Stage 5 re-runs this check on every push and fails (does not
auto-commit) on any drift between the committed `THIRD_PARTY_LICENSES.md` and what the
current dependency tree would actually produce.

**`LICENSE.md`** (drafted at Step 8, static text) needs no reconciliation the way
`THIRD_PARTY_LICENSES.md` does — Task Group 0's review is a simple accuracy check (correct
license text, correct copyright holder/year), same tier as `.gitignore`'s review.

#### 5.2.4. Productization Readiness Checklist

**Every item below is a mandatory Exit Criterion of the Final Task Group** (Development Plan
§15), checked once, at the end of the project, in addition to that Task Group's own task-level
DoD items — not a substitute for them. This checklist is the single canonical definition
referenced by both the Final Task Group's Exit Criteria and, for Maintenance Phase work on an
existing project, `agents/MAINTENANCE.md`'s Readiness Audit — defined once, here, not
restated in either place. Applicability of items 7–9 (marked *conditional*, below) is
determined at Design Step 5 (`agents/DESIGN.md` §5.5's Productization Applicability
finding) and drives which of `PROD-007`–`PROD-009` (`agents/exemplars/
development_plan_template.md` §8) are drafted into the Final Task Group at Step 8.

1. **Regression Scaffold:** every Core-status Requirement ID (Spec §3) traces to ≥1
   independently re-runnable test — re-runnable by name/tag, not merely "covered somewhere
   in the Task Group N verification run." The Requirement-to-Test traceability table exists as
   a committed artifact, `test/[projectname]_requirement_traceability.md`, not something
   reconstructed ad hoc from memory or grep. This table is drafted as a skeleton (one row
   per Core Requirement ID, empty test column) at Design Step 8, then updated
   incrementally every Development Task Group as that Task Group's own tasks add tests — it is a
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
   contains no item that should already have been reached and resolved by this Task Group.
4. **User-Facing Docs:** `README.md` reflects the as-built system (this item is satisfied
   by, not separate from, the existing `DOC-FINAL` task, `agents/exemplars/
   development_plan_template.md` §8).
5. **Developer-Facing Docs:** the Architecture Specification has no known undocumented
   divergence from the as-built system — any found divergence is either fixed or logged as
   an Appendix F Spec Amendment (`agents/exemplars/architecture_specification_template.md`
   Appendix F); `dev/plan/[projectname]_dev_risks.md` is current (every Open risk
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
10. **Documentation Coverage:** every crate's `#![deny(missing_docs)]` passes clean
    (`AGENTS.md` §2.3, `agents/CI.md` Stage 1b) — re-checked against the actual as-built
    repository, not merely trusted from each task's own self-reported rustdoc DoD item,
    the same "final backstop, not primary enforcement" relationship item 3 above already
    has to the per-task Anti-Stub mandate. `metrics/docs_coverage.toml` shows 100% across
    the workspace, with no crate left on `#![warn(missing_docs)]` at project completion.

A gap found in any of these ten items (or the applicable subset) at Final Task Group is treated
exactly like any other unmet Exit Criterion — it blocks Task Group completion, it is not
deferred past this project's own Final Verification.

#### 5.2.5. Recurring Integration-Reality Check

**Every 5th Task Group (Task Groups 5, 10, 15, ...) by index, in addition to that Task
Group's own Exit Criteria, answers one standing question directly in its Task Group
Summary:** has the assembled system, or a meaningful and growing subset of it, actually
been run together as a whole at any point so far — and if not, is that a deliberately
deferred, explicitly named decision (citing which later Task Group/Build Order Step is
expected to close the gap), rather than a silent one. This is not a new mechanical gate —
Fix 1's Test-Type-to-Naming Binding and Fix 2's Walking Skeleton/Maturity-Triggered
Component Integration milestones are what actually make this question answerable without a
forensic pass: the check here just asks it out loud, on a fixed cadence, rather than
leaving it implicit until a downstream question forces the issue by accident (the way the
original gap this section exists to close was actually discovered).

A "yes, deliberately deferred" answer with a named target Task Group is not itself a
finding. A "no, and nothing names when this will be addressed" answer, or a Task Group
Summary that skips the question entirely, is treated the same as any other Escalation
Trigger (§13's model in `agents/exemplars/development_plan_template.md`) — it stops the
session's Final Wrap-Up Submit until the question is actually answered, not just deferred
by omission.

### 5.3. Completion of Development
After all Task Groups in the `Development Plan` are complete, the agent must notify the user and await instruction on next steps, which may include a post-development remediation cycle.

### 5.4. Post-Development Remediation Cycle
This cycle begins only when the user explicitly requests it by saying "perform an audit".
1.  **Audit and Create Remediation Plan:** The agent will perform a feature audit and create the necessary planning documents.
    -   **COMMIT POINT:** After creating all documents, the agent MUST commit them together and await further user instruction.
2.  **Execute Remediation Checklist:** The agent will execute the remediation checklist using the Task-Group-by-Task-Group workflow in §5.2 (including the One Task Group Per Session mandate).
    -   **COMMIT POINT:** After each Task Group of the remediation checklist is complete and verified, the agent MUST commit the changes.
    -   Upon completion, the agent MUST notify the user and await further instructions.

## 6. Phase Completion Criteria
This phase is complete when all tasks in the `Development Plan` have been implemented, built, and committed.

---

## Appendix
See `CHANGELOG.md` for this file's full version history.
