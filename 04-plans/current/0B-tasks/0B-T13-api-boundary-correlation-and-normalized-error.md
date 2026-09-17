# 0B-T13 — The API boundary, correlation and the normalized error

- **Phase:** 4
- **Depends on:** T05, T11
- **Plan:** §12

## Objective

Build the single approved API boundary with distinct server-side and browser-safe
paths, correlation propagation parameterized over an unbound binding, and exactly
one normalized safe error representation.

## Why this task exists

S3 R26 requires that **all** access to the backend cross one boundary, and S3 R30
makes it the only place a transport outcome becomes an application-level result.
Building it generically now is what lets a real contract bind it at M1 without
edits.

## Approved requirement references

S3 R21, R22, R26, R27, R28, R29, R30, R31, R32, R33, R34, R35, R36, R37, R38,
R39, R40, R41, R42, R43, R44, R45; S4 R21, R22.

## Acceptance criteria advanced or satisfied

- **0B-AC-051** — exactly one boundary; nothing beneath it.
- **0B-AC-052** — server-side and browser-safe paths distinct; no bearer on the
  browser path; no product operation.
- **0B-AC-053** — origin from configuration; no environment read here.
- **0B-AC-054** — correlation preserved, originated or replaced; binding unbound.
- **0B-AC-055** — no browser telemetry; no credential exposure.
- **0B-AC-056** — one normalized error; unsafe material stripped.
- **0B-AC-057** — no streaming implementation or second boundary.

## Prerequisites

T05 (validated backend origin), T11 (the generated `paths` shape to be generic
over).

## Exact scope

1. `src/lib/api/types.ts` — boundary generics over the generated `paths` type.
2. `src/lib/api/client.ts` — the browser-safe path; carries **no** backend bearer
   credential.
3. `src/lib/api/server-client.ts` — the server-side path, where authenticated
   access is later performed.
4. `src/lib/api/correlation.ts` — `CorrelationBinding { headerName, generate,
   isConforming }`, a **required** boundary parameter with **no default**.
5. `src/lib/api/errors.ts` — the one `ApiError`: `code`, `message`,
   `correlationId`, `retryable`, optional `remediation`, plus the HTTP status
   class where meaningful.
6. `tests/fixtures/correlation/synthetic-binding.ts` — the clearly synthetic
   test-only binding (`X-Synthetic-Correlation`, `syn-<uuid-v4>`).
7. Component tests for every behaviour above.

## Expected files

```
src/lib/api/{types,client,server-client,correlation,errors}.ts
tests/fixtures/correlation/synthetic-binding.ts
tests/component/api/**
```

## Implementation guidance

The boundary is **generic over whatever operations its input describes** (S3 R31)
— it defines no product operation. Write it against `openapi-fetch`'s `paths`
generic so a real contract binds it later without edits.

It reads **no environment variable** and performs no parsing, validation or
defaulting of configuration (S3 R33); it consumes the already-validated origin
from T05.

**Correlation stays unbound** (S3 R36). Make the binding a required constructor
argument with no default, so no product header name or identifier representation
can be established by accident. A conforming inbound identifier is preserved and
forwarded; an absent one is originated; a **non-conforming one is replaced, never
propagated**. The synthetic binding is clearly named as synthetic and lives under
`tests/`.

> A correlation header name appearing in `knowhub-backend` implementation source
> is **not** an authoritative source (S3 R36) and must not be adopted here.

Error normalization is the only place a transport outcome becomes an
application-level result (S3 R30). `retryable` stays unset unless the contract
states it — the frontend infers no retry semantics (S3 R40). Nothing unsafe
survives: no raw exception text, stack trace, sensitive diagnostic detail,
internal configuration, credential, secret or token (S3 R41).

This task produces the representation; **T08 renders it**. Define no presentation
here (S3 R42).

## Validation

```
pnpm test && pnpm lint && pnpm typecheck
```

## Required negative tests

- A failure carrying exception text, a stack trace, internal configuration and a
  credential → none survives into `ApiError`.
- A non-conforming inbound correlation identifier → replaced, not propagated.
- An absent identifier → originated.
- A contract that does not state retryability → `retryable` stays unset.

## Evidence to leave behind

Component tests covering each case; the synthetic binding under `tests/`.

## Stop conditions

- If a product operation or a real header name seems necessary to make the
  boundary work, stop: S3 R31 and R36 forbid both at 0B.

## What must NOT be implemented

- **No product operation, no product API path, no product schema** (S3 R31).
- **No real KnowHub correlation header name and no product identifier
  representation** (S3 R36).
- **No streaming implementation**: no SSE or websocket schema, run event model,
  resumption token format or product event name (S3 R45).
- No browser tracing SDK, vendor telemetry integration or metrics framework
  (S3 R37).
- No second boundary; no route handler or server action that constitutes an
  independent backend integration.
- No error presentation or rendering logic (S3 R42).
- No session, token or credential handling — T15–T20 own the mechanics.

## Completion definition

Exactly one boundary exists with distinct server-side and browser-safe paths; the
origin comes from configuration; correlation is parameterized and unbound; one
normalized error exists and strips everything unsafe; no product operation,
streaming implementation or telemetry system has been introduced.
