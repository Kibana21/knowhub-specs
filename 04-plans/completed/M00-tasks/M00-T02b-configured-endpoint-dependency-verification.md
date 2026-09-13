# M00-T02b — Configured-endpoint dependency verification

- **Status:** Completed
- **Depends on:** T02a, T03
- **Blocks:** T08
- **Plan:** [M00-PLAN-backend-foundation](../M00-PLAN-backend-foundation.md) §3.3, §3.8, T2b

Runs **after T03** so that configured endpoints are read through the
canonical settings model. See the plan's sequencing rule: this task must not
introduce a second configuration mechanism.

## Owning specification requirements

M00-SPEC-004 R1, R1.2, R1.3, R3, R4 (minimum version), R5 and R6
(availability clauses), R24

M00-SPEC-002 R18 — `.env.example` documents every developer-supplied setting
with placeholder values only. T03 built the configuration model but does not
own this file.

## Acceptance criteria contributed to

M00-AC-018 (the externally supplied service clause), M00-AC-019 (the
configured instance clause), and **M00-AC-012's `.env.example` clause**,
which completes the criterion T03 demonstrated on the code side.

## Files created or modified

| Path | Action |
|---|---|
| `scripts/check_dependencies.py` | Create |
| `.env.example` | Create |
| `README.md` | Modify — dependency and developer-loop sections |

## Implementation steps

1. `scripts/check_dependencies.py` imports `knowhub.config` and obtains every
   endpoint from the canonical settings model. It must not define its own
   parser, precedence, defaults or validation for `KNOWHUB_*` values.
2. PostgreSQL check: connect with `psycopg`; assert `server_version_num` is
   at least 150000; assert `vector` appears in `pg_available_extensions`.
   Do **not** execute `CREATE EXTENSION`.
3. Redis check: open a socket, send the inline command `PING\r\n`, assert the
   reply is `+PONG`. Report `-NOAUTH` clearly as *"Redis reachable,
   credentials required"* — never as a healthy unauthenticated connection.
   Minimum exchange only; no `AUTH`, no further commands.
4. Azurite check: plain HTTP request to the configured blob endpoint using
   `httpx`.
5. Exit non-zero with a specific, named reason per failing service. Never
   print a credential, resolved secret or connection string.
6. Write `.env.example` documenting **both** developer paths — Mode A
   existing services, Mode B managed Compose — with every value presented as
   a configuration example, not an architectural fact. Placeholders only;
   secrets appear solely as `SecretRef` placeholders.
7. Extend `README.md` with the two dependency modes and the identical
   post-setup sequence: configure, verify, sync, migrate, run, validate.

## Tests and evidence

- The check passes against the T02a environment.
- The check passes against an externally supplied PostgreSQL and Redis, with
  only the absent services started.
- Availability observation is the evidence. Per SPEC-004 R24 this is **not**
  an integration test.

## Failure paths to exercise

- Each service unreachable → non-zero exit naming that service.
- PostgreSQL below the minimum major → named failure.
- PostgreSQL without `vector` available → named failure.
- Redis replying `-NOAUTH` → reported as credentials-required, not healthy.
- No failure message discloses a credential or connection string.

## Telemetry expectations

None. This is developer and CI tooling, not application code.

## Out of scope

Any Redis Python dependency; any Azure Storage SDK; any reusable client
abstraction; `AUTH` or any Redis command beyond `PING`; `CREATE EXTENSION
vector`; anything under `src/knowhub/`; committing a workstation-specific or
personal credential.

## Completion checklist

- [x] Every configured endpoint comes from `knowhub.config` — no second parser
- [x] Lives in `scripts/`, never under `src/knowhub/`
- [x] No Redis or Azure dependency added
- [x] Redis exchange is PING/PONG only; `-NOAUTH` reported clearly
- [x] PostgreSQL minimum and `vector` availability asserted, extension not created
- [x] `.env.example` documents both modes with placeholder values only
- [x] `.env.example` documents every developer-supplied setting (SPEC-002 R18)
- [x] M00-AC-012's `.env.example` clause demonstrated
- [x] No personal or machine-specific credential committed
- [x] No failure message leaks a secret
