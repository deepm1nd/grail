# Preferred Development Tools

**MANDATE: Adherence to this guide is not optional. It is a strict requirement.**

Governs **development tools** (binaries in the agent's environment), distinct from
`PREFERRED_DEPENDENCIES.md` (Rust library dependencies). Listed tools are usable without
approval; unlisted tools require approval (`AGENTS.md` §2.2).

See `CHANGELOG.md` for version history.

---

## Standing Mandate: Python Is Forbidden — Everywhere, Not Just Here

**Python MUST NOT be used anywhere in this workflow** — Design, Development, CI, setup
scripts, metrics parsers, one-off data munging, ad-hoc analysis, anywhere an agent might
otherwise reach for "a quick script." This is a project-wide default-deny, not a
domain-specific rule scoped to any one file or stage.

**The sole exception:** explicit, one-off user approval for a specific task, granted only
when no other option (Node.js, bash, or Rust itself) is viable for that particular task.
This is the same approval tier as an Unlisted dependency (`AGENTS.md` §2.2) — proposed with
justification, not a unilateral judgment call — and does not establish a standing exception
for that project; each occurrence is its own approval.

**Script-language preference where a choice exists** (e.g. `scripts/metrics/*` parsers,
`setup_env.sh` helpers): **Node.js is preferred** — handles JSON parsing (the actual shape of
nextest/coverage/deny/audit output) more naturally than bash, and Node is already a
first-class dependency of this stack (Trunk/WASM projects require it; GitHub-hosted runners
ship it by default). **Bash (with `jq` as needed) is acceptable for genuinely simple,
low-branching cases** — straightforward field extraction, no real control flow. A bash
script that starts accumulating conditionals, loops, or non-trivial JSON traversal should
have been Node.js from the start, not grown further in bash.

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
  `cargo install cargo-deny --version 0.19 --locked` (or whatever the current stable
  minor is at Step 5 — check and record it there); config `deny.toml`. An unpinned
  `cargo install cargo-deny --locked` risks a schema-breaking version landing between
  Design and a later Development session — this has already happened once for real: the
  `[advisories]` fields `vulnerability`/`unmaintained`/`unsound`/`yanked`/`notice` were
  **lint-level strings** (`deny`/`warn`/`allow`) through 0.14.x, were deprecated, and were
  then **removed entirely** as of a later 0.1x release — every advisory now denies by
  default with no lint-level override at all. **Always verify the exact accepted schema
  for the pinned version at Design Step 5 rather than copying the skeleton below
  unchecked** — this field set has changed shape more than once and will likely change
  again. Known-good skeleton for the current 0.19.x line:
  ```toml
  # deny.toml
  [licenses]
  # Permissive, notice-only licenses — see PREFERRED_DEPENDENCIES.md's License
  # Compatibility Criterion for the actual test this list satisfies. Without this
  # section, cargo-deny defaults to an empty allow-list and rejects virtually every
  # real dependency tree, including plain MIT/Apache-2.0 — do not omit it.
  allow = [
      "MIT",
      "Apache-2.0",
      "Apache-2.0 WITH LLVM-exception",
      "BSD-2-Clause",
      "BSD-3-Clause",
      "ISC",
      "Unicode-3.0",
      "CC0-1.0",
      "Zlib",
      "NCSA",
      "BSL-1.0",
      "CDLA-Permissive-2.0",
  ]

  [advisories]
  # As of 0.19.x, vulnerability/unmaintained/unsound/yanked/notice all deny by default —
  # there is NO lint-level field for them anymore (the old "deny"/"warn"/"allow" strings
  # from pre-0.15 configs will fail to parse). The only remaining knobs are `unmaintained`
  # and `unsound`'s dependency-SCOPE (not severity), and an explicit per-advisory `ignore`
  # allowlist for documented one-off exceptions.
  unmaintained = "all"        # "all" | "workspace" (direct workspace deps only) | "transitive"
  unsound = "workspace"       # default; "all" | "transitive" also valid
  ignore = [
      # { id = "RUSTSEC-0000-0000", reason = "documented justification, required" },
  ]
  ```
  Project-specific license needs beyond this set (a dependency whose license isn't in
  the list above) are flagged and resolved at Design Step 5, same tier as an Unlisted
  dependency (`PREFERRED_DEPENDENCIES.md`) — never silently added to `allow` without
  that approval.

  **`[advisories].ignore` policy — every entry MUST satisfy all of these:**
  - **`reason` is mandatory, every entry, no exceptions** — not "known issue," but the
    actual justification: why it's safe (e.g., compile-time-only, no runtime attack
    surface, no viable maintained alternative as of a stated date) and, for a transitive
    crate, which direct dependency pulls it in (`cargo tree -i <crate>`).
  - **Same approval tier as an Unlisted dependency** (`AGENTS.md` §2.2) — proposed with
    justification, explicit user approval, never a unilateral judgment call to unblock a
    red CI run.
  - **Never used for a `vulnerability`-class finding** — reserve `ignore` for
    `unmaintained`/`notice`-tier advisories only. A genuine CVE is resolved (upgrade,
    patch, or remove the dependency), never silently ignored, regardless of how deep it
    sits in the tree.
  - **Recorded as a standing risk to revisit**, same convention as the `[patch]` exception
    in `PREFERRED_DEPENDENCIES.md` — logged in the Architecture Specification's Risk
    Management section and `CHANGELOG.md`, with the RUSTSEC ID and date.
  - **Re-checked at each dependency bump**, not set once and forgotten — remove the entry
    the moment a maintained fork or replacement crate appears for that dependency.
  - Note that an ignored advisory still prints a note in cargo-deny's output even when
    suppressed from failing the check — the exception is visible in CI logs, not hidden.
