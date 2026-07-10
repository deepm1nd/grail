# Preferred Development Tools

**MANDATE: Adherence to this guide is not optional. It is a strict requirement.**

Governs **development tools** (binaries in the agent's environment), distinct from
`PREFERRED_DEPENDENCIES.md` (Rust library dependencies). Listed tools are usable without
approval; unlisted tools require approval (`AGENTS.md` §2.2).

See `CHANGELOG.md` for version history.

---

## Rust Toolchain

- **Rust 2024 edition mandatory**, `edition = "2024"` in every `Cargo.toml`.
- **Workspace MSRV set at Design Step 5** — check the actual build-MSRV of every
  committed tool/dependency, take the maximum, set `rust-version` in workspace
  `Cargo.toml` and `channel` in `rust-toolchain.toml`. **Preferred floor: 1.94.0** (the dev
  agent environment's pre-installed default, `RUST_PREFERENCES.md` §0) — use it whenever
  the computed value would otherwise be lower; a higher computed value still wins.
- **Dual-MSRV pattern** (publishable library crate): workspace toolchain pinned to dev
  MSRV; the publishable crate declares its own lower `rust-version`, enforced by a
  dedicated CI stage compiling only that crate.
- **Tool-driven floors** (verify at Step 5 — rise over time): `cargo-nextest` 1.91,
  `trunk` 1.81, `cargo-llvm-cov` 1.82.

```toml
# rust-toolchain.toml
[toolchain]
channel = "1.XX"   # Set at Design Step 5
components = ["rustfmt", "clippy", "llvm-tools-preview"]
targets = ["wasm32-unknown-unknown"]   # Remove if no WASM component
```

**Pre-flight version sanity check**, every session, before any build:
```bash
rustc --version
cargo --version
cargo nextest --version
trunk --version        # if WASM component
cargo llvm-cov --version
```
Mismatch = Escalation Trigger (`agents/exemplars/development_plan_template.md` §13):
self-install if a known-good version exists, append to `scripts/setup_env.sh`/`.bat`,
inform the user, proceed. Otherwise: stop the session per `agents/DEVELOPMENT.md` §4's
escalation model — do not attempt further troubleshooting.

---

## Testing

### cargo-nextest
Preferred test runner. Install: `cargo install cargo-nextest --locked`. Run:
`cargo nextest run --workspace`.

### cargo-llvm-cov
Preferred coverage tool (stable `-C instrument-coverage`, robust across Rust upgrades).
Install: `cargo install cargo-llvm-cov --locked`. Requires `llvm-tools-preview`.

---

## WASM

### trunk
**Preferred WASM web frontend build tool.** Install: `cargo install trunk --locked`.
Serve: `trunk serve`. Build: `trunk build --release`.

**Output-directory separation — mandatory.** Trunk's build output (default `dist/`,
which itself manages its own hashed-asset subtree) is a **distinct, unrelated
directory** from the project's tracked `assets/{html,images,audio,video}/` source
directory (`agents/exemplars/development_plan_template.md` §3 — the reference
HTML/image/audio/video assets developers populate). Never configure Trunk's `dist`
output to write into or under the project's root `assets/`. Pin explicitly:

```toml
# Trunk.toml
[build]
dist = "dist"   # Never "assets" or a path under assets/
```

**Note:** `Trunk.toml` lives at the repository root, not inside a project-name subfolder —
see `agents/exemplars/development_plan_template.md` §3: the project root *is* the repo
root, unnamed, since a clone lands in whatever directory name the clone tool assigns.

**`wasm-bindgen` is managed automatically by Trunk** — it downloads the exact CLI
version matching the `wasm-bindgen` crate pinned in `Cargo.lock`. **Never install
`wasm-bindgen-cli` separately** in a Trunk project: a separately-installed version will
almost always differ from the `Cargo.lock`-pinned crate version (CLI and crate must
match to the exact patch version), causing a build failure of the form *"linked against
a different version of wasm-bindgen than this binary."* `wasm-bindgen-cli` must never
appear in `scripts/setup_env.sh` for a Trunk project. See Forbidden Tools below.

### wasm-pack
Use only for **npm-ecosystem publishing** or when Trunk isn't appropriate for the
project's output type. Never alongside Trunk in the same project — different
deployment targets, conflicting pipelines. Install: `cargo install wasm-pack --locked`.

### getrandom on wasm32-unknown-unknown — mandatory backend config
`getrandom` 0.3+ no longer auto-selects a WASM backend — anything pulling it in
transitively (`rand`, `uuid`, both in `PREFERRED_DEPENDENCIES.md`) fails to compile for
`wasm32-unknown-unknown` without **both** of the following set explicitly (the feature
alone is not sufficient):

```toml
# Cargo.toml — the wasm-targeted crate (or workspace-wide if only one wasm target exists)
[target.'cfg(target_arch = "wasm32")'.dependencies]
getrandom = { version = "0.3", features = ["wasm_js"] }
```

```toml
# .cargo/config.toml
[target.wasm32-unknown-unknown]
rustflags = ["--cfg", "getrandom_backend=\"wasm_js\""]
```

Missing either half surfaces as a `getrandom`/`backends` compile error only on the WASM
target — set both at Design Step 5/6 once the tech stack is finalized, not discovered via
a failing CI run.

---

## Code Quality
- **cargo-deny** — license/advisory checks. **Pin the version explicitly**:
  `cargo install cargo-deny --version 0.16 --locked` (or whatever the current stable
  minor is at Step 5 — check and record it there); config `deny.toml`. An unpinned
  `cargo install cargo-deny --locked` risks a schema-breaking version landing between
  Design and a later Development session (e.g. `unmaintained`/`unsound` changed meaning
  from a lint-level string to a dependency-scope filter across a past major) —
  invalidating an already-approved `deny.toml` with no warning until CI fails. Known-good
  skeleton matching the pinned version:
  ```toml
  # deny.toml
  [advisories]
  unmaintained = "all"       # dependency-scope filter, not a lint level, as of 0.16
  unsound = "all"
  yanked = "deny"
  ```
- **cargo-audit** — RustSec vulnerability scan. `cargo install cargo-audit --locked`.
- **cargo-watch** — rebuild/retest on change. `cargo install cargo-watch --locked`.

## GitHub Actions
Pin actions to the current Node major at minimum (Node 24 as of this writing, e.g.
`actions/checkout@v5`) — check for a newer default if the warning recurs.

## Database
- **sqlx-cli** — `cargo install sqlx-cli --no-default-features --features native-tls,postgres --locked`.
  Migrations: `sqlx migrate run`. Check: `cargo sqlx prepare`.

## Code Generation
- **protoc** — system package manager install (not cargo), append to `scripts/setup_env.sh`.
- **cargo-expand** — `cargo install cargo-expand --locked`. Full expansion needs `+nightly`.

## Embedded
> See `agents/ESP32_ESPIDF_RUST_BUILD_GUIDE.md` for the full ESP32/ESP-IDF toolchain
> setup (`espup`, `.cargo/config.toml`, target triples) this section assumes.
### espflash
Flashing/serial-monitor for ESP32/ESP-IDF. `cargo install espflash --locked`. Flash:
`espflash flash target/<triple>/release/<binary>`. Monitor: `espflash monitor`.

---

## Missing Tool Protocol

1. Attempt to install per this file (or `scripts/setup_env.sh`).
2. Verify the install succeeded.
3. If it succeeds cleanly: append the install command idempotently to
   `scripts/setup_env.sh` and `scripts/setup_env.bat`, inform the user, and
   **proceed** — this is the sole case that continues without stopping.
4. **If the install fails, or resolves the tool but the resulting build/test still
   fails for any reason (version conflict, incompatibility, unexpected error): stop
   the session per `agents/DEVELOPMENT.md` §4's escalation model** — write the Phase
   Summary and stop. Do not attempt further troubleshooting or continue other tasks.

**Non-fatal tools:** tools documented in Development Plan §4 as non-fatal (e.g.
Playwright browser drivers in headless CI) may fail without triggering the above —
`setup_env.sh` logs the failure and continues via error-accumulation, never `set -e`.

---

## Forbidden Tools

Must never appear in `scripts/setup_env.sh`/`.bat` or any Plan §4 prerequisites table
for a project using the tool they conflict with.

### wasm-bindgen-cli (when using Trunk)
**FORBIDDEN in any Trunk-based project.** Trunk manages `wasm-bindgen` automatically at
the exact version matching `Cargo.lock`; a separately-installed CLI will almost
certainly mismatch, producing a hard build failure with no reliable fix from
`setup_env.sh` (can't predict the correct version without parsing `Cargo.lock`). Only
appropriate when using `wasm-pack` **without** Trunk, and only pinned to the
`Cargo.lock`-matching version — an unusual case.
