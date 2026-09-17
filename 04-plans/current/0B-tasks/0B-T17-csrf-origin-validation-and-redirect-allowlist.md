# 0B-T17 — CSRF, Origin validation and the redirect allowlist

- **Phase:** 5
- **Depends on:** T16
- **Plan:** §15

## Objective

Protect every **ordinary** state-changing browser → BFF request with a
double-submit token **and** Origin/Referer validation, prohibit open redirects,
and supply the redirect-validation helper the OIDC callback in T18 depends on.

## Why this task exists

S4 R63 is explicit that the cookie policy and the request-integrity control are
chosen **together** so that neither alone is relied upon. Because T16 chose
`SameSite=Lax` — required so the OIDC callback works — the token and Origin
controls carry the weight `Strict` would otherwise have carried.

## Approved requirement references

S4 R31, R60, R61, R62, R63, R64, R65.

## Acceptance criteria advanced or satisfied

- **0B-AC-074** — an out-of-allowlist redirect or callback target is rejected;
  no open redirect exists.
- **0B-AC-084** — cross-origin state-changing requests rejected; same-origin
  succeed; Origin and Referer validated; LOCAL and OIDC protected identically.

## Prerequisites

T16.

## Exact scope

1. `src/lib/auth/csrf.ts` — the double-submit token: `__Host-kh.csrf`,
   `Secure; SameSite=Lax; Path=/`, deliberately **not** `HttpOnly` so the client
   can echo it in `X-KnowHub-CSRF`. One shared helper enforcing both the token
   match and the `Origin` (falling back to `Referer`) allowlist.
2. `src/lib/auth/redirects.ts` — `assertApprovedRedirect()` accepting only a
   same-origin absolute path, or an exact match in the configured
   approved-origin list for post-logout targets.
3. Apply the regime-1 helper to every **ordinary** state-changing BFF route —
   sign-in **initiation** and sign-out — identically for LOCAL and OIDC.
   The OIDC authorization **callback** is not an ordinary browser-originated
   action and is bound differently; see the guidance below.
4. `src/app/auth/sign-out/route.ts` — clears the session cookie and redirects to
   a validated target.
5. Component tests for both controls.

## Expected files

```
src/lib/auth/{csrf,redirects}.ts
src/app/auth/sign-out/route.ts
tests/component/auth/**
```

## Implementation guidance

A CSRF token is **not** session, token or identity material, so a JS-readable
cookie holding one does not touch S4 R19. Say so in the module comment, because
the non-`HttpOnly` flag will otherwise look like an oversight.

**Both controls always, never one** — for regime-1 requests. The token proves the
request came from a page that could read the cookie; the Origin check proves it
came from an approved origin. S4 R63 requires that neither be relied upon alone.

**The OIDC authorization callback is regime 2, not regime 1** (plan §15). It is a
top-level redirect issued by the authorization server: that server cannot set a
custom header and does not present our origin, so requiring the double-submit
header of it would demand a control the redirect cannot supply. Its request
binding is Authorization Code + PKCE + the per-request `state` validated against
the sealed category-B transaction state, plus strict callback-target validation —
all owned by T18, which consumes `assertApprovedRedirect()` from this task.

**S4 R64 is satisfied, not narrowed.** Both providers' **initiation** legs pass
through the identical regime-1 helper, so no provider, mode or profile weakens
request integrity. The OIDC callback additionally carries regime-2 binding that
the LOCAL leg has no equivalent of; neither provider is protected less. R64 is not
read as requiring a mechanism an authorization-server redirect cannot carry.

**T17 precedes T18.** The callback needs `assertApprovedRedirect()` to exist, so
the ordering is an explicit dependency rather than a hidden one.

`assertApprovedRedirect()` must reject `//host`, `\host`, any scheme, and
**percent-encoded variants of each** — encoded-variant bypasses are the usual way
open redirects survive review. Parse rather than pattern-match where possible.

**LOCAL and OIDC pass through the identical helper** (S4 R64). No provider, mode
or profile weakens request integrity — a single shared helper is the simplest way
to make that true by construction rather than by discipline.

This task does not extend into CORS policy (S4 §11 guidance; S5 R47).

## Validation

```
pnpm test && pnpm typecheck && pnpm lint
```

## Required negative tests

- An **ordinary** state-changing request with no CSRF token → rejected.
- With a mismatched token → rejected.
- With a valid token but a foreign `Origin` → rejected.
- A same-origin request with valid controls → succeeds.
- The same four cases run against **both** the LOCAL and the OIDC initiation
  legs, proving S4 R64's equivalence.
- Redirect targets `//evil.example`, `\\evil.example`, `https://evil.example`,
  `%2f%2fevil.example` → each rejected.
- A post-logout target outside the approved list → rejected.

(The cross-origin browser proof is T25; these are the component-level cases.)

## Evidence to leave behind

Component tests for both controls and every redirect bypass shape.

## Stop conditions

- If any **ordinary** state-changing route cannot use the shared helper, stop: a
  second path is how S4 R64 gets broken.
- If the OIDC callback appears to need the regime-1 helper, re-read plan §15
  before adding it: the correct binding for that request is regime 2, and forcing
  regime 1 onto it would make the callback unimplementable rather than safer.

## What must NOT be implemented

- No CORS policy (S5 R47).
- No CSRF pattern the sources do not mention (S4 §11 guidance).
- No relaxation of `Secure` or SameSite in any profile.
- No open redirect, and no "trusted" redirect escape hatch.
- No provider-specific weakening of request integrity (S4 R64).
- **No double-submit CSRF header required of the authorization-server redirect
  callback** — that is regime 2 and belongs to T18.
- No guard or permission logic (T19).

## Completion definition

Every **ordinary** state-changing BFF route enforces both the double-submit token
and Origin/Referer validation through one shared helper; LOCAL and OIDC
initiation are protected identically; `assertApprovedRedirect()` exists and is
available to T18; every redirect bypass shape is rejected; no open redirect
exists. The OIDC authorization callback is correctly excluded from regime 1 and
bound by regime 2, which T18 implements.
