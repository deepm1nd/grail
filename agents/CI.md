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

## 3. Stage Skeleton (v0.9.1)

**Structural change from prior versions: Stages 0–5 now run as steps within a single
job, not as separate jobs.** GitHub Actions runs every job on its own fresh, isolated
runner even when `needs:` chains them — `needs:` orders execution, it does not share a
prior job's installed tools or cache-restored state. Since Stages 0–5 are already a strict
linear dependency chain (each needs the last; there is no real parallelism being given up),
collapsing them into one job's sequential steps means the expensive Stage 0 environment
setup runs exactly once per pipeline run, not once per stage. Per-**step** status (pass/fail,
timing, expandable log) is still visible individually in the Actions UI even inside one job
— what's lost is separate per-job badges/required-checks, which this framework's async,
human-reviewed-backstop posture (§2) doesn't depend on. **Metrics Commit remains a separate
job** (§3, Stage 6) since it runs under a different trigger condition (`main`-only) and
needs its own checkout/artifact-download context.

**Runner: `ubuntu-latest` for every job**, not a pinned Ubuntu version — consistent across
the whole workflow.

**Checkout step, exact form, every job that needs one:**
```yaml
      - uses: actions/checkout@v5
```
No inline comment, no variation — this is the literal form every job uses.

**Trigger: `on: push` only — never `pull_request`.** Consistent with §2's framing of CI as
an async, human-reviewed backstop rather than a merge gate: this workflow does not run
against pull requests, only against actual pushes (to any branch, or restricted to `main` if
preferred per-project). Running on `pull_request` implies CI-as-gate semantics (a required
check blocking merge) that this framework deliberately does not adopt — see §2's Open
Question on failure triage for why that gate model is intentionally not codified here.

```yaml
on:
  push:
    branches: ["**"]   # or restrict to specific branches per project
```

Fixed stage order. Stages marked *(conditional)* are included only when Step 5 determined
they apply to this project; every other stage runs unconditionally, on every project.

### Stage 0: Environment Setup — mandatory, first step of the main job, no exceptions

**CI has no equivalent of a Jules session's Environment Snapshot.** Every GitHub Actions
run starts from zero state — there is no persistent, pre-provisioned environment carrying
over from a prior run the way a Jules Development Phase session's Snapshot does. Because
Stages 0–5 are steps within one job (see above), Stage 0 runs exactly once per pipeline run,
at the top of that job, before any other stage's steps. Stage 0 MUST:

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

**Literal form — copy this exactly, do not paraphrase or reorder.** This exact step has had
to be independently rediscovered by multiple agent sessions; it is not optional boilerplate
and its position (before `check_env.sh`, before any tool-assuming step) is the entire point
of this stage. As of v0.9.1 this is the opening of the single main job (`ci_pipeline`), not
a standalone job:

```yaml
jobs:
  ci_pipeline:
    name: CI Pipeline
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5

      - name: Cache cargo bin/registry
        uses: actions/cache@v5
        with:
          path: |
            ~/.cargo/bin
            ~/.cargo/registry
            target
          key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}

      - name: "Stage 0: Environment Setup"
        run: bash scripts/setup_env.sh

      - name: "Stage 0: Pre-flight version check"
        run: bash scripts/check_env.sh
```

**Note on the prior cross-job tool-persistence warning:** this was a real, observed failure
mode under the old separate-jobs-per-stage structure (a tool installed in one job did not
exist in a different job on a separate runner, even with `needs:`) — e.g. `cargo deny`/
`cargo audit` reported as unknown subcommands in a Security & License Scan job when
installed only in an earlier, separate job. **Collapsing Stages 0–5 into one job (this
version) eliminates this failure mode by construction** — there is only one runner for all
of Stages 0–5, so a tool installed by Stage 0 is simply still present for every later step
in the same job. This is the main structural reason for the merge, not merely a speed
optimization. Metrics Commit (Stage 6, below) is still a genuinely separate job, since it
runs under a different trigger condition — if it needs any of the same tools, it must
install them itself.

