# Preferred Development Tools

**MANDATE: Adherence to this guide is not optional. It is a strict requirement.**

This file governs **development tools** — binaries installed in the agent's environment — as distinct from `PREFERRED_DEPENDENCIES.md`, which governs Rust library dependencies compiled into the project. Tools listed here are preferred and may be used without additional approval. Tools not listed here require explicit user approval before use (per `AGENTS.md` §2.2).

See `CHANGELOG.md` for version history.

---

## Rust Toolchain

### Edition and MSRV Policy

- **Rust 2024 edition is mandatory** for all workspace crates. Specify `edition = "2024"` in every `Cargo.toml`.
- **The workspace MSRV is determined at Design Phase Step 5 (Verification Feasibility)**, not hardcoded here. The agent checks the actual MSRV of every tool and dependency the project commits to using, takes the maximum, and sets `rust-version = "<that version>"` in the workspace `Cargo.toml`. This value also sets the `channel` in `rust-toolchain.toml`.
- **Dual-MSRV pattern** (for projects with a publishable library crate): the workspace toolchain is pinned to the development MSRV (driven by tools like nextest and trunk); the publishable crate declares a separate, lower `rust-version` in its own `Cargo.toml`, enforced by a dedicated CI stage that installs that toolchain and compiles only that crate. These two values are independent — the workspace toolchain never needs to be downgraded to match the consumer MSRV.
- **Absolute tool-driven floors** (as of v0.1.0 of this file; always verify at Step 5 — these rise over time):
  - `cargo-nextest` (build): 1.91
  - `trunk` (latest): 1.81 (transitive deps may raise this — verify at Step 5)
  - `cargo-llvm-cov`: 1.82

### rust-toolchain.toml Template

```toml
[toolchain]
channel = "1.XX"   # Set at Design Step 5; must be >= max(tool floors above)
components = ["rustfmt", "clippy", "llvm-tools-preview"]
targets = ["wasm32-unknown-unknown"]   # Remove if no WASM component
```

### Pre-Flight Version Sanity Check

At the start of every development session, the agent MUST verify that installed tool versions are compatible with the project's declared toolchain and dependency matrix. The check runs before any build or task:

```bash
# Example — fill in project-specific expected versions
rustc --version          # must match rust-toolchain.toml channel
cargo --version
cargo nextest --version  # must be installable at workspace MSRV
trunk --version          # if project has WASM component
cargo llvm-cov --version
```

A version mismatch is an Escalation Trigger (Development Plan §13): the agent self-installs if a known-good version is available, appends the install command to `scripts/setup_env.sh` and `scripts/setup_env.bat`, informs the user, and then proceeds. If the mismatch cannot be self-resolved, it escalates without proceeding.

---

## Testing

### cargo-nextest
Next-generation test runner. **Preferred over `cargo test` for all projects.**
- Install: `cargo install cargo-nextest --locked`
- Run: `cargo nextest run --workspace`
- Build MSRV: 1.91 (verify at Step 5 — this rises over time)

### cargo-llvm-cov
**Preferred code coverage tool.** Replaces cargo-tarpaulin. Uses `-C instrument-coverage` (stable compiler flag) — does not depend on ptrace or compiler internals, making it robust across Rust version updates.
- Install: `cargo install cargo-llvm-cov --locked`
- Run: `cargo llvm-cov --workspace`
- Output formats: `--html`, `--lcov`, `--json`
- Requires `llvm-tools-preview` component in `rust-toolchain.toml`

---

## WASM

### trunk
**Preferred WASM web frontend build tool.** Fully preferred; no approval needed.
- Install: `cargo install trunk --locked`
- Serve: `trunk serve`
- Build: `trunk build --release`
- Requires Rust 1.81+ (verify transitive dep floor at Step 5)

### wasm-pack
Alternative WASM packaging tool. Use when targeting npm ecosystem or when `trunk` is not appropriate.
- Install: `cargo install wasm-pack --locked`

---

## Code Quality

### cargo-deny
Dependency license and advisory checking.
- Install: `cargo install cargo-deny --locked`
- Run: `cargo deny check`
- Config: `deny.toml` in repo root

### cargo-audit
Vulnerability scanning against the RustSec advisory database.
- Install: `cargo install cargo-audit --locked`
- Run: `cargo audit`

### cargo-watch
Automatic rebuild/retest on file changes. Useful during development.
- Install: `cargo install cargo-watch --locked`
- Run: `cargo watch -x "nextest run"`

---

## Database

### sqlx-cli
Required if the project uses SQLx for database access. Runs and checks migrations.
- Install: `cargo install sqlx-cli --no-default-features --features native-tls,postgres --locked`
  (adjust `--features` for the project's actual database backend)
- Run migrations: `sqlx migrate run`
- Check (compile-time query validation): `cargo sqlx prepare`

---

## Code Generation

### protoc
Protocol Buffers compiler. Required if the project uses Protobuf (e.g., with `prost`).
- Install: via system package manager (`apt install protobuf-compiler`, `brew install protobuf`, etc.)
  — not a cargo install; append system install command to `scripts/setup_env.sh`
- Verify: `protoc --version`

### cargo-expand
Expands Rust macros to their generated output. Useful for debugging procedural macros.
- Install: `cargo install cargo-expand --locked`
- Run: `cargo expand <module>`
- Requires nightly toolchain for full expansion: `cargo +nightly expand <module>`

---

## Dependency Management

### cargo-deny + cargo-audit
See Code Quality section above. Both are required CI gates for all projects (cross-reference Architecture Specification §6.4 and Development Plan §4).

---

## Embedded

> Applies primarily to embedded Rust projects (see `agents/PREFERRED_DEPENDENCIES.md`'s
> Embedded section for the corresponding library dependencies).

### espflash
Flashing and serial-monitor tool for ESP32/ESP-IDF targets.
- Install: `cargo install espflash --locked`
- Flash: `espflash flash target/<target-triple>/release/<binary>`
- Monitor: `espflash monitor`

---

## Missing Tool Protocol

When the agent encounters a missing tool at session start or during a task:

1. Attempt to install it using the command from this file (or the project's `scripts/setup_env.sh`).
2. Verify the install succeeded.
3. **Append the install command to `scripts/setup_env.sh` (Linux/bash) and `scripts/setup_env.bat` (Windows cmd)**, creating either file if it does not exist. Use idempotent check-before-install patterns:
   - bash: `command -v <tool> >/dev/null 2>&1 || cargo install <tool> --locked`
   - bat: check with `where <tool>` before installing
4. Inform the user that the tool was missing and has been installed (Escalation Trigger #4 still fires — inform then proceed).
5. If install fails, escalate without proceeding.
