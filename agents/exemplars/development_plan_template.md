# Development Plan: [Project Name]
Complex/Multi-Phase Rust Project Variant

> **When to use this variant instead of `development_plan_template.md`:** use this template when
> the source Architecture Specification is large enough (multiple documents/parts, 100+ atomic
> requirements, an explicit multi-stage Build Order, and/or more than one Rust component — e.g.
> a backend service plus a Rust/WASM frontend or a Tauri client) that a single flat Phases list
> and a single flat Task list would not let an executing agent or a reviewer answer, at a glance:
> "what tools/config does a session need before it can do anything," "what must already be true
> before I start this phase," "how do I prove this task is actually done," "which phases block
> which," and "what do I do when something goes wrong." If the project is small enough that a
> simpler flat plan answers those questions without extra scaffolding, use that instead — this
> variant is strictly more overhead and should not be used by default.
>
> **Scope:** this variant is Rust-only, across however many Rust components the project has
> (native services, Rust/WASM web frontends, Tauri/UniFFI mobile or desktop shells, CLI tools,
> shared crates). It does not attempt to generalize to non-Rust toolchains. Sections marked
> **[EXTENDED]** add structure a simpler flat plan leaves implicit. Sections marked **[NEW]**
> have no equivalent in a simpler flat plan.

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

---

## 0. Architecture Cross-Reference **[NEW]**

> For multi-document or multi-part specs, list every source document and the
> traceability artifact that links requirements back to them, so a reader (human or agent) never
> has to guess which document is authoritative for a given decision.

| Document | Role |
|---|---|
| [Architecture Specification, Part(s) ...] | Authoritative source of all technical decisions |

> **Project note:** this project does not produce a separate Requirements & Traceability
> document. All requirement IDs cited by tasks in §8 below are defined in **Architecture
> Specification §3 (Acceptance Criteria & Traceability)** — physically split, per `CLAUDE.md`
> §4.1, across `_04_test_strategy` (§3.1–3.3: criteria, Test Case Catalog, Requirement
> Smells) and `_05_verified_traceability` (§3.4–3.5: Traceability Matrix, Acceptance Criteria
> Detail). Together these two files are the sole, authoritative traceability source for this
> project's Plan and Checklist.

**Precedence rule:** if this Plan and the Architecture Specification ever appear to conflict, the
Architecture Specification is authoritative and this Plan must be corrected, not the reverse
(see §14, Change Control).

**Audit dependency:** this Plan and its companion Checklist are not considered final until
they clear Design Step 9 (Plan & Checklist Audit, `CLAUDE.md`/`agents/DESIGN.md` §5.9) — an
independent, adversarial check run after this document's own gate approval, distinct from
and in addition to the propose-with-flagged-assumptions drafting that produced it.

## 1. Introduction
- **Purpose:**
- **Scope:**
- **References:** (Link to Architecture Specification)

## 2. Technology Stack **[EXTENDED]**

> Even within an all-Rust project, different components
> can target different compilation outputs (native binary, `wasm32-unknown-unknown`, a mobile
> shell via Tauri/UniFFI) with different crate ecosystems, different test runners, and different
> verification methods (§8). State the stack per component, not once for the whole project. If
> the project genuinely has only one component, a single row is fine — the table still makes
> that an explicit statement rather than an assumption.
>
> **Workspace edition and MSRV (per `agents/RUST_PREFERENCES.md` §0):** state the Rust 2024
> edition declaration and the workspace MSRV set at Design Step 5 here, derived from
> `agents/PREFERRED_TOOLS.md`'s tool-version floors and every dependency's own MSRV. If the
> project has a publishable library crate, state its separate, lower consumer MSRV per the
> dual-MSRV pattern (cross-reference Architecture Specification §10.2).

| Component | Crate type / target | Key crates / frameworks | Test runner | Build command |
|---|---|---|---|---|
| [e.g., `services/mmosecure`] | bin, native (`x86_64-unknown-linux-gnu`) | [Axum, SQLx, ...] | `cargo nextest run` | `cargo build -p mmosecure` |
| [e.g., `apps/web`] | bin/lib, `wasm32-unknown-unknown` | [Leptos/Dioxus/Yew, ...] | `wasm-pack test` / headless-browser harness | `trunk build` |
| [e.g., `apps/desktop`] | bin, Tauri shell | [tauri, ...] | `cargo nextest run` (Rust side) + harness for the webview | `cargo tauri build` |

