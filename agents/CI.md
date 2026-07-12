# Continuous Integration (CI) Guide

**MANDATE: Adherence to this guide is not optional. It is a strict requirement.**

Governs the project's GitHub Actions CI pipeline (`.github/workflows/ci.yml`) — distinct
from `PREFERRED_TOOLS.md` (which tools exist and their canonical commands),
`PREFERRED_SERVICES.md` (infra services CI may need to start), and `PREFERRED_DEPENDENCIES.md`
(the license-compatibility criterion CI's Stage 5 enforces). Referenced from `AGENTS.md`,
`README.md`'s repository structure list, and `agents/DESIGN.md` Steps 5 and 8.

See `CHANGELOG.md` for version history.

---

## 1. Relationship to Design Steps 5 and 8

- **Design Step 5** (`agents/DESIGN.md` §5.5) *identifies* which conditional stages below
  apply to this project — WASM/Trunk build? Playwright/E2E suite? ESP32/ESP-IDF? infra
  services via `deploy/docker-compose.dev.yml`? — as part of its existing tool/dependency
  feasibility check. No file is produced at this point.
- **Design Step 8** (`agents/DESIGN.md` §5.8) *generates* the concrete `.github/workflows/ci.yml`
  from this file's skeleton, using Step 5's stage-applicability findings as input — added to
  Step 8's Output list and GATE alongside the Development Plan, Checklist, Dev Prompt,
  README, and `.gitignore`.
- **Development Phase 0** reviews/confirms/extends the generated `ci.yml` against the
  repository as it starts to take shape — same review pattern already used for README and
  `.gitignore`, not a from-scratch authoring task.

---

## 2. Relationship to Task-Level Verification

CI is **complementary to, not a substitute for or duplicate of**, the task-level
Verification Method checks a Development Phase session runs locally before checking a DoD
box (`agents/exemplars/development_checklist_template.md`). The two operate at different
scope and by different mechanism:

- **Task-level Verification** is narrow, local, and agent-run — scoped to the current
  task's own slice of the system, self-certified by the agent working that task.
- **CI** is broad, hermetic, and repo-triggered — a full-workspace build, full test suite,
  coverage, license/security scan — run independently of what any single session claims
  locally. It is the same adversarial-independence posture the Design Phase already uses
  for Step 7/9 audits, applied to code instead of documents.

**CI is an async, human-reviewed backstop, not a session-blocking gate.** Per
`AGENT_TOOL_POLICY.md` §2, a Development Phase session stops after every `submit` and
awaits the user's next message — it does not loop waiting on a CI run, and there is no
Development Plan-level rule requiring the next task's session to wait on a prior push's CI
result. A red CI check is surfaced the same way other between-session evidence is reviewed
(`README.md`'s "Review evidence as it accumulates" workflow step) — a human notices it and
brings it back to a Design Phase session if it needs diagnosis, the same path a stuck
session's Phase Summary already takes.

**Open Question — CI failure triage criteria are deliberately not codified here.**
What's worth escalating versus routine noise is left to human judgment for now, pending
enough real-project history to generalize a rule. Two things worth stating plainly in the
meantime, without turning them into a hard rule:
- **Early-phase CI red is common and not itself alarming.** A newly-scaffolded project
  commonly has failing stages well into its first several phases; this alone is not a
  signal.
- **Persistence without narrowing, or an identical failure signature across otherwise
  unrelated stages** (e.g. the same exit code appearing on several nominally-independent
  jobs) **is worth a human look** — a tentative heuristic based on observed cases, not a
  hard rule, and not something a session should act on unilaterally.

---

## 3. Stage Skeleton

Fixed stage order. Stages marked *(conditional)* are included only when Step 5 determined
they apply to this project; every other stage runs unconditionally, on every project.

### Stage 0: Environment Setup — mandatory, first, no exceptions

**CI has no equivalent of a Jules session's Environment Snapshot.** Every GitHub Actions
run starts from zero state — there is no persistent, pre-provisioned environment carrying
over from a prior run the way a Jules Development Phase session's Snapshot does. Stage 0
MUST:

1. Install the Rust toolchain (per the pinned `rust-toolchain.toml`).
2. Run `scripts/setup_env.sh`, unconditionally — **before** any step, including
   `check_env.sh`, that assumes tools are already present. `setup_env.sh` is the
   *installer*; `check_env.sh` is a read-only *verifier* — it was never going to install
   anything itself. A workflow that runs `check_env.sh` without a preceding `setup_env.sh`
   step is a missing pipeline step (`PREFERRED_TOOLS.md`'s Missing Tool Protocol), not an
   environment quirk — see §4.1's Design Step 9 audit check below.
3. Cache `~/.cargo/bin` alongside the registry/`target` cache (keyed on `Cargo.lock` and
   toolchain version) — CI's rough equivalent of a Jules Environment Snapshot, reducing
   repeat-install cost on unchanged dependencies without skipping the install step itself
   on a cache miss.
4. Run `scripts/check_env.sh` last, as a fail-fast confirmation that Stage 0 actually
   worked — not as the first or only check.

### Stage 1: Build
Full hermetic workspace build.

### Stage 2: Test
`cargo nextest run --workspace --message-format libtest-json` (`PREFERRED_TOOLS.md`'s
Canonical Commands table) → `metrics/tests.toml`.

### Stage 3: Coverage
`cargo llvm-cov --workspace --json --output-path target/llvm-cov.json` → `metrics/coverage.toml`.

### Stage 4: WASM Build *(conditional — Trunk/WASM component present)*
`trunk build --release`, per `PREFERRED_TOOLS.md`'s Trunk conventions (output-directory
separation from `assets/`, no separately-installed `wasm-bindgen-cli`).

### Stage 5: Security & License Scan
Two independent tools, one stage — see §4 below for how to tell their failures apart.

- **cargo-deny** — `cargo deny check --format json` → `metrics/deny.toml`. Covers both
  `[licenses]` (the permissive/notice-only allow-list, `PREFERRED_DEPENDENCIES.md`'s
  License Compatibility Criterion) and `[advisories]` (`unmaintained`/`unsound`/`yanked`).
- **cargo-audit** — `cargo audit --json` → `metrics/audit.toml`. RustSec vulnerability scan,
  independent of cargo-deny.
- **`THIRD_PARTY_LICENSES.md` drift check** — regenerate the file from the current
  resolved dependency tree (fed by the same cargo-deny license enumeration) into a temp
  location, diff against the committed copy, **fail the stage on any difference — do not
  auto-commit.** A human reviews and commits the regenerated file deliberately; CI's job
  is to catch drift, not to silently resolve it, consistent with how this framework treats
  every other auto-fix decision (Trivial-vs-Substantive audit findings, `metrics/` commits
  at Stage 8 below being the one deliberate exception, made explicit because unlike a
  license disclosure, metrics have no compliance weight).

### Stage 6: E2E *(conditional — Playwright/E2E suite present)*
`npx playwright test --reporter=json` → `metrics/playwright.toml`.

### Stage 7: Infra Services *(conditional — `deploy/docker-compose.dev.yml` present)*
`docker compose -f deploy/docker-compose.dev.yml up -d`, started before Stage 2 (Test) if
any test depends on it — see `PREFERRED_SERVICES.md`'s Session Startup section.

### Stage 8: Metrics Commit *(main-branch merges only)*
Bot commit of regenerated `metrics/*.toml` back to the branch. Unlike the
`THIRD_PARTY_LICENSES.md` drift check above, metrics are auto-committed rather than
fail-on-drift — they carry no compliance/legal weight, only informational/badge value, so
routine auto-refresh is acceptable where a license disclosure's drift is not.

---

## 4. Reading a Stage 5 Failure

A red Stage 5 does not by itself tell you which of three distinct checks fired — read the
log before assuming which one:

| Signature | Source | Meaning |
|---|---|---|
| `error[rejected]: failed to satisfy license requirements`, with a named crate and license | cargo-deny, `[licenses]` | A dependency (often transitive) carries a license outside the allow-list. |
| `unmaintained`/`yanked` findings, different message shape than the above | cargo-deny, `[advisories]` | An advisory-tier finding — not a license issue. |
| CVE ID + advisory link | cargo-audit | A RustSec vulnerability — independent of cargo-deny entirely. |

Early in a project's life, a red Stage 5 (or any stage) may also simply mean a missing
Stage 0 step or an environment/tooling gap rather than a real finding at all — see §2's
early-phase-churn note above.

---

## 4.1. Design Step 9 Audit Check

`agents/DESIGN.md` §5.9's Tier-A-equivalent mechanical checks include a check specific to
this file's Stage 0 requirement above — see that section for the exact rule text.

---

## Appendix

See `CHANGELOG.md` for this file's full version history.
