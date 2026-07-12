# [Project Name]

***[One-sentence description of what this project does.]***

[![Rust](https://img.shields.io/badge/rust-2024_edition-orange?logo=rust)](https://doc.rust-lang.org/edition-guide/rust-2024/index.html)
[![Coverage](https://img.shields.io/badge/dynamic/toml?url=https://raw.githubusercontent.com/[org]/[repo]/main/metrics/coverage.toml&query=coverage.percent&suffix=%25&label=coverage&color=brightgreen)](metrics/coverage.toml)
[![Tests](https://img.shields.io/badge/dynamic/toml?url=https://raw.githubusercontent.com/[org]/[repo]/main/metrics/tests.toml&query=tests.passed&label=tests%20passing&color=brightgreen)](metrics/tests.toml)
[![Security Audit](https://img.shields.io/badge/dynamic/toml?url=https://raw.githubusercontent.com/[org]/[repo]/main/metrics/audit.toml&query=audit.status&label=security%20audit)](metrics/audit.toml)
[![License Check](https://img.shields.io/badge/dynamic/toml?url=https://raw.githubusercontent.com/[org]/[repo]/main/metrics/deny.toml&query=deny.status&label=license%20check)](THIRD_PARTY_LICENSES.md)
[![CI](https://github.com/[org]/[repo]/actions/workflows/ci.yml/badge.svg)](https://github.com/[org]/[repo]/actions/workflows/ci.yml)
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
├── assets/                     # Tracked UI/media assets
│   ├── html/
│   ├── images/
│   ├── audio/
│   └── video/
├── deploy/                     # Infra service compose files & config
│   ├── docker-compose.yml      # Project-as-container (if applicable)
│   └── docker-compose.dev.yml  # Dev/test infra dependencies
├── metrics/                    # Coverage/test/audit/deny/playwright TOML — CI-maintained
├── scripts/
│   ├── setup_env.sh / .bat     # Idempotent tool/dependency installer
│   ├── check_env.sh            # Read-only pre-flight verifier
│   └── metrics/                # Parsers feeding metrics/*.toml
├── test/                       # Per-phase verification files & evidence
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
  infra services (`deploy/docker-compose.dev.yml`).
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
   docker compose -f deploy/docker-compose.dev.yml up -d
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

The badges at the top of this file read live from `metrics/*.toml` via shields.io's
[dynamic-toml-badge](https://shields.io/badges/dynamic-toml-badge). These files are
regenerated by CI on every push to `main` (`agents/CI.md` Stage 8) — do not hand-edit them.

| File | Feeds |
|---|---|
| `metrics/coverage.toml` | Coverage badge |
| `metrics/tests.toml` | Tests-passing badge |
| `metrics/audit.toml` | Security-audit badge |
| `metrics/deny.toml` | License-check badge |
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
