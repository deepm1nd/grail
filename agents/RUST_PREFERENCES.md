# Rust-Specific Constraints on Design Document Content

**Purpose:** This guide captures Rust-language and target-platform constraints that affect
the *content* of the Architecture Specification and Development Plan — not just code style.
These are things that, if missed during Design, surface as expensive rework during
Development instead of being caught on paper. This is a content-quality reference for the
Design Phase; it supplements `agents/PREFERRED_DEPENDENCIES.md` (dependency choice),
`agents/PREFERRED_TOOLS.md` (development tool choice and MSRV policy), and `AGENTS.md`
(general naming conventions) without duplicating them.

This file is referenced from `CLAUDE.md` Section 2.2 and applies throughout the gated
workflow in `agents/DESIGN.md` Section 5 — most directly at Step 3 (Requirement
Decomposition), Step 5 (Verification Feasibility), and Step 6 (Final Architecture Synthesis,
particularly the Deployment and Interface Control viewpoints).

See `CHANGELOG.md` for version history.

---

## 0. Edition and MSRV Policy

- **Rust 2024 edition is mandatory** for every workspace crate (per
  `agents/PREFERRED_DEPENDENCIES.md`). This is a fixed requirement, not a Step 5 decision.
- **The workspace Minimum Supported Rust Version (MSRV) is NOT hardcoded in this file or
  any other process document.** Toolchain and tool versions are exactly the kind of fact
  that goes stale the moment it's written down (Trigger A, `CLAUDE.md` §3.3, applied to the
  document's own content). Instead:
  - **The workspace MSRV is determined at Design Step 5 (Verification Feasibility)**, set
    deliberately by checking the actual build-MSRV of every tool the project commits to
    using (`cargo-nextest`'s *build* MSRV, not just its run MSRV, is the specific failure
    mode that motivated this policy — see `agents/PREFERRED_TOOLS.md`) and every library
    dependency's MSRV, then taking the maximum. This value is recorded in the workspace
    `Cargo.toml`'s `rust-version` field and in `rust-toolchain.toml`'s `channel`.
  - **Dual-MSRV pattern, for any project with a publishable library crate** (an SDK or
    plugin-facing crate consumed by external projects with their own, possibly older,
    toolchains): the workspace toolchain is pinned to the development MSRV described
    above — this is what every developer and CI job actually builds the full workspace
    with. The publishable crate separately declares its own, lower `rust-version` in its
    *own* `Cargo.toml`, representing the genuinely binding consumer contract (this becomes
    part of §10.2 MSRV Policy in the Architecture Specification). A dedicated CI stage
    installs that consumer toolchain and compiles *only* that crate and its dependencies at
    that version — this is what actually enforces the contract, not the workspace toolchain
    pin. The publishable crate's own dependency list must be kept minimal and re-verified
    against the consumer MSRV whenever a dependency is added to it; a dependency requiring a
    newer compiler than the consumer contract promises is caught by this CI stage before it
    ships, not discovered after a consumer's build breaks.
  - **A pre-flight version sanity check runs at the start of every Development-Phase
    session** (see `agents/PREFERRED_TOOLS.md`): installed `rustc`/`cargo`/tool versions are
    checked against the project's declared compatibility matrix before any build is
    attempted. A mismatch is treated as a missing-prerequisite Escalation Trigger — the
    agent self-resolves via install where a known-good version exists (appending the install
    command to `scripts/setup_env.sh`/`.bat` per the Missing Tool Protocol), or escalates if
    it cannot.
- **Coverage tooling:** `cargo-llvm-cov` is the preferred code coverage tool (see
  `agents/PREFERRED_TOOLS.md`) — it uses a stable compiler flag rather than hooking compiler
  internals, and is materially more robust across Rust version upgrades than alternatives
  that have historically broken on point releases.

---

## 1. Naming and Identifier Constraints

These affect any identifier introduced in the Data Dictionary, ICD, or Development
Plan/Checklist — entity names, field names, module names, function names.

- **No hyphens.** Per `AGENTS.md` and `PREFERRED_DEPENDENCIES.md`, identifiers, file names,
  and folder names use `snake_case`, never hyphens — Rust's module/crate system treats
  hyphens as invalid in identifiers (crate names on crates.io may use them, but the actual
  Rust identifier is auto-converted to underscores). Any naming taken from an external
  source (a REST API field, a database column, a UI label) that uses hyphens or
  `kebab-case` needs an explicit underscore mapping noted in the Data Dictionary, not a
  silent one-to-one rename assumed at implementation time.
