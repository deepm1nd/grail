# Development Checklist: [Project Name] — Complex/Multi-Task-Group Rust Project Variant

> Companion to `development_plan_template.md` (Plan §15: exactly one checklist line per
> task DoD item, no drift). **Named `[projectname]_dev_checklist.md`, edited in place across
> the entire Development Phase, every session.** Per `AGENTS.md` §2.7, never copied, renamed,
> "v2'd," or reproduced. Structural changes (not just checkbox flips) are a Plan-Change
> Escalation (Plan §13/§14) — proposed, approved, edited in place, never forked.

## How to Use
- One `## Task Group N: <Title>` section per Plan Task Group, same order as Plan §6.1.
- Each Task Group opens with **Entry Criteria** (copied from Plan §6.1), verified before any task.
- Each task is a `### Task: <TASK-ID>` sub-section with DoD items as individual checkboxes.
- Each Task Group closes with an **Exit Criteria** line, checked only when every task above it is.
- `[ ]` not done · `[x]` done, DoD fully satisfied incl. artifacts · `[D]` deliberately
  deferred (only for Plan-marked-deferred tasks, never to silently skip a Core task).
- **A "Required artifact captured" item points at the Task Group's Verification file/folder** —
  `test/v[N.NN.NN]/[projectname]_task_group_[N]_verification.md` and `test/v[N.NN.NN]/task_group_[N]/`
  (`agents/exemplars/development_plan_template.md` §11.4) — not a description of the
  artifact inline in the Checklist.
- **A Development Phase session's edits to this file are bracket-content-only**
  (`AGENTS.md` §2.7): flipping a mark inside an existing `[ ]`, checking a `Submitted` box,
  or appending a new Session Log row. Rewording a task/DoD line, adding commentary next to a
  checkbox, inserting or restructuring content, or touching any other Task Group's marks are all
  out of scope for a Development Phase session — an apparent error in the wording itself is
  an Escalation Trigger, not a same-session fix.
- **No compressed formats** (`AGENTS.md` §2.3): the generated Checklist writes out every
  Task Group and every task in full, individually, in order — never a "repeat this block per
  Task Group" placeholder, an ellipsis standing in for omitted Task Groups/tasks, or any other
  shorthand. This template's own `## Task Group 1` block below is illustrative only; a real,
  delivered Checklist expands the entire Plan.
- **Continuous updates, in place** — each DoD sub-item checked the moment it's satisfied,
  not batched to end of task/Task Group.
