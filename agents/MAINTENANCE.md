# Maintenance Phase Process

**MANDATE: Adherence to this guide is not optional. It is a strict requirement.**

Governs corrective and adaptive/perfective maintenance on a Rust project **after** its
Design and Development Phases are otherwise complete — bug fixes, enhancements, and new
features. Distinct from `DESIGN.md` (initial architecture/planning) and `DEVELOPMENT.md`
(initial implementation), which this file assumes have already run at least once. See
`CHANGELOG.md` for this file's own version history.

Scope is corrective and adaptive/perfective maintenance only (ISO/IEC/IEEE 14764's
categories) — a change with no bearing on correctness or capability isn't routed through
this file.

**Dedicated templates, Development's own files untouched.** `maintenance_batch_template.md`,
`maintenance_checklist_template.md`, and `maintenance_prompt_template.md` are Maintenance's
own files — `development_plan_template.md`, `development_checklist_template.md`, and
`dev_prompt_template.md` are never modified or shared to serve this process, to keep the
proven Development pipeline's own files and outputs unaffected.

---

## 1. Relationship to Design and Development

Reuses existing mechanisms rather than inventing parallel ones: branch-per-unit-of-work
with exact-name-verbatim discipline; Claude-drafts/Jules-executes file-relay hand-off, no
direct API/credential integration between sessions (`AGENT_TOOL_POLICY.md`); append-only,
edited-in-place logging (Appendix B/F, same convention as `CHANGELOG.md`); the Complexity
Score formula and Task Template shape (`development_plan_template.md` §8), reused not
reinvented.

**Claude-side Maintenance sessions (M1–M3 below) operate in Advisory Mode** — the same
operating contract `CLAUDE.md` §1 defines for Design Phase (no persistent repo access,
Claude produces downloadable files, the user commits, session-boundary file-delivery
discipline applies). `CLAUDE.md` §1 governs both phases directly; its remaining sections
(§§2–6) stay Design-Phase-specific.

## 2. Starting and Resuming a Maintenance Session

**Starting a new item** — mirrors Design's own opening ("user states a goal or concept,
Claude helps expand"), no tier/type asserted up front since that's decided at M3:

```
Start Maintenance for [projectname]: [one-line description of the bug or idea]
```

**Resuming an already-open item:**

```
Resume Maintenance for [projectname], item [ID]
```

Same file-attachment discipline as any other session boundary — nothing carries over from
chat memory; the resuming session needs the current batch/checklist/prompt files
re-provided.

## 3. Two Entry Points — Never Interleaved

**Already-completed project** (Final Verification passed): run once, before any item is
worked, using the single merged prompt in **Appendix G** below. That prompt itself
enforces the two sub-phases and the hard stop between them:
1. **Phase Final+1a — Readiness Audit** (read-only): run the full Productization
   Readiness Checklist (`DEVELOPMENT.md` §5.2.4, all nine items) against current state.
   Output is a gap report, **one finding per checklist item**, each stated as its own
   discrete unit — never summarized or batched together.
2. **HARD STOP.** Present the gap report and wait for the user's explicit decision on
   **each finding individually** (proceed as scoped / adjust scope / defer) before any
   part of Phase Final+1b begins. This is not a formality — no remediation work of any
   kind starts until every finding has an explicit per-finding decision recorded.
3. **Phase Final+1b — Remediation**: close every gap as a normal task list, per the
   user's per-finding decisions from the stop above. **Item 1 (Regression Scaffold) is a
   hard gate** — nothing else is worked until it closes, using the Regression Scaffold
   definition in §6a below. Every other gap becomes an ordinary item in the currently
   open Maintenance Batch, classified like anything else at M1.

A project built under `DEVELOPMENT.md` §5.2.4 (v0.9.4+) should find this Audit clean —
a finding here signals a gap in that checklist, not just a sloppy project. A project that
predates the Regression Scaffold's introduction (v0.9.4) will typically find item 1's gap
report reads **"traceability file missing entirely"** rather than merely incomplete — see
§6a for why this is the same gap at maximal size, not a different problem.

