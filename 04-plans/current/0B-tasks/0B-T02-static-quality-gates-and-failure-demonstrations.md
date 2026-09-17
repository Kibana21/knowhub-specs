# 0B-T02 — Static quality gates and failure demonstrations

- **Phase:** 1 — Toolchain and repository skeleton
- **Depends on:** T01
- **Plan:** §7.3, §18

## Objective

Make lint, mechanical formatting and strict type checking exist, block, and be
demonstrably capable of failing.

## Why this task exists

S1 R23–R25 require ESLint enforcement, mechanically checkable formatting and
strict type checking; S5 R13–R15 require each to execute as a **blocking** gate
and S5 R17 requires each to be shown failing. A gate that has never failed has
not been shown to work.

## Approved requirement references

S1 R7, R23, R24, R25; S5 R13, R14, R15, R17, R18.

## Acceptance criteria advanced or satisfied

- **0B-AC-004** — strict mode; no blanket `any`; suppressions narrow and justified.
- **0B-AC-011** — a lint violation fails; a formatting deviation is detected
  mechanically.
- **0B-AC-094** — each of the three gates fails on a deliberate violation and
  green is restored by removing it.

## Prerequisites

T01 — the manifest, lockfile and `tsconfig.json` exist.

## Exact scope

1. `eslint.config.mjs` — ESLint 10 flat config composing: `@eslint/js`
   recommended, `typescript-eslint` **type-aware** configuration wired to
   `tsconfig.json`, `eslint-config-next`, `eslint-plugin-react-hooks`.
2. Rules that carry S1 R7 and R25: `@typescript-eslint/no-explicit-any` as an
   error; `@typescript-eslint/ban-ts-comment` requiring a description on any
   suppression; `no-restricted-syntax` forbidding blanket file-level
   `eslint-disable` without a rule list.
3. Prettier wired as the formatting gate via `pnpm format:check`.
4. Ensure `pnpm lint`, `pnpm format:check` and `pnpm typecheck` each exit
   non-zero on failure.

## Expected files

```
eslint.config.mjs
prettier.config.mjs        (adjusted if needed)
package.json               (script wiring only)
```

## Implementation guidance

Keep the config declarative and readable — it is itself review material, and T04
will extend the same file with boundary and unsafe-DOM rules rather than adding a
second lint mechanism.

Type-aware linting needs a `parserOptions.projectService` (or `project`) entry.
If lint time becomes a problem, scope type-aware rules to `src/**` rather than
disabling them; the boundary gate in T04 depends on them.

S1 R25 warns against configuring strictness so as to force semantically empty
annotations on third-party framework surfaces. Where Next supplies no usable
type, prefer a narrow, justified local suppression to widening a rule globally.

## Validation

```
pnpm lint
pnpm format:check
pnpm typecheck
```

## Required negative tests

Three separate deliberate violations, each demonstrated failing then removed:

1. a blanket `any` in a `src/**` file → `pnpm lint` fails;
2. a re-indented file → `pnpm format:check` fails;
3. a type error in `src/**` → `pnpm typecheck` fails.

## Evidence to leave behind

A recorded transcript of each gate failing and then passing. No violation remains
in the tree.

## Stop conditions

- A framework surface cannot be typed without a blanket suppression → stop and
  surface rather than weakening S1 R7.

## What must NOT be implemented

- No module-boundary rules, no server-only rules, no unsafe-DOM rules — all T04.
- No test runner configuration (T10).
- No CI workflow (T31).
- No application source.
- No second formatter and no lint mechanism besides ESLint (S1 R23).

## Completion definition

All three gates pass on a clean tree, each has been demonstrated failing on a
deliberate violation and green restored, and no suppression exists that is not
narrow, local and justified.
