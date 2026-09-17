# 0B-T18 — OIDC shell, PKCE and sealed transaction state

- **Phase:** 5
- **Depends on:** T16, T17
- **Plan:** §13, §14

## Objective

Implement the OIDC protocol shell — Authorization Code with PKCE, `state` and
nonce — and the category-B transaction-state mechanism that carries them across
the redirect/callback round trip without becoming session authority.

## Why this task exists

S4 R26–R33 fix the protocol shell, and S4 R30 leaves the category-B mechanism
deliberately implementation-defined while imposing eight properties it must all
satisfy. Choosing and proving that mechanism is the sharpest design obligation in
0B.

## Approved requirement references

S4 R16, R26, R27, R28, R29, R30, R31, R32, R33, R34; ADR-021, ADR-027 §2, §6.

## Acceptance criteria advanced or satisfied

- **0B-AC-070** — Authorization Code with PKCE; no implicit path; `state`
  generated per request and validated at the BFF; nonce generated per request and
  bound to the pending transaction.
- **0B-AC-071** — transaction state integrity-protected, verifier confidential,
  expiring, cleared, non-reusable.
- **0B-AC-072** — server-side authorization response; nothing secret reaches the
  browser; failure fails closed.
- **0B-AC-073** — no issuer/audience/signature/JWKS/expiry/tenant/nonce-claim
  validation; no federated-exchange contract.

## Prerequisites

T16.

## Exact scope

1. `src/lib/auth/oidc.ts` — Authorization Code with PKCE via `openid-client`,
   server-only, **restricted to the protocol-mechanics APIs listed below**.
   Generates a `state` per authorization request; generates a `nonce` per
   authorization request and **binds it to the category-B transaction state**;
   initiates through the provider registry.
2. `src/lib/auth/transaction-state.ts` — the sealed transaction cookie:
   JWE (`dir` + `A256GCM`, via `jose`), `__Host-kh.oidc-txn`,
   `HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=300`, payload `{ state,
   nonce, codeVerifier, providerId, returnTo, exp }`, sealed with a server-only
   key supplied as a **secret reference** through the configuration boundary.
3. `src/app/auth/sign-in/route.ts` — initiation. Server-only route handler.
4. `src/app/auth/callback/route.ts` — the authorization response, handled
   server-side. Unseals the transaction state and **validates `state`**; a
   mismatched, missing or unrecognised value **fails closed**. Validates the
   callback target through T17's `assertApprovedRedirect()`. Performs **no token
   exchange** and **declares no federated-exchange contract**; the production
   federated-authority binding remains unbound and fail-closed, so in a
   production profile the callback terminates in `authentication-error`. Clears
   the transaction cookie on completion, failure and expiry.
5. Component tests for every property in the S4 R30 table.

## Expected files

```
src/lib/auth/{oidc,transaction-state}.ts
src/app/auth/sign-in/route.ts  src/app/auth/callback/route.ts
tests/component/auth/**
```

## Implementation guidance

**Verify the mechanism against all eight S4 R30 properties before writing tests**,
using the plan §14 table as the checklist. Each property has a named mechanism;
none is assumed.

**Why a sealed cookie is not ADR-027 Alternative A.** ADR-027 rejected sealed
browser-carried **session** material because a sealed session survives revocation.
This seals only transaction correlation: no subject, no permission, no session
reference, 300 seconds, deleted on every terminal outcome. ADR-027 §8 leaves it
open, §0E.13.7 calls it a "transient/token cache during OIDC flow", S4 R30
expressly permits it, and S5 §14 expressly states a mechanism is **not**
implicated merely because it works across frontend instances. Record this
reasoning in the module's header comment so a later reader does not re-litigate
it.

**`iron-session` was rejected deliberately** — adopting a library framed as a
*session* store would blur the category A/B line this design depends on. `jose`
seals a transaction, names nothing a session, and is already needed for the OIDC
leg.

The sealing key is **configuration**, not frontend-owned state — which is exactly
why any instance can complete a transaction another started, with no sticky
routing and no shared runtime store.

**The BFF validates nothing federated** (S4 R34): no issuer, audience, client ID,
signature, JWKS, expiry, not-before, tenant policy or **ID-token nonce claim**.
That happens at the federated boundary, which is backend and M1. How a nonce
reaches that boundary is an M1 contract question this task does not answer and
does not claim. Stated precisely: **`state` is generated and validated by the
BFF; the nonce is generated and bound to category-B transaction state; ID-token
nonce-claim validation is not performed at 0B.**

**Permitted `openid-client` 6.8.8 APIs at 0B**, and only these:
`randomPKCECodeVerifier()`, `calculatePKCECodeChallenge()`, `randomState()`,
`randomNonce()`, `buildAuthorizationUrl()`, `buildEndSessionUrl()` and
`discovery()`.

**Prohibited**, because each performs validation S4 R34 reserves to the federated
boundary: **`authorizationCodeGrant()`** — it exchanges the code *and* validates
the ID token's issuer, audience, signature/JWKS, `exp`/`nbf` and, through
`expectedNonce`, the **nonce claim** — together with `implicitAuthentication()`,
`refreshTokenGrant()`, `fetchUserInfo()`, `tokenIntrospection()` and
`tokenRevocation()`.

