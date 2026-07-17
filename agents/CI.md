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

## 3. Stage Skeleton (v0.9.2)

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
job** (§3, Stage 6) since it needs its own checkout/artifact-download context — it runs on
every branch, same as `ci_pipeline`, not under a different (`main`-only) trigger condition.

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
needs its own checkout/artifact-download context — if it needs any of the same tools, it
must install them itself.

### Stage 1: Lint & Format
```yaml
      - name: Check formatting
        id: check_formatting
        run: cargo fmt --all -- --check

      - name: Run Clippy
        id: run_clippy
        run: cargo clippy --workspace --all-targets -- -D warnings
        # -D warnings is mandatory on every project. -D clippy::unwrap_used is a
        # per-project opt-in (PREFERRED_TOOLS.md), added here only if that project
        # elected it at Design Step 5 — do not add it by default.
```

### Stage 2a: Native Build
```yaml
      - name: Native build
        id: native_build
        run: cargo build --release
```

### Stage 2b: WASM Build *(conditional — Trunk/WASM component present)*
```yaml
      - name: WASM build
        id: wasm_build
        run: trunk build --release
```
Per `PREFERRED_TOOLS.md`'s Trunk conventions (output-directory separation from `assets/`,
no separately-installed `wasm-bindgen-cli`).

### Stage 3: Test
Sub-staged by what each test group needs, not by a unit/integration distinction — Rust's
own tooling doesn't force that split at the runtime level, and nextest discovers both
colocated `#[test]`s and `tests/*.rs` integration tests in the same invocation regardless.

- **Stage 3a — Tests not requiring infra services:**
  ```yaml
        - name: Run nextest
          id: test_nextest
          env:
            NEXTEST_EXPERIMENTAL_LIBTEST_JSON: "1"
          run: cargo nextest run --workspace --message-format libtest-json

        - name: Run doc-tests
          id: test_docs
          run: cargo test --doc --workspace
  ```
  `cargo nextest run --workspace --message-format libtest-json` (`PREFERRED_TOOLS.md`'s
  Canonical Commands table) → parsed by `scripts/metrics/parse_tests.js` → `metrics/tests.toml`.
  Doc-tests run generically for whichever crates in the workspace have doc-tests; never
  hardcode a specific crate name into this step.
