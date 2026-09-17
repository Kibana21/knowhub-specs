# 0B-T21 — Synthetic authority and local OIDC provider

- **Phase:** 5
- **Depends on:** T16, T18
- **Plan:** §13, §18

## Objective

Build the test-only, out-of-process synthetic session authority and local OIDC
provider that let the browser layer exercise the real authentication boundary
without a backend, a real provider, or any frontend-owned state.

## Why this task exists

S4 R71 requires the double to **exercise the canonical production interfaces and
state machine rather than bypassing them**, and S4 R75 requires all six states
with no backend checkout. Being out of process is what lets two frontend
instances resolve the same session in T26 without frontend-owned shared state.

## Approved requirement references

S4 R71, R72, R73, R74, R75; S5 R4, R5, R9, R10, R11, R30.

## Acceptance criteria advanced or satisfied

- **0B-AC-086** — the containment contract: exercises production interfaces; test
  profile only; unreachable from the production bundle and runtime; no product
  contract, credential store or user/role/permission model; no authorization
  authority; six states; no backend needed.

## Prerequisites

T16 (the port), T18 (the OIDC leg it must serve).

## Exact scope

1. `tests/harness/synthetic-authority/` — a small Node HTTP service implementing
   session `establish`, `resolve`, `rotate`, `terminate`, and able to report all
   six S4 R39 states on demand.
2. A minimal local OIDC provider in the same harness — **an authorization-redirect
   test authority**: discovery document and authorize endpoint, sufficient to
   drive the redirect/callback round trip. It exists to exercise the protocol
   shell, **not** to evidence that a production federated exchange exists.
3. `tests/harness/authority-binding.ts` — the verification-profile substitute for
   `src/lib/auth/authority-binding.ts`, implementing `SessionAuthorityPort`
   against the harness service.
4. A `KNOWHUB_TEST_ONLY_SENTINEL` constant exported by every harness module, so
   T28's artifact inspection has an unambiguous marker to search for.
5. Build configuration expressing the verification-profile alias.

## Expected files

```
tests/harness/synthetic-authority/**
tests/harness/authority-binding.ts
<build config>            (verification-profile alias only)
```

## Implementation guidance

**The authority is out of process on purpose.** An in-process double would work
for single-instance tests and fail T26 — and "make it shared" would be exactly the
frontend-owned session state ADR-027 §2 forbids. Running it as a separate service
puts it where the backend will be at M1, which is also why it is not frontend
infrastructure.

It **exercises the production interfaces**, it does not bypass them (S4 R71):
the frontend's own `oidc.ts`, `transaction-state.ts`, `csrf.ts`, `guard.ts` and
cookie handling all run unchanged. Exercising the double proves the boundary, not
a parallel path.

**It possesses no authorization authority** (S4 R74). It cannot grant real product
capability; the permissions port stays unbound regardless of what the authority
reports.

**It defines no product authentication API contract.** Its protocol is a harness
detail, not a contract — nothing in `src/` knows its shape, which is precisely why
the seam is the only connection.

**The harness does not invent a federated exchange.** There is no production
interface through which the callback hands an authorization response to a
federated authority — S4 R34 forbids declaring one and the previous correction
removed it — so the harness must not manufacture a private one in order to make a
login appear to succeed. A verification-profile implementation cannot perform an
exchange the production interface does not expose while also claiming to exercise
that unchanged interface.

**What the local OIDC provider is for.** It drives the authorization
redirect/callback round trip so the protocol shell can be exercised in a real
browser: authorization request → provider → callback → `state` validation →
category-B unseal → redirect-target validation → transaction clearing. The
frontend callback then reaches the **approved unbound federated boundary and
fails closed**. That is the correct and complete 0B result, not a shortfall to be
engineered around.

**What the session authority is for.** Session establishment, resolution,
rotation, revocation and the six canonical states are exercised through the
`SessionAuthorityPort` — an interface that does exist — independently of the OIDC
leg. Session evidence therefore never depends on a federated exchange.

Give every harness module the sentinel constant. T28 depends on having something
unambiguous to assert the absence of.

## Validation

```
node tests/harness/synthetic-authority/index.mjs   # starts and serves
pnpm build                                          # production build unaffected
```

## Required negative tests

- Build the **production** profile and confirm the harness module is absent from
  the output (asserted fully in T28).
- Attempt to import a harness module from `src/**` → lint fails (T04 rule).

## Evidence to leave behind

The harness; the sentinel constant; a production build demonstrably free of both.

## Stop conditions

- **If the harness requires the frontend to hold shared runtime state to work,
  STOP** — that is category A and an ADR-027 §84.4 stop condition.
- If the double can only work by bypassing a production interface, stop: S4 R71
  requires it to exercise them.

## What must NOT be implemented

- **No product authentication API contract, credential store or user/role/
  permission model** (S4 R74).
- **No federated-exchange contract, and no harness shape promoted into `src/`**
  or presented as what M1 will implement.
- **No private exchange mechanism invented to make a federated login succeed**,
  and no claim of successful federated token exchange, ID-token validation or
  OIDC-authenticated session creation.
- **No authorization authority** — it cannot grant real capability.
- **Nothing under `src/`** — every file this task creates lives under `tests/`.
- **No runtime profile selector, environment flag, dynamic import or hidden route
  in production source** (S5 R10 c).
- No real credentials, no real OIDC tenant, no committed enabled bypass
  (S4 R38; §76).
- No user-visible product authentication surface (S5 R30).

## Completion definition

The synthetic authority runs out of process and serves all six canonical session
states through the `SessionAuthorityPort`; the local OIDC provider drives the
authorization redirect/callback round trip; both are wired to the unmodified
production boundary through the build-time seam; every file lives under `tests/`;
the production build contains none of it.

**No federated exchange mechanism has been invented**, and nothing here claims
successful federated authentication. T24 and T26 consume this harness for
transaction-shell and session-boundary evidence respectively.
