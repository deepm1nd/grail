# [Project Name]

***[One-sentence description of what this project does.]***

[![Rust](https://img.shields.io/badge/rust-2024_edition-orange?logo=rust)](https://doc.rust-lang.org/edition-guide/rust-2024/index.html) <!-- BADGE:coverage:START -->[![Coverage](https://img.shields.io/badge/coverage-pending-lightgrey)](metrics/coverage.toml)<!-- BADGE:coverage:END --> <!-- BADGE:tests:START -->[![Tests](https://img.shields.io/badge/tests-pending-lightgrey)](metrics/tests.toml)<!-- BADGE:tests:END --> <!-- BADGE:audit:START -->[![Security Audit](https://img.shields.io/badge/security_audit-pending-lightgrey)](metrics/audit.toml)<!-- BADGE:audit:END --> <!-- BADGE:deny:START -->[![License Check](https://img.shields.io/badge/license_check-pending-lightgrey)](THIRD_PARTY_LICENSES.md)<!-- BADGE:deny:END --> <!-- BADGE:license_violations:START -->[![License Violations](https://img.shields.io/badge/license_violations-pending-lightgrey)](metrics/license_violations_log.jsonl)<!-- BADGE:license_violations:END --> <!-- BADGE:crates_added:START -->[![Crates Added](https://img.shields.io/badge/crates_added-pending-lightgrey)](metrics/license_additions_log.jsonl)<!-- BADGE:crates_added:END --> <!-- BADGE:source:START -->[![Branch Status](https://img.shields.io/badge/latest_push-pending-lightgrey)](metrics/source.toml)<!-- BADGE:source:END --> [![CI](https://github.com/[org]/[repo]/actions/workflows/ci.yml/badge.svg)](https://github.com/[org]/[repo]/actions/workflows/ci.yml)
[![License: PolyForm Noncommercial](https://img.shields.io/badge/license-PolyForm%20Noncommercial-blue)](LICENSE.md)

> **License note:** this project is **source-available**, not open-source in the OSI sense.
> Free for personal/noncommercial use; commercial use requires a separate license from the
> author. See [License](#license) below.

<img src="./assets/images/main_screenshot.png" alt="[Project Name] Screenshot" width="100%"/>

## Table of Contents

* [About](#about)
* [Key Features](#key-features)
* [Quick Start](#quick-start)
  * [Repository Layout](#repository-layout)
  * [Prerequisites](#prerequisites)
  * [Installation](#installation)
  * [Configuration](#configuration)
* [Tech Stack](#tech-stack)
* [Architecture](#architecture)
* [Metrics & Badges](#metrics--badges)
* [Development Workflow](#development-workflow)
* [Contributing](#contributing)
* [Security](#security)
* [License](#license)

## About

[One paragraph describing the project's purpose and what problem it solves.]

## Key Features

* **[Feature 1]:** [One-line description.]
* **[Feature 2]:** [One-line description.]
* **[Feature 3]:** [One-line description.]

## Quick Start

### Repository Layout

```
.
├── .github/workflows/          # CI pipeline (ci.yml)
├── src/                        # Single-crate project source
│                                # — mutually exclusive with crates/, decided at Design Step 5
├── crates/                     # Multi-crate workspace members (if applicable)
│   └── [crate_name]/
│       └── src/
├── tests/                      # Rust's own standard integration-test source (Cargo-compiled
│                                # `tests/*.rs` binaries) — NOT the same thing as `test/`
│                                # (singular) below, which is grail's own process/evidence
│                                # tree and is never compiled. See `agents/PREFERRED_TOOLS.md`.
├── dev/                         # grail-generated process docs (Design/Maintenance/Release)
│   ├── spec/                    # Architecture Specification files
│   ├── plan/                    # Development Plan, Checklist, Dev Prompt files
│   │   ├── [projectname]_dev_risks.md          # Development-Phase risk log
│   │   └── v0.0.1/summary/       # (v0.12.11) Task Group Summaries — fixed at v0.0.1 for the
│   │       └── [projectname]_task_group_[ID]_summary.md  # whole Dev Phase (test/v0.0.1/'s
│   │                             # own reasoning: no real version exists yet). [ID] is the
│   │                             # Task Group's [PREFIX]-NNNN identifier (§6.1) — this,
│   │                             # not the version folder, is what keeps a Dev summary and a
│   │                             # same-version Maintenance-batch summary (below) distinct.
│   ├── maintenance/
│   │   ├── v[N.NN.NN]/           # one folder per Maintenance batch — the FIRST batch a
│   │   │                         # project ever opens is always v0.0.1 by convention; every
│   │   │                         # later batch is user/batch-determined at open time
│   │   └── (Task Group Summaries for a batch live at dev/plan/v[N.NN.NN]/summary/, i.e.
│   │       alongside the Dev Phase ones above, not under this maintenance/ folder — see
│   │       agents/MAINTENANCE.md)
│   └── release/
│       └── v[N.NN.NN]/           # one folder per RELEASE-Phase pass — v0.0.1 or higher,
│                                 # user-determined at open time, never assumed
├── assets/                     # Tracked UI/media assets
│   ├── html/
│   ├── images/
│   ├── audio/
│   └── video/
├── deploy/                     # Project-as-container deploy config (if applicable)
│   └── docker-compose.yml      # The project's own packaged service(s), production-facing
├── metrics/                    # Coverage/test/audit/deny/playwright TOML — CI-maintained
├── scripts/
│   ├── setup_env.sh / .bat     # Idempotent tool/dependency installer
│   ├── check_env.sh / .bat     # Read-only pre-flight verifier
│   ├── run_ci.sh / .bat        # (v0.12.11) Local CI-equivalent run (`agents/CI.md` §6)
│   └── metrics/                # Parsers feeding metrics/*.toml
├── test/                       # grail's own process/evidence tree — Markdown verification &
│                                # traceability artifacts ONLY. NOT Rust integration-test
│                                # source (that's `tests/`, above) — never compiled, never
│                                # run by Cargo. Nothing is ever written loose at this root;
│                                # only the entries below (plus their own future subfolders)
│                                # are permitted here.
│   ├── containers/
│   │   └── docker-compose.dev.yml   # Dev/test infra dependencies (Postgres, Neo4j, ...)
│   ├── scripts/                 # Scripts/fixtures not part of the codebase itself
│   │   └── fixtures/             # Test input files, golden outputs, sample payloads
│   ├── v[N.NN.NN]/               # Per-version results. Development Phase's own folder is
│   │                             # FIXED at v0.0.1 for the entire phase (never increments
│   │                             # per Task Group, never tied to a real project version —
│   │                             # no real version exists yet). Maintenance/Release folders
│   │                             # are real batch/release versions, v0.0.1 or higher,
│   │                             # user-determined at open time.
│   │   ├── [projectname]_task_group_[N]_verification.md  # Verification evidence per Task Group
│   │   └── task_group_[N]/       # Screenshots/clips for that Task Group's Verification entries
│   └── [projectname]_requirement_traceability.md  # Requirement-to-test traceability — one
│                                                   # cumulative file for the project's whole life,
│                                                   # not version-scoped like the folders above
├── web/
│   ├── site/     # the website itself — home/landing, docs (mdBook), blog, community, ...
│   │   ├── home/       # landing page — folded into default scope, not a rare override
│   │   ├── docs/       # the mdBook project itself
│   │   ├── blog/
│   │   ├── community/
│   │   ├── ...         # playground/, showcase/, etc. — only what this project chose
│   │   └── [reference surfaces]  # api/, cli/, config/, proto/, etc. — see RELEASE.md §5
│   ├── press/    # media kit: brand assets, fact sheet, press releases, contact
│   ├── legal/    # privacy policy, ToS, security.txt/disclosure policy
│   └── social/   # social media content/templates
├── Cargo.toml
├── rust-toolchain.toml
├── deny.toml
├── THIRD_PARTY_LICENSES.md     # Auto-regenerated, drift-checked in CI
└── LICENSE.md
```

### Prerequisites

* **Rust**, installed via [rustup](https://rustup.rs/) — the pinned toolchain in
  `rust-toolchain.toml` will be selected automatically.
* **[cargo-nextest](https://nexte.st/)**, **[cargo-llvm-cov](https://github.com/taiki-e/cargo-llvm-cov)**,
  **[cargo-deny](https://github.com/EmbarkStudios/cargo-deny)**,
  **[cargo-audit](https://github.com/rustsec/rustsec)** — installed automatically by
  `scripts/setup_env.sh` if missing.
* **[Docker](https://www.docker.com/) / Docker Compose** — only if this project uses
  infra services (`test/containers/docker-compose.dev.yml`).
* [Trunk](https://trunkrs.dev/) — only if this project has a WASM/web-frontend component.

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/[org]/[repo].git
   cd [repo]
   ```
2. **Verify your environment, then install anything missing:**
   ```bash
   bash scripts/check_env.sh   # reports what's missing
   bash scripts/setup_env.sh   # installs it, idempotently
   ```
3. **Start infra services, if this project uses any:**
   ```bash
   docker compose -f test/containers/docker-compose.dev.yml up -d
   ```
4. **Build and run:**
   ```bash
   cargo build
   cargo run
   ```

### Configuration

[Environment variables / config file locations, if any. Never commit real secrets —
`deploy/` config files use placeholder values only.]

```bash
# Example — adjust to the project's actual configuration
DATABASE_URL="postgres://dev:dev@localhost:5432/[projectname]_dev"
```

## Tech Stack

| Category | Choice | Reference |
|---|---|---|
| Language | Rust (2024 edition) | `agents/PREFERRED_DEPENDENCIES.md` |
| Test runner | cargo-nextest | `agents/PREFERRED_TOOLS.md` |
| Coverage | cargo-llvm-cov | `agents/PREFERRED_TOOLS.md` |
| License/advisory scan | cargo-deny | `agents/PREFERRED_TOOLS.md` |
| Vulnerability scan | cargo-audit | `agents/PREFERRED_TOOLS.md` |
| CI | GitHub Actions | `agents/CI.md` |
| [Database, if applicable] | [PostgreSQL / Neo4j / Qdrant / etc.] | `agents/PREFERRED_SERVICES.md` |
| [WASM frontend, if applicable] | Trunk | `agents/PREFERRED_TOOLS.md` |

## Architecture

[Link to or summarize the Architecture Specification. A diagram may go here if useful —
keep it accurate to the as-built system, not the original Design-time proposal, once they
diverge.]

## Metrics & Badges

The badges at the top of this file are **static** (shields.io's plain
`label-message-color` badge type, not `dynamic/toml`) and are **written directly into
this file by CI's Metrics Commit step** (`agents/CI.md` Stage 6) on every push — never
hand-edit them. `metrics/*.toml` are regenerated first, then
`scripts/metrics/write_readme_badges.js` reads them and replaces the content between each
`<!-- BADGE:X:START -->`/`<!-- BADGE:X:END -->` marker pair above with a freshly built
badge line.

**(v0.12.10 — corrected from v0.12.9) All badges MUST stay on one shared physical line,
led by a badge whose Markdown does not start with an HTML comment (e.g. the Rust badge
first).** GFM/CommonMark's HTML-block rule triggers only when a *line itself begins with*
`<!--`: such a line becomes a raw HTML block in full, breaking it out of the surrounding
paragraph — which is what caused the v0.12.9 three-line fix to correctly stop badges
rendering as inert text, but at the cost of pushing every badge onto its own line/paragraph
(each `<!-- BADGE:X:START -->` line began with `<!--`, so each became its own block). The
real, narrower rule: a `<!-- -->` comment appearing **mid-line**, after some non-comment
content already opened that line, is just inert inline HTML — it neither breaks rendering
nor splits the paragraph. So the correct, tested layout is the **original one-line
form**, `<!-- BADGE:X:START -->[![...]](...)<!-- BADGE:X:END -->`, but with the *entire*
badge sequence kept on one physical line that starts with a plain (non-comment-prefixed)
badge first:

```
[![Rust](...)](...) <!-- BADGE:coverage:START -->[![Coverage](...)](...)<!-- BADGE:coverage:END --> <!-- BADGE:tests:START -->[![Tests](...)](...)<!-- BADGE:tests:END --> ...
```

`write_readme_badges.js`'s marker-matching regex operates on this single line — same-line
`START...content...END` matching per marker, as originally specified — but the script must
never insert a newline before the first marker or between subsequent markers; the whole
badge row is one paragraph, one line, always.

This mechanism exists specifically because the old `dynamic/toml` badge type required
shields.io's server to externally fetch `raw.githubusercontent.com/.../metrics/*.toml` —
which silently fails (an "invalid" badge) on any private repository, with no
authentication mechanism available to fix it. A static badge needs no external fetch at
all, so it works identically on public and private repos. Because the badge value is
written in-repo rather than fetched, there is also no branch-name substitution to
maintain — the old `[current_branch]` placeholder and its per-Task-Group update step no
longer apply.

| File | Feeds |
|---|---|
| `metrics/coverage.toml` | Coverage badge |
| `metrics/tests.toml` | Tests-passing badge |
| `metrics/audit.toml` | Security-audit badge |
| `metrics/deny.toml` | License-check badge |
| `metrics/licenses.toml` | License-violations and crates-added badges |
| `metrics/source.toml` | Latest-push branch/status badge |
| `metrics/playwright.toml` | (if this project has an E2E suite) |

## Development Workflow

This project follows the `grail` Design/Development Phase workflow
(`https://github.com/deepm1nd/grail`) — see `AGENTS.md` for the canonical instruction set.
In short: CI (`.github/workflows/ci.yml`) is an **async, human-reviewed backstop**, not a
merge gate — see `agents/CI.md` for what a red check does and doesn't mean at various
points in the project's life.

## Contributing

[Contribution guidelines, if this project accepts external contributions. Given the
default license below, clarify what contribution implies about licensing terms.]

## Security

If you discover a security vulnerability, please email **[security-contact@example.com]**
rather than opening a public issue.

## License

This project is licensed under the **[PolyForm Noncommercial License 1.0.0](LICENSE.md)** —
free for personal and noncommercial use; **commercial use requires a separate license from
the author.** Contact **[license-contact@example.com]** for commercial licensing inquiries.

Third-party dependency licenses are disclosed in [`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md),
auto-regenerated and drift-checked by CI (`agents/CI.md` Stage 5) — every dependency in
this project is permissively licensed (MIT/Apache-2.0/BSD/ISC/Unicode-3.0 family, see
`agents/PREFERRED_DEPENDENCIES.md`'s License Compatibility Criterion) and imposes no
obligation on this project beyond preserving its own notice.

## Acknowledgements

* [Shields.io](https://shields.io/) — dynamic badges.
* [`agents/CI.md`, `agents/PREFERRED_TOOLS.md`, `agents/PREFERRED_DEPENDENCIES.md`] — the
  tooling and dependency conventions this project follows.
