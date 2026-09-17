# 0B-T03 — App Router root layout and provider composition

- **Phase:** 1 — Toolchain and repository skeleton
- **Depends on:** T01
- **Plan:** §8, §9

## Objective

Create exactly one root layout and exactly one provider-composition point, and
prove the application boots and serves that layout.

## Why this task exists

S1 R12–R13 fix the App Router with one root layout and one composition point;
S1 R16 requires route organisation that accommodates future product modules
without restructuring. Every later surface hangs off these two files, so they are
created before anything composes them.

## Approved requirement references

S1 R12, R13, R14, R16.

## Acceptance criteria advanced or satisfied

- **0B-AC-006** — App Router; one root layout; one composition point; boots and
  serves.
- **0B-AC-008** — no product route exists; a product module can be added as a
  sibling route group without altering either file.

## Prerequisites

T01.

## Exact scope

1. `src/app/layout.tsx` — the one root layout: `<html lang>`, `<body>`, the
   `GeistSans` font from `geist/font/sans`, and exactly one `<Providers>`.
2. `src/app/providers.tsx` — the one composition point. At this task it composes
   nothing but its children; T14 adds the query client here, and nothing else
   ever composes providers elsewhere.
3. `src/app/page.tsx` — a minimal foundation demonstration surface. T09 replaces
   its body with the shell.
4. A header comment in `page.tsx` stating plainly that this is the **0B
   foundation demonstration surface, not §61A.1's Portfolio route**, so it cannot
   later be mistaken for delivered product capability.
5. `src/styles/globals.css` created with the Tailwind import only — tokens are
   T06.

## Expected files

```
src/app/layout.tsx  src/app/providers.tsx  src/app/page.tsx
src/styles/globals.css
```

## Implementation guidance

Server components are the default (plan §9). `providers.tsx` will need
`'use client'` once T14 adds the query client; keep it the only client boundary
in `src/app/`.

Organise routes so a future product module is a **sibling route group** — adding
one must not touch `layout.tsx` or `providers.tsx`. That is the property
0B-AC-008 checks, and it is cheaper to get right now than to retrofit.

Use the `geist` package's own `geist/font/sans` export (`GeistSans`) rather than
`next/font/google`. It wraps `next/font/local` over the 45 `woff2` files the
package bundles, so the font is self-hosted with no build-time network fetch,
which keeps `font-src 'self'` viable in T27 without an exception.

**This task carries the first required successful production build.** T01
deliberately stopped short of one, because no root layout existed yet.

## Validation

```
pnpm build   # the first required successful production build (deferred from T01)
pnpm dev     # confirm / renders the root layout
```

## Required negative tests

None at this task. Boot is asserted by component test in T10 and by browser test
in T22.

## Evidence to leave behind

A successful production build serving `/`.

## Stop conditions

- If a second composition point appears necessary for any reason, stop: S1 R13
  permits exactly one.

## What must NOT be implemented

- **No product route** — no `/applications`, `/admin`, `/ask`, `/sources` or any
  §61A.1 route (S1 R16; S2 R20).
- No Portfolio implementation. `page.tsx` is a foundation demonstration surface.
- No navigation entries, no application switcher content, no fabricated data.
- No auth routes (T15–T19), no query client (T14), no design tokens (T06).
- No `src/lib/telemetry/`.

## Completion definition

`pnpm build` succeeds — the first required successful production build in the
milestone; `/` serves the root layout through exactly one composition point; no
product route exists; the demonstration-surface comment is present.
