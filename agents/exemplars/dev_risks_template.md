# Development-Phase Risk Log: [Project Name]

> Companion to `CHANGELOG_template.md`, same tier. **Named `docs/[projectname]_dev_risks.md`,
> generated as an initial empty/skeleton file at Design Step 8
> (`agents/DESIGN.md` §5.8), alongside `CHANGELOG.md` — then edited in place, append-only,
> across every subsequent Development Phase session that discovers a new standing risk.**

## Purpose and Scope

This file is the durable, discoverable home for standing risks discovered **during the
Development Phase** — as distinct from risks identified **during the Design Phase**, which
belong in the Architecture Specification's Risk Management section
(`agents/exemplars/architecture_specification_template.md` §8) and are governed by that
document's own change-control process, not this one.

A Design-Phase-identified risk is never retroactively moved here, and a Development-Phase-
discovered risk is never retroactively edited into the finalized Architecture Specification
— the two documents serve different phases and different change-control regimes. If it's
unclear which a given risk is, the deciding question is *when it was discovered*, not how
severe it is: something surfaced only once real dependency resolution, a real `Cargo.lock`,
or real advisory/license scan output existed is Development-Phase by definition, since none
of that exists yet at Design time.

**Logged here, not merely as an inline `deny.toml` comment or a one-off `CHANGELOG.md`
line:**
- `deny.toml` `[advisories].ignore` entries for unmaintained/notice-tier advisories with no
  viable maintained replacement (`agents/PREFERRED_TOOLS.md`'s ignore-list policy).
- `deny.toml` `[[licenses.exceptions]]` entries for a specific third-party crate whose
  license doesn't satisfy the global `[licenses]` allow-list on its own but is accepted as a
  scoped, per-crate exception (`agents/PREFERRED_DEPENDENCIES.md`'s License Compatibility
  Criterion).
- Dependency swaps made in direct response to a security/maintenance advisory (e.g. a crate
  replaced because its predecessor was abandoned or yanked).
- A `[patch]` exception approved under `agents/PREFERRED_DEPENDENCIES.md`'s No Local
  Patching, Forking, or Vendoring mandate (the rare, approval-gated unreleased-upstream-fix
  case) — its justification and tracking issue are recorded here, not in the Architecture
  Specification's Risk Management section, since the exception is discovered and approved
  mid-Development.
- Any other standing, revisit-later exception a Development Phase session is explicitly
  approved to carry forward rather than resolve immediately.

**Why a dedicated file rather than the `deny.toml` `reason` field alone:** an inline TOML
comment is easy to write tersely and hard to search/audit later; this file gives the same
justification text a durable, more discoverable home, and gives every such exception a
consistent, reviewable shape across the whole project's history.

---

## How to Use

- **Append-only, edited in place, never copied/renamed/"v2'd"** — same convention as
  `CHANGELOG.md` and the Development Checklist (`AGENTS.md` §2.7). Correct a factual error
  in a past entry by adding a new entry noting the correction; never rewrite a past entry.
- One `### Risk: [short identifier]` entry per risk, added the same session the risk is
  discovered and its disposition approved — not batched or deferred to a later cleanup pass.
- **Re-evaluation is not optional busywork** — each entry's Re-evaluation Trigger is checked
  at the point it names (a dependency bump, a stated future event) and the entry is either
  closed out (mechanism removed, crate swapped, risk resolved) or explicitly re-affirmed
  with an updated date, never left silently stale.

---

## Entry Template

Copy this block for each new risk:

```markdown
### Risk: [short identifier, e.g. "bincode-transitive-via-gloo-worker"]

- **Date discovered:** YYYY-MM-DD
- **RUSTSEC ID / crate + version:** [e.g. RUSTSEC-2025-0141, or crate name + version if no
  RUSTSEC ID applies (e.g. a `[[licenses.exceptions]]` case)]
- **Mechanism:** [`[advisories].ignore` entry / `[[licenses.exceptions]]` entry /
  dependency swap / `[patch]` exception / other]
- **Justification:** [full reasoning — same content as the corresponding `deny.toml`
  `reason` field, written out in full here rather than left as a terse inline comment. Why
  this is safe or acceptable; for a transitive crate, which direct dependency pulls it in
  (`cargo tree -i <crate>`); for a license exception, why the specific crate's license is
  acceptable even though it doesn't satisfy the global allow-list.]
- **Re-evaluation trigger:** [e.g. "re-evaluate when <dependency> adopts <replacement>" /
  "re-check at each dependency bump" / "remove once <upstream issue/PR> ships a release"]
- **Status:** Open | Closed (YYYY-MM-DD, resolution: [what changed])
```

---

## Log

*(No risks logged yet — entries are appended here as they are discovered and approved
during the Development Phase.)*
