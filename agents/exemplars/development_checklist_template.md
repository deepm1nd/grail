# Development Checklist: [Project Name] — Complex/Multi-Phase Rust Project Variant

> Companion to `development_plan_template.md` (Plan §15: exactly one checklist line per
> task DoD item, no drift). **Named `[projectname]_dev_checklist.md`, edited in place across
> the entire Development Phase, every session.** Per `AGENTS.md` §2.7, never copied, renamed,
> "v2'd," or reproduced. Structural changes (not just checkbox flips) are a Plan-Change
> Escalation (Plan §13/§14) — proposed, approved, edited in place, never forked.

## How to Use
- One `## Phase N: <Title>` section per Plan Phase, same order as Plan §6.1.
- Each phase opens with **Entry Criteria** (copied from Plan §6.1), verified before any task.
- Each task is a `### Task: <TASK-ID>` sub-section with DoD items as individual checkboxes.
- Each phase closes with an **Exit Criteria** line, checked only when every task above it is.
- `[ ]` not done · `[x]` done, DoD fully satisfied incl. artifacts · `[D]` deliberately
  deferred (only for Plan-marked-deferred tasks, never to silently skip a Core task).
- **A "Required artifact captured" item points at the phase's Verification file/folder** —
  `test/[projectname]_phase_[N]_verification.md` and `test/phase_[N]/`
  (`agents/exemplars/development_plan_template.md` §11.4) — not a description of the
  artifact inline in the Checklist.
- **A Development Phase session's edits to this file are bracket-content-only**
  (`AGENTS.md` §2.7): flipping a mark inside an existing `[ ]`, checking a `Submitted` box,
  or appending a new Session Log row. Rewording a task/DoD line, adding commentary next to a
  checkbox, inserting or restructuring content, or touching any other phase's marks are all
  out of scope for a Development Phase session — an apparent error in the wording itself is
  an Escalation Trigger, not a same-session fix.
- **No compressed formats** (`AGENTS.md` §2.3): the generated Checklist writes out every
  phase and every task in full, individually, in order — never a "repeat this block per
  phase" placeholder, an ellipsis standing in for omitted phases/tasks, or any other
  shorthand. This template's own `## Phase 1` block below is illustrative only; a real,
  delivered Checklist expands the entire Plan.
- **Continuous updates, in place** — each DoD sub-item checked the moment it's satisfied,
  not batched to end of task/phase.
- **Every phase's Documentation/QA task carries a recurring "README badge URLs point to
  this phase's Branch Name" DoD item** (`agents/DEVELOPMENT.md` §5.2, `README_template.md`'s
  Metrics & Badges note) — not just the first and final phases. Metrics commit to whichever
  branch CI ran on, so this is re-checked every phase, not a one-time Design-time
  substitution.
- **Tasks are worked in the order they appear in this Checklist, phase by phase, task by
  task.** A Development Phase session never reorders, skips ahead, or leaves a DoD sub-item
  unchecked-but-passed-over to move on — without the user's explicit permission given that
  session. An apparently unnecessary or already-satisfied task/item is a question or
  Escalation Trigger, never a silent skip.
- **Submitted** checkbox — every task/sub-task carries its own, checked only when its
  declared Submit Point (Plan §8) has actually fired via `submit`, not when DoD items are
  merely satisfied locally. A task with all DoD items checked but `Submitted` unchecked is
  an inconsistency, not a completed task.
- **Code/Verify split tasks (Plan §8):** rendered as two adjacent `### Task:` sub-sections,
  `[DOMAIN]-[NNN]a` (Code) and `[DOMAIN]-[NNN]b` (Verify), each with its own DoD checkboxes
  and its own `Submitted` checkbox — never merged into one task block.

---

## Phase 0: [Title]

**Entry Criteria:**
- [ ] [copied verbatim from Plan §6.1]

### Task: DOC-001 — Review, confirm, and enhance project README.md
> The README itself is drafted at Design Step 8 (`agents/DESIGN.md` §5.8), not scaffolded
> here — this task reviews it against the repository as it starts to take shape.
- [ ] `README.md` (already drafted) reviewed against current repo state; overview, build/run
  instructions, and project structure confirmed accurate or corrected
