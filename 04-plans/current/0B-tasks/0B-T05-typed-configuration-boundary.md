# 0B-T05 — Typed configuration boundary

- **Phase:** 2 — Architecture boundaries and configuration
- **Depends on:** T02, T04
- **Plan:** §10

## Objective

Create the one typed configuration boundary: the only place an environment value
is read, parsed, validated, defaulted or narrowed, with an explicit
server-only/browser-exposed split and fail-closed validation.

## Why this task exists

S4 R1–R11 require exactly one boundary, and S3 R32–R33 require the API boundary
to consume an already-validated backend origin without reading the environment
itself. Both the API layer (T13) and the auth layer (T15+) depend on this
existing first.

## Approved requirement references

S4 R1, R2, R3, R4, R5, R6, R7, R8, R9, R10, R11; S3 R32, R33.

## Acceptance criteria advanced or satisfied

- **0B-AC-060** — one boundary; nothing outside it reads the environment.
- **0B-AC-061** — exposure declared per value; allowlist; secret references;
  validated backend origin produced (bundle half in T28).
- **0B-AC-062** — missing or invalid security-sensitive configuration fails
  closed with a naming message.
- **0B-AC-053** — the API boundary's origin comes from here (consumed in T13).

## Prerequisites

T04 — the boundary rules exist, so `lib-config` can be declared as an element
type with no outgoing edges.

## Exact scope

1. `src/lib/config/schema.ts` — a Zod schema in which **every** value declares
   `exposure: 'server' | 'browser'`. Browser exposure is an explicit allowlist,
   never a default.
2. `src/lib/config/server.ts` — imports `server-only`; exposes server-only
   values, including the OIDC provider registry inputs and the transaction-state
   key **reference**.
3. `src/lib/config/public.ts` — browser-exposed values only.
4. `src/lib/config/index.ts` — typed accessors, including the validated backend
   origin S3 R32 requires.
5. Fail-closed validation at module initialisation: a missing or invalid
   security-sensitive value throws with a message naming what is missing or
   invalid, and never falls back to a weaker mode.
6. `.env.example` — variable **names** and placeholder or reference values only.
7. A lint rule forbidding `process.env` anywhere in `src/**` outside this
   directory.
8. Declare `lib-config` in the boundary graph with **no outgoing edges**.

## Expected files

```
src/lib/config/{schema,server,public,index}.ts
.env.example
eslint.config.mjs        (process.env restriction)
```

## Implementation guidance

**Naming is this plan's choice — S4 R4 fixes none.** Server-only:
`KNOWHUB_<DOMAIN>__<KEY>`. Browser-exposed: `NEXT_PUBLIC_KNOWHUB_<KEY>`. The
framework prefix is required for inlining and doubles as a second signal
alongside the declared allowlist.

**Secrets are references, never values** (S4 R9). A secret-bearing setting is
expressed as an identifier such as `…_CLIENT_SECRET_REF`, resolved server-side
through the environment's approved mechanism. **Select no secret-manager product,
SDK, secret name or credential, and introduce no secret-manager infrastructure
for 0B.**

Fail-closed means at the **earliest safe point** — module initialisation, not
first use. A security-critical production default fails closed rather than open
(S4 R10).

A narrower scope may make behaviour stricter but must never silently weaken a
mandatory control (S4 R11). Express that as a validation rule, not a convention.

## Validation

```
pnpm typecheck && pnpm lint && pnpm test
```

## Required negative tests

- Omit a required security-sensitive value → initialisation throws, and the
  message names the missing key.
- Supply a malformed value → initialisation throws rather than defaulting.
- Attempt `process.env` in a `src/**` file outside the boundary → lint fails.
- Declare a value server-only and attempt to read it from `public.ts` → type
  error.

## Evidence to leave behind

Component tests covering each negative case; `.env.example` with no real value.

## Stop conditions

- If a value cannot be classified as server-only or browser-exposed, stop:
  S4 R5 requires an explicit declaration for **every** value.
- If holding a raw secret as an ordinary configuration value seems necessary,
  stop — S4 R9 forbids it.

## What must NOT be implemented

- No secret-manager product, SDK or infrastructure.
- No backend authentication endpoint configuration, path or shape (S4 R34).
- No identity policy that is backend policy — session or refresh lifetimes,
  bootstrap state, lock or rate policy, password or reset policy, provider-group
  mapping behaviour (S4 §14).
- No real credential, token or `.env` file committed.
- No configuration consumed yet — T13 and T15 consume it.

## Completion definition

One boundary exists and is the only environment reader; every value declares its
exposure; secrets are references; all four negative cases pass;
`.env.example` contains no real value.
