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

---

## agents/DEVELOPMENT.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.0.01 | 2025-10-10 | Jules | Initial creation and refactoring. |
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch.** §4 added: Missing-Tools Mandate (self-install + idempotent append to scripts/setup_env.sh and scripts/setup_env.bat; escalation trigger still fires). No-Copy-Checklist Mandate. One-Phase-Per-Session Mandate. §3 updated to reference `[projectname]_dev_prompt.md`. §5.2 README task added (Phase 0 scaffold + final-phase review). Docker/deploy guidance added (heavy services via docker-compose in deploy/; reference PREFERRED_SERVICES.md). "Verification Phase" wording replaced with "feature-complete." V/R/M refs removed. CHANGELOG.md pointer added. |
| 0.7.0 | 2026-07-03 | Claude | §4's No-Copy-Checklist Mandate extended into an explicit **exclusive-file-touch rule**: `[projectname]_dev_checklist.md` is the only documentation/planning file a development-agent session may edit — every other doc file (Architecture Spec, Plan, Kickoff/Dev-Agent Prompt, handoff notes, Phase Summaries) is read-only, with a stray inconsistency treated as an Escalation Trigger rather than patched in passing. Edits within the checklist are further scoped to box-flipping/Session-Log rows for the agent's **current Phase only** — never marks belonging to another phase, never structural text. `[projectname]_dev_prompt.md` renamed to `[projectname]_dev_agent_prompt.md` throughout. |

---

## agents/RUST_PREFERENCES.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1.0 | 2026-06-19 | Claude | Initial creation. Naming/keyword constraints, type-system/ownership constraints, WASM concurrency constraints. |
| 0.1.1 | 2026-06-19 | Claude | COOP/COEP blocking trigger. Ranked recommended-crates table. Step 5 summary updated. |
| 0.2.0 | 2026-06-20 | Claude | Crate table reordered to lead with gloo-worker as default. |
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch.** §0 added: Rust 2024 edition mandate; MSRV-at-Step-5 policy; dual-MSRV pattern for publishable library crates; pre-flight version sanity check. cargo-llvm-cov replaces cargo-tarpaulin as coverage tool. Reference to PREFERRED_TOOLS.md added. CHANGELOG.md pointer added. |

---

## agents/PREFERRED_DEPENDENCIES.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch (initial versioned entry).** Rust 2024 edition mandate added. cargo-tarpaulin removed (replaced by cargo-llvm-cov in PREFERRED_TOOLS.md). trunk removed (moved to PREFERRED_TOOLS.md). webpack removed entirely. MSRV policy pointer added. CHANGELOG.md pointer added. |

---

## agents/PREFERRED_TOOLS.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1.0 | 2026-06-29 | Claude | Initial creation. Covers: cargo-nextest, cargo-llvm-cov, cargo-deny, cargo-audit, cargo-watch, trunk, sqlx-cli, cargo-expand, wasm-pack, protoc. Includes rust-toolchain.toml template and pre-flight version check guidance. |

---

## agents/PREFERRED_SERVICES.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1.0 | 2026-06-29 | Claude | Initial creation. Approved services: PostgreSQL+SQLx, Neo4j, Qdrant, MinIO, Redis. Docker-compose preferred; configs in deploy/. Port conventions. Service snippet templates. HTTP-over-HTTPS preference for local dev. |

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

---

## agents/exemplars/dev_agent_prompt_template.md (renamed from dev_agent_kickoff_prompt_template.md, 0.8.0)

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.6.3 | 2026-06-29 | Claude | **v0.6.3 batch (initial versioned entry).** §1 doc list updated to new file naming pattern. §2 environment check expanded: missing tools self-installed + appended to setup_env.sh/bat; version sanity check against project compatibility matrix. §4 One-Phase-Per-Session mandate added. §5 No-Copy-Checklist mandate added. Session-end checklist updated. V/R/M refs removed. CHANGELOG.md pointer added. |
| 0.7.0 | 2026-07-02 | Claude | Step 1's doc list updated to the 8-file, one-per-Design-Step Architecture Spec naming (`01_introduction` through `08_constraints_and_roadmap`), fixing a prior example that implied a single shared version placeholder across all Architecture files — each is now shown as independently versioned. Added Asset Manifest cross-check as doc-review item 5, including an explicit Escalation Trigger for a manifest-listed asset missing from the repository. "Notes for Whoever Fills In This Template" corrected to match. |
| 0.8.0 | 2026-07-03 | Claude | **File renamed** `dev_agent_kickoff_prompt_template.md` → `dev_agent_prompt_template.md`; output artifact renamed `[projectname]_dev_prompt.md` → `[projectname]_dev_agent_prompt.md` — updated everywhere referenced (`AGENTS.md`, `CLAUDE.md`, `agents/DESIGN.md`, `agents/DEVELOPMENT.md`, `README.md`, `agents/exemplars/development_plan_template.md`). Prompt text now opens with an explicit statement that the agent is operating in the Development Phase under `agents/DEVELOPMENT.md` (not Design Phase), directing the agent to read that guide if not already internalized this session. |

---

## README.md

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.7.0 | 2026-07-02 | Claude | Initial versioned entry. Updated Design Phase description from 8-step to 9-step gated workflow (Step 9 Plan & Checklist Audit added). Key Mandates gained "Independent audits" and "Tracked UI/media assets" bullets to reflect the same batch of changes landing across `CLAUDE.md`/`DESIGN.md`/the exemplar templates.
| 0.7.0 | 2026-07-02 | Claude | Step 1's doc list updated to the 8-file, one-per-Design-Step Architecture Spec naming (`01_introduction` through `08_constraints_and_roadmap`), fixing a prior example that implied a single shared version placeholder across all Architecture files — each is now shown as independently versioned. Added Asset Manifest cross-check as doc-review item 5, including an explicit Escalation Trigger for a manifest-listed asset missing from the repository. "Notes for Whoever Fills In This Template" corrected to match.