- [ ] Badge URLs' `[current_branch]` placeholder set to this phase's declared Branch Name;
  `[org]`/`[repo]` confirmed correctly substituted (not left as literal placeholders)
- [ ] Content reviewed and approved by user

### Task: DOC-002 — Review, confirm, and extend CI workflow
> `ci.yml` itself is drafted at Design Step 8 from `agents/CI.md`'s skeleton, using Step 5's
> CI Stage Applicability findings — this task reviews/extends it against the repository as
> it starts to take shape (`agents/DEVELOPMENT.md` §5.2.2).
- [ ] `.github/workflows/ci.yml` (already drafted) reviewed against current repo state;
  conditional stages (WASM, Playwright/E2E, ESP32, infra services) confirmed correctly
  present or correctly absent
- [ ] Stage 0's `scripts/setup_env.sh` step confirmed to match the actual `setup_env.sh`
  content this phase produces or extends
- [ ] Content reviewed and approved by user

### Task: DOC-003 — Reconcile third-party license disclosure
> `THIRD_PARTY_LICENSES.md`'s Design Step 8 draft is necessarily provisional — no
> `Cargo.lock` existed yet to check transitive dependencies against
> (`agents/DEVELOPMENT.md` §5.2.3). This task runs the first real check.
- [ ] `deny.toml` created (with `[licenses]` allow-list, `agents/PREFERRED_TOOLS.md`) —
  has no prerequisite of its own, created alongside the workspace scaffold
