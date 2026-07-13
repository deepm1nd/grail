# Development Plan: [Project Name]
Complex/Multi-Phase Rust Project Variant

> **Use this variant** when the Architecture Spec is large (multiple files, 100+ requirements,
> multi-stage Build Order, and/or multiple Rust components) such that a flat Phase/Task list
> wouldn't let a reader answer at a glance: prerequisites, phase entry/exit conditions, DoD
> proof, phase dependencies, failure handling. Otherwise use the simpler flat plan.
>
> **Scope:** Rust-only, any number of Rust components (native services, Rust/WASM frontends,
> Tauri/UniFFI shells, CLI tools, shared crates).

## Table of Contents
0. Architecture Cross-Reference · 1. Introduction · 2. Technology Stack · 3. Project Folder
Structure · 4. Environment & Prerequisites · 5. Dev & Test Configuration · 6. Phases &
Milestones · 7. Risk Management · 8. Task Decomposition · 9. Test Strategy · 10. Logging
Strategy · 11. Session Handoff Protocol · 12. Abort/Rollback Protocol · 13. Escalation
Triggers · 14. Change Control · 15. Plan-Level Definition of Done · Appendix G Glossary ·
Appendix R Version History

---

## 0. Architecture Cross-Reference

| Document | Role |
|---|---|
| [Architecture Specification, all parts] | Authoritative source of all technical decisions |

Traceability source: Architecture Spec §3, physically split per `CLAUDE.md` §4.1 across
`_04_test_strategy` (§3.1–3.3) and `_05_verified_traceability` (§3.4–3.5). No separate
Requirements & Traceability document exists. **Precedence:** the Spec wins on conflict; this
Plan is corrected, not the reverse (§14). Not final until Design Step 9 (Plan & Checklist
Audit) clears — independent, adversarial, distinct from the drafting that produced it.

## 1. Introduction
- **Purpose / Scope / References:** link to the Architecture Specification.

## 2. Technology Stack

> State per component — different components (native, `wasm32-unknown-unknown`,
> Tauri/UniFFI) may have different crate ecosystems, test runners, verification methods.
> **Workspace edition/MSRV** (`agents/RUST_PREFERENCES.md` §0): state the Rust 2024
> declaration and workspace MSRV set at Design Step 5; if a publishable library crate exists,
> its separate lower consumer MSRV and enforcing CI stage (§10.2 cross-ref).

| Component | Crate type / target | Key crates | Test runner | Build command |
|---|---|---|---|---|

## 3. Project Folder Structure

