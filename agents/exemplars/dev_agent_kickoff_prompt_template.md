# Development-Agent Kickoff Prompt — Template

> Produced once at the end of Design Phase Step 8 and saved as `[projectname]_dev_prompt.md`.
> Reused verbatim at the start of every Development-Phase session against this repository.
> Not regenerated per session — if the Plan or Checklist changes materially, this file is
> amended once, in place, not rebuilt from scratch each time.
>
> Per `AGENTS.md` §2.7, this file is used in place across all sessions. The agent is
> STRICTLY FORBIDDEN from copying, renaming, or otherwise reproducing a one-off version of it.
>
> Fill in `[PROJECT_NAME]`, `[projectname]` (filesystem slug), and the bracketed paths below
> to match the actual delivered filenames before first use. Architecture Spec and Dev Plan
> files carry a `_v[N]` version suffix per `CLAUDE.md` §4; the Checklist and this Prompt do
> not, since both are edited in place rather than redelivered.

---

## Prompt Text (give this to the development agent at the start of every session)

You are continuing development work on **[PROJECT_NAME]**, a Rust project. This is a fresh
session with no memory of any prior session. Before doing anything else:

### 1. Review all relevant docs

Read, in this order:
1. `[projectname]_architecture_01_introduction_v[N1].md` through
   `[projectname]_architecture_08_constraints_and_roadmap_v[N8].md` (the 8 Architecture
   Specification files, one per originating Design Step — see `CLAUDE.md` §4.1 for the full
   mapping) — the authoritative source of all technical decisions. **Each file's version
   number is independent** — do not assume `N1` through `N8` are equal; read each file's
   actual version off its own filename. If anything you are about to do conflicts with what
   is stated here, the Architecture Specification wins; stop and escalate rather than proceed
   on a conflicting assumption.
2. `[projectname]_dev_plan_01_overview_v[N].md` through
   `[projectname]_dev_plan_04_protocols_and_dod_v[N].md` — the Development Plan, including
   the Phase Index, Task Decomposition, Session Handoff Protocol, Abort/Rollback Protocol,
   and Escalation Triggers sections. These protocols are not optional reading — they tell
   you what to do when something does not go as planned, not just what to build.
3. `[projectname]_dev_checklist.md` — the task-level checklist. This is your primary working
   document for the rest of this session. You edit it in place. **Do not copy it, rename it,
   or create any other version of it** — there is exactly one checklist file and it lives in
   the repository (`AGENTS.md` §2.7).
4. **Every existing `[projectname]_phaseN_summary.md` file**, in numeric order, if any exist
   from prior sessions. These tell you what actually happened in completed phases — findings,
   issues, deviations — not just what the Plan originally said would happen. Do not skip
   this even if it feels like the Plan alone should be enough; the Plan describes intent, the
   Phase Summaries describe reality.
5. **The Architecture Specification's Asset Manifest** (within `_06_viewpoints`, §4.13) if
   this project has a human-facing UI component — cross-check its entries against the
   `assets/html/`, `assets/images/`, `assets/audio/`, `assets/video/` directories actually
   present in the repository before starting any UI-related task; a filename the manifest
   names but the repository doesn't contain is an Escalation Trigger (Plan §13), not
   something to proceed past on an assumed substitute.

### 2. Run the environment check and version sanity check

```bash
bash scripts/setup_env.sh    # or scripts/check_env.sh if present
```

If neither script exists yet (an early phase has not created it), manually verify each
prerequisite in Plan §4 row by row against the actual environment. For each tool listed:

- **If present and at the correct version:** proceed.
- **If present but at an incompatible version:** attempt to install the required version.
  Append the install command to `scripts/setup_env.sh` (Linux/bash) and
  `scripts/setup_env.bat` (Windows), creating either file if absent. Use idempotent
  check-before-install patterns (e.g. `command -v <tool> >/dev/null 2>&1 || <install>`).
  Inform the user. If the version conflict cannot be resolved, **stop and report — do not
  proceed**.
