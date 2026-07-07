# CHANGELOG

> **For human reference only. Agents do not need to read this file.**
>
> This file consolidates the full version history extracted from all grail instruction
> files at v0.6.3. Each section corresponds to one source file. Prior history entries
> are preserved verbatim. The v0.6.3 entry in each section describes what changed in
> that batch; individual files now carry only a single pointer: "See CHANGELOG.md."

---

## AGENTS.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | — | Claude | Baseline: five-phase lifecycle overview; Autonomous/Advisory operating modes; Core Mandates (work/filesystem integrity, protocol adherence, quality/completeness, behavioral caution and simplicity, naming conventions); User Interaction (formal approval protocol, CHAT: protocol, synchronous gated execution with unconditional one-item-at-a-time elicitation). No version number or revision history prior to this entry; baseline reconstructed for record-keeping. |
| 1.1.0 | 2026-06-20 | Claude | Added explicit Rust-only scope statement. Added Stable Identifier Assignment and Explicit Backtracking to §2.1. Made §3.3 Iterative Elicitation mode-conditional. Tightened §2.5 naming-convention rationale. |
| 0.5.0 | 2026-06-24 | Claude | Version aligned to CLAUDE.md v0.5.0 restructure. Restructured §3.3 into clean sub-points. Added §1.5.3 acknowledging Autonomy Toggle. Version number jump from 1.1.0 to 0.5.0 to align with CLAUDE.md per explicit user instruction. |
| 0.6.1 | 2026-06-24 | Claude | Catch-up: §1.5.3 renamed to "Autonomy Toggle and Named Presets." Added "No cross-session memory" note to §1.5.2. Added handoff note manifest requirement. Added file versioning note. |
| 0.6.2 | 2026-06-24 | Claude | Added "Re-Verify Before Extending a Rule by Analogy" to §2.4.1. Added delivered-files versioning acknowledgment to §1.5.2. |
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch.** Reduced lifecycle to Design + Development only (Verification/Release/Maintenance phases removed). Added §2.6 Missing-Tools Mandate (self-install + append to setup_env.sh/bat, escalation still fires). Added §2.7 No-Copy-Checklist Mandate. Added §2.8 One-Phase-Per-Session Mandate. CHAT: protocol moved to its own §3.2.1 with explicit cross-reference from §3.2. All V/R/M phase references removed. §1 lifecycle updated to two phases. Paths corrected to reflect repo layout (CLAUDE.md + AGENTS.md in root; agents/ and agents/exemplars/). CHANGELOG.md created; Appendix R replaced with pointer. |
| 0.7.0 | 2026-07-02 | Claude | **v0.7.0 batch.** §1.5.2 Advisory Mode: removed dangling reference to never-created `agents/ADVISORY_MODE_GUIDE.md`; replaced with a generic pointer to the project's own working-arrangement file (e.g. `CLAUDE.md`), which is read unconditionally regardless of any pointer here. No other content change — this file remains lifecycle-agnostic to the CLAUDE.md-side Step 2/Step 9/Asset Manifest changes landing in the same batch. |
| 0.7.1 | 2026-07-03 | Claude | §2.7 extended: beyond the existing no-copy/no-rename rule, now states explicitly that the Development Checklist is the *only* documentation/planning file a Development Phase session may edit at all (every other doc file read-only, an inconsistency in one is an Escalation Trigger, never a same-session fix), and that edits within the Checklist are scoped to the session's own current Phase only (never another phase's marks, never structural text). Operationalized in `agents/DEVELOPMENT.md` §4. |
| 0.8.5 | 2026-07-04 | Claude | **v0.8.5 batch.** §2.6 rewritten: a missing tool/prerequisite that installs cleanly still proceeds automatically; any install failure or resulting conflict now stops the session per the Development Phase escalation model rather than escalating-and-continuing. §3.1 tightened: exact, all-uppercase `APPROVED` only — "Continue"/"Proceed"/"Go ahead" removed as substitutes, for a Step Gate, Minor Change, Major Change, and handoff-package preparation alike; added the distinct `BACKTRACK APPROVED` / `SURGICAL FIX OVERRIDE` tokens (`CLAUDE.md` §3.6.1). §2.1's Explicit Backtracking mandate gained the Surgical Fix Override cross-reference. |
| 0.8.6 | 2026-07-06 | Claude | **v0.8.6 batch.** §2.1 Mandatory Artifact Preservation rewritten: removed `target/` from the commit mandate; now explicitly distinguishes non-reproducible evidence artifacts (committed — `test_outs/`, screenshots, coverage reports) from reproducible build outputs (gitignored — `target/`, `dist/`); cross-references `development_plan_template.md` §3 for the required `.gitignore` entries. |
| 0.8.7 | 2026-07-06 | Claude | **v0.8.7 batch (part 1).** §2.2 gained two new mandates: **Selective Reading Mandate** (whole-file reads of large documents explicitly forbidden; all documentation reads must use targeted extraction via `sed`/`grep`/`awk`; cross-references `dev_prompt_template.md` reference table) and **No Broad Repository Scan Mandate** (speculative directory/file enumeration at session start or any other point explicitly forbidden; navigation to specific known paths only). |
| 0.8.7 | 2026-07-07 | Claude | **v0.8.7 batch (part 2) — work-preservation overhaul.** §2.1 rewritten: task-level `submit` cadence (via new `agents/AGENT_TOOL_POLICY.md`) is now the concrete preservation mechanism, replacing implicit phase-only checkpointing — bounds crash blast-radius to one task/sub-task. New WIP Checkpoint mechanism: a `submit` not claiming DoD, exempt from Error-Free-Builds, always bypassing code review, prefixed `[WIP-CHECKPOINT]`, mandatorily describing what was attempted/confirmed/incomplete so a resuming session can pick up in place. §2.2 gained a **Code Review Policy: BYPASSED** bullet (flip-by-deletion) and a cross-reference to the new `agents/AGENT_TOOL_POLICY.md`. §2.8 reworked: One-Phase-Per-Session generalized to a per-phase-declared **Session Unit** (`Phase`/`Task`/`Code+Verify`), changeable only via Plan-Change Escalation. |

