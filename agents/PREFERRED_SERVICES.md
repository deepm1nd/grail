# Preferred Infrastructure Services

**MANDATE: Adherence to this guide is not optional. It is a strict requirement.**

This file governs **infrastructure services** — databases, message brokers, object stores, and similar external systems that the project's code integrates with — as distinct from Rust library dependencies (`PREFERRED_DEPENDENCIES.md`) and development tools (`PREFERRED_TOOLS.md`).

See `CHANGELOG.md` for version history.

---

## Core Deployment Principles

### Docker Compose is Preferred
All infrastructure services MUST be deployed via Docker containers rather than installed directly into the agent or CI environment. This keeps environments reproducible and avoids host-system pollution.

- **Preferred tool: `docker-compose`** (or `docker compose` v2 CLI).
- Service definition files live in the `deploy/` directory in the project root.
- `deploy/docker-compose.yml` — services the project itself is packaged as (e.g., a web service that is itself containerized).
- `deploy/docker-compose.dev.yml` — infrastructure dependencies needed during development and testing (Postgres, Neo4j, Qdrant, MinIO, Redis, etc.) but not part of the project's own deployment artifact.
- CI uses the same compose files; the dev file is started before integration tests run.

### HTTP Over HTTPS for Local Development
Local/dev service exposure MUST use HTTP (not HTTPS) unless the project specifically requires TLS for a security requirement. TLS termination for production is handled at the infrastructure layer (reverse proxy, load balancer), not by the service itself in dev.

### Prefer Each Service's Own Default Port
Use each service's published default port. Do not remap ports without a documented reason. Default ports are listed per service below.

### Configuration Files in `deploy/`
All service definition files (YAML), environment variable files (`.env`), and any configuration files for infrastructure services live in `deploy/`. Never commit secrets; use placeholder values and document the required real values.

---

## Approved Services

### PostgreSQL + SQLx

**Preferred relational database.** Use `sqlx` crate for type-safe queries with compile-time verification.

- **Default ports:** 5432 (PostgreSQL)
- **Rust crates:** `sqlx` (with `postgres` feature), `sqlx-cli` (migrations)
- **Docker image:** `postgres:17` (or latest stable)

```yaml
# deploy/docker-compose.dev.yml
services:
  postgres:
    image: postgres:17
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: myproject_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Neo4j

Graph database. Use when the data model is fundamentally graph-shaped (nodes, relationships, traversals).

- **Default ports:** 7474 (HTTP browser), 7687 (Bolt protocol — used by Rust drivers)
- **Rust crates:** `neo4j` (from `PREFERRED_DEPENDENCIES.md`)
- **Docker image:** `neo4j:5` (or latest stable)

```yaml
# deploy/docker-compose.dev.yml
services:
  neo4j:
    image: neo4j:5
    environment:
      NEO4J_AUTH: neo4j/devpassword
    ports:
      - "7474:7474"
      - "7687:7687"
    volumes:
      - neo4j_data:/data

volumes:
  neo4j_data:
```

### Qdrant

Vector database. Use for semantic search, embedding storage, and similarity queries.

- **Default ports:** 6333 (HTTP REST), 6334 (gRPC)
- **Rust crates:** `qdrant-client` (from `PREFERRED_DEPENDENCIES.md`)
- **Docker image:** `qdrant/qdrant:latest`

```yaml
# deploy/docker-compose.dev.yml
services:
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - qdrant_data:/qdrant/storage

volumes:
  qdrant_data:
```

### MinIO

S3-compatible object storage. Use for file storage, blob storage, and any workload that would use AWS S3 in production.

- **Default ports:** 9000 (S3 API), 9001 (Web console)
- **Rust crates:** Use `reqwest` or `aws-sdk-s3` (with MinIO endpoint override)
- **Docker image:** `minio/minio:latest`

```yaml
# deploy/docker-compose.dev.yml
services:
  minio:
    image: minio/minio:latest
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_data:/data

volumes:
  minio_data:
```

### Redis

In-memory key-value store. Use for caching, session storage, pub/sub messaging, and rate limiting.

- **Default port:** 6379
- **Rust crates:** `redis` (async feature, not on PREFERRED_DEPENDENCIES.md — requires approval if used)
- **Docker image:** `redis:7` (or latest stable)

```yaml
# deploy/docker-compose.dev.yml
services:
  redis:
    image: redis:7
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

### Mosquitto (MQTT Broker)

Lightweight pub/sub message broker. Use for IoT/embedded device messaging, telemetry
ingestion, and command/control channels — particularly relevant alongside an embedded
component using the ESP-IDF stack (see `agents/PREFERRED_DEPENDENCIES.md`'s Embedded
section), but not exclusive to embedded use.

- **Default ports:** 1883 (MQTT, unencrypted), 8883 (MQTT over TLS), 9001 (MQTT over WebSockets, if enabled)
- **Rust crates:** requires an MQTT client crate — not currently listed in
  `agents/PREFERRED_DEPENDENCIES.md`; propose one (e.g. `rumqttc`) for explicit approval per
  that file's Unlisted Dependencies rule before use
- **Docker image:** `eclipse-mosquitto:2` (or latest stable)

```yaml
# deploy/docker-compose.dev.yml
services:
  mosquitto:
    image: eclipse-mosquitto:2
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./deploy/mosquitto/mosquitto.conf:/mosquitto/config/mosquitto.conf
      - mosquitto_data:/mosquitto/data
      - mosquitto_log:/mosquitto/log

volumes:
  mosquitto_data:
  mosquitto_log:
```

**Configuration note:** Mosquitto 2.x refuses anonymous connections and binds to no listener
by default without an explicit `mosquitto.conf` — unlike the other services in this file, a
minimal config file is required, not optional, for the container to accept any connection at
all. Place it at `deploy/mosquitto/mosquitto.conf` per this file's Configuration Files in
`deploy/` convention; do not commit real credentials if authentication is enabled —
placeholder values only, per this file's standing rule.

---

## Project-as-Container Pattern

When the project itself (or one of its services) is deployed as a container — for example, when it is one of several interrelated web services — the service definition also lives in `deploy/docker-compose.yml`.

```yaml
# deploy/docker-compose.yml — project's own services
services:
  myproject_api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgres://dev:dev@postgres:5432/myproject
    depends_on:
      - postgres

  # Supporting services used in production are also here
  postgres:
    image: postgres:17
    # ... (same as dev but with production-appropriate settings)
```

---

## Session Startup

When a development session needs infrastructure services, the agent MUST start them before running any task that depends on them:

```bash
# Start dev infrastructure
docker compose -f deploy/docker-compose.dev.yml up -d

# Verify services are healthy before proceeding
docker compose -f deploy/docker-compose.dev.yml ps
```

If Docker is not available in the session environment, this is a missing-prerequisite condition — the agent follows the Missing Tool Protocol from `PREFERRED_TOOLS.md` (inform user, attempt to resolve, escalate if it cannot).

---

## Services Requiring Approval

Services not listed above require explicit user approval before being added to a project. Propose the service with rationale; wait for approval before adding it to `deploy/` or referencing it in the Architecture Specification.