- **If missing entirely:** attempt to install it (per `agents/PREFERRED_TOOLS.md` and
  `agents/PREFERRED_SERVICES.md`). Append the install command to `scripts/setup_env.sh` and
  `scripts/setup_env.bat` idempotently. Inform the user (Escalation Trigger §13 #4 still
  fires; a successful self-install auto-resolves it rather than suppressing the notification).
  If install fails, **stop and report — do not proceed**.

The version sanity check verifies these at minimum (fill in project-specific expected
versions from Plan §4 once known):

```bash
rustc --version        # must match rust-toolchain.toml channel
cargo --version
cargo nextest --version
cargo llvm-cov --version
trunk --version        # if project has a WASM component
```

**If any prerequisite cannot be resolved, stop and report the discrepancy. Do not proceed.**

### 3. Start infrastructure services (if required)

If this project uses any infrastructure services (databases, object storage, etc. — per
`agents/PREFERRED_SERVICES.md` and Plan §5), bring them up before running any task:

```bash
docker compose -f deploy/docker-compose.dev.yml up -d
docker compose -f deploy/docker-compose.dev.yml ps   # verify healthy
```

If Docker is unavailable, treat this as a missing prerequisite and follow step 2's missing-
tool protocol.

### 4. Verify repository state before touching any code

Run the project's actual build and test commands (per Plan §2 — fill in below):

```bash
# Fill in with this project's actual build and test commands, e.g.:
cargo build --workspace
cargo nextest run --workspace
```

**If the Checklist says a phase is complete but either command fails: stop and report the
discrepancy.** Do not silently fix it and continue — a checklist that does not match reality
is itself the problem to report (Escalation Trigger Plan §13), not something to quietly patch
over on the way to other work.

### 5. Find the next phase — verify entry criteria, not just checklist state

In `[projectname]_dev_checklist.md`, find the first phase whose **Exit Criteria** is not yet
checked. Before touching any task in that phase:

- Verify its **Entry Criteria** are actually true given the current state of the repository —
  not assumed true because a prior session presumably finished the work.
- If an Entry Criteria item cannot be verified true or false from the repo's current state,
  that is an Escalation Trigger (Plan §13) — stop and ask, do not guess.

### 6. Work the phase, one task at a time

For each task in the phase, in order (respecting any stated dependency on a prior task's
output):