- [ ] `cargo deny check licenses` run against the actual resolved `Cargo.lock`
- [ ] Any violation found is an Escalation Trigger (Plan §13) — not a silent fix or
  `[patch]` workaround (`PREFERRED_DEPENDENCIES.md`'s No Local Patching mandate)
- [ ] `THIRD_PARTY_LICENSES.md` reconciled against the check's actual output
- [ ] Content reviewed and approved by user

### Task: DOC-004 — Confirm project LICENSE.md
> `LICENSE.md` is static text drafted at Design Step 8 — this is a simple accuracy check,
> not a reconciliation, since it has no dependency-tree prerequisite.
- [ ] `LICENSE.md` content confirmed correct (license text matches the project's chosen
  license, copyright holder/year correct)
- [ ] Content reviewed and approved by user

### Task: [DOMAIN-002a] (Code)
- [ ] Code implemented and hermetically builds (`[command]`)
- [ ] **Submitted** (task-complete Submit Point per Plan §8)

### Task: [DOMAIN-002b] (Verify)
- [ ] Verification Method checks pass (`[command]`)
- [ ] Test Case [ID] verified
- [ ] Required artifact captured: [artifact]
- [ ] **Submitted** (task-complete Submit Point per Plan §8)

**Exit Criteria:** [ ] [copied verbatim from Plan §6.1]

---

## Phase 1: [Title]

**Entry Criteria:**
- [ ] [...]

### Task: [DOMAIN-NNN]
- [ ] Code implemented and hermetically builds (`[command]`)
- [ ] Verification Method checks pass (`[command]`)
- [ ] Test Case [ID] verified
- [ ] Required artifact captured: [artifact]

### Task: DOC-PHASE-BADGE — Update README badge branch (every phase, recurring)
- [ ] Badge URLs' branch segment updated to this phase's declared Branch Name
- [ ] `[org]`/`[repo]` re-confirmed correctly substituted
- [ ] Content reviewed and approved by user

**Exit Criteria:** [ ] [...]

---

*(This `## Phase 1` block is this template's own illustrative placeholder — one example of
the shape a phase section takes. Per `AGENTS.md` §2.3's No Compressed Formats mandate, the
actual generated Checklist for a real project MUST NOT contain a "repeat this block"
instruction: it fully writes out one complete `## Phase N` block per phase in Plan §6.1, in
order, each with every one of its real tasks and DoD items spelled out individually — never
collapsed, abbreviated, or left as a placeholder for the user/agent to expand later.)*

---

## Final Phase: [Title]

**Entry Criteria:**
- [ ] [...]

### Task: DOC-FINAL — Final review of project README.md
- [ ] `README.md` reviewed for accuracy against the as-built system
- [ ] Any divergence from the Design-drafted / Phase-0-reviewed version corrected
- [ ] Badge URLs' branch segment updated to this (final) phase's declared Branch Name;
  `[org]`/`[repo]` re-confirmed correctly substituted
- [ ] Content reviewed and approved by user

### Task: PROD-001 — Regression Traceability
- [ ] Every Core Requirement ID (Spec §3) has a row in
  `test/[projectname]_requirement_traceability.md`
- [ ] Every listed test confirmed independently invocable by name/tag (actually run, not
  assumed present)
- [ ] **Submitted** (task-complete Submit Point per Plan §8)

### Task: PROD-002 — Versioning
- [ ] Repository tagged at the current version
- [ ] `CHANGELOG.md`'s `[Unreleased]` section empty; tag matches latest entry
- [ ] **Submitted** (task-complete Submit Point per Plan §8)

### Task: PROD-003 — Anti-Stub Final Sweep
- [ ] Maximal Implementation / Anti-Stub check (`AGENTS.md` §2.3) re-run against as-built
  repository; no unresolved stub/TODO markers found
- [ ] Open Items Register reviewed; no item that should already have been reached remains
  unresolved
- [ ] **Submitted** (task-complete Submit Point per Plan §8)

### Task: PROD-005 — Spec/Dev-Risks Currency
> `PROD-004` is satisfied by `DOC-FINAL` above — not a separate task.
- [ ] Architecture Specification reviewed against as-built system; any divergence fixed or
  logged as an Appendix F Spec Amendment
- [ ] Every Open entry in `docs/[project_name]_dev_risks.md` re-evaluated against its own
  Re-evaluation Trigger
- [ ] **Submitted** (task-complete Submit Point per Plan §8)

### Task: PROD-006 — License/Dependency Drift
- [ ] `cargo deny check licenses` run clean against current `Cargo.lock`
- [ ] `THIRD_PARTY_LICENSES.md` confirmed to match
- [ ] **Submitted** (task-complete Submit Point per Plan §8)

### Task: PROD-007 — Rollback Procedure (if applicable — release/deploy step present)
- [ ] Rollback-to-previous-tag procedure documented
- [ ] Procedure actually executed once, in a non-production environment
- [ ] Migration reversibility confirmed, if the project owns a schema
- [ ] **Submitted** (task-complete Submit Point per Plan §8)

### Task: PROD-008 — Operational Runbook (if applicable — running/deployed service present)
- [ ] Start/stop/restart procedure documented
- [ ] Common failure symptoms and remediation documented
- [ ] Backup/restore procedure documented, if the service holds persistent data
- [ ] **Submitted** (task-complete Submit Point per Plan §8)

### Task: PROD-009 — Monitoring Baseline (if applicable — running/deployed service present)
- [ ] Structured logging confirmed emitted at runtime
- [ ] Log output confirmed to include a version identifier
- [ ] **Submitted** (task-complete Submit Point per Plan §8)

### Task: [DOMAIN-NNN]
- [ ] Code implemented and hermetically builds (`[command]`)
- [ ] Verification Method checks pass (`[command]`)
- [ ] Test Case [ID] verified
- [ ] Required artifact captured: [artifact]

**Exit Criteria:** [ ] [...]

---

## Final Verification (Plan-Level Definition of Done, Plan §15)

- [ ] Every Core requirement ID (Spec §3) appears in ≥1 task's Traceability field above.
- [ ] No orphan requirement citations.
- [ ] Every phase above has Entry and Exit Criteria checked, in order.
- [ ] Full workspace build is hermetic and green: `[command(s)]`.
- [ ] Full test suite is green: `[command(s)]`.
- [ ] Every filename conforms to `CLAUDE.md` §4.

> **Note:** the checks above are local, session-run confirmations — not the same as a
> green CI run. CI (`agents/CI.md`) is an async, human-reviewed backstop; it is not a
> gate this Final Verification list waits on, and its own pass/fail status is reviewed
> separately, between sessions.

---

## Session Log

> One entry per session (Plan §11). Never edit a prior entry except to fix a factual error
> (log the fix as a new entry). For full evidence, see Phase Summary files
> (`[projectname]_phaseN_summary.md`).

| Date | Phase(s) touched | Tasks completed | Tasks aborted | Escalation (Plan §13) | Notes |
|---|---|---|---|---|---|
| YYYY-MM-DD | | | | | |