**In-flight project** (Development Plan not yet at Final Verification): finishes its own
Plan first — an urgent bug uses the existing Escalation Trigger mechanism
(`development_plan_template.md` §13), not early Maintenance entry. If it predates v0.9.4,
it runs the same Readiness Audit (Appendix G) once it reaches its own Final Phase, rather
than a separate untracked retrofit.

## 4. Routing: Every Item Starts With Claude

There is no user-judgment classification at intake and no Track A/B split decided in
advance. **Every item — bug or feature/enhancement alike — starts at M0 with Claude.**
Whether Jules ever needs Claude's full Architecture Synthesis is itself decided
mechanically, at M1 (Impact Triage, §6 below), not guessed at intake. This removes the
risk of an architecturally-significant bug being waved past Claude just because it was
labeled "a bug" at the door.

**Type** (`fix` | `feature` | `enhancement`) is still recorded, descriptively, once known
— but it no longer determines *routing*. Routing is determined solely by the M1 Impact
Triage's lightweight/full outcome.

## 5. Intake

**Bug or idea:** add an item to the currently open Maintenance Batch file — created fresh
if none is currently open, reused if one already is. **A Future Feature idea not yet
approved for active work** is instead logged in the Architecture Specification's own
**§2.6** (Future Features / Deferred Scope) — the same section Design Phase already uses.
**Promotion** pulls a §2.6 item into the open batch (carrying forward its Originating ID,
if reserved, into M2's Requirement ID field) and removes it from §2.6 at that point, per
§2.6's own existing rule. A rejected (not promoted) idea moves to §2.5 (Out-of-Scope
Features) — the same register Design Phase uses, regardless of which phase rejected it.

**Batch composition is fixed at open time.** A Maintenance Batch is not drip-fed — every
item intended for a given batch is already known before M0 work begins on any of them.
This is what allows §11's provisional versioning to be assigned once, at batch-open,
rather than left open-ended.

## 6. The M0–M4 Process

Not every item runs every step — **M1's Impact Triage decides, mechanically, whether the
item takes the lightweight path (straight to Jules artifacts) or the full path (M2→M3→M4→
Jules artifacts).** M0–M3 are Claude-side, Advisory Mode (§1); the Jules-facing artifacts
(prompt + checklist) are what Jules actually executes, always generated regardless of
path.

### M0 — Elicitation
The user presents the request (bug, feature, whatever). **Claude and the user discuss and
flesh out the request together** — this is an interactive step, not Claude taking
dictation from a one-line ask. Before moving on, **Claude gets the user's explicit
confirmation on two separate things**: (a) that the discussion represents the *complete*
scope intended for this item, and (b) that the discussed result meets the user's approval.
**No file is presented at M0** — the discussion happens conversationally; nothing is
drafted into the batch file yet. No User Story is generated at M0 either — that only
happens at M1, and only on the full path (below), since a lightweight item never needs
one.

### M1 — Impact Triage (lightweight/full decision)
Reads only `03_requirements` (plus `04_test_strategy` §3.1's 9-Criteria reference table,
*only if* M0's discussion already suggests a Requirement is being touched) against M0's
confirmed discussion — no other Architecture Specification file is read at this step.
Five mechanical yes/no questions, applied to the item as confirmed at M0:

1. Does this require adding, modifying, or removing any Requirement ID (functional or
   non-functional) as currently written in `03_requirements`?
2. Does it introduce a new external interface, new data-model/schema field, or a new
   component/module boundary?
3. Does it touch more than one existing architectural component (cross-cutting), rather
   than staying inside one?
4. Does it change the public API / consumer contract (relevant only where Spec §10 is
   populated)?
5. Does it require a new dependency or infra service not already on
   `PREFERRED_DEPENDENCIES.md`/`PREFERRED_SERVICES.md`?

**All five "no" → lightweight path.** M1 exits by stating a provisional SemVer bump
directly (near-always `patch`, since an all-no outcome strongly implies non-breaking,
non-Requirement-touching change) and hands straight to Jules-artifact generation — no
User Story, no M2, no M3, no M4. **The first file presented to the user, for a lightweight
item, is the Jules prompt + checklist entry itself** — nothing is drafted or shown before
that.

**Any "yes" → full path.** Claude generates the User Story/Stories from M0's discussion
now, in the project's real `US-[DOMAIN]-NNN` numbering (appended to `02_user_stories`,
not a separate Maintenance-specific ID scheme — a story is a story regardless of which
phase produced it), and continues to M2. **No file is presented yet at M1's full-path
exit** — the first presented file on the full path is at M3 (below).

