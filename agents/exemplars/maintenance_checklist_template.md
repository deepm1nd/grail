# Maintenance Checklist: [projectname]_v[N.NN.NN]_checklist — Batch Variant

> Companion to `maintenance_batch_template.md` and `agents/MAINTENANCE.md`. Lives at
> `dev/maintenance/v[N.NN.NN]/[projectname]_v[N.NN.NN]_checklist.md` — same folder-plus-
> filename version redundancy as the batch file, for the same provenance reason. **Named
> `[projectname]_v[N.NN.NN]_checklist.md` from the moment the batch opens** — batch
> composition is fixed at open time (`agents/MAINTENANCE.md` §5), so the target version is
> known immediately and there is no `_open` placeholder stage or release-time rename
> (`agents/MAINTENANCE.md` §8/§11), except a rare escalation-driven mid-batch correction.
> Edited in place across the batch's lifetime. Dedicated to Maintenance Phase — mirrors
> `development_checklist_template.md`'s Task Group/Task/DoD/Submit-Point/Session-Log shape
> exactly, but `development_checklist_template.md` itself is never modified or reused to
> serve this purpose, so Development Phase's own proven pipeline is unaffected.

## How to Use
- **One `## Task Group [ID]: <Title>` section per Maintenance Batch item**, same ID and order
  as the batch file (`maintenance_batch_template.md`) — no Task Group 0 scaffold section, no
  Final Task Group Productization section; a Maintenance batch has neither.
- Each Task Group opens with **Entry Criteria** (from the item's M0/M2 content — or, for a
  lightweight-path item, from M1's exit directly) and closes with **Exit Criteria** =
  Verified (`agents/MAINTENANCE.md` §10).
- **Task-Group-boundary scope rule (`agents/MAINTENANCE.md` §10):** a session closing a Task Group's
  Exit Criteria checks **only that Task Group's own Exit Criteria** — never the next Task Group's
  Entry Criteria. The next Task Group is always opened in a new session, which checks its own
  Entry Criteria itself at that time; this is not a redundant check to skip, it is simply
  not this session's job to do early.
- Each task is a `### Task: <TASK-ID>` sub-section with DoD items as individual
  checkboxes — same Task Template shape as Development (Design Refs, Verification Method,
  DoD, WIP-Checkpoint-if-Design-specified).
- `[ ]` not done · `[x]` done, DoD fully satisfied · `[D]` deliberately deferred (only for
  an item explicitly carried over to the next batch, never to silently skip).
- **Edits to this file during a Maintenance session are bracket-content-only**: flipping a
  mark, checking the Task Group's `Submitted` box, appending a Session Log row. Rewording a
  task/DoD line or restructuring content is an Escalation Trigger, not a same-session fix.
- **No compressed formats** — every Task Group and task is written out in full, individually,
  in order. This template's own `## Task Group BF-0001` block below is illustrative only.
- **Continuous updates, in place** — each DoD sub-item checked the moment it's satisfied.
- **Tasks are worked in the order they appear**, Task Group by Task Group, task by task — no
  reordering or skipping without explicit user permission that session.
- **`Submitted` checkbox — one per Task Group, not one per task**, matching
  `AGENTS.md` §2.1's cadence: no per-task or per-sub-task Submit Point exists any more. It's
  checked only when the Task Group's own Final Wrap-Up Submit has actually fired via
  `submit` and the user has responded "Continue"/"Proceed."
- **Code/Verify split tasks:** two adjacent `### Task:` sub-sections, `[ID]a` (Code) and
  `[ID]b` (Verify) — neither carries its own `Submitted` checkbox; both fold into the same
  Task Group `Submitted` checkbox as every other task.

---

## Task Group BF-0001: [short title]

**Entry Criteria:**
- [ ] [from the item's M1/M2 content in the batch file]

### Task: [ID]-001
- [ ] Code implemented and hermetically builds (`[command]`)
- [ ] Verification Method checks pass (`[command]`)
- [ ] Test Case [ID] verified
- [ ] Required artifact captured: [artifact]

**Exit Criteria:**
- [ ] Verified per `agents/MAINTENANCE.md` §10
- [ ] `scripts/run_ci.sh` run and output presented verbatim per `agents/CI.md` §6's
  mechanical contract
**Submitted:** [ ] (Task Group's Final Wrap-Up Submit — the sole submit for this Task Group)

---

*(This `## Task Group BF-0001` block is illustrative only — the actual generated Checklist for
a real batch fully writes out one `## Task Group [ID]` block per item in the batch file, in
order, each with every real task and DoD item spelled out individually, never collapsed
or left as a placeholder.)*

---

## Batch Verification (Release Checklist)

See `agents/MAINTENANCE.md` §11 for the actual patch/minor/major Release Checklist —
completed once, at release time, referencing every Task Group above. Not restated here. Since
this file's own name already carries its real target version (assigned at batch-open),
release is a confirmation pass over the checklist below, not a rename pass — absent
`MAINTENANCE.md` §8's sole mid-batch-correction exception.

---

## Session Log

> One entry per session. Never edit a prior entry except to fix a factual error (log the
> fix as a new entry).

| Date | Item(s) touched | Tasks completed | Tasks aborted | Escalation (`MAINTENANCE.md` §12) | Notes |
|---|---|---|---|---|---|
| YYYY-MM-DD | | | | | |
