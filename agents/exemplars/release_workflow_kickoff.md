# RELEASE Workflow — Research & Design Kickoff Prompt

> Give this prompt, verbatim, to a **new Claude session** to begin designing grail's
> fourth peer phase: **RELEASE** — sitting alongside DESIGN, DEVELOPMENT, and MAINTENANCE,
> governing polished, user/developer-facing documentation and docs-site collateral (the
> tier of VS Code's docs, ffmpeg.org, Dioxus/Angular's docs sites — not README.md or the
> Architecture Specification, which already exist and stay as-is).
>
> **Trigger phrase for this new session to actually generate files:** `GENERATE RELEASE -
> v0.11.0`. Until that exact phrase is given, the new session should only research,
> discuss, and iterate with the user — exactly the same discipline as the v0.10.1 batch
> that preceded this (discuss extensively, batch every decision, generate nothing until
> told).

---

## Prompt Text (paste this to the new session)

You are starting design work on a new phase for the **grail** framework — a
Design/Development/Maintenance workflow for Rust projects, governed by `AGENTS.md`,
`CLAUDE.md`, `DESIGN.md`, `DEVELOPMENT.md`, `MAINTENANCE.md`, and their companion
exemplar/template files. This is a **fourth peer phase, "RELEASE"**, sitting alongside
the existing three. Read `AGENTS.md`, `CLAUDE.md`, `DESIGN.md`, `DEVELOPMENT.md`, and
`MAINTENANCE.md` in full before starting, if not already familiar with this framework.

### What RELEASE is, and what it explicitly is not

RELEASE governs **polished, user-facing and developer-facing documentation
collateral** — the tier of documentation exemplified by:
- **https://code.visualstudio.com/docs** (UI product — full docs site, tutorials +
  guides + reference)
- **https://ffmpeg.org/ffmpeg.html** (CLI product — dense, authoritative single-page
  reference style)
- **https://dioxuslabs.com/learn/0.7/** and **https://angular.dev/overview**
  (framework/developer-facing products — heavily tutorial-driven, versioned docs sites,
  concept + guide + API reference layers)

This is **explicitly distinct from and does not replace**:
- `README.md` (`README_template.md`) — stays exactly as it is, a functional project-root
  file (Quick Start, badges, license). RELEASE's output is *additional*, richer
  collateral, not a replacement for README.
- The Architecture Specification — stays as the technical, requirements-traceable
  source of truth for Design/Development/Maintenance's own internal use. RELEASE may
  *draw on* it as source material but doesn't modify or supersede it.
- `MAINTENANCE.md`'s own process documentation — a prior discussion in this framework's
  design explicitly concluded that "maintenance documentation" (how a maintainer keeps
  a *specific* project running) does not need its own checklist item and is out of scope
  for RELEASE; RELEASE is about product/developer-facing docs collateral, not
  maintainer/ops documentation.

### Decisions already made — do not re-litigate these, build on them

1. **Cadence:** RELEASE runs **repeatedly, once per shipped version** — not a one-time
   deliverable at v1.0. Docs collateral is a living, versioned artifact, updated
   alongside Maintenance batches/releases, the same way VS Code's and Angular's docs
   sites track their own product's version history.
2. **Content authorship:** RELEASE is expected to **author substantially new content**
   (tutorials, conceptual guides, walkthroughs) that doesn't exist anywhere else in the
   project yet — this is not a mechanical curation/reformatting pass over existing
   README/Spec content, though it may draw on that material as raw input.
3. **Ownership split:** **Claude does the bulk of this work** — authorship, information
   architecture, tone, tutorial design. Jules and/or other tools may be used for
   mechanical site-generation, build pipelines, and possibly graphics/asset generation,
   but the primary authorship is Claude's, unlike Maintenance's Jules-heavy execution
   split.
4. **Toolchain, taxonomy, and hosting are explicitly open** — not yet decided. This is
   your first job (see Research Mandate below), not something to assume from the
   examples above.

### Research Mandate — this is the primary work of this session's early turns

**Do extensive web research — competitive/best-in-class analysis — before proposing any
structure.** Do not rely on your own training-data impressions of what "good docs" look
like; actually search and study current, live examples. Scale your search effort to the
number of categories below (per `web_search`'s own scaling guidance — this easily
warrants 15–25+ searches/fetches across categories, not a handful).

**Research per product category** — use grail's own existing project-type taxonomy
(`MAINTENANCE.md` §7) as your category list, since these are the shapes grail already
supports and RELEASE must serve all of them:

