# Maintenance Phase Process

**MANDATE: Adherence to this guide is not optional. It is a strict requirement.**

Governs corrective and adaptive/perfective maintenance on a Rust project **after** its
Design and Development Phases are otherwise complete — bug fixes and Future Feature
promotions. Distinct from `DESIGN.md` (initial architecture/planning) and `DEVELOPMENT.md`
(initial implementation), which this file assumes have already run at least once. See
`CHANGELOG.md` for this file's own version history.

Scope is corrective and adaptive/perfective maintenance only (ISO/IEC/IEEE 14764's
categories) — a change with no bearing on correctness or capability isn't routed through
this file.

**Two files, not one per item.** A **Maintenance Batch** file
(`maintenance_batch_template.md`) holds full detail for every item being worked toward one
release — one file per batch, not one per bug. **Future Feature ideas live in the
Architecture Specification's own §2.6** (Future Features / Deferred Scope) — no separate
backlog file; that section already exists for exactly this purpose and already defines
promotion as removal from itself. **Appendix B/F** in the Architecture Specification hold
only a terse pointer row per item, once released — full detail lives in the batch file,
once, never duplicated.

---

## 1. Relationship to Design and Development

Reuses existing mechanisms rather than inventing parallel ones: branch-per-unit-of-work
with exact-name-verbatim discipline (`dev_prompt_template.md` §3); Claude-drafts/
Jules-executes file-relay hand-off, no direct API/credential integration between sessions
(`AGENT_TOOL_POLICY.md`); append-only, edited-in-place logging (Appendix B/F, same
convention as `CHANGELOG.md`); Session Unit and Escalation Trigger concepts
(`development_plan_template.md` §11/§13) applied one item at a time instead of one phase
at a time.

## 2. Two Entry Points — Never Interleaved

**Already-completed project** (Final Verification passed): run once, before any
Bug/Feature work:
1. **Phase Final+1a — Readiness Audit** (read-only): run the full Productization
   Readiness Checklist (`DEVELOPMENT.md` §5.2.4, all nine items) against current state.
   Output is a gap report.
2. **Phase Final+1b — Remediation**: close every gap as a normal task list. **Item 1
   (Regression Scaffold) is a hard gate** — nothing else is worked until it closes. Every
   other gap becomes an ordinary item in the currently open Maintenance Batch, tiered like
   anything else.

A project built under `DEVELOPMENT.md` §5.2.4 (v0.9.4+) should find this Audit clean —
a finding here signals a gap in that checklist, not just a sloppy project.

**In-flight project** (Development Plan not yet at Final Verification): finishes its own
Plan first — an urgent bug uses the existing Escalation Trigger mechanism
(`development_plan_template.md` §13), not early Maintenance entry. If it predates v0.9.4,
apply the (one-off, untracked) retrofit trio before its own Final Phase.

## 3. Two Tracks

| | Track A — Corrective | Track B — Substantive |
|---|---|---|
| **Covers** | Tier 1/2 bug fixes | Tier 3 fixes, all Future Features |
| **Terse log (once released)** | Appendix B | Appendix F |
| **Touches a Requirement ID?** | Rarely (Tier 2 only) | Usually |
| **SemVer (this item alone)** | PATCH | MINOR (or MAJOR if breaking) |
| **Gate before Jules** | Tier 1: none beyond the batch entry itself. Tier 2: user approves the entry. | Always: user approves the entry, including its Requirement ID change. |

A Tier 3 item that started as a bug report stays in the same batch file (it's one file
per release regardless of kind) — only its `Kind`/`Track` fields and eventual Appendix
(B vs. F) differ.

## 4. Intake

**Bug:** add an entry to the currently open Maintenance Batch file — created fresh if none
is currently open, reused if one already is.

**Future Feature idea:** logged directly in the Architecture Specification's **§2.6
(Future Features / Deferred Scope)** — the same section Design Phase already uses for any
RATS-deferred item or explicitly deferred scope (`architecture_specification_template.md`
§2.6). No separate Maintenance-side backlog file; a Maintenance-session-originated idea is
logged there exactly the same way a Design-session-originated one is, with the same
per-item fields (Originating ID if one was reserved, What it is, Why deferred,
Dependency/precondition).

**Promotion:** moving a §2.6 item into active work adds it as a full item to the currently
open batch file (carrying forward its Originating ID as the batch item's Requirement
ID(s) field, if one was reserved) and, **per §2.6's own existing rule, removes it from
§2.6** at that point — not left there to accumulate as stale duplicate history. It then
proceeds through Triage (§5) like any other item. A §2.6 item that's instead *rejected*
rather than promoted follows §2.6's other existing disposition — **moved to §2.5
(Out-of-Scope Features)**, the same register Design Phase already uses for rejected scope,
regardless of which phase did the rejecting.