**The 0B BFF therefore performs no token exchange, and declares no
federated-exchange contract** (S4 R34). What this task owns at the callback is
exactly: the authorization response stays server-side; `state` is validated
against the sealed transaction; callback and redirect targets are validated; the
transaction cookie is cleared on every terminal outcome; and the production
federated-authority binding stays unbound and fail-closed.

**The concrete handoff is an M1 contract question this task does not answer.**
How the authorization code and PKCE verifier reach the federated boundary — the
request and response shape of that exchange — is deferred to M1 exactly as the
nonce's route to that boundary is. §0E.13.3 places the exchange at the FastAPI
identity boundary, S4 R34 forbids the frontend declaring its contract, and
ADR-027 §8 defers OIDC exchange details to M1. **Writing that shape now, in any
form, would be declaring the contract in all but name.**

The test-only synthetic authority may use a harness-specific mechanism to
demonstrate completion (T21). That mechanism stays under test-only containment
and establishes no product contract and no M1 API assumption.

**Request binding on the callback.** The authorization response is a top-level
redirect issued by the authorization server: it cannot set a custom header and
does not present our origin, so the ordinary double-submit CSRF header is not
applicable to it. Its binding is Authorization Code + PKCE + the per-request
`state` validated against the sealed transaction, plus strict callback-target
validation (plan §15, regime 2). Sign-in **initiation** is an ordinary
state-changing browser → BFF request and does pass through T17's regime-1 helper,
identically for LOCAL and OIDC — which is where S4 R64 binds.

Redirect-safety integration is an explicit ordering dependency, not a hidden
one: **T17 precedes T18**, so `assertApprovedRedirect()` already exists when the
callback is written (S4 R31).

## Validation

```
pnpm test && pnpm typecheck && pnpm lint && pnpm build
```

## Required negative tests

- A mismatched `state` → fails closed.
- A missing `state` → fails closed.
- An unrecognised `state` → fails closed.
- A tampered transaction cookie → decryption fails, request fails closed.
- An expired transaction cookie → rejected and cleared.
- A completed transaction's cookie replayed → rejected.

## Evidence to leave behind

Component tests covering the eight R30 properties and all six negative cases.

## Stop conditions

- **If the mechanism must become durable or cross-transaction, become a product
  session or authorization store, require sticky routing, or introduce
  frontend-owned durable or shared session or authorization persistence — STOP.**
  That is category A, contradicts ADR-027 §2 and §6, and is a §84.4 stop
  condition requiring a superseding ADR (S4 §14; S5 §14).
- If `openid-client` requires browser-side token custody, relocates session
  authority or moves entitlement into the frontend, stop — that breaches
  ADR-021, ADR-022 or ADR-027.
- **If the protocol mechanics 0B owns cannot be performed without calling a
  prohibited API — one that performs issuer, audience or client-ID, signature or
  JWKS, `exp`/`nbf`, tenant-policy or ID-token nonce-claim validation — STOP.**
  Do not work around the specification. The library is a reversible
  implementation choice: replace it with a mechanism that performs only the
  protocol mechanics 0B owns, and record the substitution.

## What must NOT be implemented

- **No OAuth implicit flow and no implicit-flow code path** (S4 R27).
- **No federated validation of any kind** and **no federated-exchange contract**
  (S4 R34). In particular **no ID-token nonce-claim validation** — the nonce is
  generated and bound here and validated at the backend federated boundary in M1.
- **No declared shape for the authorization-code / PKCE-verifier handoff** to the
  federated boundary, in production source or in the port's typing. That shape is
  M1's, and declaring it here — even indirectly, through a payload type on the
  authority port — is the thing S4 R34 forbids.
- **No call to `authorizationCodeGrant()`, `implicitAuthentication()`,
  `refreshTokenGrant()`, `fetchUserInfo()`, `tokenIntrospection()` or
  `tokenRevocation()`**, and no token exchange performed by the BFF.
- No ordinary double-submit CSRF header required of the authorization-server
  redirect callback.
- No provider token, authorization code, client credential or PKCE verifier
  reaching browser application code (S4 R32).
- No real OIDC tenant or client registration (S4 §16).
- No login page, provider chooser or provider branding (S4 §1.1).
- No durable storage of anything.
- No backend endpoint shape.

## Completion definition

The OIDC protocol shell is proved at **unit and component level**: Authorization
Code with PKCE is constructed correctly; no implicit-flow path exists; a `state`
is generated per authorization request and **validated at the BFF**; a nonce is
generated per authorization request and **bound to category-B transaction
state**; the transaction mechanism satisfies all eight S4 R30 properties with a
test for each; all six negative cases fail closed; only the permitted
`openid-client` APIs appear in the implementation; and **no federated-exchange
contract and no authorization-code handoff shape has been declared**.

**Browser-level verification is not part of this task's completion definition.**
The authorization-redirect test authority is created in T21; the browser-level
OIDC **transaction** journey is proved in T24 and, across instances, in T26.
Those journeys terminate in the expected **fail-closed** result at the unbound
federated boundary — 0B has no federated exchange to complete, so no task claims
successful federated authentication, ID-token validation or an OIDC-authenticated
session. No federated validation of any kind is performed here.