## 3. Project Folder Structure **[EXTENDED]**

> Show the workspace root and enough of each component's internal
> structure that an agent can locate where new code belongs without searching. Includes the
> `deploy/` directory for infrastructure service definitions (`agents/PREFERRED_SERVICES.md`),
> `scripts/` for environment setup (§4 below), and `assets/` for user-supplied UI/media
> reference assets (below).

(Provide a tree-view of the proposed Cargo workspace structure — workspace `Cargo.toml`,
`libs/`, `crates/`, `services/`, any frontend/mobile component directories such as
`apps/web` or `apps/desktop`, `deploy/` for docker-compose service definitions, `assets/` per
the convention below, and `scripts/` for `setup_env.sh`/`setup_env.bat`, each as their own
workspace member or top-level directory as appropriate.)

**`assets/` directory — user-populated, filename-stable across Design and Development.**
Where the Architecture Specification's Asset Manifest (§4.13, per `CLAUDE.md` §3.8) lists
user-supplied HTML, image, audio, or video reference assets, the user populates the
identical files, under the identical filenames used throughout the Architecture
Specification and this Plan, into:
- `assets/html/` — reference HTML (per `CLAUDE.md` §3.8, presumptively authoritative for the
  structure/behavior of the screen/component it represents)
- `assets/images/` — branding, iconography, typography reference, color palette, pattern/
  texture tiles, and other graphical reference material
- `assets/audio/` — audio assets, if the project specifies any
- `assets/video/` — video assets, if the project specifies any

A task consuming one of these assets cites it by filename, resolving directly against
`assets/<type>/<filename>` and the Architecture Specification's Asset Manifest entry for it
— never re-describing the asset's content in the task itself, which would risk drifting from
the actual file. The development agent MUST NOT rename, move, or duplicate these files; if
an asset appears to be missing from the repository at the path the Asset Manifest specifies,
this is an Escalation Trigger (§13) — stop and ask, do not proceed against an assumed or
recreated substitute.

## 4. Environment & Prerequisites Setup **[NEW]**

> Without this, an agent can lose an entire session discovering mid-task that a
> required tool is missing. List every prerequisite explicitly enough that a fresh session can
> verify (not assume) it has what it needs, ideally via a single setup-check command/script.

