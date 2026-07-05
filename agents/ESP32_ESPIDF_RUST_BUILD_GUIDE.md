> **Scope note:** this file applies **only** to projects with an ESP32/ESP-IDF embedded
> component. Referenced conditionally from `CLAUDE.md` §2.1, `agents/PREFERRED_TOOLS.md`'s
> Embedded section, and `agents/PREFERRED_DEPENDENCIES.md`'s Embedded section — not part of
> the standard reading set for non-embedded projects.

# ESP32 + ESP-IDF + Rust + tokio: Hard-Won Build Guide

> **Audience:** Any Rust project targeting ESP32 or related Espressif chips (ESP32-S2, S3,
> C2, C3, C5, C6) using the ESP-IDF framework (`std`/`tokio`/`esp-idf-svc` stack).
>
> **Origin:** Accumulated across multiple build failures on a real project targeting
> `xtensa-esp32-espidf` and RISC-V `espidf` targets with `tokio` as the async runtime.
> Every failure mode below was actually hit, in approximately this order. Saving you the
> same journey.
>
> **Scope:** Does NOT cover the bare-metal `esp-hal`/`embassy` stack. If your project is
> `no_std` and doesn't use ESP-IDF, this document is not for you — see the note on the
> `none-elf` vs `espidf` distinction below, which is the single most important thing to
> get right first.

---

## Quick-Start Checklist

Before attempting any `cargo build --target *-espidf`, confirm all of these:

- [ ] `espup` installed and `espup install` completed
- [ ] `source ~/export-esp.sh` run in this shell session (`echo $IDF_PATH` is non-empty)
- [ ] `rustup component add rust-src` installed
- [ ] `.cargo/config.toml` exists at repo root (exact content below)
- [ ] `[package.metadata.esp-idf-sys]` block present in `Cargo.toml` (exact content below)
- [ ] `tokio` features constrained — **not** `features = ["full"]`

If any of these is missing, a build attempt will fail with a misleading error that looks
like a completely different problem each time. See the Failure Modes section for the
exact symptom each omission produces.

---

## The Correct `.cargo/config.toml`

Create this file at the repository root. It does not exist by default — you must create it.

```toml
# .cargo/config.toml

# Applied to ALL espidf targets (both Xtensa and RISC-V) via cfg(target_os = "espidf").
# You do not need to repeat these per target triple.
#
# espidf_time64
#   Required for ESP-IDF v5+. esp-idf-sys performs a compile-time assertion for this
#   flag and refuses to compile without it on v5+ toolchains. Harmless on v4, so just
#   always include it.
#
# mio_unsupported_force_poll_poll
#   THE most important flag in this file. Without it, mio uses its default
#   epoll/kqueue event backend on any POSIX-like target — including espidf. That
#   backend unconditionally depends on signal-hook-registry, which assumes POSIX
#   signal primitives (siginfo_t, SA_RESTART, SIGKILL) that ESP-IDF's Newlib libc
#   doesn't implement. The build fails deep in signal-hook-registry with a confusing
#   C-header error. With this flag, mio switches to a poll(2)-based backend that has
#   no such dependency.
#
#   IMPORTANT: Constraining tokio's feature set (removing the "signal" feature, etc.)
#   does NOT fix this. signal-hook-registry is a hard dependency of mio on POSIX-like
#   targets regardless of which tokio features you enable. This cfg flag is the only
#   correct fix.

[target.'cfg(target_os = "espidf")']
linker    = "ldproxy"
runner    = "espflash flash --monitor"
rustflags = [
  "--cfg", "espidf_time64",
  "--cfg", "mio_unsupported_force_poll_poll",
]

# Default target for plain `cargo build` without an explicit --target flag.
# Change to whichever chip is your primary development target.
[build]
target = "xtensa-esp32-espidf"

# REQUIRED: espidf targets do not ship pre-compiled std/core/panic_abort artifacts
# with the Rust toolchain. Without this, cargo cannot find `core` for the target,
# producing the error: "error[E0463]: can't find crate for `core`"
# Also requires: rustup component add rust-src
[unstable]
build-std = ["std", "panic_abort"]
```

---

## The Required `Cargo.toml` Additions

Add this block to your `Cargo.toml` (at the end, after `[dependencies]`):