- **No reserved keywords.** Identifiers must avoid both strict keywords (`fn`, `match`,
  `impl`, `dyn`, `move`, `type`, `trait`, `async`, `await`, `loop`, `ref`, `self`, `Self`,
  `where`, `unsafe`, `enum`, `struct`, `union`, and others in the current edition) and
  keywords reserved for future use. A Data Dictionary entity or field literally named
  `Type`, `Move`, or `Async` will not compile as a plain identifier. Rust's raw-identifier
  escape (`r#type`) technically exists but should be treated as a last resort, not a design
  pattern — if a requirement or interface needs a field called "type," the Architecture
  Specification should rename it at the design stage (e.g. `kind`, `category`) rather than
  deferring an awkward escape to implementation.
- **Case convention by role, enforced at the design stage:**
  - `snake_case` — functions, variables, modules, fields.
  - `UpperCamelCase` — types, traits, enums, enum variants.
  - `SCREAMING_SNAKE_CASE` — constants and statics.
  These are lint-enforced (often promoted to warnings-as-errors in CI) — a Data Dictionary
  that names a type in `snake_case` "for consistency with the API" creates a guaranteed
  rename at implementation time. The Architecture Specification should use Rust's actual
  case conventions for any name destined to become a real identifier, even where the
  external system it's modeling (a JSON API, a database) uses a different convention — and
  should note the mapping explicitly if the two diverge.
- **No leading digits; avoid non-ASCII identifiers in practice.** Rust technically permits
  some Unicode in identifiers, but mixed-script or non-ASCII identifiers are a practical
  footgun (tooling, search, cross-team readability) and should not appear in design
  documents as the literal Rust-facing name, even if the user-facing/domain term is
  non-English. Use a transliterated or English working name for the identifier and note the
  domain term as a comment/definition alongside it.

---

## 2. Type System and Ownership Constraints

These affect how requirements, the Data Dictionary, and the Interface Control Document
(ICD) describe data and behavior — they are architecture-level decisions in Rust, not
implementation details, because they constrain what's expressible at an API boundary.

- **No null.** Rust has no null value; absence is represented by `Option<T>`. Any
  requirement or Data Dictionary entry that uses "optional," "nullable," or "may be absent"
  language (common in SQL/JSON-Schema-derived thinking) must be mapped explicitly to
  `Option<T>` in the Architecture Specification, including stating what `None` *means*
  semantically for that field (e.g. "not yet computed" vs. "intentionally cleared" vs.
  "not applicable for this variant") — these are different requirements in Rust even though
  they'd all just be `NULL` elsewhere, and conflating them is a Requirement Smell under the
  9 criteria in `agents/DESIGN.md` Section 4.5 (specifically Unambiguous and Complete).
