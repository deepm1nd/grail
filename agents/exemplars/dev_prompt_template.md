# Development-Agent Prompt — Template

> Produced once at Design Step 8, saved as `[projectname]_dev_prompt.md`. Reused verbatim
> at the start of every Development-Phase session — not regenerated per session. If the
> Plan/Checklist changes materially, amend this file once, in place (`AGENTS.md` §2.7).
>
> Fill in `[PROJECT_NAME]`, `[projectname]`, and the bracketed paths to match actual
> delivered filenames. Architecture Spec and Dev Plan files carry a `_v[N]` suffix
> (`CLAUDE.md` §4); the Checklist and this Prompt do not, since both are edited in place.

---

## Prompt Text (give this to the development agent at the start of every session)

You are continuing development work on **[PROJECT_NAME]**, a Rust project, operating in the
**Development Phase** (`agents/DEVELOPMENT.md`) — not Design Phase. Read `agents/DEVELOPMENT.md`
in full if not already internalized this session. Fresh session, no memory of any prior one.

**Code Review Policy** is governed by `AGENTS.md` §2.2 (bypassed by default — do not call
`request_code_review` unless that section states otherwise). **Tool tiers and submit
mechanics** are governed by `agents/AGENT_TOOL_POLICY.md`. **After every `submit`, stop and
wait — the user will say "Continue" or "Proceed" to resume; this is normal, expected flow at
every declared Submit Point, not an error.** **Session Unit** for this session's phase is
stated in the Checklist/Plan §6.1 (`Phase`, `Task`, or `Code+Verify`) — work only within
that scope, never beyond it, regardless of remaining capacity.

### 1. Read docs — mandatory upfront, then on demand only

**CRITICAL CONTEXT WINDOW RULES — read and follow before opening any file:**
- **No broad repository scan.** Do not enumerate, list, or speculatively read directories or
  files. Go directly to the specific paths named below and nowhere else.
- **No whole-file reads of large documents.** Use targeted extraction only:
  `sed -n 'START,ENDp' file` for line ranges; `grep -n "pattern" -A N file` for
  sections by heading or ID; `awk '/^## Section/,/^## /' file` for heading-bounded blocks.
  The only exception is `[projectname]_dev_plan_04_protocols_and_dod_v[N].md` — read it in
  full (it is short by design).
- **Architecture Spec and non-protocols plan files are NOT read upfront.** They are
  referenced on demand, only when a specific uncertainty arises during task work. The
  reference table below tells you exactly which file and section to target, and how.

**Read now (mandatory, in this order):**
1. `[projectname]_dev_checklist.md` — your primary working document. **Edit in place only.**
   Never copy/rename/version it. This is the *only* doc file you may edit, and only within
   your current, identified Phase (flip DoD/task boxes, append a Session Log row) — every
   other file is read-only; an apparent error in one is an Escalation Trigger, never a
   same-session fix.
2. Every existing `[projectname]_phaseN_summary.md`, in phase order — what actually happened
   in completed phases, not just what the Plan intended. Use `ls [projectname]_phase*_summary.md`
   to find them; read each with targeted extraction or in full if short.
3. `[projectname]_dev_plan_02_environment_and_phases_v[N].md` §6.1 — Phase Index only.
   Extract with: `grep -n "Phase Index\|^| " [projectname]_dev_plan_02_environment_and_phases_v[N].md | head -60`
4. `[projectname]_dev_plan_03_tasks_and_testing_v[N].md` §8 — current phase's tasks only.
   Extract with: `grep -n "Phase [N]\|Task\|TASK-ID\|DoR\|DoD\|Verification" [projectname]_dev_plan_03_tasks_and_testing_v[N].md -A 8`
   (replace `Phase [N]` with the actual phase identifier from the checklist).
5. `[projectname]_dev_plan_04_protocols_and_dod_v[N].md` — read in full. This is the
   Session Handoff, Abort/Rollback, and Escalation protocols. Not optional — it tells you
   what to do when things go wrong.
6. If a UI exists: verify the Asset Manifest in `[projectname]_architecture_06_viewpoints_v[N].md`
   §4.13 is satisfied by `assets/{html,images,audio,video}/`. Extract with:
   `grep -n "Asset Manifest\|Filename\|assets/" [projectname]_architecture_06_viewpoints_v[N].md -A 3`
   A missing named asset is an Escalation Trigger.

### 2. On-demand reference table — consult ONLY when uncertainty arises

**Do not read any of these upfront.** When a task produces confusion, a conflict, missing
clarity, or any abnormality — stop, identify the uncertainty type from the table, extract
only the named section using the method shown, resolve it, then continue.

