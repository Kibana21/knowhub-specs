# 0B-T04 — Module-boundary, server-only and unsafe-DOM enforcement

- **Phase:** 2 — Architecture boundaries and configuration
- **Depends on:** T02, T03
- **Plan:** §9, §18, §21

## Objective

Wire the §61A.2 layering rule, the server-only placement rule, the
test-only containment rule and the unsafe-DOM guard as **blocking** gates, each
demonstrably able to fail.

## Why this task exists

S1 R21–R22 require that a layering or server-only violation fail CI
automatically, and that the mechanism be wired and gating **before** most of the
layers it governs exist — so later milestones inherit a working gate rather than
build one. S5 R12 and R50 add the containment and unsafe-DOM gates.

## Approved requirement references

S1 R8, R9, R10, R11, R15, R17, R18, R19, R20, R21, R22; S5 R9, R10, R11, R12,
R16, R17, R50, R51.

## Acceptance criteria advanced or satisfied

- **0B-AC-005** — the materialized directory set matches S1 R10.
- **0B-AC-007** — the server/client policy is applied; no auth module is
  reachable from a client bundle.
- **0B-AC-008** — no product route (structural half).
- **0B-AC-009** — a deliberate layering or placement violation fails the gate.
- **0B-AC-010** — no frontend source implements backend domain semantics.
- **0B-AC-025** — no fabricated data reachable from production code.
- **0B-AC-043** — the contract fixture is unreachable from production code.
- **0B-AC-092** — test-only material is structurally unreachable.
- **0B-AC-095** — the boundary gate fails on both rule kinds.
- **0B-AC-109** — no production path reaches a raw-HTML or unsafe DOM boundary.

## Prerequisites

T02 (lint config), T03 (`src/app/` exists).

## Exact scope

1. Extend `eslint.config.mjs` with `eslint-plugin-boundaries`: element types
   `app`, `components`, `lib-api`, `lib-auth`, `lib-config`, `lib-query`,
   `lib-utils`, `tests`, and the permitted edge set in plan §9 — nothing else.
2. A rule forbidding **any** import from `src/**` to `tests/**`.
3. A rule forbidding client components importing `src/lib/auth/**` or
   `src/lib/config/server*`.
4. Add the `server-only` package and import it at the top of every server-only
   module created from T05 onward, so a client bundle that reaches one **fails
   the build**. Two mechanisms, because a placement rule needs both a fast lint
   signal and a hard build failure.
5. Unsafe-DOM rules: `no-restricted-syntax` / `no-restricted-properties` banning
   `dangerouslySetInnerHTML`, `innerHTML`, `outerHTML`, `insertAdjacentHTML`,
   `document.write`, `eval` and `new Function` in `src/**`.
6. A directory-inventory assertion — a small script or test comparing the
   materialized directory set against S1 R10's table, failing on any extra
   directory and on the presence of `src/lib/telemetry/`.

## Expected files

```
eslint.config.mjs           (extended)
package.json                (server-only dependency)
scripts/ or tests/unit/     directory-inventory assertion
```

## Implementation guidance

Keep all of this in the single ESLint config rather than adding a second
boundary tool: lint is already a required blocking gate, so nothing new has to be
wired into CI, and one config file stays reviewable.

The boundary graph is small now and will be extended by later tasks as they
create `lib-api`, `lib-auth`, `lib-config` and `lib-query`. Declare the element
types up front so no later task has to redesign the graph.

S1 R22 acknowledges that coverage is necessarily partial at 0B because only part
of the target tree exists. That is expected — the requirement is that the
mechanism be wired, gating, and demonstrably able to fail.

`no-restricted-syntax` covers the unsafe-DOM guard with **no extra plugin**;
resist adding one for a single rule.

## Validation

```
pnpm lint
pnpm build
node <directory-inventory assertion>
```

## Required negative tests

Each demonstrated failing and then removed:

1. an import from `src/components/**` into `src/lib/auth/**` → lint fails;
2. an upward import violating the layer graph → lint fails;
3. a client component importing a `server-only` module → **build** fails;
4. an import from `src/**` to `tests/**` → lint fails;
5. a `dangerouslySetInnerHTML` in `src/**` → lint fails;
6. an extra directory under `src/lib/` → the inventory assertion fails.

## Evidence to leave behind

A recorded transcript of all six failures and the restored green run.

## Stop conditions

- If the server-only rule cannot be enforced without a runtime check, stop:
  S1 R15 is a **placement** rule and must be enforced structurally.

## What must NOT be implemented

- No configuration boundary content (T05), no API boundary (T13), no auth modules
  (T15+). This task wires the rules; later tasks populate the layers.
- No directory created merely to give a rule something to point at — S1 R9
  forbids an empty directory mirroring the target tree.
- No relaxation of a rule to accommodate code that does not yet exist.

## Completion definition

All six negative cases have been demonstrated failing and green restored; the
directory inventory matches S1 R10; `src/lib/telemetry/` does not exist.
