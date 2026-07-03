# grail: Agent Instruction Repository

This repository contains the canonical instruction set and mandates that govern the behavior of AI software engineering agents on Rust projects. It is the single source of truth — projects that use it place a redirect file (renamed to `AGENTS.md`) in their own repo that points here; agents read this repo's actual content.

It is a Rust-centric software development process covering the Design Phase and the Development Phase.

## How It Works

A project repo contains a single `AGENTS.md` file that is a renamed copy of `AGENTS_md.template` from this repo. When an agent opens a project, it reads that file and is immediately directed to fetch the real instructions from `deepm1nd/grail`. The project repo never needs to be updated when these instruction files change — only `deepm1nd/grail` does.

## Repository Structure

```
grail/
├── AGENTS.md                        # Primary agent instruction file (read first)
├── CLAUDE.md                        # Design Phase working arrangement (Claude-specific)
├── AGENTS_md.template               # Redirect template — copy + rename to AGENTS.md in project repos
├── CHANGELOG.md                     # Full version history for all files (human reference only)
├── README.md                        # This file
└── agents/
    ├── DESIGN.md                    # Design & Planning Phase guide
    ├── DEVELOPMENT.md               # Development Phase guide
    ├── PREFERRED_DEPENDENCIES.md    # Mandated and forbidden Rust library dependencies
    ├── PREFERRED_TOOLS.md           # Approved development tools (cargo-nextest, trunk, etc.)
    ├── PREFERRED_SERVICES.md        # Approved infrastructure services (Postgres, Neo4j, etc.)
    ├── RUST_PREFERENCES.md          # Rust-specific design constraints (edition, MSRV, WASM, etc.)
    └── exemplars/
        ├── architecture_specification_template.md
        ├── development_plan_template.md
        ├── development_checklist_template.md
        └── dev_agent_prompt_template.md
```

## Development Phases

The agent workflow covers two phases:

1. **Design Phase (`agents/DESIGN.md`):** Translating a project concept into a comprehensive Architecture Specification, Development Plan, and Development Checklist through a 9-step gated workflow (including an independent Spec Audit at Step 7 and an independent Plan & Checklist Audit at Step 9).

2. **Development Phase (`agents/DEVELOPMENT.md`):** Executing the Development Plan phase by phase — one phase per agent session — following a strict build→test→evidence loop for every task.

## Key Mandates

- **Rust as the standard.** All core application logic is written in Rust. The 2024 edition is required; the workspace MSRV is determined per project at Design Step 5.
- **Gated execution.** Every design step ends with a STOP and explicit user approval before the next step begins. Each step runs in its own session.
- **Independent audits.** The Architecture Specification (Step 7) and the Development Plan/Checklist (Step 9) are each independently, adversarially audited before being relied upon — never self-certified by the same reasoning that produced them.
- **Additive-only operations.** Files and content are never deleted or overwritten without explicit approval. No "starting over."
- **Maximal implementation.** Stub or partial implementations are forbidden.
- **No cross-session memory.** Every session starts from what is actually on disk and in provided files — never from assumptions about what a prior session did.
- **One phase per development session.** A development agent completes at most one checklist phase per session.
- **Tracked UI/media assets.** User-supplied HTML, image, and audio/video reference assets are tracked by filename through an Asset Manifest and land at fixed, filename-stable paths (`assets/html/`, `assets/images/`, `assets/audio/`, `assets/video/`) in the repository for the development agent to consume directly.
- **Preferred tooling.** Dependencies, tools, and services are drawn from `agents/PREFERRED_DEPENDENCIES.md`, `agents/PREFERRED_TOOLS.md`, and `agents/PREFERRED_SERVICES.md`. Unlisted items require explicit approval before use.
