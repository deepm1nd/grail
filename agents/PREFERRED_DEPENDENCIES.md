# Mandated and Forbidden Dependencies

**MANDATE: Adherence to this guide is not optional. It is a strict requirement.**

This file governs **Rust library dependencies** compiled into the project. For development tools (cargo-nextest, trunk, cargo-llvm-cov, etc.) see `agents/PREFERRED_TOOLS.md`. For infrastructure services (Postgres, Neo4j, etc.) see `agents/PREFERRED_SERVICES.md`.

See `CHANGELOG.md` for version history.

---

## Language Specific Mandates

### Rust
- **Rust 2024 edition is mandatory.** Specify `edition = "2024"` in every workspace `Cargo.toml`. See `agents/PREFERRED_TOOLS.md` for the MSRV policy (workspace MSRV is determined at Design Step 5, not hardcoded here) and the dual-MSRV pattern for projects with publishable library crates.
- **Naming Conventions:** The agent is **STRONGLY FORBIDDEN** from using dashes (`-`) or hyphens in identifiers, variable names, file names, or folder names. These characters are incompatible with Rust's module and crate system. The agent MUST use underscores (`_`) for word separation in all contexts.

---

## Dependencies

- **Preferred Dependencies:** You SHOULD prioritize using dependencies from the preferred lists below.
- **Unlisted Dependencies:** If a desired dependency is NOT on the preferred list, you MUST follow the "Dependency and Tool Selection" mandate in `AGENTS.md` to get explicit user approval before proceeding.
- **Forbidden Dependencies:** You are **ABSOLUTELY FORBIDDEN** from using any dependency on the forbidden list. There are no exceptions.

## Web & Networking
- tokio  # MANDATE: For native applications ONLY. Incompatible with WASM.
- axum
- reqwest
- tonic
- tungstenite
- tokio-tungstenite
- url
- wtransport

## WASM & Gloo
- wasm-pack
- wasm-bindgen
- wasm-bindgen-futures
- gloo-file
- gloo-net
- gloo-console
- gloo-timers
- gloo-utils
- gloo-storage

## WASM Threading
- gloo-worker  # DEFAULT for WASM concurrency. Message-passing only (postMessage); no SharedArrayBuffer, no COOP/COEP requirement.
- wasm-bindgen-rayon  # EXCEPTION, not default — use only when message-passing is demonstrably insufficient for a data-parallel workload. Requires COOP/COEP headers.
- wasm_thread  # EXCEPTION, not default — use only when message-passing is demonstrably insufficient for a long-running background worker. Same COOP/COEP requirement. Do NOT combine with wasm-bindgen-rayon in the same component.

## Data & Serialization
- serde
- serde_json
- bincode
- prost

## Database & Clients
- neo4j
- qdrant-client
- rdkafka
- sqlx  # With appropriate feature flags (postgres, sqlite, etc.) per project

## Async
- async-trait
- futures
- futures-util

## Error Handling
- anyhow
- thiserror

## Logging
- tracing

## Utilities
- rand
- jsonwebtoken
- chrono
- uuid
- dotenv
- base64
- regex
- once_cell
- bcrypt

## Docker
- bollard

## Embedded (ESP32 / ESP-IDF)
> Applies primarily to embedded Rust projects. Only relevant when the project has an
> ESP32/ESP-IDF-targeted component; not applicable to native or WASM-only projects.
- esp-idf-hal
- esp-idf-svc
- esp-camera-rs

**MSRV note (Step 5):** these crates target `esp-idf` via `std`, not `wasm32-unknown-unknown`
or a bare native host triple. An embedded component has its own toolchain track (via
`espup`, not `rustup` alone) independent of the workspace MSRV policy in
`agents/RUST_PREFERENCES.md` §0 and independent of any WASM component's target in the same
workspace — state this explicitly in Architecture Specification §6.1/§6.2 (per-component
stack).

## Forbidden Dependencies
(None currently listed)