```toml
# Instructs esp-idf-sys's build script to download and build exactly this version
# of the ESP-IDF C framework automatically during `cargo build`. No manual ESP-IDF
# installation is required — the build script handles it.
#
# NEVER omit this block or leave esp_idf_version unset. Without it, the build script
# either uses whatever happens to be cached globally (non-reproducible) or downloads
# an unpinned/unstable version.
#
# esp_idf_tools_install_dir = "global" caches the download globally across all
# projects. Without it, every `cargo clean` triggers a fresh multi-minute download.
#
# v5.2.2 is the recommended stable release for tokio compatibility as of mid-2026.
# Check the esp-rs/esp-idf-hal release notes before bumping this.

[package.metadata.esp-idf-sys]
esp_idf_version           = "v5.2.2"
esp_idf_tools_install_dir = "global"
```

Also set your `tokio` dependency with an explicit, constrained feature set:

```toml
[dependencies]
tokio = { version = "1", features = ["rt", "net", "sync", "time", "macros"] }
```

**Do not use `features = ["full"]`.** The `full` feature set pulls in `signal-hook-registry`
via a different path. While `mio_unsupported_force_poll_poll` is the primary fix,
keeping the feature set minimal is correct hygiene and reduces binary size on an
embedded target.

---

## Required Session Setup

Run this at the start of every development session, before any `cargo build --target *-espidf`:

```bash
source ~/export-esp.sh
```

This file is created by `espup install`. It sets `IDF_PATH`, `IDF_TOOLS_PATH`, and other
environment variables that `esp-idf-sys`'s build script needs to locate the Xtensa and
RISC-V compilers. Without it, the build script cannot find the compilers even if `espup`
has installed them correctly — it silently fails with a linker-not-found error.

Verify it worked:
```bash
echo $IDF_PATH    # must be non-empty
```

Add this to your `scripts/check_env.sh` or equivalent session-startup script so it cannot
be forgotten.

---

## Required Components

```bash
# Install the esp toolchain (Xtensa + RISC-V):
cargo install espup --locked
espup install

# Install rust-src — required by build-std:
rustup component add rust-src

# Verify espidf targets are installed (NOT none-elf — see below):
rustup target list --installed | grep espidf
```

---

## The Correct Build Commands

```bash
# Classic ESP32 (Xtensa):
cargo +esp build --target xtensa-esp32-espidf

# ESP32-S3 (Xtensa):
cargo +esp build --target xtensa-esp32s3-espidf

# ESP32-C2, C3 (RISC-V, lower core count):
cargo +esp build --target riscv32imc-esp-espidf

# ESP32-C5, C6 (RISC-V, with atomics):
cargo +esp build --target riscv32imac-esp-espidf
```

The `+esp` toolchain selector is required for Xtensa targets. RISC-V targets technically
compile with the standard Rust toolchain, but using `+esp` consistently avoids surprises.

---

## The Five Failure Modes — What Each Error Actually Means

These are real failures hit in sequence on a real project. Each looks like a distinct
problem; all are caused by missing build infrastructure.

### Failure 1 — Wrong RISC-V target triples (`none-elf`)

**Symptom:** Build immediately fails with errors about missing `std`, `alloc`, or `tokio`
feature incompatibility.

**Root cause:** There are two completely different RISC-V target families that share the
same base ISA name:

| Target | OS layer | std | tokio | Use for |
|---|---|---|---|---|
| `riscv32imac-unknown-none-elf` | None (bare metal) | ✗ | ✗ | `no_std` embedded firmware |
| `riscv32imac-esp-espidf` | ESP-IDF (FreeRTOS + lwIP) | ✓ | ✓ | `std`-based ESP-IDF apps |

If your project uses `std`, `tokio`, or `esp-idf-svc`, you need the `espidf` targets.
The `none-elf` targets require a complete `no_std` rewrite and a different async runtime
(e.g. `embassy`). They are not interchangeable.

**Fix:** Always use `*-esp-espidf` target triples. Never use `*-unknown-none-elf` for
a `std`/`tokio` codebase.

---

### Failure 2 — `signal-hook-registry` POSIX build error

**Symptom:** Build fails deep in `signal-hook-registry` with errors like:
```
error[E0425]: cannot find value `SA_RESTART` in this scope
error[E0412]: cannot find type `siginfo_t` in this scope
```

**Root cause:** `mio` (a `tokio` dependency) uses OS-level event notification. On any
target that looks POSIX-like — including `espidf`, which has a POSIX compatibility layer
— `mio` defaults to its epoll/kqueue backend, which unconditionally links against
`signal-hook-registry`. That crate uses POSIX signal primitives (`siginfo_t`, `SA_RESTART`,
`SIGKILL`) that ESP-IDF's Newlib libc doesn't fully implement.

**What does NOT fix this:**
- Removing the `signal` feature from tokio — `signal-hook-registry` is a hard `mio`
  dependency, not a tokio feature dependency
