# Development Plan: [Project Name]
Complex/Multi-Task-Group Rust Project Variant

> **Use this variant** when the Architecture Spec is large (multiple files, 100+ requirements,
> multi-stage Build Order, and/or multiple Rust components) such that a flat Task-Group/Task list
> wouldn't let a reader answer at a glance: prerequisites, Task Group entry/exit conditions, DoD
> proof, Task Group dependencies, failure handling. Otherwise use the simpler flat plan.
>
> **Scope:** Rust-only, any number of Rust components (native services, Rust/WASM frontends,
> Tauri/UniFFI shells, CLI tools, shared crates).

## Table of Contents
0. Architecture Cross-Reference · 1. Introduction · 2. Technology Stack · 3. Project Folder
Structure · 4. Environment & Prerequisites · 5. Dev & Test Configuration · 6. Task Groups &
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

**`publish = false` is a mechanical scaffold default, set the moment a crate is scaffolded,
not a documentation reminder added later.** Every member crate's generated `Cargo.toml`
MUST include `publish = false` in its `[package]` table from the moment it's scaffolded
(multi-crate workspace: every crate under `crates/<crate_name>/`; single-crate project: the
root `Cargo.toml`'s `[package]` table) — enforced here, at scaffold time, not merely
cross-referenced from `PREFERRED_TOOLS.md`. This is what allows cargo-deny's
`[licenses.private]` (`agents/PREFERRED_TOOLS.md`'s `deny.toml` skeleton) to correctly
recognize the crate as private and exclude it from the dependency-license check — a crate
without `publish = false` is not reliably recognized as private by that mechanism.

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
Task Group 0**, mirroring the `README.md` convention (`agents/DEVELOPMENT.md` §5.2.1). The
mandatory entries below use **depth-agnostic patterns** — they must match regardless of
whether a Rust crate/component or Trunk build lives at the repository root or in any
nested subfolder (workspace member, `libs/`, `services/`, etc.):

```gitignore
# Cargo build outputs — all profiles (debug, release, test, bench, doc) — any depth
**/target

# Trunk WASM build output — any depth (remove if no WASM/Trunk component)
**/dist

# Node modules (Playwright/frontend tooling) — any depth (remove if no Node-based tooling)
**/node_modules

# Local environment — real credentials never committed; commit .env.example instead
.env

# IDE and OS noise (extend as needed for the team's tooling)
.DS_Store
.idea/
.vscode/
```

Additional project-specific entries (generated artifacts, local scratch dirs, etc.) are
appended at Development's Task Group 0 review, not invented speculatively at Design time.

Per `AGENTS.md` §2.1, non-reproducible evidence artifacts under `test/` (the concise
per-Task-Group Verification file plus each Task Group's detailed screenshots/logs subfolder — see
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
  test/containers/docker-compose.dev.yml up -d`), **env-var override convention.**

**Rule:** no task's DoD may depend on a production credential/provider/infrastructure — that
requirement is itself an Escalation Trigger (§13).

## 6. Task Groups and Milestones

**Each Task Group must fit one agent session** (assume an agent less capable than the one
drafting this Plan). Complexity score: `(task_count × 1) + (new_public_interfaces × 2) +
(cross_file_tasks × 2) + (cross_task_dependencies × 1.5) + (first_integration_risk × 5)`;
default ceiling **15**, project-tunable. **`first_integration_risk`** is `1` if this Task
Group's own Exit Criteria requires genuine cross-crate/cross-boundary execution that no
earlier Task Group's Exit Criteria already required, else `0` — this is the term Confirmed
Finding 7 (`lessons_learned_grail_gap.md`) identifies as missing: without it, a Build Order
Step whose Exit Criteria silently assumes prior integration that was never actually
achieved scores as if it were ordinary internal task complexity, when first-time assembly
of previously-isolated components is a qualitatively different, higher-risk kind of work.
Score above ceiling → split further, unless the user explicitly approves
a recorded override (§6.1's Complexity Score column carries the override note inline —
never a silent exception). **Independent of sizing: a session completes at most one Task Group**,
regardless of remaining capacity (`AGENTS.md` §2.8).

**Frontend Targeted Interleaving:** where a UI exists, each screen/component's frontend
task is placed in the **same Task Group** as the real (non-mock) backend/data source it depends
on — never earlier (forces a stub) and never batched into a trailing frontend-only Task Group.
Each such task's Design Refs (§8) cite the mockup's prose description (Spec §4.13) and its
logged Authority Level (`CLAUDE.md` §3.7) — a Conceptual-level mockup leaves more to the
task's own judgment than an Authoritative one, stated explicitly rather than left implicit.
A task introducing a page/view/setting not in the mockup traces back to the Design-Phase
Proactive UI-Impact flag that justified it.

**Walking Skeleton Milestone (architecture-risk reduction — distinct from MVP/product
scoping, never conflated with it):** for any Build Order with more than ~3 components that
must eventually run together, the Build Order MUST include an early, explicitly-named
Task Group — before the bulk of per-component work — whose Exit Criteria is a tiny,
real, end-to-end slice that actually links the main architectural components together
(stub/minimal pieces are fine; a thrown-away prototype is not — this is production code the
system continues to grow from). Its Verification Method requires a genuine cross-crate test
(§8's Test-Type-to-Naming Binding), living in a dedicated sibling test crate — its own
`Cargo.toml` depending on the real domain crates as ordinary dependencies — never a single
domain crate's own `tests/` directory, which by Rust's own compilation model only ever sees
that one crate's public API. This Task Group's own `first_integration_risk` term above is
always `1`. Deferring all integration to a single late capstone Task Group sized as if it's
trivial (the gap `lessons_learned_grail_gap.md` documents) is exactly what this milestone
exists to prevent — it does not replace a later, fuller integration/system-test Task Group,
it establishes early that the architecture actually composes.

**Maturity-Triggered Component Integration:** independent of the Walking Skeleton
milestone above, any individual component — not just the architecture as a whole — gets
its own minimal integration test **the moment it reaches minimum functional maturity**
(i.e. the point its own unit-level Exit Criteria would otherwise be satisfied), not
deferred to a later, larger integration effort. Concretely: the Task Group in which a
component's core functionality is completed also includes a task exercising that
component against at least one real (non-mock) counterpart it's designed to integrate
with — sized as an ordinary task within that same Task Group for a component with few
integration points, or, where a component integrates with many others (a genuine
many-to-many boundary, not just one adjacent counterpart), as its own dedicated Task Group
immediately following, rather than folded in as an afterthought. This is a standing
per-component rule, applied at Task Group sequencing time (§6.1), not left to a Step 9
audit to catch after the fact.

### 6.1. Task Group Index

**Session Unit** (`AGENTS.md` §2.8): declared per Task Group — `Task Group` (default, whole Task Group per
session), `Task` (one task per session), or `Code+Verify` (one Code or Verify sub-task per
session — only meaningful for Task Groups dominated by split tasks, §8). Changed later only via
§14 Plan-Change Escalation, never unilaterally by an executing session.

**Branch Name** (one per Task Group, assigned at drafting time): a human-readable, concise,
descriptive `snake_case` name reflecting the Task Group's actual content — e.g.
`task_group2_auth_and_session_mgmt`, not `task_group2` alone (uninformative) or the Title column
mechanically underscored. Name it for what a human skimming `git branch -a` would want to
see, based on the Task Group's real Title and task list. Assigned once here; changed later only
via §14 Plan-Change Escalation like any other Task Group Index field.

| Task Group | Build-Order Step(s) | Component(s) | Title | Branch Name | Session Unit | Entry Criteria | Exit Criteria | Req Domains | Task Count | Complexity Score |
|---|---|---|---|---|---|---|---|---|---|---|

**Complexity Score column — mandatory, computed, never left blank.** The §6 formula's value
for this Task Group, shown as computed (e.g. `11`), not just implied by Task Count. Over-ceiling
without a recorded override note in the same cell (e.g. `17 — override approved [date]`) is
a drafting defect, not a judgment call left to the executing session.

**Final Task Group includes a README review/finalization task** — the README itself is drafted
at Design Step 8, not scaffolded here; Development's Task Group 0 task is to review, confirm, and
enhance it against the repository as it develops (`agents/DEVELOPMENT.md` §5.2.1).

### 6.2. Task Group Dependency Graph

Express which Task Groups run in parallel vs. strictly sequential, which component each belongs
to, and the single highest-leverage blocker Task Group ("critical path") — state explicitly if a
Task Group in one component depends on a Task Group in another.

## 7. Risk Management & Mitigation

Populate from concrete, spec-derived risks: every external/infra dependency assumed
available, every security-critical/hard-to-test algorithm, every spec-flagged tradeoff, every
cross-component contract a later Task Group depends on.

| Risk | Impact | Likelihood | Mitigation | Affected Task Group(s) |
|---|---|---|---|---|

## 8. Task Decomposition

**Task Template:**
- **Task ID:** `[DOMAIN]-[NNN]` · **Task Group** · **Component**
- **Description**
- **DoR** (what must be true to start — distinct from Task Group Entry Criteria)
- **File(s) touched** · **Dependencies** · **Effort (S/M/L)**
- **Verification Method:** Build+Test (exact `cargo` commands) | Visual/Behavioral (exact
  rendering + check: headless-browser assertion, screenshot diff, zero console errors,
  a11y assertion) | Hybrid

  **UI-touch rule:** any task that adds, modifies, or visibly changes a UI component,
  feature, or appearance MUST use **Visual/Behavioral** or **Hybrid** — never Build+Test
  alone.

  **Visual State Capture Completeness (v0.12.8 — standalone mandate, applies to
  Development, Maintenance §6 M3 Asset Manifest items, and Release):** every **distinct
  visual state** newly introduced or visibly changed by the work gets its own screen
  capture — regardless of how many tasks/items/Steps the work is split across; this is no
  longer framed as a task-consolidation exception. A "distinct visual state" is
  mechanically defined as any of:
  1. A distinct page/route/screen.
  2. A distinct overlay, modal, dialog, dropdown, or popover triggered from a parent screen
     (captured separately from the screen that triggers it).
  3. A distinct toggled/switched view (tabs, accordions, view-mode switches).
  4. A distinct discrete interaction state that **renders visibly differently**: `default`,
     `hover`, `focus`, `active/pressed`, `disabled`, `loading`, `empty`, `error`,
     `populated`. (An interaction state that changes only a non-visual property — e.g. a
     `disabled` state with no visible style change — is exempt; the test is "renders
     visibly differently," not "has a different HTML/CSS attribute.")
  5. A distinct responsive breakpoint **only if** the Architecture Specification declares
     specific breakpoints as an in-scope UI requirement — otherwise out of scope by default,
     and even then scoped to exactly the declared breakpoint set, never an open-ended
     viewport sweep.

  A capture may be cited/reused across multiple tasks/items only when it's the literal
  same state, never a merely-similar one. A task drafted as UI-touching with no capture
  specified for each of its distinct visual states is a Plan drafting defect (§15 DoD), not
  something left to the executing agent's judgment.

  **Asset-fidelity check:** for any touched visual state governed by an Asset Manifest
  entry (`CLAUDE.md` §3.7 / `maintenance_batch_template.md` §3), Verification includes a
  basic side-by-side comparison — the captured screenshot alongside the governing asset —
  with a one-line pass/deviation note (layout, color/branding, typography, spacing — not
  pixel-perfect diffing). A deviation doesn't auto-fail the task, but must be stated
  explicitly, never silently passed over.
- **Commands**
- **DoD:** code builds (`command`) · verification passes (`command(s)`) · Test Case ID
  verified · Required Artifacts captured · **any new or modified `pub` item carries a
  rustdoc comment** (`AGENTS.md` §2.3 — mandatory, not a separate documentation task;
  includes backfilling any pre-existing undocumented `pub` item this task's own File(s)
  touched field already lists, boy-scout-rule)

  **Test-Type-to-Naming Binding (mandatory, all Test Case Types):** every test function
  implementing a Test Case cited in this task's Traceability field is named
  `test_<type>_<nnnn>__description`, where `<type>` is exactly the Test Case's own
  Design-time `Type` (Architecture Specification §3.2) lowercased —
  `test_unit_<nnnn>__description`, `test_integration_<nnnn>__description`,
  `test_system_<nnnn>__description`, `test_acceptance_<nnnn>__description`. This
  supersedes the bare `test_<nnnn>__description` convention referenced elsewhere
  (`agents/MAINTENANCE.md` §6a — that file's own naming-convention section requires a
  matching update; flagged here as a pending follow-up since it wasn't available to edit
  in this session) with the type-qualified form. **For any Test Case whose Type is
  Integration, System, or Acceptance, the DoD is not satisfied by the naming convention
  alone** — the implementing test MUST also be compiled into a binary that genuinely links
  the multiple crates/components the Type claims to exercise (e.g. a dedicated sibling
  test crate per §6's Walking Skeleton convention, never a single domain crate's own
  `tests/` directory, which by Rust's own compilation model only ever sees that one
  crate's public API). The task's DoD line states which crates/binary the test actually
  wires together — not merely that "Test Case ID verified" — since this is exactly the
  fact a Step 9 audit (`CLAUDE.md` §3.4) mechanically checks via `cargo metadata`/nextest
  binary listing against the naming convention above. A task claiming an
  Integration/System/Acceptance-type Test Case backed by a test compiled into a
  single-crate binary does not satisfy this DoD item, regardless of the test's name.
- **Traceability:** REQ-XXX-NNN[, ...] — every task cites at least one requirement
- **Design Refs** (mandatory, every task, no exceptions): the specific Architecture Spec
  file + section + item-level ID/name this task's content derives from — e.g. `_06_viewpoints
  §4.5, EntityName`; `_07_interfaces_and_stack §5, endpoint-name`. No `_v[N]` version suffix
  (only one current version of each Spec file exists in the docs folder at any time). A
  trivial task with no meaningful external reference states so explicitly (e.g. "trivial —
  no external refs needed") rather than omitting the field. This is the drafting-time
  counterpart to `dev_prompt_template.md`'s on-demand reference table: known dependencies are
  pre-resolved here; the on-demand table remains the fallback for anything unanticipated.
  **The rustdoc DoD item above is mechanical and global** (`AGENTS.md` §2.3 — applies to
  every task touching a `pub` item, stated once, not repeated per-task here); **what the
  doc content should actually say remains this field's own per-task judgment call** —
  Design still cites, in Design Refs, whichever Spec §2.2/§2.3 User-Facing Description or
  Diátaxis Destination content (if any) this task's rustdoc should draw on. The global
  rule doesn't relieve Design of thinking about doc content; it only removes the need to
  restate "rustdoc required" on every task.
- **WIP-Checkpoint:** `None`, or a **stated, concrete checkpoint point** for any task judged
  at drafting time to be long/risky (e.g. initial scaffolding, a large multi-file refactor) —
  a real sub-milestone within that task's own work (e.g. "core parsing logic implemented and
  unit-tested, before wiring to the RPC layer"), or, where no natural sub-milestone exists,
  an invented point roughly halfway through the task's expected work (e.g. "roughly halfway
  — initial struct/trait definitions compiling, before implementation logic begins"). This
  is the **only** save point that can occur before the task's own DoD is satisfied
  (`AGENTS.md` §2.1) — there is no runtime/non-convergence trigger, so a task with `None`
  here has no mid-task save point at all, by design; the executing agent recognizes and
  fires the checkpoint mechanically once the stated point is reached, it never decides
  independently whether or where to checkpoint. **This field is populated at drafting time
  for every task, never left as a bare eligibility flag** — a task judged risky enough to
  need a checkpoint but drafted with no stated location is a Plan drafting defect (§15 DoD),
  not something left to the executing session's judgment.
- **Submit Point:** individual tasks no longer carry their own task-complete Submit Point
  (`AGENTS.md` §2.1) — the Session Unit's Submit Point occurs once, at the unit's own
  completion (the Task Group's Final Wrap-Up Submit, for the default `Task Group` Session
  Unit). This field is retained only to state, where relevant, any task-specific note about
  how that task's completion feeds the unit's eventual Submit Point (e.g. a task whose
  artifacts the Wrap-Up Submit depends on) — it is not itself a per-task save point.
- **Context for executing agent:** the one or two sentences resolving the likeliest
  ambiguity a lower-context agent would get wrong

**Mandatory Code/Verify Split** (`agents/DEVELOPMENT.md` §5.1): any task whose Verification
Method is **Build+Test** or **Hybrid** is split into two sub-tasks at the same Task ID,
suffixed `a` (Code) and `b` (Verify) — e.g. `DOMAIN-005a`, `DOMAIN-005b`. Neither suffix is
ever renumbered once assigned (stable-ID rule, `AGENTS.md` §2.1). Exempt only for pure
Visual/Behavioral tasks (no build-test-debug cycle exists to isolate).
- **`005a` (Code):** DoD is a clean build only — no test execution required to close it. No
  longer has its own Submit Point — DoD satisfaction and a flipped Checklist box are
  sufficient to move to `005b`; both sub-tasks' work is saved together at the Task Group's
  single Final Wrap-Up Submit (`AGENTS.md` §2.1). May carry its own WIP-Checkpoint field
  (above) if judged long/risky on its own.
- **`005b` (Verify):** DoR is `005a`'s DoD satisfied. DoD is the task's original DoD (tests
  pass, artifacts captured) — this is where the build-test-debug loop lives, isolated from
  `005a`'s own session/context. Likewise no Submit Point of its own; may carry its own
  WIP-Checkpoint field if judged long/risky on its own.
- **Counts as two tasks** against the §6 Task Group Sizing complexity formula — stated explicitly
  so sizing doesn't silently overrun once splits are applied.

**README task (final Task Group):** `DOC-FINAL` — review the Design-drafted `README.md` against
the as-built system, correct any divergence, get user approval. **DoD:** N/A build command;
DoD is content review and user approval. **Design Refs:** N/A — sourced from the as-built
repository, not an Architecture Spec section. **WIP-Checkpoint:** `None` — not
split (Visual/Behavioral-equivalent, no build-test-debug cycle); saved with the rest of the
Task Group at its Final Wrap-Up Submit.

**Productization Readiness tasks (final Task Group):** one `PROD-NNN` task per applicable item
of the Productization Readiness Checklist (`agents/DEVELOPMENT.md` §5.2.4) — applicability
of `PROD-007`–`PROD-009` determined by Design Step 5's Productization Applicability
finding. Each follows the standard Task Template above (Design Refs, Verification Method,
DoD, WIP-Checkpoint); none are split (Code/Verify), being Visual/Behavioral-equivalent or
Hybrid review-and-confirm tasks rather than a build-test-debug cycle.

- **`PROD-001` — Regression Traceability:** confirm every Core Requirement ID (Spec §3)
  has a row in `test/[projectname]_requirement_traceability.md` and that every listed test
  is independently invocable by name/tag (actually run each by name, not merely assumed
  present). **Verification Method:** Hybrid. **DoD:** table complete, every listed test
  confirmed runnable by name. **Design Refs:** Spec §3.
- **`PROD-002` — Versioning:** tag the repository at the current version; confirm
  `CHANGELOG.md`'s `[Unreleased]` section is empty and the tag matches the latest entry.
  **Verification Method:** Visual/Behavioral. **DoD:** tag present, `CHANGELOG.md`
  reconciled. **Design Refs:** N/A — as-built repository state.
- **`PROD-003` — Anti-Stub Final Sweep:** re-run the Maximal Implementation / Anti-Stub
  check (`AGENTS.md` §2.3) against the as-built repository; review the Open Items Register
  for any item that should already have been reached. **Verification Method:**
  Visual/Behavioral. **DoD:** sweep complete, no unresolved stub/TODO markers, Open Items
  Register clean. **Design Refs:** N/A — as-built repository state.
- **`PROD-004` — README:** see `DOC-FINAL` above; this item is satisfied by that task, not
  a separate one.
- **`PROD-005` — Spec/Dev-Risks Currency:** review the Architecture Specification against
  the as-built system; any divergence is fixed or logged as an Appendix F Spec Amendment
  (`agents/exemplars/architecture_specification_template.md` Appendix F); re-evaluate every
  Open entry in `dev/plan/[projectname]_dev_risks.md` against its own Re-evaluation Trigger.
  **Verification Method:** Visual/Behavioral. **DoD:** Spec/as-built divergence resolved or
  logged; dev-risks log current. **Design Refs:** Architecture Spec, full document.
- **`PROD-006` — License/Dependency Drift:** run `cargo deny check licenses` against the
  current `Cargo.lock`; confirm `THIRD_PARTY_LICENSES.md` matches. **Verification Method:**
  Build+Test. **DoD:** `cargo deny check licenses` clean, disclosure file matches. **Design
  Refs:** `agents/PREFERRED_DEPENDENCIES.md`.
- **`PROD-010` — Documentation Coverage Sweep:** re-run `cargo doc`'s missing-docs check
  (`metrics/docs_coverage.toml`, `agents/CI.md` Stage 1b) against the as-built repository;
  any crate not yet at 100% coverage is backfilled now rather than left standing — this is
  the phase-end backstop for the per-task boy-scout backfill rule (`AGENTS.md` §2.3), the
  same Anti-Stub-Final-Sweep shape as `PROD-003` applied to doc coverage instead of stub
  markers. **Verification Method:** Build+Test. **DoD:** every crate's
  `#![deny(missing_docs)]` passes clean, `metrics/docs_coverage.toml` shows 100% across the
  workspace. **Design Refs:** `AGENTS.md` §2.3.
- **`PROD-007` — Rollback Procedure** *(conditional — release/deploy step present)*:
  document and actually execute, once, in a non-production environment, the procedure for
  reverting to the previous tagged version; confirm migration reversibility if the project
  owns a schema. **Verification Method:** Hybrid. **DoD:** procedure documented, executed
  once, reversibility confirmed if applicable. **Design Refs:** N/A — as-built deployment
  process.
- **`PROD-008` — Operational Runbook** *(conditional — running/deployed service
  component present)*: document start/stop/restart, common failure symptoms and
  remediation, and backup/restore if the service holds persistent data. **Verification
  Method:** Visual/Behavioral. **DoD:** runbook document created and reviewed. **Design
  Refs:** N/A — as-built deployment process.
- **`PROD-009` — Monitoring Baseline** *(conditional — running/deployed service component
  present)*: confirm structured logs are actually emitted at runtime and include a version
  identifier. **Verification Method:** Visual/Behavioral. **DoD:** log output confirmed to
  include version identifier. **Design Refs:** `agents/PREFERRED_DEPENDENCIES.md` (`tracing`).

## 9. Test Strategy & Plan
Unit / Integration / System & Acceptance (for any UI/Tauri component: browser/device-target
matrix and headless-test harness) plans.

## 10. Logging Strategy
Incorporate the Architecture Specification's logging strategy and instrumentation tasks.

## 11. Session Handoff Protocol

**At session start, before any task:** (1) run §4's setup-check including the version
sanity check; (2) read the Checklist (`[projectname]_dev_checklist.md`, used **in place**,
never copied) and identify the first Task Group with any unchecked task; (3) run the project's
current build/test commands and confirm the result matches what the Checklist claims — a
discrepancy is itself an escalation, not something silently fixed; (4) confirm the Task Group's
Entry Criteria by inspecting repo state directly; (5) **for the identified Task Group's tasks,
run the task-state check against the Checklist and any WIP Checkpoint**
(`agents/DEVELOPMENT.md` §5.2 step 2 — there is no per-task submit any more, per `AGENTS.md`
§2.1): no Checklist box checked and no WIP Checkpoint → not started; a WIP Checkpoint exists
(only possible where Design specified one, §8) → resume in place from that checkpoint's own
description, never redo from scratch; Checklist box checked → done. A checked box with no
corresponding Verification File entry where one is required is an Escalation Trigger (§13),
never silently patched over. (6) work only within the identified
Task Group and the session's declared Session Unit (`AGENTS.md` §2.8) — never begin work outside
that scope, even with capacity remaining.

**At session end, on normal completion:** update the Checklist in place; leave every
component hermetically buildable even if the Task Group isn't fully done; produce/update the
Task Group Summary (§11.3); stop regardless of remaining capacity.

**On an unresolved Escalation Trigger (§13): do not reach normal session end.** Stop
immediately per §13's model instead.

### 11.3. Task Group Summary

**One file per Task Group, `dev/plan/[projectname]_task_group_[N]_summary.md`**, produced at Task Group close —
whether the Task Group succeeded, partially succeeded, or the session stopped on an escalation.
Required content:
- **Header:** Task Group ID/title, Build-Order step(s), date, executing session self-description,
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

**One file per Task Group, `test/v[N.NN.NN]/[projectname]_task_group_[N]_verification.md`, appended to in
place across every session that touches the Task Group** (never copied/renamed — same
in-place discipline as the Checklist, `AGENTS.md` §2.7). It is the concise evidence
receipt for the Task Group's task claims — distinct from the Task Group Summary (§11.3), which is
the narrative. The Task Group Summary links to this file rather than repeating its content.

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
- Each entry cites the artifact by filename, pointing into that Task Group's detail folder
  (below) — e.g. `Task AUTH_003: cargo nextest ... 6 passed. Screenshot:
  AUTH_003_login_success.png`.
- **Integration/System/Acceptance-type citations carry one additional clause, inline in the
  same entry, never a separate line:** which crates/components the implementing test
  actually wires together — e.g. `Task SIM_014: cargo nextest run test_integration_0031...
  1 passed. Wires: sim_core + sensors_adapter (via test_gt_actuation_integration binary,
  crate: tests/gt_actuation)`. This is the mechanical evidence trail Fix 3's Step 9 audit
  checks against the Test Case's own Design-time Type (Architecture Specification §3.2) and
  the Task Template's Test-Type-to-Naming Binding (§8) — a citation to an Integration-type
  Test Case backed by a single-crate test is then visible in the evidence trail itself, not
  only discoverable via a forensic extract after the fact. Still evidence-line format, not
  prose — this addition is a few words, not a paragraph.

**Detail folder — `test/v[N.NN.NN]/task_group_[N]/`:** holds the actual screenshots/clips named
`[TASK_ID]_[short_description].[ext]` (e.g. `AUTH_003_login_success.png`,
`UI_007_toast_animation_start.mp4`). Raw build/test logs are **not** retained here —
only the summary line goes in the Verification file itself, per the Mandatory Artifact
Preservation policy's non-reproducible-evidence scope (`AGENTS.md` §2.1). Committed, not
gitignored (§3 above).

## 12. Abort / Rollback Protocol

Treat a task/Task Group as aborted (not silently reworked) when: the chosen approach is found to
violate an Architecture Spec constraint; a DoR turns out false; completing as specified
would require modifying a file/contract outside its stated scope. **On abort:** revert
uncommitted changes for that task to its **last saved state** — its most recent
Design-specified WIP Checkpoint if one was reached for this task (`AGENTS.md` §2.1), or the
Task Group's own starting state (its last Final Wrap-Up Submit, or session start if this is
the Task Group's first session) if no WIP Checkpoint exists. **Note the changed blast
radius under the Task-Group-level submit cadence:** since individual tasks no longer have
their own Submit Point, an abort with no WIP Checkpoint to fall back to reverts to the whole
Task Group's last saved state, not just the aborted task — sibling tasks already completed
earlier in this same, not-yet-submitted Task Group are at risk from this task's abort in a
way they were not under the prior per-task cadence. This is a known, accepted consequence of
lifting the submit cadence (`AGENTS.md` §2.1), not an oversight; do not mark the aborted task
complete; record the abort, cause, and actual revert scope in the Task Group Summary.
**Per §13, an abort the agent cannot resolve itself now stops the entire session** — it does
not continue with other unaffected tasks in the Task Group. Rollback never crosses a Task Group
boundary except when root cause is a defect in already-completed earlier-Task Group work — that's
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
2. Write `dev/plan/[projectname]_task_group_[N]_summary.md` (§11.3) with full diagnostic detail: what was
   tried, exact failure output, and — if determinable — the (A)/(B) classification.
3. Leave the repository in its last clean, committed state. No PR, no further progress.
4. Stop. The human brings the Task Group Summary to a Design Phase session, which diagnoses the
   issue, updates the Architecture Spec and/or this Plan as needed, determines the correct
   restart Task Group, and restructures the Plan/Checklist. The human then rolls the repository
   back to the end of the last known-good Task Group and hands a fresh Development Phase session
   the updated documents to resume from there.

### 13.1. Escalation Requiring Design-Phase Re-Engagement

Distinct from an in-session trigger above: evidence across multiple Task Group Summaries
revealing a structural Plan/Spec problem, not one blocked task.
- **(A) Replanning** — Plan/Checklist needs to change (bad Task Group sizing, a broken
  Verification Method, an orphan citation, Checklist/Plan drift); Spec is sound.
- **(B) Re-architecting** — the Architecture Specification itself needs to change (a
  pattern of aborts traces to one architectural assumption; a chosen approach is
  structurally unworkable).

State which applies in the Task Group Summary and direct the human to a new Design Phase
session with the **full series of Task Group Summaries produced so far**, not just the
triggering one.

## 14. Change Control for This Plan

A Task Group's task list may be amended after work starts only when a later Task Group/task discovers
the original decomposition was wrong/incomplete. Record in `CHANGELOG.md`: what changed,
which Task Group/task, why. Never retroactively mark completed tasks differently than they were
actually completed. Task/Task Group IDs are never renumbered — new tasks get new IDs, deprecated
ones marked `[DEPRECATED]`. Changes spanning more than one Task Group, or changing Exit Criteria,
are "Major" and must be surfaced to the reviewer even without a Spec change. **The Checklist
and Dev Prompt are amended in place, never reproduced as a new file** (`AGENTS.md` §2.7).

**Ambiguity-resolving amendments.** Any amendment that resolves an ambiguity — where
already-completed Task Groups might not have satisfied the now-clarified intent — MUST add a
verification task to the next dependent, not-yet-started Task Group. That task explicitly checks
whether prior work satisfies the clarified requirement, corrects it if not, and confirms the
result via the standard DoD/Verification-Method pattern (§8). It never assumes the old work
happens to be compatible.

## 15. Plan-Level Definition of Done

- [ ] Every Core-status requirement ID (Spec §3) appears in ≥1 task's Traceability field.
- [ ] No task cites a requirement ID that doesn't exist (no orphan citations).
- [ ] Every Task Group (§6.1) has non-empty Entry and Exit Criteria.
- [ ] Every task (§8) has a non-empty Verification Method and ≥1 DoD checkbox.
- [ ] The Task Group Dependency Graph (§6.2) is acyclic and every Task Group reachable from Task Group 0.
- [ ] The Checklist contains exactly one line per task DoD item — no drift.
- [ ] §0's Architecture Cross-Reference lists every source document cited in §8.
- [ ] Every filename conforms to `CLAUDE.md` §4 (`[projectname]_dev_plan_NN_topic_v[N].md`,
  `[projectname]_dev_checklist.md`, `[projectname]_dev_prompt.md`).
- [ ] Every task (§8) has a non-empty Design Refs entry (or an explicit "trivial — no
  external refs needed" statement) and a stated `WIP-Checkpoint` value (`None` or a concrete
  point) — never a per-task Submit Point, which no longer exists (`AGENTS.md` §2.1).
- [ ] Every task whose Verification Method is Build+Test or Hybrid is split into `a`/`b`
  sub-tasks per §8's mandatory Code/Verify split rule; no such task remains unsplit.
- [ ] Every task whose DoD/File(s)-touched involves a new or modified `pub` item includes
  the rustdoc DoD checkbox (`AGENTS.md` §2.3) — no task drafted as touching a `pub` item
  with that checkbox silently omitted.
- [ ] Every Task Group (§6.1) has a stated Session Unit, consistent with its task composition
  (e.g. `Code+Verify` only where the Task Group is dominated by split tasks).
- [ ] Every Task Group (§6.1) shows a computed Complexity Score, recomputable from its own listed
  tasks; none over the §6 ceiling without a recorded override note in the same cell.
- [ ] The Final Task Group's task list includes one `PROD-NNN` task per applicable
  Productization Readiness Checklist item (`agents/DEVELOPMENT.md` §5.2.4), consistent
  with Design Step 5's Productization Applicability finding.
- [ ] Every Task Group (§6.1)'s Exit Criteria include running the full local CI-equivalent
  sequence and resolving/fixing any finding it surfaces (`agents/DEVELOPMENT.md` §5.2
  step 4), cited by reference, not restated in full per Task Group.
- [ ] Every Task Group (§6.1)'s Exit Criteria distinguish a license violation (stop-and-alert,
  blocks Exit) from new-crate/dependency drift since Task Group start (notification only,
  listed inline) per `agents/DEVELOPMENT.md` §5.2 step 4 — not merged into one undifferentiated
  Security Scan line item.
- [ ] Every task whose implementing test cites an Integration/System/Acceptance-type Test Case
  (Architecture Spec §3.2) states, in its DoD, which crates/components that test actually
  wires together, and the test's own name follows the `test_<type>_<nnnn>__description`
  convention (§8's Test-Type-to-Naming Binding) — no such task left to satisfy this DoD item
  by naming convention alone without the cross-crate/component claim also stated.
- [ ] The Build Order (§6) contains at least one Task Group whose Exit Criteria requires
  genuine multi-crate/cross-boundary execution (the Walking Skeleton milestone) — not deferred
  entirely to a single late capstone Task Group.
- [ ] Every component identified in the Architecture Specification has a Task Group in which
  it receives a minimal integration test at the point it reaches functional maturity (§6's
  Maturity-Triggered Component Integration), sized into that Task Group or, where it
  integrates with many others, its own dedicated following Task Group.
- [ ] Every task (§8) states a `WIP-Checkpoint` value — `None`, or a concrete stated
  checkpoint point — never a bare eligibility flag with the location left open; any task
  judged long/risky at drafting time has a stated point, not `None`.
- [ ] No task (§8) states its own per-task/per-sub-task Submit Point as a save point — the
  Session Unit's Submit Point occurs once, at the unit's own completion, per `AGENTS.md` §2.1.
- [ ] Every 5th Task Group by index (`agents/DEVELOPMENT.md` §5.2.5) is identified as an
  Integration-Reality Check point in the Task Group Index or Dependency Graph, so a Development
  Phase session can recognize it without cross-referencing the raw index count itself.

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