---

## CLAUDE.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.5.0 | 2026-06-24 | Claude | Full restructure for clarity and token efficiency — no intended behavior change. Consolidated propose-with-flagged-assumptions principle into §3.1; Grouped Closing Protocol into §3.2; selectable-options convention into §3.3; reduced 8-step description to per-step deltas. Added §3.7 Tailored Mode. Replaced relative cross-references with stable named anchors. Squashed prior 0.1.0–0.4.0 history. |
| 0.6.0 | 2026-06-24 | Claude | Real new behavior. Step 1 always-on high-rigor research. §3.3 research trigger independent of decision importance. §4 step-number filename convention for intermediate/handoff artifacts. §3.7 redesigned as general per-step toggle with two presets. New §3.8 visual input. New §3.9 no unilateral technical sufficiency. New §3.10 "not enough—another pass" mechanism. Step 8 Phase Sizing complexity-scoring formula. |
| 0.6.1 | 2026-06-24 | Claude | Clarified session-per-step reality. New §3.11 on file delivery for session boundaries. Handoff notes must enumerate every file. Input verification against handoff manifest. |
| 0.6.2 | 2026-06-24 | Claude | Selectable-options three-layer presentation precondition. Four research trigger categories. Grouped Closing Protocol bulk-reply loophole closed. §3.8 visual input as discovery source. §3.11 pre-handoff staleness check and version-suffix scheme. Development Checklist no version suffix. New Step 8 kickoff prompt deliverable. Phase Summary §11.3 and §13.1 escalation formalized. |
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch.** §1 updated: session-per-step is now a hard rule stated explicitly. New §3.12 Step 7 Backtrack Workflow (findings always require reopening originating steps; all fixes in one backtrack session, earliest-to-latest, ending with package for new Step 7 session). Handoff note naming standardized: `[projectname]_design_handoff_stepN_passX.md`. §3.11 pre-handoff staleness check extended to verify file naming convention compliance. §4 file naming updated to new `[projectname]_artifact_NN_topic_vN.md` pattern (projectname-first, 00-based sequence). Phase Summary naming updated to `[projectname]_phaseN_summary.md`. V/R/M phase references removed. CHAT: protocol cross-referenced. CHANGELOG.md created; Appendix replaced with pointer. |
| 0.7.0 | 2026-07-02 | Claude | **v0.7.0 batch — major restructure.** Step 2 Pass A/B split folded into a single session (no more `passA`/`passB` handoff notes). Architecture Specification regrouped from 4 topic-based files to **8 files, one per originating Design Step 1–6** (§4.1), fixing both unbounded file size at scale (50+ stories/100–200+ requirements) and shared-version-bump churn across unrelated content. New §4.3 makes per-file independent, session-end-only version bumping explicit, corrects a prior example that implied a single shared `v[N]` across the file set, and explicitly extends "touched this session" to files outside the current step (backtrack/revisit cases) after observed cases of that bump being skipped. New §3.13 End-of-Step Open-Items Review: every gate now re-surfaces the full cumulative Deferred/Remaining-Ambiguous register from all prior steps, not just the current step's own batch. New §2.2/§3.8 Visual & Media Asset Input mechanism: HTML presumptively authoritative for structure/behavior, images treated as a design-token source (branding/iconography/typography/palette/tiles), audio/video tracked-only; new Asset Manifest record with immutable filenames and fixed `assets/{html,images,audio,video}/` repository paths shared verbatim between Design and Development. §3.11 handoff notes now require an explicit `## Next Action: Start Step N[, Pass X].` directive as the first line, after observed sessions reporting no explicit ask and offering the obvious next step as a question instead of simply starting it. §3.4 Step 8 row references the new Frontend Targeted Interleaving phase-sequencing principle (full mechanism in `development_plan_template.md`/`architecture_specification_template.md`). Stale `_01_overview`/"Concept Mapping"/"Requirements List" examples throughout §3.11 corrected to match the new per-Step file identity. |
| 0.7.1 | 2026-07-03 | Claude | **Verbosity pass — no intended behavior change.** Rewrote prose throughout for concision (repeated emphasis/rationale collapsed, redundant restatements removed); all mandates, tables, ID schemes, filenames, and gate conditions preserved verbatim in substance. ~16% smaller by size. |
| 0.8.0 | 2026-07-03 | Claude | §3.8 gains an explicit exception to §3.11's file-delivery rule: user-supplied UI/media assets are never re-packaged into a handoff note — only referenced by filename; a step needing an asset requests the current version directly from the user; Spec/Plan content states the asset "will be available in `assets/...` at Development Phase" rather than assuming the Design-time copy is final. Asset Manifest table itself still carries forward normally. §3.11 cross-references this exception. `[projectname]_dev_prompt.md` renamed to `[projectname]_dev_agent_prompt.md` throughout (source template also renamed, see its own CHANGELOG section). |
| 0.8.5 | 2026-07-04 | Claude | **v0.8.5 batch — major restructure.** Removed the Autonomy Toggle/Tailored/Full-Autonomous presets and the Grouped Closing Protocol/propose-with-flagged-assumptions/Selectable-Options mechanism entirely (former §3.1-3.3, §3.7). Replaced by one fixed procedure for every step but 7 and 9 (new §3.1-3.2): RCD (Research, Competitive research, Draft) generates the core batch research-backed and in full; bulk-accept closes the great majority; RATS (Research, Analysis, Table, Selectable-options) resolves every residual item to exactly one of Resolved / Deferred-to-a-specific-step / Future Feature / Rejected. Step 2 always runs in full, no skip path. New §3.6.1 Surgical Fix Override (`SURGICAL FIX OVERRIDE` vs. `BACKTRACK APPROVED`), a compressed single-session alternative to a full backtrack, still mandatorily ending in a fresh audit session when triggered by a Step 7/9 finding. §3.10 (formerly §3.11) File Delivery reworked: interim files use a lowercase-letter suffix (`v3a`, `v3b`), never version-bumped or packaged until exact `APPROVED`. §3.12 (formerly §3.13) Open Items Register updated for the four terminal outcomes. All sections from former §3.7 onward renumbered down by one. Step 8 now also drafts the project README. Step 7/9 both gain a terminal-RATS-outcome audit check. Renamed `dev_agent_prompt_template.md`/`[projectname]_dev_agent_prompt.md` back to `dev_prompt_template.md`/`[projectname]_dev_prompt.md` throughout. |
| 0.8.7 | 2026-07-07 | Claude | **v0.8.7 batch — Step 8 row.** Noted that per-task Design Refs and Submit Points, and per-phase Session Unit, are populated at Step 8 drafting time, not left as stubs; Phase Sizing formula note that a Code/Verify-split task counts as 2. |

