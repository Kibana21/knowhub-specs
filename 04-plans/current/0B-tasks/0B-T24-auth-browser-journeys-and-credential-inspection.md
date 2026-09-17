# 0B-T24 — Auth browser journeys and credential inspection

- **Phase:** 6
- **Depends on:** T22
- **Plan:** §13, §18

## Objective

Drive the authentication and session boundary in a real browser through the
synthetic double, and inspect the browser for any token, credential or
redaction-list value. Two distinct journeys, because 0B owns two distinct
properties and conflating them would overstate what the milestone proves.

## Why this task exists

S5 R31 requires browser inspection of the storage and cookie properties S4
R17–R20 demand, across every surface a token could hide in. That can only be done
in a browser, and it is the evidence that fully discharges AT-027.

## Approved requirement references

S5 R30, R31, R32, R36, R37, R38; S4 R17, R18, R19, R20, R21, R22, R31, R32, R33,
R43, R65, R66–R70.

## Acceptance criteria advanced or satisfied

- **0B-AC-065** — cookie entropy and hardened attributes, in the browser.
- **0B-AC-066 / 0B-AC-101** — no token in any browser-reachable location.
- **0B-AC-067 / 0B-AC-102** — no browser `Authorization` header; server-side
  authorization response; no product auth surface created.
- **0B-AC-072** — the authorization response is handled server-side and an
  **authentication failure fails closed**, which is journey A's expected outcome.
- **0B-AC-072** — nothing secret reaches browser application code.
- **0B-AC-079 / 0B-AC-106** — rotation, fixation resistance, cookie clearing, and
  the redaction sweep.
- **0B-AC-085** — non-leakage at the browser surface.

## Prerequisites

T22.

## Exact scope

`tests/e2e/auth/`:

### A — The OIDC transaction journey (`tests/e2e/auth/transaction/`)

Driven in a real browser against T21's authorization-redirect test authority:
authorization initiation → synthetic OIDC provider → authorization callback →
`state` validation, category-B unseal and redirect-target validation → **the
expected fail-closed result at the unbound federated boundary**. No product
authentication UI is created.

1. The authorization request is constructed correctly and the browser is
   redirected to the configured provider.
2. The callback is handled **server-side**; `state` is validated; the
   transaction cookie is unsealed and then cleared.
3. A callback or redirect target outside the approved set is rejected.
4. Replay of a completed or expired transaction is rejected.
5. **No provider token, authorization code, client credential or PKCE verifier
   reaches browser application code or any browser-reachable storage.**
6. The journey terminates in the **fail-closed** federated-boundary result,
   asserted positively as the expected 0B outcome.

### B — The session-boundary journey (`tests/e2e/auth/session/`)

Exercises the synthetic **session authority** through the `SessionAuthorityPort`,
independently of the OIDC leg, for the properties 0B actually owns:

7. Opaque session reference: opaque value, `HttpOnly`, `Secure`, SameSite,
   encoding no permissions or identity attributes.
8. Resolution across the six canonical session states.
9. Storage sweep once a session exists: `localStorage`, `sessionStorage`,
   IndexedDB, the URL path, query string and fragment, JavaScript-readable
   cookies, page props and hydrated state — asserting **no** KnowHub access,
   refresh or session token and no identity-provider token.
10. Network assertions: browser code constructs no backend `Authorization`
    header.
11. Rotation and revocation **through the canonical session-authority interface
    and the existing test seam**: the reference rotates on authentication and
    re-authentication; a pre-authentication reference cannot be reused; the
    cookie is cleared on sign-out; a stale reference is rejected.
12. A redaction sweep over console output, network responses and rendered
    surfaces for every §0E.13.9 value plus claims payloads, stack traces,
    internal provider responses and client secrets.

**Journey B's session is established through the session-authority interface, and
is not claimed to have been established by a federated token exchange** — real or
synthetic. No Approved specification provides such an interface at 0B.

## Expected files

