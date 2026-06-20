# Development Plan: [Project Name]
v.0.2.00 — Complex/Multi-Phase Rust Project Variant

> **When to use this variant instead of `development_plan_template.md`:** use this template when
> the source Architecture Specification is large enough (multiple documents/parts, 100+ atomic
> requirements, an explicit multi-stage Build Order, and/or more than one Rust component — e.g.
> a backend service plus a Rust/WASM frontend or a Tauri client) that a single flat Phases list
> and a single flat Task list would not let an executing agent or a reviewer answer, at a glance:
> "what tools/config does a session need before it can do anything," "what must already be true
> before I start this phase," "how do I prove this task is actually done," "which phases block
> which," and "what do I do when something goes wrong." If the project is small enough that
> `development_plan_template.md` answers those questions without extra scaffolding, use that
> template instead — this variant is strictly more overhead and should not be used by default.
>
> **Scope:** this variant is Rust-only, across however many Rust components the project has
> (native services, Rust/WASM web frontends, Tauri/UniFFI mobile or desktop shells, CLI tools,
> shared crates). It does not attempt to generalize to non-Rust toolchains. Everything in the
> base template is preserved (same section order, same intent per section). Sections marked
> **[EXTENDED]** add structure the base template leaves implicit. Sections marked **[NEW]** have
> no equivalent in the base template.

