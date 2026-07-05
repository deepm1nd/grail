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
- **Continuous updates, in place** — each DoD sub-item checked the moment it's satisfied,
  not batched to end of task/phase.

---

## Phase 0: [Title]

**Entry Criteria:**
- [ ] [copied verbatim from Plan §6.1]

### Task: DOC-001 — Review, confirm, and enhance project README.md
> The README itself is drafted at Design Step 8 (`agents/DESIGN.md` §5.8), not scaffolded
> here — this task reviews it against the repository as it starts to take shape.
- [ ] `README.md` (already drafted) reviewed against current repo state; overview, build/run
  instructions, and project structure confirmed accurate or corrected
- [ ] Content reviewed and approved by user

### Task: [DOMAIN-002]
- [ ] Code implemented and hermetically builds (`[command]`)
- [ ] Verification Method checks pass (`[command]`)
- [ ] Test Case [ID] verified
- [ ] Required artifact captured: [artifact]

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

**Exit Criteria:** [ ] [...]

---

*(repeat one `## Phase N` block per phase in Plan §6.1, in order)*

---

## Final Phase: [Title]

**Entry Criteria:**
- [ ] [...]

### Task: DOC-FINAL — Final review of project README.md
- [ ] `README.md` reviewed for accuracy against the as-built system
- [ ] Any divergence from the Design-drafted / Phase-0-reviewed version corrected
- [ ] Content reviewed and approved by user

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

---

## Session Log

> One entry per session (Plan §11). Never edit a prior entry except to fix a factual error
> (log the fix as a new entry). For full evidence, see Phase Summary files
> (`[projectname]_phaseN_summary.md`).

| Date | Phase(s) touched | Tasks completed | Tasks aborted | Escalation (Plan §13) | Notes |
|---|---|---|---|---|---|
| YYYY-MM-DD | | | | | |
