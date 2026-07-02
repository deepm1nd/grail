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
and re-presented before handoff, not left as a partial subset. See `CLAUDE.md` §3.11 for the
concrete requirement, and §3.12 for the Step 7 backtrack workflow specifically.

### 1.5.3. Autonomy Toggle and Named Presets
A phase guide may permit reduced-elicitation ("autonomous") handling for specific steps —
silent internal process, but the gate and its output are unchanged — invoked per step or via
a named preset for a whole phase. This is **explicitly invoked, not a standing default.**
Autonomy changes only the visibility of process within the steps it touches; it never
removes a gate defined by the phase guide, and never applies to a step the phase guide
excludes from it. See the relevant phase guide and `CLAUDE.md` for the concrete per-step/
per-preset behavior.

---

## 2. Core Mandates
### 2.1. Mandate for Work and Filesystem Integrity
**THIS IS THE MOST IMPORTANT MANDATE. THE PRESERVATION OF WORK, CONTEXT, AND THE FILESYSTEM IS THE HIGHEST PRIORITY.**
-   **Additive-Only Operations:** The agent's operations MUST be strictly additive. The agent is explicitly forbidden from performing any regressive operation—including reverting, removing, undoing, or resetting—without first receiving explicit, unambiguous user approval for that specific action. Adding new files, code, or documentation is always the default, encouraged behavior.
-   **No Resets or "Starting Over":** The agent is explicitly forbidden from using the `reset_all()` tool or from reverting, undoing, or "starting over" its work.
-   **Filesystem is Sacred:** The agent must NEVER delete, remove, move, or overwrite any file or folder without explicit, unambiguous approval.
-   **Content Preservation & Continuity:** The agent must NEVER elide, summarize, or remove any content from a document unless given explicit approval. During iterations (especially of architecture specifications), the agent MUST ensure technical continuity. All existing detail must be preserved; any additions or modifications must be strictly additive by default. Removing or "cleaning up" previously approved technical detail is forbidden without an unambiguous user instruction to delete a specific section.
-   **No Self-Referential Elision:** The agent is **ABSOLUTELY FORBIDDEN** from replacing document sections with references to "previous versions" or "Section X of version Y." Since older versions are not persistently available for reference, every document MUST remain self-contained and fully detailed.
-   **No Unauthorized Copying or Duplication of Working Documents:** The agent is explicitly forbidden from copying, cloning, or replicating any file or directory from this repository to any other location (e.g., local scratch folders, other repositories, or external services) unless specifically instructed by the user for a valid technical reason (e.g., a deployment task). This extends specifically to project working documents during the Development Phase: the agent MUST edit the Development Checklist and other shared, in-place documents directly, never create a copy, rename, "v2," or otherwise reproduce a one-off version of any input file it has been given to work from. See `agents/DEVELOPMENT.md` §4 for the concrete rule.
-   **Mandatory Artifact Preservation:** The agent MUST commit all project-created binaries (e.g., in `target/`), logs, and verification artifacts (e.g., in `test_outs/`) to the repository. These artifacts are critical for maintaining state across sessions and preventing work loss from crashes.
-   **Stable Identifier Assignment:** Any artifact requiring a unique ID under a phase guide's schema (a requirement, user story, test case, task, decision record, threat, or equivalent) MUST receive that ID **at first draft**, not deferred to a later "finalization" pass. IDs are never renumbered or reused once assigned, even if a later correction round rejects or merges the item the ID was assigned to — a rejected or merged item's ID is retired (recorded as superseded), never reassigned to a different item. This preserves the traceability links other documents may already have written against that ID.
-   **Explicit Backtracking:** A phase guide may permit the user to return to a previously approved step to refine it. When a later step reveals that an earlier step's content — flagged assumption or not — was incorrect, the agent MUST: (1) name the originating step and the specific content at issue, rather than silently patching the current step's output around the problem; (2) present the user with the choice between a local patch at the current step versus formally re-opening the earlier step, rather than deciding unilaterally which is warranted, since this determines how much already-approved work needs re-approval; (3) if the earlier step is re-opened, preserve all already-approved content from steps after it, revisiting that later content only as needed once the earlier step is re-approved — this is governed by the same additive-only, no-silent-elision rules as the rest of this section, not a license to discard later work wholesale; (4) treat the correction as a "Major Change" per the relevant phase guide's notification mandate whenever it materially changes scope, requirements, or architecture. **Exception: findings from Step 7 (Spec Audit) are never handled via the local-patch-vs-reopen choice above — they always require reopening the originating step. See `CLAUDE.md` §3.12 for the concrete, mandatory Step 7 Backtrack Workflow.**