## Table of Contents
- [0. Architecture Cross-Reference](#0-architecture-cross-reference)
- [1. Introduction](#1-introduction)
- [2. Technology Stack](#2-technology-stack)
- [3. Project Folder Structure](#3-project-folder-structure)
- [4. Environment & Prerequisites Setup](#4-environment--prerequisites-setup)
- [5. Development & Test Configuration](#5-development--test-configuration)
- [6. Phases and Milestones](#6-phases-and-milestones)
- [7. Risk Management & Mitigation](#7-risk-management--mitigation)
- [8. Task Decomposition](#8-task-decomposition)
- [9. Test Strategy & Plan](#9-test-strategy--plan)
- [10. Logging Strategy](#10-logging-strategy)
- [11. Session Handoff Protocol](#11-session-handoff-protocol)
- [12. Abort / Rollback Protocol](#12-abort--rollback-protocol)
- [13. Escalation Triggers](#13-escalation-triggers)
- [14. Change Control for This Plan](#14-change-control-for-this-plan)
- [15. Plan-Level Definition of Done](#15-plan-level-definition-of-done)
- [Appendix G - Glossary](#appendix-g---glossary)
- [Appendix R - Revision History](#appendix-r---revision-history)

---

## 0. Architecture Cross-Reference **[NEW]**

> Not present in the base template, which assumes one Architecture Specification document close
> at hand. For multi-document or multi-part specs, list every source document and the
> traceability artifact that links requirements back to them, so a reader (human or agent) never
> has to guess which document is authoritative for a given decision.

| Document | Role |
|---|---|
| [Architecture Specification, Part(s) ...] | Authoritative source of all technical decisions |

> **Project note:** this project does not produce a separate Requirements & Traceability
> document. All requirement IDs cited by tasks in §8 below are defined in **Architecture
> Specification §3 (Acceptance Criteria & Traceability)** — that section is the sole,
> authoritative traceability source for this project's Plan and Checklist.

**Precedence rule:** if this Plan and the Architecture Specification ever appear to conflict, the
Architecture Specification is authoritative and this Plan must be corrected, not the reverse
(see §14, Change Control).

## 1. Introduction
- **Purpose:**
- **Scope:**
- **References:** (Link to Architecture Specification)

## 2. Technology Stack **[EXTENDED]**

> Extends the base template's flat list. Even within an all-Rust project, different components
> can target different compilation outputs (native binary, `wasm32-unknown-unknown`, a mobile
> shell via Tauri/UniFFI) with different crate ecosystems, different test runners, and different
> verification methods (§8). State the stack per component, not once for the whole project. If
> the project genuinely has only one component, a single row is fine — the table still makes
> that an explicit statement rather than an assumption.

| Component | Crate type / target | Key crates / frameworks | Test runner | Build command |
|---|---|---|---|---|
| [e.g., `services/mmosecure`] | bin, native (`x86_64-unknown-linux-gnu`) | [Axum, SQLx, ...] | `cargo test` / `cargo nextest` | `cargo build -p mmosecure` |
| [e.g., `apps/web`] | bin/lib, `wasm32-unknown-unknown` | [Leptos/Dioxus/Yew, ...] | `wasm-pack test` / headless-browser harness | `trunk build` / `dioxus build` |
| [e.g., `apps/desktop`] | bin, Tauri shell | [tauri, ...] | `cargo test` (Rust side) + harness for the webview | `cargo tauri build` |

## 3. Project Folder Structure **[EXTENDED]**

> Extends the base template's single tree-view to explicitly support a Cargo workspace with
> multiple member crates/components, since that is the normal shape once a project has more
> than one Rust component. Show the workspace root and enough of each component's internal
> structure that an agent can locate where new code belongs without searching.

(Provide a tree-view of the proposed Cargo workspace structure — workspace `Cargo.toml`,
`libs/`, `crates/`, `services/`, and any frontend/mobile component directories such as
`apps/web` or `apps/desktop`, each as their own workspace member.)

## 4. Environment & Prerequisites Setup **[NEW]**

> Neither base-template section addresses what a session needs *before* any task can be
> attempted. Without this, an agent can lose an entire session discovering mid-task that a
> required tool is missing. List every prerequisite explicitly enough that a fresh session can
> verify (not assume) it has what it needs, ideally via a single setup-check command/script.

| Prerequisite | Required for | Verification command | Notes |
|---|---|---|---|
| Rust toolchain (version pinned via `rust-toolchain.toml`) | All components | `rustc --version` matches pinned version | |
| `cargo-nextest` (if used) | Faster parallel test execution | `cargo nextest --version` | |
| `cargo-deny` | Dependency-graph / license CI gates | `cargo deny --version` | |
| `cargo-audit` | Vulnerability scanning CI gate | `cargo audit --version` | |
| `sqlx-cli` (if a SQL store is used) | Running/checking migrations | `sqlx --version` | |
| `protoc` (if Protobuf/Kafka messages are used) | Generating types from `.proto` | `protoc --version` | |
| Vendored/installed policy-engine binary (e.g. `opa`, if used) | Policy syntax validation / policy tests | `opa version` | |
| Container runtime (e.g. Docker) | `testcontainers`-based integration tests | `docker info` succeeds | |
| WASM target (if a Rust/WASM component exists) | Building the web component | `rustup target list --installed \| grep wasm32` | |
| WASM build tool (`trunk`, `wasm-pack`, or framework CLI) | Building/serving the web component | `trunk --version` (or equivalent) | |
| Headless browser harness (e.g. Playwright, if visual verification is used) | Frontend behavioral DoD checks (§8) | harness-specific version check | |
| Tauri CLI (if a desktop/mobile shell exists) | Building the shell | `cargo tauri --version` | |

**A session MUST run a setup-check (ideally a single script, e.g. `scripts/check_env.sh`, per
`AGENTS.md`'s Session Initialization mandate) before starting any task, and MUST halt and report
rather than proceed if any required prerequisite is missing or fails verification.**

## 5. Development & Test Configuration **[NEW]**

> Neither base-template section addresses how a session actually gets a runnable local
> environment — distinct from production secret/config handling, which the Architecture
> Specification owns. Without this, even a fully-built Phase 1 has nothing to run against.

- **Local configuration file(s):** [e.g., `config/development.toml` — path, and which values
  differ from `config/default.toml`]
- **Mock/test credential and secret seeding:** [e.g., which provider implementation is used
  in dev/test (a mock/in-memory provider per the Architecture Spec), what deterministic test
  values it returns, and where those are defined]
- **Test key material:** [e.g., how a session generates or obtains test signing keys (JWT,
  TLS, etc.) — MUST be clearly marked as non-production and MUST NOT be reused as real secrets]
- **Local infrastructure bring-up:** [e.g., a `docker-compose.yml` or equivalent for databases/
  message brokers/caches the components depend on; the command to start it; the command to
  verify it is healthy before tasks begin]
- **Environment variable overrides:** [naming convention, e.g. `<SERVICE>__<SECTION>__<KEY>`,
  and where the canonical list of overridable values is documented]

**Rule:** no task's Definition of Done may depend on a production credential, a production
provider, or any non-local infrastructure. If a task appears to require this, treat it as an
Escalation Trigger (§13), not something to work around silently.

## 6. Phases and Milestones **[EXTENDED]**
(Break down the development work into logical phases with clear milestones. **MANDATE:** Each
phase must be sized to comfortably fit within a single agent session, including a buffer for
complications.)

### 6.1. Phase Index

> Extends the base template's free-form breakdown with a fixed schema per phase, so phase
> boundaries are objectively checkable rather than descriptive. Every column is mandatory.

| Phase | Maps to Build-Order Step(s) | Component(s) | Title | Entry Criteria | Exit Criteria (DoD gate) | Primary Requirement Domains | Task Count |
|---|---|---|---|---|---|---|---|
| [N] | [ref] | [which component(s) this phase touches] | [title] | [what must already be true/exist before this phase can start] | [the single objective condition that proves this phase is complete] | [REQ-XXX domains] | [count] |

### 6.2. Phase Dependency Graph

> A flat phase list does not communicate which phases can run in parallel (if the executing
> environment ever supports that) versus which are strictly sequential, which phases belong to
> which component, nor which single phase is the highest-leverage blocker. Express this
> explicitly, even in simple text-arrow form, so a reader doesn't have to infer it from prose.

```
[Phase 0] → [Phase 1] → [Phase 2] → ...
                                  ↘
                                    [Phase N] (blocks all of: ...)
```

State explicitly: which phase(s), if delayed, delay the largest number of downstream phases
("critical path" phases) — these deserve the most risk-mitigation attention in §7. If the
project has multiple components, also state explicitly where a phase in one component depends
on a phase in another (e.g., a frontend phase that consumes an API contract a backend phase
must finish first).

## 7. Risk Management & Mitigation **[EXTENDED]**
(Identify high-risk units, such as complex algorithms or 3rd-party integrations, and plan their
development/mitigation.)

> Extends the base template by requiring the table to be populated from concrete, spec-derived
> risks identified during planning — not left as a single illustrative example row. At minimum,
> include: every external/infra dependency the plan assumes is available (databases, message
> brokers, container runtimes, vendored binaries, browser/headless-test harnesses), every
> algorithm whose correctness is security-critical or hard to unit-test, every place the source
> spec itself flagged a tradeoff or an open question, and every cross-component contract a
> later phase depends on being stable.

| Risk | Impact | Likelihood | Mitigation Strategy | Affected Phase(s) |
| :--- | :--- | :--- | :--- | :--- |
| [e.g., Latency in external API] | High | [Low/Med/High] | [e.g., Implement local cache and mock for testing] | [Phase N] |

## 8. Task Decomposition **[EXTENDED]**
(For each phase, provide a detailed list of tasks.)

> Extends the base template's per-task schema with fields aimed specifically at a
> lower-capability, single-session executing agent, and generalizes the Definition of Done so it
> is not implicitly backend-biased — a Rust/WASM frontend task is not "done" by `cargo build`
> alone the way a backend task often is.

**Task Template:**
- **Task ID:** `[DOMAIN]-[NNN]`
- **Phase:** [phase number]
- **Component:** [which workspace member this task belongs to]
- **Description:**
- **Definition of Ready (DoR):** (what must be true to *start* this specific task — e.g., a
  dependency Task ID is complete, a migration has been applied to a live local DB, an upstream
  API contract this task consumes is already implemented and tested. Distinct from the phase's
  Entry Criteria in §6.1, which gates the whole phase; this gates the individual task.)
- **File(s) touched:** (explicit paths — new files to create, existing files to edit)
- **Dependencies:** (other Task IDs that must be complete first; package/system-level dependency changes)
- **Estimated Effort:** (S / M / L, relative to other tasks in the same phase — not wall-clock time)
- **Verification Method:** one of:
    - **Build+Test** (the default for non-UI Rust code): exact `cargo` commands the agent must
      run.
    - **Visual/Behavioral** (for Rust/WASM frontend or Tauri-shell UI work): exact steps to
      render the component (e.g., `trunk serve`) and the exact check to perform against it
      (e.g., a headless-browser script asserting an element/state, a screenshot comparison, a
      check for zero browser-console errors, an accessibility-tree assertion).
    - **Hybrid:** both, when a task has both a logic surface and a UI surface.
- **Commands:** (the exact commands the agent must run to evaluate this task's DoD, matching the
  Verification Method above)
- **Definition of Done (DoD):**
    - [ ] Code implemented and hermetically builds. (`[exact command]`)
    - [ ] Verification Method's checks pass. (`[exact command(s)]`)
    - [ ] **Test Case:** [ID] (e.g., Verify that input A results in output B / Verify that UI
      state X renders element Y)
    - [ ] **Required Artifacts:** (e.g., test output log, screenshot, accessibility report)
- **Traceability:** REQ-XXX-NNN[, REQ-XXX-NNN...] (every task must cite at least one requirement ID; a task with no traceable requirement should not exist)
- **Context for executing agent:** (one or two sentences resolving the most likely ambiguity or judgment call in this task — not a restatement of the description, only what an agent with less context than the planner might get wrong)

## 9. Test Strategy & Plan
- **Unit Test Plan:**
- **Integration Test Plan:**
- **System & Acceptance Test Plan:** (for any Rust/WASM frontend or Tauri component, state the
  browser/device-target matrix and the headless-test harness used, in addition to backend system
  tests)

## 10. Logging Strategy
(Incorporate the logging strategy defined in the Architecture Specification, including tasks for
instrumentation.)

## 11. Session Handoff Protocol **[NEW]**

> Addresses a gap specific to multi-phase plans executed across many independent, stateless
> agent sessions (e.g., one Google Jules session per phase) with no memory of prior sessions.
> Without this section, an agent starting a new session has no prescribed way to (a) confirm
> which phases are actually complete versus merely checked off, (b) recover from a session that
> was interrupted mid-phase, or (c) avoid re-doing or conflicting with already-merged work.

**At the start of every session, before starting any task, the agent MUST:**
1. Complete the §4 Environment setup-check.
2. Read the Development Checklist in full and identify the first phase with any unchecked task.
3. Run the project's build and test commands, for every component touched by that phase, as they
   currently stand (before making any change) and confirm the result matches what the checklist
   claims — if the checklist says a phase is done but the build/tests fail, STOP and report the
   discrepancy rather than proceeding or silently fixing it (see §13, Escalation Triggers).
4. Confirm the phase's **Entry Criteria** (§6.1) are actually satisfied by inspecting the
   relevant files/state directly, not by trusting the checklist alone.
5. Work only within the identified phase's tasks. Do not begin a later phase's tasks even if
   idle capacity remains in the session, unless explicitly instructed.

**At the end of every session, the agent MUST:**
1. Update the Development Checklist to reflect exactly what was completed (no partial credit —
   a task is checked only when its full DoD, including required artifacts, is satisfied).
2. Leave the repository in a hermetically buildable state for every component, even if the
   phase is not fully complete — never commit a known-broken intermediate state as the
   session's final output.
3. Record, in a brief session note (location: [e.g., a `SESSION_LOG.md` or PR description]), any
   deviation from the plan, any requirement that proved infeasible as written, or any new risk
   discovered — for the next session and for the human reviewer, per DESIGN.md's Major Change
   Notification mandate.

## 12. Abort / Rollback Protocol **[NEW]**

> Session Handoff (§11) covers *resuming* a session; this covers what an agent should do if a
> phase or task turns out to be unsalvageable *within* a session — a distinct situation that
> neither base template, nor §11 alone, addresses.

**An agent MUST treat a task or phase as aborted (not silently reworked into something else) when any of the following occur:**
- The chosen implementation approach is discovered, partway through, to violate a constraint
  stated in the Architecture Specification (e.g., introduces an I/O dependency into a crate
  required to have none).
- A task's Definition of Ready turns out to be false (a dependency the task assumed was complete
  is not actually complete or correct).
- Completing the task as specified would require modifying a file or contract outside this
  task's stated File(s)/Component scope in a way not anticipated by the plan.

**On abort, the agent MUST:**
1. Revert uncommitted changes for the aborted task back to the last known-good state (the state
   at the start of the session, or the last completed task's end-state, whichever is more
   recent) — never leave a half-applied change in place.
2. NOT mark the task complete in the Checklist.
3. Record the abort and its cause in the session note (§11) in enough detail that the next
   session (or the human reviewer) can decide whether the task needs re-specification rather
   than re-attempting the same approach blind.
4. Continue with other, unaffected tasks in the same phase if any remain and are still valid;
   otherwise end the session per §11's end-of-session steps.

**Rollback MUST NOT cross a phase boundary** (i.e., aborting a Phase N task must never revert
already-completed, already-checked-off work from Phase N-1 or earlier) **except** when the
abort's root cause is a defect discovered in that earlier work — in which case this is an
Escalation Trigger (§13), not a unilateral rollback, since it may invalidate downstream
completed work whose own validity depended on it.

## 13. Escalation Triggers **[NEW]**

> Neither base template states, concretely, when an executing agent should stop and ask a human
> rather than make its own judgment call and proceed. DESIGN.md's general mandates (Proactive
> Elicitation, Anti-Stub) imply this but do not enumerate it at the plan/task level. List the
> concrete conditions here so a lower-capability agent has a checklist, not a principle to infer.

**The agent MUST stop and request human input (not guess, not silently choose an interpretation) when:**
- A task's Architecture Specification source (cited in its Traceability field) is ambiguous,
  silent on the question actually blocking implementation, or appears to conflict with another
  requirement.
- A Definition of Ready (§8) or phase Entry Criteria (§6.1) cannot be verified true or false
  from the repository's current state.
- An Abort (§12) occurs and the cause appears to be a defect in already-completed,
  already-checked-off work from a prior phase.
- A required prerequisite (§4) is unavailable in the session environment and has no documented
  fallback.
- Completing a task's DoD would require a production credential, production infrastructure, or
  any resource excluded by §5's rule.
- The agent's confidence that a generated artifact (code, config, policy) is correct is
  meaningfully lower than for comparable completed tasks — this is a judgment call the agent is
  expected to make conservatively, not optimistically.
- Any condition the Architecture Specification or DESIGN.md elsewhere explicitly names as
  requiring user approval (e.g., a "Major Change" to the architecture).

**When escalating, the agent MUST:** state the specific task/phase, the specific blocking
question, and (if applicable) the options it has identified — matching the project's general
"Super-Strict Workflow" mandate of presenting an interpretation and plan before acting, applied
here to the narrower case of being blocked rather than merely about to start.

## 14. Change Control for This Plan **[NEW]**

> Distinct from DESIGN.md's "Major Change Notification" mandate, which governs changes to the
> *Architecture Specification*. This section governs changes to the *Plan and Checklist
> documents themselves* once execution has begun — neither base template addresses this at all,
> and without it, a plan silently drifts from what was actually built.

- A phase's task list MAY be amended after work has started only when a later phase or task
  discovers the original decomposition was incorrect or incomplete (e.g., a missing task, a
  task whose scope was wrongly estimated, a dependency that was missed).
- Any such amendment MUST be recorded in this Plan's Revision History (Appendix R) with: what
  changed, which phase/task, and why — not silently edited with no trace.
- Amendments MUST NOT retroactively mark already-completed tasks differently than they were
  actually completed (i.e., do not rewrite history to make a past task's DoD match what was
  actually delivered if it diverged — record the divergence instead, per §11's session-note
  requirement, and raise it per §13 if it is significant).
- Renumbering existing Task IDs or Phase numbers is NOT permitted (per the ID-stability rule
  defined in Architecture Specification §3 and referenced via this Plan's §0 project note);
  new tasks are appended with new IDs, and deprecated tasks are marked `[DEPRECATED]` rather
  than deleted.
- Changes that affect more than one phase, or that change a phase's Exit Criteria, count as
  "Major" for the purposes of DESIGN.md's notification mandate and MUST be surfaced to the human
  reviewer even if the underlying Architecture Specification did not itself change.

## 15. Plan-Level Definition of Done **[NEW]**

> The Architecture Specification's own GA/Definition-of-Done criteria (if any) govern the
> *product*. This section is narrower and more mechanical: it defines when the Plan and
> Checklist documents themselves can be considered complete and internally consistent, which
> neither base template states.

The Plan is complete only when **all** of the following hold:
- [ ] Every Core-status requirement ID defined in Architecture Specification §3 (per the §0
  project note) appears in the Traceability field of at least one task in §8.
- [ ] No task in §8 cites a requirement ID that does not exist in Architecture Specification
  §3 (no orphan citations).
- [ ] Every phase in §6.1 has a non-empty Entry Criteria and Exit Criteria.
- [ ] Every task in §8 has a non-empty Verification Method and at least one DoD checkbox.
- [ ] The Phase Dependency Graph (§6.2) contains no cycles and every phase is reachable from
  Phase 0 (or the project's first phase).
- [ ] The Development Checklist (companion document) contains exactly one checklist line per
  task DoD item in §8 — no drift between the two documents.
- [ ] This Plan's own Architecture Cross-Reference (§0) lists every source document actually
  cited anywhere in §8's Traceability fields.

## Appendix G - Glossary **[NEW]**

> Neither base template includes a glossary. For a spec with dense, project-specific
> vocabulary, list every term and acronym a session needs to hold consistently across a long,
> single-pass session, so meaning doesn't drift task-to-task. Populate from the source
> Architecture Specification's own glossary/terminology sections where one exists, plus any
> abbreviations introduced in this Plan itself (e.g., DoD, DoR).

| Term / Acronym | Meaning |
|---|---|
| DoD | Definition of Done (per task or per phase, §8/§6.1) |
| DoR | Definition of Ready (per task, §8) |
| [project-specific terms...] | [...] |

---

## Appendix R - Revision History
| Version | Date | Author | Changes |
|---|---|---|---|
| 0.2.01  | 2026-06-20 | Claude | Removed the "Requirements & Traceability Backfill" row from §0's table (projects that don't produce a separate traceability document point directly at the Architecture Specification's own traceability section instead); added a project note to §0 and tightened the §15 Definition-of-Done and corresponding Checklist Final Verification checks to reference that resolution directly rather than an indirect cross-reference-table lookup. |
| 0.2.00  | YYYY-MM-DD | [author] | Rust-only complex/multi-phase variant: generalized to multi-component (native/WASM/Tauri) projects; added Environment & Prerequisites Setup, Dev/Test Configuration, Session Handoff, Abort/Rollback, Escalation Triggers, Change Control, Plan-Level DoD, and Glossary; generalized task Verification Method beyond build+test. |
| 0.1.00  | YYYY-MM-DD | [author] | Initial creation of complex/multi-phase variant, extending the base `development_plan_template.md`. |
