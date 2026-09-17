# 0B-T14 — Query client and cache-key convention

- **Phase:** 4
- **Depends on:** T03, T13
- **Plan:** §17

## Objective

Establish TanStack Query as the server-state foundation: one query client with
central defaults at the single composition point, and one cache-key convention
able to carry authoritative scope while declaring none.

## Why this task exists

S3 R46 fixes TanStack Query as the server-state foundation and S3 R47 requires
exactly one configuration composed at the one composition point. S4 R44 then
depends on this layer to invalidate cached state on a session transition — so the
convention must exist before the guard does.

## Approved requirement references

S3 R46, R47, R48, R49, R50, R51, R52; S4 R44, R53.

## Acceptance criteria advanced or satisfied

- **0B-AC-058** — one client; central defaults; keys through one convention; no
  product query or key; no duplicated server state; invalidation through the
  layer.
- **0B-AC-080** — a session transition invalidates cached server state
  (mechanism half; the session trigger is T19).

## Prerequisites

T03 (`providers.tsx`), T13 (the boundary the query layer sits above).

## Exact scope

1. `src/lib/query/client.ts` — a `makeQueryClient()` factory holding **all**
   application-level defaults.
2. `src/app/providers.tsx` — composes the provider at the one composition point.
3. `src/lib/query/keys.ts` — `createQueryKey({ scope, resource, params })`, the
   only way a key is built. `QueryScope` **can** carry authoritative scope
   dimensions and **declares none** at 0B.
4. A `resetServerState()` helper the session layer calls on a transition.
5. Component tests, using test-only synthetic state where an executable
   demonstration is needed.

## Expected files

```
src/lib/query/{client,keys}.ts
src/app/providers.tsx        (extended)
tests/component/query/**
```

## Implementation guidance

**Create the client per request on the server and once per browser.** A module-level
singleton shared across server requests would leak one user's cache into another's
response — a security defect, not just a correctness one. The standard pattern is
a server-side factory call plus a browser-side memoised instance.

Defaults are implementation-plan choices where the blueprint mandates none
(S3 R47). Reasonable starting values: `staleTime` 30 s, `gcTime` 5 min, `retry` 1,
`refetchOnWindowFocus` false, `throwOnError` false. **No feature or query module
defines its own default policy.**

The cache key must be able to express an authoritative scope dimension so a
cached result is never served across a scope boundary (S3 R49; S4 R53) — and must
declare none at 0B, because the capabilities defining scope do not exist.

For the session-transition case, clearing the client outright is the deterministic
way to guarantee nothing survives an authorization scope change. Prefer it to
selective invalidation, which is easy to get subtly wrong.

## Validation

```
pnpm test && pnpm typecheck
```

## Required negative tests

- Build a key by hand instead of through the convention → rejected by review and
  by the convention's typing.
- Simulate a session transition → cached synthetic state is gone afterwards.

## Evidence to leave behind

Component tests for the single client, central defaults, key construction and
transition invalidation.

## Stop conditions

- If a second server-state store seems necessary, stop: S3 R50 forbids
  duplicating backend server state into another client store.

## What must NOT be implemented

- **No product query and no product cache key** — no application, user, source,
  Ask, graph or other domain query (S3 R52).
- No scope dimension declared (S3 R49).
- No second server-state store, and no duplication of backend state (S3 R50).
- No per-feature query-client default policy (S3 R47).
- No devtools package — a browser debug surface is at odds with §0E.17.8.
- No ad-hoc cross-component invalidation signalling (S3 R51).

## Completion definition

Exactly one query client with central defaults exists at the one composition
point; every key is built through the one convention; the convention can express
scope but declares none; a session transition clears cached server state; no
product query or key exists.