| Uncertainty type | File | Section | Extraction method |
|---|---|---|---|
| Interface contract, API signature, endpoint schema | `[projectname]_architecture_07_interfaces_and_stack_v[N].md` | §5 External Interfaces, §4.10 ICD | `grep -n "interface-name\|REQ-ID\|endpoint" FILE -A 20` |
| Data model, field type, struct layout, constraints | `[projectname]_architecture_06_viewpoints_v[N].md` | §4.5 Data Dictionary | `grep -n "EntityName\|field-name" FILE -A 15` |
| Requirement wording, acceptance criteria | `[projectname]_architecture_03_requirements_v[N].md` | §2.3–2.4 | `grep -n "REQ-ID" FILE -A 10` |
| Test case definition, DoD for a requirement | `[projectname]_architecture_04_test_strategy_v[N].md` | §3.2 Test Case Catalog | `grep -n "TEST-ID\|Verifies" FILE -A 10` |
| Traceability — which component covers a requirement | `[projectname]_architecture_05_verified_traceability_v[N].md` | §3.4 Traceability Matrix | `grep -n "REQ-ID\|Component" FILE -A 5` |
| Concurrency model, ownership, Send/Sync boundary | `[projectname]_architecture_06_viewpoints_v[N].md` | §4.7 Process View, §4.9 Rust Conventions | `awk '/^### 4\.7/,/^### 4\.8/' FILE` |
| Security constraint, threat model, trust boundary | `[projectname]_architecture_06_viewpoints_v[N].md` | §4.8 Security | `grep -n "THREAT-ID\|trust boundary\|mitigation" FILE -A 10` |
| Technology choice, crate version, MSRV | `[projectname]_architecture_07_interfaces_and_stack_v[N].md` | §6 Technology Stack | `grep -n "crate-name\|MSRV\|edition" FILE -A 5` |
| Build order, phase dependency, sequencing | `[projectname]_architecture_08_constraints_and_roadmap_v[N].md` | §9.2 Build Order | `grep -n "Step\|Phase [0-9]" FILE -A 5` |
| Architectural risk, known constraint, assumption | `[projectname]_architecture_08_constraints_and_roadmap_v[N].md` | §7 Constraints, §8 Risks | `grep -n "RISK-ID\|constraint-keyword" FILE -A 8` |
| User story intent, persona, interaction sequence | `[projectname]_architecture_02_user_stories_v[N].md` | §2.2 User Stories | `grep -n "US-ID\|persona-name" FILE -A 15` |
| System overview, product goals, problem framing | `[projectname]_architecture_01_introduction_v[N].md` | §1.2–1.4 | `awk '/^## 1\.2/,/^## 1\.5/' FILE` |
| Deployment topology, WASM hosting, COOP/COEP | `[projectname]_architecture_06_viewpoints_v[N].md` | §4.6 Deployment View | `awk '/^### 4\.6/,/^### 4\.7/' FILE` |
| Dev plan: risk management | `[projectname]_dev_plan_03_tasks_and_testing_v[N].md` | §7 Risk Management | `awk '/^## 7\./,/^## 8\./' FILE` |
| Dev plan: test/logging strategy | `[projectname]_dev_plan_03_tasks_and_testing_v[N].md` | §9–10 | `awk '/^## 9\./,/^## 11\./' FILE` |
| Dev plan: environment, prerequisites, config | `[projectname]_dev_plan_02_environment_and_phases_v[N].md` | §4–5 | `awk '/^## 4\./,/^## 6\./' FILE` |
| Dev plan: technology stack | `[projectname]_dev_plan_01_overview_v[N].md` | §2 | `awk '/^## 2\./,/^## 3\./' FILE` |

### 3. Environment check and version sanity check
```bash
bash scripts/setup_env.sh    # or scripts/check_env.sh if present
```
If absent, manually verify each Plan §4 prerequisite. Per tool: present+correct → proceed;
**missing or wrong version, install succeeds cleanly** → append idempotently to
`scripts/setup_env.sh`/`.bat`, inform the user, proceed; **install fails, or any resulting
conflict/incompatibility surfaces → stop the session now** (§4 below).