## 5. Triage and Tiering

A Claude session (not Jules) triages every new/`Reported`/`Proposed` batch item — a
short, structured pass, not a full gated Design session:

1. Reproduce (bug) or confirm scope (feature).
2. Classify **severity** (technical impact) and **priority** (business urgency) —
   separate fields in the batch entry, never conflated.
3. Assign a **Tier**, updating the entry's `Status`:

| Tier | Definition | Track |
|---|---|---|
| **1 — Trivial** | Reproducible, isolated, obvious root cause | A |
| **2 — Standard** | Real logic/data bug, moderate blast radius | A |
| **3 — Architectural/Feature** | Spec-level gap, or any Future Feature | B |

Tiering is a working hypothesis at intake — if a Tier 1/2 root cause turns out to need a
Requirement change (e.g. algorithm X → Y, or any Requirement superseding), reclassify to
Tier 3 in the same batch entry.

## 6. Briefing

Fill in the batch entry's remaining fields directly (root cause/design, Requirement
ID(s), regression scope, target branch, Instructions to Jules — see
`maintenance_batch_template.md`'s full field list), updating `Status` to `Briefed`. Tier 1
proceeds straight to hand-off once filled in; Tier 2/3 require explicit user approval of
the entry first.

**Regression scope is determined jointly by Claude and the user** — never by Jules alone,
never inferred from the diff after the fact. See §7 for starting points by project type.

## 7. Regression Scope by Project Type

A `retest-all` default is always safe but often wasteful; pure selective scope risks
missing cross-cutting impact. Starting points (combine as a project's actual shape
requires):

| Project type | Regression scope starting point |
|---|---|
| Web client/server | Contract tests for touched endpoint(s); E2E only if user-facing surface changed |
| CLI | Unit/integration for touched command(s); output-format stability check if formatted output changed |
| Embedded/IoT | HIL or simulator run for touched subsystem; full retest-all if shared toolchain/HAL touched |
| Multi-container services | Cross-service contract tests for every consumer of the touched interface |
| Headless server + remote client | Backend regression, plus screenshot-diff if rendering is affected |
| Library/SDK | API/ABI compatibility check in addition to the crate's own suite |
| Mobile/desktop/browser extension | Full retest-all often justified — store-review latency makes a missed regression expensive |
| Batch/data pipeline | Re-run against fixture datasets; diff actual vs. expected output |
| Serverless/on-prem/vendor-installed | Check the project's own Spec for deployment-model constraints not listed here |

Jules verifies the entry's stated scope; it does not independently decide scope during
implementation.

## 8. Hand-off to Jules

Use `maintenance_prompt_jules_template.md` — a thin, reusable prompt that points Jules at
the batch file and item ID (plus the Architecture Specification and any other files the
entry references) rather than restating the entry's content inline. Same file-relay
pattern as Design→Development: no direct integration, no shared credentials
(`AGENT_TOOL_POLICY.md`). Jules implements exactly as briefed, runs the stated regression
scope, and reports back (PR/diff/summary) for the user to relay into a Verification
session.

## 9. Verification

A Claude session reviews Jules's result against the batch entry: does it match root
cause/scope, was the stated regression scope actually run, does anything suggest the
scope was too narrow. Updates the entry's `Status` to `Fixed`/`Verified`. Flags — does not
silently fix — anything inconsistent. Does not submit/merge/tag; that stays
user-directed.

## 10. Versioning, Batching, and Release

- **One batch file per release, at user discretion** — items accumulate in the currently
  open batch file until the user decides to cut a release.
- **SemVer bump = highest bump any single item in the batch requires** — an all-Track-A
  batch is PATCH; any Track B item makes it at least MINOR; a breaking Requirement change
  is MAJOR.
- **Fix-forward only, one supported line** — no concurrent maintenance of older release
  branches at this time. Every Appendix B/F row already records its shipping version,
  which is what a future multi-line model would need — adding one later is additive, not
  a redesign.
- **At release** (`maintenance_batch_template.md`'s own Batch Release Checklist): confirm
  every item is `Verified` or explicitly carried over; add the terse Appendix B/F row for
  each item, referencing the batch file and item ID; add the `CHANGELOG.md` entry and
  empty `[Unreleased]`; tag the repo; rename/archive the batch file to its released
  version's filename; start a new open batch file for the next release.

## 11. Escalation

Stop, summarize, wait (`development_plan_template.md` §13) when: a Tier 1/2 root cause
needs a Requirement change (reclassify, §5); Jules's implementation reaches further than
the entry's stated scope (return to Claude for a scope revision, not unilateral
expansion); or anything that would itself raise a standing safety concern outside this
framework's scope, handled as it would be in any other context.
