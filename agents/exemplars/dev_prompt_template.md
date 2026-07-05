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

### 1. Review all relevant docs, in order
1. `[projectname]_architecture_01_introduction_v[N1].md` through `..._08_constraints_and_roadmap_v[N8].md`
   (8 files, one per Design Step — `CLAUDE.md` §4.1) — authoritative for all technical
   decisions. **Each file's version is independent** — read off its own filename. Conflict
   with what you're about to do → the Architecture Spec wins; stop and escalate.
2. `[projectname]_dev_plan_01_overview_v[N].md` through `..._04_protocols_and_dod_v[N].md` —
   including the Session Handoff, Abort/Rollback, and Escalation protocols. Read these, not
   just skim — they tell you what to do when something goes wrong.
3. `[projectname]_dev_checklist.md` — your primary working document. **Edit in place only.**
   Never copy/rename/version it. This is the *only* doc file you may edit, and only within
   your current, identified Phase (flip DoD/task boxes, append a Session Log row) — every
   other file (Spec, Plan, this Prompt, prior Phase Summaries) is read-only; an apparent
   error in one is an Escalation Trigger, never a same-session fix.
4. Every existing `[projectname]_phaseN_summary.md`, in order — what actually happened in
   completed phases, not just what the Plan intended.
5. The Architecture Spec's Asset Manifest (`_06_viewpoints` §4.13), if a UI exists —
   cross-check against `assets/{html,images,audio,video}/`. A missing named asset is an
   Escalation Trigger.

### 2. Environment check and version sanity check
```bash
bash scripts/setup_env.sh    # or scripts/check_env.sh if present
```
If absent, manually verify each Plan §4 prerequisite. Per tool: present+correct → proceed;
**missing or wrong version, install succeeds cleanly** → append idempotently to
`scripts/setup_env.sh`/`.bat`, inform the user, proceed; **install fails, or any resulting
conflict/incompatibility surfaces → stop the session now** (§4 below — this is no longer a
trivial resolve).

### 3. Start infrastructure services (if required)
```bash
docker compose -f deploy/docker-compose.dev.yml up -d
docker compose -f deploy/docker-compose.dev.yml ps
```
Docker unavailable → treat as a missing prerequisite (step 2's rule).

### 4. Verify repository state before touching any code
Run the project's actual build/test commands (Plan §2/§4). **If the Checklist claims a
phase is complete but either fails: stop the session now** — write the Phase Summary
(step 7) describing the discrepancy and stop. Do not silently fix and continue.

### 5. Find the next phase
In the Checklist, find the first phase whose Exit Criteria isn't checked. Verify its Entry
Criteria are actually true from current repo state — not assumed from the Checklist alone.
Unverifiable → stop the session now.

### 6. Work the phase, one task at a time
Confirm each task's DoR before starting. Code in `src\`, never `docs\`. **Check off DoD
sub-items the moment each is satisfied** — continuously, not batched. Mark the task line
itself `[x]` only when every DoD sub-item is `[x]` and Required Artifacts exist.

**If anything goes wrong that you cannot resolve yourself** — a package/version conflict, a
persistent test failure, an ambiguous spec question, an unverifiable DoR, a low-confidence
artifact, a requirement for a production credential — **stop immediately.** Do not
troubleshoot further, do not continue with other tasks in the phase. Go to step 7 now.

### 7. Write the Phase Summary — on normal completion or on stopping
Write/update `[projectname]_phaseN_summary.md` (Plan §11.3): header block, tasks completed
with evidence, deviations, issues/problems (with full diagnostic detail if this is why you
stopped), assumptions, unplanned changes, incomplete tasks, open items, and **Escalation
Required: Yes/No** — if Yes, classify (A) replanning or (B) re-architecting if you can tell.

**If you stopped on an unresolved issue: this is the end of the session.** No PR, no further
progress, no commit of anything beyond what's already clean. The human takes this Phase
Summary to a Design Phase session to diagnose, update the Spec/Plan, and hand back a
restructured Plan/Checklist for a fresh session to resume from the last known-good phase.

### 8. On normal completion only: commit and stop
Commit all changes. Confirm build/tests green. Notify the user the phase is complete. **Do
not begin the next phase**, regardless of remaining capacity — it starts in a new session.

### 9. Session-End Checklist
- [ ] Every DoD item you completed is checked in the Checklist — all of them.
- [ ] Any aborted task has its boxes unchecked, with a note explaining why.
- [ ] Build and test commands pass (unless you stopped per step 6/7, in which case this is
  the reason the Phase Summary exists).
- [ ] Phase Summary written and saved.
- [ ] No half-applied change left uncommitted.
- [ ] `scripts/setup_env.sh`/`.bat` reflect any prerequisites self-installed this session.
- [ ] You have not begun any task belonging to the next phase.
- [ ] The Checklist was the only doc file you edited, and every mark belongs to your
  current Phase — no other phase's box touched, no structural text edited.

**If any of the above is not true, fix it before ending the session.**

---

## Notes for Whoever Fills In This Template
- Replace every placeholder with actual project name/slug/versions before handing to a
  development agent. Architecture Spec file versions are independent per file.
- Fill step 4's build/test commands and step 2's version-check commands from Plan §2/§4.
- Fill step 3 only if the project uses infrastructure services; remove entirely if not.
- Reused verbatim every session — fix this file once, in place, if something's wrong for
  this project; don't hand-edit it per session.
