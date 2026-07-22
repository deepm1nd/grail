# Maintenance Checklist: [projectname]_maintenance_checklist_open — Batch Variant

> Companion to `maintenance_batch_template.md` and `agents/MAINTENANCE.md`. **Named
> `[projectname]_maintenance_checklist_open.md` while the batch is open, renamed to
> `[projectname]_v[N.NN.NN]_checklist.md` at release** (`agents/MAINTENANCE.md` §8/§11),
> edited in place across the batch's lifetime. Dedicated to Maintenance Phase — mirrors
> `development_checklist_template.md`'s Phase/Task/DoD/Submit-Point/Session-Log shape
> exactly, but `development_checklist_template.md` itself is never modified or reused to
> serve this purpose, so Development Phase's own proven pipeline is unaffected.

## How to Use
- **One `## Phase [ID]: <Title>` section per Maintenance Batch item**, same ID and order
  as the batch file (`maintenance_batch_template.md`) — no Phase 0 scaffold section, no
  Final Phase Productization section; a Maintenance batch has neither.
- Each phase opens with **Entry Criteria** (from the item's M1/M2 content) and closes with
  **Exit Criteria** = Verified (`agents/MAINTENANCE.md` §10).
- Each task is a `### Task: <TASK-ID>` sub-section with DoD items as individual
  checkboxes — same Task Template shape as Development (Design Refs, Verification Method,
  DoD, Submit Point).
- `[ ]` not done · `[x]` done, DoD fully satisfied · `[D]` deliberately deferred (only for
  an item explicitly carried over to the next batch, never to silently skip).
- **Edits to this file during a Maintenance session are bracket-content-only**: flipping a
  mark, checking a `Submitted` box, appending a Session Log row. Rewording a task/DoD
  line or restructuring content is an Escalation Trigger, not a same-session fix.
- **No compressed formats** — every phase and task is written out in full, individually,
  in order. This template's own `## Phase BF-0001` block below is illustrative only.
- **Continuous updates, in place** — each DoD sub-item checked the moment it's satisfied.
- **Tasks are worked in the order they appear**, phase by phase, task by task — no
  reordering or skipping without explicit user permission that session.
- **Submitted** checkbox — every task carries its own, checked only when its Submit Point
  has actually fired via `submit`.
- **Code/Verify split tasks:** two adjacent `### Task:` sub-sections, `[ID]a` (Code) and
  `[ID]b` (Verify), each with its own DoD and `Submitted` checkbox.

---

## Phase BF-0001: [short title]

**Entry Criteria:**
- [ ] [from the item's M1/M2 content in the batch file]

### Task: [ID]-001
- [ ] Code implemented and hermetically builds (`[command]`)
- [ ] Verification Method checks pass (`[command]`)
- [ ] Test Case [ID] verified
- [ ] Required artifact captured: [artifact]
- [ ] **Submitted**

**Exit Criteria:** [ ] Verified per `agents/MAINTENANCE.md` §10

---

*(This `## Phase BF-0001` block is illustrative only — the actual generated Checklist for
a real batch fully writes out one `## Phase [ID]` block per item in the batch file, in
order, each with every real task and DoD item spelled out individually, never collapsed
or left as a placeholder.)*

---

## Batch Verification (Release Checklist)

See `agents/MAINTENANCE.md` §11 for the actual patch/minor/major Release Checklist —
completed once, at release time, referencing every Phase above. Not restated here.

---

## Session Log

> One entry per session. Never edit a prior entry except to fix a factual error (log the
> fix as a new entry).

| Date | Item(s) touched | Tasks completed | Tasks aborted | Escalation (`MAINTENANCE.md` §12) | Notes |
|---|---|---|---|---|---|
| YYYY-MM-DD | | | | | |
