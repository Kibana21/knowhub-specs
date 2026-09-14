# ADR-027 — Frontend BFF session state ownership

- **Status:** Accepted
- **Scope:** product
- **Date:** 2026-09-14
- **Origin:** this ADR
- **Supersedes:** —
- **Superseded by:** —

## Context

[ADR-021](ADR-021-provider-pluggable-authentication.md) establishes that
KnowHub authentication is provider-pluggable, that LOCAL and OIDC both
terminate in one internal KnowHub session/access-token model behind a minimal
frontend BFF, and that browser JavaScript never persists access or refresh
tokens.

Blueprint [§0E.13.3](../../knowhub_master_blueprint.html#runtime-architecture)
shows a "Next.js server-side session" holding a short-lived internal access
token, and [§0E.13.7](../../knowhub_master_blueprint.html#runtime-architecture)
requires that refresh and session material stay server-side while the browser
receives only an opaque HttpOnly cookie.

Neither section says **where the server side of that BFF keeps session
material**, and the three viable answers have materially different
consequences for infrastructure, scaling and revocation:

- **A — sealed, stateless, browser-carried session material.** The encrypted
  session travels in the cookie; the BFF holds nothing.
- **B — a frontend-owned shared session store**, such as a Redis or database
  instance belonging to `knowhub-frontend`.
- **C — backend-authoritative durable session with an opaque browser
  reference**, the frontend BFF holding no durable session state.

The question cannot be left to an implementation plan. It determines whether
`knowhub-frontend` is stateless or stateful, whether it acquires its own
durable authentication datastore, whether horizontal frontend scaling is safe,
and where session revocation is authoritative. It binds both repositories and
spans milestone 0B and milestone 1, so it is resolved before the frontend
session boundary is specified and before backend identity is implemented.

This ADR refines the ownership question left open by ADR-021. It reverses
nothing.

## Decision

### 1. The backend owns durable, authoritative authenticated-session state

The KnowHub backend is authoritative for:

- canonical user identity;
- durable authenticated session state;
- the internal access-token lifecycle;
- the refresh/session lifecycle;
- session revocation, including administrative revocation;
- session invalidation;
- effective authorization state.

### 2. The frontend BFF owns no durable session datastore

`knowhub-frontend` does not acquire a Redis instance, a database or any other
durable store for session state. It persists no access token, no refresh
token, no external identity-provider token and no durable session state — in
browser-visible storage or in frontend-owned infrastructure.

### 3. The browser holds only an opaque session reference

The browser receives only an opaque, high-entropy, HttpOnly, Secure,
SameSite-governed session cookie. It encodes no permissions, no identity
attributes and no token material.

### 4. The frontend BFF is a boundary, not an authority

The frontend BFF is responsible for:

- the browser-safe session boundary;
- opaque cookie handling;
- same-origin session mediation;
- server-only forwarding of authenticated requests;
- provider-neutral session presentation;
- route and session shell behaviour.

It makes no entitlement decision and holds no authoritative session fact.

### 5. LOCAL and OIDC converge on one canonical model

LOCAL authentication and OIDC authentication ultimately terminate into the
same canonical KnowHub backend identity and session model. Authentication
provider choice does not shape downstream frontend authorization semantics.

### 6. The frontend BFF remains horizontally scalable

Any number of `knowhub-frontend` instances serve the same browser session
without shared frontend-owned session storage.

### 7. Revocation is backend-authoritative; its convergence is deferred

Session revocation is decided and enforced by the backend. **The mechanism by
which the frontend boundary converges on a revocation — and the latency that
mechanism guarantees — is deferred to the M1 identity and session design.**

### 8. What this ADR does not decide

This ADR fixes ownership and architectural boundaries only. It deliberately
specifies none of the following, all of which belong to the M1
identity/session specifications and plans: backend session table or schema
design, token schema, token lifetimes, refresh-rotation algorithm, revocation
polling or convergence intervals, backend endpoint shapes, Redis usage,
PostgreSQL table design, OIDC exchange details, and LOCAL credential
verification.

## Alternatives considered

### A — sealed, stateless, browser-carried session material — rejected

- Conflicts with the requirement for authoritative revocation: a sealed
  cookie remains valid until it expires, so a revoked session can continue to
  be presented.
- Moves sensitive authenticated material into the browser cookie, weakening
  the separation ADR-021 establishes.
- Complicates immediate administrative revoke, which
  [§0E.13.9](../../knowhub_master_blueprint.html#runtime-architecture)
  requires as a first-class capability.

### B — a frontend-owned shared session store, such as Redis — rejected

- Gives `knowhub-frontend` its own durable authentication datastore.
- Creates additional frontend infrastructure to provision, secure and operate.
- Couples horizontal frontend scaling to frontend-owned shared state.
- Duplicates session ownership that already belongs to the canonical KnowHub
  identity/session model.
- Conflicts with the standing preference against introducing infrastructure
  without measured need
  ([§0E.9](../../knowhub_master_blueprint.html#runtime-architecture);
  [§84.1](../../knowhub_master_blueprint.html#coding-harness-instructions)).

### C — backend-authoritative durable session with an opaque browser reference — chosen

The frontend BFF is stateless with respect to durable session storage.

## Consequences

**Positive**

- One authoritative session model, in one place.
- LOCAL and OIDC converge cleanly, with no provider-shaped divergence
  downstream.
- Revocation stays centralized and authoritative.
- `knowhub-frontend` scales horizontally with no shared frontend state.
- No frontend session datastore exists to provision, secure or operate.
- Frontend authorization presentation remains provider-neutral.
- Fewer security boundaries hold sensitive material.

**Trade-offs**

- Authenticated BFF requests depend on backend session authority.
- Backend identity and session availability becomes important to the
  authenticated UI, not only to data access.
- M1 must define the revocation, token and session lifecycle precisely; this
  ADR deliberately leaves that undone.

## Relationship to existing ADRs

This ADR supersedes nothing.

- [ADR-016](ADR-016-two-independently-versioned-repositories.md) — the
  repositories stay independently versioned and deployable. Declining a
  frontend-owned session store keeps `knowhub-frontend` deployable without
  frontend-specific stateful infrastructure.
- [ADR-021](ADR-021-provider-pluggable-authentication.md) — refined, not
  reversed. ADR-021 places a minimal BFF between browser and backend and bars
  browser token persistence; ADR-027 answers where the server side of that
  boundary keeps state.
- [ADR-022](ADR-022-hybrid-rbac-abac-authorization-default-deny.md) —
  authorization remains backend-enforced and default-deny. A boundary that
  holds no authoritative session fact cannot become a second authorization
  point.
- [ADR-025](ADR-025-canonical-user-model-independent-of-provider.md) — the
  canonical user model stays independent of provider; §5 of this decision is
  the session-state expression of that independence.
- [ADR-026](ADR-026-adr-ownership-and-location.md) — this decision binds both
  repositories, so under ADR-026's scope test it is product-wide and its
  canonical file lives in `knowhub-specs/02-adrs/`.

---

Registered in [the ADR register](README.md).