### Stage 1: Lint & Format
```yaml
      - name: Check formatting
        run: cargo fmt --all -- --check

      - name: Run Clippy
        run: cargo clippy --workspace --all-targets -- -D warnings
        # -D warnings is mandatory on every project. -D clippy::unwrap_used is a
        # per-project opt-in (PREFERRED_TOOLS.md), added here only if that project
        # elected it at Design Step 5 — do not add it by default.
```

### Stage 2a: Native Build
Full hermetic workspace build: `cargo build --release`.

### Stage 2b: WASM Build *(conditional — Trunk/WASM component present)*
`trunk build --release`, per `PREFERRED_TOOLS.md`'s Trunk conventions (output-directory
separation from `assets/`, no separately-installed `wasm-bindgen-cli`).

### Stage 3: Test
Sub-staged by what each test group needs, not by a unit/integration distinction — Rust's
own tooling doesn't force that split at the runtime level, and nextest discovers both
colocated `#[test]`s and `tests/*.rs` integration tests in the same invocation regardless.

- **Stage 3a — Tests not requiring infra services:**
  `cargo nextest run --workspace --message-format libtest-json` (`PREFERRED_TOOLS.md`'s
  Canonical Commands table) → `metrics/tests.toml`. Requires
  `NEXTEST_EXPERIMENTAL_LIBTEST_JSON: "1"` in the step's `env:` wherever this exact
  message format is used.
- **Doc-tests, same step or immediately adjacent:** `cargo test --doc --workspace` — run
  generically for whichever crates in the workspace have doc-tests; never hardcode a
  specific crate name into this step.
