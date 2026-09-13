# M00-T02a — Managed Compose dependency environment

- **Status:** Completed
- **Depends on:** T01
- **Blocks:** T02b, T07
- **Plan:** [M00-PLAN-backend-foundation](../M00-PLAN-backend-foundation.md) §3.2, T2a

Establishes the R1.1 reproducible container-based environment. **This task
reads no KnowHub setting**, so it may run before `knowhub.config` exists.

## Owning specification requirements

M00-SPEC-004 R1.1, R2, R4 (image pinning), R7, R8

## Acceptance criteria contributed to

M00-AC-018 (the R1.1 environment clause), M00-AC-019 (the pinning clause)

## Files created or modified

| Path | Action |
|---|---|
| `docker-compose.dev.yml` | Create |

## Implementation steps

1. Define exactly three services — PostgreSQL, Redis, Azurite. No fourth.
2. Pin each image by tag **and** SHA256 digest:
   `pgvector/pgvector:pg15`, `redis:7.4-alpine`,
   `mcr.microsoft.com/azure-storage/azurite:3.x`.
3. Publish on the non-default ports from plan §3.2 — 55432, 56379,
   10000–10002 — so the environment can never collide with a service the
   developer already runs.
4. Give each service a **service-native** health check that reads no KnowHub
   setting: `pg_isready` for PostgreSQL, `redis-cli PING` inside the Redis
   container, an HTTP probe for the Azurite blob endpoint.
5. Supply non-production credentials as Compose-local values. Nothing usable
   against any deployed environment.
6. Use named volumes so `docker compose down -v` is a clean reset.

## Tests and evidence

- Bring-up on a clean machine reaches all three services healthy.
- `docker compose config` shows a digest on every image reference.
- Health status is the evidence. Per SPEC-004 R24 this is **not** an
  integration test and must not be counted as one.

## Failure paths to exercise

- A service that never reports healthy fails bring-up visibly, naming the
  service.
- A port collision surfaces as a named service failure, not a silent hang.

## Telemetry expectations

None.

## Out of scope

`scripts/check_dependencies.py` and anything that reads a configured endpoint
(T02b); `.env.example` (T02b); any application client for Redis or Azurite;
`CREATE EXTENSION vector`; any fourth service — Neo4j, a search platform,
Service Bus or Kafka each require an ADR.

## Completion checklist

- [x] Exactly three services, each pinned by tag and digest
- [x] Non-default published ports
- [x] Each health check is service-native and reads no KnowHub setting
- [x] No configuration parsing anywhere in this task
- [x] `docker compose down -v` resets cleanly
- [x] No credential usable against a deployed environment
