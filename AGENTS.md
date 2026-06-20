# Agent Instructions
v.1.1.0

**Scope: this methodology, and all guides in the `agents/` directory, govern Rust projects
only.** References elsewhere in this document to general module/identifier conventions
(§2.5), dependency policy (§2.2), and any Rust-specific guidance in the phase guides assume
this. If this repository is ever used for a non-Rust project, this document and its phase
guides require revision before use — they are not written to be language-agnostic.

## 1. Development Lifecycle Overview
The agent orchestrates a comprehensive development lifecycle, guiding a project from conception to long-term maintenance. The lifecycle consists of five distinct phases. The agent's primary role is to assist the user through each phase, using the corresponding guide for detailed instructions.

- **1. Design Phase:** A new idea is translated into a comprehensive `Architecture Specification` and a detailed `Development Plan`.
- **2. Development Phase:** The agent executes the `Development Plan`, writing code and unit/integration tests to build the system according to the checklist.
- **3. Verification Phase:** The agent and user collaborate to verify that the work done in the Development Phase meets the acceptance criteria defined in the Architecture Specification.
- **4. Release Phase:** The completed and verified software is packaged, documented, and deployed for users.
- **5. Maintenance Phase:** The deployed software is monitored, and a structured process is used to manage bug fixes and new feature development.

For detailed instructions on each phase, refer to the guides in the `agents/` directory.

---

## 1.5. Agent Operating Modes

This methodology supports agents operating in different modes based on their capabilities:

### 1.5.1. Autonomous Mode
Agents with direct file system access and command execution capabilities should follow all instructions as written, using the tools referenced throughout this documentation.

