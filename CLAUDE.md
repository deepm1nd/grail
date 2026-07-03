# CLAUDE.md — Design Phase Working Arrangement

See `CHANGELOG.md` for full version history (human reference only; agents skip it).

Governs Claude's operation of the **Design Phase** of this repository, per `AGENTS.md` and
`agents/DESIGN.md`. Scope: producing the Architecture Specification, Development Plan, and
Development Checklist only — not Development Phase execution. Where this file conflicts with
a future revision of `AGENTS.md`/`agents/DESIGN.md`, those win; this is a working-arrangement
summary, not a replacement.

---

## 1. Operating Mode: Advisory

Per `AGENTS.md` §1.5.2, not Autonomous:

- No persistent repo access. Claude can't read repo state, commit, or verify prior files
  exist unless pasted, uploaded, or linked in-conversation.
- Claude produces deliverables as downloadable files; **the user commits them.** Wherever
  `AGENTS.md`/`agents/DESIGN.md` says "the agent must commit"/"ensure repository state,"
  read: Claude prepares, the user commits. `agents/DESIGN.md` §6 Phase Completion is
  satisfied once the user has actually committed — Claude can't verify this and won't claim it.
- Session-Initialization steps assuming shell access (`scripts/start_services.sh`, etc.)
  don't apply; Claude won't simulate running them.
- If a mandate assumes access Claude doesn't have, Claude says so rather than skipping or
  improvising silently.
- **Every Step, and every repeated run of a Step (user-requested per §3.10, or automatic,
  e.g. a Step 7 re-audit per §3.12), runs in its own separate chat session — a hard rule.**
  Step 2 (User Story Elicitation) is one session, not split into named passes (§3.4). Nothing
  "earlier in this conversation" can be referred back to once a new session starts; whatever a
  new session needs from a prior Step must be re-provided directly (uploaded/pasted/linked),
  same as any other file dependency in Advisory Mode. Consequence: **every Step Approval Gate
  (§3.4) is also a session-boundary packaging point** — presenting a step's output for approval
  always means packaging every file and handoff note the next session could need, not just the
  step's headline deliverable. See §3.11 for what this requires as files (not chat text), §3.12
  for Step 7 re-runs specifically (Step 9's backtrack mechanics, `agents/DESIGN.md` §5.9, mirror
  §3.12), and §3.13 for the open-items review every gate must also carry forward.

---

## 2. Scope, Templates, and Project-Specific Constraints

**Artifacts (3 total, per user direction — never a 4th):** Architecture Specification spans
**8 files, one per originating Design Step 1–6** (§4 for the mapping/rationale — supersedes
any earlier 3–5-file grouping). Development Plan spans 3–5 files, ~1000+ lines each.
Development Checklist is one file unless content genuinely warrants a split (flag and propose
before splitting). This sizing is a user scoping choice, not an `agents/DESIGN.md`
requirement — it determines packaging, not gate count. The Spec's finer 8-way split vs. the
Plan's coarser 3–5 is deliberate: each Spec file tracks exactly one Design Step's own
session-versioning history (§4), which only works cleanly at one-file-per-Step granularity;
the Plan has no equivalent per-step authorship history to preserve.

**Templates (all four provided, binding):**
`agents/exemplars/architecture_specification_template.md`,
`agents/exemplars/development_plan_template.md`,
`agents/exemplars/development_checklist_template.md`,
`agents/exemplars/dev_agent_prompt_template.md` (complex/multi-phase Rust variant).
Plan and Checklist stay in lockstep (one checklist line per task DoD item). A later template
revision means Claude conforms existing documents to it and flags the restructuring as a
Major Change (§3.6) — never silently.

**Stable IDs**, assigned at first draft, never renumbered (`US-`, `REQ-`, `TEST-`,
`THREAT-`, `ADR-`, task IDs per template schemas). A rejected/merged item's ID is retired, not
reassigned — this is what keeps cross-document traceability links from breaking.

**No separate Requirements & Traceability document.** Every such reference (template or
conversation) resolves to the Architecture Spec's Traceability content (template §3 — see §4
for which physical file currently holds it). Templates are corrected accordingly.

**File-split boundaries follow each template's own top-level sections**, grouped toward the
file-count target in force for that artifact — see §4 for the Architecture Spec's concrete,
Step-aligned grouping and naming convention.

### 2.1. Rust/WASM-Specific Constraints

Full detail: `agents/RUST_PREFERENCES.md` (naming/keywords, type-system/ownership mapping,
WASM/Web Worker concurrency, edition and MSRV policy), `agents/PREFERRED_DEPENDENCIES.md`
(Rust library policy, checked at Step 5), `agents/PREFERRED_TOOLS.md` (dev tool policy,
including MSRV-determination mechanics), `agents/PREFERRED_SERVICES.md` (infra service
policy — Docker deployment, `deploy/` conventions). This project is a Rust web framework: a
design goal is **multithreading wherever possible, native and WASM**, via Web Workers — not a
constraint to design around. Requirement language using null/inheritance/exception framing
that doesn't map onto `Option<T>`/traits/`Result` is a Requirement Smell (§3, Step 3) needing
translation, not pass-through. `agents/PREFERRED_TOOLS.md` is relevant here (dev tool/MSRV
policy); `agents/SCRIPT_RULES.md` is not (Advisory Mode runs no scripts).

### 2.2. Visual & Media Asset Input

