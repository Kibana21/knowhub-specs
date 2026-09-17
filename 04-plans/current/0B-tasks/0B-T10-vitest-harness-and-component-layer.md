# 0B-T10 — Vitest harness, component layer and structural accessibility

- **Phase:** 3
- **Depends on:** T07
- **Plan:** §18, §19

## Objective

Stand up the component and unit verification layer as a blocking gate, and run
the structural accessibility checks that jsdom can decide honestly.

## Why this task exists

S5 R19–R20 require a component and unit layer that executes, blocks, and
demonstrates the obligations S5 §2.1 assigns to it — rather than asserting them.
It is created as soon as there are real surfaces to test.

## Approved requirement references

S5 R1, R2, R3, R19, R20, R21, R22; S2 R20, R21, R23, R27, R30, R31, R33, R36,
R39; S1 R12, R13.

## Acceptance criteria advanced or satisfied

- **0B-AC-006** — the root layout boots (component half).
- **0B-AC-020, 021** — token single source; computed values derive from tokens.
- **0B-AC-023** — every primitive renders each declared state.
- **0B-AC-024** — the shell renders with nothing supplied.
- **0B-AC-029, 030** — state classes; safe error content.
- **0B-AC-031** — the structural half: names, labels, error association,
  landmarks.
- **0B-AC-090** — only layers with real tests are materialized.
- **0B-AC-096** — the component layer executes the S5 §2.1 inherited set.

## Prerequisites

T07. T08 and T09 supply further surfaces and may land in parallel.

## Exact scope

1. `vitest.config.ts` — two projects: `node` (unit, contract) and `jsdom`
   (component, structural accessibility). `restoreMocks`; fake timers where
   behaviour is time-sensitive.
2. `tests/unit/`, `tests/component/`, `tests/accessibility/` — created **only**
   because they hold real tests (S5 R1–R2).
3. Component tests covering: the root layout and single composition point; the
   **token-propagation** test (change a token, assert every consumer's computed
   value moves); every primitive in each declared state; the shell with all slots
   empty; each state-class presentation; both safe-error negative fixtures from
   T08; the unauthorized-navigation fixture from T09.
4. Structural accessibility tests with axe-core over the shell, the primitive
   layer and the state presentations.

## Expected files

```
vitest.config.ts
tests/unit/**  tests/component/**  tests/accessibility/**
```

## Implementation guidance

**Run only the axe rules jsdom can decide.** `color-contrast` and `target-size`
need real layout and are the authoritative pass in T23 under Chromium. Enabling
them here would produce a green result that proves nothing — explicitly disable
them in this layer and note why in the config.

Determinism is a requirement, not a nicety (S5 R3): no dependence on execution
order or ambient wall-clock time; control time with fake timers where the
behaviour under test is time-sensitive.

Use `@testing-library/react` with `user-event` for interaction, and assert on
accessible roles and names rather than implementation details — that is also what
makes these tests evidence for S2 R33.

Create no test-layer directory without real tests in it (S5 R1). `tests/contract`
arrives with T12; `tests/e2e` with T22.

## Validation

```
pnpm test
```

## Required negative tests

- Change a token value → the propagation test detects every consumer moving;
  reverting restores.
- The two T08 error fixtures → unsafe material absent, correlation identifier
  present.
- The T09 unauthorized fixture → item absent, not disabled.

## Evidence to leave behind

A JUnit report; a green `pnpm test` run.

## Stop conditions

- If a criterion can only be made green by enabling a layout-dependent axe rule
  in jsdom, stop: that is fabricated evidence (S5 R22) and belongs in T23.

## What must NOT be implemented

- **No `color-contrast` or `target-size` assertion in jsdom** — T23 owns them.
- No coverage threshold, no coverage package (S5 §14 defers thresholds).
- No empty test-layer directory (S5 R1).
- No browser tests, no Playwright (T22).
- No fabricated metric and no aesthetic judgement expressed as a threshold
  (S5 R22).
- No product fixture data beyond what a 0B surface needs.

## Completion definition

`pnpm test` is green and blocking; the token-propagation test passes; every
primitive, state presentation and the shell is covered; structural accessibility
checks run with layout-dependent rules explicitly deferred to T23; only layers
holding real tests exist.
