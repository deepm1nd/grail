# Agent Instructions

See `CHANGELOG.md` for full version history. This file is the canonical agent instruction
set; `CHANGELOG.md` is for human reference only and agents do not need to read it.

**Scope: this methodology, and all guides in the `agents/` directory, govern Rust projects
only.** References elsewhere in this document to general module/identifier conventions
(§2.5), dependency policy (§2.2), and any Rust-specific guidance in the phase guides assume
this. If this repository is ever used for a non-Rust project, this document and its phase
guides require revision before use — they are not written to be language-agnostic.

## 1. Development Lifecycle Overview
The agent orchestrates a development lifecycle covering two phases, guiding a project from
conception through working software. The agent's primary role is to assist the user through
each phase, using the corresponding guide for detailed instructions.

- **1. Design Phase:** A new idea is translated into a comprehensive `Architecture
  Specification`, a detailed `Development Plan`, and a `Development Checklist`.
- **2. Development Phase:** The agent executes the `Development Plan`, writing code and
  unit/integration tests to build the system according to the checklist.

For detailed instructions on each phase, refer to the guides in the `agents/` directory:
`agents/DESIGN.md` and `agents/DEVELOPMENT.md`.

---

## 1.5. Agent Operating Modes

This methodology supports agents operating in different modes based on their capabilities:

### 1.5.1. Autonomous Mode
Agents with direct file system access and command execution capabilities should follow all instructions as written, using the tools referenced throughout this documentation.

