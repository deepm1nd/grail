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
Triggers · 14. Change Control · 15. Plan-Level Definition of Done · Appendix G Glossary

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

Workspace root tree: `Cargo.toml`, `libs/`/`crates/`/`services/`, frontend/mobile dirs,
`deploy/` (compose files), `assets/` (below), `scripts/` (`setup_env.sh`/`.bat`).

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

**`.gitignore` — required file, Phase 0 responsibility.** Every project MUST have a
`.gitignore` at the repository root. The following entries are mandatory for any Rust
project under this methodology; additional project-specific entries are added as needed:

```gitignore
# Cargo build outputs — all profiles (debug, release, test, bench, doc)
/target

# Trunk WASM build output (remove if no WASM/Trunk component)
/dist

# Local environment — real credentials never committed; commit .env.example instead
.env

# IDE and OS noise (extend as needed for the team's tooling)
.DS_Store
.idea/
.vscode/
```

Per `AGENTS.md` §2.1, non-reproducible evidence artifacts (`test_outs/`, screenshots,
coverage reports) are **not** gitignored — they are committed, as they cannot be recreated
after a session crash. `/target` and `/dist` are reproducible from source and MUST be
gitignored, not committed.

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
project-tunable. Score above ceiling → split further. **Independent of sizing: a session
completes at most one phase**, regardless of remaining capacity (`AGENTS.md` §2.8).

**Frontend Targeted Interleaving:** where a UI exists, each screen/component's frontend
task is placed in the **same phase** as the real (non-mock) backend/data source it depends
on — never earlier (forces a stub) and never batched into a trailing frontend-only phase.

### 6.1. Phase Index

| Phase | Build-Order Step(s) | Component(s) | Title | Entry Criteria | Exit Criteria | Req Domains | Task Count |
|---|---|---|---|---|---|---|---|

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
- **Commands**
- **DoD:** code builds (`command`) · verification passes (`command(s)`) · Test Case ID
  verified · Required Artifacts captured
- **Traceability:** REQ-XXX-NNN[, ...] — every task cites at least one requirement
- **Context for executing agent:** the one or two sentences resolving the likeliest
  ambiguity a lower-context agent would get wrong

**README task (final phase):** `DOC-FINAL` — review the Design-drafted `README.md` against
the as-built system, correct any divergence, get user approval. **DoD:** N/A build command;
DoD is content review and user approval.

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
Entry Criteria by inspecting repo state directly; (5) work only within the identified
phase — never begin a later phase's tasks, even with capacity remaining.

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

## 12. Abort / Rollback Protocol

Treat a task/phase as aborted (not silently reworked) when: the chosen approach is found to
violate an Architecture Spec constraint; a DoR turns out false; completing as specified
would require modifying a file/contract outside its stated scope. **On abort:** revert
uncommitted changes for that task to last known-good; do not mark it complete; record the
abort and cause in the Phase Summary. **Per §13, an abort the agent cannot resolve itself
now stops the entire session** — it does not continue with other unaffected tasks in the
phase. Rollback never crosses a phase boundary except when root cause is a defect in
already-completed earlier-phase work — that's an Escalation Trigger (§13), not a unilateral
rollback.

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

## Appendix G — Glossary

| Term | Meaning |
|---|---|
| DoD / DoR | Definition of Done / Ready |
| [project-specific terms] | |

---
See `CHANGELOG.md` for this file's full version history.
