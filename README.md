# grail: Agent Instruction Repository

Canonical instruction set governing AI software engineering agents on Rust projects.
Single source of truth — projects place a redirect file (`AGENTS.md`, from
`AGENTS_md.template`) pointing here; agents read this repo's actual content.

Rust-centric process covering the Design Phase and the Development Phase.

## How It Works
A project repo's `AGENTS.md` is a renamed copy of `AGENTS_md.template`, redirecting agents
to fetch real instructions from `deepm1nd/grail`. The project repo never needs updating when
these instruction files change.

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
1. **Design Phase (`agents/DESIGN.md`):** Concept → Architecture Specification,
   Development Plan, Development Checklist, through a 9-step gated workflow (independent
   Spec Audit at Step 7, independent Plan & Checklist Audit at Step 9).
2. **Development Phase (`agents/DEVELOPMENT.md`):** Executes the Development Plan phase by
   phase — one phase per agent session — following a build→test→evidence loop per task.

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
- **One phase per development session.**
- **Development agent escalation: stop, don't troubleshoot further.** A missing tool that
  installs cleanly is the sole case that continues automatically. Everything else the agent
  can't resolve itself — conflicts, failures, ambiguity — stops the entire session
  immediately: write the Phase Summary, no PR, no further progress. The human brings it to
  a Design Phase session to diagnose and restructure the Plan/Checklist.
- **Tracked UI/media assets.** User-supplied HTML/image/audio/video assets tracked by
  filename via an Asset Manifest, landing at fixed `assets/{html,images,audio,video}/`
  paths — never packaged into a Design-Phase handoff note, only referenced by filename.
- **Development agents write to one file.** The Development Checklist, and only within the
  agent's current phase — every other doc file is read-only to it.
- **Project README drafted at Design Step 8**, reviewed/confirmed/enhanced during
  Development Phase's Phase 0, not scaffolded from scratch there.
- **Preferred tooling.** Dependencies/tools/services drawn from
  `agents/PREFERRED_DEPENDENCIES.md`, `agents/PREFERRED_TOOLS.md`,
  `agents/PREFERRED_SERVICES.md`. Unlisted items require explicit approval.
- **ESP32/ESP-IDF projects** additionally consult `agents/ESP32_ESPIDF_RUST_BUILD_GUIDE.md`.