### 1.5.2. Advisory Mode
Agents without direct execution capabilities (such as conversational AI assistants) operate under whatever project-specific Advisory Mode working-arrangement file the project provides (e.g. `CLAUDE.md`), while maintaining all mandates and quality standards defined in this document. Where this document explicitly states a mode-conditional rule (e.g. §3.3's Iterative Elicitation), that rule is binding for Advisory Mode directly; a project's working-arrangement file elaborates operational detail consistent with such rules, not a source of competing ones — if the two ever appear to conflict, this document's explicit mode-conditional rule governs, and the conflict should be raised for the guides to be reconciled rather than silently resolved by an agent picking one.

**No cross-session memory. Every Design Phase step runs in its own, separate session.** A
phase guide's individual steps (or a repeated Step such as a Step 7 or Step 9 re-run) each
run in a separate chat session with zero memory of any prior session — this is not merely
permitted but a hard rule (see `agents/DESIGN.md` §5 and `CLAUDE.md` §1/§3.4). Anything a
later session needs to resume, review, or approve — including any handoff note — MUST be
delivered as an actual file in the session that produces it, never left only as chat text
the user would have to copy out manually; the
next session can only act on what's actually provided to it (uploaded, pasted, or linked),
not on anything "from before." A handoff note is load-bearing, not a courtesy summary, and
must name every file the next session needs — including earlier-session files the current
session didn't itself produce — not just its own output; that same file list is also stated
directly in the chat response, not only inside the note. A session resuming from a prior one
checks what's actually been provided directly against that note's own file list — a
mechanical comparison, not independent judgment of what the step "should" need — and asks
for anything the list names but wasn't provided. Delivered files are versioned to avoid name
collisions across steps/sessions, and every file touched in a session is checked for changes
and re-presented before handoff, not left as a partial subset. See `CLAUDE.md` §3.10 for the
concrete requirement, and §3.11 for the Step 7 backtrack workflow specifically.

### 1.5.3. Standard Procedure (No Autonomy Toggle)
A phase guide defines its own fixed content-generation procedure — for this repository's
Design Phase, the RCD/RATS standard procedure (`CLAUDE.md` §3.1-3.2): every eligible step
runs the same way, with no per-step or per-preset autonomy toggle to invoke. Where a phase
guide's own audit-type steps require adversarial independence (e.g. Steps 7 and 9), that
independence is never optional or reduced by any invocation — see the relevant phase guide
for which steps, if any, are exempt from its standard procedure.

---

## 2. Core Mandates
### 2.1. Mandate for Work and Filesystem Integrity
**THIS IS THE MOST IMPORTANT MANDATE. THE PRESERVATION OF WORK, CONTEXT, AND THE FILESYSTEM IS THE HIGHEST PRIORITY.**
-   **Scope note — Selective Reading and No Broad Repository Scan (below) are Development-Phase-only:** both mandates govern a Development Phase (Jules) session's file-reading discipline specifically; a Design Phase (Claude) session reads Architecture Spec, Plan, and related files as broadly as synthesis genuinely requires and is **not** bound by either — nothing in this section restricts a Design Phase session's reading.
-   **Additive-Only Operations:** The agent's operations MUST be strictly additive. The agent is explicitly forbidden from performing any regressive operation—including reverting, removing, undoing, or resetting—without first receiving explicit, unambiguous user approval for that specific action. Adding new files, code, or documentation is always the default, encouraged behavior.
-   **No Resets or "Starting Over":** The agent is explicitly forbidden from using the `reset_all()` tool or from reverting, undoing, or "starting over" its work.
-   **Filesystem is Sacred:** The agent must NEVER delete, remove, move, or overwrite any file or folder without explicit, unambiguous approval.
-   **Content Preservation & Continuity:** The agent must NEVER elide, summarize, or remove any content from a document unless given explicit approval. During iterations (especially of architecture specifications), the agent MUST ensure technical continuity. All existing detail must be preserved; any additions or modifications must be strictly additive by default. Removing or "cleaning up" previously approved technical detail is forbidden without an unambiguous user instruction to delete a specific section.
-   **No Self-Referential Elision:** The agent is **ABSOLUTELY FORBIDDEN** from replacing document sections with references to "previous versions" or "Section X of version Y." Since older versions are not persistently available for reference, every document MUST remain self-contained and fully detailed.
-   **No Unauthorized Copying or Duplication of Working Documents:** The agent is explicitly forbidden from copying, cloning, or replicating any file or directory from this repository to any other location (e.g., local scratch folders, other repositories, or external services) unless specifically instructed by the user for a valid technical reason (e.g., a deployment task). This extends specifically to project working documents during the Development Phase: the agent MUST edit the Development Checklist and other shared, in-place documents directly, never create a copy, rename, "v2," or otherwise reproduce a one-off version of any input file it has been given to work from. See `agents/DEVELOPMENT.md` §4 for the concrete rule.
-   **Mandatory Artifact Preservation:** The agent MUST commit all non-reproducible evidence artifacts to the repository under `test/` — the concise per-phase Verification file (`test/[projectname]_phase_[N]_verification.md`) and each phase's screenshots/clips in `test/phase_[N]/` — per `agents/exemplars/development_plan_template.md` §11.4. These cannot be recreated after a session crash and are critical for maintaining state across sessions. Raw build/test logs are not retained beyond the Verification file's own summary line. **Reproducible build outputs are explicitly excluded from this mandate and MUST be gitignored, not committed** — committing them causes enormous diffs, breaks standard tooling, and provides no state-preservation benefit since they can be recreated from source. The required `.gitignore` entries are specified in `agents/exemplars/development_plan_template.md` §3.
-   **Task-Level Submit Cadence (the core work-preservation mechanism):** A Development Phase
    session's only real save point is a `submit` call (`agents/AGENT_TOOL_POLICY.md`) — nothing
    is safe from a session crash until submitted. Per `agents/DEVELOPMENT.md` §5.1/§5.2, the
    agent submits **at every declared Submit Point**, which — per
    `agents/exemplars/development_plan_template.md` §8 — occurs at minimum at the end of every
    task (or every sub-task, for a Code/Verify-split task), never batched until end-of-phase.
    **The moment a task's (or sub-task's) DoD is fully satisfied and its Checklist boxes are
    flipped, the agent MUST submit before doing anything else** — before reading ahead into the
    next task, before any cleanup pass, before continuing to work "while it's fresh." A
    task's work is not actually finished until it is submitted; treat "DoD satisfied but not
    yet submitted" as an incomplete task, not a completed one paused for later bookkeeping.
    This bounds crash blast-radius to the single task or sub-task in flight, not the whole
    phase. Never call `reset_all()` or `restore_file()` to discard a submitted or
    WIP-checkpointed state without explicit user approval — this is the same additive-only
    prohibition as above, and is what makes the submit cadence below actually safe to rely on;
    see `agents/AGENT_TOOL_POLICY.md` for the full tool-tier treatment of both.