- **Clippy strictness — mandatory floor vs. per-project opt-in.** `-D warnings` (deny
  actual compiler/clippy warnings) is **mandatory on every project, no exceptions** — cheap
  and catches real defects. `-D clippy::unwrap_used` (deny every `.unwrap()` call,
  including in test code, forcing `?`/`match`/`.expect("reason")` instead) is **NOT** a
  fleet-wide mandate — it is a **per-project opt-in**, proposed and confirmed at Design
  Step 5 for projects where a production-path panic-on-unwrap is a real risk (long-running
  services, embedded targets). Left out by default because it commonly fires on legitimate
  test-code `.unwrap()`s too, adding friction disproportionate to the bug class it catches
  in prototype-stage or test-heavy code.
- **cargo-audit** — RustSec vulnerability scan. `cargo install cargo-audit --locked`.
- **cargo-watch** — rebuild/retest on change. `cargo install cargo-watch --locked`.

## Canonical Commands (CI Invocations)
> Maps each tool to the exact command CI runs it with, and the `metrics/*.toml` file
> (`agents/CI.md`) it feeds. Design Step 5 confirms which of these apply per project
> (e.g. Playwright only if the project has an E2E suite); Step 8 bakes the confirmed
> set into the generated `ci.yml`.

| Tool | Canonical CI command | Feeds |
|---|---|---|
| cargo-nextest | `cargo nextest run --workspace --message-format libtest-json` | `metrics/tests.toml` |
| cargo-llvm-cov | `cargo llvm-cov --workspace --json --output-path target/llvm-cov.json` | `metrics/coverage.toml` |
| cargo-deny | `cargo deny --format json check` | `metrics/deny.toml`, and feeds the `THIRD_PARTY_LICENSES.md` regeneration/drift-check (`agents/CI.md` Stage 5) |
| cargo-audit | `cargo audit --json` | `metrics/audit.toml` |
| Playwright | `npx playwright test --reporter=json` | `metrics/playwright.toml` (conditional — only if the project has an E2E suite) |

Each row's JSON output is parsed into its target TOML by a small script in
`scripts/metrics/` (one script per domain) — see `agents/CI.md` for where in the
pipeline this runs.

## GitHub Actions
**Pin every action referenced in `ci.yml` to a version targeting the current Node major**
(Node 24 as of this writing) — not just `actions/checkout`. This has recurred more than
once across different actions as GitHub deprecates Node 20 runners; treat it as a property
every action pin needs checked, not a one-off fixed for a single action:

| Action | Minimum pin (Node 24) |
|---|---|
| `actions/checkout` | `@v5` |
| `actions/cache` | `@v5` (requires Actions Runner ≥2.327.1 — already satisfied on every GitHub-hosted runner; only matters for self-hosted) |

Check for a newer default whenever the warning recurs on an action not yet in this table —
add it here rather than treating each recurrence as a one-off.

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

**CI has no equivalent of a Jules session's Environment Snapshot.** A Development Phase
session may start from an already-provisioned environment (a prior Snapshot); a CI run
never does — every GitHub Actions job starts from zero state. This means `scripts/setup_env.sh`
MUST be its own explicit step in `ci.yml` (`agents/CI.md`'s Stage 0), run before any
step — including a version/environment check like `check_env.sh` — that assumes tools
are already present. A `ci.yml` that runs `check_env.sh` without a preceding
`setup_env.sh` step is not an environment quirk; it is a missing pipeline step, and
Design Step 9's audit checks for it explicitly (`agents/DESIGN.md` §5.9).

**Apt-based installs in `setup_env.sh` must be defensive, in every environment, not
just in CI.** Do not assume the environment's default apt sources already include
everything needed (observed failure: `libusb-1.0-dev`, a `universe`-component package,
unavailable on a GitHub Actions run despite `universe` nominally being enabled by
default — likely a stale/incomplete package index or mirror substitution at that
moment, not a fundamentally missing component). Every apt-based install step should
read:

```bash
sudo apt-get update
sudo add-apt-repository -y universe   # idempotent — no-ops if already enabled
sudo apt-get update                    # re-run so the index reflects universe's packages
sudo apt-get install -y <package>
```

This costs three extra lines and guards against package-index/mirror drift generally —
it is not a workaround specific to any one cloud provider or runner image.

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
