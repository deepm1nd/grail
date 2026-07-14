# Patch: agents/ESP32_ESPIDF_RUST_BUILD_GUIDE.md — Version Reference Table

**Reason:** Phase 0 Escalation (fesp32_phase0_summary.md) — the guide did not previously
pin an `espup` version, leaving `espup install` to resolve whatever was "latest" at
install time. The installed result (`1.82.0-nightly`, 2024-10-24) predated Rust 2024
edition stabilization and sat below any reasonable MSRV target. This patch adds an
explicit, pinned `espup` version to the guide's existing Version Reference table so this
can't silently drift again.

## Apply this change

In the **"Version Reference (Confirmed Working, mid-2026)"** table, change the `espup` row
from:

```
| `espup` | latest | `cargo install espup --locked` |
```

to:

```
| `espup` | `0.14.1` (pinned — do not use unpinned `latest`) | `cargo install espup --version 0.14.1 --locked` |
```

And add one sentence immediately below the table:

> **Do not install `espup` unpinned.** An unpinned `espup install` resolves whatever
> release is current at install time, which may install a toolchain predating Rust 2024
> edition stabilization or the project's workspace MSRV — exactly the failure mode that
> produced a full Phase 0 escalation on a real project. Always pin the exact `espup`
> version in `scripts/setup_env.sh`, and re-verify it against this table's `rustc`
> version before bumping.

## Also add a row to the Quick-Start Checklist

After the existing `espup install` checklist item, add:

```
- [ ] `espup` was installed at the version pinned in `scripts/setup_env.sh` — not
      whatever `cargo install espup --locked` resolved to unpinned
```

This is a Trivial, mechanically-dictated correction (no judgment call) — apply directly.
