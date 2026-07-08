# grail: Agent Instruction Repository

Canonical instruction set governing AI software engineering agents on Rust projects.
Single source of truth — projects place a redirect file (`AGENTS.md`, from
`AGENTS_md.template`) pointing here; agents read this repo's actual content.

Rust-centric process covering the Design Phase and the Development Phase, run by two
different agents: **Design Phase sessions are Claude-based** (`CLAUDE.md` is Claude's
working arrangement for that phase); **Development Phase sessions are Jules-based** (Jules
executes the Development Plan the Design Phase produced, one phase per session).

## How It Works
A project repo's `AGENTS.md` is a renamed copy of `AGENTS_md.template`, redirecting agents
to fetch real instructions from `deepm1nd/grail`. The project repo never needs updating when
these instruction files change.

## Workflow for Humans

1. **Start a Design Phase session with Claude**, in the target project's repo, having it
   read `AGENTS.md` (which redirects here) and `CLAUDE.md`. Work through
   `agents/DESIGN.md`'s 9 steps **one step per session** — Concept Intake, User Stories,
   Requirement Decomposition, Questioning & Test ID, the Decomposition Gate, Architecture
   Synthesis, an independent Spec Audit (Step 7), Development Plan & Checklist Generation
   (Step 8), and an independent Plan & Checklist Audit (Step 9).
2. **Approve each step explicitly before the next begins.** Every step ends with a STOP and
   a summary; you reply with the exact token `APPROVED` (no "Continue"/"Proceed"
   substitutes) to advance. This is a deliberate gate, not a formality — each step's output
   becomes load-bearing for everything after it.
3. **Step 8's output is the handoff package:** the Development Plan (with a Branch Name and
   Session Unit declared per phase), the Development Checklist, the reusable Dev Prompt
   (`[projectname]_dev_prompt.md`), and a draft `README.md`. Step 9 independently audits
   this package before you move on — any finding sends you back to Step 8, never patched in
   place at Step 9.
4. **Hand the project to Jules for the Development Phase.** Give Jules the Dev Prompt at the
   start of every session — it's reused verbatim, not regenerated. Each session checks out
   the current phase's branch, works exactly one Session Unit (a full Phase by default, or a
   single Task/Code+Verify sub-task if the Plan declared a finer unit), submits at every
   task's declared Submit Point, and stops — you say "Continue" to start the next session.
5. **Review evidence as it accumulates**, not just at the end: each phase's
   `test/[projectname]_phase_[N]_verification.md` (build/test summary lines, screenshots,
   clips) and `[projectname]_phaseN_summary.md` (the narrative — what happened, deviations,
   issues). A session that hits something it can't resolve stops immediately and writes its
   Phase Summary instead of guessing — bring that back to a **Design Phase session with
   Claude** to diagnose and restructure the Plan/Checklist, then hand a fresh Dev Prompt back
   to Jules to resume.
6. **Repeat step 4–5 until every phase's Exit Criteria is checked.** Development then stops
   and awaits your instruction — including, if you want it, the optional post-development
   remediation cycle (`agents/DEVELOPMENT.md` §5.4), triggered only when you explicitly ask
   for an audit.

## Repository Structure
```
grail/
├── AGENTS.md                        # Primary agent instruction file (read first)
├── CLAUDE.md                        # Design Phase working arrangement (Claude-specific)
├── AGENTS_md.template               # Redirect template
├── CHANGELOG.md                     # Full version history (human reference only)
├── README.md                        # This file
└── agents/
    ├── DESIGN.md                    # Design & Planning Phase guide
    ├── DEVELOPMENT.md                # Development Phase guide
    ├── PREFERRED_DEPENDENCIES.md     # Mandated/forbidden Rust library dependencies
    ├── PREFERRED_TOOLS.md            # Approved development tools
    ├── PREFERRED_SERVICES.md         # Approved infrastructure services
    ├── RUST_PREFERENCES.md           # Rust-specific design constraints
    ├── ESP32_ESPIDF_RUST_BUILD_GUIDE.md  # ESP32/ESP-IDF only — conditionally referenced
    └── exemplars/
        ├── architecture_specification_template.md
        ├── development_plan_template.md
        ├── development_checklist_template.md
        └── dev_prompt_template.md
```