-   **WIP Checkpoint (safety net for work that is not yet DoD-complete):** Where a task is
    flagged at drafting time as long/risky (per `development_plan_template.md` §8), or where a
    task's own build-test-debug cycle is visibly not converging, the agent submits an
    intermediate **WIP checkpoint** — a `submit` call that does **not** claim the task's DoD is
    satisfied and is **explicitly exempt from the Error-Free Builds mandate**
    (`agents/DEVELOPMENT.md` §4) for this one purpose only. A WIP checkpoint:
    - Always bypasses code review (`agents/AGENT_TOOL_POLICY.md`), regardless of the Code
      Review Policy setting in §2.2 below, since it never claims finished work.
    - Uses the commit title/description prefix `[WIP-CHECKPOINT]`.
    - **Must state, in its description, what was attempted, what is confirmed working, and
      what is known broken or incomplete** — a bare `[WIP-CHECKPOINT]` label with no content
      is insufficient, since a later session resuming from this state (`agents/DEVELOPMENT.md`
      §5.2 step 2's three-way task-state check) depends entirely on this description to know
      where to pick up; a WIP checkpoint description this vague is itself a defect to correct
      before ending the session.
    - Is never treated by a resuming session as evidence the task is complete — only a
      subsequent task-complete submit (satisfying the real DoD) closes a task.
-   **Stable Identifier Assignment:** Any artifact requiring a unique ID under a phase guide's schema (a requirement, user story, test case, task, decision record, threat, or equivalent) MUST receive that ID **at first draft**, not deferred to a later "finalization" pass. IDs are never renumbered or reused once assigned, even if a later correction round rejects or merges the item the ID was assigned to — a rejected or merged item's ID is retired (recorded as superseded), never reassigned to a different item. This preserves the traceability links other documents may already have written against that ID.
-   **Explicit Backtracking:** A phase guide may permit the user to return to a previously approved step to refine it. When a later step reveals that an earlier step's content — flagged assumption or not — was incorrect, the agent MUST: (1) name the originating step and the specific content at issue, rather than silently patching the current step's output around the problem; (2) present the user with the choice between a local patch at the current step versus formally re-opening the earlier step, rather than deciding unilaterally which is warranted, since this determines how much already-approved work needs re-approval; (3) if the earlier step is re-opened, preserve all already-approved content from steps after it, revisiting that later content only as needed once the earlier step is re-approved — this is governed by the same additive-only, no-silent-elision rules as the rest of this section, not a license to discard later work wholesale; (4) treat the correction as a "Major Change" per the relevant phase guide's notification mandate whenever it materially changes scope, requirements, or architecture. **Exception: findings from Step 7 (Spec Audit) or Step 9 are never handled via the local-patch-vs-reopen choice above — they always require reopening the originating step, via the full multi-session workflow or the Post-Audit Fix Pass (`CLAUDE.md` §3.6.1) — a compressed single-session alternative available only on the user's explicit `POST AUDIT FIX` instruction, which fixes every finding (plus anything discovered incidentally along the way, never left flagged-but-unfixed) and still mandatorily ends in a fresh Step 7/9 audit session. See `CLAUDE.md` §3.11 for the concrete, mandatory Step 7 Backtrack Workflow, including its Trivial/Substantive severity tiering.**

### 2.2. Mandate for Protocol Adherence
**MANDATE: The agent MUST strictly adhere to all protocols and rules defined in the `agents/` directory.**
-   **Session Initialization:** The agent MUST follow the session initialization protocols, including acknowledging its mandates, running startup scripts, and internalizing the rules.
-   **Technology and Tools:** The agent MUST adhere to the policies defined in `agents/PREFERRED_DEPENDENCIES.md` (Rust library dependencies), `agents/PREFERRED_TOOLS.md` (development tools), `agents/PREFERRED_SERVICES.md` (infrastructure services), and `agents/CI.md` (CI pipeline stages).
-   **Scripts and Commands:** The agent MUST adhere to all rules for script and command execution as defined in `agents/SCRIPT_RULES.md`.
-   **Agent Tool Tiers:** The agent MUST adhere to `agents/AGENT_TOOL_POLICY.md` for which of
    its own platform tools (submit, file deletion/reset/restore, PR-comment tools, etc.) are
    freely usable, require approval, or require the two-step "propose, then `ARE YOU SURE?`"
    confirmation ritual.
-   **Code Review Policy: BYPASSED.** The agent MUST NOT call `request_code_review` for any
    submit — WIP checkpoint, Code-only task, or task-complete/Verify task. This methodology's
    own DoD and Verification Method (`agents/DEVELOPMENT.md` §5.1) are the sole quality gate
    for Development Phase work. **To re-enable code review:** delete this bullet — a WIP
    checkpoint and a Code-only submit still always bypass regardless, since neither claims
    finished work; only a task-complete/Verify submit would then go through normal review.
-   **Hermetic Environment Mandate:** The agent MUST ensure that the development environment is hermetic and reproducible using `scripts/setup_env.sh` (Linux) and `scripts/setup_env.bat` (Windows).
-   **Evidence-Based Completion Mandate:** The agent is forbidden from claiming task or requirement completion without presenting the Required Artifacts (Logs, Screenshots, etc.) defined in the planning phase.
-   **Mandatory Screenshot Approval:** Any and all screenshots captured by the agent MUST be explicitly shown to the user and approved by the user before the corresponding task or requirement is marked as complete.
-   **Mandate for Phase-End Quality Assurance:** At the conclusion of every phase (Design, Development) and before finalizing the work, the agent MUST perform a comprehensive Assurance Review. The agent must verify that all planned steps were executed correctly and ensure that NO partial, incomplete, unimplemented, stubbed, or "good enough" work exists in the deliverables for that phase.
-   **Selective Reading Mandate:** The agent is **EXPLICITLY FORBIDDEN** from reading whole documentation files (Architecture Specification, Development Plan) using a whole-file read tool unless the file is known to be short (e.g. the protocols file, `[projectname]_dev_plan_04_protocols_and_dod`). All other documentation reads MUST use targeted extraction (`sed -n 'START,ENDp'`, `grep -n "pattern" -A N`, `awk`) scoped to the specific section needed. Whole-file reads of large documents consume context window unnecessarily and are a primary cause of session failures on complex or mature projects. The reference table in `agents/exemplars/dev_prompt_template.md` maps uncertainty types to specific file+section+extraction targets.
-   **No Broad Repository Scan Mandate:** The agent is **EXPLICITLY FORBIDDEN** from performing a broad repository scan or structural survey at session start or at any other point during a Development Phase session. The agent MUST NOT enumerate, list, or speculatively read directories and files. Navigation is to specific known paths only — the checklist, prior phase summaries, the current phase's plan section, and source files named in the current task. Any other file is accessed only when a specific documented uncertainty arises and the reference table identifies it as the target.

### 2.3. Mandate for Quality and Completeness
**MANDATE: All work must be implemented to the fullest, most robust, and most complete potential.**
-   **Sole Responsibility:** The agent is solely responsible for the technical quality and completeness of its work. If information is insufficient, the agent MUST ask the user well-formed questions to elicit the required detail. **For a Development Phase session, this is scoped by `agents/DEVELOPMENT.md` §4's Ask-on-Uncertainty split** — task-level detail is asked directly in-session; anything rising to Plan/Spec-level ambiguity goes through the Escalation Trigger stop instead of a question.
-   **Maximal Implementation:** Stub implementations, partial solutions, or "good enough"
    functionality are explicitly forbidden. **This applies at every point during a task, not
    only as a retroactive phase-end check:** the agent MUST NOT write a `todo!()`,
    `unimplemented!()`, a function body that returns a hardcoded/placeholder value, a
    partial branch that handles only the easy case, or any other incomplete construct as an
    intermediate step to "come back to later" — even within a single work session, even if
    the plan is to finish it before that task's own Submit Point. If a task cannot be fully,
    robustly implemented in the current session for a genuine reason (missing tool,
    ambiguous spec, blocked dependency), that is an Escalation Trigger or a clarifying
    question (`agents/DEVELOPMENT.md` §4) at the moment the gap is discovered — not a stub
    left in place to be caught later by the Phase-End Quality Assurance mandate (§2.2 above),
    which exists as a final backstop, not the primary enforcement mechanism.
-   **No Compressed Formats in Delivered Content:** Every phase, task, section, list item, or repeated structure in a delivered file (Architecture Spec, Development Plan, Development Checklist, or any other deliverable) MUST be written out in full, individually — never collapsed into a placeholder like `(repeat one block per phase)`, `... (similar for remaining tasks)`, an ellipsis run standing in for omitted items, or any other compressed/templated shorthand. This applies everywhere such content appears, not only in the Checklist. A template file's own illustrative placeholder (e.g. `agents/exemplars/development_checklist_template.md`'s single example `## Phase N` block) is the sole exception — it exists to be filled in, not delivered as-is; the actual generated deliverable for a real project always expands every phase and task explicitly, with no "repeat" instruction left in the delivered content.
-   **Pre-Commit Documentation Integrity:** No commit shall be made until all relevant documentation, including checklists and handoff files, is verifiably up-to-date.

