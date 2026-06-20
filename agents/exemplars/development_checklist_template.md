# Development Checklist: [Project Name] — Complex/Multi-Phase Rust Project Variant

> Companion to `development_plan_template.md`. Use this checklist template whenever
> that Plan template is used — the two are designed to stay in lockstep (see Plan §15,
> Plan-Level Definition of Done: "exactly one checklist line per task DoD item, no drift").
>
> Differs from the base `development_checklist_template.md` by: (1) nesting each task's
> individual DoD sub-items as their own checkable lines instead of one line per task, since a
> single-session low-capability agent benefits from checking off "builds" separately from
> "tests pass" separately from "artifact captured" rather than inferring all three from one
> line; (2) adding an explicit phase Entry/Exit marker pair, so the checklist alone (without the
> Plan open alongside it) tells a fresh session whether it's safe to start a phase and how to
> know the phase is over; (3) adding a per-session log line so the Session Handoff Protocol
> (Plan §11) has somewhere to record itself without a separate file, for projects that prefer
> that over a standalone `SESSION_LOG.md`.

## How to Use This Checklist

- One `## Phase N: <Title>` section per Plan Phase, in the same order as the Plan's Phase Index
  (Plan §6.1).
- Each phase opens with an **Entry Criteria** checklist (copied from Plan §6.1) — a fresh
  session verifies every line here before touching any task, per Plan §11.
- Each task appears as a `### Task: <TASK-ID>` sub-section with its DoD items as individual
  checkboxes — not one collapsed line — mirroring Plan §8's DoD list exactly.
- Each phase closes with an **Exit Criteria** line — the single objective condition from Plan
  §6.1 that proves the phase is complete — checked only when every task above it is checked.
- `[ ]` = not done. `[x]` = done, DoD fully satisfied including required artifacts. There is no
  partial-credit state; if a task is partially done, it stays `[ ]` and the specifics belong in
  the session log (below) or `SESSION_LOG.md`, per Plan §11/§12.
- `[D]` = deliberately deprioritized/deferred out of current scope (e.g., maps to a
  `Status: Deferred` requirement) — distinct from incomplete. Only use `[D]` for tasks the Plan
  itself marks as deferred-scope; never use it to silently skip a Core task.

---

## Phase 0: [Title]

**Entry Criteria:**
- [ ] [copied verbatim from Plan §6.1]

### Task: [DOMAIN-001]
- [ ] Code implemented and hermetically builds (`[command]`)
- [ ] Verification Method checks pass (`[command]`)
- [ ] Test Case [ID] verified
- [ ] Required artifact captured: [artifact]

### Task: [DOMAIN-002]
- [ ] Code implemented and hermetically builds (`[command]`)
- [ ] Verification Method checks pass (`[command]`)
- [ ] Test Case [ID] verified
- [ ] Required artifact captured: [artifact]

**Exit Criteria:** [ ] [copied verbatim from Plan §6.1 — checked only when every task above is checked]

---

## Phase 1: [Title]

**Entry Criteria:**
- [ ] [...]

### Task: [DOMAIN-NNN]
- [ ] Code implemented and hermetically builds (`[command]`)
- [ ] Verification Method checks pass (`[command]`)
- [ ] Test Case [ID] verified
- [ ] Required artifact captured: [artifact]

**Exit Criteria:** [ ] [...]

---

*(repeat one `## Phase N` block per phase in Plan §6.1, in order)*

---

## Final Verification (Plan-Level Definition of Done, Plan §15)

- [ ] Every Core-status requirement ID defined in Architecture Specification §3 (per
  Plan §0's project note) appears in at least one task's Traceability field above
  (cross-check against the Plan's §8).
- [ ] No orphan requirement citations (every REQ-ID cited actually exists in Architecture
  Specification §3).
- [ ] Every phase above has both Entry Criteria and Exit Criteria checked, in order — no phase
  was started out of sequence without an explicit, escalated exception.
- [ ] Full workspace build is hermetic and green for every component: `[command(s)]`.
- [ ] Full test suite is green for every component: `[command(s)]`.
- [ ] Project-specific release/security gate (e.g., Architecture Spec's GA checklist) is
  satisfied: `[reference]`.

---

## Session Log

> Lightweight alternative to a standalone `SESSION_LOG.md` (Plan §11). Append one entry per
> session; never edit a prior entry except to fix a factual error, and note the fix as a new
> entry rather than a silent edit.

| Date | Phase(s) touched | Tasks completed | Tasks aborted (Plan §12) | Escalations raised (Plan §13) | Notes |
|---|---|---|---|---|---|
| YYYY-MM-DD | | | | | |
