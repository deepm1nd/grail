# Mandated and Forbidden Dependencies

**MANDATE: Adherence to this guide is not optional. It is a strict requirement.**

Governs **Rust library dependencies** compiled into the project. Dev tools:
`agents/PREFERRED_TOOLS.md`. Infra services: `agents/PREFERRED_SERVICES.md`.

See `CHANGELOG.md` for version history.

---

## Language Specific Mandates

### Rust
- **Rust 2024 edition is mandatory.** `edition = "2024"` in every workspace `Cargo.toml`.
  MSRV policy: `agents/PREFERRED_TOOLS.md` (set at Design Step 5, not hardcoded here).
- **No dashes/hyphens** in identifiers, file names, or folder names — use underscores.

## Dependencies
- **Preferred:** use without additional approval.
- **Unlisted:** requires explicit user approval (`AGENTS.md` §2.2) before use.
- **Forbidden:** never used, no exceptions.

## Web & Networking
- tokio  # Native only. Incompatible with WASM.
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
- gloo-worker  # DEFAULT. Message-passing (postMessage) only; no SharedArrayBuffer/COOP/COEP requirement.
- wasm-bindgen-rayon  # EXCEPTION — data-parallel workloads only. Requires COOP/COEP.
- wasm_thread  # EXCEPTION — long-running background worker only. Requires COOP/COEP. Do not combine with wasm-bindgen-rayon in the same component.

## WASM Component Model (Sandboxed Execution)
> For any project needing to execute untrusted/user-supplied code in a sandbox — not
> project-specific; a general pattern approved for reuse whenever that need exists.
- wasmtime  # Sandboxed component runtime; deny-by-default WASI capability model.
- wasmtime-wasi  # WASI host implementation. Tied to a tokio executor.
- wit-bindgen  # WIT interface codegen (guest side). Host side ships inside wasmtime itself.

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

## Messaging / IoT
- rumqttc  # MQTT client, pairs with the Mosquitto service in PREFERRED_SERVICES.md.
  Now approved — resolves the prior "propose one, e.g. rumqttc" placeholder there.

## Bluetooth
- btleplug  # Cross-platform BLE. Native only — incompatible with WASM.

## Machine Learning / Inference
> Use when a project needs ONNX-model inference in pure Rust (no C-binding surface).
- tract-onnx
- tokenizers  # Hugging Face Rust tokenizer, commonly paired with tract-onnx.

## Cryptography
- aes-gcm  # RustCrypto AEAD (NCC-audited, pure Rust). Standard choice for
  application-level encryption-at-rest of secrets/tokens.
- totp-rs  # RFC 6238 TOTP generation/verification, standard 2FA building block.

## Scheduling
- tokio-cron-scheduler  # STANDING CONVENTION for in-process periodic/deferred jobs,
  unless a case needs to survive a process restart in a way this alone can't guarantee.

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
- rand  # WASM target: requires the `getrandom` wasm_js backend config, `PREFERRED_TOOLS.md`'s WASM section.
- jsonwebtoken
- chrono
- uuid  # WASM target: same `getrandom` wasm_js backend config as rand.
- dotenv
- base64
- regex
- once_cell
- bcrypt
- qrcode  # QR matrix generation, commonly paired with totp-rs.
- image  # Rasterizes qrcode output to PNG/SVG; general-purpose image handling.

## Docker
- bollard

## Testing-Only Dependencies
> MUST NOT ship in any production build artifact — dev-dependencies/test-target only.
- wiremock  # HTTP mocking; asserting zero calls to an external service in tests.

## Embedded (ESP32 / ESP-IDF)
> Primarily for embedded Rust projects with an ESP32/ESP-IDF-targeted component. See
> `agents/ESP32_ESPIDF_RUST_BUILD_GUIDE.md` for the full toolchain/build setup this
> pairs with.
- esp-idf-hal
- esp-idf-svc
- esp-camera-rs

## Display / Graphics (ESP32 / ESP-IDF)
> `embedded-graphics`'s `DrawTarget` trait is the core display abstraction — the
> extension point for any panel, not a limit on what's supported. `ssd1306`/`mipidsi`
> are the two concretely wired backends.
- embedded-graphics  # Core display abstraction (DrawTarget trait).
- ssd1306            # OLED backend (I2C/SPI), implements DrawTarget.
- mipidsi            # TFT backend (MIPI DSI/SPI), implements DrawTarget.

**MSRV note:** embedded components target `esp-idf` via `std`, on a separate
`espup`-managed toolchain track independent of the workspace MSRV and any WASM
component's target in the same workspace — state explicitly in Architecture
Specification §6.1/§6.2.

## Dependencies Requiring Approval (Seen Before, Not Pre-Blessed)
> Vetted in a prior project engagement but vendor/integration-specific — still requires
> per-project approval (`AGENTS.md` §2.2) rather than being pre-approved policy.
- oauth2, oauth10a  (SSO / broker-specific OAuth clients)
- async-stripe, autogen-squareup  (payment processors)
- feed-rs  (RSS/Atom parsing)

## Forbidden Dependencies
(None currently listed)
