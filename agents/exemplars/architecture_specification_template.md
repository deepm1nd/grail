# Architecture Specification: [Project Name]
Complex/Multi-Domain Rust Project Variant

> **When to use:** substantial Rust projects running `agents/DESIGN.md`'s 9-step gated
> workflow in full. Counterpart to `development_plan_template.md` (its §0 points at this
> doc's §3, physically split per `CLAUDE.md` §4.1 across `_04_test_strategy` and
> `_05_verified_traceability`, as the sole traceability source — no separate Traceability doc).
> **Scope:** Rust-only — native services, Rust/WASM, Tauri/UniFFI shells (§4.9/§4.6).
> **Multi-file packaging:** splits per `CLAUDE.md` §4's 8-file, one-per-Design-Step scheme;
> section numbering stays stable across the split.
> **Every requirement/decision/interface/scenario carries a stable, unique ID**, never
> reused or renumbered (Plan's Change Control) — this is what makes Steps 3–8 checkable.

## Table of Contents
1. Introduction · 2. Product & User Requirements · 3. Acceptance Criteria & Traceability ·
4. System Architecture (ISO 42010) · 5. External Interfaces & Integrations · 6. Technology
Stack & Dependencies · 7. Constraints & Assumptions · 8. Risks & Technical Debt ·
9. Implementation Roadmap & Build Order · 10. Public API & Framework Consumer Contract ·
11. Appendices · Appendix R Version History

---

## 1. Introduction
1.1 **Document Purpose & Audience** — who this is for, what it's binding for once approved
(Step 6/7); the Plan's §0/§14 precedence rule from the Plan's side.
1.2 **Product/System Overview** — Step 1's structural draft.
1.3 **Problem Statement & Vision**
1.4 **Goals & Objectives** — distinguish business/product goals from architectural goals
(§4.1 elaborates the latter).
1.5 **Definitions, Acronyms, Abbreviations** — local to this doc; durable terms go in §11.1.
1.6 **References** — originating concept statement, prior spec if any, governing process
docs, external standards cited elsewhere (ISO 42010, RFCs).
1.7 **Open Items Register** (`CLAUDE.md` §3.12) — the running, project-wide log of every
item that has ever entered RATS, one row per item, updated at every Step's gate from Step 1
onward: ID, originating Step, one-line description, and outcome (`Resolved` /
`Deferred to Step N` / `Future Feature` / `Rejected` — never left blank past the gate that
raised it). Lives here, not as a separate file, because this is the one Architecture file
created at Step 1 and present at every subsequent gate; every Step's session touches this
section to append/update entries even when its own owned file is a different one (the
ordinary "touched file" versioning rule, `CLAUDE.md` §4.3, already covers a session bumping
more than just its home file). Historical entries (`Resolved`, `Future Feature`, `Rejected`)
are never deleted or reworded once terminal — only appended to and, for a `Deferred` entry,
updated in place when its target Step actually resolves it.

---

## 2. Product & User Requirements

### 2.1. Target Audience & User Personas
Include machine/service personas (e.g. "an AI agent executing orchestration steps"), not
just human ones.

### 2.2. User Stories & Interaction Sequences
Step 2's RCD/RATS output, by persona then theme. Stable ID `US-[DOMAIN]-NNN`.
- **User Story `US-[DOMAIN]-001`:** As a {persona}, I want to {action}, so that {benefit}.
    - **Interaction Sequence:** 1. ... 2. ...
    - **Notes/Assumptions from RATS:** any assumption raised while drafting this story and
      its resolved outcome (Resolved/Deferred/Future Feature/Rejected), retained here.

### 2.3. Core Functional Requirements
Each requirement: unique ID `[PROJ]-FUNC-[DOMAIN]-NNNN`, atomic, includes a Verification
Protocol, traces to ≥1 User Story ID.

| Req ID | Description | Originating User Story | Verification Protocol |
|---|---|---|---|
| `[PROJ]-FUNC-[DOMAIN]-0001` | | `US-[DOMAIN]-001` | |

### 2.4. Non-Functional Requirements (Quality Attribute Scenarios)
**MANDATE: expressed as stimulus/response scenarios**, not bare statements. Same ID/
Verification-Protocol rigor as §2.3.

**Fields:** Source · Stimulus · Environment (system state when stimulus occurs — normal,
degraded, peak load) · Artifact · Response · Response Measure (the checkable bar) ·
Verification Protocol.

| Req ID | Quality Attribute | Source | Stimulus | Environment | Artifact | Response | Response Measure | Verification Protocol |
|---|---|---|---|---|---|---|---|---|

**Categories to consider** (state explicitly which are out of scope): Performance/Latency,
Throughput, Availability, Scalability, Security (§4.8), Reliability, Data
Durability/Consistency, Observability, Maintainability, Testability, Usability/
Accessibility, Portability, Interoperability, Compliance.

### 2.5. Out-of-Scope Features
Enumerate explicitly, with *why* excluded (owned elsewhere / not validated / rejected).
Distinct from §2.6 (wanted-but-deferred) and from an open question (§7.3/escalation).

### 2.6. Future Features (Deferred Scope)
Any RATS-deferred-to-future-feature item, or explicitly deferred scope, lands here — not
merged into §2.5. Per item: **Originating ID** (if assigned before deferral — never
reassigned) · **What it is** (sufficient for a future session to act on) · **Why
deferred** · **Dependency/precondition**, if known. Additive-only; an item here is never
silently removed, only explicitly promoted (removed from here) or dropped (moved to §2.5).

---

## 3. Acceptance Criteria & Traceability

### 3.1. The 9 Requirement Quality Criteria (Self-Check Reference)

| # | Criterion | Failure mode it catches |
|---|---|---|
| 1 | Necessary | Doesn't trace to a real story/goal |
| 2 | Atomic | Bundles >1 independently testable statement |
| 3 | Unambiguous | >1 reasonable implementation reading |
| 4 | Verifiable | No test/artifact could prove/disprove it |
| 5 | Feasible | Not achievable given tech/constraints |
| 6 | Complete | Missing a condition/exception/boundary |
| 7 | Consistent | Contradicts another requirement |
| 8 | Design-independent | Specifies *how* not *what* (waivable — state explicitly if the decision genuinely *is* the implementation, e.g. "MUST use Argon2id") |
| 9 | Traceable | No stable ID / no link to origin |

### 3.2. Test Case Catalog
Every Test Case referenced anywhere (§2.3/§2.4 Verification Protocols, §3.4) is defined
here first — an undefined `TEST-[NNN]` reference is a Traceable-criterion failure. Unique
ID `TEST-[NNN]`, traces to ≥1 Req ID, states type, states the DoD this test enforces.

| Test ID | Verifies Req ID(s) | Type | Test Description | Definition of Done |
|---|---|---|---|---|
| `TEST-001` | | Unit/Integration/System/Acceptance | | |

### 3.3. Requirement Smells (Rust-Specific)
Per `agents/RUST_PREFERENCES.md` §2.

| Smell | Why it's a smell in Rust | Preferred reframing |
|---|---|---|
| Nullable-value language w/o absent-vs-empty distinction | No implicit null — must resolve to `Option<T>` | State "`None` when X" vs. "empty `Vec` when X" |
| Inheritance/"is-a" framing | No impl inheritance | State trait composition / enum variant |
| Exception/"throws" framing | No exceptions | State `Result<T, E>` + specific `Err` variant |
| Unbounded "and so on" enumeration | Exhaustive `match` needs every variant named | Enumerate fully, or flag as needing `#[non_exhaustive]` |
| Implicit shared-mutable-state w/o ownership | Needs an explicit owner/concurrency answer | Resolve ownership/concurrency, or flag as §4.7/§4.9 pending |

### 3.4. Traceability Matrix
Master cross-reference; Step 7 checks completeness.

| Req ID | Component / Unit | Test ID | Verification Artifact |
|---|---|---|---|

### 3.5. Acceptance Criteria Detail
Per requirement needing more than a one-line Verification Protocol: preconditions, steps,
postconditions, edge cases. One subsection per Req ID.

---

## 4. System Architecture (ISO/IEC/IEEE 42010 Viewpoints)

Step 6's output. State briefly, per element, whether formal notation (SysML-style) or prose
was used and why.

### 4.1. Architectural Goals & Constraints
Architecture-specific goals from §1.4 plus shaping constraints (cross-ref §4.9/§6 for
Rust/WASM-driven ones, e.g. "multithreading on both native and WASM").

### 4.2. Architectural Principles
Standing principles (e.g. "single public entry point," "hexagonal architecture,"
"default-deny authorization") — every viewpoint traces back to one of these.

### 4.3. Solution Strategy
Short — the small number of fundamental decisions shaping everything else: chosen style,
how top §2.4 quality goals are achieved structurally, key §6 decisions at rationale level.

### 4.4. Functional View
- **Description:** components/responsibilities/collaborations.
- **Diagrams:** System Context, Container/Component, Sequence (for non-obvious
  multi-component flows) — state why formal notation per diagram.
- **Component Responsibility Summary:** per component, core responsibility + explicitly
  what it's **not** responsible for.
- **CRC Cards:** Responsibility / Collaborators / Notes, where collaboration isn't obvious.

### 4.5. Information View (Data Dictionary & Models)
- **Data Dictionary:**

| Entity | Attribute | Type | Description | Constraints |
|---|---|---|---|---|

- **Data Models:** ERDs/JSON Schema/Protobuf/Rust struct-enum — state which representation
  is authoritative where more than one exists.
- **Persistence Schema:** table/column/index detail, migration ordering. If >1 storage tech
  serves the same entity, state source of truth and consistency mechanism.
- **Migration Safety & Rollback Policy:** destructive-change policy (direct vs.
  deprecate-then-remove), backward-compat window, rollback procedure, irreversible
  migrations flagged with a mandatory backup step.
- **Data Lifecycle & Retention:** per entity with a retention/tiering/deletion requirement.

### 4.6. Deployment View
- **Diagrams:** Physical/Deployment, Network Topology, WASM hosting topology if applicable.
- **Per-Target Detail:** native service target(s) (orchestrator, scaling, network exposure);
  WASM target(s) (hosting requirements + **COOP/COEP feasibility** if `SharedArrayBuffer`
  threading is used — which environments can guarantee the headers, and the fallback for
  those that can't); mobile/desktop shell target(s) (distribution, FFI boundary);
  infrastructure services (compose file, port, HTTP-not-HTTPS for local dev unless stated).
- **Configuration Reference:** deployment-relevant high points only (full schema in §11.3).

### 4.7. Process View (Runtime/Concurrency)
- **Concurrency Model, per component:** native (async runtime/task model, `Send`/`Sync`
  justification for boundary-crossing types); WASM (message-passing via Web Workers,
  default/preferred, vs. `SharedArrayBuffer` — only where message-passing is demonstrably
  insufficient, and only with §4.6's COOP/COEP confirmed). **State explicitly: no component
  combines two Worker-pool-owning threading crates without an explicit recorded decision.**
- **Sequence Diagrams:** for non-obvious runtime flows.
- **Synchronization & Shared State:** ownership model per shared-state instance, and why.

### 4.8. Security Architecture & Threat Model
- **Defense-in-Depth Layers:** layer model + mechanism per layer.
- **STRIDE Threat Model:** per externally-reachable/trust-boundary-crossing component.

| Threat ID | Component | STRIDE Category | Threat Description | Mitigation | Residual Risk |
|---|---|---|---|---|---|

- **Trust Boundaries:** enumerate explicitly; anchor every threat to one.
- **Data Governance:** what's checked pre-persistence (secret detection, PII
  classification) vs. read-time; failure mode for a rejected write.
- **Secrets & Key Management:** sourcing, rotation, scoping; cross-ref §6 and §4.9.

### 4.9. Rust-Specific Architectural Conventions
- **Edition and MSRV:** workspace edition/MSRV (`agents/RUST_PREFERENCES.md` §0), dual-MSRV
  detail if a publishable crate exists (cross-ref §10.2).
- **Error Handling Strategy:** `Result`/error-type conventions, layered-hierarchy detail,
  what detail may cross a layer boundary (e.g. never into an external API response body).
- **Ownership & Lifetime Conventions at Component Boundaries.**
- **Concurrency Primitives & `Send`/`Sync` Policy:** default primitive, WASM/Worker-first
  stance, `SharedArrayBuffer` exception criteria (§4.6/§4.7).
- **Trait-Based Composition / Hexagonal Conventions**, if used.
- **`unsafe` Code Policy:** permitted-with-review-standard, or `#![forbid(unsafe_code)]`
  (cross-ref §7.1).
- **WASM-Specific Conventions:** target triple(s), `trunk` (preferred), `#[cfg]` divergence
  convention.
- **ESP32/ESP-IDF Conventions, if applicable:** cross-ref `agents/ESP32_ESPIDF_RUST_BUILD_GUIDE.md`
  for toolchain (`espup`), target triple, and `tokio`-on-ESP-IDF configuration.

### 4.10. Interface Control Document (ICD)
- **Internal APIs:** signatures, schemas (cross-ref §4.5, don't redefine), sync/async +
  why, error responses, versioning policy.
- **External Integrations:** contract as the external party defines it (linked, not
  reproduced) + this system's specific usage.
- **Event/Message Interfaces:** topic/schema ownership, delivery semantics, dead-letter
  handling.

### 4.11. Architecture Decision Records (ADRs)
Every architecturally significant decision, discrete and ID-stable.

#### `ADR-[NNN]`: [Decision Title]
- **Status:** Proposed | Accepted | Superseded (by `ADR-[NNN]`) | Deprecated
- **Context** · **Decision** · **Consequences** (both positive and negative) ·
  **Paths Not Taken** (credible alternatives and why rejected).

### 4.12. Technical Non-Functional Requirements
Internal/technical NFRs not naturally a stimulus/response scenario (build time budgets,
WASM binary size, CI duration limits). Most NFRs belong in §2.4; keep this short.

### 4.13. User Experience (UX) & UI Design
For any human-facing interface: information architecture, key flows (cross-ref §2.2),
accessibility, framework/crate + architectural implication for Rust/WASM UI. State
explicitly if no human-facing interface exists.

**Asset Manifest.** Per `CLAUDE.md` §2.2/§3.7. Columns: `Filename | Type (html/image/audio/
video) | Repository Target Path | Authoritative/Informative For | Provided At (Step)`.
Filenames immutable once logged; target paths fixed (`assets/html/`, `assets/images/`,
`assets/audio/`, `assets/video/`). HTML is presumptively authoritative for structure/
behavior; images source Design System content. Every logged asset referenced substantively
somewhere in this section, not manifest-only. **Assets themselves are never carried in
Design-Phase handoff notes** (`CLAUDE.md` §3.7/§3.10) — referenced by filename only; actual
files are a Development-Phase input.

| Filename | Type | Repository Target Path | Authoritative/Informative For | Provided At (Step) |
|---|---|---|---|---|

### 4.14. Logging and Monitoring
- **Logging:** 5-level system (Error/Warn/Info/Debug/Trace), default `INFO`,
  env-var-configurable, structured-logging field/correlation-ID conventions +
  redaction policy for secret-bearing fields (cross-ref §4.8).
- **Monitoring:** key metrics mapped to §2.4 Response Measures, alerting thresholds,
  distributed tracing strategy if multi-component.

### 4.15. Deferred Implementation Alternatives (Noted, No Commitment)
Distinct from §4.11's ADR "Paths Not Taken" (closed) — these are credible, may be adopted
later, no extension-point commitment made now.

#### Originating: `[REQ-ID or topic]`
- **Decision point** · **Option taken** (pointer to full spec) · **Option(s) deferred,
  noted only** (why credible, cost/gain vs. chosen) · **Revisit trigger**, if known.

### 4.16. Deferred Implementation Alternatives (Extensibility Commitment Required)
Stronger case: current implementation MUST expose a real extension point for a deferred
option, specified in §4.4/§4.9 — this subsection indexes it, doesn't substitute for it.

#### Originating: `[REQ-ID or topic]`
- **Decision point** · **Option taken (shipped now)** · **Option(s) deferred, with
  extension point required** (the specific mechanism, e.g. a named trait boundary) ·
  **Where the commitment is specified** (exact §4.4/§4.9 cross-reference).

---

## 5. External Interfaces & Integrations
Distinct from §4.10's binding contract detail — the inventory and rationale.

5.1 **External System Interfaces** — inventory, cross-ref §4.10 for detail.
5.2 **Third-Party Integrations** — services/libraries constituting an integration
(identity provider, payment processor, LLM API), distinct from §6's compiled-in stack.
5.3 **API Specifications (External)** — public contract, versioning/deprecation policy for
externally-reachable APIs.

---

## 6. Technology Stack & Dependencies
> See `agents/PREFERRED_DEPENDENCIES.md`, `agents/PREFERRED_TOOLS.md`,
> `agents/PREFERRED_SERVICES.md` for the binding rules this section's choices must satisfy.

6.1 **Core Technologies** — language/edition, MSRV (§4.9), async runtime, web framework.

| Category | Technology | Version | Rationale |
|---|---|---|---|

6.2 **Per-Component Technology Stack**

| Component | Crate type / target | Key crates | Rationale |
|---|---|---|---|

6.3 **External Dependencies (Infrastructure)** — from `PREFERRED_SERVICES.md`.

| Component | Technology | Notes |
|---|---|---|

6.4 **Dependency Governance** — vetting, license/supply-chain scanning (`cargo audit`/
`cargo deny`), approval tiering per `PREFERRED_DEPENDENCIES.md`.

6.5 **CI/CD Pipeline** — pipeline stages and what gates each; trigger conditions; environment
promotion strategy; bad-deploy rollback procedure (distinct from §4.5's data rollback) +
target time-to-rollback; per-target build artifacts; **Required CI Gates (Consolidated)** —
single list cross-referencing every CI gate named elsewhere (§6.4, §10.2, §10.3, MSRV
sanity check).

---

## 7. Constraints & Assumptions

7.1 **Technical Constraints** — hard limits (`unsafe` policy if hard, deployment env limits
e.g. COOP/COEP).
7.2 **Business Constraints** — budget, timeline, regulatory.
7.3 **Assumptions** — only ones that, if wrong, would require revisiting approved content.

| Assumption ID | Assumption | Originating Step/Section | Consequence if Wrong |
|---|---|---|---|

7.4 **Dependencies** — cross-team/system/project delivery dependencies (distinct from §6).

---

## 8. Risks & Technical Debt
Distinct from Plan §7 (delivery risk) — this is architectural risk.

8.1 **Known Architectural Risks**

| Risk ID | Description | Impact | Likelihood | Mitigation/Monitoring | Related ADR |
|---|---|---|---|---|---|

8.2 **Accepted Technical Debt** — deliberate simpler-now choices with a revisit plan (or
explicit no-plan-yet), cross-ref ADR.
8.3 **Scenarios This Architecture Does Not Handle Well** — honest weak-point statement,
distinct from §2.5 Out-of-Scope.

---

## 9. Implementation Roadmap & Build Order

### 9.1. Sequencing Principles
Dependency logic determining build order (e.g. "domain before infra adapters," "identity
before authz before accounting").

**Frontend Targeted Interleaving,** where a UI exists: each screen/component's Build Order
step sits alongside the backend deliverable supplying its real data — never earlier, never
batched into a trailing frontend-only step. See `development_plan_template.md` §6/§9.1.

### 9.2. Build Order

| Step | Deliverable | Component(s) | Exit Criteria |
|---|---|---|---|

### 9.3. Phased Delivery Milestones
MVP/beta/GA definitions and the Build Order steps each requires. Name §2.6 Future Features
items explicitly where a milestone is expected to bring them in.

---

## 10. Public API & Framework Consumer Contract
> For external code depending on this as a library/framework — doesn't exist for a typical
> internal service, but load-bearing for a framework.

10.1 **Semantic Versioning & Breaking-Change Policy** — what counts as breaking for this
project's actual surface (non-exhaustive enum variant addition, trait-bound tightening,
MSRV bump); pre-1.0 policy if applicable; deprecation notice period; changelog convention.
10.2 **MSRV Policy** — current consumer MSRV (explicit, not "latest stable"); bump policy;
enforcement (dedicated CI stage compiling only the publishable crate at that version).
10.3 **Consumer-Facing Documentation & Examples** — rustdoc requirements (`# Panics`/
`# Errors`), doctests as a CI gate, example-crate convention, `pub`-surface review process.

---

## 11. Appendices

11.1 **Glossary of Terms** — durable, project-lifetime term registry.

| Term / Acronym | Meaning |
|---|---|

11.2 **Detailed Diagrams** — anything too large to inline in its viewpoint section.
11.3 **Configuration Reference** — complete config schema, defaults, env-var override
convention, explicit non-secret-vs-secrets-manager distinction (Plan §5 derives dev/test
config from this).

```toml
[server]
host = "0.0.0.0"
port = 8080
```

11.4 **Decision Log Index** — if §4.11 grows numerous.

| ADR ID | Title | Status |
|---|---|---|

## Appendix R — Version History

**Per `CLAUDE.md` §4.3 — this file's own version history, and only this file's.** One row
per session that bumped *this specific* numbered file (e.g. `_01_introduction`), never the
whole Spec's combined history — each of the 8 Architecture Spec files carries its own
independent Appendix R, matching its own independent `_v[N]` counter. Appended to, one new
row per bump, in the same pass as the bump itself; prior rows are never rewritten or removed.
This is the file's *only* changelog surface — no version/revision commentary anywhere else
in the file (not under the title, not in §1 Introduction, not inline near changed content).

| Version | Date | Session / Step | Changes |
|---|---|---|---|
| v1 | [date] | Step [N] (initial) | Initial creation. |

---
See `CHANGELOG.md` for this file's full version history.