---

## agents/DESIGN.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.0.01 | 2025-10-10 | Jules | Initial creation. |
| 0.0.02 | 2025-10-10 | Jules | Complete overhaul to enforce rigor. |
| 0.0.03 | 2025-10-12 | Jules | Integrated iterative 6-step workflow. |
| 0.2.00 | 2025-10-12 | Jules | DESIGN v2.0: 8-Step Gated Protocol. |
| 0.2.01 | 2025-10-13 | Jules | Explicit 'STOP and Ask' mandates. |
| 0.3.00 | 2026-06-19 | Claude | Generalized propose-with-flagged-assumptions into Steps 1, 3, 4, 5. Step 2 mode-conditional elicitation. Notation-judgment flagging in Step 6. Environment/configuration elicitation sub-step in Step 8. §2.1 Multi-File Packaging. |
| 0.3.01 | 2026-06-20 | Claude | Step 7 independence framing added. Rust-only scope statement in §1. Step 5 dependency-feasibility language tightened. |
| 0.5.0 | 2026-06-24 | Claude | Mechanical-vs-substantive principle consolidated into §5 opening. §5.9 Autonomy Toggle/Named Presets acknowledged. 18% shorter. |
| 0.6.0 | 2026-06-24 | Claude | Step 1 always-on high-rigor research. Phase Sizing Mandate updated with complexity-scoring formula. §5.9 pointer broadened. |
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch.** §5 opening: each step begins in its own new session stated explicitly. §5.7 Step 7 updated: clean report → proceed; any finding → trigger backtrack workflow (defined in CLAUDE.md §3.12). Step 7 audit now includes file-naming compliance check. Handoff note naming standard referenced throughout. §2.1 multi-file packaging updated to new filename pattern. V/R/M refs removed. CHANGELOG.md pointer added. |
| 0.7.0 | 2026-07-02 | Claude | **v0.7.0 batch.** Lifecycle extended from 8 to **9 steps**: new §5.9 Step 9 (Plan & Checklist Audit) — independent, adversarial audit of the Development Plan/Checklist against the finalized Architecture Specification and against `development_plan_template.md` §15's own DoD, mirroring Step 7's independence; findings classify as (A) Plan/Checklist-only (reopen Step 8 alone) or (B) Spec-originating (reopen the relevant Spec step and re-clear Step 7 before returning to Step 8). §6 Phase Completion Criteria now requires both Step 7 and Step 9 clean audits, not just artifact delivery. §5.2 Step 2 folds Pass A/B into one session. §5.1 Step 1 broadens asset solicitation to HTML/images/brand assets/audio/video, cross-referencing `CLAUDE.md` §3.8. §5.8 Step 8 adds the Frontend Targeted Interleaving phase-sequencing principle. §2 Goal section notes both audits as completion prerequisites. Autonomy toggle section renumbered to §5.10 and updated: Step 9 can never be autonomous, same rationale as Step 7. |
| 0.7.1 | 2026-07-03 | Claude | §5.1 Step 1 row's asset cross-reference to `CLAUDE.md` §3.8 clarified: assets are never packaged into a Design-Phase session handoff note, only referenced by filename — a later step needing one requests the current version from the user directly. `dev_agent_kickoff_prompt_template.md` references updated to renamed `dev_agent_prompt_template.md`. |
| 0.8.5 | 2026-07-04 | Claude | **v0.8.5 batch.** §5's shared-mechanism intro replaced with the fixed RCD/RATS procedure description. Removed §5.10 Autonomy Toggle entirely. Step 2 always runs in full. Step 7/9 gain the terminal-RATS-outcome audit check and a Surgical Fix Override cross-reference. Step 8 gains README drafting as an explicit deliverable. Template filename references updated to `dev_prompt_template.md`. |
| 0.8.7 | 2026-07-07 | Claude | **v0.8.7 batch.** §5.8 Step 8 gained: per-task Design Refs and Submit Points, and per-phase Session Unit, are populated at drafting time, not left as stubs. |