| Prerequisite | Required for | Verification command | Notes |
|---|---|---|---|
| Rust toolchain (version pinned via `rust-toolchain.toml`, per §2's workspace MSRV) | All components | `rustc --version` matches pinned version | |
| `cargo-nextest` (preferred test runner, per `agents/PREFERRED_TOOLS.md`) | Test execution | `cargo nextest --version` | |
| `cargo-llvm-cov` (preferred coverage tool) | Coverage reporting | `cargo llvm-cov --version` | |
| `cargo-deny` | Dependency-graph / license CI gates | `cargo deny --version` | |
| `cargo-audit` | Vulnerability scanning CI gate | `cargo audit --version` | |
| `sqlx-cli` (if a SQL store is used) | Running/checking migrations | `sqlx --version` | |
| `protoc` (if Protobuf/Kafka messages are used) | Generating types from `.proto` | `protoc --version` | |
| Docker / `docker-compose` (if any `agents/PREFERRED_SERVICES.md` service is used) | Infrastructure service bring-up | `docker compose version` | |
| WASM target (if a Rust/WASM component exists) | Building the web component | `rustup target list --installed \| grep wasm32` | |
| `trunk` (if a Rust/WASM component exists) | Building/serving the web component | `trunk --version` | |
| Headless browser harness (e.g. Playwright, if visual verification is used) | Frontend behavioral DoD checks (§8) | harness-specific version check | |
| Tauri CLI (if a desktop/mobile shell exists) | Building the shell | `cargo tauri --version` | |

**A session MUST run a setup-check (`scripts/check_env.sh` and, on Windows,
`scripts/check_env.bat`, per `AGENTS.md`'s Session Initialization mandate) before starting
any task.** Per `AGENTS.md` §2.6, if a required prerequisite is missing or fails
verification, the agent MUST NOT simply halt and report: it attempts to install the missing
prerequisite for the current session, **appends the install command idempotently to
`scripts/setup_env.sh` (Linux/bash) and `scripts/setup_env.bat` (Windows)** — creating either
file if absent — informs the user (Escalation Trigger §13 #4 still fires, auto-resolving on
success), and only escalates without proceeding if no known-good install path exists.

**Pre-flight version sanity check (cross-reference `agents/PREFERRED_TOOLS.md`):** beyond
mere presence, the setup-check verifies installed tool versions are compatible with this
project's declared compatibility matrix (the table above, with specific version floors filled
in once known) — a present-but-incompatible tool is treated the same as a missing one.

## 5. Development & Test Configuration **[NEW]**

> Distinct from production secret/config handling, which the Architecture
> Specification owns. Without this, even a fully-built Phase 1 has nothing to run against.

- **Local configuration file(s):** [e.g., `config/development.toml` — path, and which values
  differ from `config/default.toml`]
- **Mock/test credential and secret seeding:** [e.g., which provider implementation is used
  in dev/test (a mock/in-memory provider per the Architecture Spec), what deterministic test
  values it returns, and where those are defined]
- **Test key material:** [e.g., how a session generates or obtains test signing keys (JWT,
  TLS, etc.) — MUST be clearly marked as non-production and MUST NOT be reused as real secrets]
- **Local infrastructure bring-up:** infrastructure services this project depends on
  (databases, object storage, etc. — per `agents/PREFERRED_SERVICES.md`) are brought up via
  `docker compose -f deploy/docker-compose.dev.yml up -d`; service definitions live in
  `deploy/`. State the command to verify services are healthy before tasks begin.
- **Environment variable overrides:** [naming convention, e.g. `<SERVICE>__<SECTION>__<KEY>`,
  and where the canonical list of overridable values is documented]

**Rule:** no task's Definition of Done may depend on a production credential, a production
provider, or any non-local infrastructure. If a task appears to require this, treat it as an
Escalation Trigger (§13), not something to work around silently.

## 6. Phases and Milestones **[EXTENDED]**
(Break down the development work into logical phases with clear milestones. **MANDATE:** Each
phase must be sized to comfortably fit within a single agent session — assume the executing
agent is less capable than the one drafting this Plan — including a buffer for complications.
Score each phase's complexity as `(task_count × 1) + (new_public_interfaces × 2) +
(cross_file_tasks × 2) + (cross_task_dependencies × 1.5)`; default ceiling is 15, adjustable
by the project owner based on experience with the actual executing agent. A phase scoring
above the ceiling should be split further before this Plan is finalized. **Independent of
sizing: per `AGENTS.md` §2.8, a Development Phase session completes at most one phase,
regardless of how much session capacity remains after that phase's Exit Criteria is
satisfied** — phase sizing controls what fits comfortably within a session; it never licenses
attempting more than one phase in a session.)

**Frontend Targeted Interleaving [NEW]:** where the project has a human-facing UI
component, phase sequencing does NOT group all backend work first and all frontend work
into a trailing phase, and does NOT build a screen/component's frontend against mocked or
stubbed data ahead of its real dependency being available. Instead: **each screen/component's
frontend implementation task is placed in the same phase as the real (non-mock) backend
endpoint or data source it depends on** — see `agents/exemplars/architecture_specification_template.md`
§9.1 (Sequencing Principles) for how this constrains the underlying Build Order this Plan's
Phase Index derives from, and that same template's §4.13 Asset Manifest for which
screens/components have a tracked HTML/image reference asset a phase's frontend task should
be built against. A phase that introduces a new backend
endpoint/data source without also scheduling the UI component(s) that consume it in the same
phase (where that component is already in scope per the Architecture Specification) should
be flagged and reconsidered before this Plan is finalized, the same as an over-scored phase.

### 6.1. Phase Index

> A fixed schema per phase, so phase boundaries are objectively checkable rather than
> descriptive. Every column is mandatory.

| Phase | Maps to Build-Order Step(s) | Component(s) | Title | Entry Criteria | Exit Criteria (DoD gate) | Primary Requirement Domains | Task Count |
|---|---|---|---|---|---|---|---|
| [N] | [ref] | [which component(s) this phase touches] | [title] | [what must already be true/exist before this phase can start] | [the single objective condition that proves this phase is complete] | [REQ-XXX domains] | [count] |

**Note:** the first or second phase's task list should include a task to scaffold the
project's root `README.md` (derived from the Architecture Specification's overview, stack,
and roadmap) — see §8's Task Template and `agents/DEVELOPMENT.md` §5.2.1. The final phase's
task list should include a corresponding README review/finalization task.

### 6.2. Phase Dependency Graph

> Express explicitly which phases can run in parallel (if the executing
> environment ever supports that) versus which are strictly sequential, which phases belong to
> which component, nor which single phase is the highest-leverage blocker — even in simple
> text-arrow form, so a reader doesn't have to infer it from prose.

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

> Populate from concrete, spec-derived
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

> Per-task schema aimed specifically at a
> lower-capability, single-session executing agent, generalized so it is not implicitly
> backend-biased — a Rust/WASM frontend task is not "done" by `cargo build` alone the way a
> backend task often is.

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
      run (e.g. `cargo nextest run -p <crate>`).
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

**Example: README scaffolding task (Phase 0 or 1):**
- **Task ID:** `DOC-001`
- **Description:** Scaffold the project root `README.md`, derived from Architecture
  Specification §1.2 (Product/System Overview), §6 (Technology Stack), and §9 (Implementation
  Roadmap).
- **Verification Method:** Build+Test (N/A — documentation task; DoD is content review).
- **Definition of Done:** README exists at repo root; covers overview, build/run instructions,
  and project structure; reviewed and approved by the user.

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
> agent sessions with no memory of prior sessions. Without this section, an agent starting a new
> session has no prescribed way to (a) confirm which phases are actually complete versus merely
> checked off, (b) recover from a session that was interrupted mid-phase, or (c) avoid re-doing
> or conflicting with already-merged work.

**At the start of every session, before starting any task, the agent MUST:**
1. Complete the §4 Environment setup-check (including the pre-flight version sanity check).
2. Read the Development Checklist (`[projectname]_dev_checklist.md`, used **in place** —
   never copied or renamed, per `AGENTS.md` §2.7) in full and identify the first phase with
   any unchecked task.
3. Run the project's build and test commands, for every component touched by that phase, as they
   currently stand (before making any change) and confirm the result matches what the checklist
   claims — if the checklist says a phase is done but the build/tests fail, STOP and report the
   discrepancy rather than proceeding or silently fixing it (see §13, Escalation Triggers).
4. Confirm the phase's **Entry Criteria** (§6.1) are actually satisfied by inspecting the
   relevant files/state directly, not by trusting the checklist alone.
5. Work only within the identified phase's tasks. **Per `AGENTS.md` §2.8, do not begin a later
   phase's tasks under any circumstances within this session** — not even with idle capacity
   remaining — unless explicitly instructed to treat this as an exception.

**At the end of every session, the agent MUST:**
1. Update the Development Checklist **in place** to reflect exactly what was completed (no
   partial credit — a task is checked only when its full DoD, including required artifacts, is
   satisfied).
2. Leave the repository in a hermetically buildable state for every component, even if the
   phase is not fully complete — never commit a known-broken intermediate state as the
   session's final output.
3. Produce/update this phase's **Phase Summary** (§11.3) — the structured record replacing
   an ad hoc session note — for the next session, the human reviewer, and any future
   Design-Phase session that may need it per `agents/DESIGN.md`'s Major Change Notification
   mandate.
4. **Stop.** Per `AGENTS.md` §2.8, the session does not proceed into the next phase even if
   this phase closed early with capacity remaining.

### 11.3. Phase Summary **[NEW]**

> Formalizes what was previously an unstructured "session note." A narrative-only note
> loses the actual evidence behind "this phase worked" — this section aggregates it, so a
> human, a future session, or a Design-Phase Claude session reopening this project for
> re-architecting has one place to find both *what happened* and *what proves it*, without
> re-deriving it from the Checklist's raw task entries.

**One file per phase, named `[projectname]_phaseN_summary.md`, produced at phase close
(whether the phase succeeded, partially succeeded, or was aborted).** Required content:

- **Header block:** Phase ID and title, Build Order step(s) it maps to (per §6.1), date
  completed, a brief self-description of the executing agent/session, and whether every DoD
  item in this phase is checked (Yes/No — if No, state which and why).
- **Tasks completed, with evidence aggregated per task** — for every task closed this phase,
  cite its Task ID and pull forward its actual Verification Method result: which exact
  commands were run, pass/fail, and a pointer to (or inline copy of, if short) each Required
  Artifact named in that task's DoD (test output, screenshot, accessibility report, etc.).
  This is aggregation, not re-invention — the evidence already exists per-task in the
  Checklist; this section collects it into one place rather than leaving a reader to
  reconstruct it task-by-task.
- **Deviations from the Plan** — any place where what was actually built differs from what
  the Plan specified, even minor ones. State "None" explicitly if there are none, rather
  than omitting the section.
- **Issues and problems encountered** — compiler errors, test failures, unexpected behavior,
  or implementation difficulties beyond what the Plan anticipated: symptom, root cause if
  determined, resolution. State "None" explicitly if there are none.
- **Assumptions made — a distinct section, not folded into Findings or Deviations.** Any
  decision the agent made that was not explicitly resolved in the Plan or Architecture
  Specification, flagged clearly so a future session or human reviewer can confirm or
  override it. This is the Development-Phase counterpart to the Design Phase's
  flagged-assumption discipline (`CLAUDE.md` §3.1) — an assumption silently buried inside a
  general "findings" note defeats the purpose the same way an unflagged Design-Phase
  assumption would.
- **Unplanned changes** — any change to files, interfaces, or behavior not specified in the
  task definitions, including changes to files outside a task's stated scope.
- **Tasks not completed**, and why (if known) — distinct from tasks deferred to a later
  phase on purpose.
- **Open items / risks introduced** — anything left unresolved that a future phase or
  session needs to know about, and any new risk discovered that isn't already in §7's Risk
  Register.
- **Escalation required: Yes/No.** If Yes, state the issue clearly, and classify which kind
  it is — **(A) a replanning escalation** (the Development Plan or Checklist needs to
  change) or **(B) a re-architecting escalation** (the Architecture Specification itself
  needs to change), per §13.1 — so the receiving session knows where to start reading and
  what kind of change to expect, rather than an undifferentiated "something needs to
  change." Provide enough detail that a Design-Phase Claude session can pick this up cold.

**If a phase's evidence reveals a problem severe enough that re-architecting or
re-development-planning is needed** (not a normal Escalation Trigger an agent resolves
in-session, but a structural problem with the Architecture Spec or Plan itself): the Phase
Summary states this explicitly, and the human is directed to start a new Design-Phase Claude
session, providing the **full series of Phase Summary files produced so far** (not just the
triggering one) as input — so that session has the complete history, not just the latest
snapshot, per `CLAUDE.md` §3.11's file-manifest principle applied to this scenario.

## 12. Abort / Rollback Protocol **[NEW]**

> Session Handoff (§11) covers *resuming* a session; this covers what an agent should do if a
> phase or task turns out to be unsalvageable *within* a session — a distinct situation that
> §11 alone does not address.

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
3. Record the abort and its cause in this phase's **Phase Summary** (§11.3) in enough detail
   that the next session (or the human reviewer) can decide whether the task needs
   re-specification rather than re-attempting the same approach blind.
4. Continue with other, unaffected tasks in the same phase if any remain and are still valid;
   otherwise end the session per §11's end-of-session steps (without beginning a new phase,
   per §6/§11/`AGENTS.md` §2.8).

**Rollback MUST NOT cross a phase boundary** (i.e., aborting a Phase N task must never revert
already-completed, already-checked-off work from Phase N-1 or earlier) **except** when the
abort's root cause is a defect discovered in that earlier work — in which case this is an
Escalation Trigger (§13), not a unilateral rollback, since it may invalidate downstream
completed work whose own validity depended on it.

## 13. Escalation Triggers **[NEW]**

> List the concrete conditions for stopping and asking a human rather than making an agent's own
> judgment call and proceeding, so a lower-capability agent has a checklist, not a principle to
> infer.

**The agent MUST stop and request human input (not guess, not silently choose an interpretation) when:**
- A task's Architecture Specification source (cited in its Traceability field) is ambiguous,
  silent on the question actually blocking implementation, or appears to conflict with another
  requirement.
- A Definition of Ready (§8) or phase Entry Criteria (§6.1) cannot be verified true or false
  from the repository's current state.
- An Abort (§12) occurs and the cause appears to be a defect in already-completed,
  already-checked-off work from a prior phase.
- A required prerequisite (§4) is unavailable in the session environment and has no documented
  fallback, **or a pre-flight version sanity check finds an installed tool version incompatible
  with the project's declared matrix and no known-good version can be installed** — in either
  case, after the self-install attempt and `scripts/setup_env.sh`/`.bat` append required by §4
  and `AGENTS.md` §2.6 has been tried and failed.
- Completing a task's DoD would require a production credential, production infrastructure, or
  any resource excluded by §5's rule.
- The agent's confidence that a generated artifact (code, config, policy) is correct is
  meaningfully lower than for comparable completed tasks — this is a judgment call the agent is
  expected to make conservatively, not optimistically.
- Any condition the Architecture Specification or `agents/DESIGN.md` elsewhere explicitly names
  as requiring user approval (e.g., a "Major Change" to the architecture).

**When escalating, the agent MUST:** state the specific task/phase, the specific blocking
question, and (if applicable) the options it has identified — matching the project's general
"Super-Strict Workflow" mandate of presenting an interpretation and plan before acting, applied
here to the narrower case of being blocked rather than merely about to start.

### 13.1. Escalation Requiring Design-Phase Re-Engagement (Distinct from In-Session Escalation Above)

The triggers above are resolved *within* the current Development-Phase session, by a human
answering a question. A different, more severe case exists: evidence accumulated in one or
more **Phase Summaries** (§11.3) reveals a structural problem with the **Development Plan or
the Architecture Specification itself** — not a single blocked task, but a sign that what was
designed doesn't actually hold up against what's been built so far. This cannot be resolved
in-session, since it requires re-opening Design Phase work, not answering a question.

This category splits into two, and the Phase Summary's Escalation Required field must state
which:

- **(A) Replanning escalation** — the **Development Plan or Checklist** needs to change
  (e.g. a phase was sized wrong, a task's Verification Method doesn't actually work in
  practice, the Phase Index needs reordering) but the underlying Architecture Specification
  is still sound. The receiving Design-Phase session should start by reading the Plan, not
  the Spec.
- **(B) Re-architecting escalation** — the **Architecture Specification itself** needs to
  change (e.g. a pattern of aborts traces back to the same architectural assumption, not a
  one-off task-level mistake; a chosen approach proves structurally unworkable). The
  receiving session needs to revisit Spec content, which will likely also require revisiting
  the Plan downstream of whatever changes.

When either is recognized: state explicitly, in the current Phase Summary, which category
applies and why, then direct the human to start a **new Design-Phase Claude session**,
providing the **full series of Phase Summary files produced so far** (not only the
triggering one) as input, so that session has complete history rather than a single
snapshot.

## 14. Change Control for This Plan **[NEW]**

> Governs changes to the *Plan and Checklist documents themselves* once execution has begun —
> without this, a plan silently drifts from what was actually built.

- A phase's task list MAY be amended after work has started only when a later phase or task
  discovers the original decomposition was incorrect or incomplete (e.g., a missing task, a
  task whose scope was wrongly estimated, a dependency that was missed).
- Any such amendment MUST be recorded in `CHANGELOG.md` (or, for projects not using a
  consolidated changelog, this Plan's own appended history) with: what changed, which
  phase/task, and why — not silently edited with no trace.
- Amendments MUST NOT retroactively mark already-completed tasks differently than they were
  actually completed (i.e., do not rewrite history to make a past task's DoD match what was
  actually delivered if it diverged — record the divergence instead, per §11's Phase Summary
  requirement, and raise it per §13 if it is significant).
- Renumbering existing Task IDs or Phase numbers is NOT permitted (per the ID-stability rule
  defined in Architecture Specification §3 and referenced via this Plan's §0 project note);
  new tasks are appended with new IDs, and deprecated tasks are marked `[DEPRECATED]` rather
  than deleted.
- Changes that affect more than one phase, or that change a phase's Exit Criteria, count as
  "Major" for the purposes of `agents/DESIGN.md`'s notification mandate and MUST be surfaced
  to the human reviewer even if the underlying Architecture Specification did not itself
  change.
- **The Development Checklist and Kickoff Prompt are amended in place, never reproduced as a
  new file** — per `AGENTS.md` §2.7; this applies to amendments under this section just as
  much as to routine checkbox updates.

## 15. Plan-Level Definition of Done **[NEW]**

> This section is narrower and more mechanical than the Architecture Specification's own
> GA/Definition-of-Done criteria (if any): it defines when the Plan and Checklist documents
> themselves can be considered complete and internally consistent.

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
- [ ] Every filename produced for this Plan conforms to the naming convention in `CLAUDE.md`
  §4 (`[projectname]_dev_plan_NN_topic_v[N].md`, `[projectname]_dev_checklist.md`,
  `[projectname]_dev_agent_prompt.md`).

## Appendix G - Glossary **[NEW]**

> For a spec with dense, project-specific
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

## Appendix
See `CHANGELOG.md` for this file's full version history.
