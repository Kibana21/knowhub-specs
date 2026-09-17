# 0B-T25 — The security-negative layer

- **Phase:** 6
- **Depends on:** T22, T17
- **Plan:** §15, §18

## Objective

Prove that request integrity holds, that tampering grants nothing, and that deny
behaviour is deterministic.

## Why this task exists

S5 R33 and R35 require executed negative cases, not asserted ones. §0E.16 makes
negative authorization testing a required control area, and 0B-AC-084's
cross-origin case is the evidence that fully discharges AT-029.

## Approved requirement references

S5 R33, R35; S4 R51, R52, R54–R59, R60–R65; ADR-022, ADR-023.

## Acceptance criteria advanced or satisfied

- **0B-AC-074** — an out-of-allowlist redirect or callback target is rejected.
- **0B-AC-082** — URL, state, storage and JavaScript edits grant no capability;
  claims are not authority.
- **0B-AC-083** — one guard at the server boundary; deterministic deny.
- **0B-AC-084 / 0B-AC-103** — cross-origin state-changing requests rejected,
  same-origin succeed, Origin and Referer validated, LOCAL and OIDC **initiation**
  protected identically; the OIDC callback bound by regime 2.
- **0B-AC-105** — tampering grants nothing; no product protected route exists.

## Prerequisites

T22 (the harness and the attacker origin), T17 (the controls under test).

## Exact scope

`tests/e2e/security/`:

1. **CSRF matrix** from the attacker origin at `http://localhost:3900`: a
   state-changing request with no token → rejected; with a mismatched token →
   rejected; with a valid token but a foreign `Origin` → rejected; a legitimate
   same-origin request with valid controls → succeeds. Run identically for the
   LOCAL and OIDC legs.
2. **Redirect allowlist** in the browser: `//evil.example`, `\\evil.example`,
   `https://evil.example` and percent-encoded variants as post-sign-in,
   post-logout and callback targets → each rejected.
3. **Tampering**: edit the route URL; edit client state; write to browser
   storage; mutate JavaScript state → **none** grants capability.
4. **Claims**: a browser- or provider-supplied role or permission value is not
   treated as authority; no permission is inferred from a display role label.
5. **Determinism**: the same state produces the same outcome across repeated
   runs; a loading, pending or unknown state grants no access.
6. **No product protected route** exists — asserted by inventory.

## Expected files

```
tests/e2e/security/**
```

## Implementation guidance

**The attacker origin must be same-host, different-port.** `127.0.0.1` and
`localhost` are different hosts, so a page there sends no cookies and the CSRF
test would pass for the wrong reason — a false green on the single most important
negative test in 0B. `http://localhost:3900` sends cookies while presenting a
different `Origin`, which isolates the Origin and token controls precisely.

Run the regime-1 matrix against **both** the LOCAL and the OIDC **initiation**
legs. S4 R64 requires equivalent protection, and running it once proves only one
path.

**Do not assert the regime-1 mechanism of the OIDC callback.** An
authorization-server redirect cannot set `X-KnowHub-CSRF` and does not present our
origin; its binding is PKCE + `state` + sealed transaction + callback-target
validation (plan §15, regime 2). Asserting a control the request cannot carry
would either fail permanently or be satisfied by weakening the flow — neither is
evidence.

For determinism, run the deny path repeatedly and assert an identical outcome —
S4 R57 makes this a property, not an incidental observation.

## Validation

```
pnpm test:e2e --project=security
```

## Required negative tests

The whole task is negative tests. Additionally, prove the harness itself works:

- temporarily disable the Origin check → the cross-origin case **passes**, showing
  the test would have caught a real regression; restore it and confirm the case
  fails again.

## Evidence to leave behind

The Playwright `security` project report covering every case above.

## Stop conditions

- If any tampering path grants capability, stop immediately: that is an
  authorization defect, and S4 R51 admits no exception.
- If the cross-origin case passes without cookies being sent, the attacker origin
  is misconfigured — fix it rather than accepting the green.

## What must NOT be implemented

- **No product protected route created to have something to test** (S4 R59).
- No weakening of a control to make a case reproducible.
- No real credentials or real provider.
- No CORS policy (S5 R47).
- No multi-instance cases (T26), no accessibility cases (T23).

## Completion definition

Every regime-1 CSRF case, regime-2 callback-binding case, redirect, tampering,
claims and determinism case passes; the regime-1 matrix runs identically for the
LOCAL and OIDC initiation legs; the harness has been shown capable of catching a
disabled control; no product protected route exists.