---

## agents/DEVELOPMENT.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.0.01 | 2025-10-10 | Jules | Initial creation and refactoring. |
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch.** §4 added: Missing-Tools Mandate (self-install + idempotent append to scripts/setup_env.sh and scripts/setup_env.bat; escalation trigger still fires). No-Copy-Checklist Mandate. One-Phase-Per-Session Mandate. §3 updated to reference `[projectname]_dev_prompt.md`. §5.2 README task added (Phase 0 scaffold + final-phase review). Docker/deploy guidance added (heavy services via docker-compose in deploy/; reference PREFERRED_SERVICES.md). "Verification Phase" wording replaced with "feature-complete." V/R/M refs removed. CHANGELOG.md pointer added. |
| 0.7.0 | 2026-07-03 | Claude | §4's No-Copy-Checklist Mandate extended into an explicit **exclusive-file-touch rule**: `[projectname]_dev_checklist.md` is the only documentation/planning file a development-agent session may edit — every other doc file (Architecture Spec, Plan, Kickoff/Dev-Agent Prompt, handoff notes, Phase Summaries) is read-only, with a stray inconsistency treated as an Escalation Trigger rather than patched in passing. Edits within the checklist are further scoped to box-flipping/Session-Log rows for the agent's **current Phase only** — never marks belonging to another phase, never structural text. `[projectname]_dev_prompt.md` renamed to `[projectname]_dev_agent_prompt.md` throughout. |
| 0.8.5 | 2026-07-04 | Claude | **v0.8.5 batch.** Missing-Tool/Escalation mandate rewritten to Stop-Summarize-Wait: a cleanly-installing missing tool is the sole case continuing without stopping; every other unresolvable issue now stops the entire session immediately — write Phase Summary, no PR, no partial continuation. §5.2.1 reworked: Design Step 8 drafts the README; Development's Phase 0 task is review/confirm/enhance. Reverted naming to `dev_prompt_template.md`/`[projectname]_dev_prompt.md`. |
| 0.8.7 | 2026-07-06 | Claude | **v0.8.7 batch (part 1).** §5.2 step 1 Session-Start Sequence rewritten to reflect lean reading model: reads checklist, phase summaries, current phase's plan section (§6.1 + §8), and protocols file upfront using targeted extraction; explicitly prohibits reading all Architecture Spec files or all Dev Plan files upfront; prohibits broad repository scan; Architecture Spec and remaining plan sections referenced on demand only via `dev_prompt_template.md` reference table. |
| 0.8.7 | 2026-07-07 | Claude | **v0.8.7 batch (part 2) — work-preservation overhaul.** §4 gained the WIP-Checkpoint exception to Error-Free Builds. New §5.1 Mandatory Code/Verify Split (a/b sub-tasks, derived from Verification Method) and Submit Point cadence (submit at every declared checkpoint, resumed via plain "Continue"/"Proceed"). §5.2 step 2 gained the explicit three-way task-state check (not-started / WIP-checkpointed-resume-in-place / complete) against declared Submit Points — the direct fix for a crash mid-build-test-debug losing sibling-task work. Step 7 narrowed to phase-level wrap-up only; step 8 generalized to "next Session Unit." Cross-referenced new `agents/AGENT_TOOL_POLICY.md`. |

