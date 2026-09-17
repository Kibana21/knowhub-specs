# 0B-T16 — Session authority port, binding seam and session cookie

- **Phase:** 5
- **Depends on:** T15
- **Plan:** §13

## Objective

Declare the session authority port with **no backend endpoint shape**, create the
single build-time binding seam that stays unbound in production, and implement
the opaque browser session reference and its rotation.

## Why this task exists

ADR-027 makes the backend authoritative for durable session state and the
frontend BFF a boundary, not an authority. S4 R34 and 0B-AC-073 forbid declaring
any backend authentication endpoint shape at 0B — so the port must exist as an
interface whose production binding is unbound and fails closed.

## Approved requirement references

S4 R17, R18, R19, R20, R22, R23, R24, R25, R34, R35, R36, R42, R43, R45;
S5 R9, R10, R11; ADR-027 §1–§6.

## Acceptance criteria advanced or satisfied

- **0B-AC-065** — opaque ≥128-bit reference; `HttpOnly`, `Secure`, explicit
  SameSite; encodes nothing.
- **0B-AC-068** — no durable frontend session datastore; request-scoped custody.
- **0B-AC-073** — no backend endpoint shape declared.
- **0B-AC-075** — LOCAL traverses the same path shape; no bypass.
- **0B-AC-078** — security-sensitive uncertainty fails closed; no frontend
  session authority.
- **0B-AC-079** — rotation on authentication and re-authentication; cookie
  cleared on sign-out (browser proof in T24).

## Prerequisites

T15.

## Exact scope

1. `src/lib/auth/session-authority.ts` — `SessionAuthorityPort` naming the
   operations the boundary needs — `establish`, `resolve`, `rotate`,
   `terminate`. **No transport, no path, and no request or response shape** —
   including no payload type for how an authorization code or PKCE verifier
   would reach a federated boundary. Those shapes arrive in M1, generated from
   the pinned contract (ADR-017).
2. `src/lib/auth/authority-binding.ts` — the single seam. Production content
   resolves to an **unbound** implementation whose every call yields
   `authentication-error`, so the production runtime fails closed.
3. Session cookie handling: `__Host-kh.session`, 32 random bytes (256-bit)
   base64url, `HttpOnly; Secure; SameSite=Lax; Path=/`. Opaque: it encodes no
   permissions, no identity attributes and no token material.
4. Rotation on authentication and re-authentication; clearing on sign-out; a
   pre-authentication reference is unusable afterwards.
5. `src/app/auth/session/route.ts` — returns the canonical session state for the
   shell. Server-only.
6. The build-time alias point for the verification profile, documented but with
   **no test implementation in `src/`**.

## Expected files

```
src/lib/auth/{session-authority,authority-binding}.ts
src/lib/auth/session.ts        (extended: cookie handling, rotation)
src/app/auth/session/route.ts
tests/component/auth/**
```

## Implementation guidance

**The port declares no shape.** This is the crux of 0B-AC-073: an HTTP adapter
naming `/api/v1/auth/...` with a request body would declare a backend contract
the backend has not published, which S4 R34 prohibits. The same applies to the
**federated-exchange handoff**: giving the port a payload type describing how an
authorization code and PKCE verifier travel to the federated boundary would
declare that contract indirectly. ADR-027 §8 defers OIDC exchange details to M1,
so keep the interface abstract; M1 replaces exactly `authority-binding.ts` with a
binding generated from the pinned contract.

**Unbound means fails closed** (S4 R42; ADR-022 default deny), not "throws
later". Every call on the unbound implementation yields `authentication-error`
and grants nothing.

**The seam is compile-time.** Use a module alias resolved by build
configuration — **no environment flag, no dynamic import, no hidden route, no
runtime profile selector**, any of which could make a synthetic authority
reachable from the production runtime and would breach S5 R10(c).

`SameSite=Lax`, not `Strict`: the OIDC callback in T18 is a cross-site top-level
GET navigation that `Strict` would strip. Lax plus the T17 integrity controls is
how S4 R63 is satisfied — neither relied upon alone.

**`Secure`, `HttpOnly`, the SameSite policy and the `__Host-` prefix are never
relaxed, in any profile** (S4 R36, R38). Whether the **pinned Chromium build**
accepts `Secure` and `__Host-` cookies over HTTP on localhost is **verified by
executing the check in T22**, not assumed here. If they are rejected, the browser
harness uses local TLS. Removing `Secure`, dropping the prefix or creating an
insecure profile is never an acceptable workaround.

Use `crypto.randomBytes(32)`; 256 bits comfortably exceeds the ≥128-bit floor.

## Validation

```
pnpm test && pnpm typecheck && pnpm lint && pnpm build
```

## Required negative tests

- With the production (unbound) binding, resolve any reference → 
  `authentication-error`; nothing is granted.
- A tampered or unknown reference → rejected.
- A pre-authentication reference presented after authentication → rejected.

## Evidence to leave behind

Component tests for the unbound-fails-closed path, rotation and clearing.

## Stop conditions

- **If any durable frontend store — database, Redis, session store, or
  frontend-owned persistence called a cache, session cache, token cache or warm
  store — appears necessary, STOP.** That is ADR-027 §2 and a §84.4 stop
  condition requiring a superseding ADR.
- If sticky routing appears necessary, stop for the same reason (ADR-027 §6).
- If the seam cannot be aliased without a test artifact entering the production
  build, stop: S5 R10 is at risk.

## What must NOT be implemented

- **No backend authentication endpoint request or response shape, and no
  federated-exchange contract** (S4 R34) — including no payload type for an
  authorization-code or PKCE-verifier handoff.
- **No durable frontend session datastore under any name** (S4 R23).
- No token of any kind in browser-reachable storage (S4 R19).
- No synthetic or test authority implementation under `src/` — T21 builds it in
  `tests/`.
- No runtime profile selector, environment flag or dynamic import at the seam.
- No credential verification, password handling or MFA (S4 R37).
- No OIDC protocol logic (T18), no CSRF (T17), no guard (T19).

## Completion definition

The port declares no backend shape; the production binding is unbound and fails
closed; the browser receives only an opaque ≥128-bit `HttpOnly; Secure;
SameSite=Lax` reference encoding nothing; rotation and clearing work; no durable
frontend session store exists.
