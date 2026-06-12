# Development Checklist: nverse Resync (Strategy B)

## Phase 1: Protocol & Shared Libs
- [x] PROT-001: Update `nverse.proto` with versioning and AOI message types.
- [x] PROT-001: Recompile protobuf and verify Rust code generation.
- [x] PROT-001: Add unit tests for new protocol fields.

## Phase 2: Shard Management
- [x] GATE-001: Create `simulation_shards` database table via SQL migration.
- [x] GATE-001: Implement `ShardRegistry` struct and logic in `nverse_gateway`.
- [x] GATE-002: Add `/api/internal/shards/register` endpoint to Gateway.
- [x] GATE-002: Add `/api/internal/shards/heartbeat` endpoint to Gateway.
- [x] GATE-002: Verify shard registration via API integration tests.

## Phase 3: Shard-to-Gateway Sync
- [x] SIM-001: Implement registration call in `nverse_server_godot` startup sequence.
- [x] SIM-001: Handle registration failure/retry logic in Godot server.
- [x] SIM-002: Create internal streaming protocol (e.g., dedicated WebSocket) between Shard and Gateway.
- [x] SIM-002: Stream real `WorldStateDelta` packets from Godot server to Gateway.

## Phase 4: E2E Distributed Flow
- [x] GATE-003: Implement packet routing in Gateway's WebSocket handler based on `shard_id`.
- [x] GATE-003: Remove dummy broadcast in Gateway and replace with real proxied data.
- [x] WEB-001: Update `nverse_client_web` to handle shard assignment and routing.
- [ ] WEB-001: Verify end-to-end movement synchronization between Godot server and Web Client.

## Final Verification
- [ ] Run `scripts/run_system_test.sh` and ensure all tests pass.
- [x] Verify logs for correct shard and user context.
- [ ] Ensure `nverse_resync_0` issues are resolved.