- **Stage 3b — Infra-dependent tests** *(conditional — `deploy/docker-compose.dev.yml`
  present)*: start infra first (`docker compose -f deploy/docker-compose.dev.yml up -d`,
  `PREFERRED_SERVICES.md`'s Session Startup section), then run the test subset that needs
  it.
- **Stage 3c(+) — E2E/Playwright** *(conditional — Playwright/E2E suite present)*:
  ```yaml
        - name: Install Playwright browsers
          id: playwright_install
          run: |
            cd test/playwright
            npm install
            npx playwright install --deps

        - name: Run Playwright tests
          id: playwright_test
          run: |
            cd test/playwright
            npx playwright test --reporter=json > /tmp/playwright.json

        - name: Parse Playwright results
          id: playwright_metrics
          run: node scripts/metrics/parse_playwright.js /tmp/playwright.json
  ```
  **Install browsers before running tests** — `npx playwright install --deps` is a
  mandatory step immediately before the test invocation; a fresh runner has no browsers
  installed and the test run will fail before producing any JSON at all without this. This
  CI-side browser install is distinct from `setup_env.sh`'s one-time environment-install
  guarded block (`PREFERRED_TOOLS.md`) — that block only fires when Playwright itself is
  entirely missing, not necessarily on every fresh CI runner in exactly the way CI needs, so
  both are required.
  **`playwright_test` is a hard gate, consistent with every other CI stage (nextest,
  coverage, deny, audit, drift-check) — no `continue-on-error: true` and no trailing
  `|| true` on the test-execution step itself.** `parse_playwright.js` → `metrics/playwright.toml`.
  This stage's outputs follow the same `test/phase_N/ci/` evidence-capture convention as
  every other stage — no special-casing. Additional lettered sub-stages (3d, 3e...) as a
  project needs further test-category subdivision — same conditional pattern, not a fixed
  ceiling.

### Stage 4: Coverage
```yaml
      - name: Run coverage
        id: coverage_run
        run: cargo llvm-cov --workspace --json --output-path target/llvm-cov.json

      - name: Parse coverage results
        id: coverage_metrics
        run: node scripts/metrics/parse_coverage.js target/llvm-cov.json
```
`cargo llvm-cov --workspace --json --output-path target/llvm-cov.json` → parsed by
`scripts/metrics/parse_coverage.js` → `metrics/coverage.toml`.

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
        id: cargo_deny
        run: cargo deny --format json check > /tmp/deny.json

      - name: Parse deny results
        id: parse_deny
        run: node scripts/metrics/parse_deny.js /tmp/deny.json   # or bash+jq if trivial

      - name: cargo-audit
        id: cargo_audit
        run: cargo audit --json > /tmp/audit.json

      - name: Parse audit results
        id: parse_audit
        run: node scripts/metrics/parse_audit.js /tmp/audit.json

      - name: Parse license violation/addition metrics
        id: parse_license_metrics
        run: node scripts/metrics/parse_license_metrics.js /tmp/deny.json
        # Writes metrics/licenses.toml (violations_count, crates_added_count — feeds
        # the two README license badges) and appends one line per newly-seen crate to
        # metrics/license_additions_log.jsonl or metrics/license_violations_log.jsonl
        # (branch, commit, date, crate, license, direct_dep, chain — chain derived
        # from cargo-deny's own inverse-dependency-graph output for the flagged crate).
        # Both logs are committed to the working branch itself; because each phase
        # branch descends from the prior phase's branch state, they accumulate
        # cumulatively across the whole project's history via ordinary git ancestry —
        # no cross-branch or branch-specific commit logic needed — every branch is
        # self-contained.

      - name: THIRD_PARTY_LICENSES.md drift check
        id: drift_check
        run: bash scripts/metrics/check_license_drift.sh
```

- **cargo-deny** covers both `[licenses]` (the permissive/notice-only allow-list,
  `PREFERRED_DEPENDENCIES.md`'s License Compatibility Criterion — see that file and
  `PREFERRED_TOOLS.md`'s `deny.toml` skeleton for the current 0.19.x-compatible schema) and
  `[advisories]` (`unmaintained = "all"` by standing default, with individual exceptions
  via a documented `ignore` entry per `PREFERRED_TOOLS.md`'s ignore-list policy — never a
  blanket scope relaxation to silence a specific finding).
- **cargo-audit** — RustSec vulnerability scan, independent of cargo-deny.
- **`THIRD_PARTY_LICENSES.md` format — lightweight, no prose.** The file is a one-line
  description followed by a dependency/license table (crate name, version, license
  identifier) — no header/footer prose, no full license text bodies, no per-license
  explanatory sections. This is the same format at every stage of the file's life: Design
  Step 8's initial draft (direct dependencies only, no `Cargo.lock` yet) is a shorter
  version of the identical table; CI's Stage 5 regeneration produces the complete
  resolved-tree version in that same format. Phase 0's reconciliation (`DEVELOPMENT.md`
  §5.2.3) is therefore a table expansion, not a restructuring.
- **`THIRD_PARTY_LICENSES.md` drift check** — regenerate the file (in the format above)
  from the current
  resolved dependency tree (fed by the same cargo-deny license enumeration) into a temp
  location, diff against the committed copy, **fail the stage on any difference — do not
  auto-commit.** A human reviews and commits the regenerated file deliberately; CI's job
  is to catch drift, not to silently resolve it, consistent with how this framework treats
  every other auto-fix decision (Trivial-vs-Substantive audit findings, `metrics/` commits
  at Stage 6 below being the one deliberate exception, made explicit because unlike a
  license disclosure, metrics have no compliance weight).

### Branch/Status Marker (`metrics/source.toml`), then Artifact Upload, then Evaluate Required Gates
Three steps close out `ci_pipeline`, in this exact order — **the order matters and is a
corrected fix from an earlier draft that had it wrong (see the note below).**

```yaml
      - name: Write branch/status marker
        id: write_source_marker
        if: always()
        run: |
          node scripts/metrics/write_source_marker.js \
            --branch "${{ github.ref_name }}" \
            --commit "${{ github.sha }}" \
            --status "${{ job.status }}"

      - name: Upload metrics artifact
        id: upload_metrics_artifact
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: metrics
          path: metrics/*.toml

      - name: Evaluate required gates
        id: evaluate_required_gates
        if: always()
        run: |
          success=true
          if [ "${{ steps.test_nextest.outcome }}" != "success" ]; then echo "nextest failed"; success=false; fi
          if [ "${{ steps.test_docs.outcome }}" != "success" ]; then echo "doc tests failed"; success=false; fi
          if [ "${{ steps.playwright_test.outcome }}" != "success" ]; then echo "playwright test failed"; success=false; fi
          if [ "${{ steps.playwright_metrics.outcome }}" != "success" ]; then echo "playwright metrics failed"; success=false; fi
          if [ "${{ steps.coverage_run.outcome }}" != "success" ]; then echo "coverage run failed"; success=false; fi
          if [ "${{ steps.coverage_metrics.outcome }}" != "success" ]; then echo "coverage metrics failed"; success=false; fi
          if [ "${{ steps.cargo_deny.outcome }}" != "success" ]; then echo "cargo-deny failed"; success=false; fi
          if [ "${{ steps.parse_deny.outcome }}" != "success" ]; then echo "parse-deny failed"; success=false; fi
          if [ "${{ steps.cargo_audit.outcome }}" != "success" ]; then echo "cargo-audit failed"; success=false; fi
          if [ "${{ steps.parse_audit.outcome }}" != "success" ]; then echo "parse-audit failed"; success=false; fi
          if [ "${{ steps.drift_check.outcome }}" != "success" ]; then echo "drift-check failed"; success=false; fi

          if [ "$success" = false ]; then
            exit 1
          fi
```

`metrics/source.toml` output shape:
```toml
[source]
branch = "phase-3-auth"
commit = "a1b2c3d"
status = "pass"        # or "fail" — reflects this run's actual pipeline outcome
updated = "2026-07-14T18:03:00Z"
```

**Ordering fix — why "Write branch/status marker" must come before "Upload metrics
artifact":** an earlier draft of this skeleton ran these in the order Stage 5 steps →
Upload metrics artifact → Evaluate required gates → Write branch/status marker. Because the
marker write (which produces `metrics/source.toml`) ran *after* the artifact upload, the
uploaded artifact never actually contained `source.toml` — it didn't exist yet at upload
time. The corrected order above writes the marker first, so the upload captures the
complete, current `metrics/*.toml` set, including `source.toml`.

**"Evaluate required gates" runs last, after the artifact upload, not before it** — every
`steps.<id>.outcome` reference above reflects that step's own pass/fail, ignoring any
`continue-on-error`, so the branch marker and artifact upload's own `if: always()` steps
still run and their content is captured even when an earlier stage failed; only the final
verdict step actually fails the job. Each referenced step's `id:` is defined earlier in this
job (§3's stage-by-stage yaml above) — omit or no-op the `playwright_test`/
`playwright_metrics` lines for a project without an E2E suite, same conditional-inclusion
pattern used elsewhere in this document. Because `playwright_test`'s outcome participates in
this gate as a hard failure, confirm the Playwright test step itself carries no
`continue-on-error: true` and no trailing `|| true` — a bare, ungated `npx playwright test`
invocation (§3 Stage 3c, above).

**This job's own working branch's README badge** reads `metrics/source.toml` (same
shields.io dynamic-TOML pattern as the other metrics badges) so it always reflects the
latest push's branch, commit, and pass/fail state on whichever branch that push was on —
updated on **every** push, not gated to a clean run. This is intentional: during active
development the README is functioning as a personal status dashboard, and a stale "last
known green" badge would hide exactly the information (a currently-red push) that badge
exists to surface. Consistent with the rest of this document's branch-local framing: you
check each branch's own README as you work — there is nothing `main`-specific about this
badge or the file that feeds it.

### Stage 6: Metrics Commit *(every branch, separate job)*
Bot commit of regenerated `metrics/*.toml` (including `source.toml`) back to the triggering
branch. Unlike the `THIRD_PARTY_LICENSES.md` drift check above, metrics are auto-committed
rather than fail-on-drift — they carry no compliance/legal weight, only informational/badge
value, so routine auto-refresh is acceptable where a license disclosure's drift is not. This
job remains genuinely separate from `ci_pipeline` (not folded into the merge above) because
it needs its own artifact-download/commit context — **not**, as an earlier draft implied,
because it runs under a different (`main`-only) trigger condition; it now runs on every
branch, same as `ci_pipeline`.

**Corrected job — replace any `main`-gated version with exactly this:**
```yaml
  metrics_commit:
    name: Metrics Commit
    needs: ci_pipeline
    if: always()
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with:
          ref: ${{ github.ref_name }}

      - name: Download metrics artifact
        id: download_metrics_artifact
        uses: actions/download-artifact@v4
        with:
          name: metrics
          path: metrics/

      - name: Commit updated metrics
        id: commit_metrics
        run: |
          git config user.name "<project>-ci-bot"
          git config user.email "ci-bot@<project>.example"
          git add metrics/*.toml
          git diff --staged --quiet || git commit -m "ci: refresh metrics [skip ci]"
          git push origin HEAD:${{ github.ref_name }}
```

**Three fixes from an earlier draft, and why each matters:**
1. **`if: always() && github.ref == 'refs/heads/main'` → `if: always()`** — this job now
   runs on **every** branch push, not `main` only. Metrics Commit is not a `main`-gated job
   at all.
2. **Bare `actions/checkout@v5` → add `with: ref: ${{ github.ref_name }}`** — without an
   explicit `ref`, checkout can leave the workspace in detached-HEAD state for the
   triggering commit. Pinning `ref` guarantees the job checks out the actual branch that
   triggered the run.
3. **Bare `git push` → `git push origin HEAD:${{ github.ref_name }}`** — a bare push from a
   detached-HEAD checkout has no reliable target branch. The explicit refspec guarantees the
   commit goes back to whichever branch triggered the run — never `main` specifically,
   whichever branch it actually was.

**Nothing in this CI pipeline is `main`-specific.** Every branch is self-contained: its own
metrics, its own README badge state, its own Metrics Commit run. If a future revision of
this document reintroduces a `main`-only assumption anywhere, that is a defect to fix, not a
default to preserve — this branch-uniform behavior is the settled design, not an interim
state.

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
