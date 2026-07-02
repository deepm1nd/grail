# Development Checklist: [Project Name] — Complex/Multi-Phase Rust Project Variant

> Companion to `agents/exemplars/development_plan_template.md`. Use this checklist template
> whenever that Plan template is used — the two are designed to stay in lockstep (see Plan §15,
> Plan-Level Definition of Done: "exactly one checklist line per task DoD item, no drift").
>
> **This file is named `[projectname]_dev_checklist.md` and is edited in place throughout the
> entire Development Phase** — across every session, by every agent. Per `AGENTS.md` §2.7, the
> executing agent is **STRICTLY FORBIDDEN** from creating a copy, a renamed version, a "v2," a
> session-specific duplicate, or any other one-off reproduction of this file. There is one
> checklist file, it lives in the repository, and it is the single source of truth for
> Development Phase progress. If an agent believes the checklist needs structural change (not
> just checkbox updates), that is a Plan-Change Escalation (Plan §13/§14) — proposed and
> approved, then edited in place — never silently forked into a new file.
>
> Differs from a simpler checklist by: (1) nesting each task's individual DoD sub-items as
> their own checkable lines instead of one line per task, since a single-session low-capability
> agent benefits from checking off "builds" separately from "tests pass" separately from
> "artifact captured" rather than inferring all three from one line; (2) adding an explicit
> phase Entry/Exit marker pair, so the checklist alone (without the Plan open alongside it)
> tells a fresh session whether it's safe to start a phase and how to know the phase is over;
> (3) adding a per-session log line so the Session Handoff Protocol (Plan §11) has somewhere to
> record itself without a separate file, for projects that prefer that over standalone Phase
> Summary files.

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
  the Session Log (below) or the Phase Summary file (`[projectname]_phaseN_summary.md`), per
  Plan §11/§12.
- `[D]` = deliberately deprioritized/deferred out of current scope (e.g., maps to a
  `Status: Deferred` requirement) — distinct from incomplete. Only use `[D]` for tasks the Plan
  itself marks as deferred-scope; never use it to silently skip a Core task.
- **Continuous updates, in place.** Each DoD sub-item is checked the moment it is actually
  satisfied — not batched until the end of a task or phase. If a session is interrupted, the
  checklist on disk must already reflect exactly what was completed, with no reconstruction
  needed.

---

## Phase 0: [Title]

**Entry Criteria:**
- [ ] [copied verbatim from Plan §6.1]

### Task: DOC-001 — Scaffold project README.md
- [ ] `README.md` created at repo root, covering overview, build/run instructions, and project structure (derived from Architecture Specification §1.2, §6, §9)
- [ ] Content reviewed and approved by user

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

*(The final phase must include a README review/finalization task — see Plan §6.1 note and
`agents/DEVELOPMENT.md` §5.2.1.)*

---

## Final Phase: [Title]

**Entry Criteria:**
- [ ] [...]

### Task: DOC-FINAL — Review and finalize project README.md
- [ ] `README.md` reviewed for accuracy against the as-built system (build instructions, features, configuration)
- [ ] Any divergence from Phase 0 scaffold corrected
- [ ] Content reviewed and approved by user

### Task: [DOMAIN-NNN]
- [ ] Code implemented and hermetically builds (`[command]`)
- [ ] Verification Method checks pass (`[command]`)
- [ ] Test Case [ID] verified
- [ ] Required artifact captured: [artifact]

**Exit Criteria:** [ ] [...]

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
- [ ] Every filename in this project conforms to the naming convention in `CLAUDE.md` §4.

---

## Session Log

> Lightweight per-session record (Plan §11). Append one entry per session; never edit a prior
> entry except to fix a factual error (note the fix as a new entry rather than a silent edit).
> For full evidence aggregation, see the Phase Summary files (`[projectname]_phaseN_summary.md`)
> produced at the close of each phase.

| Date | Phase(s) touched | Tasks completed | Tasks aborted (Plan §12) | Escalations raised (Plan §13) | Notes |
|---|---|---|---|---|---|
| YYYY-MM-DD | | | | | |