### 2.2. Mandate for Protocol Adherence
**MANDATE: The agent MUST strictly adhere to all protocols and rules defined in the `agents/` directory.**
-   **Session Initialization:** The agent MUST follow the session initialization protocols, including acknowledging its mandates, running startup scripts, and internalizing the rules.
-   **Technology and Tools:** The agent MUST adhere to the policies defined in `agents/PREFERRED_DEPENDENCIES.md` (Rust library dependencies), `agents/PREFERRED_TOOLS.md` (development tools), and `agents/PREFERRED_SERVICES.md` (infrastructure services).
-   **Scripts and Commands:** The agent MUST adhere to all rules for script and command execution as defined in `agents/SCRIPT_RULES.md`.
-   **Hermetic Environment Mandate:** The agent MUST ensure that the development environment is hermetic and reproducible using `scripts/setup_env.sh` (Linux) and `scripts/setup_env.bat` (Windows).
-   **Evidence-Based Completion Mandate:** The agent is forbidden from claiming task or requirement completion without presenting the Required Artifacts (Logs, Screenshots, etc.) defined in the planning phase.
-   **Mandatory Screenshot Approval:** Any and all screenshots captured by the agent MUST be explicitly shown to the user and approved by the user before the corresponding task or requirement is marked as complete.
-   **Mandate for Phase-End Quality Assurance:** At the conclusion of every phase (Design, Development) and before finalizing the work, the agent MUST perform a comprehensive Assurance Review. The agent must verify that all planned steps were executed correctly and ensure that NO partial, incomplete, unimplemented, stubbed, or "good enough" work exists in the deliverables for that phase.

### 2.3. Mandate for Quality and Completeness
**MANDATE: All work must be implemented to the fullest, most robust, and most complete potential.**
-   **Sole Responsibility:** The agent is solely responsible for the technical quality and completeness of its work. If information is insufficient, the agent MUST ask the user well-formed questions to elicit the required detail.
-   **Maximal Implementation:** Stub implementations, partial solutions, or "good enough" functionality are explicitly forbidden.
-   **Pre-Commit Documentation Integrity:** No commit shall be made until all relevant documentation, including checklists and handoff files, is verifiably up-to-date.

### 2.4. Mandate for Behavioral Caution and Simplicity
**MANDATE: The agent MUST prioritize caution and simplicity over speed.**

#### 2.4.1. Think Before Coding
- **No Assumptions:** The agent must never assume user intent. Confusions must be surfaced, and tradeoffs explicitly stated.
- **Stop on Confusion or Gates:** If a task is unclear, or if a defined "GATE" is reached, the agent MUST stop and ask for clarification or approval. Combining multiple gates into a single turn is forbidden.
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
**MANDATE: When the agent encounters a missing tool or prerequisite during any session,
it MUST NOT simply report the gap and stop.** Concretely, in this order:
1. Attempt to install the missing tool/prerequisite for the current session (per
   `agents/PREFERRED_TOOLS.md` and `agents/PREFERRED_SERVICES.md` where applicable).
2. **Append the install command(s) to `scripts/setup_env.sh`** (Linux/bash, assume a
   Linux-based environment as the default) **and `scripts/setup_env.bat`** (Windows),
   creating either file if it does not already exist. Appends MUST be idempotent —
   check whether the tool is already present before attempting to (re-)install it (e.g.
   `command -v <tool> >/dev/null 2>&1 || <install command>` in bash; an equivalent
   `where`-based check in the `.bat` file) — so re-running the script never redundantly
   reinstalls.
3. Inform the user that the prerequisite was missing and has been installed, per the
   relevant phase guide's Escalation Trigger for missing prerequisites — the escalation
   trigger still fires even on a successful self-install; it auto-resolves rather than
   being suppressed. The user is always told, not just silently routed around.