### 4. Start infrastructure services (if required)
```bash
docker compose -f deploy/docker-compose.dev.yml up -d
docker compose -f deploy/docker-compose.dev.yml ps
```
Docker unavailable → treat as a missing prerequisite (step 3's rule).

### 5. Verify repository state before touching any code
Run the project's actual build/test commands (Plan §2/§4). **If the Checklist claims a
phase is complete but either fails: stop the session now** — write the Phase Summary
(step 8) describing the discrepancy and stop. Do not silently fix and continue.

### 6. Find the next unit
In the Checklist, find the first phase whose Exit Criteria isn't checked. Verify its Entry
Criteria are actually true from current repo state — not assumed from the Checklist alone.
Unverifiable → stop the session now.

**Then run the three-way task-state check** against each task/sub-task's declared Submit
Point (Plan §8) — not raw git-log archaeology:
- **No submit exists** → not started; begin fresh.
- **A `[WIP-CHECKPOINT]` submit exists, no task-complete submit** → resume **in place** from
  exactly what that checkpoint's own description states was attempted, confirmed working,
  and known incomplete. Never redo from scratch, never discard it. If the description is too
  vague to resume from, that's a defect to flag, not something to guess past.
- **A task-complete submit exists** → done; move to the next task/sub-task.

A Checklist mark with no matching submit, or a submit with no matching Checklist mark, is an
inconsistency — go to step 8 now, do not silently reconcile it yourself.

Work only within the phase's declared **Session Unit** (`Phase` / `Task` / `Code+Verify`) —
never begin work outside that scope even with capacity remaining.

### 7. Work the unit, one task (or sub-task) at a time
Confirm each task's DoR before starting. Code in `src\`, never `docs\`. **Check off DoD
sub-items the moment each is satisfied** — continuously, not batched. Mark the task line
itself `[x]` only when every DoD sub-item is `[x]` and Required Artifacts exist.

**If the task's Verification Method is Build+Test or Hybrid, it is already split into `a`
(Code) and `b` (Verify) sub-tasks in the Plan/Checklist** — work `a` to its own DoD (clean
build only) and its own Submit Point first; only then start `b`, where the build-test-debug
loop lives. **Submit at every declared Submit Point immediately upon reaching it** — do not
batch multiple tasks' completions into one later submit. Where a task is flagged
WIP-Checkpoint-Eligible, or its build-test-debug cycle is visibly not converging, issue a
`[WIP-CHECKPOINT]` submit stating what was attempted, what's confirmed working, and what's
known incomplete — this always bypasses code review regardless of the current policy
setting.

**When uncertainty arises** — a missing detail, a conflict between what the task says and
what you find in the code, an ambiguous type or interface, anything that makes you unsure
how to proceed correctly — **stop and consult the reference table in step 2** before
guessing or improvising. Use the named extraction command; read only the identified section.
Do not open other files or scan the repository.

**If anything cannot be resolved via the reference table** — a package/version conflict, a
persistent test failure, an unverifiable DoR, a low-confidence artifact, a requirement for
a production credential — **stop immediately.** Do not troubleshoot further, do not continue
with other tasks in the phase. Go to step 8 now.

### 8. Write the Phase Summary — on normal completion or on stopping
Write/update `[projectname]_phaseN_summary.md` (Plan §11.3): header block, tasks completed
with evidence, deviations, issues/problems (with full diagnostic detail if this is why you
stopped), assumptions, unplanned changes, incomplete tasks, open items, and **Escalation
Required: Yes/No** — if Yes, classify (A) replanning or (B) re-architecting if you can tell.

**If you stopped on an unresolved issue: this is the end of the session.** No PR, no further
progress, no commit of anything beyond what's already clean. The human takes this Phase
Summary to a Design Phase session to diagnose, update the Spec/Plan, and hand back a
restructured Plan/Checklist for a fresh session to resume from the last known-good phase.

### 9. On normal completion only: final wrap-up submit and stop
Individual tasks are already submitted at their own declared Submit Points (step 7) — this
is the phase-level wrap-up only: docs, README updates, and the Phase Summary, submitted
together. Confirm build/tests green. Notify the user the declared Session Unit is complete.
**Do not begin the next unit**, regardless of remaining capacity — it starts in a new
session.

### 10. Session-End Checklist
- [ ] Every DoD item you completed is checked in the Checklist — all of them.
- [ ] Any aborted task has its boxes unchecked, with a note explaining why.
- [ ] Build and test commands pass (unless you stopped per step 7/8, in which case this is
  the reason the Phase Summary exists).
- [ ] Phase Summary written and saved.
- [ ] No half-applied change left uncommitted.
- [ ] `scripts/setup_env.sh`/`.bat` reflect any prerequisites self-installed this session.
- [ ] You have not begun any task belonging to the next phase.
- [ ] The Checklist was the only doc file you edited, and every mark belongs to your
  current Phase — no other phase's box touched, no structural text edited.
- [ ] You did not perform a broad repository scan or read any Architecture Spec / Dev Plan
  file in full (except the protocols file). Every doc reference was a targeted extraction
  triggered by a specific uncertainty, using the method in the step 2 table.
- [ ] Every task/sub-task you completed this session has a matching `submit` and its
  Checklist `Submitted` box checked — no DoD-satisfied task left unsubmitted.
- [ ] Any WIP Checkpoint you issued this session has a description sufficient for a future
  session to resume from without redoing your work.
- [ ] You did not call `request_code_review` (per `AGENTS.md` §2.2's current setting).

**If any of the above is not true, fix it before ending the session.**

---

## Notes for Whoever Fills In This Template
- Replace every `[PROJECT_NAME]`, `[projectname]`, and `_v[N]` placeholder with actual
  values. Architecture Spec file versions are independent per file — each has its own `[N]`.
- In step 1 items 3 and 4, replace `[N]` in the grep/awk commands with the actual phase
  identifier from the checklist before handing this to a development agent.
- Fill step 5's build/test commands from Plan §2/§4.
- Fill step 4 only if the project uses infrastructure services; remove entirely if not.
- In the reference table, replace `FILE` with the actual filename in each row; replace
  example IDs (`REQ-ID`, `EntityName`, etc.) with project-specific examples where helpful.
- Reused verbatim every session — fix this file once, in place, if something's wrong for
  this project; don't hand-edit it per session.