---

## agents/AGENT_TOOL_POLICY.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.8.7 | 2026-07-07 | Claude | **New file.** Resurrects and updates the Allowed/Requires-Approval/Forbidden tool-tiering content from a prior, now-repurposed file (tool names updated to current platform vocabulary: `submit`, `replace_with_git_merge_diff`, etc.). Forbidden tier retains the two-step "propose → `ARE YOU SURE?`" confirmation ritual. Cross-references `AGENTS.md` §2.1 (reset/restore prohibition) and §2.2 (Code Review Policy) rather than duplicating either. States explicitly that post-`submit` session pause is resumed via plain "Continue"/"Proceed," distinct from `AGENTS.md` §3.1's `APPROVED` token. |

---

## agents/RUST_PREFERENCES.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1.0 | 2026-06-19 | Claude | Initial creation. Naming/keyword constraints, type-system/ownership constraints, WASM concurrency constraints. |
| 0.1.1 | 2026-06-19 | Claude | COOP/COEP blocking trigger. Ranked recommended-crates table. Step 5 summary updated. |
| 0.2.0 | 2026-06-20 | Claude | Crate table reordered to lead with gloo-worker as default. |
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch.** §0 added: Rust 2024 edition mandate; MSRV-at-Step-5 policy; dual-MSRV pattern for publishable library crates; pre-flight version sanity check. cargo-llvm-cov replaces cargo-tarpaulin as coverage tool. Reference to PREFERRED_TOOLS.md added. CHANGELOG.md pointer added. |
| 0.8.5 | 2026-07-04 | Claude | Added a cross-reference from the Coverage Tooling note to `agents/ESP32_ESPIDF_RUST_BUILD_GUIDE.md` for embedded ESP32/ESP-IDF components. |

---

## agents/PREFERRED_DEPENDENCIES.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch (initial versioned entry).** Rust 2024 edition mandate added. cargo-tarpaulin removed (replaced by cargo-llvm-cov in PREFERRED_TOOLS.md). trunk removed (moved to PREFERRED_TOOLS.md). webpack removed entirely. MSRV policy pointer added. CHANGELOG.md pointer added. |
| 0.8.5 | 2026-07-04 | Claude | Added `btleplug`, `rumqttc`, `aes-gcm`, `tokio-cron-scheduler`, `totp-rs`, `qrcode`, `image`, `wiremock`, `wasmtime`/`wasmtime-wasi`/`wit-bindgen`, `embedded-graphics`/`ssd1306`/`mipidsi`. Added a Dependencies Requiring Approval section for vendor/integration-specific crates seen in a prior engagement. Cross-referenced `agents/ESP32_ESPIDF_RUST_BUILD_GUIDE.md`. |

---

## agents/PREFERRED_TOOLS.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1.0 | 2026-06-29 | Claude | Initial creation. Covers: cargo-nextest, cargo-llvm-cov, cargo-deny, cargo-audit, cargo-watch, trunk, sqlx-cli, cargo-expand, wasm-pack, protoc. Includes rust-toolchain.toml template and pre-flight version check guidance. |
| 0.8.5 | 2026-07-04 | Claude | Merged in the Forbidden Tools section and non-fatal-tools guidance from `PREFERRED_TOOLS_proposed.md` v0.2.0. Added explicit Trunk `dist/` vs. project `assets/` disambiguation with a `Trunk.toml` pin example. Missing Tool Protocol updated to Stop-Summarize-Wait for anything beyond a clean install. Cross-referenced `agents/ESP32_ESPIDF_RUST_BUILD_GUIDE.md`. |

---

## agents/PREFERRED_SERVICES.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1.0 | 2026-06-29 | Claude | Initial creation. Approved services: PostgreSQL+SQLx, Neo4j, Qdrant, MinIO, Redis. Docker-compose preferred; configs in deploy/. Port conventions. Service snippet templates. HTTP-over-HTTPS preference for local dev. |
| 0.8.5 | 2026-07-04 | Claude | `rumqttc` approved in `PREFERRED_DEPENDENCIES.md` — removed the prior placeholder in the Mosquitto section. |

---