- **No inheritance.** Rust has no class hierarchy / subclassing. Any architecture content
  using inheritance language ("Type B extends Type A," "B is a subclass of A," "B
  overrides A's behavior") needs to be re-expressed as Rust actually models it: shared
  behavior via traits, shared data via composition (a struct holding/wrapping another), or
  an enum representing a closed set of variants. The Functional and Information viewpoints
  (Step 6) should use trait/composition language directly rather than OOP language that
  will mislead an implementer about what's actually being designed.
- **Errors are values, not exceptions.** Rust has no exception mechanism; failures are
  either a returned `Result<T, E>` (recoverable) or a panic (intended to be unrecoverable —
  a programming-error signal, not a control-flow tool). Per `PREFERRED_DEPENDENCIES.md`,
  `anyhow` and `thiserror` are the mandated error-handling dependencies. Any requirement
  phrased as "the system throws an exception" or "raises an error" needs translating at
  design time into: (a) is this recoverable — specify the `Result` and what the `Err`
  variant communicates to the caller — or (b) is this a true invariant violation that should
  panic, in which case the requirement should say so explicitly, since "the system never
  panics on this path" vs. "this path may panic on invariant violation" are different,
  individually testable requirements.
- **Ownership and borrowing are interface-level decisions.** Any two components described
  as sharing or exchanging data need the Interface Control Document to specify *who owns the
  data* and whether the other side receives an owned copy, a borrowed reference, or a
  reference-counted handle (`Rc`/`Arc`). A requirement or ICD entry that says "Component B
  holds a reference to Component A's state" is incomplete as written under the Complete and
  Unambiguous criteria — it must specify the ownership/lifetime relationship, because that
  determines what's actually implementable, not just how it's coded.
- **Concurrency safety boundaries (`Send`/`Sync`) are architectural, not incidental.** Any
  requirement implying shared state accessed from more than one task/thread/Worker needs
  the Deployment or Interface Control viewpoint to state the intended synchronization
  approach (e.g. message-passing via channels, `Arc<Mutex<T>>`, an actor-style design) —
  this determines real feasibility and should be decided and recorded at design time, not
  discovered as a compiler error during Development. See Section 3 below for how this
  applies specifically to the WASM/Web Worker target.

---

## 3. Multithreading and WASM/Web Worker Concurrency

**Project-specific context:** this project targets a Rust web framework, and multithreading
is a goal to support **wherever possible, across both native and WASM targets** — not a
capability to avoid in WASM. The constraint below replaces a more restrictive "no threads in
WASM" framing that does not reflect this project's actual goals.

- **WASM has no native thread primitive equivalent to `std::thread`,** but the browser
  provides a real parallelism primitive: **Web Workers**, each running an independent WASM
  instance with its own linear memory. `std::thread::spawn` does not transparently become a
  Worker when compiled to `wasm32-unknown-unknown` — Worker-based concurrency must be
  designed for explicitly; it does not fall out of writing ordinary multithreaded Rust and
  recompiling for the web.
- **Two distinct concurrency models exist for WASM, with different architectural
  consequences, and the Architecture Specification must state which one a given component
  uses:**
  - **Message-passing (structured clone / transferable objects).** Simplest and safest;
    works in every browser without special hosting requirements. Data crossing the
    Worker boundary is copied or moved, not shared — any Data Dictionary type that needs to
    cross this boundary should be cheaply (de)serializable (this connects directly to
    `serde`/`bincode` and `wasm-bindgen` in `PREFERRED_DEPENDENCIES.md`). Large or deeply
    nested types crossing this boundary frequently are a real performance consideration the
    Deployment Viewpoint should note, not a detail left for Development to discover.
  - **`SharedArrayBuffer` + atomics.** Enables genuine shared memory across Workers and lets
    Rust's `std::thread` + `Mutex`/atomics model work close to natively (this is what
    `wasm-bindgen`'s threading support and crates such as `wasm-bindgen-rayon` rely on).
    This requires the hosting page to serve `Cross-Origin-Opener-Policy` and
    `Cross-Origin-Embedder-Policy` headers, and is **not available in every embedding
    context** — for example, some third-party-iframe or restrictive-hosting scenarios may
    not permit setting these headers. This is a deployment/hosting constraint, not purely a
    code constraint, and belongs in the Deployment Viewpoint.
- **A single logical component may need two concurrency implementations behind one stable
  interface.** Where "multithreading wherever possible" is a goal, the *interface* (trait or
  API boundary) should stay target-agnostic in the Functional Viewpoint, while the
  Interface Control Document documents that the implementation diverges per target —
  typically via `#[cfg(target_arch = "wasm32")]` conditional compilation — rather than the
  spec assuming one canonical implementation that happens to differ "in the code" later.
- **COOP/COEP is a hard trigger condition, not just a consideration.** The moment any
  requirement or component implies `SharedArrayBuffer`-based threading, Claude flags it
  explicitly — not as a passing note, but as a blocking question — using language to the
  effect of: *"This component's threading model requires `Cross-Origin-Opener-Policy` and
  `Cross-Origin-Embedder-Policy` headers on the hosting page. If [project] is meant to be
  usable embedded in third-party pages, or in any context where response headers aren't
  controllable, this will not work in that context."* Claude then asks directly whether the
  project's deployment model includes any such context (embeddable widget, third-party
  iframe, CDN-hosted script dropped into pages the project doesn't control, etc.) — this is
  a product/deployment decision Claude cannot infer from the architecture alone.
  - **If the answer is yes** (some deployment context can't guarantee these headers), the
    component needs a **documented fallback** as part of the Architecture Specification —
    typically: detect `SharedArrayBuffer` availability at runtime and degrade to
    message-passing — not an afterthought left to Development to improvise.
  - **If the deployment model isn't decided yet** when this comes up, Claude treats it as an
    open flagged assumption that blocks that component's Step 5 sign-off, the same as any
    other unresolved feasibility question — Claude does not assume an answer either way and
    move on.
- **Recommended threading crates — ordered to match this project's stated default, not just
  general capability.** `agents/exemplars/architecture_specification_template.md` §4.7 and
  §4.9 establish a specific preference order for this project: **message-passing via Web
  Workers is the default/preferred model; `SharedArrayBuffer`-based shared-memory threading
  is an approved exception, used only where message-passing is demonstrably insufficient**
  (e.g. genuine data-parallel performance needs that copying would defeat). The crate
  recommendations below are ordered to match that stance — default option first — rather
  than by raw feature power, since leading with the most capable option would silently
  contradict the architecture's own stated default. `agents/PREFERRED_DEPENDENCIES.md`'s
  WASM & Gloo section does not currently list any threading-specific crate; the following
  are reasonable candidates, not a pre-approval — each still requires explicit user approval
  before being added to `PREFERRED_DEPENDENCIES.md` and treated as part of a plan, per that
  file's own rule and `AGENTS.md`'s "Dependency and Tool Selection" mandate:

  | Order | Crate | Concurrency model | When to reach for it |
  |---|---|---|---|
  | 1 (default) | `gloo-worker` | Message-passing only (`postMessage`), no shared memory | Start here for most components. No `SharedArrayBuffer`, no COOP/COEP requirement, no Worker-pool contention with anything else. Good fit for "delegate this unit of work, get a typed response back" — which covers most multithreading needs in a web framework context. |
  | 2 (exception — data-parallel) | `wasm-bindgen-rayon` | `SharedArrayBuffer`-backed work-stealing pool (rayon API) | Use only when a component has a genuine data-parallel workload (parallel iteration over large collections) where message-passing's copy overhead would be demonstrably insufficient — per the architecture's stated exception criterion, not as a default. Requires the COOP/COEP trigger below to be resolved first. |
  | 3 (exception — long-running worker) | `wasm_thread` | `SharedArrayBuffer`-backed, one Worker per `spawn()` | Use only when a component needs a genuinely long-running background thread with a `std::thread`-like API and message-passing has been ruled out for the same reason as above. Less mature than rayon's bridge; also requires COOP/COEP. |
  | 4 (manual fallback) | `web-sys` raw `Worker`/`SharedArrayBuffer` bindings | Either, fully manual | Not a higher-level crate; use only when none of the above fit a component's interaction pattern. Already a transitive dependency of all the others. |

  **Compatibility note — these are not mutually exclusive, but two of them conflict at
  runtime if used carelessly together:** `wasm-bindgen-rayon` and `wasm_thread` each want to
  *own the Worker pool* — rayon initializes a fixed-size pool up front; `wasm_thread` spins
  up a new Worker per call. Running both **within the same component** risks two
  uncoordinated pools competing for the same `SharedArrayBuffer`-backed memory and the
  browser's actual core count. This is a runtime resource-planning conflict, not a compile
  error, so it will not be caught by the build — it must be caught at design time. Per
  Section 4 below, Step 5's feasibility check includes confirming that a component does not
  already use one Worker-pool-owning crate before introducing a second. `gloo-worker` has no
  such conflict with either, since it doesn't compete for the same primitive — it may be
  freely combined with `wasm-bindgen-rayon` or `wasm_thread` in the same component if their
  concurrency needs differ (e.g. rayon for parallel computation, gloo-worker for a separate
  delegate-and-respond task), though this combination should still be confirmed per
  component, not assumed.
- **Filesystem/network access in WASM remains browser-mediated**, independent of the
  threading question above: a WASM component (whether on the main thread or in a Worker)
  does not get raw filesystem or socket access — only what the browser exposes (`gloo-net`,
  fetch, WebSocket APIs, etc.). Any requirement assuming raw file or socket I/O on a
  WASM-targeted component is infeasible as stated and needs redesigning around the
  available browser-mediated APIs, independent of whichever concurrency model is chosen.

---

## 4. How These Constraints Are Applied During the Design Phase

Per `CLAUDE.md` Section 2.2:

- **Step 3 (Requirement Decomposition):** when checking a requirement against the 9 quality
  criteria in `agents/DESIGN.md` Section 4.5, the constraints in Sections 1–2 above are part
  of what makes a requirement *Unambiguous* and *Complete* — e.g. a requirement using
  nullable/inheritance/exception language is not yet Design-independent or Unambiguous in a
  Rust context until it's translated.
- **Step 5 (Verification Feasibility):** the constraints in Section 3 above are checked
  directly as part of technical sufficiency — specifically: the per-component
  message-passing-vs-`SharedArrayBuffer` question, the COOP/COEP trigger condition (and its
  required fallback if any deployment context can't guarantee the headers), confirming
  no component combines two Worker-pool-owning crates (`wasm-bindgen-rayon` +
  `wasm_thread`) without an explicit decision to do so, and **setting the workspace MSRV**
  per Section 0 above.
- **Step 6 (Final Architecture Synthesis):** the Deployment Viewpoint and Interface Control
  Document are where target-specific concurrency implementation, ownership/lifetime
  relationships, and COOP/COEP hosting constraints are formally recorded.

As with the rest of the Design Phase workflow, any translation Claude performs to satisfy
these constraints (e.g. converting "nullable" to `Option<T>` with a stated meaning for
`None`, or choosing message-passing vs. `SharedArrayBuffer` for a given component) is
presented as a flagged assumption for user confirmation, not asserted as a settled fact —
consistent with `CLAUDE.md` Section 3's propose-with-flagged-assumptions model.
