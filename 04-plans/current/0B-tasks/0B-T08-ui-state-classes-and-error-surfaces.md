# 0B-T08 — UI state classes and safe error surfaces

- **Phase:** 3
- **Depends on:** T07
- **Plan:** §11

## Objective

Give every §61A.3 state class a reusable presentation, and make error
presentation safe by construction.

## Why this task exists

S2 R21 requires a reusable presentation for each state class, and S2 R27 requires
that a user-facing error present safe language only — the frontend must not undo
the backend's safe-error model. Building these once, before any consumer exists,
is what stops each future screen reinventing them.

## Approved requirement references

S2 R21, R22, R23, R24, R25, R26, R27, R28, R29, R35, R44 e.

## Acceptance criteria advanced or satisfied

- **0B-AC-029** — every state class has a representative rendered state; a
  known-layout loading case renders a skeleton; an empty state presents one next
  action; async progress derives nothing.
- **0B-AC-030** — no unsafe material reaches the rendered output; a supplied
  correlation identifier is displayed; an important error renders near the failed
  object.
- **0B-AC-032** — no state signalled by colour alone.

## Prerequisites

T07.

## Exact scope

1. `src/components/states/` — loading, skeleton, empty, partial, error,
   forbidden, not-found, retry, disabled-action and asynchronous-progress
   presentations.
2. `src/app/error.tsx`, `src/app/global-error.tsx`, `src/app/not-found.tsx` —
   the App Router error boundary, route-level error and not-found surfaces.
3. Safe error presentation: renders a safe message and, when supplied, a support
   or correlation identifier the user can quote. It renders **no** raw backend
   exception text, stack trace, sensitive implementation or diagnostic detail,
   internal configuration, credential, secret or token.
4. The asynchronous-progress presentation renders **externally supplied** stage,
   progress where measurable, last event and actionable error, and derives none
   of them and models no run.

## Expected files

```
src/components/states/*.tsx
src/app/error.tsx  src/app/global-error.tsx  src/app/not-found.tsx
```

## Implementation guidance

Prefer a skeleton to a generic spinner wherever the resulting layout is known
(S2 R22) — that is most 0B surfaces.

An empty state explains the emptiness and presents **one** obvious next action
where an action exists (S2 R23). Where no action exists at 0B, explain rather
than showing a bare absence.

Toasts carry transient confirmation only; an important error lives **near the
object that failed**, with a remediation path where one exists (S2 R24). Build
the error presentation so proximity is the default, not an option a caller can
forget.

Safety is structural: accept only a narrow, already-safe props shape rather than
an arbitrary error object. The presentation cannot leak what it was never given.
This task defines **no error schema** — T13 owns the normalized representation
(S2 §8; S3 R42).

Freshness, authorization problems and missing evidence are never discoverable
only on hover (S2 R29; R44 e). Where such state is present it is visible in the
rendered layout.

## Validation

```
pnpm lint && pnpm typecheck && pnpm build
```

## Required negative tests

Authored here, executed by the T10 component suite:

- an error carrying backend exception text, a stack trace, a credential and an
  internal configuration value → none appears in the rendered output;
- the same error carrying a correlation identifier → the identifier **is**
  displayed.

## Evidence to leave behind

The state presentations and the negative error fixtures the T10 suite consumes.

## Stop conditions

- If a presentation appears to need the raw error object to be useful, stop:
  S2 R27 forbids rendering unsafe material, and T13 owns producing a safe one.

## What must NOT be implemented

- **No error schema or error shape** — that is T13 / S3 R38–R42.
- No run model; the progress presentation derives nothing (S2 R25).
- No product-specific empty or error content.
- No toast used for an important error.
- No state communicated by colour alone (S2 R35).
- No hover-only critical state.

## Completion definition

Every §61A.3 state class has a reusable presentation; the App Router error,
global-error and not-found surfaces exist; both negative error fixtures are
authored; nothing derives progress, stage or run state.