## Development Phases
1. **Design Phase (`agents/DESIGN.md`, Claude):** Concept → Architecture Specification,
   Development Plan, Development Checklist, through a 9-step gated workflow (independent
   Spec Audit at Step 7, independent Plan & Checklist Audit at Step 9).
2. **Development Phase (`agents/DEVELOPMENT.md`, Jules):** Executes the Development Plan
   phase by phase — one Session Unit per agent session, on that phase's own branch —
   following a build→test→evidence loop per task.

## Key Mandates
- **Rust as the standard.** 2024 edition required; workspace MSRV set per project at
  Design Step 5.
- **Gated execution.** Every design step ends with a STOP and exact `APPROVED` before the
  next step begins — no "Continue"/"Proceed" substitutes. Each step runs in its own session.
- **One fixed procedure, every step but 7 and 9: RCD → bulk-accept → RATS.** Core content is
  drafted in one research-backed batch (Research, Competitive research, Draft), bulk-accepted,
  and every residual assumption/flagged item runs through Research-Analysis-Table-
  Selectable-options, resolving to exactly one of Resolved / Deferred-to-a-step / Future
  Feature / Rejected. No autonomy toggle, no Tailored Mode — Step 2 (User Stories) always
  runs in full regardless of project size.
- **Independent audits.** Step 7 and Step 9 are pure finders, never fixers — no RCD/RATS.
  Any finding requires reopening the originating step (full multi-session workflow, or a
  user-invoked **Surgical Fix Override** — a compressed single-session equivalent that still
  mandatorily ends in a fresh audit session).
- **Additive-only operations.** No deletion/overwrite without explicit approval.
- **Maximal implementation.** No stubs or partial implementations.
- **No cross-session memory.** Every session starts from what's actually on disk/provided.
- **One Session Unit per development session** — a full Phase by default, or a finer
  Task/Code+Verify unit where the Plan declares one; never more than one per session.
- **Per-phase branch, per-task submit.** Each phase gets its own human-readable Branch Name
  (assigned at Design Step 8); every task submits immediately on DoD completion — never
  batched to end-of-phase — bounding crash blast-radius to the task in flight.
- **Development agent escalation: stop, don't troubleshoot further.** A missing tool that
  installs cleanly is the sole case that continues automatically. Everything else the agent
  can't resolve itself — conflicts, failures, ambiguity — stops the entire session
  immediately: write the Phase Summary, no PR, no further progress. The human brings it to
  a Design Phase session to diagnose and restructure the Plan/Checklist.
- **Tracked UI/media assets.** User-supplied HTML/image/audio/video assets tracked by
  filename via an Asset Manifest, landing at fixed `assets/{html,images,audio,video}/`
  paths — never packaged into a Design-Phase handoff note, only referenced by filename.
- **Per-phase Verification file.** `test/[projectname]_phase_[N]_verification.md` — concise
  build/test summary lines and pointers to `test/phase_[N]/` screenshots/clips — evidence,
  not narrative; the Phase Summary links to it rather than repeating it.
- **Development agents write to one file, bracket-content-only.** The Development Checklist,
  and only within the agent's current phase: flipping a mark in an existing `[ ]`, checking
  a `Submitted` box, or appending a Session Log row — never rewording, restructuring, or
  touching another phase's marks. Every other doc file is read-only to it.
- **Project README drafted at Design Step 8**, reviewed/confirmed/enhanced during
  Development Phase's Phase 0, not scaffolded from scratch there.
- **Preferred tooling.** Dependencies/tools/services drawn from
  `agents/PREFERRED_DEPENDENCIES.md`, `agents/PREFERRED_TOOLS.md`,
  `agents/PREFERRED_SERVICES.md`. Unlisted items require explicit approval.
- **ESP32/ESP-IDF projects** additionally consult `agents/ESP32_ESPIDF_RUST_BUILD_GUIDE.md`.