- **README badges are static and CI-written** (`agents/DEVELOPMENT.md` §5.2,
  `README_template.md`'s Metrics & Badges section, `agents/CI.md` Stage 6) — there is no
  per-Task Group Branch-Name substitution DoD item any more; CI rewrites badge values on every
  push regardless of which Task Group or branch is active.
- **Task-Group-boundary scope rule (`agents/MAINTENANCE.md` §10, same principle applied here):**
  a session closing a Task Group's Exit Criteria checks **only that Task Group's own Exit Criteria**
  — never the next Task Group's Entry Criteria. The next Task Group is always opened in a new
  session, which checks its own Entry Criteria itself, at that time.
- **Tasks are worked in the order they appear in this Checklist, Task Group by Task Group, task by
  task.** A Development Phase session never reorders, skips ahead, or leaves a DoD sub-item
  unchecked-but-passed-over to move on — without the user's explicit permission given that
  session. An apparently unnecessary or already-satisfied task/item is a question or
  Escalation Trigger, never a silent skip.
- **`Submitted` checkbox — Task Group level only, not per-task/per-sub-task.** Per
  `AGENTS.md` §2.1, individual tasks and sub-tasks no longer carry their own Submit Point;
  the Task Group's own Exit Criteria line carries the single `Submitted` checkbox, checked
  only when the Task Group's Final Wrap-Up Submit (`agents/DEVELOPMENT.md` §5.2 step 7) has
  actually fired. A Task Group with all task DoD items checked but its Exit Criteria's
  `Submitted` box unchecked is an inconsistency, not a completed Task Group. A checked
  individual task DoD box, on its own, is never treated as evidence of a save point.
- **WIP-Checkpoint entries:** where a task's Plan §8 entry states a concrete `WIP-Checkpoint`
  point (not `None`), a `[ ] WIP-Checkpoint reached (if applicable)` line appears under that
  task, checked only once that specific point is actually reached and checkpointed — this is
  the sole mid-Task-Group save-point indicator; a task with `WIP-Checkpoint: None` in the Plan
  carries no such line at all.
- **Code/Verify split tasks (Plan §8):** rendered as two adjacent `### Task:` sub-sections,
  `[DOMAIN]-[NNN]a` (Code) and `[DOMAIN]-[NNN]b` (Verify), each with its own DoD checkboxes —
  neither carries its own `Submitted` checkbox any more; both are covered by the Task Group's
  single Exit-Criteria-level `Submitted` box.

---

## Task Group 0: [Title]

**Entry Criteria:**
- [ ] [copied verbatim from Plan §6.1]

### Task: DOC-001 — Review, confirm, and enhance project README.md
> The README itself is drafted at Design Step 8 (`agents/DESIGN.md` §5.8), not scaffolded
> here — this task reviews it against the repository as it starts to take shape.
- [ ] `README.md` (already drafted) reviewed against current repo state; overview, build/run
  instructions, and project structure confirmed accurate or corrected
- [ ] Content reviewed and approved by user

### Task: DOC-002 — Review, confirm, and extend CI workflow
> `ci.yml` itself is drafted at Design Step 8 from `agents/CI.md`'s skeleton, using Step 5's
> CI Stage Applicability findings — this task reviews/extends it against the repository as
> it starts to take shape (`agents/DEVELOPMENT.md` §5.2.2).
- [ ] `.github/workflows/ci.yml` (already drafted) reviewed against current repo state;
  conditional stages (WASM, Playwright/E2E, ESP32, infra services) confirmed correctly
  present or correctly absent
- [ ] Stage 0's `scripts/setup_env.sh` step confirmed to match the actual `setup_env.sh`
  content this Task Group produces or extends
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
- [ ] WIP-Checkpoint reached (if applicable — see Plan §8's stated point for this task)

### Task: [DOMAIN-002b] (Verify)
- [ ] Verification Method checks pass (`[command]`)
- [ ] Test Case [ID] verified
- [ ] Required artifact captured: [artifact]
- [ ] WIP-Checkpoint reached (if applicable — see Plan §8's stated point for this task)

**Exit Criteria:** [ ] [copied verbatim from Plan §6.1] · [ ] **Submitted** (Task Group's
sole Final Wrap-Up Submit, `AGENTS.md` §2.1 — the only Submit Point for this Task Group)

---

## Task Group 1: [Title]

**Entry Criteria:**
- [ ] [...]

### Task: [DOMAIN-NNN]
- [ ] Code implemented and hermetically builds (`[command]`)
- [ ] Verification Method checks pass (`[command]`)
- [ ] Test Case [ID] verified
- [ ] Required artifact captured: [artifact]

**Exit Criteria:**
- [ ] [copied verbatim from Plan §6.1]
- [ ] `scripts/run_ci.sh` run at Task Group level and output presented verbatim per
  `agents/CI.md` §6's mechanical contract (`agents/DEVELOPMENT.md` §5.2 step 4): [ ] Lint &
  Format · [ ] Build · [ ] Test · [ ] Coverage · [ ] Security Scan
- [ ] Security Scan's license check: no license violation found (a violation found blocks this
  Exit Criteria — Escalation Trigger, not a same-Task-Group fix)
- [ ] New-crate/dependency drift since Task Group start (if any) listed inline in the Task
  Group Summary — name/version/license — notification only, does not block this Exit Criteria
- [ ] **Submitted** (Task Group's sole Final Wrap-Up Submit, `AGENTS.md` §2.1)

---

*(This `## Task Group 1` block is this template's own illustrative placeholder — one example of
the shape a Task Group section takes. Per `AGENTS.md` §2.3's No Compressed Formats mandate, the
actual generated Checklist for a real project MUST NOT contain a "repeat this block"
instruction: it fully writes out one complete `## Task Group N` block per Task Group in Plan §6.1, in
order, each with every one of its real tasks and DoD items spelled out individually — never
collapsed, abbreviated, or left as a placeholder for the user/agent to expand later.)*

---

## Final Task Group: [Title]

**Entry Criteria:**
- [ ] [...]

### Task: DOC-FINAL — Final review of project README.md
- [ ] `README.md` reviewed for accuracy against the as-built system
- [ ] Any divergence from the Design-drafted / Task-Group-0-reviewed version corrected
- [ ] Content reviewed and approved by user

### Task: PROD-001 — Regression Traceability
- [ ] Every Core Requirement ID (Spec §3) has a row in
  `test/[projectname]_requirement_traceability.md`
- [ ] Every test named per the convention in `agents/exemplars/development_plan_template.md`
  §8's Test-Type-to-Naming Binding (`test_<type>_<nnnn>__description` for Rust — supersedes
  the bare `test_<nnnn>__description` form previously in `agents/MAINTENANCE.md` §6a, pending
  that file's own matching update; `TEST-<nnnn>: description` title for Playwright)
- [ ] Every listed test confirmed independently invocable by name/tag via
  `scripts/verify_traceability.js` (actually run, not assumed present)
- [ ] WIP-Checkpoint reached (if applicable — see Plan §8's stated point for this task)

### Task: PROD-002 — Versioning
- [ ] Repository tagged at the current version
- [ ] `CHANGELOG.md`'s `[Unreleased]` section empty; tag matches latest entry
- [ ] WIP-Checkpoint reached (if applicable — see Plan §8's stated point for this task)

### Task: PROD-003 — Anti-Stub Final Sweep
- [ ] Maximal Implementation / Anti-Stub check (`AGENTS.md` §2.3) re-run against as-built
  repository; no unresolved stub/TODO markers found
- [ ] Open Items Register reviewed; no item that should already have been reached remains
  unresolved
- [ ] WIP-Checkpoint reached (if applicable — see Plan §8's stated point for this task)

### Task: PROD-005 — Spec/Dev-Risks Currency
> `PROD-004` is satisfied by `DOC-FINAL` above — not a separate task.
- [ ] Architecture Specification reviewed against as-built system; any divergence fixed or
  logged as an Appendix F Spec Amendment
- [ ] Every Open entry in `dev/plan/[projectname]_dev_risks.md` re-evaluated against its own
  Re-evaluation Trigger
- [ ] WIP-Checkpoint reached (if applicable — see Plan §8's stated point for this task)

### Task: PROD-006 — License/Dependency Drift
- [ ] `cargo deny check licenses` run clean against current `Cargo.lock`
- [ ] `THIRD_PARTY_LICENSES.md` confirmed to match
- [ ] WIP-Checkpoint reached (if applicable — see Plan §8's stated point for this task)

### Task: PROD-010 — Documentation Coverage Sweep
- [ ] `cargo doc`'s missing-docs check re-run against as-built repository
  (`metrics/docs_coverage.toml`, `agents/CI.md` Stage 1b); any crate below 100% backfilled
- [ ] Every crate's `#![deny(missing_docs)]` passes clean
- [ ] WIP-Checkpoint reached (if applicable — see Plan §8's stated point for this task)

### Task: PROD-007 — Rollback Procedure (if applicable — release/deploy step present)
- [ ] Rollback-to-previous-tag procedure documented
- [ ] Procedure actually executed once, in a non-production environment
- [ ] Migration reversibility confirmed, if the project owns a schema
- [ ] WIP-Checkpoint reached (if applicable — see Plan §8's stated point for this task)

### Task: PROD-008 — Operational Runbook (if applicable — running/deployed service present)
- [ ] Start/stop/restart procedure documented
- [ ] Common failure symptoms and remediation documented
- [ ] Backup/restore procedure documented, if the service holds persistent data
- [ ] WIP-Checkpoint reached (if applicable — see Plan §8's stated point for this task)

### Task: PROD-009 — Monitoring Baseline (if applicable — running/deployed service present)
- [ ] Structured logging confirmed emitted at runtime
- [ ] Log output confirmed to include a version identifier
- [ ] WIP-Checkpoint reached (if applicable — see Plan §8's stated point for this task)

### Task: [DOMAIN-NNN]
- [ ] Code implemented and hermetically builds (`[command]`)
- [ ] Verification Method checks pass (`[command]`)
- [ ] Test Case [ID] verified
- [ ] Required artifact captured: [artifact]

**Exit Criteria:**
- [ ] [copied verbatim from Plan §6.1]
- [ ] `scripts/run_ci.sh` run at Task Group level and output presented verbatim per
  `agents/CI.md` §6's mechanical contract (`agents/DEVELOPMENT.md` §5.2 step 4): [ ] Lint &
  Format · [ ] Build · [ ] Test · [ ] Coverage · [ ] Security Scan
- [ ] Security Scan's license check: no license violation found (a violation found blocks this
  Exit Criteria — Escalation Trigger, not a same-Task-Group fix)
- [ ] New-crate/dependency drift since Task Group start (if any) listed inline in the Task
  Group Summary — name/version/license — notification only, does not block this Exit Criteria
- [ ] **Submitted** (Task Group's sole Final Wrap-Up Submit, `AGENTS.md` §2.1)

---

## Final Verification (Plan-Level Definition of Done, Plan §15)

- [ ] Every Core requirement ID (Spec §3) appears in ≥1 task's Traceability field above.
- [ ] No orphan requirement citations.
- [ ] Every Task Group above has Entry and Exit Criteria checked, in order.
- [ ] Full workspace build is hermetic and green: `[command(s)]`.
- [ ] Full test suite is green: `[command(s)]`.
- [ ] Every filename conforms to `CLAUDE.md` §4.
- [ ] Build Order contained at least one genuine multi-crate/cross-boundary Walking Skeleton
  Task Group (`development_plan_template.md` §6) — confirmed present, not merely assumed.
- [ ] Every Integration/System/Acceptance-type Test Case citation resolves to a
  `test_<type>_<nnnn>__description`-named test in a genuinely multi-crate binary (§8's
  Test-Type-to-Naming Binding) — not left unverified at project completion.
- [ ] No task in the completed Plan retains a per-task/per-sub-task Submit Point — Submitted
  is checked only at Task Group Exit Criteria, per `AGENTS.md` §2.1.

> **Note:** the checks above are local, session-run confirmations — not the same as a
> green CI run. CI (`agents/CI.md`) is an async, human-reviewed backstop; it is not a
> gate this Final Verification list waits on, and its own pass/fail status is reviewed
> separately, between sessions.

---

## Session Log

> One entry per session (Plan §11). Never edit a prior entry except to fix a factual error
> (log the fix as a new entry). For full evidence, see Task Group Summary files
> (`dev/plan/[projectname]_task_group_[N]_summary.md`).

| Date | Task Group(s) touched | Tasks completed | Tasks aborted | Escalation (Plan §13) | Notes |
|---|---|---|---|---|---|
| YYYY-MM-DD | | | | | |
