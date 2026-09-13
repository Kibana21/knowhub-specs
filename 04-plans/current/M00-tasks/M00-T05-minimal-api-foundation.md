# M00-T05 — Minimal API foundation

- **Status:** Not started
- **Depends on:** T03, T04
- **Blocks:** T06, T08, T09
- **Plan:** [M00-PLAN-backend-foundation](../M00-PLAN-backend-foundation.md) §3.6, T5

## Owning specification requirements

M00-SPEC-001 R8, R9, R10, R11; M00-SPEC-003 R2, R10

M00-SPEC-001 R4, R5, R6, R7 — `knowhub.api` is the last package of the M00
inventory, so this task completes and is verified against it.

## Acceptance criteria contributed to

M00-AC-003 — the M00 package inventory is complete and contains no package
for a later-milestone capability. T01, T03 and T04 each uphold it for the
packages they create; this task is where it is verified whole.

M00-AC-008, and M00-AC-015 demonstrated end to end over HTTP

## Files created or modified

| Path | Action |
|---|---|
| `src/knowhub/api/__init__.py` | Create |
| `src/knowhub/api/main.py` | Create — application factory and lifespan |
| `src/knowhub/api/health.py` | Create — `/healthz`, `/readyz` |
| `src/knowhub/api/middleware.py` | Create — correlation middleware |

## Implementation steps

1. Wire in order: load and validate settings → initialise observability →
   construct the application → register routes. Nothing constructs the
   application before configuration has been validated.
2. Use application lifespan for setup and teardown of what the process owns,
   including observability shutdown.
3. `middleware.py` holds the ASGI correlation middleware. It calls
   `knowhub.observability.correlation` for accept, validate and generate, and
   sets the response `X-Correlation-ID` header. The validation logic itself
   stays in the observability package.
4. `health.py` exposes exactly two endpoints:
   - `/healthz` — the process is running and able to serve.
   - `/readyz` — startup completed and M00-scoped prerequisites are
     satisfied, meaning settings loaded and observability initialised.
5. Readiness must not probe the database schema, identity, the run model,
   Redis, blob storage or any connector. None of those exist at M00 and
   reaching for them would broaden the milestone.
6. Neither endpoint exposes a secret, credential, connection string,
   internal host address or any configuration value beyond a coarse status.
   Failure detail is safe detail.
7. Register no other route, router or API surface.
8. Instrument the request path so every request produces a span carrying the
   correlation identifier.

## Tests and evidence

Unit tests via ASGI transport:

- `/healthz` responds with the expected shape.
- `/readyz` responds ready when settings and observability are both good.
- `/readyz` reports not-ready in a controlled, observable way when
  observability initialisation did not complete.
- Neither endpoint discloses a configuration value or secret.
- A valid `X-Correlation-ID` is echoed on the response.
- A malformed inbound `X-Correlation-ID` yields a fresh server-generated
  identifier on the response.
- A request emits a span carrying the correlation identifier.
- A listing of `src/knowhub/` shows exactly `api`, `config` and
  `observability`, and no package for a later-milestone capability.

## Failure paths to exercise

Degraded readiness; malformed inbound correlation header.

## Telemetry expectations

Every request produces a span carrying the correlation identifier. An
unhandled error marks the span failed with a safe message.

## Out of scope

Any business endpoint, router or schema. `api/routers/` — health probes are
not business routers, and the directory arrives with the first business
router in M1. OpenAPI artifact publication. Authentication, authorization or
session handling. Any database, Redis or blob access from a handler.

## Completion checklist

- [ ] Wiring order is settings → observability → app → routes
- [ ] Exactly two endpoints; no other route registered
- [ ] Readiness reflects only settings load and observability initialisation
- [ ] Readiness references no later-milestone dependency
- [ ] Neither endpoint discloses configuration or secrets
- [ ] Correlation middleware lives here, validation stays in observability
- [ ] Every request emits a span carrying the correlation identifier
- [ ] `src/knowhub/` holds exactly the three M00 packages, nothing more