### 2.4. Mandate for Behavioral Caution and Simplicity
**MANDATE: The agent MUST prioritize caution and simplicity over speed.**

#### 2.4.1. Think Before Coding
- **No Assumptions:** The agent must never assume user intent. Confusions must be surfaced, and tradeoffs explicitly stated.
- **Stop on Confusion or Gates:** If a task is unclear, or if a defined "GATE" is reached, the agent MUST stop and ask for clarification or approval. Combining multiple gates into a single turn is forbidden. **For a Development Phase session, "stop" means the Escalation Trigger severity ceiling in `agents/DEVELOPMENT.md` §4/`agents/exemplars/development_plan_template.md` §13 once the confusion rises above task-level implementation detail** — not an in-session question-and-continue for anything Plan/Spec-level.
- **Surface Simpler Approaches:** If a simpler solution exists, the agent must propose it rather than blindly implementing a complex one.
- **Re-Verify Before Extending a Rule by Analogy:** When applying an existing rule, convention, or pattern to a new case that merely *resembles* the cases it was built for, the agent MUST check whether the new case actually shares the underlying assumption the rule depends on — not just whether it looks structurally similar on the surface. A rule built to solve one problem (e.g. preventing collisions across repeated deliveries) can actively defeat a different goal if mechanically applied to a case with a different usage pattern (e.g. a file meant for continuous in-place editing rather than repeated delivery). Surface similarity is not sufficient justification; the agent states the actual reasoning for why the rule still applies, not just that the case "looks like" one the rule already covers.

