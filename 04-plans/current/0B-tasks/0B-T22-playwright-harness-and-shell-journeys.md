# 0B-T22 — Playwright harness and shell journeys

- **Phase:** 6 — Browser, accessibility, security-negative, multi-instance
- **Depends on:** T09, T21
- **Plan:** §18

## Objective

Stand up the Playwright harness on Chromium — including the verification build,
the two frontend instances, the synthetic authority and the attacker origin — and
prove the shell's browser-level behaviour.

## Why this task exists

S5 R28 requires a browser layer that executes and blocks, and several S2 criteria
(0B-AC-024, 028, 033, 034, 035) can only be decided in a real browser. This task
builds the harness every other phase-6 task consumes.

## Approved requirement references

S5 R2, R3, R5, R28, R29, R30; S2 R15, R31, R32, R33, R38, R40, R41, R42, R43.

## Acceptance criteria advanced or satisfied

- **0B-AC-006** — the application boots and serves the root layout.
- **0B-AC-024** — the shell renders with nothing supplied; width behaviour.
- **0B-AC-026** — the command affordance is reachable, enabled, and states its
  unavailability.
- **0B-AC-027** — keyboard operability, visible focus, focus order.
- **0B-AC-033** — transition duration; `prefers-reduced-motion`.
- **0B-AC-034** — modal contains and restores focus; a non-modal drawer does not
  trap.
- **0B-AC-035** — tablet collapse.
- **0B-AC-098** — the layer executes and blocks on Chromium.

## Prerequisites

T09 (the shell), T21 (the harness authority and the verification alias).

## Exact scope

1. `playwright.config.ts` — projects `chromium-desktop`, `chromium-tablet`
   (834×1112) and `security` (used by T25). `retries: 0`. Reporters: HTML plus
   JUnit.
2. A `webServer` set starting: the synthetic authority; frontend instance A on
   `:3100`; frontend instance B on `:3101`; the attacker origin on `:3900`
   (`tests/harness/attacker-origin/`). Both frontend instances are built from the
   **verification profile**.
3. `tests/e2e/shell/` — journeys for boot, empty-slot rendering, workspace width,
   the command affordance, keyboard operability and focus order, modal and
   non-modal focus behaviour, reduced motion, and the tablet collapse.
4. `tests/e2e/shell/cookie-baseline/` — the **cookie-behaviour baseline** against
   the pinned Chromium build: asserts that a `Secure`, `HttpOnly`, `__Host-`
   prefixed cookie set over `http://localhost` is stored and returned. This
   establishes by execution what the plan deliberately does not assert as fact.
   If it fails, switch the harness to local TLS — never relax a cookie attribute.

## Expected files

```
playwright.config.ts
tests/harness/attacker-origin/**
tests/e2e/shell/**                 (including cookie-baseline/)
```

## Implementation guidance

**`retries: 0`.** S5 R3 requires deterministic execution; retries mask exactly the
non-determinism it forbids. Fix a flaky test rather than retrying it.

**Both instances use the verification profile** — T26 depends on this, because a
cross-instance OIDC transaction must exercise the approved category-B mechanism
on both ends.

**Run against `http://localhost`, and verify the cookie behaviour rather than
assuming it.** `Secure`, `HttpOnly`, the SameSite policy and the `__Host-` prefix
are never relaxed in any profile. Whether the **pinned Chromium build** accepts
`Secure` and `__Host-` cookies over HTTP on localhost is established by executing
the check against that build here — it is not assumed. **If those cookies are
rejected, switch the harness to local TLS.** Removing `Secure`, dropping the
prefix or creating an insecure profile is never an acceptable workaround.

**The attacker origin is a port, not a host.** `127.0.0.1` and `localhost` are
different *hosts*, so a page there carries no cookies and T25's CSRF test would
pass for the wrong reason. `http://localhost:3900` is the same host — cookies are
sent — while `Origin` differs, because origins include the port. Build it here so
T25 can rely on it.

S5 R5 is explicit that loopback traffic to locally started instances and locally
hosted synthetic authorities does **not** violate the independence requirement.
No general offline requirement applies to this layer.

Use emulation for `prefers-reduced-motion`, and assert that motion is **reduced or
removed**, not merely shortened (S2 R38; 0B-AC-033).

## Validation

```
pnpm exec playwright install --with-deps chromium
# establish the pinned build's cookie behaviour before relying on it:
pnpm test:e2e --project=chromium-desktop -- tests/e2e/shell/cookie-baseline
pnpm test:e2e --project=chromium-desktop
pnpm test:e2e --project=chromium-tablet
```

## Required negative tests

- With `prefers-reduced-motion` set, a transition that merely shortens rather than
  reduces → the assertion fails.
- A non-modal drawer that traps focus → the assertion fails.
- The cookie baseline is itself a determination, not an assumption: whichever way
  it resolves, record the result and, if cookies are rejected over HTTP
  localhost, re-run the suite under local TLS and confirm green.

## Evidence to leave behind

`playwright-report/` and a JUnit XML; both published as CI evidence in T31.

## Stop conditions

- If the pinned Chromium build rejects `Secure` or `__Host-` cookies on
  `http://localhost`, switch the harness to a local TLS certificate. **Never**
  relax a cookie flag, drop the `__Host-` prefix or introduce an insecure profile
  to make a test pass.
- If a test is only green with retries, stop and fix the non-determinism (S5 R3).

## What must NOT be implemented

- **No Firefox, WebKit or mobile-browser project** — not required (S5 R29).
- No retries.
- No product journey — no Ask, evidence, source onboarding, traceability, admin
  RBAC or application navigation (S5 §14).
- **No user-visible product authentication surface** created in order to test
  (S5 R30) — auth journeys are T24 and drive the boundary through the double.
- No accessibility axe pass (T23), no auth journeys (T24), no security-negative
  cases (T25), no multi-instance cases (T26).
- No test-only route added to `src/app/`.

## Completion definition

The harness starts all four services; the cookie-behaviour baseline has been
**executed against the pinned Chromium build** and its result recorded, with the
harness on local TLS if `__Host-`/`Secure` cookies are rejected over HTTP
localhost and no cookie attribute relaxed either way; Chromium desktop and tablet
projects are green and blocking; every listed shell journey passes; no retries
are configured and no additional browser engine is enabled.