4. If the install cannot be completed (no known-good install path, permissions issue,
   etc.), the agent stops and escalates without proceeding — it does not guess at a
   workaround.

### 2.7. Mandate Against Reproducing Shared Working Documents
**MANDATE: The agent MUST use the Development Checklist and any other shared, in-place
project document exactly as provided — in place, by direct edit.** Creating a copy, a
renamed version, a "v2," or any other one-off reproduction of an input file the agent has
been given to work from is explicitly forbidden. This applies most concretely to the
Development Checklist (continuously edited in place across many sessions, per
`agents/DEVELOPMENT.md` and `agents/exemplars/development_checklist_template.md`), but the
principle extends to any file the workflow defines as a single, persistent, shared artifact
rather than a per-session deliverable.

### 2.8. Mandate for One Phase Per Development Session
**MANDATE: A Development Phase agent session completes at most one Development Plan phase.**
Even where session capacity or time remains after a phase's Exit Criteria is satisfied and
its Phase Summary is written, the agent MUST stop and await a new session rather than
beginning the next phase's tasks. This bounds the blast radius of any single session's
mistakes to one phase and keeps Phase Summaries meaningful as a per-session record. See
`agents/DEVELOPMENT.md` §5.2 for the concrete workflow this constrains.

## 3. User Interaction
### 3.1. Formal Approval Protocol
When "user approval" is required, the agent must follow this protocol:
1.  **Propose with ID and Wait:** The agent must state its proposal and assign it a unique ID (e.g., `PLAN-20251010-0001`). It must then wait for an explicit instruction to proceed (e.g., "Proceed", "Continue") or the keyword "APPROVED".
2.  **Clarification is not Approval:** A user response that only asks a question or provides clarification is NOT approval. The agent must update its plan based on the clarification and re-request approval with a new plan ID.
3.  **Partial Approval:** If the user provides the keyword "APPROVED" but also includes additional instructions, the approval is considered partial. The agent MUST update its plan to incorporate the new instructions and then re-request approval. Final, unambiguous approval is achieved only when the user responds with the single word "APPROVED" or a direct, explicit instruction to proceed.

**EXCEPTION: Minor Change Protocol**
For trivial changes that do not impact system logic, architecture, or critical protocols (e.g., fixing typos, updating non-functional comments, or minor formatting), the agent may propose the change and request permission to execute it in a single turn without assigning a unique Plan ID. The agent must still wait for an explicit "Go ahead" or "APPROVED" before proceeding.

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
- **Iterative Elicitation:** For iterative loops (e.g., User Stories, Requirements, Test Identification), the elicitation mechanism depends on operating mode — the two modes have genuinely different interaction costs per round-trip:
    - **Autonomous Mode:** elicit and process **one item at a time** — present the processed item, ask for the next, exit only when the user confirms no more items remain.
    - **Advisory Mode:** strict one-at-a-time dictation is a real, avoidable cost where each round-trip is a full message exchange, not a live interaction. Instead, **propose-with-flagged-assumptions**: draft a full candidate batch from already-approved upstream content, present it at once (organized by theme/category where applicable), and **explicitly flag every item where something was inferred rather than stated** — an unflagged assumption folded silently into a batch item violates this mandate. The user corrects the batch in bulk; the agent then asks explicitly whether anything is missing. The underlying guarantee — every item individually visible and individually confirmed or rejected before being final — is the same in both modes; only the mechanism differs. May repeat if corrections reveal the batch was built on a substantially incorrect reading.
    - **Completeness signal (either mode):** the user's explicit confirmation, never the agent's own assessment of coverage/variety/quality, closes the loop.
    - **Mechanical vs. substantive delegation (Advisory Mode):** an objective, checkable quality bar (e.g. a fixed set of criteria every requirement must satisfy) may be applied unsupervised while drafting a batch — it's mechanical, not a judgment call about the user's actual needs. This doesn't extend to what the criteria don't cover: necessity, correctness, real-world feasibility, or whether an assumption made to satisfy a mechanical criterion (e.g. inventing a missing detail to make an item "Complete") is actually correct. Those remain the user's call and must be flagged, not folded silently into the mechanical result.

---

## Appendix
See `CHANGELOG.md` for this file's full version history.
