# 0B-T26 — Multi-instance verification

- **Phase:** 6
- **Depends on:** T22, T18
- **Plan:** §13, §14, §18

## Objective

Prove that two concurrently running frontend instances serve the same browser
session with no sticky routing, and that an OIDC **transaction** initiated on one
completes on the other — with no frontend-owned shared session state.

**What "the transaction completes on the other instance" means at 0B**, precisely:
instance A initiates the authorization transaction; the browser passes through the
synthetic provider; instance **B** receives the callback; B opens the category-B
state using the **same configured sealing key**; B validates `state` and
redirect/callback safety; B clears the transaction; replay then fails; and the
federated authentication boundary **remains unbound and fails closed, as
designed**.

It does **not** mean that B performs a token exchange or creates a federated
authenticated session. No Approved criterion asks it to: 0B-AC-069 and 0B-AC-104
both say "completes an OIDC **transaction**", cited against S4 R25 and R30 — the
category-B transaction state.

## Why this task exists

ADR-027 §6 makes horizontal scalability a decision, not an aspiration, and S5 R34
requires it to be executed against at least two instances. It is also the test
that proves the category-B mechanism chosen in T18 genuinely needs no sticky
routing.

## Approved requirement references

S5 R34; S4 R25, R30; ADR-027 §2, §6.

## Acceptance criteria advanced or satisfied

- **0B-AC-069** — a second instance serves the same session and completes the
  OIDC **transaction** the first initiated (transaction, not federated
  authentication).
- **0B-AC-071** — transaction state remains transaction-scoped across instances.
- **0B-AC-104** — two instances, no sticky routing, no shared frontend-owned
  durable session state or authority.

## Prerequisites

T22 (both instances running), T18 (the transaction mechanism).

## Exact scope

`tests/e2e/multi-instance/`:

1. **Horizontal session-boundary proof.** Establish a session through the
   synthetic **session authority** — clearly test-harness session setup, **not**
   claimed OIDC authentication — then resolve it against instance A on `:3100`
   and instance B on `:3101`; assert identical resolution from either instance.
2. **Cross-instance transaction proof.** Initiate the authorization transaction
   on instance A; deliver the callback to instance B; assert that B unseals the
   category-B state with the same configured key, validates `state`, validates
   the callback target, clears the transaction, and **reaches the intentionally
   unbound federated boundary and fails closed**.
3. Assert no frontend-owned durable session state or session authority is shared
   between the instances — the only shared things are the configured sealing key
   and the out-of-process synthetic authority, which stands in for the backend.
4. Assert transaction state remains transaction-scoped: after the transaction is
   consumed on B, the cookie is cleared and cannot be replayed on either
   instance.

## Expected files

```
tests/e2e/multi-instance/**
```

## Implementation guidance

**No proxy is needed.** Cookies are not port-scoped, so `localhost:3100` and
`localhost:3101` share one cookie jar — navigate directly between ports and the
browser carries the session and transaction cookies to both. This is simpler and
more deterministic than a round-robin proxy, and it makes the absence of sticky
routing explicit rather than inferred.

Both instances must be built from the **verification profile** (T22), so the
cross-instance OIDC transaction exercises the approved category-B mechanism on
both ends rather than a shortcut on one.

What makes this work is that the sealing key is **configuration**, not
frontend-owned state. Assert that explicitly in the test's comments so a later
reader understands why cross-instance completion is not a violation — S5 §14
states a mechanism is **not** implicated merely because short-lived category-B
correlation works across instances.

**Keep the two proofs separate and honestly labelled.** Step 1's session is set
up through the session-authority interface and must be described as test-harness
session setup, never as OIDC authentication. Step 2 ends in a fail-closed
federated-boundary result, which is the correct 0B outcome — assert it positively
rather than treating it as a shortfall.

## Validation

```
pnpm test:e2e --project=chromium-desktop -- tests/e2e/multi-instance
```

## Required negative tests

- Give instance B a **different** sealing key → the cross-instance callback fails
  closed rather than succeeding insecurely. Restore and confirm green.
- Replay a completed transaction cookie on either instance → rejected.

## Evidence to leave behind

Playwright report sections for cross-instance session resolution and
cross-instance OIDC completion, plus the two negative cases.

## Stop conditions

- **If cross-instance completion requires shared frontend runtime state, a durable
  frontend session store, or sticky routing — STOP.** That is category A,
  contradicts ADR-027 §2 and §6, and is a §84.4 stop condition requiring a
  superseding ADR (S4 §14; S5 §14).

## What must NOT be implemented

- **No shared frontend session store, cache or coordination service** of any kind.
- **No token exchange on either instance**, and no claim that B created a
  federated authenticated session.
- No sticky routing, no session affinity, no instance-pinning cookie.
- No proxy that hides which instance served a request — the test must be able to
  tell.
- No product journey across instances.
- No third instance or scaling infrastructure.

## Completion definition

Either instance resolves the same harness-established browser session; the OIDC
**transaction** initiated on A is completed on B — unsealed, `state`-validated,
target-validated, cleared, and terminating in the expected **fail-closed**
federated-boundary result; the mismatched-key and replay negatives both fail
closed; no frontend-owned shared session state exists. **No token exchange and no
federated authenticated session is performed or claimed on either instance.**
