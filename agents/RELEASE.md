# Release Phase Guide

> Peer phase to `DESIGN.md`, `DEVELOPMENT.md`, and `MAINTENANCE.md` — governs polished,
> user/developer-facing documentation and docs-site collateral: the tier of VS Code's docs,
> ffmpeg.org, and Dioxus/Angular's docs sites. Read `AGENTS.md`, `CLAUDE.md`, `DESIGN.md`,
> `DEVELOPMENT.md`, and `MAINTENANCE.md` in full before starting, if not already familiar.
>
> See `CHANGELOG.md` for version history.

---

## 0. Reading Requirements (Selective File Reading)

**Invariant set — every Release Phase session, regardless of Step:** `AGENTS.md`,
`CLAUDE.md`, and this file (`RELEASE.md`) itself. No table entry below repeats these
three. Project-specific Architecture Specification/`CHANGELOG.md` content (e.g. §1.5, §2.1–
§2.3) is not a grail file and so is not listed here either — it's read per each Step's own
instructions in §6 above, regardless of which grail files also apply.

**File-level selectivity, escape valve, and Step re-run rule** — identical mechanics to
`DESIGN.md` §0; not restated here.

| Step | Title | Additional grail files this Step needs (beyond the invariant set) |
|---|---|---|
| 1 | Elicitation | None. |
| 2 | Scaffold | None — mechanical, Jules-executable. |
| 3 | Terminology/Concept Spine | None. |
| 4 | Tutorials | None. |
| 5 | How-To Guides | None. |
| 6 | Explanation | None. |
| 7 | Reference-Sync | `MAINTENANCE.md` §11 (Release Checklist gating state), or `DEVELOPMENT.md`'s Final Verification section for a project's first release. |
| 8 | Finalize | None. |
| 9 | Consistency Audit | None. |

## 1. What RELEASE Is, and What It Explicitly Is Not

RELEASE governs **polished, user-facing and developer-facing documentation collateral** —
tutorials, conceptual guides, how-to walkthroughs, and reference material, published as a
standalone docs site — plus the optional peer web content (landing page, blog, community,
press kit, legal pages) that accompanies it.

**Explicitly distinct from, and does not replace:**
- **`README.md`** (`README_template.md`) — stays exactly as it is, a functional project-root
  file (Quick Start, badges, license). RELEASE's output is additional, richer collateral —
  see §11 for the precise relationship.
- **The Architecture Specification** — stays the technical, requirements-traceable source of
  truth for Design/Development/Maintenance's own internal use. RELEASE draws on it as source
  material but never modifies or supersedes it.
- **`MAINTENANCE.md`'s own process documentation** — maintainer/ops documentation (how a
  maintainer keeps a project running) is out of scope for RELEASE, which is about
  product/developer-facing docs collateral, not maintainer/ops documentation.

## 2. Cadence and Authorship

- **Cadence:** RELEASE runs **repeatedly, once per shipped version** — a living, versioned
  artifact updated alongside Maintenance batches/releases, not a one-time v1.0 deliverable.
- **Content authorship:** RELEASE authors **substantially new content** — tutorials,
  conceptual guides, walkthroughs — that doesn't exist elsewhere in the project. It is not a
  mechanical curation/reformatting pass over existing README/Spec content, though it draws
  on that material as raw input (§11).
- **Ownership split:** **Claude does the bulk of this work** — authorship, information
  architecture, tone, tutorial design. Jules handles mechanical site-generation, build
  pipelines, and rustdoc/doctest extraction (§8's Reference-Sync, §9's handoff) — never
  original authorship.

## 3. Toolchain

