# 0B-T19 — Route/session guard, permissions port and access components

- **Phase:** 5
- **Depends on:** T16, T09, T14
- **Plan:** §16

## Objective

Create the one canonical route and session guard, the one permissions port bound
to nothing real, and the access components that render an authorization
determination.

## Why this task exists

S4 R54 requires exactly one guard abstraction and S4 R49 exactly one permissions
port. ADR-022 makes authorization backend-authoritative and default-deny, so an
unbound port must fail closed rather than default to permissive — which is only
true if it is built that way from the start.

## Approved requirement references

S4 R41, R42, R44, R45, R46, R47, R48, R49, R50, R51, R52, R53, R54, R55, R56,
R57, R58, R59; S2 R17, R26; ADR-022, ADR-023, ADR-027 §4.

## Acceptance criteria advanced or satisfied

- **0B-AC-028** — an unauthorized item is absent, not disabled.
- **0B-AC-077** — unauthenticated initiates sign-in; forbidden reaches
  access-denied.
- **0B-AC-078** — uncertainty fails closed; no frontend session authority.
- **0B-AC-080** — a session transition invalidates cached server state.
- **0B-AC-081** — one port; unknown fails closed; no taxonomy; no TS evaluator.
- **0B-AC-083** — one guard at the server boundary; deterministic deny; no
  product protected route.

## Prerequisites

T16 (session states), T09 (the shell that renders navigation), T14 (the query
layer that invalidation flows through).

## Exact scope

1. `src/lib/auth/guard.ts` — `resolveSessionState()` and `requireSession()`,
   server-only, called at the **server boundary** of any protected segment.
2. `src/lib/auth/permissions.ts` — one `PermissionsPort` returning
   `{ known: false }` or `{ known: true, allows(capabilityId: string) }`. The
   0B binding is the unbound one, returning `{ known: false }`.
3. `src/components/access/` — a capability gate, the access-denied surface and a
   session-state badge, composing T07 primitives and T08 state presentations.
4. `src/app/auth/access-denied/page.tsx` — the forbidden surface.
5. Wire the session transition to `resetServerState()` from T14.
6. Component tests for every behaviour.

## Expected files

```
src/lib/auth/{guard,permissions}.ts
src/components/access/*.tsx
src/app/auth/access-denied/page.tsx
tests/component/auth/**
```

## Implementation guidance

**The Next 16 Proxy is not the guard.** The guard is called at the server
boundary of the segment being protected — a server component layout or route
handler — so the check happens before the browser receives protected content
(S4 R55). The Proxy (`src/proxy.ts`, T27) carries the header baseline and
nothing security-deciding.

**Unbound means every capability query fails closed** (S4 R50). Write
`{ known: false }` as the default return, not as an error path — a capability is
never presented as usable without a **positive** determination.

**Rendering an authorization determination is not enforcement** (S4 R47; S2 R17).
An item the shell is told is unauthorized is **absent**, never rendered and
disabled. Hidden UI is not a security control and no trust decision derives from
one — the backend authorizes every operation independently.

Protected content is never rendered and then hidden after hydration where that
would disclose it (S4 R56).

Redirect and deny behaviour is **deterministic**: the same state yields the same
outcome (S4 R57). Make that a property test.

No feature implements its own authentication check and no feature reads a cookie
or token directly (S4 R58) — that is why the guard is one abstraction, not a
convention.

## Validation

```
pnpm test && pnpm typecheck && pnpm lint && pnpm build
```

## Required negative tests

- Unknown effective authorization → the capability is not presented.
- A loading or pending state → grants no access.
- An indeterminate session → grants nothing.
- The unauthorized-navigation fixture → the item is absent, not disabled.
- The same state twice → the same outcome (determinism).

## Evidence to leave behind

Component tests for the fail-closed matrix, the absent-item rendering and
determinism.

## Stop conditions

- If an entitlement decision seems to need deriving in TypeScript, stop:
  S4 R48 forbids any frontend policy evaluator, and ADR-022 is the reason.

## What must NOT be implemented

- **No permission name, role name, DTO or product permission taxonomy**
  (S4 R49).
- **No TypeScript RBAC, ABAC or policy evaluator** deriving entitlement from
  roles, provider claims or client state (S4 R48).
- **No product protected route** (S4 R59) and no product route of any kind.
- No trust in browser- or provider-supplied role or permission claims (S4 R52).
- No permission inferred from a display role label.
- No administrative bypass (ADR-023).
- No feature-local authentication check and no direct cookie or token read
  (S4 R58).
- No user, role or permission administration UI — M1.

## Completion definition

Exactly one guard and one permissions port exist; the port is unbound and every
capability query fails closed; an unauthorized item is absent rather than
disabled; deny behaviour is deterministic; a session transition invalidates
cached server state; no product protected route and no permission taxonomy
exists.