| Project type | Research this category's best-in-class docs |
|---|---|
| Web client/server | e.g. Stripe API docs, Supabase docs |
| CLI | e.g. ffmpeg.org, ripgrep/fd README+docs, `gh` CLI docs |
| Embedded/IoT | e.g. Espressif ESP-IDF docs, Zephyr project docs |
| Multi-container services | e.g. Docker Compose docs, Kubernetes docs structure |
| Headless server + remote client | e.g. Postgres docs, Redis docs |
| Library/SDK | e.g. Dioxus, Angular, Rust's own `docs.rs`/mdBook-based crate docs, Serde |
| Mobile/desktop/browser extension | e.g. VS Code docs, Tauri docs, a well-regarded
  browser-extension SDK's docs |
| Batch/data pipeline | e.g. Apache Airflow docs, dbt docs |
| Serverless/on-prem/vendor-installed | e.g. a representative Terraform-provider or
  vendor-installed product's docs |

For each category, identify: the **information architecture** used (what top-level
sections exist — Getting Started, Tutorials, Guides, Reference, Concepts, etc. — and how
they differ by category), the **toolchain** if discoverable (Docusaurus, mdBook,
VitePress, a custom framework, plain static site, single long page, etc.), and what makes
that specific example genuinely good versus merely present (navigability, search,
versioning UX, code-sample quality, progressive disclosure from beginner to advanced).

**Rust-specific research, since grail is Rust-first:** investigate `mdBook` (Rust-native,
used for the Rust Book itself and many Rust project docs) as a strong default toolchain
candidate — is it sufficient across all these categories, or does it fall short for some
(e.g. a UI-heavy product needing richer interactive components)? Research how Rust
projects in each category above actually publish their docs today, not just
general-purpose examples.

### Design Questions to Resolve With the User (after research, before drafting RELEASE.md)

Work through these with the user, one at a time or in small batches, the same
discussion-then-decide discipline used for the rest of this session's prior work
(reference: this framework's Maintenance-phase redesign was conducted entirely through
iterative discussion before any file was generated). Do not generate any RELEASE.md,
templates, or prompt files until the user explicitly says **"GENERATE RELEASE -
v0.11.0."**

1. **Toolchain:** single mandated toolchain (e.g. mdBook everywhere) vs. per-category or
   per-project Design-time choice (like WASM/Playwright/infra services already are).
2. **Taxonomy:** a fixed category set (Getting Started / Tutorials / Guides / Reference /
   Concepts, or your research-derived equivalent) applied uniformly, vs. a category set
   that varies meaningfully by project type from the table above.
3. **Process shape:** does RELEASE get its own M-step process analogous to Maintenance's
   M0–M4 (e.g. an elicitation step for "what should the tutorials actually teach," an
   information-architecture-drafting step, an authoring step, a review step)? Propose a
   concrete shape from your research rather than assuming Maintenance's shape transfers
   directly — RELEASE's work (long-form authorship) is a very different kind of task than
   Maintenance's (bug/feature classification and remediation).
4. **Versioning UX:** does every project need versioned doc paths (`/learn/0.7/` style),
   or only projects with a track record of breaking changes across versions? Should this
   tie to the project's own SemVer bump (from `MAINTENANCE.md` §11) or run independently?
5. **Hosting/deployment scope:** does RELEASE.md's scope end at "the docs site exists and
   builds locally," or does it include deployment (GitHub Pages, a docs subdomain, etc.)?
6. **Relationship to existing artifacts:** exactly what, if anything, gets pulled from
   README/Architecture Spec/CHANGELOG as source material vs. authored fresh — and how do
   you keep the two from drifting apart once RELEASE's docs site exists alongside them
   (e.g. does README become a stub that links out, or do the two remain fully
   independent, accepting some duplication)?
7. **File/template naming and structure** for `RELEASE.md` itself, its companion
   exemplar/template files (mirroring how `MAINTENANCE.md` has
   `maintenance_batch_template.md`/`maintenance_checklist_template.md`/
   `maintenance_prompt_template.md`), and where generated docs-site content physically
   lives in a project's repository (a new top-level `docs-site/`? reusing `docs/`
   alongside `dev_risks.md`? something else — note `docs/` is already used for
   Development-Phase risk logs and the Productization retrofit's extract files, so
   confirm there's no collision).

### Working discipline for this session

- Discuss and batch every decision, the same way the rest of this framework's recent
  changes were made — no premature drafting.
- Track a running numbered decision list as you go (the same convention used
  throughout this framework's design conversations), so the user can review the full
  state of decisions at any point without you re-deriving it.
- **Generate no files — RELEASE.md, templates, prompts, or anything else — until the
  user types exactly `GENERATE RELEASE - v0.11.0`.**