**mdBook is the single mandated default toolchain for every project category** — no
per-category split. This was confirmed against real-world precedent: even UI/interactivity-
heavy Rust projects (Leptos's own book) use plain mdBook, not a heavier framework.

- **Versioning UX (publish-time convention, not a plugin):** one independent mdBook build per
  published version, published to a version-named path (mirroring Rust's own
  `stable`/`beta`/`nightly` book builds), with a small version-switcher snippet in the
  sidebar. See §7 for exactly which releases get a new snapshot.
- **Live interactive demos:** a separately Trunk-built WASM artifact, iframe-embedded into
  the relevant page (`<iframe src="wasm/index.html" width="..." height="...">`) — no mdBook
  plugin, no Rust code changes, isolated by construction so it can never break the book's
  own chrome/sidebar.
- **Narrow override escape hatch:** a different toolchain may be proposed and justified at
  Design time for a genuine edge case (e.g., large-scale i18n, full-text search needs beyond
  mdBook's built-in search) — an explicit, justified exception, never a standing per-project
  or per-category default choice.

## 4. Taxonomy

**Diátaxis** (Tutorials / How-To Guides / Reference / Explanation) is the default taxonomy
for every project's docs content.

- A quadrant genuinely inapplicable to a given project may be omitted (e.g., a narrow CLI
  tool may need no Explanation quadrant) — decided explicitly at Elicitation (§6.1), not
  silently skipped.
- A narrow, explicitly-justified override to a different structure entirely remains
  available, same shape as §3's toolchain override — proposed and justified, not a default.
- **Purity principle (governs Draft-pass ordering, §6.4–6.6):** a Tutorial or How-To Guide
  must not digress into conceptual "why" content, and Explanation must not carry
  step-by-step instruction — mixing types is the anti-pattern Diátaxis exists to prevent.

## 5. Directory Structure

RELEASE's output lives under a dedicated top-level `web/` directory — never inside the
project-root `dev/` tree (reserved for grail-generated process docs: `dev/spec/`,
`dev/plan/`, `dev/maintenance/`, `dev/release/`, `dev/plan/[projectname]_dev_risks.md` — unrelated to
RELEASE's own published output) — and never using mdBook's own default `src/` name at
project root (would collide with the project's actual Rust source).

```
web/
├── site/     # the website itself — home/landing, docs (mdBook), blog, community, ...
│   ├── home/       # landing page — folded into default scope (§6.1's checklist), not a rare override
│   ├── docs/       # the mdBook project itself
│   │   ├── book.toml
│   │   ├── src/
│   │   └── book/   # build output (gitignored, or published per §7's versioning)
│   ├── blog/
│   ├── community/
│   ├── ...         # playground/, showcase/, etc. — only the ones this project's Elicitation step chose
│   └── [reference surfaces — see below]
├── press/    # media kit: brand assets, fact sheet, press releases, contact
├── legal/    # privacy policy, ToS, security.txt/disclosure policy
└── social/   # social media content/templates
```

**Generated reference surfaces** — a project's *externally-consumed, generated-from-source*
contracts (as distinct from the hand-authored `docs/` mdBook site) each get their own
sibling folder directly under `web/site/`, published with the identical `latest/` +
`vX.Y.Z/` versioning as `docs/` (§7), the same CI-deploy mechanism (§8), and the same
completeness gate as `docs/`'s own Step 7 (§6.7). Decided per-project at Design Step 5,
recorded in the Architecture Specification — not every project has every surface, and not
every project needs any:

| Folder | Surface | Typical generator |
|---|---|---|
| `web/site/api/` | Rustdoc — a Rust library API | `cargo doc` |
| `web/site/cli/` | CLI command/flag/subcommand reference | e.g. `clap`'s markdown-help output |
| `web/site/config/` | Config-file schema reference | e.g. `schemars`-derived JSON Schema |
| `web/site/http/` | HTTP/REST API reference (wire-level contract, distinct from `api/`'s Rust-internal rustdoc) | OpenAPI/Swagger spec |
| `web/site/proto/` | Protobuf interface reference (regardless of gRPC vs. plain protobuf transport) | `protoc-gen-doc` or equivalent |
| `web/site/dds/` | DDS topic/IDL reference | vendor/OMG DDS doc tooling |
| `web/site/mqtt/` | MQTT topic/message-schema reference | project-authored or generated from a schema |
| `web/site/wit/` | WIT (WASM Component Model) interface reference | `wit-bindgen`-adjacent tooling |
| `web/site/schema/` | Data-model/DB schema reference | ERD/schema-dump tooling |

**This list is illustrative, not a closed enum.** Any other externally-consumed contract a
project exposes gets the same treatment — versioned path, same publish gate — under a
project-chosen folder name; a future surface type not named above never requires a grail
change to support, only an update to this table for discoverability if it recurs across
projects. The mdBook `docs/` Reference quadrant links out to whichever of these exist for
this project (§6.7) rather than absorbing their content.

**`web/site/`'s own content checklist**, presented at Elicitation (§6.1) — the user selects
which of these this project wants, beyond the always-present Docs (mdBook) quadrant:

| Content type | Universality | Notes |
|---|---|---|
| Landing/Home page | Near-universal | Hero, value proposition, quick example, CTA into Docs/repo |
| Blog | Near-universal | Release announcements, technical deep-dives, roadmap updates |
| Community/Ecosystem page | Near-universal | Discord/forum/discussions links, contributor directory, related-crate ecosystem |
| Playground / Try-it-online | Situational | Reuses §3's Trunk-built-WASM-demo + iframe mechanism; strongest fit for library/SDK and web-client/server categories |
| Showcase/Gallery | Situational | "Built with X" — strongest fit for library/SDK and framework-shaped projects |
| Comparison/Benchmarks page | Situational | Where performance or feature comparison is a genuine selling point |
| Sponsors/Backers page | Situational | Only if the project has a funding/sponsorship dimension |
| Roadmap/Status page (public roadmap/RFC tracker) | Situational | More relevant for larger, community-facing projects |
| Changelog/Release-notes page | Situational | A public-facing rendering of `CHANGELOG.md` — mechanical regeneration, distinct from the authored Blog |

A public **uptime/status page** is explicitly **not** a `web/` category — if a project has
one, it is typically hosted externally (a third-party service, linked to from `site/`'s
nav/footer) and overlaps with `DEVELOPMENT.md`'s existing `PROD-008`/`PROD-009` operational
checklist items, not a RELEASE concern.


## 6. The Release Phase Process

RELEASE runs as a sequence of **Steps** — the same term used for Design's own numbered
gated divisions and Maintenance's M0–M4, deliberately, after explicit review (see the
framework's own naming conventions). Each Step runs in its own session, one Step at a time
(`AGENTS.md` §3.3's One Gate at a Time — no exception for RELEASE), with no assumed memory
of any prior session. **No formal name has been assigned to the individual sub-items a Step
may produce or check** (e.g., an individual glossary entry, an individual consistency
check) — this is a deliberate, open decision, not an oversight; do not invent one.

### 6.1. Step 1: Elicitation

**Process:** Establish scope (explicit in-scope/out-of-scope) and audience/personas for
this release's documentation content — distinct from the Architecture Spec's own product
personas (§2.1), since a docs reader's needed skill level often differs from a product
persona. Confirm which Diátaxis quadrants apply (§4) and which `web/site/` content types
(§5's checklist) this project wants this release.

**Seed material — do not start cold:**
- Every User Story tagged with a **Diátaxis Destination** field at Design time (Architecture
  Spec §2.2) — Tutorial / How-To Guide / neither / deferred-to-triage.
- Every Core Functional Requirement's optional **User-Facing Description** field
  (Architecture Spec §2.3) — a plain-language seed for Explanation/How-To prose.
- This release's `CHANGELOG.md` entries — an input signal for "what changed this release
  that needs a doc update," not source prose to copy.

**Output:** Scope statement, audience/persona list, quadrant/content-type selection for this
release — feeds directly into §6.3 (Spine).

### 6.2. Step 2: Scaffold

**Process:** Mechanical — `book.toml`, `SUMMARY.md`/chapter stubs, theme/CSS, branding
placement, preprocessor config (`mdbook-mermaid`, `mdbook-admonish`, etc.), publish-pipeline
setup. No elicitation or discussion needed; this Step is a strong candidate for Jules to
execute mechanically once Step 1's scope is known, though authored under Claude's direction
per §2's ownership split.

**Output:** A building, empty-content mdBook skeleton matching Step 1's chosen taxonomy and
content-type selection.

### 6.3. Step 3: Terminology/Concept Spine

**Process:** Substantive, discussion-driven — a settled glossary and narrative outline that
keeps Steps 4–6's parallel authors from drifting into inconsistent terminology. Seeded from
the Architecture Specification's own `§1.5 Definitions, Acronyms, Abbreviations` — already a
maintained, continuous artifact; this Step is not inventing a glossary from nothing, it is
confirming and adapting an already-current one for a docs-reader audience.

**Output:** A settled glossary/terminology reference and narrative outline, referenced by
Steps 4–6.

### 6.4–6.6. Steps 4–6: Tutorials / How-To Guides / Explanation (Draft Passes)

**Order-independent, not concurrent.** These three Steps may run in any order, or be
treated as parallelizable in the sense that no fixed dependency exists between them —
**never** as simultaneous/concurrent execution, which `AGENTS.md` §3.3's One Gate at a Time
mandate forbids regardless. Diátaxis's own purity principle (§4) is why no ordering
dependency exists: Tutorials and How-To Guides deliberately avoid carrying "why" content, so
they do not need Explanation's deeper prose to exist first — they need only the terminology
Step 3 already fixed.

- **Step 4 — Tutorials:** hands-on, learning-oriented walkthroughs for stories tagged
  Tutorial at Step 1.
- **Step 5 — How-To Guides:** task-oriented guides for an already-competent user, for
  stories tagged How-To Guide at Step 1.
- **Step 6 — Explanation:** conceptual, understanding-oriented prose — the deepest
  elaboration of Step 3's Spine, consumed last by readers but not authored last by
  necessity.

**No new Session Unit concept required** for these three Steps — each is simply an ordinary
gated Step, run in whichever sequence the user chooses.

### 6.7. Step 7: Reference-Sync

**Gating (this is the load-bearing constraint on this whole Step — not a judgment call):**
Reference-Sync does not begin until the batch/phase this release documents has reached
`MAINTENANCE.md` §11's own Release Checklist state — every item's Checklist Task Group
Verified, SemVer bump re-confirmed — or, for a project's first release, `DEVELOPMENT.md`'s
own **Final Verification** state. This is the same signal already computed elsewhere in the
framework; RELEASE does not invent an independent readiness signal.

**Process:** Because of the continuous-documentation discipline (a separate, standalone
proposal — see `grail_continuous_documentation_proposal_v1.md`, pending its own
implementation session), rustdoc comments and `# Examples` doctests should already exist,
current and CI-verified, on every `pub` item touched since the last release, by the time
this gate fires. Reference-Sync is therefore **generation and publishing, not
authorship or extraction**: for every reference surface this project declared at Design
Step 5 (§5's table — `api/`, `cli/`, `proto/`, etc.), Jules runs that surface's generator,
publishes the output to its own versioned path (§7) via the same CI-deploy mechanism as
`docs/` (§8), and the mdBook Reference quadrant's own pages become thin **link-out** pages
pointing at each published surface's matching version — never a copy of that surface's
content pasted into a hand-maintained mdBook page, since a pasted copy silently goes stale
the moment the underlying source changes and nobody remembers to re-extract it. **Each
`docs/vX.Y.Z/` snapshot links to its own matching `vX.Y.Z/` version of every reference
surface it references — never to that surface's `latest/`** — so an older, pinned docs
snapshot never accidentally shows a reader newer API surface than what its own prose
describes. `docs/latest/` is the sole exception: since "latest" and "the current release's
own version" are the same thing by definition, it links to each surface's `latest/` too.
If a project's upstream continuous-documentation discipline has not yet been implemented,
this Step reverts to genuine authorship for the reference content and should be flagged as
a scope/timeline risk, not silently absorbed.

**Same completeness gate as `docs/`, applied per-surface.** Any declared reference surface
whose generated content is missing or stale for a `pub`/exposed item touched since the
last release blocks *that surface's* publish step — the risk (a Reference-quadrant page
confidently linking to an incomplete target) is identical regardless of which surface is
gated, so the gate is not weakened for any of them.

**Output:** A complete, as-built-accurate Reference quadrant of link-out pages, plus every
declared reference surface published at its own matching versioned path.

### 6.8. Step 8: Finalize

**Process:** Technical-accuracy reconciliation — check Steps 4–6's drafted prose against
Step 7's now-stable Reference content, resolving any drift between what was drafted
(possibly before the underlying surface fully stabilized) and what actually shipped. This is
a distinct activity from Step 9's editorial/terminology check — best practice explicitly
separates technical-accuracy review from editorial review, run in sequence, not merged into
one pass.

**Output:** Draft content reconciled against the finalized Reference; no known technical
inaccuracies remaining.

### 6.9. Step 9: Consistency Audit

**Process:** A distinct, lighter-weight final step than Design's adversarial Steps 7/9 —
editorial/terminology-focused, not a formal requirement-traceability audit. Checks: every
Spine (§6.3) term used consistently across all quadrants; no orphaned or drifted
terminology; style-guide adherence. This exists specifically to catch **ground-truth
drift** — independent authors (Steps 4–6) silently choosing conflicting anchors — the
named risk of any parallel/order-independent authoring process.

**Output:** A consistency-audited, ready-to-publish docs site — the input to §9's Jules
handoff.

## 7. Versioning UX

Doc versioning ties directly to `MAINTENANCE.md` §11's existing SemVer bump computation —
no independent trigger invented:

- **Patch-only release:** update the current/`latest` published docs in place — no new
  version-path snapshot. Nothing user-facing changed.
- **Minor release:** create a new version-path snapshot (§3's mechanism).
- **Major release:** new version-path snapshot **plus** required migration-guidance content
  published in the docs site itself (not only `CHANGELOG.md`) — reusing `MAINTENANCE.md`
  §11's existing MAJOR-only Release Checklist item.
- **Versioning is opt-out at the project level** — a `latest`-only, no-version-history model
  (VS Code's own real-world convention) is a legitimate Design-time choice, not a deviation
  requiring justification.
- **Every declared generated reference surface (§5's table) gets the identical treatment,
  in the same publish step, at the same tier** — a minor/major release snapshots `docs/` and
  every reference surface together, since they describe the same commit's state and must
  never drift apart in which version they represent; a patch release updates `latest/` for
  all of them in place, same as `docs/`.

## 8. Hosting and Deployment

RELEASE's scope **includes deployment**, not just "the site builds locally" — a docs site
nobody can reach isn't done.

- **Default: GitHub Pages**, with a custom subdomain via `CNAME` if wanted (this alone does
  not require leaving GitHub Pages). GitHub Pages remains a legitimate default **regardless
  of a project's commercial-licensing status** — its restriction targets the site itself
  functioning as an e-commerce/SaaS transaction surface, not a project merely having a
  commercial-licensing option (e.g., grail's own PolyForm Noncommercial + paid-license
  model). The one genuine trigger to reconsider is the docs site itself needing to process
  a transaction or gate content behind a paywall — not the underlying software's license.
- **Deploy mechanism: `peaceiris/actions-gh-pages`**, invoked once per generated surface
  (`docs/` plus every declared reference surface) from within `.github/workflows/release_docs.yml`
  (`agents/CI.md` §5.2, v0.12.8) — none of these are
  ever git-committed to the branch the project edits; every one is a build artifact,
  generated fresh and pushed straight to the Pages-serving target by CI, the same way
  `docs/`'s own mdBook output already is. This applies uniformly regardless of which
  surfaces a given project declares.
- **Escalation to full hosting** (Cloudflare Pages/Netlify/a real subdomain deployment),
  triggered by an explicit, checkable condition, not a vague judgment call: media/repo size
  approaching GitHub Pages' ~1 GB ceiling, expected traffic approaching its ~100 GB/month
  soft bandwidth limit, or an explicit product decision that the docs site is part of a
  broader branded web presence beyond project documentation.

## 9. Relationship to Existing Artifacts

- **README stays structurally as-is** (`README_template.md`'s existing sections are not
  reduced to a stub) — it keeps its "when to use this, get it running" role. It gains **one
  new, explicit element**: a prominent link to RELEASE's docs site for anything deeper
  (tutorials, conceptual guides, full reference).
- **Architecture Specification is source material only** — never modified or superseded.
  The actual pull mechanism is the User Story/Requirement tagging described in §6.1 — not
  re-derived by RELEASE itself.
- **`CHANGELOG.md` entries are an input signal, not source prose** — feed Step 1's
  Elicitation as a mechanical "what needs a doc update this release" checklist.
- **No wholesale copying** into RELEASE's authored prose from any of these three artifacts —
  each is a seed/signal, consistent with §2's authorship mandate.

## 10. Jules Handoff

**Single handoff point, after Step 9 (Consistency Audit) closes** — mirroring
`MAINTENANCE.md`'s existing Claude-drafts/Jules-executes, file-relay pattern
(`AGENT_TOOL_POLICY.md` — no direct API integration). Claude produces a **release prompt
file** (`release_prompt_template.md` → `[projectname]_release_v[N.NN.NN]_prompt.md`)
directing Jules to run the mechanical pipeline against the now-finalized, audited content:

- `mdbook build`
- `mdbook test` (validates every Tutorial/How-To/Reference code example actually compiles
  and runs, the same mechanism `cargo test --doc` uses for rustdoc examples)
- Link-check / lint pass
- Publish to the version path (§7) and update the version-switcher snippet
- Deploy per §8's hosting decision

Jules touches build/deploy/extraction machinery only — **never authored prose.**

## 11. Escalation

**(v0.12.11)** Per `AGENTS.md` §2.4.1's Autonomous Continuation mandate, a Release session
does not pause for permission, confirmation, or affirmation to continue authorized work —
the only stop is a genuine Escalation Trigger below or a Step's own Entry Criteria not yet
satisfied.

Same discipline as `MAINTENANCE.md` §12 and `development_plan_template.md` §13 — stop,
summarize, and wait on: a genuinely ambiguous scope decision at Step 1; discovery, at any
Step, that the continuous-documentation discipline (§6.7) was not actually followed
upstream, meaning Reference-Sync cannot be pure extraction as assumed; a Step 9 Consistency
Audit finding that implies a Step 1 scope or Step 3 terminology decision needs to be
revisited (a backtrack, not a same-session silent fix); or any standing safety/compliance
concern (e.g., a §8 hosting question genuinely bordering on GitHub Pages' commercial-use
restriction, per §8's note — not a determination this guide makes unilaterally).

---

## Appendix — Template Files

| File | Produced at | Purpose |
|---|---|---|
| `release_plan_template.md` → `[projectname]_release_v[N.NN.NN]_plan.md` | Start of a release's RELEASE work | The 9-Step plan for this specific release, Entry/Exit criteria per Step |
| `release_checklist_template.md` → `[projectname]_release_v[N.NN.NN]_checklist.md` | Alongside the plan | Companion checklist, edited in place across the release's RELEASE work |
| `release_prompt_template.md` → `[projectname]_release_v[N.NN.NN]_prompt.md` | End of Step 9 | The Jules handoff prompt (§10) |

**Naming note:** the `release` tag precedes the version (`_release_v[N.NN.NN]_`), mirroring
`maintenance_batch_template.md`'s own `[projectname]_[type]_v[N.NN.NN].md` pattern — this
avoids an exact-filename collision with `MAINTENANCE.md`'s own
`[projectname]_v[N.NN.NN]_checklist.md`/`_prompt.md` for the same version.

**Versioning note (v0.12.8):** `v[N.NN.NN]` above — and `dev/release/v[N.NN.NN]/`'s own
folder — is `v0.0.1` or any higher version, entirely user-determined at the point this
release's RELEASE work opens; it is never assumed to be `v0.1.0` or any other specific
value. Unlike Development Phase's `test/` output (fixed at `v0.0.1` for that phase's whole
duration, `agents/DEVELOPMENT.md`), a release's version is a real project SemVer value set
by whoever opens the RELEASE work, not a framework default — same treatment as
`MAINTENANCE.md` §8's batch versioning.

**Publishing note (v0.12.8):** the mdBook build/test/publish pipeline (§6.7's Step
7/`release_prompt_template.md` steps 2–6) now runs as its own named, manually-triggered
workflow, `.github/workflows/release_docs.yml` (`agents/CI.md` §5.2) — not "the same CI
job" as any prior phrasing implied. `ci.yml` itself never runs this pipeline.
