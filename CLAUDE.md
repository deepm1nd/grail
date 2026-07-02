# CLAUDE.md — Design Phase Working Arrangement

See `CHANGELOG.md` for full version history. This file is binding agent instruction;
`CHANGELOG.md` is for human reference only and agents do not need to read it.

Governs how Claude operates on the **Design Phase** of this repository, per `AGENTS.md` and
`agents/DESIGN.md`. Scope: producing the Architecture Specification, Development Plan, and
Development Checklist only — not Development Phase execution.
Where this file conflicts with a future revision of `AGENTS.md`/`agents/DESIGN.md`, those
win; this is a working-arrangement summary, not a replacement.

---

## 1. Operating Mode: Advisory

Per `AGENTS.md` §1.5.2, not Autonomous. Concretely:

- No persistent repo access. Claude can't read repo state, commit, or verify prior files
  exist unless pasted, uploaded, or linked in-conversation.
- Claude produces deliverables as downloadable files; **the user commits them.** Anywhere
  `AGENTS.md`/`agents/DESIGN.md` says "the agent must commit" or "ensure repository state,"
  read: Claude prepares, the user commits. `agents/DESIGN.md` §6 Phase Completion is
  satisfied once the user has actually committed — Claude can't verify this and won't claim
  it.
- Session-Initialization steps assuming shell access (`scripts/start_services.sh`, etc.)
  don't apply; Claude won't simulate running them.
- If a mandate assumes access Claude doesn't have here, Claude says so rather than silently
  skipping or improvising.
- **Every Step, and every repeated run of a Step (whether user-requested per §3.10 or
  automatically triggered, e.g. a Step 7 re-audit per §3.12), runs in its own, separate chat
  session. This is a hard rule, not a default or a convenience.** Step 2 (User Story
  Elicitation) is a single session, not split into named passes — see §3.4's Step 2 row.
  Nothing "earlier in this conversation" can be referred back to once a new session starts;
  whatever the new session needs from a prior Step must be re-provided to it directly
  (uploaded, pasted, or linked), the same way any other file dependency works in Advisory
  Mode. A direct consequence: **every Step Approval Gate (§3.4) is therefore not just a
  review checkpoint but a session-boundary packaging point** — the act of presenting a
  step's output for approval always means packaging every file and handoff note the next
  session could need, never just the step's headline deliverable. See §3.11 for what this
  requires Claude to produce as files, not chat text, as a consequence, §3.12 for how this
  applies specifically to Step 7 re-runs (Step 9's own backtrack mechanics, per
  `agents/DESIGN.md` §5.9, mirror §3.12's pattern), and §3.13 for the open-items review every
  gate must also carry forward.

---

## 2. Scope, Templates, and Project-Specific Constraints

**Artifacts (3 total, per user direction — never a 4th):** Architecture Specification spans
**8 files, one per originating Design Step** (Steps 1–6; see §4 for the full mapping and
rationale — this supersedes any earlier 3–5-file grouping). Development Plan spans 3–5
files, ~1000+ lines each. Development Checklist is one file unless content genuinely
warrants a split (Claude flags and proposes before splitting). This sizing is a user scoping
choice, not an `agents/DESIGN.md` requirement — it determines packaging, not how many gates
exist. The Architecture Specification's finer 8-way split (versus the Plan's coarser 3–5) is
a deliberate, content-driven difference, not an inconsistency: each Architecture Spec file
tracks exactly one Design Step's output through that Step's own session-versioning history
(§4), which only works cleanly at one-file-per-Step granularity; the Development Plan has no
equivalent per-step authorship history to preserve.

**Templates (all four provided, binding):**
`agents/exemplars/architecture_specification_template.md`,
`agents/exemplars/development_plan_template.md`,
`agents/exemplars/development_checklist_template.md`,
`agents/exemplars/dev_agent_kickoff_prompt_template.md` (complex/multi-phase Rust variant).
Plan and Checklist stay in lockstep (one checklist line per task DoD item). If a template is
later revised, Claude conforms existing documents to it and flags the restructuring as a
Major Change (§3.6) — never silently.

**Stable IDs, assigned at first draft, never renumbered** (`US-`, `REQ-`, `TEST-`,
`THREAT-`, `ADR-`, task IDs per template schemas). A rejected/merged item's ID is retired,
not reassigned. This is what keeps cross-document traceability links from breaking.

**No separate Requirements & Traceability document.** Confirmed by the user: every such
reference (template or conversation) resolves to the Architecture Spec's Traceability
content (§3 of the template — see §4 for which physical file currently holds it). Templates
have been corrected accordingly.

**File-split boundaries follow each template's own top-level sections** (templates state
this themselves), grouped toward whatever file-count target is in force for that artifact —
see §4 for the Architecture Specification's concrete grouping and naming convention, which
is now Step-aligned rather than topic-grouped.

### 2.1. Rust/WASM-Specific Constraints

Full detail lives in `agents/RUST_PREFERENCES.md` (naming/keywords, type-system/ownership
mapping, WASM/Web Worker concurrency, edition and MSRV policy), `agents/PREFERRED_DEPENDENCIES.md`
(Rust library dependency policy, checked directly in Step 5 below), `agents/PREFERRED_TOOLS.md`
(development tool policy, including the MSRV-determination mechanics referenced from
`agents/RUST_PREFERENCES.md` §0), and `agents/PREFERRED_SERVICES.md` (infrastructure
service policy — Docker-based deployment, `deploy/` directory conventions). This project is
a Rust web framework: a design goal is **multithreading wherever possible, native and
WASM**, via Web Workers — not a constraint to design around. Where requirement language uses
null/inheritance/exception framing that doesn't map onto `Option<T>`/traits/`Result`, that's
a Requirement Smell (§3, Step 3) needing translation, not pass-through.
`agents/PREFERRED_TOOLS.md` is relevant to this engagement (development tool/MSRV policy);
`agents/SCRIPT_RULES.md` is not relevant to this engagement (Advisory Mode runs no scripts).

### 2.2. Visual & Media Asset Input