- Using `features = ["rt", "net", "sync", "time", "macros"]` instead of `full` — same
  reason; this is `mio`'s decision, not tokio's

**Fix:** Add `"--cfg", "mio_unsupported_force_poll_poll"` to rustflags in `.cargo/config.toml`
(under `[target.'cfg(target_os = "espidf")']`). This switches `mio` to a poll(2)-based
backend that has no dependency on POSIX signal primitives.

---

### Failure 3 — "can't find crate for `core`"

**Symptom:**
```
error[E0463]: can't find crate for `core`
```

**Root cause:** `espidf` targets do not ship pre-compiled standard library artifacts
(`core`, `std`, `panic_abort`) with the Rust toolchain installation. Unlike tier-1 targets
(x86_64-unknown-linux-gnu etc.), these are not bundled. Cargo needs to compile them from
source for these targets.

**Fix:** Two things, both required:

1. Add to `.cargo/config.toml`:
   ```toml
   [unstable]
   build-std = ["std", "panic_abort"]
   ```
2. Install the `rust-src` component:
   ```bash
   rustup component add rust-src
   ```

Without `rust-src`, Cargo has no source to compile even if `build-std` is set.

---

### Failure 4 — Compiler not found / linker not found

**Symptom:** Build fails immediately with something like:
```
error: could not exec the linker `ldproxy`
error: linker `xtensa-esp32-elf-gcc` not found
```

**Root cause:** The Xtensa and RISC-V compilers installed by `espup` are only available
in the shell environment after sourcing `~/export-esp.sh`. This file sets `IDF_PATH`,
`IDF_TOOLS_PATH`, `PATH`, and other variables that `esp-idf-sys`'s build script uses to
locate the toolchain. In a fresh shell — or any CI environment that doesn't source this
file — the compilers are invisible to Cargo.

**Fix:** Run at the start of every session:
```bash
source ~/export-esp.sh
echo $IDF_PATH   # verify: must be non-empty
```

In CI, add this as a step before any `cargo build` invocation.

---

### Failure 5 — Non-reproducible build / wrong ESP-IDF version

**Symptom:** Build succeeds on one machine but fails on another; or worked yesterday but
fails after a `cargo clean`; or different API is available than expected.

**Root cause:** Without `[package.metadata.esp-idf-sys]` in `Cargo.toml`, `esp-idf-sys`'s
build script either uses whatever ESP-IDF version is cached globally from a previous
download, or fetches the latest unstable version. This makes builds non-reproducible and
can silently introduce API changes.

**Secondary symptom:** If `esp_idf_tools_install_dir` is not set to `"global"`, every
`cargo clean` triggers a fresh ESP-IDF download (several minutes, several GB).

**Fix:** Pin the version explicitly:
```toml
[package.metadata.esp-idf-sys]
esp_idf_version           = "v5.2.2"
esp_idf_tools_install_dir = "global"
```

---

## ESP-IDF vs Embassy — Choosing the Right Stack

If you are reading this wondering whether to use `esp-idf-svc`/`tokio` or
`esp-hal`/`embassy`, this is the key question: **what ecosystem will downstream consumers
of your crate be in?**

- **ESP-IDF consumers** (using `esp-idf-svc` for WiFi, cameras, MQTT, HTTP, etc.) need
  your crate to be on the ESP-IDF stack. `esp-hal` and `esp-idf-svc` **cannot coexist in
  the same binary** — they each own the chip at the hardware level (FreeRTOS vs bare
  metal, lwIP vs smoltcp). Choosing `embassy`/`esp-hal` locks out all ESP-IDF consumers.

- **Bare-metal consumers** (fully `no_std`, using `esp-hal` for everything) need your
  crate on the `embassy`/`esp-hal` stack.

This is not a style preference — it is a link-time incompatibility. Decide once, at design
time, based on your consumer ecosystem. This guide covers the ESP-IDF path.

---

## Tokio on ESP-IDF — What Actually Works

`tokio` does work on ESP-IDF with the correct configuration above. The task/concurrency
model (`tokio::spawn`, channels, `async`/`await`) is fully functional. What you lose
compared to a native Linux build:

- `tokio::signal` — POSIX signal handling; not available on ESP-IDF. Don't use it.
- `tokio::fs` — filesystem; ESP-IDF has a VFS layer but it's limited. Use with caution.
- `tokio::process` — no child processes on ESP-IDF. Not available.
- Raw `UdpSocket`/`TcpListener` — available via `tokio::net`, works correctly.