## agents/exemplars/architecture_specification_template.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1.00 | — | — | Initial creation: Solution Strategy, Security Architecture/STRIDE, Rust-Specific Conventions, promoted Technology Stack, ADRs, Risks & Technical Debt, Implementation Roadmap, Configuration Reference, Quality Attribute Scenarios, 9-criteria/Rust-smell self-check tables. |
| 0.1.01 | 2026-06-20 | Claude | Fixed stale filename reference. Updated traceability note. |
| 0.2.00 | 2026-06-20 | Claude | Added §6.5 CI/CD Pipeline. Added §10 Public API & Framework Consumer Contract. Renumbered §10 Appendices to §11. Added Migration Safety & Rollback Policy to §4.5. |
| 0.2.01 | 2026-06-20 | Claude | Added §2.6 Future Features (Deferred Scope). Tightened §2.5 language. Cross-reference §9.3 to §2.6. |
| 0.2.02 | 2026-06-20 | Claude | Added §4.15 and §4.16 Deferred Implementation Alternatives. |
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch.** Header note updated to reference new file path `agents/exemplars/`. All internal cross-references to companion files updated to correct paths. File naming pattern in header updated to `[projectname]_architecture_NN_topic_vN.md`. V/R/M phase refs removed. CHANGELOG.md pointer added. |
| 0.7.0 | 2026-07-02 | Claude | **v0.7.0 batch.** §4.13 gains an Asset Manifest subsection (filename, type, repo target path, authoritative/informative-for, provided-at-step) per `CLAUDE.md` §3.8, with the requirement that every logged asset is referenced substantively elsewhere in §4.13, not left as a manifest-only entry. §9.1 Sequencing Principles gains the Frontend Targeted Interleaving principle (UI screen/component Build Order steps placed alongside their real backend data dependency, never before it and never batched into a trailing frontend-only step). No change to the file-split header note — it already deferred to `CLAUDE.md` §4, which now specifies the 8-file one-per-Design-Step split without requiring this template's own text to change. |
| 0.7.1 | 2026-07-04 | Claude | **Verbosity pass — no intended behavior change.** Prose throughout condensed (repeated rationale/phrasing tightened); every section, table (columns and example rows), mandate, and cross-reference preserved verbatim in substance — header count and table-row count identical to the pre-pass file. §4.13's Asset Manifest note updated to state explicitly that asset *files* are never carried in Design-Phase handoff notes (`CLAUDE.md` §3.8/§3.11) — only referenced by filename; the manifest table itself is unaffected. ~16% smaller by size. |
| 0.8.5 | 2026-07-04 | Claude | **Major compression pass** (874 → 412 lines): schema-only, every table/ID-format/mandate preserved verbatim, rationale prose condensed. Added an ESP32/ESP-IDF cross-reference (§4.9). Pre-compression version retained by the user separately for human reference. |

---

