# M00-T08 — Test harness and failure-path tests

- **Status:** Not started
- **Depends on:** T02b, T03, T04, T05, T07
- **Blocks:** T10
- **Plan:** [M00-PLAN-backend-foundation](../M00-PLAN-backend-foundation.md) §3.7, T8

## Owning specification requirements

M00-SPEC-004 R22, R23, R24, R25

## Acceptance criteria contributed to

M00-AC-024, M00-AC-026

## Files created or modified

| Path | Action |
|---|---|
| `tests/conftest.py` | Create |
| `tests/unit/` | Create — the unit tests specified by T03, T04 and T05 |
| `tests/integration/conftest.py` | Create |
| `tests/integration/test_migrations.py` | Create — the T07 migration tests |

## Implementation steps

1. Create **only** `tests/unit/` and `tests/integration/`. No directory for
   contract, golden, parity, security or load layers — each is created by the
   milestone that first supplies real tests for it.
2. `tests/conftest.py` holds shared fixtures. Set the telemetry exporter to
   `none` for unit tests.
3. `tests/integration/conftest.py` gates the layer on the configured
   PostgreSQL being reachable, skipping with a clear message rather than
   failing obscurely.
4. `test_migrations.py` is the **only** integration test at M00, because the
   Alembic scaffolding is the only real M00 integration code.
5. Do not add a Redis or Azurite integration test, and do not add a client,
   fake adapter or placeholder wiring so that either has something to test.
   Their availability is observed by T02b, which is not an integration test.
6. Verify the failure paths SPEC-004 R25 requires are all present and
   passing: configuration, boundary, readiness and migration.
7. Keep tests deterministic and order-independent — no shared mutable state,
   no wall-clock dependence, no network beyond the configured services.

## Tests and evidence

- The full suite passes locally and, by T10, in CI.
- A listing of `tests/` shows only `unit/` and `integration/`.
- A repository search confirms no Redis client, Blob client, fake adapter or
  placeholder integration exists.

## Failure paths to exercise

Aggregates and verifies the failure paths owned by earlier tasks:
configuration rejection and startup failure (T03), correlation rejection and
telemetry-initialisation failure (T04), degraded readiness (T05), boundary
violation (T06), migration against an unreachable database (T07).

## Telemetry expectations

Exporter set to `none` for unit tests; telemetry asserted present for the API
tests.

## Out of scope

Contract, golden, parity, security and load test layers and their
directories. Redis and Blob integration tests — both begin with the M2
functionality that uses them. Playwright or any browser test, which belongs
to `knowhub-frontend`. Mocking a service that M00 has real integration code
for.

## Completion checklist

- [ ] Only `unit/` and `integration/` exist under `tests/`
- [ ] `test_migrations.py` is the only integration test
- [ ] Integration layer runs against the configured PostgreSQL, not a mock
- [ ] Integration layer skips clearly when the endpoint is unreachable
- [ ] No Redis or Blob client, fake adapter or placeholder integration anywhere
- [ ] All four R25 failure-path families exercised
- [ ] Tests deterministic and order-independent
