# Mandated and Forbidden Dependencies

**MANDATE: Adherence to this guide is not optional. It is a strict requirement.**

## Language Specific Mandates

### Rust
- **Rust Edition:** The agent MUST prefer the latest stable Rust edition (e.g., 2021 or 2024) that is supported by the project's current toolchain and library ecosystem.
- **Naming Conventions:** The agent is **STRONGLY FORBIDDEN** from using dashes (`-`) or hyphens in identifiers, variable names, file names, or folder names. These characters are incompatible with Rust's module and crate system. The agent MUST use underscores (`_`) for word separation in all contexts.

## Dependencies

- **Preferred Dependencies:** You SHOULD prioritize using dependencies from the preferred lists below.
- **Unlisted Dependencies:** If a desired dependency is NOT on the preferred list, you MUST follow the "Dependency and Tool Selection" mandate in `AGENTS.md` to get explicit user approval before proceeding.
- **Forbidden Dependencies:** You are **ABSOLUTELY FORBIDDEN** from using any dependency on the forbidden list. There are no exceptions. Proposing or using a forbidden dependency is a critical process failure.

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
- wasm-bindgen-rayon  # Preferred for data-parallel work. SharedArrayBuffer-backed; requires COOP/COEP headers on the hosting page. See agents/RUST_PREFERENCES.md Section 3.
- wasm_thread  # Preferred for long-running background workers (std::thread-like API). SharedArrayBuffer-backed; same COOP/COEP requirement. NOTE: do NOT combine with wasm-bindgen-rayon within the same component — both own a Worker pool and will contend for the same SharedArrayBuffer memory and core count at runtime; this is not caught at compile time. See agents/RUST_PREFERENCES.md Section 3.
- gloo-worker  # Message-passing only (postMessage); no SharedArrayBuffer, no COOP/COEP requirement, no conflict with the other two. May be freely combined with either above when a component's concurrency needs differ.

## Data & Serialization
- serde
- serde_json
- bincode
- prost

## Database & Clients
- neo4j
- qdrant-client
- rdkafka

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

## Dependencies Requiring Approval
The following dependencies are allowed but REQUIRE explicit user approval before being added to a project.
- trunk
- webpack

## Forbidden Dependencies
(None currently listed)
