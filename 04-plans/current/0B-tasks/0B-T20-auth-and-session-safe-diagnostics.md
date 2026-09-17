# 0B-T20 — Auth and session safe diagnostics

- **Phase:** 5
- **Depends on:** T16, T13
- **Plan:** §13

## Objective

Normalize every provider, token, session and authorization failure into a safe
outcome, and enforce the §0E.13.9 redaction list across frontend diagnostics.

## Why this task exists

S4 R67 binds the §0E.13.9 redaction list to frontend diagnostics, and S4 R68
requires callback and provider error material to be **normalized, not passed
through**. Raw provider errors are a classic leakage path, and the guard is worth
building once rather than per call site.

## Approved requirement references

S4 R20, R66, R67, R68, R69, R70; S3 R38–R42; S2 R27.

## Acceptance criteria advanced or satisfied

- **0B-AC-085** — nothing on the redaction list reaches a log, diagnostic,
  telemetry field or browser surface; failures surface only as safe outcomes;
  outcomes map into the T13 representation; no browser telemetry introduced.

## Prerequisites

T16 (the failure paths), T13 (the normalized representation to map into).

## Exact scope

1. `src/lib/auth/diagnostics.ts` — a redaction helper covering the §0E.13.9 list:
   session identifier, raw cookie, access token, refresh token, external
   identity-provider token, password, reset token, OIDC authorization code, MFA
   secret — plus claims payload, stack trace, internal provider response and
   client secret.
2. Normalization of provider, token, session and authorization failures into the
   **T13 `ApiError`** — no second error shape is defined here.
3. Callback and provider error material normalized rather than passed through.
4. Component tests asserting non-leakage on every path.

## Expected files

```
src/lib/auth/diagnostics.ts
tests/component/auth/**
```

## Implementation guidance

Redaction is structural, not textual: build safe outcomes from a narrow allowlist
of fields rather than scrubbing a rich object. A scrubber eventually misses a
field; a constructor cannot emit one it never accepts.

Authentication failures return **generic** messages (§0E.14.9): a failure must not
reveal whether a user exists, which provider rejected it, or why.

**Map into T13's representation; define no second error shape and no
presentation** (S4 R70). T08 renders; T13 produces the shape; this task produces
safe *auth* outcomes that flow into it.

## Validation

```
pnpm test && pnpm typecheck && pnpm lint
```

## Required negative tests

For each of: a provider error body containing a client secret; a callback error
containing an authorization code; a session failure containing a raw cookie; an
exception carrying a stack trace and a claims payload —

- assert the value appears in **no** log, diagnostic field, returned outcome or
  rendered surface.

## Evidence to leave behind

Component tests for each leakage path. The browser-level sweep is T24.

## Stop conditions

- If a diagnostic is only useful with sensitive content included, stop: S4 R67
  admits no exception, and §0E.13.9 is a normative security contract.

## What must NOT be implemented

- **No second error shape and no error presentation** (S4 R70).
- **No browser telemetry system** — no tracing SDK, vendor integration or metrics
  framework (S4 R70; S3 R37).
- No `src/lib/telemetry/` (S1 R9).
- No pass-through of provider error material (S4 R68).
- No user-enumerating or provider-revealing error message.
- No logging of any redaction-list value, at any level, in any profile.

## Completion definition

Every provider, token, session and authorization failure normalizes into the T13
representation as a safe outcome; no redaction-list value reaches any log,
diagnostic, telemetry field or browser surface; no second error shape and no
telemetry system exists.
