# 0B-T02 — Static quality gates and failure demonstrations

- **Phase:** 1 — Toolchain and repository skeleton
- **Depends on:** T01
- **Plan:** §7.1, §7.3, §7.5, §18

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

1. `eslint.config.mjs` — ESLint **9** flat config composing: `@eslint/js`
   recommended, `typescript-eslint` **type-aware** configuration wired to
   `tsconfig.json`, `eslint-config-next`, `eslint-plugin-react-hooks`.

   An earlier draft of this task said ESLint 10. Verified at implementation,
   `eslint-config-next@16.3.5` cannot run under ESLint 10 at all — plan §7.1
   records both causes and the corrected pins (`eslint` and `@eslint/js` at
   **9.39.5**; `eslint-config-next` and `typescript-eslint` unchanged). The
   composition itself is exactly as listed above; only the ESLint line moved.
2. Rules that carry S1 R7 and R25: `@typescript-eslint/no-explicit-any` as an
   error; `@typescript-eslint/ban-ts-comment` requiring a description on any
   suppression; `linterOptions.reportUnusedDisableDirectives: 'error'`, so a
   suppression that no longer suppresses anything is itself an error.
3. `scripts/check-eslint-suppressions.mjs`, chained into `pnpm lint`, rejecting
   any `eslint-disable`, `eslint-disable-next-line`, `eslint-disable-line` or
   `eslint-enable` directive that names no rule. A **scoped** suppression — one
   that names the rules it covers and says why — remains available and is what
   S1 R7 asks for; only the rule-less form is rejected.

   An earlier draft of this task assigned this to ESLint's
   `no-restricted-syntax`. **That rule cannot carry it**: it does not visit
   comment nodes, and a rule-less directive suppresses the very rule that would
   report it. Plan §7.5 records the verified behaviour, why the replacement is
   not a second lint tool and why S1 R23 is not weakened.

   The two mechanisms answer different questions and neither replaces the other:

   ```text
   reportUnusedDisableDirectives   -> a suppression that suppresses nothing
   check-eslint-suppressions.mjs   -> a suppression that names no rule
   ```
4. Prettier wired as the formatting gate via `pnpm format:check`.
5. Ensure `pnpm lint`, `pnpm format:check` and `pnpm typecheck` each exit
   non-zero on failure.

## Expected files

```
eslint.config.mjs
scripts/check-eslint-suppressions.mjs
prettier.config.mjs        (adjusted if needed)
package.json               (script wiring; plus the §7.1 ESLint pin correction)
pnpm-lock.yaml             (regenerated for that correction only)
```

## Implementation guidance

Keep the config declarative and readable — it is itself review material, and T04
will extend the same file with boundary and unsafe-DOM rules rather than adding a
second lint mechanism.

Type-aware linting needs a `parserOptions.projectService` (or `project`) entry.
If lint time becomes a problem, scope type-aware rules to `src/**` rather than
disabling them; the boundary gate in T04 depends on them.

Scope the type-aware block to `**/*.{ts,tsx}`. The repository's `.mjs`
configuration files are not in the `tsconfig.json` program, and a type-aware
rule applied to them fails to resolve rather than reporting anything useful.

T04's use of `no-restricted-syntax` for the unsafe-DOM guard is unaffected by
§7.5: that rule matches AST nodes such as `dangerouslySetInnerHTML`, which it
sees perfectly well. Only comment directives are invisible to it.

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

Four separate deliberate violations, each demonstrated failing then removed:

1. a blanket `any` in a `src/**` file → `pnpm lint` fails;
2. a blanket `eslint-disable` over that same `any` → `pnpm lint` fails. Record
   that `eslint .` alone exits **0** on this input: that is the evidence for
   §7.5, and a run showing only the gate's exit code does not demonstrate it;
3. a re-indented file → `pnpm format:check` fails;
4. a type error in `src/**` → `pnpm typecheck` fails.

`src/` does not exist until T03, so tests 1, 2 and 4 create it. **Remove the
directory as well as the file** when restoring — S1 R9 forbids an empty
directory that carries no real content.

## Evidence to leave behind

A recorded transcript of each gate failing and then passing. No violation remains
in the tree.

## Stop conditions

- A framework surface cannot be typed without a blanket suppression → stop and
  surface rather than weakening S1 R7.
- A lint package named by this task turns out not to run, or not to enforce what
  it is relied on to enforce, under the pinned versions → stop, record the
  verified behaviour, and express the policy through the mechanism that **is**
  effective rather than leaving an inert setting in place. Both §7.1 and §7.5
  came from this condition firing. It is a §84.4 stop condition only if no
  available mechanism can hold the requirement.

## What must NOT be implemented

- No module-boundary rules, no server-only rules, no unsafe-DOM rules — all T04.
  `eslint-plugin-boundaries` is installed by T01 and stays unconfigured here.
- No test runner configuration (T10).
- No CI workflow (T31).
- No application source. `src/` exists only inside a negative test.
- No second formatter, and no lint mechanism besides ESLint (S1 R23). The
  single-rule suppression check of scope item 3 is the one exception the plan
  makes, bounded by §7.5: every rule deciding whether code is acceptable stays
  in `eslint.config.mjs`.
- No dependency change beyond the §7.1 ESLint pin correction.

## Completion definition

All three gates pass on a clean tree, each has been demonstrated failing on a
deliberate violation and green restored, the blanket-suppression check has been
demonstrated failing and green restored, and no suppression exists that is not
narrow, local and justified.

Gates run on the Node baseline of plan §7.2 (S5 R18), not on whatever Node
happens to be on `PATH`.

`pnpm build` is **not** part of this task: there is still no `src/app/layout.tsx`
and no `src/app/page.tsx` until T03.