**The project root is the repository root — unnamed, never a project-name subfolder** (a
clone lands in whatever directory name the clone tool assigns, not the project's name).

**`src/` vs `crates/` — mutually exclusive, decided once at Design Step 5.** A single-crate
project uses a root `Cargo.toml` (with `[package]`) plus `src/` — no `crates/` directory at
all. A multi-crate workspace uses a root `Cargo.toml` declaring `[workspace]` only (no
`[package]`, no root `src/`); every member crate lives under `crates/<crate_name>/`, each
with its own `Cargo.toml` and `src/`. A project never has both `src/` and `crates/` at the
root level — which one applies is recorded in the Architecture Specification's
Deployment/structure viewpoint, not left for a later session to infer from what happens to
exist. Member-crate folder names follow the existing no-dashes/underscores rule
(`agents/PREFERRED_DEPENDENCIES.md`) — not a new rule, just applying the existing one.

Workspace root tree: `Cargo.toml`, `src/` **or** `crates/` (never both, above), `services/`,
frontend/mobile dirs, `deploy/` (compose files), `assets/` (below), `scripts/`
(`setup_env.sh`/`.bat`, plus `scripts/metrics/` — parser scripts feeding `metrics/*.toml`,
`agents/PREFERRED_TOOLS.md`'s Canonical Commands table), `metrics/` (`coverage.toml`,
`tests.toml`, `deny.toml`, `audit.toml`, `playwright.toml` — CI-maintained, committed, not
gitignored, `agents/CI.md`), `.github/workflows/` (`ci.yml`), `deny.toml`,
`LICENSE.md` (static license text, final at Design Step 8, no reconciliation needed),
`THIRD_PARTY_LICENSES.md` (CI-maintained, drift-checked, `agents/CI.md` Stage 5).

**`assets/` — user-populated, filename-stable across Design and Development.** Where the
Architecture Spec's Asset Manifest (§4.13) lists user-supplied HTML/image/audio/video
assets, the user populates identical files under identical filenames into `assets/html/`,
`assets/images/`, `assets/audio/`, `assets/video/`. A task consuming one cites it by
filename against `assets/<type>/<filename>` and the Manifest entry — never re-describing
content in the task. Never rename/move/duplicate; a missing Manifest-specified asset is an
Escalation Trigger (§13). **Asset files themselves were never carried through Design
handoff notes** — this directory is genuinely the first place the actual files land,
possibly updated relative to what was shown during Design.

**Note — Trunk projects:** Trunk's build output directory (`dist/`) is distinct from this
`assets/` directory; see `agents/PREFERRED_TOOLS.md` for the required `Trunk.toml` pin
preventing collision.

**`.gitignore` — drafted at Design Step 8, reviewed/confirmed/extended at Development
Phase 0**, mirroring the `README.md` convention (`agents/DEVELOPMENT.md` §5.2.1). The
mandatory entries below use **depth-agnostic patterns** — they must match regardless of
whether a Rust crate/component or Trunk build lives at the repository root or in any
nested subfolder (workspace member, `libs/`, `services/`, etc.):

```gitignore
# Cargo build outputs — all profiles (debug, release, test, bench, doc) — any depth
**/target

# Trunk WASM build output — any depth (remove if no WASM/Trunk component)
**/dist

# Local environment — real credentials never committed; commit .env.example instead
.env

# IDE and OS noise (extend as needed for the team's tooling)
.DS_Store
.idea/
.vscode/
```

Additional project-specific entries (generated artifacts, local scratch dirs, etc.) are
appended at Development Phase 0 review, not invented speculatively at Design time.

Per `AGENTS.md` §2.1, non-reproducible evidence artifacts under `test/` (the concise
per-phase Verification file plus each phase's detailed screenshots/logs subfolder — see
§11.4 below) are **not** gitignored — they are committed, as they cannot be recreated after
a session crash. `/target` and `/dist` are reproducible from source and MUST be gitignored,
not committed.

## 4. Environment & Prerequisites Setup

| Prerequisite | Required for | Verification command |
|---|---|---|
| Rust toolchain (`rust-toolchain.toml`, §2's MSRV) | All components | `rustc --version` |
| `cargo-nextest` | Test execution | `cargo nextest --version` |
| `cargo-llvm-cov` | Coverage | `cargo llvm-cov --version` |
| `cargo-deny` / `cargo-audit` | CI gates | `cargo deny --version` / `cargo audit --version` |
| `sqlx-cli` (if SQL store) | Migrations | `sqlx --version` |
| `protoc` (if Protobuf/Kafka) | Codegen | `protoc --version` |
| Docker/`docker-compose` (if any `PREFERRED_SERVICES.md` service) | Infra bring-up | `docker compose version` |
| WASM target + `trunk` (if WASM component) | Web build | `trunk --version` |
| Tauri CLI (if desktop/mobile shell) | Shell build | `cargo tauri --version` |
| ESP-IDF toolchain, if ESP32 component (`agents/ESP32_ESPIDF_RUST_BUILD_GUIDE.md`) | Embedded build | `cargo +esp --version`, `echo $IDF_PATH` |

**`scripts/check_env.sh`/`.bat` run before any task.** Missing/failing prerequisite: agent
attempts install for the current session; **if it installs cleanly**, appends idempotently
to `scripts/setup_env.sh`/`.bat`, informs the user, proceeds (`AGENTS.md` §2.6). **If the
install fails, or any resulting version conflict/incompatibility surfaces:** this is no
longer a trivial resolve — the session stops per §13's escalation model, full stop.

## 5. Development & Test Configuration
- **Local config file(s), mock/test credential seeding, test key material** (clearly
  marked non-production), **local infra bring-up** (`docker compose -f
  deploy/docker-compose.dev.yml up -d`), **env-var override convention.**

**Rule:** no task's DoD may depend on a production credential/provider/infrastructure — that
requirement is itself an Escalation Trigger (§13).

## 6. Phases and Milestones

**Each phase must fit one agent session** (assume an agent less capable than the one
drafting this Plan). Complexity score: `(task_count × 1) + (new_public_interfaces × 2) +
(cross_file_tasks × 2) + (cross_task_dependencies × 1.5)`; default ceiling **15**,
project-tunable. Score above ceiling → split further, unless the user explicitly approves
a recorded override (§6.1's Complexity Score column carries the override note inline —
never a silent exception). **Independent of sizing: a session completes at most one phase**,
regardless of remaining capacity (`AGENTS.md` §2.8).

**Frontend Targeted Interleaving:** where a UI exists, each screen/component's frontend
task is placed in the **same phase** as the real (non-mock) backend/data source it depends
on — never earlier (forces a stub) and never batched into a trailing frontend-only phase.
Each such task's Design Refs (§8) cite the mockup's prose description (Spec §4.13) and its
logged Authority Level (`CLAUDE.md` §3.7) — a Conceptual-level mockup leaves more to the
task's own judgment than an Authoritative one, stated explicitly rather than left implicit.
A task introducing a page/view/setting not in the mockup traces back to the Design-Phase
Proactive UI-Impact flag that justified it.

### 6.1. Phase Index

**Session Unit** (`AGENTS.md` §2.8): declared per phase — `Phase` (default, whole phase per
session), `Task` (one task per session), or `Code+Verify` (one Code or Verify sub-task per
session — only meaningful for phases dominated by split tasks, §8). Changed later only via
§14 Plan-Change Escalation, never unilaterally by an executing session.

**Branch Name** (one per phase, assigned at drafting time): a human-readable, concise,
descriptive `snake_case` name reflecting the phase's actual content — e.g.
`phase2_auth_and_session_mgmt`, not `phase2` alone (uninformative) or the Title column
mechanically underscored. Name it for what a human skimming `git branch -a` would want to
see, based on the phase's real Title and task list. Assigned once here; changed later only
via §14 Plan-Change Escalation like any other Phase Index field.

| Phase | Build-Order Step(s) | Component(s) | Title | Branch Name | Session Unit | Entry Criteria | Exit Criteria | Req Domains | Task Count | Complexity Score |
|---|---|---|---|---|---|---|---|---|---|---|

**Complexity Score column — mandatory, computed, never left blank.** The §6 formula's value
for this phase, shown as computed (e.g. `11`), not just implied by Task Count. Over-ceiling
without a recorded override note in the same cell (e.g. `17 — override approved [date]`) is
a drafting defect, not a judgment call left to the executing session.

**Final phase includes a README review/finalization task** — the README itself is drafted
at Design Step 8, not scaffolded here; Development's Phase 0 task is to review, confirm, and
enhance it against the repository as it develops (`agents/DEVELOPMENT.md` §5.2.1).

### 6.2. Phase Dependency Graph

Express which phases run in parallel vs. strictly sequential, which component each belongs
to, and the single highest-leverage blocker phase ("critical path") — state explicitly if a
phase in one component depends on a phase in another.

## 7. Risk Management & Mitigation

Populate from concrete, spec-derived risks: every external/infra dependency assumed
available, every security-critical/hard-to-test algorithm, every spec-flagged tradeoff, every
cross-component contract a later phase depends on.

| Risk | Impact | Likelihood | Mitigation | Affected Phase(s) |
|---|---|---|---|---|

## 8. Task Decomposition

**Task Template:**
- **Task ID:** `[DOMAIN]-[NNN]` · **Phase** · **Component**
- **Description**
- **DoR** (what must be true to start — distinct from phase Entry Criteria)
- **File(s) touched** · **Dependencies** · **Effort (S/M/L)**
- **Verification Method:** Build+Test (exact `cargo` commands) | Visual/Behavioral (exact
  rendering + check: headless-browser assertion, screenshot diff, zero console errors,
  a11y assertion) | Hybrid

  **UI-touch rule:** any task that adds, modifies, or visibly changes a UI component,
  feature, or appearance MUST use **Visual/Behavioral** or **Hybrid** — never Build+Test
  alone — and MUST specify at least one screen capture in its DoD/Commands.
  **Consolidation:** if a phase has multiple UI-touching tasks whose changes are all
  visible together in one screen/state, one shared capture may satisfy all of them — cite
  it from each task's DoD rather than duplicating. If the changes are **not** all visible
  in the same screen/state (different pages, different interaction states, or a change not
  visible until a separate action), each requires its own capture. A task drafted as
  UI-touching with no capture specified is a Plan drafting defect (§15 DoD), not something
  left to the executing agent's judgment.
- **Commands**
- **DoD:** code builds (`command`) · verification passes (`command(s)`) · Test Case ID
  verified · Required Artifacts captured
- **Traceability:** REQ-XXX-NNN[, ...] — every task cites at least one requirement
- **Design Refs** (mandatory, every task, no exceptions): the specific Architecture Spec
  file + section + item-level ID/name this task's content derives from — e.g. `_06_viewpoints
  §4.5, EntityName`; `_07_interfaces_and_stack §5, endpoint-name`. No `_v[N]` version suffix
  (only one current version of each Spec file exists in the docs folder at any time). A
  trivial task with no meaningful external reference states so explicitly (e.g. "trivial —
  no external refs needed") rather than omitting the field. This is the drafting-time
  counterpart to `dev_prompt_template.md`'s on-demand reference table: known dependencies are
  pre-resolved here; the on-demand table remains the fallback for anything unanticipated.
- **Submit Point:** where in this task's execution a `submit` fires (`AGENTS.md` §2.1,
  `agents/AGENT_TOOL_POLICY.md`). At minimum, every task (or sub-task, if split below) has
  exactly one task-complete Submit Point at its own DoD satisfaction. Additionally flag
  **WIP-Checkpoint-Eligible: Yes/No** — Yes for any task identified at drafting time as
  long/risky (e.g. initial scaffolding, a large multi-file refactor) where a mid-task
  crash-safety submit is worth pre-declaring; the WIP checkpoint itself still fires only if
  actually needed during execution (§5.1 mechanics in `agents/DEVELOPMENT.md`).
- **Context for executing agent:** the one or two sentences resolving the likeliest
  ambiguity a lower-context agent would get wrong

**Mandatory Code/Verify Split** (`agents/DEVELOPMENT.md` §5.1): any task whose Verification
Method is **Build+Test** or **Hybrid** is split into two sub-tasks at the same Task ID,
suffixed `a` (Code) and `b` (Verify) — e.g. `DOMAIN-005a`, `DOMAIN-005b`. Neither suffix is
ever renumbered once assigned (stable-ID rule, `AGENTS.md` §2.1). Exempt only for pure
Visual/Behavioral tasks (no build-test-debug cycle exists to isolate).
- **`005a` (Code):** DoD is a clean build only — no test execution required to close it. Own
  Submit Point at its own DoD.
- **`005b` (Verify):** DoR is `005a`'s Submit Point reached. DoD is the task's original DoD
  (tests pass, artifacts captured) — this is where the build-test-debug loop lives, isolated
  from `005a`'s own session/context. Own Submit Point at its own DoD.
- **Counts as two tasks** against the §6 Phase Sizing complexity formula — stated explicitly
  so sizing doesn't silently overrun once splits are applied.

**README task (final phase):** `DOC-FINAL` — review the Design-drafted `README.md` against
the as-built system, correct any divergence, get user approval. **DoD:** N/A build command;
DoD is content review and user approval. **Design Refs:** N/A — sourced from the as-built
repository, not an Architecture Spec section. **Submit Point:** at DoD satisfaction; not
split (Visual/Behavioral-equivalent, no build-test-debug cycle).

## 9. Test Strategy & Plan
Unit / Integration / System & Acceptance (for any UI/Tauri component: browser/device-target
matrix and headless-test harness) plans.

## 10. Logging Strategy
Incorporate the Architecture Specification's logging strategy and instrumentation tasks.

## 11. Session Handoff Protocol

**At session start, before any task:** (1) run §4's setup-check including the version
sanity check; (2) read the Checklist (`[projectname]_dev_checklist.md`, used **in place**,
never copied) and identify the first phase with any unchecked task; (3) run the project's
current build/test commands and confirm the result matches what the Checklist claims — a
discrepancy is itself an escalation, not something silently fixed; (4) confirm the phase's
Entry Criteria by inspecting repo state directly; (5) **for the identified phase's tasks,
run the three-way task-state check against declared Submit Points**
(`agents/DEVELOPMENT.md` §5.2 step 2): no submit → not started; a WIP-Checkpoint submit only
→ resume in place from that checkpoint's own description, never redo from scratch; a
task-complete submit → done. A checklist mark with no matching submit, or vice versa, is an
Escalation Trigger (§13), never silently patched over. (6) work only within the identified
phase and the session's declared Session Unit (`AGENTS.md` §2.8) — never begin work outside
that scope, even with capacity remaining.

**At session end, on normal completion:** update the Checklist in place; leave every
component hermetically buildable even if the phase isn't fully done; produce/update the
Phase Summary (§11.3); stop regardless of remaining capacity.

**On an unresolved Escalation Trigger (§13): do not reach normal session end.** Stop
immediately per §13's model instead.

### 11.3. Phase Summary

**One file per phase, `[projectname]_phaseN_summary.md`**, produced at phase close —
whether the phase succeeded, partially succeeded, or the session stopped on an escalation.
Required content:
- **Header:** Phase ID/title, Build-Order step(s), date, executing session self-description,
  every DoD item checked (Yes/No — if No, which and why).
- **Tasks completed, with evidence** — Task ID, exact Verification Method result
  (commands, pass/fail), pointer to each Required Artifact.
- **Deviations from the Plan** (state "None" explicitly if none).
- **Issues and problems encountered** — symptom, root cause if known, resolution, or full
  diagnostic detail if this is the reason the session stopped (§13).
- **Assumptions made** — own section, each flagged clearly.
- **Unplanned changes.**
- **Tasks not completed, and why.**
- **Open items / risks** not already in §7.
- **Escalation Required: Yes/No.** If Yes: state the issue and classify **(A) replanning**
  (Plan/Checklist needs to change) or **(B) re-architecting** (Architecture Spec needs to
  change) — enough detail for a Design Phase session to act on cold, since per §13 the
  agent does not attempt further resolution itself.

### 11.4. Verification File

**One file per phase, `test/[projectname]_phase_[N]_verification.md`, appended to in
place across every session that touches the phase** (never copied/renamed — same
in-place discipline as the Checklist, `AGENTS.md` §2.7). It is the concise evidence
receipt for the phase's task claims — distinct from the Phase Summary (§11.3), which is
the narrative. The Phase Summary links to this file rather than repeating its content.

**Scope — which tasks get an entry:** any task whose Verification Method
(`agents/exemplars/development_plan_template.md` §8) is **Build+Test**, **Hybrid**, or
**Visual/Behavioral** — i.e., any task that produces a real build result, test result, or
UI state. A task with no Verification Method beyond human review/approval (e.g. a
documentation-only task) has nothing to log and is exempt.

**Format — evidence lines, not prose. Target 20–30 lines for the whole file.** One entry
per task, appended immediately when that task's DoD is satisfied (same continuous-update
timing as the Checklist), each entry no more than a few lines:
- **Build/test tasks:** **the actual final line of terminal output, copied verbatim** — the
  real `Finished`/`error` line `cargo build`/`cargo check` prints, or the real `cargo
  nextest` summary line (e.g. `Summary [0.021s] 14 tests run: 14 passed, 0 skipped`).
  **Never a paraphrase, description, or restated claim in the agent's own words** — a line
  like "Result: Successfully compiled using `cargo check -p [crate]`" is a violation of this
  format, not an acceptable summary, because it isn't evidence the command was actually run
  or what it actually printed. If the true output line is long, truncate trailing
  whitespace/noise only — never rewrite its content. Never the full raw log.
- **UI/visual tasks (static views** — landing, sign-in, sign-out, settings, a UI element
  added to an existing page, or similar): one final-state screenshot is sufficient.
- **UI/visual tasks (dynamic scenes** — animation, multi-step interaction, anything that
  changes meaningfully over time): three short clips (start, middle, finishing — roughly
  5 seconds each), not one long recording.
- Each entry cites the artifact by filename, pointing into that phase's detail folder
  (below) — e.g. `Task AUTH_003: cargo nextest ... 6 passed. Screenshot:
  AUTH_003_login_success.png`.

**Detail folder — `test/phase_[N]/`:** holds the actual screenshots/clips named
`[TASK_ID]_[short_description].[ext]` (e.g. `AUTH_003_login_success.png`,
`UI_007_toast_animation_start.mp4`). Raw build/test logs are **not** retained here —
only the summary line goes in the Verification file itself, per the Mandatory Artifact
Preservation policy's non-reproducible-evidence scope (`AGENTS.md` §2.1). Committed, not
gitignored (§3 above).

## 12. Abort / Rollback Protocol

Treat a task/phase as aborted (not silently reworked) when: the chosen approach is found to
violate an Architecture Spec constraint; a DoR turns out false; completing as specified
would require modifying a file/contract outside its stated scope. **On abort:** revert
uncommitted changes for that task to its **last submitted state** (its most recent
task-complete submit, or its most recent WIP checkpoint if no task-complete submit exists,
or a clean pre-task state if neither exists) — never to a phase-level checkpoint, since
per-task Submit Points (§8) mean sibling tasks' completed work is never at risk from this
task's abort; do not mark it complete; record the abort and cause in the Phase Summary.
**Per §13, an abort the agent cannot resolve itself now stops the entire session** — it does
not continue with other unaffected tasks in the phase. Rollback never crosses a phase
boundary except when root cause is a defect in already-completed earlier-phase work — that's
an Escalation Trigger (§13), not a unilateral rollback.

## 13. Escalation Triggers — Stop, Summarize, Wait

**The sole case that continues without stopping:** a missing tool/prerequisite with a known
install command that installs cleanly (`agents/PREFERRED_TOOLS.md` Missing Tool Protocol) —
self-install, append to `scripts/setup_env.sh`/`.bat`, inform the user, proceed.

**Everything else the agent cannot resolve itself — a package/version conflict, an install
that fails, a persistent test failure, an ambiguous Spec question, an unverifiable DoR/Entry
Criteria, a low-confidence artifact, a task requiring a production credential/
infrastructure — stops the entire session immediately:**
1. Halt all task work — no partial continuation to other tasks, no further troubleshooting.
2. Write `[projectname]_phaseN_summary.md` (§11.3) with full diagnostic detail: what was
   tried, exact failure output, and — if determinable — the (A)/(B) classification.
3. Leave the repository in its last clean, committed state. No PR, no further progress.
4. Stop. The human brings the Phase Summary to a Design Phase session, which diagnoses the
   issue, updates the Architecture Spec and/or this Plan as needed, determines the correct
   restart phase, and restructures the Plan/Checklist. The human then rolls the repository
   back to the end of the last known-good phase and hands a fresh Development Phase session
   the updated documents to resume from there.

### 13.1. Escalation Requiring Design-Phase Re-Engagement

Distinct from an in-session trigger above: evidence across multiple Phase Summaries
revealing a structural Plan/Spec problem, not one blocked task.
- **(A) Replanning** — Plan/Checklist needs to change (bad phase sizing, a broken
  Verification Method, an orphan citation, Checklist/Plan drift); Spec is sound.
- **(B) Re-architecting** — the Architecture Specification itself needs to change (a
  pattern of aborts traces to one architectural assumption; a chosen approach is
  structurally unworkable).

State which applies in the Phase Summary and direct the human to a new Design Phase
session with the **full series of Phase Summaries produced so far**, not just the
triggering one.

## 14. Change Control for This Plan

A phase's task list may be amended after work starts only when a later phase/task discovers
the original decomposition was wrong/incomplete. Record in `CHANGELOG.md`: what changed,
which phase/task, why. Never retroactively mark completed tasks differently than they were
actually completed. Task/Phase IDs are never renumbered — new tasks get new IDs, deprecated
ones marked `[DEPRECATED]`. Changes spanning more than one phase, or changing Exit Criteria,
are "Major" and must be surfaced to the reviewer even without a Spec change. **The Checklist
and Dev Prompt are amended in place, never reproduced as a new file** (`AGENTS.md` §2.7).

**Ambiguity-resolving amendments.** Any amendment that resolves an ambiguity — where
already-completed phases might not have satisfied the now-clarified intent — MUST add a
verification task to the next dependent, not-yet-started phase. That task explicitly checks
whether prior work satisfies the clarified requirement, corrects it if not, and confirms the
result via the standard DoD/Verification-Method pattern (§8). It never assumes the old work
happens to be compatible.

## 15. Plan-Level Definition of Done

- [ ] Every Core-status requirement ID (Spec §3) appears in ≥1 task's Traceability field.
- [ ] No task cites a requirement ID that doesn't exist (no orphan citations).
- [ ] Every phase (§6.1) has non-empty Entry and Exit Criteria.
- [ ] Every task (§8) has a non-empty Verification Method and ≥1 DoD checkbox.
- [ ] The Phase Dependency Graph (§6.2) is acyclic and every phase reachable from Phase 0.
- [ ] The Checklist contains exactly one line per task DoD item — no drift.
- [ ] §0's Architecture Cross-Reference lists every source document cited in §8.
- [ ] Every filename conforms to `CLAUDE.md` §4 (`[projectname]_dev_plan_NN_topic_v[N].md`,
  `[projectname]_dev_checklist.md`, `[projectname]_dev_prompt.md`).
- [ ] Every task (§8) has a non-empty Design Refs entry (or an explicit "trivial — no
  external refs needed" statement) and a stated Submit Point.
- [ ] Every task whose Verification Method is Build+Test or Hybrid is split into `a`/`b`
  sub-tasks per §8's mandatory Code/Verify split rule; no such task remains unsplit.
- [ ] Every phase (§6.1) has a stated Session Unit, consistent with its task composition
  (e.g. `Code+Verify` only where the phase is dominated by split tasks).
- [ ] Every phase (§6.1) shows a computed Complexity Score, recomputable from its own listed
  tasks; none over the §6 ceiling without a recorded override note in the same cell.

## Appendix G — Glossary

| Term | Meaning |
|---|---|
| DoD / DoR | Definition of Done / Ready |
| [project-specific terms] | |

## Appendix R — Version History

**Per `CLAUDE.md` §4.3 — this file's own version history, and only this file's.** One row
per session that bumped *this specific* numbered file (e.g. `_01_overview`), never the whole
Plan's combined history — each of the 4 Dev Plan files carries its own independent Appendix
R, matching its own independent `_v[N]` counter. Appended to, one new row per bump, in the
same pass as the bump itself; prior rows are never rewritten or removed. This is the file's
*only* changelog/metadata surface — **no frontmatter or header box of any kind at the top of
the file** (no `Status:`, `Owner:`, `Last Updated:`, or similar field, in any format) **and
no version/revision commentary anywhere else in the file** — everything of that nature lives
in this appendix alone.

| Version | Date | Session / Step | Changes |
|---|---|---|---|
| v1 | [date] | Step 8 (initial) | Initial creation. |

---
See `CHANGELOG.md` for this file's full version history.
