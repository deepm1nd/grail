# Release Checklist: [projectname]_release_v[N.NN.NN]_checklist

> Companion to `release_plan_template.md` and `RELEASE.md`. Lives at
> `dev/release/v[N.NN.NN]/[projectname]_release_v[N.NN.NN]_checklist.md` — same
> folder-plus-filename version redundancy as the Plan, for the same provenance reason. Named
> `[projectname]_release_v[N.NN.NN]_checklist.md` from the moment this release's RELEASE
> work opens — same no-`_open`-placeholder, no-rename-at-publish convention as the Plan.
> Edited in place across this release's RELEASE work. Mirrors
> `development_checklist_template.md`'s Entry/Exit/DoD/Submitted/Session-Log shape, at
> **Step** granularity rather than Task Group/Task — RELEASE has no Task Group layer, and
> the individual sub-items a Step produces are deliberately left unnamed (`RELEASE.md` §6);
> DoD items below are plain checkboxes without a formal collective name for what each one
> represents.

## How to Use
- One `## Step N: <Title>` section per `release_plan_template.md` §2's Step Index, same
  order.
- Each Step opens with **Entry Criteria** (copied from the Plan) and closes with **Exit
  Criteria**, checked only when every DoD item above it is.
- `[ ]` not done · `[x]` done · `[D]` deliberately deferred (only if the Plan explicitly
  marks a Step as deferred for this release — e.g., an omitted Diátaxis quadrant per
  `RELEASE.md` §4).
- **No compressed formats** — every Step is written out in full, in order, per
  `AGENTS.md` §2.3.
- **Steps 4–6 (Tutorials/How-To Guides/Explanation) are order-independent** (`RELEASE.md`
  §6.4–6.6) — worked in whichever sequence the Plan/user chooses, one Step at a time, never
  concurrently.
- **`scripts/run_ci.sh` (v0.12.8):** any Step touching buildable/testable content (Reference-Sync,
  Finalize) runs it and presents output verbatim per `agents/CI.md` §6's mechanical contract,
  as part of that Step's own DoD, before its Exit Criteria can be checked.
- **Visual State Capture Completeness (v0.12.8, `development_plan_template.md` §8) applies**
  to any Step touching UI/doc-site visual content — every distinct visual state introduced
  or changed gets its own screen capture, plus an asset-fidelity comparison where an Asset
  Manifest entry governs it.
- **Submitted** checkbox — every Step carries its own, checked once its declared Submit
  Point actually fires via `submit`. Per `agents/AGENT_TOOL_POLICY.md` §2 (v0.12.8), when a
  Step's DoD/Exit Criteria are genuinely satisfied with no issue requiring intervention,
  `submit` fires directly — no "Should I proceed?"/"finalize and submit?" confirmation turn
  beforehand — and the call itself then pauses the session for the user's "Continue"/
  "Proceed" response.

---

## Step 1: Elicitation

**Entry Criteria:**
- [ ] (none — first Step)

- [ ] Scope statement (in-scope/out-of-scope) drafted and user-confirmed
- [ ] Audience/persona summary drafted and user-confirmed
- [ ] Diátaxis quadrant selection confirmed (`RELEASE.md` §4)
- [ ] `web/site/` content-type checklist selection confirmed (`RELEASE.md` §5)
- [ ] Seed material reviewed: tagged User Stories, tagged Requirements, flagged
      `CHANGELOG.md` entries
- [ ] **Submitted**

**Exit Criteria:** [ ] §1 output complete and user-confirmed

---

## Step 2: Scaffold

**Entry Criteria:**
- [ ] Step 1 Exit Criteria met

- [ ] `book.toml` / `SUMMARY.md` chapter stubs match Step 1's quadrant/content selections
- [ ] Theme/CSS, branding placement configured
- [ ] Preprocessor config (mermaid/admonish/etc.) wired
- [ ] Publish pipeline stubbed (build/test/link-check/deploy — mechanics only, no content)
- [ ] Skeleton builds cleanly (empty content, correct structure)
- [ ] **Submitted**

**Exit Criteria:** [ ] Skeleton builds and matches Step 1's selections