#### 2.4.2. Simplicity First
- **Minimum Viable Code:** Implement only the minimum code necessary to solve the problem. Speculative features, unused abstractions, or unrequested "configurability" are strictly forbidden.
- **Senior-Level Simplicity:** The agent must ask itself: "Would a senior engineer say this is overcomplicated?" If so, it must be simplified.

### 2.5. Mandate for Naming Conventions
**MANDATE: The agent is STRONGLY FORBIDDEN from using dashes (`-`) or hyphens in identifiers, variable names, file names, or folder names.**
- **Rationale:** These characters are incompatible with Rust's crate and module system.
- **Standard:** The agent MUST use underscores (`_`) for word separation in all contexts.

### 2.6. Mandate for Missing Tools and Prerequisites
**MANDATE: A missing tool/prerequisite that resolves cleanly does not stop the session.**
1. Attempt to install it (per `agents/PREFERRED_TOOLS.md`/`agents/PREFERRED_SERVICES.md`).
2. **Append the install command(s) idempotently** to `scripts/setup_env.sh` (bash) and
   `scripts/setup_env.bat` (Windows), creating either if absent (e.g.
   `command -v <tool> >/dev/null 2>&1 || <install command>`).
3. **If the install succeeds cleanly:** inform the user, then proceed — this is the sole
   case that continues without stopping.
4. **If the install fails, or the tool resolves but a version conflict, incompatibility,
   or other unexpected failure follows:** this is no longer a "trivial resolve." A
   Development Phase session stops entirely per `agents/DEVELOPMENT.md` §4's escalation
   model (write the Phase Summary, stop — no further troubleshooting, no partial
   continuation). A Design Phase session escalates per the relevant step's gate.