### M2 — Requirements, Test, Verification *(full path only)*
Which Requirement ID(s) this item touches (existing, modified, or new), what test/
regression coverage applies. **Self-check after M2:** the 9 Requirement Quality
Criteria / Requirement Smell catalog (`DESIGN.md` §4.5), applied only to whatever
Requirement text this item actually touched — mechanical, not a gated audit.

### M3 — Classification, Architecture Synthesis, Impact Assessment *(full path only)*
**This is the earliest point any file artifact is presented to the user on the full
path.** Determines Tier, Type, and SemVer recommendation for the item:
- **Tier:** 1 — Trivial | 2 — Standard | 3 — Architectural/Feature — descriptive severity,
  recorded but no longer used to route around Claude (§4).
- **Type:** `fix` | `feature` | `enhancement` — descriptive, not a separate process lane.
- **Architecture Synthesis:** real ISO-42010-style content (interface/data-model/viewpoint
  impact), reading only whichever of `06_viewpoints`/`07_interfaces_and_stack`/
  `08_constraints_and_roadmap` the M1 question(s) that fired actually implicate — not all
  three by default even on the full path.
- **Impact Assessment → SemVer recommendation**, stated explicitly per item:
  ```
  Impact assessment: [what changed, concretely] → Recommended bump: patch | minor | major
  Rationale: [why this is/isn't user-perceptible enough to warrant more than patch]
  ```
  This is a judgment call the user confirms or overrides, not a mechanical formula — a
  10% performance gain and a 2x one are the same *kind* of change with different
  *magnitude*, and magnitude — not category — drives the recommendation.
- **Asset Manifest** (feature/enhancement items with visual/media content only): same
  mechanics as `CLAUDE.md` §3.7 — table (Filename, Type, Repository Target Path,
  Authoritative/Informative For, Authority Level, Provided At), fixed paths
  (`assets/html|images|audio|video/`), never re-attached at a session boundary, only
  referenced by filename.

### M4 — Task Decomposition *(full path only)*
Generates this item's tasks, following the standard Task Template shape (Design Refs,
Verification Method, DoD, Submit Point — `development_plan_template.md` §8's shape,
reused directly). **Self-check after M4:** every task has a Verification Method and DoD —
mechanical completeness check, not a gated audit. **Complexity Score
(`development_plan_template.md` §8's formula, reused unmodified) applied across the
batch's full task list** determines Phase count in the generated Checklist — one batch is
at least one Phase, more if the ceiling requires a split, exactly as an over-ceiling
Development phase already gets split.

**One item = one Phase** in the generated `maintenance_checklist_template.md` output —
Entry Criteria from M0/M2's preconditions (or, on the lightweight path, from M1's exit
directly), Exit Criteria = Verified (§10 below), tasks from M4 (full path) or directly
from M1's exit (lightweight path), identical Task/DoD/Submit-Point/Session-Log shape as
Development's own Checklist.

**Jules artifacts — always generated, one prompt + one checklist per batch**, regardless
of how many items in the batch took the lightweight path versus the full path. Each
item's own Phase within these shared files reflects only the depth its own path actually
produced.

## 6a. Regression Scaffold — Definition, Naming Convention, and Retrofit Policy

This section is the canonical definition referenced by `DEVELOPMENT.md` §5.2.4 item 1,
by Appendix G's gap report, and by Phase Final+1b's hard gate — defined once, here.

**The gap this closes:** a Requirement-to-Test traceability table
(`test/[projectname]_requirement_traceability.md`) asserting a test is "independently
re-runnable by name/tag" is meaningless without a concrete mechanism connecting an
abstract `TEST-[NNN]` catalog ID (Architecture Spec §3.2) to an actual, invocable test in
source. The naming conventions below are that mechanism.