The concurrency model (tasks, channels, timers, async I/O) is the same on both
`host` (dev/CI) and `embedded` (device) profiles, which is the main reason to prefer
`tokio`+ESP-IDF over `embassy` when downstream consumers are in the ESP-IDF world.

---

## Version Reference (Confirmed Working, mid-2026)

| Component | Version | Notes |
|---|---|---|
| ESP-IDF | `v5.2.2` | Pin via `[package.metadata.esp-idf-sys]` |
| `esp-idf-svc` | match `espup` output | Record in `docs/toolchain_versions.md` |
| `esp-idf-hal` | match `espup` output | Same |
| `tokio` | `1.x` | Features: `["rt", "net", "sync", "time", "macros"]` |
| `rust-src` | (current stable) | `rustup component add rust-src` |
| `espup` | latest | `cargo install espup --locked` |
| `espflash` | latest | For flashing; `cargo install espflash` |

---

## Minimum `scripts/check_env.sh` for ESP-IDF Projects

```bash
#!/usr/bin/env bash
set -e

echo "=== ESP-IDF Environment Check ==="

# 1. Source the ESP environment if not already done
if [ -z "$IDF_PATH" ]; then
  echo "Sourcing ~/export-esp.sh..."
  source ~/export-esp.sh
fi
echo "✓ IDF_PATH = $IDF_PATH"

# 2. Verify espup toolchain
echo "✓ Rust toolchain:"
rustc --version
cargo +esp --version 2>/dev/null || { echo "✗ esp toolchain not found — run: espup install"; exit 1; }

# 3. Verify rust-src
rustup component list --installed | grep -q rust-src \
  || { echo "✗ rust-src missing — run: rustup component add rust-src"; exit 1; }
echo "✓ rust-src installed"

# 4. Verify espidf targets (NOT none-elf)
rustup target list --installed | grep -q "espidf" \
  || { echo "✗ No espidf targets found — run: espup install"; exit 1; }
echo "✓ espidf targets installed:"
rustup target list --installed | grep espidf

# 5. Verify .cargo/config.toml exists
[ -f ".cargo/config.toml" ] \
  || { echo "✗ .cargo/config.toml missing — see build guide"; exit 1; }
echo "✓ .cargo/config.toml present"

# 6. Verify esp-idf-sys metadata in Cargo.toml
grep -q "esp_idf_version" Cargo.toml \
  || { echo "✗ [package.metadata.esp-idf-sys] missing from Cargo.toml — see build guide"; exit 1; }
echo "✓ ESP-IDF version pinned in Cargo.toml"

echo ""
echo "=== Environment check passed ==="
```

---

## What a Correct First Embedded Build Looks Like

```bash
# Fresh session startup:
source ~/export-esp.sh

# Smoke test (single target first — fail fast):
cargo +esp build --target xtensa-esp32-espidf

# If that passes, build all confirmed targets:
cargo +esp build --target xtensa-esp32-espidf
cargo +esp build --target xtensa-esp32s3-espidf
cargo +esp build --target riscv32imc-esp-espidf
cargo +esp build --target riscv32imac-esp-espidf
```

The first build will take significantly longer than subsequent ones — ESP-IDF itself is
being downloaded (if not cached) and the standard library is being compiled from source.
On a fast machine with a warm cache, subsequent builds are comparable to normal Rust
compile times. Expect the first build to take 10–30 minutes depending on download speed.

---

## Common Mistakes Summary

| Mistake | Actual error | Fix |
|---|---|---|
| Using `*-unknown-none-elf` RISC-V targets | `std` not available, tokio fails | Use `*-esp-espidf` targets |
| Missing `mio_unsupported_force_poll_poll` | `siginfo_t` / `SA_RESTART` not found in signal-hook-registry | Add cfg flag to `.cargo/config.toml` |
| Missing `build-std` | `can't find crate for core` | Add `[unstable] build-std` to `.cargo/config.toml` |
| Missing `rust-src` component | `can't find crate for core` (same symptom as above) | `rustup component add rust-src` |
| Not sourcing `export-esp.sh` | linker or compiler not found | `source ~/export-esp.sh` at session start |
| Missing `[package.metadata.esp-idf-sys]` | Non-reproducible build, wrong API | Add block to `Cargo.toml` with pinned version |
| `tokio = { features = ["full"] }` | `signal-hook-registry` failure (secondary path) | Constrain features; primary fix is still the cfg flag |
| Assuming `espidf_time64` is optional | `esp-idf-sys` compile-time assertion failure on v5+ | Always include it in rustflags |