- **Stage 3b — Infra-dependent tests** *(conditional — `deploy/docker-compose.dev.yml`
  present)*: start infra first (`docker compose -f deploy/docker-compose.dev.yml up -d`,
  `PREFERRED_SERVICES.md`'s Session Startup section), then run the test subset that needs
  it.
- **Stage 3c(+) — E2E/Playwright** *(conditional — Playwright/E2E suite present)*:
  `npx playwright test --reporter=json` → `metrics/playwright.toml`. Additional lettered
  sub-stages (3d, 3e...) as a project needs further test-category subdivision — same
  conditional pattern, not a fixed ceiling.

### Stage 4: Coverage
`cargo llvm-cov --workspace --json --output-path target/llvm-cov.json` → `metrics/coverage.toml`.

### Stage 5: Security & License Scan
Two independent tools, one stage — see §4 below for how to tell their failures apart.

**Use direct `cargo` commands, not the dedicated GitHub Actions
(`EmbarkStudios/cargo-deny-action@v2`, `actions-rust-lang/audit@v1`).** These dedicated
Actions have caused recurring issues in practice and are **not** used, superseding prior
guidance. Since Stage 0 already installed the pinned `cargo-deny`/`cargo-audit` binaries
once, earlier in this same job, Stage 5 simply reuses them directly — no separate
installation action needed, and the cross-job problem that originally motivated the
dedicated Actions no longer applies now that everything is one job:

```yaml
      - name: cargo-deny
        run: cargo deny check --format json > /tmp/deny.json

      - name: Parse deny results
        run: node scripts/metrics/parse_deny.js /tmp/deny.json   # or bash+jq if trivial

      - name: cargo-audit
        run: cargo audit --json > /tmp/audit.json

      - name: Parse audit results
        run: node scripts/metrics/parse_audit.js /tmp/audit.json

      - name: THIRD_PARTY_LICENSES.md drift check
        run: bash scripts/metrics/check_license_drift.sh
```

- **cargo-deny** covers both `[licenses]` (the permissive/notice-only allow-list,
  `PREFERRED_DEPENDENCIES.md`'s License Compatibility Criterion — see that file and
  `PREFERRED_TOOLS.md`'s `deny.toml` skeleton for the current 0.19.x-compatible schema) and
  `[advisories]` (`unmaintained`/`unsound`/`ignore`-based exceptions; there is no
  lint-level severity field on this schema version).
- **cargo-audit** — RustSec vulnerability scan, independent of cargo-deny.
- **`THIRD_PARTY_LICENSES.md` drift check** — regenerate the file from the current
  resolved dependency tree (fed by the same cargo-deny license enumeration) into a temp
  location, diff against the committed copy, **fail the stage on any difference — do not
  auto-commit.** A human reviews and commits the regenerated file deliberately; CI's job
  is to catch drift, not to silently resolve it, consistent with how this framework treats
  every other auto-fix decision (Trivial-vs-Substantive audit findings, `metrics/` commits
  at Stage 6 below being the one deliberate exception, made explicit because unlike a
  license disclosure, metrics have no compliance weight).

### Final step of the main job: Branch/Status Marker (`metrics/source.toml`)
Written as the **last step of `ci_pipeline`**, with `if: always()` so it fires whether the
preceding stages passed or failed — this is deliberate: the marker's entire purpose is to
let a human refresh the `main`-branch README and immediately see the state of the most
recent push on whatever branch is currently being worked, without opening Actions.

```yaml
      - name: Write branch/status marker
        if: always()
        run: |
          node scripts/metrics/write_source_marker.js \
            --branch "${{ github.ref_name }}" \
            --commit "${{ github.sha }}" \
            --status "${{ job.status }}"
```

`metrics/source.toml` output shape:
```toml
[source]
branch = "phase-3-auth"
commit = "a1b2c3d"
status = "pass"        # or "fail" — reflects this run's actual pipeline outcome
updated = "2026-07-14T18:03:00Z"
```

The `main`-branch README's badge reads this file (same shields.io dynamic-TOML pattern as
the other metrics badges) so it always reflects the latest push's branch, commit, and
pass/fail state — regardless of which branch that push was on — updated on **every** push,
not gated to a clean run. This is intentional: during active development the README is
functioning as a personal status dashboard, and a stale "last known green" badge would hide
exactly the information (a currently-red push) that badge exists to surface.

### Stage 6: Metrics Commit *(main-branch merges only, separate job)*
Bot commit of regenerated `metrics/*.toml` (including `source.toml`) back to the branch.
Unlike the `THIRD_PARTY_LICENSES.md` drift check above, metrics are auto-committed rather
than fail-on-drift — they carry no compliance/legal weight, only informational/badge value,
so routine auto-refresh is acceptable where a license disclosure's drift is not. This job
remains genuinely separate from `ci_pipeline` (not folded into the merge above) because it
runs under a different trigger condition (`if: github.ref == 'refs/heads/main'`) and needs
its own artifact-download/commit context.

---

## 4. Reading a Stage 5 Failure

A red Stage 5 does not by itself tell you which of three distinct checks fired — read the
log before assuming which one. As of v0.9.1, Stage 5 runs `cargo deny`/`cargo audit`
directly (not via the dedicated GitHub Actions), so these signatures appear in a plain
`run:` step's output rather than inside a third-party Action's own log formatting:

| Signature | Source | Meaning |
|---|---|---|
| `error[rejected]: failed to satisfy license requirements`, with a named crate and license | cargo-deny, `[licenses]` | A dependency (often transitive) carries a license outside the allow-list. |
| `unmaintained`/`yanked` findings, different message shape than the above | cargo-deny, `[advisories]` | An advisory-tier finding — not a license issue. |
| CVE ID + advisory link | cargo-audit | A RustSec vulnerability — independent of cargo-deny entirely. |

Early in a project's life, a red Stage 5 (or any stage) may also simply mean a missing
Stage 0 step or an environment/tooling gap rather than a real finding at all — see §2's
early-phase-churn note above.

---

## 4.1. Design Step 9 Audit Checks

`agents/DESIGN.md` §5.9's Tier-A-equivalent mechanical checks include two checks specific
to this file — see that section for the exact rule text:
1. `check_env.sh` referenced without a preceding `setup_env.sh` step (Stage 0, above).
2. Any action pinned below the current Node-major requirement (e.g. `actions/checkout@v4`
   when `v5`+ is current) — `PREFERRED_TOOLS.md`'s GitHub Actions pinning rule, observed in
   practice to recur unless mechanically caught rather than left to a generation-time
   reminder alone.

---

## Appendix

See `CHANGELOG.md` for this file's full version history.