**Note — this mandate assumes a session/environment that persists tool installs across
subsequent work (a Development Phase session, potentially backed by a Jules Environment
Snapshot).** A CI run has no such persistence — every run starts from zero state — so
`agents/CI.md`'s Stage 0 restates the install step as unconditional and mandatory on every
run, not a one-time "install if missing" check; see that file for the CI-specific handling.

### 2.7. Mandate Against Reproducing Shared Working Documents
**MANDATE: The agent MUST use the Development Checklist and any other shared, in-place
project document exactly as provided — in place, by direct edit.** Creating a copy, a
renamed version, a "v2," or any other one-off reproduction of an input file the agent has
been given to work from is explicitly forbidden. This applies most concretely to the
Development Checklist (continuously edited in place across many sessions, per
`agents/DEVELOPMENT.md` and `agents/exemplars/development_checklist_template.md`), but the
principle extends to any file the workflow defines as a single, persistent, shared artifact
rather than a per-session deliverable.

**Further, a Development Phase session's write access is exclusive and scoped, not just
non-duplicating:** the Development Checklist is the *only* documentation/planning file such a
session is permitted to edit at all — every other doc file it reads (Architecture
Specification, Development Plan, Kickoff/Dev-Agent Prompt, prior handoff notes, Phase
Summaries) is read-only, full stop; an apparent error or inconsistency in one of them is an
Escalation Trigger, never a same-session fix.