```
tests/e2e/auth/transaction/**      journey A
tests/e2e/auth/session/**          journey B
```

## Implementation guidance

Read `HttpOnly` cookies through the Playwright **context**, not through page
JavaScript — the point of the assertion is that page JavaScript cannot see them.

Assert the absence of an `Authorization` header by observing requests the browser
actually issues, not by inspecting source. The property is about runtime
behaviour.

For fixation resistance, capture the reference before authentication and attempt
to reuse it afterwards; the synthetic authority must reject it. Use the
**canonical session-authority interface and the existing test seam** to drive
rotation and re-authentication — **do not invent an OIDC exchange merely to
produce that evidence**. 0B-AC-079 and 0B-AC-106 both require this evidence
"against the synthetic session authority", which is exactly journey B.

**Keep the two journeys' claims separate.** Journey A proves the OIDC
*transaction* shell and ends, correctly, in a fail-closed result at the unbound
federated boundary. Journey B proves the session boundary through the interface
that exists. Neither journey claims a successful federated token exchange, ID-token
validation, OIDC-authenticated session creation or end-to-end federated login —
none of which 0B delivers, and none of which any Approved criterion requires.

**Record the non-claims in each suite's own header** (S5 R38): no real backend
token rejection, session invalidation, server-side token-cache invalidation,
refresh-token rotation, LOCAL credential verification, federated-token validation,
OIDC tenant integration or backend authorization is claimed — and, for journey A,
no successful federated token exchange and no OIDC-authenticated session. AT-028
and AT-026 remain **partial**; AT-027 is fully discharged by journey B's storage
sweep.

## Validation

```
pnpm test:e2e --project=chromium-desktop -- tests/e2e/auth/transaction
pnpm test:e2e --project=chromium-desktop -- tests/e2e/auth/session
```

## Required negative tests

- Deliberately place a token-shaped value in `localStorage` → the sweep **fails**;
  remove it and confirm green. This proves the sweep can detect what it claims to.
- Reuse a pre-authentication reference → rejected.
- Present a stale reference after sign-out → rejected.
- Journey A's fail-closed outcome is asserted **positively**: if the callback ever
  yielded an authenticated session, the assertion fails — that would mean a
  federated exchange had been introduced somewhere it must not exist.

## Evidence to leave behind

Playwright report sections for the storage sweep, the cookie assertions, rotation
and redaction; the non-claim header recorded in the suite.

## Stop conditions

- If any token appears in a browser-reachable location, stop: S4 R19 is
  absolute and there is no acceptable mitigation short of removing it.
- **If a criterion appears to require successful federated authentication, STOP
  and surface the exact criterion and the conflicting requirement** rather than
  defining the forbidden exchange contract to satisfy it. (None does: 0B-AC-072
  expects an authentication failure to fail closed, and 0B-AC-079 and 0B-AC-106
  scope their evidence to the synthetic **session** authority.)

## What must NOT be implemented

- **No product login page, provider chooser, credential-entry UI or
  identity-provider branding** created in order to test (S5 R30; S4 §1.1).
- No real credentials, no real OIDC tenant.
- **No claim of real backend session invalidation, token-cache invalidation,
  refresh rotation, credential verification, federated validation or backend
  authorization** (S5 R38).
- No relaxation of a cookie flag to make a test pass.
- **No claim of successful federated token exchange, ID-token validation,
  OIDC-authenticated session creation or end-to-end federated login**, and no
  invented exchange mechanism to produce one.
- No security-negative cases (T25) or multi-instance cases (T26).

## Completion definition

Journey A proves the OIDC transaction shell in a real browser and terminates in
the expected **fail-closed** federated-boundary result, with no token, code or
verifier reaching browser application code. Journey B proves the session boundary
through the canonical session-authority interface: opaque reference, the six
states, storage sweep, header assertion, rotation, revocation, cookie clearing
and redaction. No product authentication UI was created; the storage sweep has
been demonstrated capable of failing; and the non-claims — including the absence
of any federated exchange — are recorded in each suite's header.