Where a project has a human-facing UI, the user supplies reference **assets** — HTML,
images, and optionally audio/video — tracked as first-class, filename-stable artifacts across
both Design (this file) and Development, not consumed once and discarded. Full mechanism:
§3.8. Physical tracking record: the **Asset Manifest**, a table inside the Architecture
Specification (§3.8 states which file currently owns it under §4's Step-aligned split).

---

## 3. Core Mechanisms and the Gated Workflow

`agents/DESIGN.md` §5 defines a **9-step gated lifecycle** — no skipping, no combining, no
proceeding past a gate without explicit approval (full-rigor, all-interactive baseline; §3.7
for the autonomy toggle and its presets). §§3.1–3.3 define mechanisms shared across steps;
§3.4 lists the steps, stating only what's specific to each.

### 3.1. Propose-with-Flagged-Assumptions

Within any of Steps 1, 3–5: some work is **mechanical** (checkable against an explicit,
objective standard — e.g. the 9 requirement-quality criteria, deriving a test case from an
already-verifiable requirement, matching a tool to a need category) and some is
**substantive** (depends on facts about the user's actual needs/environment Claude cannot
verify — necessity, correctness, real feasibility). Claude does mechanical work unsupervised;
for substantive work it drafts its best answer but **flags every place it had to assume or
infer** rather than derive from something already stated. Only the user's explicit
confirmation — never Claude's own sense of "probably fine" — settles a flagged item.
Concretely: draft fully, flag inline, route everything (batch + flagged items) through the
Grouped Closing Protocol (§3.2), and never declare a set "sufficient"/"done" on Claude's own
judgment alone.

### 3.2. The Grouped Closing Protocol

How Claude closes any batch this phase produces (User Stories at Step 2; requirements, test
cases, tool choices, etc. at Steps 3–5) — governs **every** batch-producing step, not just
Step 2. Replaces one-item-at-a-time dictation while keeping the same guarantee: every item
individually visible, individually confirmed or rejected, before being final.

**Grouping:** the full batch is organized into groups at draft time (actor/theme/sub-topic),
decided once, not discovered incrementally.

**Per group, in order, not advancing until each stage settles:**
1. **Content pass.** Present the batch as plain numbered text. User strikes/edits/confirms/
   adds in bulk. Settles *what exists*, not attached assumptions.
2. **Assumption stage.** Table: **Item — Choices — Recommendation**, covering **every**
   flagged assumption on a surviving item, no exceptions — including items where the
   resolution seems obvious or is reuse of an accepted pattern; "doesn't need real
   deliberation" is not license to skip the table row (a one-line row still gets one). A
   struck item drops its assumption automatically. User resolves in bulk, or names specific
   items for individual treatment ("show me the analysis for items 6, 9, 13"). For named
   items: full research/analysis in prose (§3.3's three-layer requirement), then the table
   row(s), **then** the selectable-options question — no analysis option (already given;
   dead weight to re-offer) — keeping Defer + substantive choices, recommendation first and
   labeled (§3.3). Anything still open (Deferred, or ambiguous) flows to the next stage, not
   re-litigated here.
3. **Deferred-items stage.** Separate table (not merged) of everything marked Defer, any
   source. Same bulk-then-named pattern; leftovers flow forward.
4. **Remaining-ambiguous stage.** Separate table of what's still unresolved. Same pattern.
   Residue after this stage carries into later-step Defer resolution and, per §3.13, is
   re-surfaced at every subsequent gate until resolved or explicitly accepted as standing.
5. **Next group**, restarting at content pass.

**Bulk-reply handling, every stage — and any standalone flagged item outside the Grouped
Closing Protocol (e.g. Step 6's per-element notation judgment, Step 8's environment/config
assumptions):** one covering reply ("all accepted") closes the whole table. A partial reply
closes only what it addresses — **whatever it leaves unresolved automatically proceeds into
the per-item selectable-options loop (§3.3's three-layer presentation, then the tool), no
separate request required.** "Review individually" still works as an accelerator straight to
per-item, no longer the *only* door in. An ambiguous reply (genuinely unclear, not merely
partial) gets a plain clarifying question first (§3.1, `AGENTS.md` §3.1) — never treated as a
trigger for anything.

**Early-resolution exception:** if a content-pass reply already explicitly answers a flagged
assumption (not just touches the item), Claude doesn't re-ask later — unless the answer
raises a *new* unstated assumption, which still goes through the assumption stage.

**Completeness:** after the last group's last stage, Claude asks once whether anything's
missing from the batch overall — user confirmation closes it, not Claude's judgment.

**Expected scale:** bulk resolution should shrink each stage to a handful of items — an
expectation, not a hard cap; an unshrunk stage still runs against what remains.

---

### 3.3. The Selectable-Options Convention

Applies wherever Claude uses the selectable-options/multiple-choice tool (a genuinely
discrete choice — a multi-item list, like a Grouped Closing Protocol content pass, stays
plain text; see §3.2).

- **Three-layer presentation, every time — precondition for everything below.** The tool is a
  picker, not a display surface (truncates to ~one line), so it can never be the sole place
  substantive content lives. Before the tool is invoked, for any decision — standalone or
  within a Grouped Closing Protocol round — Claude presents, in order: (1) full
  research/analysis in prose (findings, reasoning, uncompressed); (2) the full **Issue —
  Choices — Recommendation & Rationale** table, complete sentences, not truncated; (3) only
  then the selectable-options prompt, which exists solely as the input mechanism for a
  decision the user is already fully informed about — its own truncation is fine *because*
  it's no longer the only place the content lives.
- **Mandatory pre-analysis.** Before building *any* such question, Claude works out real
  pros/cons of the substantive options and derives its recommendation from that — never picks
  a recommendation first and rationalizes after. Unconditional, independent of whether shown.
  **Exception:** in a Grouped Closing Protocol individual round (§3.2), the analysis option
  (below) is omitted — already shown as the reason the item reached individual treatment.
- **Research triggers — four categories, independent of decision "importance"** (not
  exhaustive; the user may flag more as real cases surface):
  - **A — checkable/time-sensitive factual claims:** a specific tool/crate/library/version,
    or a "current best practice" assertion that could be stale (e.g. naming a crate without
    checking; any specific Rust toolchain/dev-tool version — `agents/RUST_PREFERENCES.md`
    §0 — which is why no Rust version is ever hardcoded into a process document; determined
    fresh per project at Step 5).
  - **B — under-determined design parameters with no objectively correct value:** research
    grounds the choice in what comparable/competitive systems converged on (e.g. an
    outage-detection threshold — no "true" answer, but convergence is informative).
  - **C — option-space completeness:** not "is this stale" but "is Claude even aware of the
    full option set" — training-era knowledge may miss a newer/domain-specific approach a
    search would surface (e.g. serialization format, concurrency pattern, auth scheme).
  - **D — user-facing capability/feature-surface decisions:** opportunity-seeking, not
    risk-avoidance. Checks whether a best-in-class/competitive-advantage/novel-capability
    option exists that wouldn't otherwise get proposed — distinct motivation from A–C; can
    overlap with B for product-facing parameters.
  A purely conceptual tradeoff fitting none of these doesn't need a search.
- **Recommendation first, labeled.** Option #1 is Claude's recommendation, explicitly
  labeled — position never silently implies endorsement. Most of these questions exist
  *because* the answer depends on something only the user knows; the recommendation is a
  convenience, not a claim of likely correctness.
- **Standing "Defer" option.** Not now. Resolved via the Grouped Closing Protocol's
  deferred-items stage (§3.2) — never an ad hoc list, never silently dropped, re-surfaced at
  every subsequent gate per §3.13 until closed.
- **Standing "Provide analysis and recommendation" option, always listed last where
  present.** Since the three-layer rule already shows the analysis upfront, this exists for
  **re-surfacing** it (user lost track) or where the tool-capacity ladder (below) dropped
  analysis into inline framing text instead. Doesn't close the question — re-presents it
  (Defer and this option included again). Repeatable; a repeated request may mean the
  question needs reframing.
- **Tool-capacity ceiling (checked before the above): max 4 options total.** Defer + analysis
  option are 2 standing slots, leaving room for only 2 substantive choices at full capacity.
  - **3 substantive options:** drop the analysis option (keep Defer); state the analysis
    inline in the framing text instead — same information, different delivery; pre-analysis
    still mandatory.
  - **4+ substantive options:** even Defer alone would overflow. First try to consolidate to
    ≤3 genuinely distinct choices (4+ often means the question is underspecified). If
    consolidation isn't honest, fall back to plain text: analysis and recommendation inline,
    Defer noted as available by reply.
- **3+ surviving options is expected to be rare — flag it when it happens.** Real analysis
  should usually collapse apparent 3-way choices to 2; a genuine 3-way survivor (most often an
  architectural tradeoff with no dominant option) gets a brief note distinguishing it from
  Claude simply failing to narrow the question. Frequent unflagged 3+-option questions would
  itself indicate the analysis isn't rigorous.

---

### 3.4. The 9 Steps

| # | Step | Output | Mechanism |
|---|---|---|---|
| 1 | Concept Intake & Context Mapping | Architecture file `_01_introduction` (Draft) | §3.1. Mechanical: structuring the concept statement into problem-space + boundaries. Substantive (flag, don't assume): actual in/out-of-scope, real boundaries, which adjacent actors matter — the highest-cost place in the workflow for an unflagged wrong guess, since every later step inherits it; never autonomous, any mode (§3.7). **Always** attempts deep web research and competitive analysis (how comparable systems/frameworks solve the same problem), aiming at best-in-class/competitive-advantage/novel-capability framing — an honest "no meaningful competitive landscape, here's why" is an acceptable outcome; a fabricated comparison is not. **Actively solicits screen mockups, reference HTML, brand/image assets, and (if relevant) audio/video assets, as early as possible** (§3.8/§2.2) — not deferred to when UI work is imminent. |
| 2 | User Story Elicitation | Architecture file `_02_user_stories` | §3.1 + §3.2 in full, one **single session** (no A/B split). Claude drafts the full candidate batch (by actor/theme) from `_01_introduction`; user runs the Grouped Closing Protocol over it in one pass, covering both the story list and each story's Interaction Sequence together, not as separate-session stages. |
| 3 | Requirement Decomposition | Architecture file `_03_requirements` | §3.1 + §3.2. Mechanical: 3-pass decomposition (Functional → Logical → Detailed) and the 9-criteria/Requirement-Smell check (`agents/DESIGN.md` §4.5), unsupervised — internal iteration passes within the one Step 3 session, not separate sessions. Substantive: necessity, correctness, completeness — flagged per requirement, resolved via §3.2. |
| 4 | Test Identification | Architecture file `_04_test_strategy` | §3.1 + §3.2. Mechanical: deriving test cases from an already-verifiable requirement, plus the 9-criteria table and Rust Requirement Smell catalog. Substantive: what bar a DoD must clear ("verifiable" only guarantees *some* objective test exists, not which one) — flagged, resolved via §3.2. |
| 5 | Verification Feasibility | Architecture file `_05_verified_traceability` | §3.1 + §3.2. Mechanical: matching a need to a tool *category*; checking Rust dependencies directly against `agents/PREFERRED_DEPENDENCIES.md` (preferred used without confirmation; **Forbidden** never proposed; **Requires-Approval**/unlisted flagged; `tokio`+WASM never proposed together); checking dev tools/`agents/PREFERRED_TOOLS.md` and infra services/`agents/PREFERRED_SERVICES.md`; **setting the workspace MSRV** per `agents/RUST_PREFERENCES.md` §0 (and dual-MSRV if a publishable library crate exists) — recorded here as a flagged placeholder, relocated to `_07_interfaces_and_stack` §6 at Step 6. Substantive: real environment feasibility (infra, budget, licensing, team familiarity) for anything not list-checkable — flagged, resolved via §3.2. Claude never certifies "technical sufficiency" unilaterally; it's a proposal pending confirmation. |
| 6 | Final Architecture Synthesis (ISO 42010) | Architecture files `_06_viewpoints`, `_07_interfaces_and_stack`, `_08_constraints_and_roadmap` (new this step); `_05_verified_traceability` (finalized — MSRV relocated out) | Mostly recombination of already-approved content from `_01`–`_05` — less to delegate. One live judgment call: formal notation (SysML-style) vs. prose, per element, with a brief stated reason so the user can override per-element without re-litigating the viewpoint. Must cover all 4 mandatory viewpoints: Functional, Information (Data Dictionary), Deployment, Interface Control. Also where the Asset Manifest (§3.8) migrates into its permanent home, `_06`'s §4.13 (UX & UI Design). |
| 7 | Spec Audit & Phase-End QA | Final Deficiency Audit Report | **No propose-with-flagged-assumptions here — by design.** An audit's value depends on adversarial independence from the rest of the process, including Claude's own prior work; the last systematic check before anything is built on the Spec. Checks, on Claude's own analysis: every User Story maps to a requirement; every requirement is atomic; no approved content elided/summarized/replaced with a "see previous version" reference; no gap could force a stub/partial implementation downstream; every Asset Manifest entry is actually referenced where it should be; **every deliverable filename conforms to the §4 naming convention.** **Zero findings closes Step 7 normally — proceed to Step 8. Any finding at all (even one) triggers the Step 7 Backtrack Workflow (§3.12) instead of normal gate approval.** |
| 8 | Development Plan & Checklist | Development Plan + Checklist + Dev-Agent Kickoff Prompt | Before drafting environment/config content (toolchain, CI, local setup — Steps 1–7 are about *what*, not *where/how built*): elicit concrete facts directly; for anything unspecified with a reasonable default, propose the default as a flagged assumption (§3.1), once per Plan, not once per phase. **Phase Sizing:** each phase must complete within one agent session, sized for an agent **less capable than Claude**, with margin for debugging. Complexity is computed, not eyeballed: `Score = (task_count × 1) + (new_public_interfaces × 2) + (cross_file_tasks × 2) + (cross_task_dependencies × 1.5)` — interfaces/cross-file coordination weighted higher (likeliest error source for a weaker agent). Default ceiling **≤15**, user-tunable over time; the weighting is the stable part, the ceiling the adjustable part. **Frontend phase sequencing, where a UI exists:** *targeted interleaving* — each screen/component's frontend task sits in the same phase as its real (non-mock) backend/data dependency, never earlier (forces a throwaway stub, contradicting the Anti-Stub Mandate) and never deferred once its real dependency is available. **Final deliverable:** a reusable kickoff prompt (`agents/exemplars/dev_agent_prompt_template.md`), produced once, reused at the start of every Development-Phase session, not regenerated per session. |
| 9 | Plan & Checklist Audit | Plan & Checklist Audit Report | **No propose-with-flagged-assumptions here either — by design, mirroring Step 7.** Step 8's output used the same flagged-assumption mechanism as every step; nothing so far independently verified it. Step 9 runs `agents/exemplars/development_plan_template.md` §15 (Plan-Level DoD) directly as an audit checklist, on Claude's own analysis: every Core requirement traced, no orphan citations, every phase has non-empty Entry/Exit Criteria, every task has a Verification Method and DoD, the Phase Dependency Graph is acyclic, Checklist/Plan lockstep with no drift, every filename conforms to §4. Also cross-checks Build Order fidelity against the finalized Spec and, where a UI exists, that phase sequencing reflects Frontend Targeted Interleaving rather than a backend-then-frontend split. **Zero findings closes the Design Phase. Any finding classifies as (A) Plan/Checklist-only** — reopen Step 8 alone, fix, re-package, re-audit — **or (B) Spec-originating** — reopen the relevant Architecture Spec step (1–6), re-clear Step 7, then return to Step 8 — never patch the Plan around a still-defective Spec. Repeats until a Step 9 run is clean. See `agents/DESIGN.md` §5.9 for full backtrack mechanics, mirroring §3.12. |

Every step ends with **STOP, present output, await explicit approval** — never combined,
never skipped, in full-rigor mode (§3.7 for the autonomy toggle). Per §1, every step (and
every repeated run) is its own session — a step's gate output is always a complete file
package for the next session (§3.11), and always includes the §3.13 accumulated open-items
review.

---

### 3.5. Scope: a Three-Way Question, Not Binary

"Is this in scope" has a real third answer: wanted, just not in this build. Claude never
collapses that into a plain exclusion.

- **Default case — ask which is meant:** not wanted at all (Spec §2.5, Out-of-Scope) vs.
  wanted but deferred (§2.6, Future Features) — materially different downstream handling, so
  Claude doesn't guess. Once "deferred" is confirmed, record it in §2.6 per that section's
  mandate (originating ID if any, sufficient description, deferral reason, known
  dependency/precondition) — not just a conversational mention. If the user already states
  which is meant in the same message, don't ask redundantly (§3.2's early-resolution
  exception).
- **Feature-level stackable/incremental choices (e.g. MVP-floor decisions) default to Future
  Features**, stated explicitly ("the richer level goes to Future Features unless you'd rather
  drop it"), overridable in the same reply. This default is for genuine feature/product-scope
  choices only.
- **Implementation-level choices are a distinct category** — an algorithm/strategy choice for
  an already-in-scope requirement isn't a scope decision (the requirement is in scope
  regardless; what's decided is *how*). These route to Spec **§4.15** (deferred, noted only —
  no commitment) or **§4.16** (deferred, with an extension point Claude must actually specify
  in §4.4/§4.9, not just index) — Claude asks which is meant when unclear, sub-grouped by
  originating ID either way.
  - **Claude actively weighs "ship simple + commit an extension point" within the mandatory
    pre-analysis itself (§3.3)** — not only after the user has chosen — especially where an
    option carries real scope/effort impact, since the retrofit-later-vs.-cheap-extension-now
    cost is itself part of what the analysis must surface.

### 3.6. Backtracking & Major Changes

Per `agents/DESIGN.md`, the user may return to any previous step, most often when a later
step reveals an earlier step's content — flagged or not — was wrong. For normal forward
iteration (i.e. *not* a Step 7 or Step 9 finding — §3.12 for Step 7's mandatory case,
`agents/DESIGN.md` §5.9 for Step 9's analogous one), Claude: (1) names the originating step
and specific content at issue, rather than patching around it; (2) presents — doesn't
unilaterally decide — the choice between a local patch vs. reopening the earlier step, since
that determines how much upstream work needs re-approval; (3) if reopened, preserves all
already-approved later content, revisiting only as needed once the earlier step is
re-approved (additive-only, no wholesale discard); (4) flags it as a Major Change whenever it
materially changes scope/requirements/architecture — not just when convenient. **Major
Changes generally** are flagged explicitly, by name, the moment Claude recognizes one — never
buried in a diff or folded silently into the next deliverable.

---

### 3.7. Autonomy: a Per-Step Toggle, with Two Named Presets

Each step is either **interactive** (normal §3.4 behavior, full elicitation per §3.1/3.2) or
**autonomous** (internal process runs silently, no mid-step elicitation, but the **gate
itself is unchanged** — Claude still stops and presents output for approval). This is
**prompt-invoked**, never a standing default; full-rigor, all-interactive (§3.4) is the
baseline absent an explicit instruction.

**Which steps can ever be autonomous:** 2, 3, 4+5, 6, 8. **Steps 1, 7, and 9 can never be
autonomous, in any mode** — Step 1 because it's the highest-cost place for an unflagged wrong
guess (§3.4); Step 7 and Step 9 because audit independence is non-negotiable (§3.4). No
prompt overrides this. ("4+5" = Steps 4 and 5 handled as one combined autonomous session when
this toggle is invoked — the only case two Steps share a session; in interactive/default mode
they remain separate sessions per §1/§3.4.)

**Autonomous gate output, every eligible step:** not just a result — a consolidated
**"Assumption/Issue — Choices — Decision & Rationale"** note per substantive item resolved
internally, alongside the step's normal output, plus the §3.13 open-items review like any
other gate. Richer than a flat assumptions list: the user sees what the live options were and
why Claude picked one — the same content a selectable-options question would have shown
interactively, presented as a statement rather than a question.

**Two named presets** (the only real difference between "small project" and "trust this
step's judgment" is whether Step 2 happens at all):

- **Tailored Mode** ("treat this as a small project, tailored mode"): Step 2 **skipped**
  entirely (not autonomous — gone; Step 1 absorbs what it would have surfaced); Steps 3, 4+5,
  6, 8 autonomous by default; Step 1 unconditionally high-rigor per §3.4 *and* additionally
  expanded to cover persona/interaction/edge-case content Step 2 would otherwise surface.
- **Full Autonomous Mode** ("run this project in full autonomous mode"): identical to
  Tailored Mode except Step 2 **runs**, autonomously, instead of being skipped — for a
  full-size project where Step 2's actual artifact is still wanted, just without interactive
  elicitation.

**Single-step invocation** (independent of either preset): "run Step [N] in autonomous mode"
— toggles one eligible step (2, 3, 4+5, 6, or 8) without invoking a whole preset. No combined
shorthand exists for "preset + extra step" — name the preset and the additional step
explicitly if stacking (little left to stack once a preset is active, since both presets
already make every eligible step but 2-in-Tailored autonomous by default).

Every gate in §3.4 still applies regardless of preset/toggle — autonomy changes only the
*visibility of process*, never removes a stop-and-approve point.

### 3.8. Visual & Media Asset Input: Discovery Source and Tracked Artifact

The user may provide **HTML, images, and (where relevant) audio/video** showing or
constituting how the system's UI should look, sound, and behave. This plays two roles that
must both be honored: a **discovery source** (revealing data-model concepts, missing
entities, or requirements no verbal description surfaced), and a **tracked artifact** carried
by filename through Design and into Development, landing at fixed repository paths for the
development agent to consume directly.

**Authoritativeness differs by asset type — deliberate, non-uniform:**
- **HTML is presumptively authoritative for structure and behavior.** Per the user's own
  statement that HTML input is "highly representative of the desired final appearance,"
  Claude treats a provided HTML file as the default source of truth for a screen/component's
  markup structure, layout, conditional visibility, validation behavior, and exact copy
  text — not merely informative styling guidance to reinterpret freely. Deviating from what an
  HTML asset specifies is itself a flagged item (§3.1), same as any other departure from
  stated input.
- **Images are informative, not structurally binding**, treated as a **design-token source**:
  branding, iconography, typography, color palette, pattern/texture tiles, plus any other
  graphical info they carry. Feeds the Spec's Design System content (§4.13) rather than
  dictating layout structure the way HTML does.
- **Audio/video assets** are logged in the Asset Manifest (below) and their intended usage is
  captured in requirements/UX content as normal, but Claude does not "consume" their content
  directly the way it reads HTML/images — tracked for the development agent, not analyzed by
  Claude for discovery.

**Solicited as early as possible, not just at Step 1.** Step 1 intake actively asks whether
mockups, reference HTML, brand/image assets, or audio/video exist (§3.4's Step 1 row), but
the user is encouraged to provide assets whenever they become available — later steps benefit
from having them sooner. **Delivery channel:** same as any other Advisory Mode input (§1) —
attached to a message or made available via the project space — distinct from, and preceding,
the fixed repository paths below, which govern where the *same* files land for the
development agent once Development Phase begins.

**The Asset Manifest — mandatory tracking record.** The moment any asset is provided, at any
step, Claude logs it: a table with columns `Filename | Type (html / image / audio / video) |
Repository Target Path | What It's Authoritative/Informative For | Provided At (Step)`.
Filenames are **immutable once logged** — Claude never proposes a rename; a collision or
ambiguity is a flagged item for user resolution, never silently disambiguated by Claude
picking a new name. Repository target paths are fixed: `assets/html/`, `assets/images/`,
`assets/audio/`, `assets/video/` — the same paths/filenames the user will populate in the
repo for the development agent, so any asset referenced anywhere in the Spec or Plan resolves
unambiguously with zero translation between Design and Development. Until Step 6 relocates it
to its permanent home in `_06_viewpoints` §4.13, the Manifest accumulates inside whichever
file is current at the step the asset was provided (§4's Step-aligned mapping) — the same
flagged-placeholder-then-relocate pattern used for the Step 5 MSRV decision.

**Triggers a completeness check whenever provided, at any point** — not only at Step 1.
Claude checks already-approved Data Dictionary/requirements content against what the visual
input implies. Any gap is a **flagged open item for user confirmation** (§3.1), routed through
the Grouped Closing Protocol (§3.2) like any other flagged batch item — never silently
patched into already-approved content, however obvious the gap seems.

**Both image and HTML are reviewed when both are provided for the same screen/component —
neither substitutes for the other:** HTML exposes behavior/state/content an image can't show;
an image shows visual layout/design tokens raw markup doesn't make obvious. Each is checked
for what only it reveals.

**Frontend framework/crate implications.** Where HTML input implies a specific Rust/WASM UI
framework capability, that choice still goes through Step 5's normal
`agents/PREFERRED_DEPENDENCIES.md`/`agents/PREFERRED_TOOLS.md` feasibility check like any
other dependency — visual input motivates the need but does not bypass approval.

**Assets are never carried forward as files in a handoff note (§3.11) — a deliberate
exception to §3.11's general "anything crossing a session boundary is a file" rule.** During
Design, a provided asset is reasonably authoritative only for informing UI vision as it bears
on whichever step is currently using it — not a durable input the next session needs
re-attached. By Development, the actual asset will likely be refined/modified somewhat from
whatever was shown during Design, so re-bundling the Design-time copy at every handoff would
just propagate a stale version. Concretely:
- A step's own session may request the asset directly from the user (per the delivery channel
  above) if that step's work is affected by it — a live input to that session, not something
  inherited from a prior handoff.
- The handoff note and Architecture Specification content **reference** the asset (filename,
  Asset Manifest entry, what it's authoritative/informative for) — never re-attach or
  re-package the asset file itself.
- The Architecture Specification and Development Plan state, wherever a UI element depends on
  an asset, that the (possibly-updated) asset **will be available in the repository's
  `assets/` directories at Development Phase** (fixed paths above) — the actual file is a
  Development-Phase input, not a Design-Phase handoff artifact.
- The Asset Manifest itself (the tracking table) *is* carried forward normally, as ordinary
  Architecture Specification content (§4.1) — only the underlying asset files are excluded
  from handoff packaging.

### 3.9. No Agent-Side Certification of Technical Sufficiency

Claude cannot certify technical sufficiency unilaterally at any step (§3.4 Step 5 states this
explicitly; it holds everywhere). Research (§3.3, §3.4 Step 1) narrows the substantive gap
between Claude's proposal and the right answer — it never closes it, since real
infrastructure, budget, and team constraints remain unverifiable from inside this chat. More
rigor produces a better-informed proposal, never a certified one.

### 3.10. Requesting More Depth ("Not Enough — Another Pass")

After any Steps 1, 2, 3, 4, 5, 6, or 8 gate (not 7 or 9 — see §3.4, §3.12, `agents/DESIGN.md`
§5.9; both audit steps use their own backtrack workflow instead), the user may request a
deeper pass rather than a correction — e.g. "not detailed enough, another pass," "double the
number of stories," "more detail in the interface specification." Distinct from the Grouped
Closing Protocol (§3.2), which handles *correctness*; this handles *insufficiency*. The user
may scope narrowly (a viewpoint, a phase) or broadly (the whole step), and may give a concrete
metric — Claude treats a given metric as the actual target, not a vague cue to add a little
more. Each repeated run is its own new session (§1), named `pass2`, `pass3`, etc. in its
handoff-note filename (§4) — the re-touched Architecture file keeps its own independent
version history, simply bumped again at that session's end (§4's versioning rule).

### 3.11. File Delivery for Anything Crossing a Session Boundary

Per §1's session-per-Step reality: anything a *later* session needs to resume, review, or
approve must be an actual downloadable file — never left as chat text to copy manually.
Covers, at minimum: the handoff note itself (below); the Architecture Specification file(s)
current as of session end — per §4, each Step owns and directly edits one numbered file, so
"deliver the Step's output" and "deliver the current file version" are the same action. In
short — **anything presented at a Step Approval Gate is a file.** Content that only matters
within the single session that produced it (e.g. back-and-forth resolving a Grouped Closing
Protocol stage before it's finalized) may stay as chat text — the next session only needs the
*settled* result. **Exception: user-supplied UI/media assets (HTML/image/audio/video) are
never re-packaged into a handoff, per §3.8 — only referenced by filename.**

**Handoff notes are themselves a separate `.md` file** — never inline chat text — named:

```
[projectname]_design_handoff_stepN_passX.md
```

`N` = step number, `X` = pass identifier: `1` for a step's first run, incrementing
(`pass2`, `pass3`, ...) for a repeated run (user-requested per §3.10, or automatic, e.g. a
Step 7 re-audit per §3.12). Step 2, now a single session (§1/§3.4), uses this same scheme —
there is no `passA`/`passB` variant anywhere. Examples:
`myproject_design_handoff_step1_pass1.md`, `myproject_design_handoff_step7_pass2.md` (a
second Step 7 audit after a backtrack remediation).

A handoff note is load-bearing, not a courtesy summary: since the next session has no memory
of the current one, starting it requires *actually providing* the handoff note file plus the
current version of whatever Architecture file(s) it hands off — announcing "continue from
Step 3" without attaching `_03_requirements` at its current version and the handoff note
gives the new session nothing to resume from.

**Every handoff note MUST open with an explicit, unambiguous Next Action directive.** A
session that receives files but no imperative instruction has, in practice, reported "no
explicit ask" and offered the obvious next step as a question instead of starting it — an
observed failure that wastes a round-trip. The directive is the note's first line, exact form:

```
## Next Action
Start Step N[, Pass X].
```

(e.g. `Start Step 4.`, `Start Step 7, pass 2.`) — a command to execute immediately, not a
suggestion; the receiving session opens with the Step's own work (§3.4's row for that Step),
not "what would you like me to do?" A note without this line is incomplete, corrected before
handoff, same as a missing file-list entry.

**The handoff note MUST explicitly enumerate every file the next session needs** — a complete
list, not just this session's own output. Includes earlier-session files still relevant — e.g.
Step 4's note names `_01_introduction`, `_02_user_stories`, `_03_requirements` (current
versions) alongside Step 4's own `_04_test_strategy`, since Step 4 reads all of them — so the
user knows what to gather without reconstructing the dependency chain. A note that only names
this session's own output, silently omitting earlier-session files still needed, doesn't
satisfy this. **This same file list is also stated directly in the chat response**, same turn
as the note — not only inside the file — so the user can see what to gather without opening it.

**Pre-handoff staleness check, mandatory, immediately before producing the handoff note.**
Claude enumerates every Architecture file touched this session and compares each against what
was last actually shown/downloaded. Any file changed since — including ones edited early and
not revisited — is re-presented in the same handoff batch. **This explicitly includes files
belonging to a step other than the current one** — a backtrack session (§3.12) or any session
revisiting an earlier step's file (§3.6) touches files outside its "home" step, and those
files are exactly as subject to this check, and the version-bump rule below, as the current
step's own. This has been observed failing specifically on second/third backtrack passes and
on sessions touching a previous step's file — the actual test is "did I touch a file this
session, regardless of whose step," never "did I touch this step's own file." Never a partial
subset left for the user to ask "and the rest?" — every touched file, every time. **This same
check also verifies every file produced or touched this session conforms to the §4 naming
convention** — a misnamed file is corrected before handoff, not flagged for later. Step 7's
audit (§3.4, §3.12) re-checks this project-wide, independently, as part of its own review.

**Version-bump enforcement, mandatory, as the literal last action before handoff.** Per
§4.3's rule (bump exactly once, at session end, per file actually touched): immediately after
the staleness check above, Claude bumps each touched file's version by exactly one before
presenting it — no exceptions for a file touched only briefly, only on a backtrack pass, or
only for a small cascading fix. A file listed as touched but handed off at an unchanged
version is a defect in the handoff itself — precisely the observed failure mode this final
check exists to catch before the file reaches the user.

**The next session verifies its inputs against the handoff note's manifest before
proceeding.** Where a handoff note exists (any session resuming from a prior one — not a
fresh Step 1), Claude checks files actually provided against the note's own enumerated list —
a concrete comparison, not independent judgment of what the step "should" need. If something
named is missing, Claude says so and asks for it directly, rather than proceeding incomplete
or silently assuming. Mirrors `agents/exemplars/development_plan_template.md` §11's Session
Handoff Protocol, applied to Design Phase sessions too.

---

### 3.12. The Step 7 Backtrack Workflow

Step 7 (Spec Audit) is, by design (§3.4), a pure **finder**, never a fixer — no
propose-with-flagged-assumptions, no Grouped Closing Protocol, never patches content itself.
This is what makes its independence meaningful. Consequence: **a Step 7 finding is never
resolved via the ordinary local-patch-vs-reopen choice in §3.6/`AGENTS.md` §2.1** — every
finding requires reopening the step that should have produced or caught it; there is no
"local patch at Step 7" to choose between, since Step 7 itself never touches content.

**Zero findings:** the gate behaves normally — present the clean report, await approval,
proceed to Step 8.

**Any finding at all (even one):** the normal Step 7 gate does not close as an approval gate.
Instead:

1. **Map every finding to its originating step.** For each finding, identify which step
   (1–6) should have produced/caught the content — and, per §4, which numbered Architecture
   file that step owns.
2. **Determine the single earliest originating step across all findings.** If findings trace
   to multiple steps, do **not** reopen each separately or reopen the same step multiple
   times — start at the earliest.
3. **Open one new "backtrack" session** (per §1; Step 7's handoff note for this session makes
   clear what kind of session it is and what it must accomplish). Working forward from the
   earliest originating step through to Step 7 again:
   - Fix every finding tracing to the current step, editing that step's own owned file
     directly.
   - **Re-check already-approved later-step content for cascading effects of the fix**, per
     §3.6's "revisit only as needed" — discovered by re-checking as each fix is made, not
     pre-identified by Step 7 (which only reports what it found, not downstream consequences
     of a fix that hasn't happened yet).
   - Move to the next step in sequence (finding or not), applying any cascading fix plus any
     finding tracing directly to it.
   - Continue through Step 6, so every originating file and every downstream file is
     reconciled by session end — both original findings and cascading effects.
   - **Do not re-run Step 7 itself within this session.** The job is to get the Spec ready
     for a fresh, independent audit — not to self-certify success.
4. **At session end, package every file exactly as at any other Step Approval Gate** (§1,
   §3.11): every touched Architecture file, version bumped exactly once per §4 (never once per
   finding fixed — one session, one bump, per file actually touched); a handoff note for the
   next session.
5. **That handoff note MUST state plainly the next session is a new Step 7 audit** (pass2,
   pass3, ... per §3.11), not a continuation, and MUST summarize: which findings were fixed,
   which steps/files were reopened and in what order, and **explicitly that cascading effects
   were discovered and resolved by re-checking later-step files during the backtrack session
   itself** (not pre-identified by the original report) — so the new Step 7 session
   understands the full scope of what changed and why.
6. **The new Step 7 session re-audits the entire Spec from scratch**, same adversarial
   independence as any Step 7 run (§3.4) — it does not take the backtrack session's own
   account as given; it re-derives findings independently. Clean → proceed to Step 8
   normally. Any finding (even something new) → this workflow (1–6) repeats.

**Major Change Notification still applies** (§3.6 point 4): a backtrack fix that materially
changes scope/requirements/architecture is flagged the same as any other backtracking.

---

### 3.13. End-of-Step Open-Items Review

**Every Step Approval Gate (§3.4), without exception, includes a standing review of every
unresolved item accumulated so far — not only items from the current step.** Distinct from,
and additional to, the Grouped Closing Protocol's own per-batch Deferred/Remaining-Ambiguous
stages (§3.2), which close out *that step's own* batch before the gate; this reviews the
**running total across every step so far**, in an explicit attempt to shrink the open list
before it compounds.

**Mechanism, every gate:**
1. Claude maintains a running **Open Items Register** — every Deferred item (§3.2 stage 3),
   every Remaining-Ambiguous item (stage 4) not since resolved, and any other
   explicitly-flagged-but-unresolved assumption from any prior step, each tagged with its
   originating step and where it's already been re-surfaced.
2. At every gate, before the step's own headline output, Claude presents the current Register
   in full — not just items newly added this step — organized by originating step, oldest
   first.
3. For each open item, Claude states plainly whether anything learned since changes the
   analysis (a later step may have implicitly answered it, or narrowed the live options) —
   an active attempt to close items, not a passive restatement.
4. The user resolves in bulk or per-item, using §3.2's bulk-reply/per-item mechanics
   directly. An item the user chooses to leave open stays on the Register and is carried
   forward again next gate — never dropped just because it was re-presented and not addressed.
5. The Register itself is a required §3.11 handoff file, not chat-text-only, since a later
   session has no memory of what's still open otherwise.

**This does not relax §3.2's own per-step closure stages** — a step's own batch still goes
through its full Grouped Closing Protocol before its gate. §3.13 is the additional cumulative
check on top, aimed at preventing an item deferred at, say, Step 2 from silently riding
unresolved all the way to Step 7's audit without ever being re-offered resolution.

---

## 4. Multi-File Output & Cross-Linking

Applies once a Spec or Plan deliverable spans multiple files. All files for one artifact live
in the same flat directory (no nesting, unless the user changes this). Names below are **base
names** — Architecture Spec and Plan files (not the Checklist) carry a version suffix per
§4.3. `[projectname]` is the actual project's filesystem-safe slug (underscores, not hyphens,
`AGENTS.md` §2.5), confirmed with the user at Step 1 if not already established.

**Naming convention — `[projectname]` first, then artifact type, then a zero-padded two-digit
sequence starting at `01`, then topic, then version suffix.** Projectname-first keeps every
file belonging to one project's Spec and Plan sorting together, with architecture/dev-plan/
checklist/prompt files interleaving sensibly in an alphabetically sorted listing, rather than
scattering by generic numeric prefix alone.

### 4.1. Architecture Specification — 8 Files, One Per Design Step

**Split into 8 files, one per originating Design Step (1–6; Steps 4 and 5 each still get
their own file even though §3.7's autonomous "4+5" combination can run them in one session —
file ownership tracks the template's content boundary, not whether a run combined sessions).**
This supersedes any coarser topic-based grouping: smaller projects could once fold multiple
Steps' output into a shared handful of files, but at this template's target scale (50+ User
Stories, 100–200+ requirements are common), a coarser grouping produces multi-thousand-line
files and forces every Step's session to bump a shared file's version even when it touched
none of the file's pre-existing content. One-file-per-Step avoids both: each file's size stays
bounded by what one Step actually produces, and its version history reflects only the
sessions that actually changed it.

Pattern: `[projectname]_architecture_NN_<topic>_v[N].md`, `NN`/`<topic>` fixed per the table,
`[N]` that file's own independent version counter (§4.3):

| File (`NN_topic`) | Template content | Owning Step | Created at |
|---|---|---|---|
| `01_introduction` | §1 Introduction | Step 1 | Step 1 |
| `02_user_stories` | §2.1 Personas, §2.2 User Stories, §2.5 Out-of-Scope, §2.6 Future Features | Step 2 | Step 2 |
| `03_requirements` | §2.3 Core Functional Requirements, §2.4 Non-Functional/Quality Attribute Scenarios | Step 3 | Step 3 |
| `04_test_strategy` | §3.1 9 Criteria (self-check ref), §3.2 Test Case Catalog, §3.3 Rust Requirement Smells | Step 4 | Step 4 |
| `05_verified_traceability` | §3.4 Traceability Matrix, §3.5 Acceptance Criteria Detail; temporarily holds the Step 5 MSRV decision and any pre-Step-6 Asset Manifest entries, both flagged as placeholders pending relocation | Step 5 | Step 5 |
| `06_viewpoints` | §4, all four ISO 42010 viewpoints (Functional, Information, Deployment, Interface Control) plus §4.5–4.16, including §4.13's Asset Manifest (relocated here from `05`) | Step 6 | Step 6 |
| `07_interfaces_and_stack` | §5 External Interfaces & Integrations, §6 Technology Stack & Dependencies (including finalized MSRV, relocated from `05`) | Step 6 | Step 6 |
| `08_constraints_and_roadmap` | §7 Constraints & Assumptions, §8 Risks & Technical Debt, §9 Implementation Roadmap & Build Order, §10 Public API & Framework Consumer Contract, §11 Appendices | Step 6 | Step 6 |

If `06_viewpoints` alone overruns a reasonable size once real content volume is known
(likely, per the template's own note that §4 is its largest section), Claude flags this at
Step 6 and proposes a further split (e.g. `06a_functional_view`, `06b_information_view`, ...)
rather than silently overrunning — the same scale-driven-split principle applies recursively.

**Step 7's Audit Report and every handoff note remain outside this numbered set**, using the
handoff-note convention (§3.11) instead — process/review artifacts, never assigned an
`NN_topic` slot.

**Do not conflate the two `_01_...` files.**
`[projectname]_architecture_01_introduction_v1.md` (this section) and
`[projectname]_dev_plan_01_overview_v1.md` (§4.2) are independent artifacts with
independently-chosen naming schemes that happen to both start `_01_` and share the
version-suffix format — neither is stale/superseded relative to the other, and the Plan's
continued use of a topic word (`overview`) isn't evidence the Spec still uses topic-based
names too. A name like `[projectname]_architecture_01_overview_v[N].md` (combining a Spec
`NN` slot with a Plan-style topic word) is invalid under this scheme — almost certainly a
pre-restructure leftover, a naming error, or a hallucination, not evidence the 8-file table
has changed. Verify against this table directly before accepting a cited filename at face
value.

### 4.2. Development Plan (15 sections → 4 files), Development Checklist, Kickoff Prompt

Unchanged from prior structure — the Plan's coarser grouping is intentional and distinct from
the Spec's Step-aligned scheme (see §2's note on why the two split at different
granularities). Pattern `[projectname]_dev_plan_NN_<topic>_v[N].md`:
- `..._01_overview_v[N].md` — §0 Architecture Cross-Reference, §1 Introduction, §2 Technology
  Stack, §3 Project Folder Structure
- `..._02_environment_and_phases_v[N].md` — §4 Environment Setup, §5 Dev/Test Configuration,
  §6 Phases and Milestones (including the frontend targeted-interleaving sequencing
  principle from §3.4's Step 8 row, elaborated per §9.1)
- `..._03_tasks_and_testing_v[N].md` — §7 Risk Management, §8 Task Decomposition, §9 Test
  Strategy, §10 Logging Strategy
- `..._04_protocols_and_dod_v[N].md` — §11 Session Handoff, §12 Abort/Rollback, §13
  Escalation Triggers, §14 Change Control, §15 Plan-Level Definition of Done

**Development Checklist** (single file unless content genuinely warrants a split, §2):
`[projectname]_dev_checklist.md`, base name, no numeric prefix — conventionally one file
covering the whole Plan, not a set member. **No version suffix**, same in-place-editing
reason as the Kickoff Prompt. Per `AGENTS.md` §2.7, edited in place — never copied, renamed,
or otherwise reproduced.

**Development-Agent Kickoff Prompt:** `[projectname]_dev_agent_prompt.md`, base name, no numeric
prefix, no version suffix, produced once at Step 8's end, reused verbatim every
Development-Phase session.

**Phase Summaries** (Development Phase, per `agents/exemplars/development_plan_template.md`
§11.3): `[projectname]_phaseN_summary.md` — carries the `[projectname]` prefix for
consistency, even though it's a Development-, not Design-Phase, artifact.

This grouping is a starting proposal for the Plan (the Spec's 8-file grouping is fixed, tied
directly to Step ownership) — Claude revisits the Plan's grouping once real content volume is
known at Step 8, flagging a different grouping if warranted, rather than forcing content into
a pre-decided shape.

**Linking:** the first file in each numbered set is an index (overview + linked TOC, relative
links, co-located assumed) — for the Spec, `01_introduction`. Every other file links back to
the index and forward/sideways wherever there's a substantive content dependency — based on
actual dependency, not sequence, using heading-derived anchors for a specific section, not
just a file's top. (E.g. `06_viewpoints`'s ICD referencing a Data Dictionary type earlier in
the same file is an in-file anchor; `08_constraints_and_roadmap`'s Roadmap referencing a
requirement in `03_requirements` is cross-file.) The Spec and Plan file sets are independent —
no cross-linking absent a concrete reason (e.g. a Plan phase implementing a specific ICD
interface).

**Intermediate/handoff artifacts** (handoff notes, the Open Items Register per §3.13,
working notes, draft batches, anything mid-step rather than settled Architecture content) use
the §3.11 handoff-note convention: `[projectname]_design_handoff_stepN_passX.md` — distinct
from the numbered Architecture/Plan files, which are organized by content section (and, for
the Spec, by owning Step), not by which session produced a given revision.

### 4.3. Versioning: Per-File, Bumped Once at Session End

**Version format is a plain integer counter — `v1`, `v2`, `v3`, ... — nothing else.** Fixed,
not a preference: no decimals, no semver triplets (`v1.0.0`, `v1.0.01`), no zero-padding
(`v01`). `[N]` increments by exactly `1` per bump, starting at `v1` at creation. Stated
explicitly because different sessions have independently adopted different schemes — every
file, in every project, uses plain-integer versioning only. A file found using another format
(e.g. `v1.0.01`) is corrected to plain-integer at its *next* bump (continuing from the closest
sensible integer, flagged as a one-time correction) rather than perpetuated.

**Every Architecture Spec and Dev Plan file carries its own independent `_v[N]` suffix,
starting at `v1` at creation.** Two rules govern `[N]`, both material:

- **Independent per file, not shared across a set.** No single project-wide or artifact-wide
  version. `01_introduction` may be at `v4` (revisited across backtrack sessions) while
  `03_requirements` is still `v1` (untouched since Step 3) — normal, not an inconsistency.
  Any example filename elsewhere implying a single shared `[N]` across files (e.g. a
  kickoff-prompt file list at one placeholder version) is illustrative shorthand only — a
  file's real version is always read off its own actual name, never inferred from a sibling.
- **Bumped exactly once, at the end of the session that touched it — never mid-session, never
  more than once per session** regardless of edit/fix count. Within a session, a file may be
  edited many times (Grouped Closing Protocol rounds, corrections, flagged-item resolution) —
  all under the *same* version number, since the bump exists to prevent download/upload
  collisions *across* sessions, not to track every internal edit. Only at session end, when
  packaged for handoff (§3.11), does the version increase by exactly one. An untouched file
  keeps its version, not redelivered just because siblings in the same batch changed.
  **"Touched" means any file whose content changed this session, full stop — not limited to
  the current step's own file.** A backtrack session (§3.12) or ordinary §3.6 revisit editing
  an earlier step's file bumps it exactly once too, at the same session-end point, via the
  §3.11 enforcement check — stated explicitly because this is the specific case observed
  slipping through: a session correctly bumping its own file while leaving a revisited
  earlier-step file's version unchanged.

This fixes a real, repeated download/upload collision that motivated file versioning in the
first place: the same base filename re-downloaded across sessions under an identical name.
Bumping strictly at session-end, per file actually changed, solves the collision without
punishing every file in a set for one file's edits, and without turning mid-session iteration
into untrackable version churn.

**This has a real, accepted cost: cross-links break on every version bump, and rewriting them
is mandatory.** Per §4's cross-linking convention, files in a set link to each other by name —
a version bump means every *other* linking file now points at a stale name. Whenever Claude
re-presents a versioned file with new content, it MUST, in the same pass: (1) update that
file's own internal links if their targets also changed version, and (2) check every other
file in the current set for links to this file's *old* version name, updating them. This
recurs every bump; skipping it silently reintroduces broken cross-links the same way skipping
filename versioning reintroduced download collisions.

---

## 5. Other Standing Mandates (from `AGENTS.md`)

Pulled forward as directly applicable to document production/revision here: **additive-only
by default** (no removing/"cleaning up" approved content without explicit deletion approval);
**no self-referential elision** (never "see Section X of the previous version" — every
document stays self-contained); **no stubs** (sections written to full depth when presented,
not sketched and deferred); **formal approval protocol** (only an unambiguous
"approved"/"proceed" is the green light; new instructions + approval = partial approval,
incorporated then re-presented); **`CHAT:` prefix** = pure question, no document changes, no
gate progression, no side effects of any kind (`AGENTS.md` §3.2.1); **no assumptions on
ambiguity** — ask rather than guess and build on the guess; **no reproducing shared working
documents** — the Development Checklist and Kickoff Prompt are edited/reused in place, never
copied or renamed (`AGENTS.md` §2.7).

---

## 6. Out of Scope for This File

Development Phase execution behavior (`agents/DEVELOPMENT.md`); `agents/SCRIPT_RULES.md` (no
script execution in Advisory Mode). The concrete task-level mechanics of frontend targeted
interleaving (§3.4's Step 8 row states only the principle Design Phase applies when sizing
phases — the full mechanism belongs in
`agents/exemplars/development_plan_template.md` §6/§9.1 and `agents/DEVELOPMENT.md`).
`AGENTS.md` §2.5's naming convention (underscores, not hyphens) isn't a standalone concern for
these Markdown documents, but is followed anyway for any Rust identifiers/crate names Claude
introduces in examples, for consistency.

---

## Appendix
See `CHANGELOG.md` for this file's full version history. This revision folds Step 2's Pass
A/B split into a single session, replaces the Architecture Specification's prior file
grouping with an 8-file, one-per-Design-Step structure, clarifies per-file independent
session-end-only version bumping, adds the Visual & Media Asset Input mechanism (§2.2/§3.8)
with its Asset Manifest, references the Development Plan's new frontend targeted-interleaving
sequencing principle, and adds the End-of-Step Open-Items Review (§3.13). All of the above
should be recorded as a new dated entry in `CHANGELOG.md`'s `CLAUDE.md` section and flagged
to the user as a Major Change per §3.6, since it materially changes document structure and
session mechanics.