## agents/exemplars/development_plan_template.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1.00 | — | — | Initial creation of complex/multi-phase variant. |
| 0.2.00 | — | — | Rust-only complex/multi-phase variant generalized. |
| 0.2.01 | 2026-06-20 | Claude | §0 project note updated; §15 DoD tightened. |
| 0.2.02 | 2026-06-24 | Claude | §11.3 Phase Summary formalized. §13.1 escalation path added. |
| 0.2.03 | 2026-06-24 | Claude | Phase Summary gained header block and Assumptions section. §13.1 split into (A) replanning and (B) re-architecting. |
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch.** §4 Missing-Tools: agent installs missing prerequisites idempotently and appends commands to scripts/setup_env.sh and scripts/setup_env.bat. §5 pre-flight version sanity check added. §6 One-Phase-Per-Session mandate in Phase Index. §8 README task template added. §11 Phase Summary naming updated to `[projectname]_phaseN_summary.md`. §12 No-Copy-Checklist mandate. Deploy/docker guidance in §5. File naming throughout updated to new pattern. V/R/M refs removed. CHANGELOG.md pointer added. |
| 0.7.0 | 2026-07-02 | Claude | **v0.7.0 batch.** §0 project note updated to cite the split Traceability location (`_04_test_strategy` + `_05_verified_traceability`, per `CLAUDE.md` §4.1) instead of a single §3; new Audit dependency note citing Design Step 9. §3 Project Folder Structure gains the `assets/{html,images,audio,video}/` convention — user-populated, filename-stable, immutable, cross-referenced against the Architecture Spec's Asset Manifest; missing asset file is now an explicit Escalation Trigger. §6 gains the Frontend Targeted Interleaving phase-sequencing mandate, cross-referencing the corresponding principle now in `architecture_specification_template.md` §9.1. |
| 0.7.1 | 2026-07-04 | Claude | **Verbosity pass — no intended behavior change.** Prose condensed throughout; every section, table, mandate, and cross-reference preserved verbatim in substance — header and table-row counts identical to the pre-pass file. §3's asset note extended to clarify the files themselves were never carried through Design-Phase handoff notes (`CLAUDE.md` §3.8/§3.11) — this directory is genuinely where the actual files first land, possibly updated relative to what was shown during Design. §15's filename list updated to `[projectname]_dev_agent_prompt.md`. |
| 0.8.5 | 2026-07-04 | Claude | **Major compression pass** (555 → 273 lines). §13 Escalation Triggers rewritten to Stop-Summarize-Wait. §6.1/§8: README now drafted at Design Step 8, not scaffolded here — final-phase task is review/finalization only. §15 filenames reverted to `[projectname]_dev_prompt.md`. Added ESP-IDF toolchain row to §4. |
| 0.8.6 | 2026-07-06 | Claude | **v0.8.6 batch.** §14 Change Control: added Ambiguity-Resolving Amendments rule — any amendment resolving an ambiguity where already-completed phases might not satisfy the clarified intent MUST add a verification task to the next dependent not-yet-started phase (check, correct, confirm via standard DoD/Verification-Method; never assumes compatibility). §3 Project Folder Structure: added required `.gitignore` specification — `/target`, `/dist` (Trunk projects), `.env` (with `.env.example` committed), and IDE/OS noise entries; cross-referenced `AGENTS.md` §2.1's corrected preservation mandate distinguishing committed evidence artifacts from gitignored build outputs. |
| 0.8.7 | 2026-07-07 | Claude | **v0.8.7 batch — work-preservation overhaul.** §6.1 Phase Index gained a **Session Unit** column (`Phase`/`Task`/`Code+Verify`). §8 Task Template gained mandatory **Design Refs** (item-level Spec citation, no version suffix, mandatory every task) and **Submit Point**/**WIP-Checkpoint-Eligible** fields; new **Mandatory Code/Verify Split** rule (`a`/`b` sub-tasks for any Build+Test/Hybrid Verification Method task, counts as 2 against Phase Sizing). §11 Session Handoff gained the three-way task-state check against declared Submit Points (not raw git-log). §12 Abort/Rollback scoped to reverting to a task's own last submitted state, not a phase-level checkpoint. §15 DoD gained checks for Design Refs completeness, mandatory split compliance, and Session Unit consistency. |

---

## agents/PREFERRED_DEPENDENCIES.md (continued)

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.7.0 | 2026-07-02 | Claude | Added Embedded (ESP32/ESP-IDF) section: `esp-idf-hal`, `esp-idf-svc`, `esp-camera-rs`, with a Step 5 MSRV/toolchain note (separate `espup`-managed track, independent of workspace/WASM MSRV). Applies primarily to embedded Rust projects. |

---

## agents/PREFERRED_TOOLS.md (continued)

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.7.0 | 2026-07-02 | Claude | Added Embedded section: `espflash` (flashing/serial-monitor tool for ESP32/ESP-IDF targets). |

---

## agents/PREFERRED_SERVICES.md (continued)

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.7.0 | 2026-07-02 | Claude | Added Mosquitto (MQTT broker) as an approved service, with compose snippet and a note that Mosquitto 2.x requires an explicit `mosquitto.conf` (unlike this file's other services) since it refuses anonymous connections and binds no listener by default. No MQTT client crate is yet listed in `PREFERRED_DEPENDENCIES.md` — flagged as Requires-Approval until one is explicitly added. |

---

## agents/exemplars/development_checklist_template.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch (initial versioned entry).** How-to-use section: No-Copy-Checklist mandate added explicitly. README tasks added to Phase 0 and final phase. File naming updated. CHANGELOG.md pointer added. |
| 0.8.5 | 2026-07-04 | Claude | Compression pass (135 → 101 lines). Phase 0's `DOC-001` task reworded from "scaffold" to "review, confirm, and enhance" the Design-Step-8-drafted README. |
| 0.8.7 | 2026-07-07 | Claude | **v0.8.7 batch.** Added a **Submitted** checkbox per task/sub-task (checked only once its declared Submit Point actually fires, distinct from local DoD satisfaction). Added the Code/Verify `a`/`b` split rendering convention as two adjacent Task sub-sections, each with its own DoD and Submitted checkboxes; example `DOMAIN-002` task updated to demonstrate the split. |

---

## agents/exemplars/dev_prompt_template.md (renamed from dev_agent_kickoff_prompt_template.md 0.8.0, then dev_agent_prompt_template.md 0.8.5)

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch (initial versioned entry).** §1 doc list updated to new file naming pattern. §2 environment check expanded: missing tools self-installed + appended to setup_env.sh/bat; version sanity check against project compatibility matrix. §4 One-Phase-Per-Session mandate added. §5 No-Copy-Checklist mandate added. Session-end checklist updated. V/R/M refs removed. CHANGELOG.md pointer added. |
| 0.7.0 | 2026-07-02 | Claude | Step 1's doc list updated to the 8-file, one-per-Design-Step Architecture Spec naming (`01_introduction` through `08_constraints_and_roadmap`), fixing a prior example that implied a single shared version placeholder across all Architecture files — each is now shown as independently versioned. Added Asset Manifest cross-check as doc-review item 5, including an explicit Escalation Trigger for a manifest-listed asset missing from the repository. "Notes for Whoever Fills In This Template" corrected to match. |
| 0.8.0 | 2026-07-03 | Claude | **File renamed** `dev_agent_kickoff_prompt_template.md` → `dev_agent_prompt_template.md`; output artifact renamed `[projectname]_dev_prompt.md` → `[projectname]_dev_agent_prompt.md` — updated everywhere referenced (`AGENTS.md`, `CLAUDE.md`, `agents/DESIGN.md`, `agents/DEVELOPMENT.md`, `README.md`, `agents/exemplars/development_plan_template.md`). Prompt text now opens with an explicit statement that the agent is operating in the Development Phase under `agents/DEVELOPMENT.md` (not Design Phase), directing the agent to read that guide if not already internalized this session. |
| 0.8.5 | 2026-07-04 | Claude | **File renamed back** to `dev_prompt_template.md`; output reverted to `[projectname]_dev_prompt.md`, per explicit user instruction reverting the 0.8.0 rename. Compression pass (245 → 112 lines). Session steps rewritten to Stop-Summarize-Wait: any unresolved issue stops the session immediately at the Phase Summary — no PR, no further progress. |
| 0.8.7 | 2026-07-06 | | Claude | **v0.8.7 batch — major restructure.** Step 1 rewritten from "read all docs upfront" to a two-tier model: mandatory-upfront (checklist, phase summaries, current phase §6.1+§8 via targeted extraction, protocols file in full) then on-demand only. New step 2: on-demand reference table mapping 17 uncertainty types (interface contract, data model, requirement wording, test case, traceability, concurrency/ownership, security, technology/MSRV, build order, architectural risk, user story, system overview, deployment, dev plan risk/test/environment/stack) to specific `[projectname]_architecture_NN` or `_dev_plan_NN` file + section + `sed`/`grep`/`awk` extraction command — consulted only when confusion, conflict, missing clarity, or abnormality arises during task work. CRITICAL context window rules block added at top of step 1: no broad repository scan, no whole-file reads of large documents (protocols file excepted), all doc access via targeted extraction. Former steps 2–9 renumbered 3–10. Session-End Checklist gains verification item for the no-scan/no-whole-file-read rules. Notes section updated with fill-in guidance for extraction commands and reference table placeholders. |
| 0.8.7 | 2026-07-07 | Claude | **v0.8.7 batch (part 2) — work-preservation overhaul.** Prompt text opens with Code Review Policy / Agent Tool Policy / post-submit Continue-Proceed / Session Unit cross-references. Step 6 ("find the next unit") gained the three-way task-state check against declared Submit Points, resuming in-place from a WIP checkpoint's own description rather than redoing work. Step 7 gained mandatory Code/Verify split handling and immediate per-Submit-Point submission instructions. Step 9 reframed as phase-level wrap-up submit only, generalized to Session Unit completion. Session-End Checklist gained items verifying every completed task/sub-task has a matching submit and Submitted checkbox, WIP checkpoint description sufficiency, and no code-review calls. |

---

## README.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.7.0 | 2026-07-02 | Claude | Initial versioned entry. Updated Design Phase description from 8-step to 9-step gated workflow (Step 9 Plan & Checklist Audit added). Key Mandates gained "Independent audits" and "Tracked UI/media assets" bullets to reflect the same batch of changes landing across `CLAUDE.md`/`DESIGN.md`/the exemplar templates.
| 0.7.1 | 2026-07-03 | Claude | "Tracked UI/media assets" bullet clarified: asset files are never packaged into a Design Phase handoff note, only referenced by filename. New "Development agents write to one file" bullet added, reflecting `AGENTS.md` §2.7's extended exclusive-file-touch/phase-scoped-marks mandate. Repository tree entry renamed to `dev_agent_prompt_template.md`. |
| 0.7.0 | 2026-07-02 | Claude | Step 1's doc list updated to the 8-file, one-per-Design-Step Architecture Spec naming (`01_introduction` through `08_constraints_and_roadmap`), fixing a prior example that implied a single shared version placeholder across all Architecture files — each is now shown as independently versioned. Added Asset Manifest cross-check as doc-review item 5, including an explicit Escalation Trigger for a manifest-listed asset missing from the repository. "Notes for Whoever Fills In This Template" corrected to match.
| 0.8.5 | 2026-07-04 | Claude | **v0.8.5 batch.** Key Mandates rewritten: fixed RCD/RATS procedure and Surgical Fix Override description replace the autonomy-toggle framing; added the Stop-Summarize-Wait escalation model; added the Design-Step-8-drafts-README note; added the ESP32/ESP-IDF build guide to the repository tree and Key Mandates. Tightened the gated-execution bullet to state exact `APPROVED` (no synonyms). |

---

## agents/ESP32_ESPIDF_RUST_BUILD_GUIDE.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.8.5 | 2026-07-04 | Claude | New file, added to `agents/` — user-supplied ESP32/ESP-IDF/Rust/tokio build guide, scoped to embedded projects only, cross-referenced from `CLAUDE.md` §2.1, `agents/RUST_PREFERENCES.md`, and both `PREFERRED_DEPENDENCIES.md`/`PREFERRED_TOOLS.md` Embedded sections. |