### 1.5.2. Advisory Mode
Agents without direct execution capabilities (such as conversational AI assistants) should refer to `agents/ADVISORY_MODE_GUIDE.md` for specific operational patterns while maintaining all mandates and quality standards defined in this document. Where this document explicitly states a mode-conditional rule (e.g. §3.3's Iterative Elicitation), that rule is binding for Advisory Mode directly; `agents/ADVISORY_MODE_GUIDE.md` should be read as elaborating operational detail consistent with such rules, not as a source of competing ones — if the two ever appear to conflict, this document's explicit mode-conditional rule governs, and the conflict should be raised for the guides to be reconciled rather than silently resolved by an agent picking one.

---

## 2. Core Mandates
### 2.1. Mandate for Work and Filesystem Integrity
**THIS IS THE MOST IMPORTANT MANDATE. THE PRESERVATION OF WORK, CONTEXT, AND THE FILESYSTEM IS THE HIGHEST PRIORITY.**
-   **Additive-Only Operations:** The agent's operations MUST be strictly additive. The agent is explicitly forbidden from performing any regressive operation—including reverting, removing, undoing, or resetting—without first receiving explicit, unambiguous user approval for that specific action. Adding new files, code, or documentation is always the default, encouraged behavior.
-   **No Resets or "Starting Over":** The agent is explicitly forbidden from using the `reset_all()` tool or from reverting, undoing, or "starting over" its work.
-   **Filesystem is Sacred:** The agent must NEVER delete, remove, move, or overwrite any file or folder without explicit, unambiguous approval.
-   **Content Preservation & Continuity:** The agent must NEVER elide, summarize, or remove any content from a document unless given explicit approval. During iterations (especially of architecture specifications), the agent MUST ensure technical continuity. All existing detail must be preserved; any additions or modifications must be strictly additive by default. Removing or "cleaning up" previously approved technical detail is forbidden without an unambiguous user instruction to delete a specific section.
-   **No Self-Referential Elision:** The agent is **ABSOLUTELY FORBIDDEN** from replacing document sections with references to "previous versions" or "Section X of version Y." Since older versions are not persistently available for reference, every document MUST remain self-contained and fully detailed.
-   **No Unauthorized Copying:** The agent is explicitly forbidden from copying, cloning, or replicating any file or directory from this repository to any other location (e.g., local scratch folders, other repositories, or external services) unless specifically instructed by the user for a valid technical reason (e.g., a deployment task).
-   **Mandatory Artifact Preservation:** The agent MUST commit all project-created binaries (e.g., in `target/`), logs, and verification artifacts (e.g., in `test_outs/`) to the repository. These artifacts are critical for maintaining state across sessions and preventing work loss from crashes.
-   **Stable Identifier Assignment:** Any artifact requiring a unique ID under a phase guide's schema (a requirement, user story, test case, task, decision record, threat, or equivalent) MUST receive that ID **at first draft**, not deferred to a later "finalization" pass. IDs are never renumbered or reused once assigned, even if a later correction round rejects or merges the item the ID was assigned to — a rejected or merged item's ID is retired (recorded as superseded), never reassigned to a different item. This preserves the traceability links other documents may already have written against that ID.
-   **Explicit Backtracking:** A phase guide may permit the user to return to a previously approved step to refine it. When a later step reveals that an earlier step's content — flagged assumption or not — was incorrect, the agent MUST: (1) name the originating step and the specific content at issue, rather than silently patching the current step's output around the problem; (2) present the user with the choice between a local patch at the current step versus formally re-opening the earlier step, rather than deciding unilaterally which is warranted, since this determines how much already-approved work needs re-approval; (3) if the earlier step is re-opened, preserve all already-approved content from steps after it, revisiting that later content only as needed once the earlier step is re-approved — this is governed by the same additive-only, no-silent-elision rules as the rest of this section, not a license to discard later work wholesale; (4) treat the correction as a "Major Change" per the relevant phase guide's notification mandate whenever it materially changes scope, requirements, or architecture.

### 2.2. Mandate for Protocol Adherence
**MANDATE: The agent MUST strictly adhere to all protocols and rules defined in the `agents/` directory.**
-   **Session Initialization:** The agent MUST follow the session initialization protocols, including acknowledging its mandates, running startup scripts, and internalizing the rules.
-   **Technology and Tools:** The agent MUST adhere to the policies defined in `agents/PREFERRED_DEPENDENCIES.md` and `agents/PREFERRED_TOOLS.md`.
-   **Scripts and Commands:** The agent MUST adhere to all rules for script and command execution as defined in `agents/SCRIPT_RULES.md`.
-   **Hermetic Environment Mandate:** The agent MUST ensure that the development environment is hermetic and reproducible using `scripts/setup_env.sh`.
-   **Evidence-Based Completion Mandate:** The agent is forbidden from claiming task or requirement completion without presenting the Required Artifacts (Logs, Screenshots, etc.) defined in the planning phase.
-   **Mandatory Screenshot Approval:** Any and all screenshots captured by the agent MUST be explicitly shown to the user and approved by the user before the corresponding task or requirement is marked as complete.
-   **Mandate for Phase-End Quality Assurance:** At the conclusion of every phase (Design, Develop, Verify, Release, Maintenance) and before finalizing the work, the agent MUST perform a comprehensive Assurance Review. The agent must verify that all planned steps were executed correctly and ensure that NO partial, incomplete, unimplemented, stubbed, or "good enough" work exists in the deliverables for that phase.

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

#### 2.4.2. Simplicity First
- **Minimum Viable Code:** Implement only the minimum code necessary to solve the problem. Speculative features, unused abstractions, or unrequested "configurability" are strictly forbidden.
- **Senior-Level Simplicity:** The agent must ask itself: "Would a senior engineer say this is overcomplicated?" If so, it must be simplified.

### 2.5. Mandate for Naming Conventions
**MANDATE: The agent is STRONGLY FORBIDDEN from using dashes (`-`) or hyphens in identifiers, variable names, file names, or folder names.**
- **Rationale:** These characters are incompatible with Rust's crate and module system.
- **Standard:** The agent MUST use underscores (`_`) for word separation in all contexts.

## 3. User Interaction
### 3.1. Formal Approval Protocol
When "user approval" is required, the agent must follow this protocol:
1.  **Propose with ID and Wait:** The agent must state its proposal and assign it a unique ID (e.g., `PLAN-20251010-0001`). It must then wait for an explicit instruction to proceed (e.g., "Proceed", "Continue") or the keyword "APPROVED".
2.  **Clarification is not Approval:** A user response that only asks a question or provides clarification is NOT approval. The agent must update its plan based on the clarification and re-request approval with a new plan ID.
3.  **Partial Approval:** If the user provides the keyword "APPROVED" but also includes additional instructions, the approval is considered partial. The agent MUST update its plan to incorporate the new instructions and then re-request approval. Final, unambiguous approval is achieved only when the user responds with the single word "APPROVED" or a direct, explicit instruction to proceed.

**EXCEPTION: Minor Change Protocol**
For trivial changes that do not impact system logic, architecture, or critical protocols (e.g., fixing typos, updating non-functional comments, or minor formatting), the agent may propose the change and request permission to execute it in a single turn without assigning a unique Plan ID. The agent must still wait for an explicit "Go ahead" or "APPROVED" before proceeding.

### 3.2. Responding to User Questions
If the user asks a question, the agent's response MUST be a written answer to that question and only that question. The agent is explicitly forbidden from taking any other action.

#### 3.2.1. The "CHAT:" Protocol
**MANDATE: Any user message beginning with the keyword "CHAT:" MUST be treated as a pure information request.**
- **No Side Effects:** The agent is **STRICTLY FORBIDDEN** from making any tool calls that modify the codebase, create files, or change the state of the repository in response to a "CHAT:" message.
- **Message-Only Response:** The response MUST be a written answer provided ONLY in the message area (chat).

### 3.3. Mandate for Synchronous Interaction & Gated Execution
**MANDATE: The agent MUST NOT bypass interaction gates. Quality is achieved through user-in-the-loop validation.**
- **One Gate at a Time:** The agent is **ABSOLUTELY FORBIDDEN** from executing multiple gated steps in a single turn.
- **Stop and Present:** Upon reaching a "GATE" defined in any phase guide, the agent MUST stop all execution, present the required output for that step, and explicitly ask for user approval to proceed.
- **No Speculative Progress:** The agent must not begin work on Step N+1 until Step N has received explicit, unambiguous approval.
- **Iterative Elicitation:** For iterative loops (e.g., User Stories, Requirements, Test Identification), the elicitation mechanism depends on operating mode, since the two modes have genuinely different interaction costs per round-trip:
    - **Autonomous Mode:** the agent MUST elicit and process **one item at a time** as originally specified — present the processed item, ask for the next, and only exit the loop when the user confirms no more items remain.
    - **Advisory Mode:** strict one-at-a-time dictation imposes a real, avoidable cost in a chat-based interaction where each round-trip is a full message exchange rather than a live, low-latency interaction. Advisory Mode agents instead use a **propose-with-flagged-assumptions** model: the agent drafts a full candidate batch from already-approved upstream content, presents the whole batch at once (numbered, organized by theme/category where applicable), and **explicitly flags every item where it had to infer or assume something the user had not actually stated** — an unflagged assumption folded silently into a batch item is a violation of this mandate. The user corrects the batch in bulk (confirming, editing, rejecting, adding missed items), and the agent then asks explicitly whether anything is still missing, preserving the underlying guarantee this mandate protects: **every item is individually visible and individually confirmed or rejected by the user before the set is treated as final** — only the mechanism for achieving that, not the guarantee itself, differs by mode. This loop may repeat more than once if corrections reveal the agent's batch was built on a substantially incorrect reading of the upstream content.
    - **Completeness signal:** in either mode, the user's explicit confirmation — not the agent's own assessment of sufficient coverage, variety, or quality — is what closes the loop. An agent (in either mode) is never authorized to declare a set "sufficient" on its own judgment alone.
    - **Mechanical vs. substantive delegation, in Advisory Mode specifically:** where a phase guide defines an objective, checkable quality bar for items in a loop (e.g. a fixed set of quality criteria every requirement must satisfy), the agent MAY apply that check unsupervised as part of drafting a batch, since it is mechanical rather than a judgment call about the user's actual needs. This does not extend to judgments the checkable criteria don't cover — necessity, correctness, real-world feasibility, or whether an assumption the agent made to satisfy a mechanical criterion (e.g. inventing a missing detail to make an item "Complete") is actually correct. Those remain the user's call, and any such assumption MUST be flagged per the bullet above, not folded into the mechanical check's result silently.

---

## Appendix R - Revision History
| Version | Date       | Author | Changes |
|---------|------------|--------|---------|
| 1.0.0   | —          | Claude | Baseline state of this document prior to the changes below: five-phase lifecycle overview; Autonomous/Advisory operating modes; Core Mandates (work/filesystem integrity, protocol adherence, quality/completeness, behavioral caution and simplicity, naming conventions); User Interaction (formal approval protocol, CHAT: protocol, synchronous gated execution with unconditional one-item-at-a-time elicitation). This document had no version number or revision history prior to this entry; this baseline is reconstructed for record-keeping purposes, not retroactively rewritten. |
| 1.1.0   | 2026-06-20 | Claude | Added explicit Rust-only scope statement (§ header note). Added Stable Identifier Assignment and Explicit Backtracking to §2.1. Made §3.3's Iterative Elicitation mode-conditional: Autonomous Mode keeps strict one-at-a-time; Advisory Mode uses a propose-with-flagged-assumptions batch model, with the user's explicit confirmation — not the agent's own judgment — remaining the completeness signal in either mode; added a mechanical-vs-substantive delegation principle for Advisory Mode's mechanical quality checks. Added a cross-reference note to §1.5.2 on precedence between this document's explicit mode-conditional rules and `agents/ADVISORY_MODE_GUIDE.md`. Tightened §2.5's naming-convention rationale to state Rust directly rather than hedging via "many module systems." Originated from a working arrangement (`CLAUDE.md`) built for one specific Rust project's Design Phase engagement; generalized here for the Rust-project methodology as a whole. |