**Rust tests:**
```rust
#[test]
fn test_<nnn>__<snake_case_description>() { /* ... */ }
```
e.g. `TEST-0042` → `fn test_0042__login_rejects_invalid_password()`. Underscore-only (no
dashes — `PREFERRED_DEPENDENCIES.md`'s identifier rule / `RUST_PREFERENCES.md`), the
Test ID's numeric portion left-padded to 4 digits, a fixed double-underscore separating
the ID segment from the free-text description so parsing is unambiguous. **The
description segment is 3–5 words, no more** — enough to identify the test at a glance in
`cargo nextest` output, not a restatement of its full assertion (that belongs in the Test
Case Catalog, Spec §3.2, not the function name). Resolves via
`cargo nextest run test_0042` (substring match) or `cargo nextest run test_0042 --exact`
(exact match) — no new tooling, no tag plugin. One `TEST-[NNN]` = one canonical test
function by default; a test legitimately covering multiple Test IDs names as
`test_0042_0043__description()` (description still 3–5 words), and nextest substring
match on either ID still resolves it.

**Non-Rust tests (Playwright/E2E):**
```javascript
test('TEST-0042: login rejects invalid password', async ({ page }) => { /* ... */ });
```
Same **3–5 word** limit on the description following the colon. Resolves via
`npx playwright test -g "TEST-0042"` — Playwright's own native title-filter mechanism, no
new tooling.

**Test ID → Requirement ID mapping** stays exactly where it already lives — the Test Case
Catalog (Spec §3.2) and Traceability Matrix (Spec §3.4), which already support
many-to-many. The naming convention above only fixes the previously-missing link: Test ID
→ actual runnable name.

**Verification script** (`scripts/verify_traceability.js` — a project-specific script
authored during Development Phase, not a grail-supplied file; grail specifies its
required behavior here, the same way it specifies `ci.yml`'s shape without shipping a
finished workflow). **Required behavior:**
- Parses every row of `test/[projectname]_requirement_traceability.md`; tolerant of
  column order, locating the Requirement ID and Test ID columns by header name rather
  than fixed position.
- For each cited Test ID, confirms a matching `test_<nnnn>__*` Rust function or
  `TEST-<nnnn>: ...` Playwright title actually exists in source — not merely that the
  traceability row names one.
- **Actually runs** the corresponding test (`cargo nextest run <id> --exact` for Rust,
  `npx playwright test -g "TEST-<nnnn>"` for Playwright/E2E) and confirms it resolves to
  **exactly one** test and passes — never assumes a name's correctness from static
  inspection alone.
- Flags, as distinct categories: **orphan Requirement IDs** (a Core Requirement ID with no
  traceability row at all — requires the Architecture Specification's Requirement list to
  cross-check against), **dangling rows** (a traceability row citing a Test ID with no
  matching test found in source), and **naming-convention violations** (a test found but
  not matching the `test_<nnnn>__description` / `TEST-<nnnn>: description` shape).
- Exits non-zero if any row fails any of the above checks, so it can be wired into CI or
  used as the Readiness Audit / Remediation hard-gate signal (§3 above) rather than relying
  on a human eyeballing output.
- This is what makes `development_checklist_template.md`'s `PROD-001` DoD item — "every
  listed test confirmed independently invocable by name/tag (actually run, not assumed
  present)" — a mechanical check rather than a manual trust-fall.

**Retrofit policy (pre-v0.9.4 projects, or any project whose traceability table is
partial or absent):**
1. **A missing `test/[projectname]_requirement_traceability.md` file is the same gap as
   an incomplete one, at maximal size** — "0 of N Core Requirement IDs have a traceability
   row" rather than "M of N." The remediation process is identical either way; only the
   size of the gap differs. Appendix G's gap report distinguishes "missing entirely" from
   "exists but incomplete" per-finding, since a user approving the remediation should know
   which they're signing up for.
2. Enumerate every Core Requirement ID from the Architecture Specification.
3. For each, locate the existing test(s) covering it — manual matching against existing
   test names/bodies, since no naming convention existed before this policy. This step is
   real archaeology, not automatable by the verification script above.
4. **Default: rename the existing test function in place** to the `test_<nnnn>__desc`
   convention once a Test ID is assigned to it. A Rust `#[test]` rename is safe — no
   external callers by construction.
5. **Exception, not default:** if the existing test's name genuinely cannot be changed
   (e.g. proc-macro-generated test names, a framework that derives the name mechanically),
   add a thin wrapper (`fn test_<nnnn>__desc() { existing_test_fn_or_call() }`) instead —
   logged as a named exception in `docs/[project_name]_dev_risks.md`
   (`dev_risks_template.md`), same tier as any other Development-Phase-discovered standing
   deviation. This is a logged exception, not a silently-accepted default.
6. Any Core Requirement ID with **no** existing corresponding test is itself a gap this
   phase must close by writing a new test — never deferred, since this is the hard gate.

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

Regression scope is determined jointly by Claude and the user at M2/M3 — never by Jules
alone, never inferred from the diff after the fact. Jules verifies the stated scope; it
does not independently decide scope during implementation.

## 8. Generated Artifacts and Filenames

Three files, **named with their real target version from the moment the batch opens** —
no placeholder/`_open` staging name, no rename step at release:

| Content | Template | Filename (assigned at batch-open) |
|---|---|---|
| Spec-equivalent (M0–M3, per item) | `maintenance_batch_template.md` | `[projectname]_[type]_v[N.NN.NN].md` |
| Checklist (M4 output, or M1 exit on the lightweight path) | `maintenance_checklist_template.md` | `[projectname]_v[N.NN.NN]_checklist.md` |
| Jules hand-off prompt | `maintenance_prompt_template.md` | `[projectname]_v[N.NN.NN]_prompt.md` |

Because batch composition is fixed at open time (§5) — no drip-feed — the target version
is computed **once, at batch-open**, as the roll-up across every item already known to be
in the batch (highest of all items' individual SemVer recommendations, per §11). `[type]`
is `fix` | `feature` | `enhancement`, reflecting whichever category actually drove that
roll-up.

**Sole exception — a mid-batch rename:** if Jules's escape valve (§12) reveals an item is
materially bigger than its M1 Impact Triage suggested, and this genuinely changes the
batch's target SemVer bump, the files are renamed at that point as a real, rare
correction — not a routine occurrence, and not evidence that the no-drip-feed rule (§5)
has been violated.

## 9. Hand-off to Jules

The generated Prompt file (§8) points Jules at the generated Checklist and the batch's
M0–M3 content for the item(s) it names — same file-relay pattern as Design→Development, no
direct integration, no shared credentials. Jules implements per the Checklist's tasks,
runs the stated regression scope, and reports back (PR/diff/summary) for the user to relay
into a Verification session.

**Escape valve.** If, once underway, an item's actual scope reaches further than what was
briefed — whether briefed via the full M0–M4 pipeline or handed off directly from M1's
lightweight-path exit — Jules stops and reports back for a scope revision rather than
proceeding wider unilaterally (§12). This applies identically regardless of which path
produced the hand-off: **"M1 said lightweight but the actual implementation surfaced real
architectural impact" is the same escalation as "the briefed scope was too narrow,"**
not a special case requiring separate handling. A lightweight-path item that trips this
valve returns to Claude for what amounts to a fresh M1/M2/M3 pass on the now-better-
understood item — the user does not need a separate mechanism for this; §12's existing
Escalation Trigger already covers it.

## 10. Verification

A Claude session reviews Jules's result against the item's M0–M3 content (or, for a
lightweight-path item, its M1 exit statement) and Checklist: does it match root
cause/scope, was the stated regression scope actually run, does anything suggest the scope
was too narrow. Marks the Phase's Exit Criteria met in the Checklist. Flags — does not
silently fix — anything inconsistent. Does not submit/merge/tag; that stays user-directed.

**Phase-boundary scope rule:** a session verifying/closing Phase N's Exit Criteria checks
**only Phase N's own Exit Criteria** — it never also checks, confirms, or comments on
Phase N+1's Entry Criteria. Phase N+1 is always opened in a fresh session, and that
session checks its own Entry Criteria itself, at that time — this is not a redundant
re-check, it is the correct and only place that check belongs. A session that reaches
forward into the next Phase's Checklist section, however trivial the look-ahead seems, is
out of its own scope.

## 11. Release Checklist and Versioning

- **Batching at user discretion, composition fixed at open** — the user assembles the
  full intended item list before M0 work starts on any of them (§5); items are not added
  to an already-open batch mid-stream.
- **SemVer bump = highest confirmed recommendation across every item in the batch**,
  computed once at batch-open (§8) — an all-patch batch stays patch; any item recommending
  minor or higher makes the whole batch's target at least that; a breaking change is
  major. Re-confirmed, not re-derived, at actual release time.
- **Fix-forward only, one supported line** — no concurrent maintenance of older release
  branches at this time. Every Appendix B/F row records its shipping version, which is
  what a future multi-line model would need — adding one later is additive.

**Every release (patch baseline):**
- [ ] Every item's Checklist Phase is Exit-Criteria-met (Verified), or explicitly carried
      over to the next batch (noted, not silently dropped)
- [ ] SemVer bump re-confirmed against the batch-open roll-up (§8/§11); any mid-batch
      correction (§8's sole exception) accounted for
- [ ] Appendix B/F rows added for every item, referencing the released batch file + item ID
- [ ] `CHANGELOG.md` entry added; `[Unreleased]` section emptied
- [ ] Repo tagged (filenames already carry their real version from batch-open — §8 — so no
      rename step is needed here, absent §8's sole exception)

**Additional, MINOR or higher:**
- [ ] New/modified Requirement ID(s) reflected in Appendix F, old→new text
- [ ] Any Asset Manifest entries (§6 M3) confirmed logged with fixed repo paths present
- [ ] README updated if the new capability changes Quick Start/Tech Stack/feature list
- [ ] Aggregate regression scope re-confirmed adequate for the whole batch, not just
      per-item

**Additional, MAJOR only:**
- [ ] Breaking change(s) explicitly called out in `CHANGELOG.md` with migration guidance
- [ ] Deprecation/removal notices added where applicable
- [ ] Consumer-facing contract (Architecture Spec §10) reviewed and updated if affected

## 12. Escalation

Stop, summarize, wait (`development_plan_template.md` §13) when: an item's M1 Impact
Triage or M3 classification is genuinely ambiguous between two outcomes with materially
different consequences; Jules's implementation reaches further than the Checklist's
stated scope, **including a lightweight-path item that turns out to have real
architectural impact** (return to Claude for a scope revision, not unilateral expansion —
§9's escape valve); or anything that would itself raise a standing safety concern outside
this framework's scope, handled as it would be in any other context.

---

## Appendix G — Merged Readiness Audit + Remediation Prompt

> Use this prompt verbatim to start §3's "Already-completed project" entry point. It
> enforces the hard stop between the gap report and any remediation work — the two
> sub-phases are never run in a single uninterrupted pass.

```
Start Maintenance Readiness Audit for [projectname] — this project completed Final
Verification before v0.9.4 (or has otherwise never had the Productization Readiness
Checklist run against it, per DEVELOPMENT.md §5.2.4). This is Phase Final+1a per
MAINTENANCE.md §3 — read-only, no fixes yet.

Run all nine Productization Readiness Checklist items against the attached current
project state and produce a gap report. For item 1 (Regression Scaffold), explicitly
state whether the traceability file is MISSING ENTIRELY or EXISTS BUT INCOMPLETE
(MAINTENANCE.md §6a) — these are different-sized asks and must be distinguished.

Present the gap report as one finding per checklist item, each on its own, and then
STOP. Do not proceed to any remediation. Wait for my explicit decision on each finding —
proceed as scoped, adjust scope, or defer — before continuing.

Once I have given a decision on every finding, proceed to Phase Final+1b Remediation per
MAINTENANCE.md §3, per my per-finding decisions above. Close item 1 (Regression Scaffold)
first — it is a hard gate; nothing else in this remediation proceeds until it closes, per
the retrofit policy in MAINTENANCE.md §6a. Build (or complete) the requirement
traceability table and confirm every existing test is independently runnable by name/tag,
using scripts/verify_traceability.js to confirm rather than assume.
```