Where a project has a human-facing UI component, the user supplies reference **assets** —
HTML, images, and optionally audio/video — that are tracked as first-class, filename-stable
artifacts across both the Design Phase (this file's scope) and the later Development Phase,
not consumed once and discarded. Full mechanism: §3.8. Physical tracking record: the
**Asset Manifest**, a table inside the Architecture Specification (§3.8 states which file
currently owns it under the §4 Step-aligned split).

---

## 3. Core Mechanisms and the Gated Workflow

`agents/DESIGN.md` §5 defines a **9-step gated lifecycle** — no step-skipping, no
combining, no proceeding past a gate without explicit approval (full-rigor, all-interactive
baseline; see §3.7 for the user-invocable autonomy toggle and its two presets). §§3.1–3.3
define mechanisms shared across multiple steps; §3.4 lists the steps themselves, each
stating only what's specific to it.

### 3.1. Propose-with-Flagged-Assumptions

The principle behind Steps 1 and 3–5: within any step, some work is **mechanical** (checkable
against an explicit, objective standard — e.g. the 9 requirement-quality criteria, deriving
a test case from an already-verifiable requirement, matching a tool to a need category) and
some is **substantive** (depends on facts about the user's actual needs/environment that
Claude cannot verify — necessity, correctness, real feasibility). Claude does mechanical work
unsupervised; for substantive work, Claude drafts its best answer but **explicitly flags every
place it had to assume or infer rather than derive from something already stated.** The
user's explicit confirmation — never Claude's own sense of "this is probably fine" — is what
settles a flagged item. Concretely, per step, this means: draft fully, flag assumptions
inline, route everything (batch + flagged items) through the Grouped Closing Protocol (§3.2)
for resolution, and never declare a set "sufficient" or "done" on Claude's own judgment alone.

### 3.2. The Grouped Closing Protocol

How Claude closes out any batch this phase produces (User Stories in Step 2; requirements,
test cases, tool choices, etc. in Steps 3–5) — governs **every** batch-producing step, not
just Step 2. Replaces one-item-at-a-time dictation while preserving the same guarantee: every
item individually visible, individually confirmed or rejected, before being treated as final.

**Grouping:** the full batch is organized into groups at draft time (by actor/theme, by
sub-topic, etc.) — decided once, not discovered incrementally.

**Per group, in order, not advancing until each stage is settled:**
1. **Content pass.** Present the group's batch as plain numbered text. User strikes/edits/
   confirms/adds in bulk. Settles *what exists*, not yet any attached assumptions.
2. **Assumption stage.** Table: **Item — Choices — Recommendation**, covering **every**
   flagged assumption on a surviving item, with no exceptions — including items where the
   resolution seems obvious or is just reuse of an already-accepted pattern. "This doesn't
   need real deliberation" is not license to drop to prose instead of the table; a struck
   item drops its assumption automatically, but a kept item's assumption always gets a row,
   even a one-line one. User resolves in bulk, or names specific items for individual
   treatment (e.g. "show me the analysis for items 6, 9, 13"). For named items: the full
   research/analysis summary in prose (§3.3's three-layer requirement), then the full table
   row(s) for that item, **then** the selectable-options question — no analysis option (it
   was just given; offering to provide what's already provided is dead weight) — keeping
   Defer and the substantive choices, recommendation first and labeled (§3.3). Anything
   still open (Deferred, or still ambiguous) flows forward to the next stage, not
   re-litigated here.
3. **Deferred-items stage.** Separate table (not merged with the above) of everything marked
   Defer, from any source. Same bulk-then-named-individual pattern; leftovers flow forward.
4. **Remaining-ambiguous stage.** Separate table of whatever's still unresolved. Same pattern.
   Genuine residue after this stage carries into later-step Defer resolution — and, per
   §3.13, is surfaced again at every subsequent gate until resolved or explicitly accepted
   as a standing open item.
5. **Next group**, restarting at the content pass.

**Bulk-reply handling, every stage — and any standalone flagged item outside the Grouped
Closing Protocol entirely (e.g. Step 6's per-element notation judgment, Step 8's
environment/config assumptions):** a single covering reply (e.g. "all accepted") closes the
whole table at once. A partial reply closes only what it addresses — **whatever it leaves
unresolved automatically proceeds into the per-item selectable-options loop (§3.3's
three-layer presentation first, then the tool), with no separate explicit request required.**
An explicit request to "review individually" still works, as an accelerator that skips the
bulk-offer step entirely and goes straight to per-item — it's no longer the *only* door into
per-item treatment, just the fastest one when the user already knows they want it. Claude
never treats an ambiguous reply as a trigger for anything — an ambiguous reply gets a plain
clarifying question first, per §3.1's general rule and `AGENTS.md` §3.1's Formal Approval
Protocol; "ambiguous" here means genuinely unclear what was meant, not merely partial.

**Early-resolution exception:** if a content-pass reply already explicitly answers a flagged
assumption (not just touches the item), Claude doesn't re-ask it later — unless the answer
itself raises a *new* unstated assumption, which still goes through the assumption stage.

**Completeness:** after the last group's last stage, Claude asks once whether anything's
missing from the batch as a whole — user confirmation closes it, not Claude's judgment.

**Expected scale:** bulk resolution should shrink each stage's individual round to a handful
of items — an expectation, not a hard cap; an unshrunk stage still runs against what remains.

---

### 3.3. The Selectable-Options Convention

Applies wherever Claude uses the selectable-options/multiple-choice tool (a genuinely
discrete choice — a multi-item list, like a Grouped Closing Protocol content pass, is plain
text instead; see §3.2).

- **Three-layer presentation, every time — precondition for everything below.** The
  selectable-options tool is a picker, not a display surface: it truncates each option to
  roughly one line, so it can never be the sole place substantive content lives. Before the
  tool is ever invoked, for any decision — standalone or within a Grouped Closing Protocol
  round — Claude presents, in order: (1) the full research/analysis summary in prose
  (findings, reasoning — not compressed); (2) the full **Issue — Choices — Recommendation &
  Rationale** table, complete sentences per cell, not truncated; (3) only then the
  selectable-options prompt itself, which exists solely as the input mechanism for a
  decision the user is already fully informed about from (1) and (2) — its own truncation is
  fine specifically *because* it's no longer the only place the content lives.
- **Mandatory pre-analysis.** Before building *any* such question, Claude works out real
  pros/cons of the substantive options and derives its recommendation from that — never
  picks a recommendation first and rationalizes after. This is unconditional, independent of
  whether the analysis is ever shown. **Exception:** in a Grouped Closing Protocol individual
  round (§3.2), the analysis option (below) is omitted — it was already shown as the reason
  the item reached individual treatment, so re-offering it is dead weight.
- **Research triggers — four categories, independent of decision "importance."** Not an
  exhaustive list; the user may flag additional categories as real cases surface.
  - **A — checkable/time-sensitive factual claims:** a specific tool/crate/library/version,
    or a "current best practice" assertion that could be stale. (E.g.: naming a specific
    crate without checking is exactly the kind of claim that goes stale silently. This
    explicitly includes any specific Rust toolchain or development-tool version — see
    `agents/RUST_PREFERENCES.md` §0, which is why no specific Rust version is ever hardcoded
    into a process document; it's determined fresh per project at Step 5.)
  - **B — under-determined design parameters with no objectively correct value:** research
    grounds the choice in what comparable/competitive systems converged on, rather than
    correcting a stale fact (e.g. an outage-detection threshold — no "true" answer exists,
    but convergence across real systems is informative).
  - **C — option-space completeness:** not "is this option stale" but "is Claude even aware
    of the full set of real options" — training-era knowledge may miss a newer or
    domain-specific approach a search would surface (e.g. serialization format, concurrency
    pattern, auth scheme).
  - **D — user-facing capability/feature-surface decisions:** opportunity-seeking, not
    risk-avoidance. Even when Claude's answer is correct and not stale, research checks
    whether a best-in-class/competitive-advantage/novel-capability option exists that
    wouldn't otherwise get proposed. Distinct motivation from A–C (avoiding wrong) — this is
    about not settling for adequate when better is reachable. Can overlap with B for
    product-facing parameters.
  A purely conceptual tradeoff fitting none of these doesn't need a search.
- **Recommendation first, labeled.** Option #1 is Claude's recommendation, explicitly
  labeled as such — position never silently implies endorsement. Most of these questions
  exist *because* the answer depends on something only the user knows; listing a
  recommendation is a convenience, not a claim of likely correctness.
- **Standing "Defer" option.** Means: not now. Resolved via the Grouped Closing Protocol's
  deferred-items stage (§3.2) — never an ad hoc engagement-wide list, never silently dropped,
  and re-surfaced at every subsequent gate per §3.13 until closed.
- **Standing "Provide analysis and recommendation" option, always listed last where present.**
  Since the three-layer rule above already shows the analysis upfront, this option exists
  for **re-surfacing** it (the user lost track, wants it restated) or for cases where the
  tool-capacity ladder (below) dropped analysis from inline framing text back into its own
  option. Doesn't close the question — re-presents the same question (Defer and this option
  included again) after restating. Can be selected more than once; a repeated request may
  mean the question itself needs reframing.
- **Tool-capacity ceiling (checked before the above): max 4 options total.** Defer + the
  analysis option are 2 standing slots, leaving room for only 2 substantive choices at full
  capacity.
  - **3 substantive options:** drop the analysis option from the list (keep Defer); state
    the analysis inline in the question's framing text instead. Same information, different
    delivery — pre-analysis still mandatory either way.
  - **4+ substantive options:** even Defer alone would overflow. First try to consolidate to
    ≤3 genuinely distinct choices (4+ live alternatives often means the question itself is
    underspecified). If consolidation isn't honest, fall back to plain text entirely —
    analysis and recommendation inline, Defer noted as available by reply.
- **3+ surviving options is expected to be rare — flag it when it happens.** Real analysis
  should usually collapse apparent 3-way choices to 2; a genuine 3-way survivor (most often
  an architectural tradeoff with no dominant option, e.g. performance vs. compatibility vs.
  simplicity) gets a brief explicit note distinguishing it from Claude having simply failed
  to narrow the question. Frequent unflagged 3+-option questions would itself indicate the
  analysis isn't being done rigorously.

---

### 3.4. The 9 Steps

| # | Step | Output | Mechanism |
|---|---|---|---|
| 1 | Concept Intake & Context Mapping | Architecture file `_01_introduction` (Draft) | §3.1. Mechanical: organizing/structuring the concept statement into problem-space + boundaries. Substantive (flag, don't assume): what's actually in/out of scope, where real boundaries sit, which adjacent actors matter. The single highest-cost place in the workflow for an unflagged wrong guess, since every later step inherits it — never autonomous, in any mode (§3.7). **Always attempts deep web research and competitive analysis** (how comparable systems/frameworks solve the same problem), aiming at best-in-class/competitive-advantage/novel-capability framing — an honest "no meaningful competitive landscape exists for X, here's why" is an acceptable reported outcome; a fabricated comparison is not. **Actively solicits screen mockups, reference HTML, brand/image assets, and (if relevant) audio/video assets, as early as possible** (§3.8/§2.2) — not deferred to when UI work is imminent. |
| 2 | User Story Elicitation | Architecture file `_02_user_stories` | §3.1 + §3.2 in full, run as a **single session** (no A/B pass split). Claude drafts the full candidate batch (organized by actor/theme) from `_01_introduction`; user runs the Grouped Closing Protocol over it in one pass, covering both the story list and each story's Interaction Sequence detail together, not as separate stages requiring separate sessions. |
| 3 | Requirement Decomposition | Architecture file `_03_requirements` | §3.1 + §3.2. Mechanical: 3-pass decomposition (Functional → Logical → Detailed) and the 9-criteria/Requirement-Smell check (§4.5 of `agents/DESIGN.md`), run unsupervised — these are internal iteration passes within the one Step 3 session, not separate sessions. Substantive: necessity, correctness, completeness — flagged per requirement, resolved via §3.2. |
| 4 | Test Identification | Architecture file `_04_test_strategy` | §3.1 + §3.2. Mechanical: deriving test cases from an already-verifiable requirement, plus the 9-criteria table and Rust-specific Requirement Smell catalog. Substantive: what bar a DoD must actually clear ("verifiable" only guarantees *some* objective test exists, not which one) — flagged, resolved via §3.2. |
| 5 | Verification Feasibility | Architecture file `_05_verified_traceability` | §3.1 + §3.2. Mechanical: matching a need to a tool *category*; checking Rust dependencies directly against `agents/PREFERRED_DEPENDENCIES.md` (preferred-list used without confirmation; **Forbidden** never proposed, no exceptions; **Requires-Approval** or unlisted flagged for explicit approval; `tokio`+WASM is a hard incompatibility, never proposed together); checking development tools against `agents/PREFERRED_TOOLS.md`; checking infrastructure services against `agents/PREFERRED_SERVICES.md`; **setting the project's workspace MSRV** per `agents/RUST_PREFERENCES.md` §0 (and the dual-MSRV pattern if a publishable library crate exists) — recorded here as a flagged placeholder, relocated into `_07_interfaces_and_stack` §6 at Step 6 synthesis, since that file doesn't exist yet. Substantive: real environment feasibility (infra, budget, licensing, team familiarity) for anything not list-checkable — flagged, resolved via §3.2. Claude never certifies "technical sufficiency" unilaterally; it's a proposal pending confirmation. |
| 6 | Final Architecture Synthesis (ISO 42010) | Architecture files `_06_viewpoints`, `_07_interfaces_and_stack`, `_08_constraints_and_roadmap` (created new this step); `_05_verified_traceability` (finalized — MSRV relocated out per Step 5's row above) | Mostly recombination of already-approved content from files `_01` through `_05` — less to delegate. One live judgment call: formal notation (SysML-style) vs. prose, per element. Claude states briefly *why* an element got one or the other, so the user can override per-element without re-litigating the viewpoint. Must cover all 4 mandatory viewpoints: Functional, Information (Data Dictionary), Deployment, Interface Control. This is also where the Asset Manifest (§3.8) content already accumulated in `_01` migrates into its permanent home in `_06`'s §4.13 (User Experience & UI Design). |
| 7 | Spec Audit & Phase-End QA | Final Deficiency Audit Report | **No propose-with-flagged-assumptions here — by design.** An audit's value depends on adversarial independence from the rest of the process, including Claude's own prior work; this is the last systematic check before anything is built on the Spec. Checks, on Claude's own analysis, not by asking the user to confirm assumptions: every User Story maps to a requirement; every requirement is atomic; no approved content was elided/summarized/replaced with a "see previous version" reference; no gap could force a stub/partial implementation downstream; every Asset Manifest entry is actually referenced somewhere it should be; **every deliverable file's name conforms to the §4 naming convention.** **A clean report (zero findings) closes Step 7 normally — proceed to Step 8. Any finding at all (even one) triggers the Step 7 Backtrack Workflow, §3.12, instead of normal gate approval.** |
| 8 | Development Plan & Checklist | Development Plan + Checklist + Dev-Agent Kickoff Prompt | Before drafting Plan content on environment/config (toolchain, CI, local setup — not addressed by Steps 1–7, which are about *what*, not *where/how built*): elicit concrete facts directly; for anything unspecified with a reasonable default, propose the default as a flagged assumption (§3.1), once per Plan, not once per phase. **Phase Sizing:** each phase must be completable within one agent session, sized for an agent **less capable than Claude**, with margin for debugging/complications. Complexity is computed, not eyeballed: `Score = (task_count × 1) + (new_public_interfaces × 2) + (cross_file_tasks × 2) + (cross_task_dependencies × 1.5)` — interfaces and cross-file coordination weighted higher since that's where a weaker agent is likeliest to err. Default ceiling **≤15**; user-tunable over time/as models improve — the formula's weighting is the stable part, the ceiling number is the adjustable part. **Frontend phase sequencing, where a UI component exists:** phase boundaries use *targeted interleaving* — each screen/component's frontend implementation task is placed in the same phase as the real (non-mock) backend/data dependency it needs, never earlier (which would force a throwaway stub) and never pushed wholesale into a trailing "frontend phase" — this is a Development Plan §9.1/§6 concern in detail; this file only states the principle Step 8 must apply when sizing/ordering phases. **Final deliverable:** a reusable kickoff prompt for the executing development agent, using `agents/exemplars/dev_agent_kickoff_prompt_template.md` — produced once, reused at the start of every Development-Phase session, not regenerated per session. |
| 9 | Plan & Checklist Audit | Plan & Checklist Audit Report | **No propose-with-flagged-assumptions here either — by design, mirroring Step 7.** Step 8's output was produced by the same flagged-assumption mechanism as every other step; nothing so far has independently verified it. Step 9 runs `agents/exemplars/development_plan_template.md` §15 (Plan-Level Definition of Done) directly as an audit checklist on Claude's own analysis: every Core requirement traced, no orphan citations, every phase has non-empty Entry/Exit Criteria, every task has a Verification Method and DoD, the Phase Dependency Graph is acyclic, Checklist/Plan lockstep with no drift, every filename conforms to §4. Also cross-checks Build Order fidelity against the finalized Architecture Specification and, where a UI component exists, that phase sequencing actually reflects Frontend Targeted Interleaving rather than a backend-then-frontend split. **A clean report closes the Design Phase. Any finding classifies as (A) Plan/Checklist-only — reopen Step 8 alone, fix, re-package, re-audit — or (B) Spec-originating — reopen the relevant Architecture Specification step (1–6), re-clear Step 7 to a clean result, then return to Step 8** — never patch the Plan around a still-defective Spec. This repeats until a Step 9 run is clean. See `agents/DESIGN.md` §5.9 for the full backtrack mechanics, which mirror §3.12 below. |

Every step ends with **STOP, present output, await explicit approval** — never combined,
never skipped, in full-rigor mode (§3.7 for the autonomy toggle and its presets). Per §1,
every step (and every repeated run of a step) is its own session — a step's gate output is
always a complete file package for the next session, per §3.11, and always includes the
accumulated open-items review required by §3.13.

---

### 3.5. Scope: a Three-Way Question, Not Binary

"Is this in scope" has a real third answer: wanted, just not in this build. Claude never
collapses that into a plain exclusion.

- **Default case — ask which is meant:** not wanted at all (Architecture Spec §2.5,
  Out-of-Scope) vs. wanted but deferred (§2.6, Future Features). Claude doesn't guess between
  these — they have materially different downstream handling. Once "deferred" is confirmed,
  Claude records it in §2.6 per that section's mandate (originating ID if any, sufficient
  description, deferral reason, known dependency/precondition) — not just a conversational
  mention. If the user already states which is meant in the same message, Claude doesn't ask
  redundantly (§3.2's early-resolution exception).
- **Feature-level stackable/incremental choices (e.g. MVP-floor decisions) default to
  Future Features**, stated explicitly not silently (e.g. "the richer level goes to Future
  Features unless you'd rather drop it"), overridable in the same reply. This default is for
  genuine feature/product-scope choices only.
- **Implementation-level choices are a distinct category** — an algorithm or strategy choice
  for an already-in-scope requirement isn't a scope decision at all (the requirement is in
  scope regardless; what's decided is *how*). These route to the Architecture Spec's **§4.15**
  (deferred, noted only — no commitment) or **§4.16** (deferred, with an extension point
  Claude must actually specify in §4.4/§4.9, not just index) — Claude asks which is meant
  when unclear, sub-grouped by originating ID in either case.
  - **Claude actively weighs "ship simple + commit an extension point" as a candidate
    answer within the mandatory pre-analysis itself (§3.3)** — not only after the user has
    already chosen — especially whenever an option carries real scope/implementation-effort
    impact, where the cost of retrofitting later vs. committing a cheap extension point now
    is itself part of what the analysis must surface.

### 3.6. Backtracking & Major Changes

Per `agents/DESIGN.md`, the user may return to any previous step. Most often this surfaces
when a later step reveals an earlier step's content — flagged or not — was wrong. When it
happens via normal forward iteration (i.e. *not* a Step 7 finding — see §3.12 for that
specific, mandatory case), Claude: (1) names the originating step and the specific content
at issue, rather than patching around it; (2) presents — doesn't unilaterally decide — the
choice between a local patch versus re-opening the earlier step, since that determines how
much upstream work needs re-approval; (3) if re-opened, preserves all already-approved later
content, revisiting it only as needed once the earlier step is re-approved (additive-only,
no wholesale discard); (4) flags it as a Major Change whenever it materially changes
scope/requirements/architecture — not just when convenient to call one. **Major Changes
generally** are flagged explicitly, by name, the moment Claude recognizes one from any
iteration — never buried in a diff or folded silently into the next deliverable.

---

### 3.7. Autonomy: a Per-Step Toggle, with Two Named Presets

Each step is either **interactive** (its normal §3.4 behavior, full elicitation per §3.1/3.2)
or **autonomous** (internal process runs silently — no mid-step elicitation — but the
**gate itself is unchanged**: Claude still stops and presents output for approval). This is
**prompt-invoked**, never a standing default; full-rigor, all-interactive (§3.4) is the
baseline absent an explicit instruction.

**Which steps can ever be autonomous:** 2, 3, 4+5, 6, 8. **Steps 1, 7, and 9 can never be
autonomous, in any mode** — Step 1 because it's the highest-cost place for an unflagged
wrong guess (§3.4); Step 7 and Step 9 because audit independence is non-negotiable for both
(§3.4). No prompt overrides this. ("4+5" here means Steps 4 and 5 are handled as one combined
autonomous session when this toggle is invoked — the only case where two Steps share a
session; in interactive/default mode they remain separate sessions per §1/§3.4.)

**Autonomous gate output, every eligible step:** not just a result — a consolidated
**"Assumption/Issue — Choices — Decision & Rationale"** note per substantive item resolved
internally, alongside the step's normal output, plus the §3.13 open-items review like any
other gate. This is richer than a flat assumptions list: the user sees what the live options
were and why Claude picked one, the same content a selectable-options question would have
shown if asked interactively, just presented as a statement rather than a question.

**Two named presets**, since the only real difference between "small project" and "trust
this step's judgment" is whether Step 2 happens at all:

- **Tailored Mode** ("treat this as a small project, tailored mode"): Step 2 **skipped**
  entirely (not autonomous — gone, since the project is small enough that Step 1 absorbs
  what it would have surfaced); Steps 3, 4+5, 6, 8 autonomous by default; Step 1 unconditionally
  high-rigor per §3.4 *and* additionally expanded to cover persona/interaction/edge-case
  content Step 2 would otherwise have surfaced.
- **Full Autonomous Mode** ("run this project in full autonomous mode"): identical to
  Tailored Mode except Step 2 **runs**, autonomously, instead of being skipped — for a
  full-size project where Step 2's actual User Story artifact is still wanted, just without
  interactive elicitation.

**Single-step invocation** (independent of either preset, on any project): "run Step [N] in
autonomous mode" — toggles one eligible step (2, 3, 4+5, 6, or 8) without invoking a whole
preset. No combined shorthand exists for "preset + extra step" — name the preset and the
additional step explicitly if stacking (though note there's little left to stack once a
preset is active, since both presets already make every eligible step but 2-in-Tailored
autonomous by default).

Every gate in §3.4 still applies regardless of preset or toggle — autonomy changes only the
*visibility of process*, never removes a stop-and-approve point.

### 3.8. Visual & Media Asset Input: Discovery Source and Tracked Artifact

The user may provide **HTML, images, and (where relevant) audio/video** showing or
constituting how the system's UI should look, sound, and behave. This input plays two
distinct roles that must both be honored: it is a **discovery source** (revealing data-model
concepts, missing entities, or requirements no verbal description surfaced), and it is a
**tracked artifact** carried by filename through Design and into Development, where the same
files land in fixed repository paths for the development agent to consume directly.

**Authoritativeness differs by asset type — this is a deliberate, non-uniform rule:**
- **HTML is presumptively authoritative for structure and behavior.** Because the user has
  stated HTML input is "highly representative of the desired final appearance," Claude treats
  a provided HTML file as the default source of truth for a screen/component's markup
  structure, layout, conditional visibility, validation behavior, and exact copy text — not
  merely informative styling guidance to be freely reinterpreted. Deviating from what an HTML
  asset specifies is itself a flagged item (§3.1), not a silent judgment call, the same as any
  other case where Claude departs from stated input.
- **Images are informative, not structurally binding**, and are treated as a **design-token
  source**: branding, iconography, typography, color palette, and pattern/texture tiles, plus
  any other graphical information they carry. This feeds the Architecture Spec's Design
  System content (within §4.13, User Experience & UI Design) rather than dictating layout
  structure the way HTML does.
- **Audio/video assets** are logged in the Asset Manifest (below) and their intended usage is
  captured in requirements/UX content as normal, but Claude does not attempt to "consume"
  their content directly the way it reads HTML/images — they are tracked for the development
  agent's benefit, not analyzed by Claude for discovery purposes.

**Solicited as early as possible, not just at Step 1.** Step 1 intake actively asks whether
screen mockups, reference HTML, brand/image assets, or (if applicable) audio/video assets
exist, per §3.4's Step 1 row — but the user is explicitly encouraged to provide these assets
at whatever point they become available during the engagement, since later steps benefit from
having them sooner rather than only being asked once at the start.

**The Asset Manifest — mandatory tracking record.** The moment any asset is provided, at any
step, Claude logs it in the Asset Manifest: a table with columns `Filename | Type (html /
image / audio / video) | Repository Target Path | What It's Authoritative/Informative For |
Provided At (Step)`. Filenames are **immutable once logged** — Claude never proposes a
rename; a filename collision or ambiguity is a flagged item requiring user resolution, never
silently disambiguated by Claude picking a new name. Repository target paths are fixed by
convention: `assets/html/`, `assets/images/`, `assets/audio/`, `assets/video/` — the same
paths and the same filenames the user will populate in the project's GitHub repository for
the development agent to consume, so an asset referenced anywhere in the Architecture
Specification or Development Plan resolves unambiguously to a real repo file with zero
translation needed between Design and Development. Until Step 6 relocates it into its
permanent home in `_06_viewpoints`'s §4.13, the Asset Manifest accumulates inside whichever
file is current at the step the asset was provided (per §4's Step-aligned file mapping) —
this is the same flagged-placeholder-then-relocate pattern used for the Step 5 MSRV decision.

**Triggers a completeness check whenever provided, at any point in the workflow** — not only
at Step 1. Claude checks already-approved Data Dictionary/requirements content against what
the visual input implies. Any gap found is a **flagged open item for user confirmation**
(§3.1), routed through the Grouped Closing Protocol (§3.2) the same as any other flagged
batch item — never silently patched into already-approved content, regardless of how
obviously implied the gap seems.

**Both image and HTML are reviewed when both are provided for the same screen/component —
neither substitutes for the other**, consistent with their different roles above: HTML
exposes behavior/state/content an image can't show; an image shows visual layout and design
tokens raw markup doesn't make obvious. Each is checked for what only it reveals.

**Frontend framework/crate implications.** Where HTML input implies a specific Rust/WASM UI
framework's capabilities are needed (e.g. specific interactivity patterns), that choice still
goes through Step 5's normal `agents/PREFERRED_DEPENDENCIES.md`/`agents/PREFERRED_TOOLS.md`
feasibility check like any other dependency — visual input motivates the need but does not
bypass the preferred-dependency approval flow.

### 3.9. No Agent-Side Certification of Technical Sufficiency

Confirmed, not a gap: Claude cannot certify technical sufficiency unilaterally at any step
(§3.4, Step 5 states this explicitly; it holds everywhere else too). Research (§3.3, §3.4
Step 1) narrows the substantive gap between Claude's proposal and the right answer — it
never closes it, since real infrastructure, budget, and team constraints remain unverifiable
from inside this chat. More rigor produces a better-informed proposal, never a certified one.

### 3.10. Requesting More Depth ("Not Enough — Another Pass")

At any point after a Steps 1, 2, 3, 4, 5, 6, or 8 gate (not 7 — see §3.4 and §3.12), the user
may request a deeper pass rather than a correction — e.g. "not detailed enough, another
pass," or "double the number of stories," "more detail in the interface specification."
This is distinct from the Grouped Closing Protocol (§3.2), which handles *correctness*
("this item is wrong"); this handles *insufficiency* ("this is right but thin"). The user
may scope the request narrowly (a specific viewpoint, a specific phase) or broadly (the
whole step's output), and may give a concrete metric — Claude treats a given metric as the
actual target, not a vague cue to add a little more. Each such repeated run of a step is its
own new session (§1) and is named `pass2`, `pass3`, etc. in its handoff-note filename per
§4's naming convention — the underlying Architecture file it re-touches keeps its own
independent version history and is simply bumped again at that session's end, per §4's
versioning rule.

### 3.11. File Delivery for Anything Crossing a Session Boundary

Per §1's session-per-Step reality: anything a *later* session needs to resume, review, or
approve must be delivered as an actual downloadable file — never left only as chat text the
user would have to copy out manually. This covers, at minimum: the handoff note itself (see
below); the Architecture Specification file(s) current as of the end of this session — per
§4, each Step now owns and directly edits one specific numbered file rather than producing a
separately-named intermediate artifact, so "deliver the Step's output" and "deliver the
current version of that file" are the same action, not two. In short — **anything presented
at a Step Approval Gate is a file.** Content that only matters within the single session that
produced it (e.g. back-and-forth while resolving a Grouped Closing Protocol stage before that
stage's table is finalized) may stay as chat text, since the next session only needs the
*settled* result, not the process that produced it.

**Handoff notes are themselves a separate `.md` file** — never inline chat text the user
would need to copy and paste — named per the standardized convention:

```
[projectname]_design_handoff_stepN_passX.md
```

Where `N` is the step number and `X` is the pass identifier: `1` for a step's first run,
incrementing (`pass2`, `pass3`, ...) for a repeated run of that step (user-requested per
§3.10, or an automatically-triggered re-run such as a Step 7 re-audit per §3.12). Since Step
2 is now a single session (§1, §3.4), it uses this same `pass1`/`pass2`/... scheme like every
other step — there is no `passA`/`passB` variant anywhere in this convention. Examples:
`myproject_design_handoff_step1_pass1.md`, `myproject_design_handoff_step2_pass1.md`,
`myproject_design_handoff_step7_pass1.md`, `myproject_design_handoff_step7_pass2.md` (a
second Step 7 audit after a backtrack remediation per §3.12).

A handoff note is load-bearing, not a courtesy summary: since the next session has no memory
of the current one, starting that next session requires *actually providing* the handoff
note file (plus the current version of whatever Architecture file(s) it hands off) —
announcing "continue from Step 3" without attaching `_03_requirements` at its current version
and the handoff note gives the new session nothing to resume from.

**Every handoff note MUST open with an explicit, unambiguous Next Action directive — not
just a file list and background.** A session that receives a full file package but no
imperative instruction has, in practice, reported back "no explicit ask for this session"
and offered the obvious next step as one option among several instead of simply starting it
— this is a real, observed failure mode, not a hypothetical one, and it wastes a full
round-trip on something the handoff note already knew. The directive is the first line of
the note, in this exact form:

```
## Next Action
Start Step N[, Pass X].
```

(e.g. `Start Step 4.`, or `Start Step 7, pass 2.` for a repeated run). This is a command to
execute immediately upon receiving the note and its accompanying files, not a suggestion to
surface as a question — the receiving session opens with the Step's own work (per §3.4's
row for that Step), not with "what would you like me to do?" A handoff note without this
line is incomplete and must be corrected before being considered ready for handoff, the same
as a missing file-list entry.

**The handoff note MUST explicitly enumerate every file the next session needs** — a
complete list, not just the current session's own output. This includes files produced in an
*earlier* session that remain relevant — e.g. Step 4's handoff note names `_01_introduction`,
`_02_user_stories`, and `_03_requirements` (at their current versions) as still-needed
alongside Step 4's own `_04_test_strategy`, since Step 4 reads all of them — so the user knows
exactly what to gather and attach before starting the next session, without having to
reconstruct the dependency chain themselves. A handoff note that only names what *this*
session produced, silently omitting earlier-session files the next session will still need,
doesn't satisfy this requirement. **This same file list is also stated directly in the chat
response itself**, in the same turn the handoff note is produced — not only inside the file —
so the user can see what to gather without first having to open and read the note.

**Pre-handoff staleness check, mandatory, immediately before producing the handoff note.**
Claude enumerates every Architecture file touched at any point this session and compares each
against what was last actually shown/downloaded to the user. Any file that changed since —
including ones edited early in the session and not revisited since — is re-presented for
download in the same handoff batch. **This check explicitly includes files belonging to a
step other than the current one** — a backtrack session (§3.12) or any session that revisits
an earlier step's file per §3.6 touches files outside its own "home" step, and those files
are exactly as subject to this check, and to the version-bump rule immediately below, as the
current step's own file. This has been observed failing in practice specifically on
second/third backtrack passes and on any session that had to touch a previous step's file —
Claude treats "did I touch a file this session, regardless of whose step it belongs to" as
the actual test, never "did I touch this step's own file." This is never a partial subset
left for the user to separately ask "and the rest?" — the check covers every touched file,
every time, as a deliberate step before handoff, not something left to memory of what feels
relevant. **This same check also verifies that every file produced or touched this session
conforms to the §4 naming convention** — a misnamed file is corrected (renamed/regenerated
under the correct name) before handoff, not flagged for a later pass to fix. Step 7's audit
(§3.4, §3.12) re-checks this project-wide, independent of any single session's own staleness
check, as part of its adversarial review.

**Version-bump enforcement, mandatory, as the literal last action before handoff.** Per
§4.3's rule (bump exactly once, at session end, per file actually touched this session):
immediately after the staleness check above identifies every file touched this session —
including files outside the current step, per the previous paragraph — Claude bumps each
one's version number by exactly one before presenting it, with no exceptions for a file that
was only touched briefly, only touched on a backtrack pass, or only touched for a small
cascading fix. A file listed as "touched this session" that is handed off at an unchanged
version number is a defect in the handoff itself, not a minor omission — this is precisely
the failure mode that has been observed recurring on backtrack passes and on sessions
touching a previous step's file, and it is what this explicit final check exists to catch
before the file ever reaches the user.

**The next session verifies its inputs against the handoff note's manifest before
proceeding.** Where a handoff note exists (i.e. any session resuming from a prior one — not
a fresh Step 1 with nothing to resume from), Claude checks the files actually provided
(uploaded, pasted, linked) directly against that note's own enumerated file list — a
concrete comparison against what's already written down, not Claude's independent judgment
of what the step "should" need. If something the manifest names is missing, Claude says so
and asks for it directly, rather than proceeding on incomplete inputs or silently assuming
the missing content. This mirrors `agents/exemplars/development_plan_template.md` §11's
Session Handoff Protocol (confirm state before starting any task) applied to Design Phase
sessions as well, not only Development Phase ones.

---

### 3.12. The Step 7 Backtrack Workflow

Step 7 (Spec Audit) is, by design (§3.4), a pure **finder**, never a fixer — it runs no
propose-with-flagged-assumptions, no Grouped Closing Protocol, and never patches content
itself. This is what makes its independence meaningful. A direct consequence: **a Step 7
finding is never resolved via the ordinary local-patch-vs-reopen choice in §3.6/`AGENTS.md`
§2.1** — every finding, without exception, requires reopening the step that should have
produced or caught it. The choice §3.6 normally presents (patch locally vs. reopen) doesn't
apply here, because Step 7 itself never touches content — there is no "local patch at Step
7" to choose between in the first place.

**If Step 7's audit report has zero findings:** the gate behaves normally — present the
clean report, await approval, proceed to Step 8.

**If Step 7's audit report has any findings at all (even one):** the normal Step 7 gate does
not close as an approval gate. Instead:

1. **Map every finding to its originating step.** For each finding, identify which step
   (1–6) should have produced or caught the content at issue — and, per §4, which specific
   numbered Architecture file that step owns.
2. **Determine the single earliest originating step across all findings.** If findings trace
   back to multiple different steps (e.g. one to Step 3, one to Step 4, one to Step 5), do
   **not** reopen each step separately or reopen the same step multiple times. Identify the
   earliest of the originating steps and start there.
3. **Open one new "backtrack" session** (per §1, this is its own new session — Step 7's
   handoff note for this session, see below, makes clear what kind of session it is and
   what it must accomplish). Within that single session, working forward from the earliest
   originating step through to Step 7 again:
   - Fix every finding that traces to the current step, editing that step's own owned
     Architecture file directly.
   - **Re-check already-approved later-step content (in later steps' own owned files) for
     cascading effects of the fix**, per §3.6's existing "revisit only as needed" principle —
     this is discovered by re-checking within the backtrack session as each fix is made, not
     pre-identified by Step 7 itself (Step 7 only ever reports what it found at the step it
     could observe; it does not predict downstream consequences of a fix that hasn't happened
     yet).
   - Move to the next step in sequence (whether or not a finding originally traced to it),
     applying any cascading fix that step's owned file now needs as a result of the prior
     step's fix, plus any finding that traced directly to this step.
   - Continue this way through every step up to and including Step 6, so that by the end of
     the session every originating step's file and every downstream file has been
     reconciled — both the original findings and their cascading effects.
   - **Do not re-run Step 7 itself within this session.** The backtrack session's job is to
     get the Spec back to a state ready for a fresh, independent Step 7 audit — not to
     self-certify that it succeeded.
4. **At the end of the backtrack session, package every file exactly as at any other Step
   Approval Gate** (§1, §3.11): every Architecture Specification file touched, each with its
   own version bumped exactly once for this session per §4's versioning rule (never once per
   finding fixed within it — one session, one bump, per file actually touched); a handoff
   note specifically for the next session.
5. **The handoff note for this packaging MUST state plainly that the next session is a new
   Step 7 audit (pass2, pass3, ... — see §3.11's naming convention), not a continuation of
   the backtrack session, and MUST summarize**: which findings were fixed, which steps (and
   which owned files) were reopened and in what order, and **explicitly that any cascading
   effects beyond the directly-fixed content were discovered and resolved by re-checking
   later-step files during the backtrack session itself** (not pre-identified by the original
   Step 7 report) — so the new Step 7 session understands the full scope of what changed and
   why, not just the original finding list.
6. **The new Step 7 session re-audits the entire Spec from scratch**, with the same
   adversarial independence as any Step 7 run (§3.4) — it does not take the backtrack
   session's own account of what it fixed as given; it re-derives findings independently.
   If this audit is clean, proceed to Step 8 normally. If it finds anything (even something
   new, unrelated to the original findings), this entire workflow (steps 1–6 above) repeats.

**Major Change Notification still applies** (§3.6 point 4): if any fix made during a
backtrack session materially changes scope, requirements, or architecture, Claude flags it
as a Major Change at the point the fix is made, the same as any other backtracking.

---

### 3.13. End-of-Step Open-Items Review

**Every Step Approval Gate (§3.4), without exception, includes a standing review of every
unresolved item accumulated so far — not only items generated by the current step.** This
is distinct from, and additional to, the Grouped Closing Protocol's own per-batch Deferred
and Remaining-Ambiguous stages (§3.2): those close out *that step's own* batch before the
gate; this reviews the **running total across every step so far**, including items deferred
or left ambiguous in earlier, already-approved steps, in an explicit attempt to shrink the
open list before it compounds further.

**Mechanism, every gate:**
1. Claude maintains a running **Open Items Register** — every Deferred item (§3.2 stage 3),
   every Remaining-Ambiguous item (§3.2 stage 4) not subsequently resolved, and any other
   explicitly-flagged-but-unresolved assumption from any prior step, each tagged with its
   originating step and the step(s) it has already been re-surfaced at.
2. At every gate, before presenting the step's own headline output for approval, Claude
   presents the current state of this Register in full — not just items newly added this
   step — organized by originating step, oldest first.
3. For each item still open, Claude states plainly whether anything learned in the
   intervening steps changes the analysis (e.g. a later step's content may have implicitly
   answered an earlier open question, or narrowed the live options) — this is an active
   attempt to close items, not a passive restatement of the same list every time.
4. The user resolves in bulk or per-item, using the same bulk-reply/per-item mechanics as
   the Grouped Closing Protocol (§3.2's bulk-reply handling applies directly here). An item
   the user explicitly chooses to leave open (rather than resolve) stays on the Register and
   is carried forward again at the next gate — it is never dropped silently just because it
   was re-presented and not addressed in a given reply.
5. The Register itself is part of what §3.11 requires as a file at every handoff — it is not
   chat-text-only, since a later session has no memory of what's still open otherwise.

**This does not relax §3.2's own per-step closure stages** — a step's own batch still goes
through its full Grouped Closing Protocol before that step's gate. §3.13 is the additional,
cumulative check layered on top, specifically aimed at preventing an item deferred at, say,
Step 2 from silently riding unresolved all the way to Step 7's audit without ever being
re-offered a chance at resolution in between.

---

## 4. Multi-File Output & Cross-Linking

Applies once a Spec or Plan deliverable spans multiple files. All files for one artifact
live in the same flat directory (no nesting, unless the user changes this). Names below are
**base names** — Architecture Spec and Plan files (not the Checklist — see below) carry a
version suffix per the versioning rule further down this section. `[projectname]` is the
actual project's name (a short, filesystem-safe slug — underscores, not hyphens, per
`AGENTS.md` §2.5), confirmed with the user at Step 1 if not already established by the
concept statement.

**Naming convention — `[projectname]` first, then artifact type, then a zero-padded
two-digit sequence number starting at `01`, then topic, then version suffix.** This ordering
(projectname-first) is deliberate: it keeps every file belonging to one project's Spec and
Plan sorting together, and architecture/dev-plan/checklist/prompt files interleaving in a
sensible order, when a directory listing is sorted alphabetically — rather than scattering
files across the listing by generic numeric prefix alone.

### 4.1. Architecture Specification — 8 Files, One Per Design Step

**The Architecture Specification is split into 8 files, one per originating Design Step
(Steps 1–6; Steps 4 and 5 each still get their own file even though §3.7's autonomous "4+5"
combination can run them in one session — file ownership tracks the template's content
boundary, not whether a given run happened to combine sessions).** This supersedes any
coarser topic-based grouping: earlier, smaller projects could reasonably fold multiple Design
Steps' output into a shared handful of files, but for projects of the scale this template
targets (50+ User Stories, 100–200+ requirements are common), a coarser grouping produces
single files thousands of lines long and — more importantly — forces every Step's session to
bump a shared file's version number even when that session touched none of the file's
pre-existing content. One-file-per-Step avoids both problems: each file's size stays bounded
by what one Step actually produces, and each file's version history reflects only the
sessions that actually changed it.

Pattern: `[projectname]_architecture_NN_<topic>_v[N].md`, where `NN` and `<topic>` are fixed
per the table below and `[N]` is that specific file's own independent version counter (see
§4.3):

| File (`NN_topic`) | Template content | Owning Step | Created at |
|---|---|---|---|
| `01_introduction` | §1 Introduction | Step 1 | Step 1 |
| `02_user_stories` | §2.1 Personas, §2.2 User Stories, §2.5 Out-of-Scope, §2.6 Future Features | Step 2 | Step 2 |
| `03_requirements` | §2.3 Core Functional Requirements, §2.4 Non-Functional/Quality Attribute Scenarios | Step 3 | Step 3 |
| `04_test_strategy` | §3.1 9 Criteria (self-check reference), §3.2 Test Case Catalog, §3.3 Rust-Specific Requirement Smells | Step 4 | Step 4 |
| `05_verified_traceability` | §3.4 Traceability Matrix, §3.5 Acceptance Criteria Detail; temporarily also holds the Step 5 MSRV decision and any Asset Manifest entries logged before Step 6, both flagged as placeholders pending relocation | Step 5 | Step 5 |
| `06_viewpoints` | §4, all four ISO 42010 viewpoints (Functional, Information, Deployment, Interface Control) plus §4.5–4.16 in full, including §4.13's Asset Manifest (relocated here from `05` at this step) | Step 6 | Step 6 |
| `07_interfaces_and_stack` | §5 External Interfaces & Integrations, §6 Technology Stack & Dependencies (including the finalized MSRV, relocated here from `05` at this step) | Step 6 | Step 6 |
| `08_constraints_and_roadmap` | §7 Constraints & Assumptions, §8 Risks & Technical Debt, §9 Implementation Roadmap & Build Order, §10 Public API & Framework Consumer Contract, §11 Appendices | Step 6 | Step 6 |

If `06_viewpoints` alone overruns a reasonable file size once real content volume is known
(likely, per the template's own note that §4 is "by far the largest section"), Claude flags
this at Step 6 and proposes a further split (e.g. one file per viewpoint,
`06a_functional_view`, `06b_information_view`, etc.) rather than silently overrunning — the
same scale-driven-split principle that produced the 8-file structure in the first place
applies recursively here if warranted.

**Step 7's Audit Report and every handoff note remain outside this numbered set**, using the
handoff-note naming convention (§3.11) instead — they are process/review artifacts, not
Architecture Specification content, and are never assigned an `NN_topic` slot.

### 4.2. Development Plan (15 sections → 4 files), Development Checklist, Kickoff Prompt

Unchanged from prior structure — the Development Plan's coarser grouping is intentional and
distinct from the Architecture Specification's new Step-aligned scheme (see §2's note on why
these two artifacts are split at different granularities). Pattern
`[projectname]_dev_plan_NN_<topic>_v[N].md`:
- `[projectname]_dev_plan_01_overview_v[N].md` — §0 Architecture Cross-Reference, §1
  Introduction, §2 Technology Stack, §3 Project Folder Structure
- `[projectname]_dev_plan_02_environment_and_phases_v[N].md` — §4 Environment Setup, §5
  Dev/Test Configuration, §6 Phases and Milestones (including the frontend targeted-
  interleaving sequencing principle from §3.4's Step 8 row, elaborated in full here per
  §9.1's Sequencing Principles)
- `[projectname]_dev_plan_03_tasks_and_testing_v[N].md` — §7 Risk Management, §8 Task
  Decomposition, §9 Test Strategy, §10 Logging Strategy
- `[projectname]_dev_plan_04_protocols_and_dod_v[N].md` — §11 Session Handoff, §12
  Abort/Rollback, §13 Escalation Triggers, §14 Change Control, §15 Plan-Level Definition of
  Done

**Development Checklist** (single file unless content genuinely warrants a split, per §2):
`[projectname]_dev_checklist.md`, base name — no numeric prefix, since it's conventionally
one file covering the whole Plan, not a member of a numbered set. **Does NOT carry a
version suffix**, for the same in-place-editing reason as the Kickoff Prompt below. Per
`AGENTS.md` §2.7, development agents MUST edit this file in place — never copy, rename, or
otherwise reproduce a one-off version of it.

**Development-Agent Kickoff Prompt:** `[projectname]_dev_prompt.md`, base name — no numeric
prefix, no version suffix, produced once at the end of Step 8, reused verbatim at the start
of every Development-Phase session.

**Phase Summaries** (produced during Development Phase, per
`agents/exemplars/development_plan_template.md` §11.3): `[projectname]_phaseN_summary.md` —
carries the `[projectname]` prefix for consistency with every other artifact in this naming
scheme, even though it's a Development-Phase, not Design-Phase, artifact.

This grouping is a starting proposal for the Plan (the Architecture Spec's 8-file grouping
above is fixed, not provisional, given it's now tied directly to Step ownership) — Claude
revisits the Plan's grouping once real content volume is known at Step 8, flagging if a
different grouping serves the target better, rather than forcing content into a pre-decided
shape.

**Linking:** the first file in each numbered set is an index (overview + linked TOC to the
rest, relative links, co-located assumed) — for the Architecture Specification, this is
`01_introduction`. Every other file links back to the index and forward/sideways wherever
there's a substantive content dependency — based on actual dependency, not just sequence,
and using heading-derived anchors to point at a specific section, not just a file's top.
(E.g.: `06_viewpoints`'s Interface Control Document referencing a Data Dictionary type
defined earlier in the same file is an in-file anchor; `08_constraints_and_roadmap`'s Roadmap
referencing a requirement defined in `03_requirements` is a cross-file link.) The Spec and
Plan file sets are independent — they don't cross-link into each other absent a concrete
reason (e.g. a Plan phase explicitly implementing a specific ICD interface).

**Intermediate/handoff artifacts** (handoff notes, the Open Items Register per §3.13,
working notes, draft batches, anything produced mid-step rather than as settled Architecture
file content) use the standardized handoff-note naming convention from §3.11:
`[projectname]_design_handoff_stepN_passX.md`. This is distinct from the numbered
Architecture/Plan files above, which are organized by content section (and, for the
Architecture Spec, by owning Step), not by which specific session produced a given revision.

### 4.3. Versioning: Per-File, Bumped Once at Session End

**Every Architecture Specification and Development Plan file carries its own independent
version suffix, `_v[N]`, starting at `v1` when the file is first created.** Two rules govern
how `[N]` changes, and both matter:

- **Independent per file, not shared across a set.** There is no single project-wide or
  artifact-wide version number. `01_introduction` may be at `v4` (revisited across several
  backtrack sessions) while `03_requirements` is still at `v1` (untouched since Step 3) —
  this is normal and expected, not an inconsistency to reconcile. Any example filename shown
  elsewhere in this document or its companion templates that appears to imply a single shared
  `[N]` across multiple files (e.g. a kickoff-prompt file list showing every Architecture
  file at the same placeholder version) is illustrative shorthand only — the real,
  authoritative version of each file must always be read off that file's own actual name, not
  inferred from a sibling file's version.
- **Bumped exactly once, at the end of the session that touched it — never mid-session, and
  never more than once per session regardless of how many edits or how many distinct fixes
  that session made to the file.** Within a single session, a file may be edited many times
  (responding to Grouped Closing Protocol rounds, incorporating corrections, resolving
  flagged items) — all of that happens under the *same* version number, since a version bump
  exists to prevent download/upload collisions *across* session boundaries, not to track
  every internal edit. Only when the session ends and the file is packaged for handoff (§3.11)
  does its version number increase by exactly one, reflecting that this file is now different
  from what the next session (or the user's own records) last saw. A file untouched in a
  given session keeps its existing version number unchanged — it is not redelivered or
  re-bumped just because other files in the same handoff batch were. **"Touched" means any
  file whose content Claude changed this session, full stop — it is not limited to the
  current step's own file.** A backtrack session (§3.12) or any ordinary §3.6 revisit that
  edits an earlier step's file bumps that file exactly once too, at the same session-end
  point as the current step's own file, via the mandatory enforcement check in §3.11. This
  needs stating explicitly because it is the specific case that has been observed slipping
  through in practice — a session correctly bumping its own step's file while leaving a
  revisited earlier-step file's version number unchanged.

This is the fix for a real, repeated download/upload collision that motivated file
versioning in the first place: the same base filename being re-downloaded across multiple
sessions under an identical name. Bumping strictly at session-end, per file actually changed,
solves that collision without also punishing every file in a set for one file's edits, and
without turning mid-session iteration into a churn of version numbers no one can usefully
track.

**This has a real, accepted cost: cross-links break on every version bump, and rewriting
them is mandatory, not optional.** Per §4's cross-linking convention, files in a set link to
each other by name — when a file's version suffix changes, every *other* file in the set
that links to it now points at a stale name. Whenever Claude re-presents a versioned file
with new content, it MUST, in the same pass: (1) update that file's own internal links if
their targets also changed version, and (2) check every other file in the current set for
links pointing at this file's *old* version name, updating them to the new one. This is not
a one-time fix — it recurs every version bump, and skipping it silently reintroduces broken
cross-links the same way skipping the filename versioning reintroduced download collisions.

---

## 5. Other Standing Mandates (from `AGENTS.md`)

Pulled forward because they apply directly to how Claude produces/revises documents here:
**additive-only by default** (no removing/"cleaning up" approved content without explicit
deletion approval); **no self-referential elision** (never "see Section X of the previous
version" — every document stays self-contained); **no stubs** (sections written to full
depth when presented, not sketched and deferred); **formal approval protocol** (only an
unambiguous "approved"/"proceed" is the green light; new instructions + approval = partial
approval, incorporated then re-presented); **`CHAT:` prefix** = pure question, no document
changes, no gate progression, no side effects of any kind (`AGENTS.md` §3.2.1); **no
assumptions on ambiguity** — ask rather than guess and build on the guess; **no reproducing
shared working documents** — the Development Checklist and Kickoff Prompt are edited/reused
in place, never copied or renamed (`AGENTS.md` §2.7).

---

## 6. Out of Scope for This File

Development Phase execution behavior (see `agents/DEVELOPMENT.md`); `agents/SCRIPT_RULES.md`
(no script execution in Advisory Mode). The concrete task-level sequencing mechanics of
frontend targeted interleaving (§3.4's Step 8 row states only the principle Design Phase
must apply when sizing phases — the full mechanism belongs in
`agents/exemplars/development_plan_template.md` §6/§9.1 and `agents/DEVELOPMENT.md`).
`AGENTS.md` §2.5's naming convention (underscores, not hyphens) isn't a standalone concern
for these Markdown documents, but is followed anyway for any Rust identifiers/crate names
Claude introduces in examples, for consistency.

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
