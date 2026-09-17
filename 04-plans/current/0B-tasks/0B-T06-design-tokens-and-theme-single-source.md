# 0B-T06 — Design tokens and the theme single source

- **Phase:** 3 — Design tokens, primitives, shell, UI states
- **Depends on:** T03, T05
- **Plan:** §11

## Objective

Establish one design-token system as the sole authoritative source of visual
values, covering the S2 R2 areas within the §0E.17.1 direction.

## Why this task exists

S2 R8 requires primitives to be built **before** page composition, and every
primitive consumes tokens. S2 R1 requires exactly one authoritative definition
per token, which 0B-AC-020 tests by changing a token and asserting every consumer
moves. Tokens must therefore exist before anything renders.

## Approved requirement references

S2 R1, R2, R3, R4, R5, R6, R7; ADR-024.

## Acceptance criteria advanced or satisfied

- **0B-AC-020** — one authoritative definition; a change propagates to every
  consumer.
- **0B-AC-021** — coverage of the R2 areas within the R3 direction; no extra
  palette; generic status semantics only.
- **0B-AC-022** — no dark theme, no switcher, no contained dark surface.
- **0B-AC-032** — status treatments distinguishable without colour (token half).

## Prerequisites

T03 (`src/styles/globals.css` exists), T05.

## Exact scope

One `@theme` block in `src/styles/globals.css` covering canvas/background,
primary surface, primary text, secondary text, accent, border, semantic status,
typography scale, spacing scale, radius, elevation, icon sizing and a
tabular-numeral treatment for metrics.

Resolved values within the §0E.17.1 direction, which S2 R3 states as
approximations and permits the plan to resolve:

| Token area | Value |
|---|---|
| canvas | `#F7F9FC` |
| surface | `#FFFFFF` |
| text primary | `#0F172A` |
| text secondary | `#475569` |
| accent | `#2563EB` |
| accent secondary (restrained teal) | `#0D9488` |
| border | `#E5EAF1` |
| radius | 8 / 10 / 12px |
| type scale | 12 / 14 / 16 / 20 / 24 / 32px |
| elevation | drawers and modal surfaces only |

Semantic status tokens: **neutral, info, success, warning, error** — generic
presentation semantics only, at accessible contrast.

## Expected files

```
src/styles/globals.css
src/lib/utils.ts            (`cn` helper only)
```

## Implementation guidance

Tailwind v4's `@theme` emits each token as a CSS custom property **and** as a
utility, so a primitive using `bg-surface` and a rule reading
`var(--color-surface)` share one definition. That is what makes 0B-AC-020
demonstrable rather than aspirational — do not introduce a second token mechanism
alongside it.

Typography uses `GeistSans` from the `geist` package's `geist/font/sans` export,
which wraps `next/font/local` over bundled `woff2` files — self-hosted, so no
build-time font fetch and no `font-src` exception in T27. `next/font/google` is
not used.

Where §0E.17.1 does not supply an accessible value for a status colour, define
one (S2 R3 permits this). It stays generic presentation semantics and creates no
business-state taxonomy.

## Validation

```
pnpm build && pnpm lint && pnpm typecheck
```

## Required negative tests

- Introduce a literal colour in a component instead of a token → caught by the
  0B-AC-021 review and by the token-propagation test once T07 exists.

## Evidence to leave behind

The `@theme` block; the token-propagation test lands in T10 and asserts this
task's property.

## Stop conditions

- If a token cannot be expressed with exactly one authoritative definition, stop:
  S2 R1 admits no second source.

## What must NOT be implemented

- **No dark theme, no dark-mode-first styling, no user-facing theme switcher, no
  alternate full-application theme** (S2 R6; §0E.17.8).
- **No contained dark code/evidence surface** — S2 R7 permits one only when real
  0B content requires it, and none does.
- No additional brand or accent palette beyond the one accent and the restrained
  teal secondary.
- No run, source, claim, freshness or governance status taxonomy — status tokens
  are generic presentation semantics only (S2 R5).
- No gradients or multi-hue decorative effects used to create hierarchy
  (S2 R44 d).
- No components — T07.

## Completion definition

Every R2 area has exactly one authoritative definition within the R3 direction;
no dark theme, switcher or dark surface exists; no additional palette and no
business-state taxonomy has been introduced.
