# 0B-T15 — Provider-selection shell and the canonical session

- **Phase:** 5 — BFF, session, OIDC, CSRF, guard, permissions
- **Depends on:** T05
- **Plan:** §13

## Objective

Define the enabled-provider set and the one provider-neutral canonical session
shape with its six states.

## Why this task exists

ADR-025 and S4 R15 require one canonical session regardless of provider, and
S4 R12 requires the provider model to express an **enabled provider set** rather
than a two-valued mode, so HYBRID — local plus one or more OIDC providers
simultaneously — is representable. Everything in phase 5 consumes these types.

## Approved requirement references

S4 R12, R13, R14, R15, R16, R35, R39; ADR-021, ADR-025, ADR-027 §5.

## Acceptance criteria advanced or satisfied

- **0B-AC-063** — configuration-driven selection through one stable interface;
  LOCAL, OIDC and HYBRID representable; no provider hard-coded.
- **0B-AC-064** — provider-neutral semantics; no downstream provider branch;
  provider identity displayable only.
- **0B-AC-075** — LOCAL terminates in the same canonical abstraction (shape half).
- **0B-AC-077** — six states represented and distinguishable.

## Prerequisites

T05 — the configuration boundary produces the provider registry inputs.

## Exact scope

1. `src/lib/auth/providers.ts` — one `AuthProviderRegistry` produced by the
   configuration layer: `local.enabled` plus zero or more OIDC providers keyed by
   an opaque `providerId`. Server-only.
2. `src/lib/auth/session.ts` — the canonical session as a discriminated union
   over the six S4 R39 states: `unauthenticated`, `authenticated`, `expired`,
   `revoked`, `forbidden`, `authentication-error`.
3. `providerId` available on the authenticated variant as a **displayable
   attribute only**.
4. Component tests: all three modes representable; multiple simultaneous OIDC
   providers representable; all six states distinguishable.

## Expected files

```
src/lib/auth/{providers,session}.ts
tests/component/auth/**
```

## Implementation guidance

Model the enabled provider **set**, not a mode enum. S4 R12 is explicit that
HYBRID means local plus one or more OIDC providers enabled simultaneously — an
enum cannot express that, and retrofitting it later would touch every consumer.

**One canonical session shape.** There is no `LocalSession`/`OidcSession` split,
and no downstream canonical consumer, route guard, permission gate or entitlement
decision branches on provider type (S4 R15). Provider-conditional behaviour is
confined to the authentication boundary's protocol mechanics in T18 and changes
no entitlement, permission or identity semantics (S4 R16).

**No provider identity is hard-coded** in any route, component or module — no
page names Entra, Okta, Keycloak or any other provider (S4 R14).

§0E.13.2's `IdentityProvider` Protocol is a **backend** abstraction. Do not mirror
it into TypeScript; under ADR-017 the real types arrive generated at M1.

Mark these modules `server-only`.

## Validation

```
pnpm test && pnpm typecheck && pnpm lint
```

## Required negative tests

- Configure LOCAL only, OIDC only, and HYBRID with two OIDC providers → all three
  representable.
- Grep for any hard-coded provider name in `src/**` → none.

## Evidence to leave behind

Component tests for the three modes and the six states.

## Stop conditions

- If a downstream consumer appears to need the provider type for anything beyond
  display, stop: S4 R15 forbids it and ADR-025 is the reason.

## What must NOT be implemented

- **No `LocalSession`/`OidcSession` split** (S4 R15).
- **No hard-coded provider identity** anywhere (S4 R14).
- No mirror of §0E.13.2's backend `IdentityProvider` Protocol.
- **No backend authentication endpoint request or response shape** (S4 R34).
- No credential store, no credential verification, no password handling (S4 R37).
- No login page, provider chooser, provider branding or credential-entry UI —
  all M1 (S4 §1.1).
- No session cookie yet (T16), no OIDC protocol (T18).

## Completion definition

One provider registry expresses an enabled provider set covering LOCAL, OIDC and
HYBRID including multiple simultaneous OIDC providers; one canonical
provider-neutral session shape exists with all six states; no provider identity
is hard-coded and no downstream consumer branches on provider type.