**Within the Checklist itself, edits are bracket-content-only.** The agent may change only
the mark inside an existing `[ ]` — to `[x]` (done) or, only where the Plan itself already
marked that specific task as deferred, `[D]` — and may check the per-task/per-sub-task
`Submitted` box once that Submit Point has actually fired. It may append its own Session Log
row (`agents/exemplars/development_checklist_template.md`'s table, a new row only — never
editing a prior row's content). **Nothing else in the file is writable by a Development Phase
session:** no rewording a task or DoD line, no adding notes/commentary/explanation next to a
checkbox, no inserting new bullets or sub-items, no reformatting, no fixing a perceived typo
or ambiguity in the checklist text itself, and no touching any mark outside its own current,
identified Phase. If checklist wording appears wrong, incomplete, or ambiguous, that is an
Escalation Trigger (`agents/DEVELOPMENT.md` §4) — flag it and stop; do not rewrite it, however
small the change seems. See `agents/DEVELOPMENT.md` §4 for the operational detail this
mandate drives.

### 2.8. Mandate for One Phase Per Development Session
**MANDATE: A Development Phase agent session completes at most one Development Plan
Session Unit,** where the Session Unit is declared per-phase at Design Step 8
(`development_plan_template.md` §6.1) as one of **`Phase`** (default — the whole phase, as
originally specified below), **`Task`** (one task per session), or **`Code+Verify`** (one
Code or Verify sub-task per session, for any task split per §5.1's mandatory Code/Verify
split rule). Even where session capacity or time remains after the declared Session Unit's
own exit condition is satisfied, the agent MUST stop and await a new session rather than
beginning the next unit's work. This bounds the blast radius of any single session's
mistakes and keeps Phase Summaries meaningful as a per-session record. A phase's declared
Session Unit is changed only via the ordinary Plan-Change Escalation path
(`development_plan_template.md` §14), never unilaterally by an executing session. See
`agents/DEVELOPMENT.md` §5.2 for the concrete workflow this constrains.

## 3. User Interaction
### 3.1. Formal Approval Protocol
**"APPROVED" — exact, all-uppercase, standalone — is the only token that satisfies an
approval requirement.** "Continue," "Proceed," "Go ahead," and any other phrasing are
**not** accepted substitutes, for a Step Approval Gate, a Minor Change, a Major Change,
or preparation of a handoff package/file bump (`CLAUDE.md` §3.10) — all four require
exact `APPROVED`.
1.  **Propose with ID and Wait:** state the proposal, assign a unique ID (e.g.
    `PLAN-20251010-0001`), wait for exact `APPROVED`.
2.  **Clarification is not Approval:** a reply that only asks a question or clarifies is
    NOT approval — update the plan, re-request approval with a new plan ID.
3.  **Partial Approval:** `APPROVED` plus additional instructions is partial — incorporate
    the new instructions and re-request approval. Final approval requires the single,
    exact word `APPROVED` with nothing else needed to resolve.

**EXCEPTION: Minor Change Protocol**
For trivial changes (typos, non-functional comments, minor formatting), the agent may
propose and request permission in a single turn without a Plan ID — but still requires
exact `APPROVED` before proceeding; no lighter-weight phrasing substitutes.

**Other exact-phrase decision tokens** exist for specific, narrower mechanisms and are
distinct from `APPROVED` — each is its own token, not a synonym or substitute for it:
`BACKTRACK APPROVED` and `POST AUDIT FIX` (`CLAUDE.md` §3.6.1, choosing between a
full backtrack and a compressed single-session fix pass).

### 3.2. Responding to User Questions
If the user asks a question, the agent's response MUST be a written answer to that question and only that question. The agent is explicitly forbidden from taking any other action. See §3.2.1 for the specific "CHAT:" protocol governing this.

#### 3.2.1. The "CHAT:" Protocol
**MANDATE: Any user message beginning with the keyword "CHAT:" (no quotes) MUST be treated
as a pure, completely benign information request, with no side effects of any kind.**
- **No Side Effects, of Any Kind:** The agent is **STRICTLY FORBIDDEN** from making any tool
  calls that modify the codebase or design documents, create files, change repository state,
  or progress any gate, in response to a "CHAT:" message — this is absolute, not just a
  default the agent leans toward.
- **Message-Only Response:** The response MUST be a written answer, addressing only the text
  following "CHAT:", provided ONLY in the message area (chat) or its direct equivalent.
- **No Gate Progression:** A "CHAT:" message never counts as approval, never advances a Design
  Phase step, and never closes an open question — it is answered and the workflow state is
  otherwise left exactly as it was.

### 3.3. Mandate for Synchronous Interaction & Gated Execution
**MANDATE: The agent MUST NOT bypass interaction gates. Quality is achieved through user-in-the-loop validation.**
- **One Gate at a Time:** The agent is **ABSOLUTELY FORBIDDEN** from executing multiple gated steps in a single turn.
- **Stop and Present:** Upon reaching a "GATE" defined in any phase guide, the agent MUST stop all execution, present the required output for that step, and explicitly ask for user approval to proceed.
- **No Speculative Progress:** The agent must not begin work on Step N+1 until Step N has received explicit, unambiguous approval.
- **Iterative Elicitation:** For iterative loops (e.g., User Stories, Requirements, Test Identification), the elicitation mechanism depends on operating mode:
    - **Autonomous Mode:** elicit and process **one item at a time** — present the processed item, ask for the next, exit only when the user confirms no more items remain.
    - **Advisory Mode:** strict one-at-a-time dictation is a real, avoidable cost where each round-trip is a full message exchange, not a live interaction. Instead: **draft the full candidate batch in one pass, research-backed** (general and competitive research folded into the drafting itself, not bolted on after); **present it as a whole for bulk acceptance**; **run every item the agent could not just decide** — an assumption, a flagged item, an open question — **individually through research → full analysis and recommendation → the same in table form → a selectable-options prompt**, resolving each to exactly one of: Resolved, Deferred to a specific later point, Future Feature, or Rejected. A phase guide's own working-arrangement file (e.g. `CLAUDE.md`) elaborates this into its concrete named procedure (e.g. RCD/RATS) — the guarantee itself (every batch is research-backed; every non-obvious item is individually visible, researched, and resolved to a terminal outcome before being final) is what's binding here, not any particular procedure name.
    - **Completeness signal (either mode):** the user's explicit confirmation, never the agent's own assessment of coverage/variety/quality, closes the loop.
    - **Mechanical vs. substantive delegation (Advisory Mode):** an objective, checkable quality bar (e.g. a fixed set of criteria every requirement must satisfy) may be applied unsupervised while drafting a batch — it's mechanical, not a judgment call about the user's actual needs. This doesn't extend to what the criteria don't cover: necessity, correctness, real-world feasibility, or whether an assumption made to satisfy a mechanical criterion (e.g. inventing a missing detail to make an item "Complete") is actually correct. Those remain the user's call and must be flagged, not folded silently into the mechanical result.

---

## Appendix
See `CHANGELOG.md` for this file's full version history.