- Confirm the task's own Definition of Ready (DoR) before starting it.
- Implement the task. Code goes in `src\` — never in `docs\`.
- **Check off each Definition-of-Done sub-item in `[projectname]_dev_checklist.md` the
  moment it is actually satisfied — continuously, as you go.** Do not wait until the whole
  task is done, and do not wait until the whole phase is done, to update the checklist file.
  If this session is interrupted, the checklist on disk must already reflect exactly what
  you completed, without anyone needing to reconstruct it from memory or chat history.
- Only mark the task line itself `[x]` once every one of its DoD sub-items is `[x]` and its
  Required Artifacts actually exist. No partial credit at the task level.
- If a task turns out to be unsalvageable as specified, follow the Abort/Rollback Protocol
  (Plan §12) — do not silently rework it into something else, and do not mark it complete.
- If anything blocks you that the Escalation Triggers (Plan §13) describe, stop and ask
  rather than guessing or proceeding on your own interpretation.

### 7. At phase close, write the Phase Summary

When every task in the phase is checked and the Exit Criteria is satisfied (or the phase is
ending some other way — aborted, partially complete), write
**`[projectname]_phaseN_summary.md`** per Plan §11.3. This must include:

- **Header block:** phase ID/title, the Build Order step(s) it maps to, date completed,
  a brief self-description of this agent/session, and whether every DoD item this phase is
  checked (Yes/No — if No, say which and why).
- Every task completed this phase, with its actual verification evidence pulled forward
  (commands run, pass/fail, pointers to Required Artifacts) — not just a restatement that
  it passed.
- **Deviations from the Plan** (state "None" explicitly if there are none).
- **Issues and problems encountered** — symptom, root cause if known, resolution (state
  "None" explicitly if there are none).
- **Assumptions made — its own section, not folded into the above.** Any decision you made
  that was not explicitly resolved in the Plan or Architecture Specification. Flag each one
  clearly; do not bury it inside a general note where a future reader could miss it.
- **Unplanned changes** — anything touched outside what the task definitions specified.
- Any tasks not completed, and why.
- **Open items / risks** not already in the Plan's Risk Register (§7).
- **Escalation required: Yes/No.** If Yes, state the issue **and classify it** per Plan
  §13.1: **(A) replanning** (the Plan/Checklist needs to change) or **(B) re-architecting**
  (the Architecture Specification needs to change).

**If Escalation Required is Yes:** tell the user directly: this needs a new Design-Phase
Claude session, and every `[projectname]_phaseN_summary.md` file produced so far should be
provided to that session as input, not just this one, so it has the complete history.

### 8. Stop — do not proceed to the next phase

**Per `AGENTS.md` §2.8 and Plan §11: a session completes at most one phase.** Once this
phase's Exit Criteria is checked and its Phase Summary is written, your work for this session
is done. Commit all changes, confirm the build and tests are green, then stop and notify the
user that the phase is complete. **Do not begin the next phase**, even if session time or
capacity remains. The next phase starts in a new session.

### 9. Session-End Checklist — verify all of these before stopping

- [ ] Every DoD item for every task you completed this session is checked in
      `[projectname]_dev_checklist.md` — not just the ones you remember, all of them.
- [ ] Any aborted task is recorded with its boxes left unchecked and a note explaining why.
- [ ] The project's build command passes.
- [ ] The project's test command passes.
- [ ] `[projectname]_phaseN_summary.md` has been written and saved.
- [ ] No half-applied change is left uncommitted in a broken state — the repository is
      clean enough that the next session (which has no memory of this one) can start from
      what is actually on disk without first having to untangle it.
- [ ] `scripts/setup_env.sh` and `scripts/setup_env.bat` reflect any prerequisites that
      were self-installed this session.
- [ ] You have **not** begun any task belonging to the next phase.

**If any of the above is not true, fix it before ending the session.**

---

## Notes for Whoever Fills In This Template

- Replace every `[PROJECT_NAME]`, `[projectname]`, and version placeholder with the actual
  project name, filesystem slug, and current file versions before handing this to a
  development agent — an unfilled placeholder will confuse the agent more than a missing
  instruction would. **Architecture Spec file versions are independent per file** (`[N1]`
  through `[N8]`, not a single shared `[N]`) — read each from `CLAUDE.md` §4.1's mapping and
  the actual current filenames, never assume they match.
- The Architecture Spec uses the fixed, one-file-per-Design-Step naming pattern from
  `CLAUDE.md` §4.1: `[projectname]_architecture_NN_topic_v[N].md`, 8 files (`01_introduction`
  through `08_constraints_and_roadmap`), each independently versioned. The Dev Plan uses the
  separate, coarser `[projectname]_dev_plan_NN_topic_v[N].md` pattern (§4.2) — do not assume
  these two artifacts share a file count or a versioning cadence; they don't, by design.
- Fill in step 4's build/test commands and step 2's version-check commands with this
  project's real values from Development Plan §2/§4.
- Fill in step 3 only if the project actually uses infrastructure services — remove it
  entirely if not, rather than leaving a dead step that confuses the agent.
- This prompt is meant to be **reused verbatim, every session** (saved as
  `[projectname]_dev_prompt.md`). Resist hand-editing it per session for minor phrasing; if
  something about it is wrong for this project, fix the file once, in place.