---

## Step 3: Terminology/Concept Spine

**Entry Criteria:**
- [ ] Step 2 Exit Criteria met

- [ ] Glossary drafted from Architecture Spec §1.5, adapted for docs-reader audience
- [ ] Narrative outline drafted
- [ ] User-confirmed
- [ ] **Submitted**

**Exit Criteria:** [ ] Glossary/outline settled and user-confirmed

---

## Step 4: Tutorials

**Entry Criteria:**
- [ ] Step 3 Exit Criteria met

- [ ] Draft complete for every Tutorial-tagged User Story (§1's seed material)
- [ ] No conceptual "why" digressions (Diátaxis purity principle, `RELEASE.md` §4)
- [ ] **Submitted**

**Exit Criteria:** [ ] All Tutorial-tagged stories drafted

---

## Step 5: How-To Guides

**Entry Criteria:**
- [ ] Step 3 Exit Criteria met

- [ ] Draft complete for every How-To-tagged User Story
- [ ] No conceptual "why" digressions
- [ ] **Submitted**

**Exit Criteria:** [ ] All How-To-tagged stories drafted

---

## Step 6: Explanation

**Entry Criteria:**
- [ ] Step 3 Exit Criteria met

- [ ] Draft complete per Step 1's Explanation scope
- [ ] No step-by-step instructional content (Diátaxis purity principle)
- [ ] **Submitted**

**Exit Criteria:** [ ] Explanation content drafted per scope

---

## Step 7: Reference-Sync

**Entry Criteria:**
- [ ] Steps 4–6 Exit Criteria met
- [ ] **`MAINTENANCE.md` §11 Release Checklist state confirmed for this version** (or
      `DEVELOPMENT.md` Final Verification, first release only) — this Step does not begin
      without this, regardless of Steps 4–6's own completion

- [ ] Rustdoc/doctest content confirmed current and CI-verified for every `pub` item
      touched since the last release (continuous-documentation discipline,
      `grail_continuous_documentation_proposal_v1.md`) — if not, flag per §4/Escalation,
      this Step reverts to authorship
- [ ] Reference quadrant populated via extraction (Jules) or linked to generated
      rustdoc/docs.rs-style output
- [ ] **Submitted**

**Exit Criteria:** [ ] Reference quadrant complete and as-built-accurate

---

## Step 8: Finalize

**Entry Criteria:**
- [ ] Step 7 Exit Criteria met

- [ ] Steps 4–6's drafted prose reconciled against Step 7's Reference content
- [ ] No known technical inaccuracies remaining
- [ ] **Submitted**

**Exit Criteria:** [ ] Draft content reconciled, no known inaccuracies

---

## Step 9: Consistency Audit

**Entry Criteria:**
- [ ] Step 8 Exit Criteria met

- [ ] Every Spine (Step 3) term checked for consistent use across all quadrants
- [ ] No orphaned or drifted terminology found (or all findings resolved)
- [ ] Style-guide adherence checked
- [ ] **Submitted**

**Exit Criteria:** [ ] Consistency-audited, ready-to-publish

---

## Final Verification (Release-Level Definition of Done)

- [ ] Every Step above has Entry and Exit Criteria checked, in order.
- [ ] `mdbook build` succeeds locally.
- [ ] `mdbook test` passes (every code example compiles/runs).
- [ ] Link-check/lint pass clean.
- [ ] Release prompt file (`[projectname]_release_v[N.NN.NN]_prompt.md`) generated and
      handed off per `RELEASE.md` §10.
- [ ] Version-path publish and switcher update confirmed per `RELEASE.md` §7's tier
      (patch: `latest` updated in place; minor/major: new version-path snapshot).
- [ ] Major-tier releases only: migration-guidance content confirmed present in the docs
      site itself, not only `CHANGELOG.md`.

---

## Session Log

> One entry per session. Never edit a prior entry except to fix a factual error (log the
> fix as a new entry).

| Date | Step(s) touched | Completed | Aborted | Escalation (`RELEASE.md` §11) | Notes |
|---|---|---|---|---|---|
| YYYY-MM-DD | | | | | |
