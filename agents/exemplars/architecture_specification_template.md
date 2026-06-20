# Architecture Specification: [Project Name]
v.0.1.00 — Complex/Multi-Domain Rust Project Variant

> **When to use this template:** use this template for Rust projects substantial enough to
> warrant a multi-document Architecture Specification — multiple services/components, a
> security-sensitive surface, a persistence layer with real schema evolution, or any project
> where `DESIGN.md`'s 8-step gated workflow (Concept Intake → User Stories → Requirement
> Decomposition → Test Identification → Verification Feasibility → Architecture Synthesis →
> Audit → Plan Generation) is being run in full rather than abbreviated. It is the architecture
> counterpart to `development_plan_template.md` and is designed to be consumed by that
> template's §0 (Architecture Cross-Reference), which points directly at this document's §3
> (Acceptance Criteria & Traceability) as the sole, authoritative traceability source — no
> separate Requirements & Traceability Backfill document is produced.
>
> **Scope:** Rust-only, consistent with this repository's Rust-first mandate. It supports any
> mix of native services, Rust/WASM web components, and Tauri/UniFFI mobile or desktop shells —
> see §4.9 (Rust-Specific Architectural Conventions) and §4.6 (Deployment View) for how a
> multi-target project expresses target-specific concerns without duplicating shared ones.
>
> **Multi-file packaging:** this template is written as a single logical document but is
> expected to be split across multiple physical files for large projects (see the working
> arrangement governing this engagement, e.g. `CLAUDE.md` §4, for the cross-linking convention).
> Each top-level section below is a natural file-split boundary; section numbering is stable
> across a split so cross-references remain valid regardless of how files are divided.
>
> **Traceability is load-bearing throughout this document.** Every requirement, every
> architectural decision, every interface, and every quality attribute scenario carries a
> stable, unique ID. IDs are never reused or renumbered (see Appendix R and the companion
> Development Plan's Change Control section). This is what makes Steps 3–8 of the gated
> workflow mechanically checkable rather than a matter of editorial judgment.

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Product & User Requirements](#2-product--user-requirements)
- [3. Acceptance Criteria & Traceability](#3-acceptance-criteria--traceability)
- [4. System Architecture (ISO/IEC/IEEE 42010 Viewpoints)](#4-system-architecture-isoiecieee-42010-viewpoints)
- [5. External Interfaces & Integrations](#5-external-interfaces--integrations)
- [6. Technology Stack & Dependencies](#6-technology-stack--dependencies)
- [7. Constraints & Assumptions](#7-constraints--assumptions)
- [8. Risks & Technical Debt](#8-risks--technical-debt)
- [9. Implementation Roadmap & Build Order](#9-implementation-roadmap--build-order)
- [10. Public API & Framework Consumer Contract](#10-public-api--framework-consumer-contract)
- [11. Appendices](#11-appendices)
- [Appendix R - Revision History](#appendix-r---revision-history)

---

## 1. Introduction

### 1.1. Document Purpose and Audience
(State who this document is for — senior Rust engineers, AI agents implementing the system,
architects/reviewers — and what decisions it is and is not authoritative for. State explicitly
that this document, once approved per `DESIGN.md` Step 6/7, is the binding source of truth for
the Development Plan; see the Plan template's §0 Architecture Cross-Reference and §14 Change
Control, which state the precedence rule from the Plan's side.)

### 1.2. Product/System Overview
(Concise description of what the system is and does. This is the output of Step 1's structural
drafting — see §1.3 for how this relates to the original concept statement.)

### 1.3. Problem Statement & Vision
(What problem this system solves and for whom; the vision for the system once realized.)

### 1.4. Goals & Objectives
(Concrete, ideally measurable goals. Distinguish business/product goals from architectural
goals — architectural goals are elaborated further in §4.1.)

### 1.5. Definitions, Acronyms, and Abbreviations
(Document-local definitions needed to read *this document*. This is distinct from the
project-wide Glossary in §11.1 — Appendix — which is the durable, project-lifetime term registry.
Terms that matter beyond this document belong in both places; terms that are only meaningful as
local shorthand within this document belong here only.)

### 1.6. References
(Links/citations to the originating concept statement, any prior-version specifications being
superseded or reconciled, any governing process documents such as `AGENTS.md`/`DESIGN.md`/the
working-arrangement file for this engagement, and any external standards referenced elsewhere in
this document — ISO/IEC/IEEE 42010, the relevant RFCs for any protocol this system implements,
etc.)

---

## 2. Product & User Requirements

### 2.1. Target Audience & User Personas
(Who uses this system, directly or as another system integrating with it. Include
machine/service personas — e.g. "an AI agent executing orchestration steps" — not just human
personas, where the system has non-human callers; this project's own consuming agents are a
persona in their own right per `DESIGN.md`'s Anti-Stub Mandate motivation.)

### 2.2. User Stories & Interaction Sequences
(Output of Step 2 — the propose-then-correct loop. Organize by persona, then by theme: happy
path, edge case, error/failure state, admin/operational. Every story carries a stable ID,
e.g. `US-AUTH-001`, referenced later by the requirements it decomposes into.)

- **User Story `US-[DOMAIN]-001`:** [As a {persona}, I want to {action}, so that {benefit}.]
    - **Interaction Sequence:**
        1. [Step 1]
        2. [Step 2]
    - **Notes/Assumptions flagged during Step 2:** [any assumption the drafting agent flagged
      when proposing this story, and its resolution once the user corrected/confirmed it —
      retained here rather than discarded once resolved, so the record of *why* the story reads
      the way it does survives past the gate that approved it.]
- **User Story `US-[DOMAIN]-002`:** ...

### 2.3. Core Functional Requirements
**MANDATE:** Each requirement MUST have a unique, machine-readable ID (e.g.,
`[PROJ]-FUNC-[DOMAIN]-NNNN`), be written clearly and concisely, be atomic (one testable
statement, not a bundle), and include a **Verification Protocol** (the specific test/artifact
that will prove completion). Each requirement MUST trace back to at least one User Story ID from
§2.2 — a functional requirement with no originating story is itself a smell to flag, per Step 3's
"Necessary" criterion (§3.3 below restates the full 9-criteria checklist this table is held to).

| Req ID | Description | Originating User Story | Verification Protocol |
| :--- | :--- | :--- | :--- |
| `[PROJ]-FUNC-[DOMAIN]-0001` | [Requirement Description] | `US-[DOMAIN]-001` | [e.g., Integration test asserting X given Y] |

### 2.4. Non-Functional Requirements (Quality Attribute Scenarios)
**MANDATE:** Non-functional requirements MUST be expressed as **Quality Attribute Scenarios**
(stimulus/response format), not bare statements — a bare statement like "the system must be
fast" is not verifiable on its own and fails the 9-criteria check at Step 3/5. Each scenario
MUST still carry a unique, machine-readable ID and a Verification Protocol, exactly as Core
Functional Requirements do; the scenario format below is a stricter shape for the Description
column, not a parallel or lesser requirement type.

**Scenario fields:**
- **Source:** what triggers the stimulus (a user, another system, an internal timer/condition,
  an attacker — see §4.8 Security Architecture for adversarial stimuli specifically).
- **Stimulus:** the concrete triggering condition or event.
- **Environment:** the system state/conditions under which the stimulus occurs (normal
  operation, degraded/partial-outage, peak load, etc.) — many quality requirements only hold,
  or only matter, under a specific environment, and leaving this implicit is a common source of
  an unverifiable NFR.
- **Artifact:** the part of the system the stimulus acts on.
- **Response:** the system's required behavior.
- **Response Measure:** the quantitative or otherwise objectively checkable bar the response
  must clear — this is what makes the scenario verifiable, and is what the Verification Protocol
  column directly tests against.

| Req ID | Quality Attribute | Source | Stimulus | Environment | Artifact | Response | Response Measure | Verification Protocol |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `[PROJ]-NFR-[DOMAIN]-0001` | [e.g., Performance] | [e.g., mmoClient] | [e.g., issues a query request] | [e.g., normal load, <1000 concurrent users] | [e.g., mmoRouter query endpoint] | [e.g., returns ranked results] | [e.g., P95 < 200ms] | [e.g., k6 load test scenario `query_latency`, threshold assertion] |

**Quality attribute categories to consider systematically** (not every category applies to every
project; state explicitly which are out of scope for this system rather than silently omitting
them): Performance/Latency, Throughput, Availability, Scalability, Security (see §4.8 for the
full threat-model treatment; high-level scenarios here, full catalog there), Reliability/Fault
Tolerance, Data Durability/Consistency, Observability, Maintainability/Modifiability,
Testability, Usability/Accessibility (for any UI-bearing component), Portability (for any
multi-target Rust component — native/WASM/mobile), Interoperability, Compliance/Regulatory.

### 2.5. Out-of-Scope Features
(Explicitly enumerate what this system does NOT do, especially anything a reader might
reasonably assume is in scope given the problem statement. Each out-of-scope item should state
*why* — deferred to a later version, owned by a different system, not validated as a real need —
distinguishing "we decided not to" from "we haven't decided yet," since the latter is an open
question that belongs in §7.3 Assumptions or as an explicit escalation, not a quiet omission.)

---

## 3. Acceptance Criteria & Traceability

**MANDATE:** This section MUST contain a subsection for each requirement from Section 2,
identified by its unique Requirement ID. Each acceptance criterion must be detailed,
unambiguous, and verifiable — this is the Step 4/5 output (Test Strategy Mapping and Verified
Requirements & V&V Protocol) recombined into the final specification.

### 3.1. The 9 Requirement Quality Criteria (Self-Check Reference)

Every requirement in §2.3 and §2.4 MUST satisfy all nine of the following before being
considered final. This table is the mechanical check Step 3/5 runs against every candidate
requirement; restated here so the finished specification carries its own quality bar rather than
requiring a reader to go find it in `DESIGN.md`.

| # | Criterion | Failure mode it catches |
|---|---|---|
| 1 | **Necessary** | The requirement doesn't trace to any real user story or stated goal |
| 2 | **Atomic** | The requirement bundles more than one independently testable statement |
| 3 | **Unambiguous** | The requirement admits more than one reasonable implementation reading |
| 4 | **Verifiable** | No test or artifact could prove or disprove this requirement as stated |
| 5 | **Feasible** | The requirement is not achievable given the stated technology/constraints |
| 6 | **Complete** | The requirement is missing a condition, exception, or boundary case it implies |
| 7 | **Consistent** | The requirement contradicts another requirement elsewhere in this document |
| 8 | **Design-independent** (where applicable) | The requirement specifies *how* when it should specify *what* — except where the source decision genuinely *is* the implementation (e.g., "MUST use Argon2id"), in which case state explicitly that design-independence is intentionally waived for this requirement and why |
| 9 | **Traceable** | The requirement has no stable ID, or no link back to its originating story/decision |

### 3.2. Test Case Catalog

The Step 4 output (Test Strategy Mapping): every Test Case referenced anywhere in this document
(by the Verification Protocol columns in §2.3/§2.4, and by the Traceability Matrix in §3.4) MUST
be defined here first — a `TEST-[NNN]` ID used in a Verification Protocol column with no
corresponding entry here is a Traceable-criterion failure (§3.1, criterion 9) and a finding
Step 7's audit MUST catch.

**MANDATE:** Each Test Case MUST have a unique, stable ID (`TEST-[NNN]`), trace to at least one
Requirement ID it verifies, state its type (unit / integration / system / acceptance — informing
the Development Plan's §9 Test Strategy & Plan split), and state the Definition of Done this
specific test enforces — distinct from, and more concrete than, the parent requirement's
Verification Protocol one-liner.

| Test ID | Verifies Req ID(s) | Type | Test Description | Definition of Done |
| :--- | :--- | :--- | :--- | :--- |
| `TEST-001` | `[PROJ]-FUNC-[DOMAIN]-0001` | [Unit / Integration / System / Acceptance] | [What the test does, concretely — inputs, action, expected outcome] | [The specific pass/fail condition, e.g. "assertion X holds for inputs A, B, and the boundary case C"] |

### 3.3. Requirement Smells (Rust-Specific)

Beyond the 9 generic criteria, requirement language that does not map cleanly onto Rust's type
system is flagged as a smell rather than passed through as-is, per this project's Rust-specific
process constraints:

| Smell | Why it's a smell in Rust | Preferred reframing |
|---|---|---|
| Nullable-value language ("may be empty," "if present") without specifying which case is absent vs. zero-valued | Rust has no implicit null; this must resolve to a deliberate `Option<T>` (genuinely absent) vs. a default/sentinel value (present but empty) | State explicitly: "returns `None` when X" vs. "returns an empty `Vec` when X" |
| Inheritance/"is-a" framing ("Component B extends Component A's behavior") | Rust has no implementation inheritance; this must resolve to trait composition, delegation, or an explicit enum variant | State explicitly which trait(s) are implemented/composed, or which enum models the variant behavior |
| Exception/"throws" framing ("the operation throws an error if X") | Rust has no exceptions; this must resolve to a `Result<T, E>` and a specific error variant | State explicitly: "returns `Err(SpecificErrorType::Variant)` when X" |
| Unbounded/open-ended "and so on" enumeration of variants | Rust's exhaustive `match` requires every variant to be named; an open-ended requirement can't compile to an exhaustive type | Either enumerate every variant explicitly, or explicitly mark the requirement as needing an extensible (non-exhaustive) design and say so |
| Implicit shared-mutable-state language ("the cache is updated by") without stating ownership | Rust requires an explicit answer to who owns the data and how concurrent access is mediated (`Arc<Mutex<_>>`, message-passing, etc.) | Resolve to an explicit ownership/concurrency statement, or flag as a §4.7/§4.9 architectural decision still pending |

### 3.4. Traceability Matrix

The master cross-reference: every requirement, the component(s) that implement it, the test(s)
that verify it, and the artifact(s) produced as evidence. This table is what Step 7's audit
checks for completeness (every requirement has a row; no row points at a component or test that
doesn't exist elsewhere in this document or the Development Plan).

| Req ID | Component / Unit | Test ID | Verification Artifact |
| :--- | :--- | :--- | :--- |
| `[PROJ]-FUNC-[DOMAIN]-0001` | `[component]` | `TEST-001` | `[artifact reference]` |

### 3.5. Acceptance Criteria Detail

For requirements whose Verification Protocol (§2.3/§2.4) needs more than a one-line test
reference to be unambiguous — multi-step scenarios, specific input/output fixtures, specific
failure-mode assertions — expand here, one subsection per requirement, keyed by Req ID.

#### `[PROJ]-FUNC-[DOMAIN]-0001`
(Detailed acceptance criteria: preconditions, steps, expected postconditions, specific edge
cases this requirement must handle.)

---

## 4. System Architecture (ISO/IEC/IEEE 42010 Viewpoints)

This section is the Step 6 output: the Final Architecture Synthesis. Per the governing process,
formal notation (SysML-style block/sequence diagrams, or equivalent) is used where it clarifies
non-obvious structure, and prose is used where formal notation would be decorative rather than
informative — each subsection below states briefly which choice was made and why, so the choice
is visible and overridable rather than silent.

### 4.1. Architectural Goals & Constraints
(Restate the architectural-specific goals from §1.4, plus any constraint that shapes the
architecture itself — distinct from the project-level constraints in §7, which may be business
or process constraints not specific to the architecture. Cross-reference: hard Rust/WASM
constraints from §4.9 and the Technology Stack in §6 often originate as architectural goals
stated here, e.g. "support multithreading on both native and WASM targets.")

### 4.2. Architectural Principles
(The standing principles this architecture is built on — e.g. "single public entry point,"
"hexagonal architecture / ports and adapters," "default-deny authorization," "Kafka for events
only, never for request-response." State each principle once here; every viewpoint below should
be traceable back to one or more of these principles rather than introducing new unstated ones
mid-document.)

### 4.3. Solution Strategy
(The high-level "why this approach" — the small number of fundamental decisions that shape
everything else, stated before the detailed viewpoints so a reader has the frame before the
detail. Typical content: the chosen architectural style/pattern and why, how the top quality
goals from §2.4 are to be achieved structurally, and the most important technology decisions
from §6 stated at the level of rationale rather than the full stack listing. Keep this section
short — a few paragraphs to a page — its job is orientation, not completeness; completeness
lives in the viewpoints below and in §6.)

### 4.4. Functional View
- **Description:** Describes the system's functionality and its responsibilities — what the
  system does, decomposed into components/services and their collaborations.
- **Diagrams:** System Context (C4 Level 1), Container/Component (C4 Level 2/3), Sequence
  Diagrams for any non-obvious multi-component interaction. State explicitly, per diagram,
  why a formal diagram was used here rather than prose (per this section's opening note).
- **Component Responsibility Summary:** for each component named in the diagrams, a short table
  or list stating its core responsibility and, explicitly, what it is **not** responsible for
  (mirroring the "NOT Responsible For" pattern that prevents responsibility drift between
  closely related components — this is especially important where two components could
  plausibly each claim a responsibility, e.g. a gateway component and an orchestration
  component that are deployed together).
- **Component Responsibility Collaborator (CRC) Cards:** for components whose collaboration
  pattern is non-obvious from the diagrams alone, a CRC card per component: Responsibility /
  Collaborators / Notes.

### 4.5. Information View (Data Dictionary & Models)
- **Description:** Describes the information handled by the system and its structure — every
  significant data entity, its attributes, and the constraints/invariants that hold over it.
- **Data Dictionary:**

| Entity | Attribute | Type | Description | Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `[Entity]` | `[attribute]` | `[Rust type, e.g. Ulid, Option<String>]` | [description] | [e.g., Primary Key, Foreign Key, NOT NULL, unique index] |

- **Data Models:** Logical ERDs, JSON Schemas, Protobuf definitions, and/or Rust type
  definitions (`struct`/`enum`) where the type definition itself is the clearest specification.
  State explicitly the canonical/authoritative representation when more than one exists (e.g.
  "the Rust `enum` is authoritative; the Protobuf message is a wire-format projection of it" or
  vice versa) — an unstated authority between two representations of the same entity is a
  Consistency failure under the 9 criteria (§3.1).
- **Database / Persistence Schema:** for any component with a durable store, the schema MUST be
  specified here at the level a migration could be written directly from it: table/collection
  names, column/field names and types, indexes (and what query pattern each index serves), and
  migration ordering if the schema evolves in numbered steps. If more than one storage
  technology is used for different data (e.g. a relational store plus a graph store plus a
  vector store for the same logical entity), state explicitly which technology is the source of
  truth and how consistency across the others is maintained (synchronous write, eventual
  consistency with reconciliation, etc.) — this is itself an architectural decision and should
  also appear in §4.11 ADRs if it was non-obvious.
- **Migration Safety & Rollback Policy:** distinct from migration *ordering* above — this
  states what happens when a deployed service version needs to be rolled back *after* a
  migration has already run against live data, which forward-ordering alone does not answer.
  State explicitly:
  - **Destructive-change policy:** whether destructive migrations (dropping/renaming a
    column, narrowing a type, removing a table) are permitted directly, or whether they
    require a two-step deprecate-then-remove pattern (e.g. add the new column, dual-write
    for one release, backfill, only then drop the old column in a later release) — and which
    of these this project defaults to as a hard rule versus a per-migration judgment call.
  - **Backward compatibility window:** how many releases (or how long) a schema must remain
    readable/writable by the *previous* service version, so a rollback of the service to N-1
    doesn't immediately break against a schema already migrated to N.
  - **Rollback procedure:** the concrete steps to roll back a bad deploy whose migration has
    already run — whether this means a compensating down-migration, a feature flag that
    disables the new code path while leaving the schema forward-migrated, or another
    documented mechanism. State which approach is the project's default and why.
  - **Irreversible migrations:** any migration that cannot be safely rolled back at all (e.g.
    one involving an unrecoverable data transformation) MUST be flagged explicitly as such
    here, with the mitigation being deploy-time caution (e.g. a mandatory backup/snapshot
    step) rather than an assumed rollback path that doesn't actually exist.
- **Data Lifecycle & Retention:** for each entity that has a retention policy, tiering strategy
  (hot/warm/cold), or deletion/anonymization requirement, state it here or cross-reference
  §4.8's data governance treatment if retention is itself a security/compliance concern.

### 4.6. Deployment View
- **Description:** Describes the physical/runtime environment(s) in which the system is
  deployed — for a multi-target Rust project, this includes every target (native service
  deployment, WASM hosting environment, mobile/desktop shell distribution), not just the
  primary one.
- **Diagrams:** Physical View (Deployment Diagram), Network Topology, and — for any WASM web
  component — the hosting/serving topology (CDN, origin server headers, etc.).
- **Per-Target Deployment Detail:** one subsection per deployment target.
  - **Native service target(s):** runtime environment (e.g. container orchestrator), resource
    allocation, scaling strategy, network exposure (which services are public vs.
    internal-only — state this as an explicit, enforced rule if any service must never be
    publicly reachable, mirroring how a single-public-entry-point architecture would state it).
  - **WASM web target(s), if any:** hosting requirements, and explicitly, **COOP/COEP header
    feasibility** if any component uses or implies `SharedArrayBuffer`-based threading (see
    §4.9 for the concurrency-model decision this depends on) — state which deployment
    environments can guarantee `Cross-Origin-Opener-Policy: same-origin` and
    `Cross-Origin-Embedder-Policy: require-corp`, and the documented fallback behavior
    (typically: message-passing-only concurrency, no shared-memory threading) for any
    environment that cannot.
  - **Mobile/desktop shell target(s), if any:** distribution mechanism, platform-specific
    constraints, and how the shell's Rust core relates to the native/WASM components above
    (shared crate, FFI boundary, etc.).
- **Configuration Reference:** see §11.3 Appendix for the full configuration schema; this
  subsection states only the deployment-relevant high points (which values are
  environment-specific, which are secrets sourced from a secret manager rather than
  config files, etc.).

### 4.7. Process View (Runtime/Concurrency)
- **Description:** Describes the system's runtime behavior, concurrency, and synchronization —
  what executes concurrently, what shares state, and how access to shared state is mediated.
- **Concurrency Model, per component:** state explicitly, per component, which concurrency
  model applies:
  - Native Rust components: async runtime (e.g. Tokio) task model, thread-pool sizing strategy
    if relevant, and which types cross task/thread boundaries (with their `Send`/`Sync`
    justification — see §4.9).
  - WASM components: message-passing via Web Workers (the default/preferred model per this
    project's standing preference for multithreading wherever feasible) vs.
    `SharedArrayBuffer`-based shared-memory threading (only where message-passing is
    demonstrably insufficient, and only with the COOP/COEP feasibility from §4.6 confirmed).
    State explicitly: **no component combines two distinct Worker-pool-owning threading crates
    without an explicit decision recorded here and in §4.11 ADRs** — this is a hard
    process-level check, not a style preference, since two independent pool owners can silently
    oversubscribe available hardware threads.
- **Sequence Diagrams:** for any runtime flow whose timing, ordering, or failure-mode behavior
  is non-obvious from the Functional View's structural diagrams alone (e.g. a multi-step
  request that fans out to several components and must handle partial failure).
- **Synchronization & Shared State:** for every piece of state shared across concurrent
  execution contexts, state explicitly: the ownership model (`Arc<Mutex<_>>`,
  actor/message-passing exclusive ownership, lock-free structure, etc.), and why that model was
  chosen over the alternatives — an implicit "the cache is updated by" without this is a
  Requirement Smell per §3.3 if it appears in a requirement, and an incomplete architectural
  decision if it appears only here.

### 4.8. Security Architecture & Threat Model
- **Description:** The system's security posture, expressed both as standing principles (defense
  in depth layering, default-deny posture, etc.) and as a structured, per-component threat
  catalog. This subsection is mandatory for any system handling authentication, authorization,
  user data, or any externally-reachable surface — which is to say, nearly every system; state
  explicitly if any component is judged to have no meaningful attack surface and why.
- **Defense-in-Depth Layers:** state the layer model this architecture uses (e.g. Network →
  Authentication → Authorization → Application → Data, or an equivalent the project prefers),
  and which architectural mechanism satisfies each layer.
- **STRIDE Threat Model:** for every component with an externally-reachable or
  trust-boundary-crossing surface, a structured threat catalog using the STRIDE categories
  (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of
  Privilege). Each row is a specific, concrete threat — not a restatement of the category name —
  with its mitigation and an honest statement of residual risk (a mitigation that fully
  eliminates a threat is rare; say so when residual risk is genuinely zero, and say what remains
  when it isn't).

| Threat ID | Component | STRIDE Category | Threat Description | Mitigation | Residual Risk |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `THREAT-[DOMAIN]-001` | `[component]` | Spoofing | [e.g., attacker forges a token claiming another user's identity] | [e.g., RS256/ES256 signature validation against KMS-sourced keys, audience claim restricted to known services] | [e.g., low — requires private key compromise] |

- **Trust Boundaries:** explicitly enumerate every trust boundary in the system (e.g. public
  internet ↔ gateway, gateway ↔ internal service mesh, service ↔ database) — the STRIDE catalog
  above should be organized so every threat is anchored to a specific boundary crossing, not
  treated as boundary-less.
- **Data Governance (write-path and read-path):** for any system that persists user-supplied or
  otherwise untrusted data, state explicitly what is checked *before* persistence (not just at
  read/query time) — e.g. secret-material detection, PII classification — and what the failure
  mode is when a write is rejected on these grounds (a distinct error code from a generic
  authorization failure is preferred, since the failure is about content, not identity).
- **Secrets & Key Management:** how cryptographic keys, credentials, and other secrets are
  sourced, rotated, and scoped — cross-reference §6 for the specific KMS/secrets-manager
  technology and §4.9 for any Rust-specific memory-safety handling (zeroization, etc.) of
  secret material in process memory.

### 4.9. Rust-Specific Architectural Conventions
- **Description:** Architectural decisions specific to Rust's type system and memory model that
  apply across components, stated once here rather than re-derived ad hoc in each viewpoint
  above. Every other viewpoint may reference this subsection rather than restating these
  conventions locally.
- **Error Handling Strategy:** the project's `Result<T, E>` / error-type conventions — whether
  errors are modeled as a layered hierarchy (e.g. a domain-layer error type distinct from an
  API-layer error type, with an explicit mapping between them), which crate (if any) is used for
  error derivation/boilerplate, and the rule for what detail is permitted to cross a layer
  boundary (e.g. internal error detail must never reach an external API response body — state
  this as a hard rule if it applies, since it is a common security-relevant requirement).
- **Ownership & Lifetime Conventions at Component Boundaries:** how data crossing a
  component/crate boundary is owned — by value, by reference with an explicit lifetime, or
  behind a shared-ownership pointer (`Arc`) — stated as a general convention so individual
  interface specifications in §4.10 ICD don't have to re-derive it per interface. State
  explicitly any case where a non-owning reference crosses an `async` boundary and how its
  lifetime is guaranteed valid for the duration needed.
- **Concurrency Primitives & `Send`/`Sync` Policy:** which concurrency primitives are the
  project's default choice (channels vs. shared-state locking, which lock type, etc.) and the
  policy for types that must be `Send`/`Sync` to cross the boundaries described in §4.7's
  Concurrency Model — including the project's WASM/Web-Worker-first stance and when (if ever)
  `SharedArrayBuffer`-based shared memory is an approved exception (see §4.6 and §4.7 for the
  feasibility/decision detail this connects to).
- **Trait-Based Composition / Hexagonal Architecture Conventions:** the project's port-and-adapter
  pattern, if used — how domain logic crates are kept free of I/O dependencies, how this is
  enforced (e.g. a CI dependency-graph check), and the naming/module convention for ports
  (traits) vs. adapters (concrete implementations).
- **`unsafe` Code Policy:** whether `unsafe` is permitted anywhere in this codebase, and if so,
  under what review/justification standard (e.g. every `unsafe` block requires a `// SAFETY:`
  comment explaining the invariant being upheld, and a named reviewer sign-off in code review).
  If `unsafe` is categorically forbidden (e.g. via `#![forbid(unsafe_code)]`), state that
  explicitly as a hard constraint, cross-referenced from §7.1.
- **WASM-Specific Conventions, if applicable:** target triple(s) supported
  (`wasm32-unknown-unknown`, etc.), the build tool used to produce the deployable artifact, and
  any API surface that is conditionally compiled per target (`#[cfg(target_arch = "wasm32")]`)
  — state the convention for keeping conditional-compilation forks from silently diverging in
  behavior (e.g. a shared trait both targets implement, tested on both targets in CI).

### 4.10. Interface Control Document (ICD)
- **Description:** Formalized contracts for all internal and external system interfaces — the
  binding specification an implementer can build directly against without needing to infer
  intent from the Functional View's prose.
- **Internal APIs:** for each internal service-to-service interface — endpoint/method
  signatures, request/response schemas (cross-referencing §4.5's Data Dictionary for shared
  types rather than redefining them), synchronous vs. asynchronous communication pattern (and
  why, per the project's standing communication philosophy from §4.2 if one is stated there),
  error responses and their meaning, and any ownership/ID-stability conventions for evolving the
  interface (e.g. an API versioning policy).
- **External Integrations:** for each external system/third-party API this system depends on or
  exposes itself to — the contract as the external party defines it (linked, not
  reproduced, where it's a standard like OAuth2/OIDC), and this system's specific usage of it
  (which grant types, which scopes, which endpoints).
- **Event/Message Interfaces:** for any message-bus/event-streaming interface (Kafka topics or
  equivalent) — topic/channel ownership (which component produces, which consumes, who owns the
  schema), message schema, delivery semantics (at-least-once, exactly-once, ordering
  guarantees), and dead-letter/error-handling behavior for malformed messages.

### 4.11. Architecture Decision Records (ADRs)
- **Description:** Every architecturally significant decision — expensive to reverse, affecting
  more than one component, or chosen over a credible alternative — recorded as a discrete,
  ID-stable record, not buried in viewpoint prose. A reader should be able to find *why* a
  decision was made without re-reading the full viewpoint that resulted from it.
- **ADR format (mandatory fields per record):**

#### `ADR-[NNN]`: [Decision Title]
- **Status:** Proposed | Accepted | Superseded (by `ADR-[NNN]`) | Deprecated
- **Context:** What problem or forcing function led to this decision being needed.
- **Decision:** The decision, stated plainly and specifically.
- **Consequences:** Both positive and negative — an ADR that lists only benefits has not
  honestly considered the tradeoff.
- **Paths Not Taken:** The credible alternative(s) considered and why each was rejected — this
  is what distinguishes an ADR from a simple decision log entry, and is often the most valuable
  part of the record for a future reader wondering "why didn't they just do X."

### 4.12. Technical Non-Functional Requirements
(Cross-reference: the primary NFR treatment is §2.4's Quality Attribute Scenarios. This
subsection is for any NFR that is purely internal/technical — not user-facing — and therefore
doesn't naturally fit a stimulus/response scenario framed around an external actor, e.g. build
time budgets, binary size constraints for a WASM target, or CI pipeline duration limits. Keep
this subsection short; most NFRs belong in §2.4.)

### 4.13. User Experience (UX) & User Interface (UI) Design
(For any system with a human-facing interface — applies to web/desktop/mobile components, not
typically to a pure backend service. Cover: information architecture, key user flows
cross-referenced to the User Stories in §2.2, accessibility requirements, and — for Rust/WASM UI
components — which UI framework/crate is used and any architectural implication of that choice,
e.g. virtual-DOM diffing strategy. If this system has no human-facing interface, state that
explicitly rather than leaving this section silently empty.)

### 4.14. Logging and Monitoring
- **Logging Strategy:** This subsection must detail the system-wide logging strategy.
  - **Log Levels:** The design must incorporate a standard 5-level logging system: Error, Warn,
    Info, Debug, Trace.
  - **Default Level:** The default logging level at application launch must be `INFO`.
  - **Configuration:** The logging level must be configurable at launch via environment
    variables or configuration files.
  - **Structured Logging Conventions:** field-naming conventions, correlation/trace ID
    propagation strategy, and — cross-referencing §4.8 — the redaction policy for any field that
    could carry secret material (a mandatory subsection for any system handling credentials).
- **Monitoring Strategy:** Outline the approach for monitoring the system's health and
  performance, including key metrics to track (per component, the metrics that map back to
  the Response Measures in §2.4's Quality Attribute Scenarios — a metric with no corresponding
  scenario, or a scenario with no corresponding metric, is a Completeness gap worth flagging),
  alerting thresholds, and distributed tracing strategy if the system spans multiple components.


---

## 5. External Interfaces & Integrations

> Distinct from §4.10's Interface Control Document, which is the formal, binding contract
> detail. This section is the higher-level inventory and rationale — *what* external systems
> this system touches and *why* — with §4.10 as the place a reader goes for the exact schema.

### 5.1. External System Interfaces
(Every external system this system sends data to or receives data from, at an inventory level —
one entry per external system, cross-referencing the detailed contract in §4.10 where one
exists.)

### 5.2. Third-Party Integrations
(Every third-party service/library dependency that constitutes an integration rather than a
library dependency — an identity provider, a payment processor, an LLM API provider. Distinct
from §6's Technology Stack, which covers libraries/crates compiled into the system; this section
covers services the running system calls over the network.)

### 5.3. API Specifications (External)
(For any API this system exposes to external (non-internal-service-mesh) callers — the public
contract, versioning policy, and deprecation policy. Cross-reference §4.10 for the same level of
schema detail the internal APIs receive; external APIs deserve at least that level of rigor
since external callers can't be coordinated with as easily as internal ones when a breaking
change is needed.)

---

## 6. Technology Stack & Dependencies

> Promoted to a top-level section (rather than a single bullet under System Architecture)
> because dependency selection is itself an architectural decision with real constraints — see
> the project's dependency-preference policy (e.g. `agents/PREFERRED_DEPENDENCIES.md` if this
> repository has one) for the binding rules this section's choices must satisfy.

### 6.1. Core Technologies
(Language version/edition, async runtime if applicable, web framework, and any other foundational
technology used across most/all components — the equivalent of mmoMental-style "Core
Technologies" table: Category | Technology | Version | Rationale.)

| Category | Technology | Version | Rationale |
|---|---|---|---|

### 6.2. Per-Component Technology Stack
(For a multi-component project, the stack actually differs per component — mirroring the
Development Plan's §2 Technology Stack table. State it here as the architectural source of
truth; the Plan's §2 should be derived from this, not an independent decision.)

| Component | Crate type / target | Key crates / frameworks | Rationale |
|---|---|---|---|

### 6.3. External Dependencies (Infrastructure)
(Databases, message brokers, caches, object storage, and any other infrastructure dependency —
mirroring mmoMental/mmoSecure-style "External Dependencies" tables: Component | Technology |
Notes.)

| Component | Technology | Notes |
|---|---|---|

### 6.4. Dependency Governance
(How new dependencies are vetted before being added — license compatibility, supply-chain
security scanning (`cargo audit`/`cargo deny` or equivalent), and the approval tiering if this
project has a preferred/forbidden/requires-approval dependency policy. State explicitly which
CI gates enforce this, cross-referencing the Development Plan's Environment & Prerequisites
Setup section for the tooling this requires.)

### 6.5. CI/CD Pipeline
> Referenced piecemeal elsewhere in this document (dependency-graph checks, MSRV enforcement,
> doctest gates) but stated here as its own coherent subject: how code actually gets from a
> merged change to running in production, and what happens when that goes wrong. Without this
> stated explicitly, "how does this get deployed" is left to be discovered ad hoc once
> Development begins — exactly the kind of late discovery this document exists to prevent.

- **Pipeline Stages:** the concrete stages a change passes through (e.g. lint → build → unit
  test → integration test → security/dependency scan → package → deploy-to-staging →
  smoke-test → promote-to-production), stated as an ordered list with, for each stage, what
  gates passage to the next (a required check, a required approval, or fully automated).
- **Trigger Conditions:** what triggers each stage — e.g. every PR triggers lint/build/test;
  merge to the main branch triggers a staging deploy; a tagged release triggers a production
  promotion. State explicitly whether any stage requires manual approval (a human "promote to
  production" click) versus being fully automated, and who is authorized to approve it.
- **Environment Promotion Strategy:** the concrete environments code passes through (e.g.
  dev → staging → production, or a canary/blue-green scheme) and what distinguishes
  "promoted" from "deployed" if they differ — e.g. a canary deploy that's live but receiving
  no production traffic until explicitly promoted.
- **Bad-Deploy Rollback Procedure:** distinct from §4.5's Migration Safety & Rollback Policy,
  which covers *data* rollback — this covers rolling back the *running service version*
  itself when a deploy is bad (crashing, error-rate spike, etc.) without yet implicating a
  migration: state whether rollback means redeploying the previous container image/binary, a
  traffic-shifting mechanism (canary/blue-green cutover reversal), or another concrete
  mechanism — and the maximum acceptable time-to-rollback as a stated target, since "we can
  roll back eventually" is not the same operational guarantee as "we can roll back in under
  N minutes."
- **Per-Target Build Artifacts:** for a multi-target project (native binary, WASM bundle,
  mobile/desktop shell), state explicitly what artifact each pipeline run produces for each
  target, and whether all targets are built/deployed on the same cadence or independently
  (e.g. the WASM frontend may deploy more frequently than a native backend it depends on —
  state the compatibility contract between them if their release cadences diverge).
- **Required CI Gates (Consolidated):** a single list cross-referencing every CI-enforced gate
  named elsewhere in this document, so a reader doesn't have to hunt through every section to
  assemble the full picture: dependency-graph/license check (§6.4), vulnerability scan (§6.4),
  MSRV-pinned build (§10.2), doctest execution (§10.3), and any project-specific gate not
  already covered above.

---

## 7. Constraints & Assumptions

### 7.1. Technical Constraints
(Hard technical limits — language/platform constraints, the `unsafe` policy from §4.9 restated
as a constraint if it's a hard rule, performance ceilings imposed by a dependency, deployment
environment limitations such as the COOP/COEP constraint from §4.6.)

### 7.2. Business Constraints
(Budget, timeline, regulatory/compliance obligations, organizational policy constraints not
specific to the architecture itself.)

### 7.3. Assumptions
**MANDATE:** every assumption listed here MUST be one that, if wrong, would require revisiting
an already-approved part of this document — assumptions with no real consequence if wrong don't
belong here. Each assumption should ideally trace back to where it was first flagged during the
gated workflow (Step 1's boundary assumptions, Step 3's requirement-completion assumptions,
etc.), preserving the "flagged, not silently resolved" record the governing process requires.

| Assumption ID | Assumption | Originating Step/Section | Consequence if Wrong |
|---|---|---|---|

### 7.4. Dependencies
(Cross-team, cross-system, or cross-project dependencies this architecture's delivery relies on
— distinct from §6's technology dependencies. E.g. "depends on Team X's service exposing
endpoint Y by date Z.")

---

## 8. Risks & Technical Debt

> Distinct from the Development Plan's §7 Risk Management, which covers *delivery* risk
> (sequencing, session-sizing, tooling availability). This section covers *architectural* risk —
> known weaknesses, technical debt knowingly accepted, and scenarios this architecture is not
> well-suited to — independent of how the implementation work is sequenced.

### 8.1. Known Architectural Risks

| Risk ID | Description | Impact | Likelihood | Mitigation / Monitoring Strategy | Related ADR (if any) |
|---|---|---|---|---|---|

### 8.2. Accepted Technical Debt
(Anywhere this architecture deliberately chooses a simpler or less robust approach now, with an
explicit plan (or explicit decision not to have one yet) for revisiting it — e.g. "single-region
deployment accepted for v1; multi-region federation deferred, see ADR-`[NNN]` and §9's roadmap
for when this is expected to be revisited." Undocumented technical debt is, by definition, not
capturable here — the point of this subsection is to make debt that *is* known a visible,
tracked decision rather than tribal knowledge.)

### 8.3. Scenarios This Architecture Does Not Handle Well
(Honest statement of the architecture's weak points — load patterns, failure modes, or future
requirements this design would need to be revisited for. This is not the same as §2.5
Out-of-Scope Features; out-of-scope features were never intended to be handled, while this
subsection covers things arguably in scope that this architecture handles poorly or not at all.)

---

## 9. Implementation Roadmap & Build Order

> This section is what Step 8 (Development Plan & Checklist Generation) consumes most directly —
> the Development Plan's Phase Index should be derivable from this roadmap without inventing new
> sequencing logic. Keeping the *sequencing rationale* here, in the architecture document, and
> the *session-sized task decomposition* in the Plan keeps the two documents from duplicating
> judgment calls: this section answers "in what order must this be built, and why," and the Plan
> answers "how is that order broken into single-session phases."

### 9.1. Sequencing Principles
(The dependency logic that determines build order — e.g. "domain logic before infrastructure
adapters," "identity before authorization before accounting" — stated as principles before the
concrete step list, so the *reasoning* survives even if the concrete list needs to be revised.)

### 9.2. Build Order

| Step | Deliverable | Component(s) | Exit Criteria |
|---|---|---|---|

### 9.3. Phased Delivery Milestones
(If this system is delivered in stages — an MVP exit criterion, a beta milestone, a GA
definition-of-done — state each milestone and which Build Order steps it requires. This is the
architecture-level statement of "done"; the Development Plan's own Plan-Level Definition of Done
governs the *planning documents'* completeness, a related but distinct concept.)

---

## 10. Public API & Framework Consumer Contract

> Distinct from §4.10's Interface Control Document, which governs interfaces *within* this
> system (service-to-service, component-to-component). This section governs the contract
> this system exposes to **external code that depends on it as a library/framework** —
> downstream crates calling `pub` items in this codebase. This category of contract doesn't
> exist for a typical internal service and is easy to omit by analogy with application
> architecture templates, but is load-bearing for a framework: a breaking change here breaks
> every consumer's build, not just one deployment.

### 10.1. Semantic Versioning & Breaking-Change Policy
**MANDATE:** state explicitly, not by reference to "standard SemVer," since the *specific*
rules that count as breaking for this project's actual API surface are what an implementer
needs, not the general SemVer spec:
- **What counts as a breaking change for this project specifically:** beyond the SemVer
  basics (removing/renaming a `pub` item, changing a function signature, adding a required
  field to a `pub` struct) — state the project's own judgment calls, e.g. whether adding a
  new variant to a `pub` non-exhaustive enum is breaking (it generally isn't, if the enum is
  marked `#[non_exhaustive]` — state whether that attribute is the project's standing policy
  for all public enums, and if not, why not), whether tightening a previously-permissive
  trait bound is breaking, and whether a change to this crate's MSRV (below) itself counts as
  a breaking change for the project's own versioning purposes.
- **Pre-1.0 policy, if applicable:** if this project is pre-`1.0.0`, state explicitly what
  stability guarantee (if any) applies to `0.x` releases — many Rust projects treat a `0.x`
  minor bump as carrying the same weight as a `1.x` major bump; state whether this project
  follows that convention or a different one, since the difference materially affects how
  consumers pin their dependency.
- **Deprecation policy:** the minimum notice period (in releases or time) a `pub` item must
  carry `#[deprecated]` before removal, what the deprecation message is required to state
  (the replacement API, at minimum), and whether deprecated items are tracked anywhere
  consumers can see what's coming (a changelog section, tracking issue convention, etc.).
- **Changelog convention:** which format/tool this project uses (e.g. Keep a Changelog,
  `cargo-release`-generated, conventional commits driving an automated changelog) and where
  it lives — this is what a consumer actually reads before upgrading, and an unstated
  convention here means every release reinvents the format.

### 10.2. Minimum Supported Rust Version (MSRV) Policy
- **Current MSRV:** the specific Rust version this project commits to supporting, stated
  explicitly (not "latest stable") so a consumer on an older toolchain knows immediately
  whether this framework is usable for them.
- **MSRV bump policy:** under what conditions the MSRV is allowed to increase (e.g. only on a
  minor/major version bump, never on a patch release), how far behind the latest stable
  release the project commits to staying (e.g. "supports the last N stable releases"), and how
  an MSRV bump is communicated (changelog entry, is it itself treated as a breaking change per
  §10.1 — state this explicitly, since projects differ on this and an unstated answer is a
  real consumer-facing ambiguity).
- **MSRV enforcement:** how the project verifies it actually still builds on its stated MSRV
  (e.g. a dedicated CI job pinned to that toolchain version — cross-reference §6.5 CI/CD
  Pipeline) rather than letting MSRV silently drift upward as new code accidentally relies on
  a newer language feature.

### 10.3. Consumer-Facing Documentation & Examples Convention
- **Rustdoc conventions:** the project's standing requirement for `pub` item documentation —
  e.g. every `pub fn`/`pub struct`/`pub trait` MUST have a doc comment, MUST include a runnable
  example (` ```rust ` doctest) where the API's usage isn't self-evident from its signature
  alone, and MUST document panics/errors explicitly (`# Panics` / `# Errors` sections) rather
  than leaving failure behavior to be discovered by the consumer at runtime.
- **Doctests as a verification gate:** state explicitly whether `cargo test --doc` (or
  equivalent) is a required, CI-enforced gate (cross-reference §6.5) — a framework whose
  documented examples don't actually compile/run is a common and entirely preventable
  consumer-trust failure.
- **"Getting Started" / example-crate convention:** whether this project maintains a
  separate, runnable example (an `examples/` directory, a dedicated starter-template crate,
  or both) distinct from inline rustdoc examples, and what level of completeness that example
  is held to (a toy snippet vs. a realistic minimal application).
- **API surface review for new public items:** the project's process (if any) for reviewing
  whether a new `pub` item should actually be public before it ships — once something is
  public, removing it is a breaking change per §10.1, so the cost of an unreviewed `pub` is
  asymmetric; state whether this is a checklist item in code review, an explicit ADR-worthy
  decision (cross-reference §4.11) for any non-trivial new public surface, or left to reviewer
  judgment with no stated policy.

---

## 11. Appendices

### 11.1. Glossary of Terms
(The durable, project-lifetime term registry — distinct from §1.5's document-local definitions.
Every term introduced anywhere in this document that another document (the Development Plan, a
future architecture revision) will need to use consistently belongs here.)

| Term / Acronym | Meaning |
|---|---|

### 11.2. Detailed Diagrams
(Any diagram too large or detailed to inline in its natural viewpoint section above — full-page
sequence diagrams, complete ER diagrams, etc. — collected here and cross-referenced from the
relevant viewpoint.)

### 11.3. Configuration Reference
(The concrete, complete configuration schema for the system — default values, environment
variable override convention, and an explicit statement of which values are non-secret defaults
appropriate for a config file versus which MUST be sourced from a secrets manager and never
appear in version control. This is the artifact the Development Plan's §5 Dev/Test Configuration
section derives its local/dev configuration from, explicitly distinguishing dev/test values from
this section's production schema.)

```toml
# Illustrative structure — replace with the project's actual configuration schema.
[server]
host = "0.0.0.0"
port = 8080
```

### 11.4. Decision Log Index
(If ADRs in §4.11 grow numerous, an index table here — ID, Title, Status, one-line summary — so
a reader can scan for relevance before opening the full record.)

| ADR ID | Title | Status |
|---|---|---|

---

## Appendix R - Revision History
| Version | Date | Author | Changes |
|---|---|---|---|
| 0.2.00  | 2026-06-20 | Claude | Added §6.5 (CI/CD Pipeline) and a new top-level §10 (Public API & Framework Consumer Contract: SemVer/breaking-change policy, MSRV policy, consumer-facing rustdoc/examples conventions) per gap-finding review. Renumbered former §10 Appendices to §11 (and its subsections 10.1–10.4 to 11.1–11.4) to accommodate the new top-level section without disturbing §1–§9's existing numbering or any of their internal cross-references. Added a Migration Safety & Rollback Policy subsection to §4.5, distinct from the existing migration-ordering content, covering rollback of a deployed service version after a migration has already run. |
| 0.1.01  | 2026-06-20 | Claude | Fixed a stale `development_plan_template_complex.md` filename reference in the header note (actual companion file is `development_plan_template.md`); updated the same note's mention of a "Requirements & Traceability Backfill artifact" to reflect that no separate traceability document is produced — §3 of this document is the sole, authoritative traceability source, consumed directly by the Plan template's §0. |
| 0.1.00  | YYYY-MM-DD | [author] | Initial creation of complex/multi-domain Rust variant, extending the base `architecture_specification_template.md` with: Solution Strategy, Security Architecture & Threat Model (STRIDE), Rust-Specific Architectural Conventions, promoted Technology Stack & Dependencies, promoted Architecture Decision Records with mandatory Paths-Not-Taken field, Risks & Technical Debt, Implementation Roadmap & Build Order, Configuration Reference, Quality Attribute Scenarios (stimulus/response NFR format), and the 9 Requirement Quality Criteria / Rust-specific Requirement Smells self-check tables. |
