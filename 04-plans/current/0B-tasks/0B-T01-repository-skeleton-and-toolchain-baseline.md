# 0B-T01 — Repository skeleton and toolchain baseline

- **Phase:** 1 — Toolchain and repository skeleton
- **Depends on:** —
- **Plan:** [`../0B-PLAN-frontend-foundation.md`](../0B-PLAN-frontend-foundation.md) §7, §8

## Objective

Turn `knowhub-frontend` from two files into a repository that installs
reproducibly from a committed lockfile on a declared Node baseline, and that
carries a `README.md` describing the documented local workflow.

## Why this task exists

Every later task depends on a resolvable dependency graph and a shared runtime.
S1 R2–R6 make the lockfile the single resolution path and require the Node,
Next.js and React baselines to be declared and shared between contributors and
CI. Nothing can be gated before this exists.

## Approved requirement references

S1 R1, R2, R3, R4, R5, R6, R26, R27, R29, R30.

## Acceptance criteria advanced or satisfied

- **0B-AC-001** — clean checkout reaches a working environment through pnpm alone.
- **0B-AC-002** — manifest and lockfile consistent; staleness detected mechanically.
- **0B-AC-003** — Node, Next.js and React baselines declared (CI half in T31).
- **0B-AC-004** — strict TypeScript configured (rule enforcement in T02).
- **0B-AC-012** — `docs/adr/` present; `README.md` describes the workflow.
- **0B-AC-013** — the documented sequence exists and runs with this repo alone.

## Prerequisites

None. `knowhub-frontend` is clean at `b2787a57`.

## Exact scope

1. `package.json` — name, private, `engines.node: ">=24.21.0 <25"`,
   `packageManager: "pnpm@12.4.2"`, the dependency set pinned in plan §7.2, and
   the script set in plan §7.3.
2. `.nvmrc` containing `24.21.0`.
3. `.npmrc` — `engine-strict=true`, `frozen-lockfile=true`.
4. `pnpm-workspace.yaml` — `onlyBuiltDependencies` allowlist for the native
   packages that legitimately need a build step.
5. `tsconfig.json` — `strict: true`, `noUncheckedIndexedAccess`,
   `noImplicitOverride`, `moduleResolution: "bundler"`, the `@/*` path alias, and
   an `include` that covers `src` and configuration only — **`tests/` is excluded
   from the build program** (this is one of the four containment mechanisms).
6. `.gitignore` — `node_modules`, `.next`, `playwright-report`, `test-results`,
   `*.local`, coverage output.
7. `.prettierignore`, `prettier.config.mjs` (with `prettier-plugin-tailwindcss`).
8. `postcss.config.mjs` — `@tailwindcss/postcss`.
9. `next.config.ts` — `output: 'standalone'`; nothing else yet.
10. `README.md` — what the repository is, the Node/pnpm baseline, the documented
    local workflow (`pnpm install` → `pnpm verify`), and the stated
    **server-components-by-default** policy required by S1 R14.
11. **First-lockfile bootstrap.** The repository begins with no
    `pnpm-lock.yaml`, so frozen mode cannot create it. Generate it once with an
    explicit non-frozen bootstrap command — `pnpm install --no-frozen-lockfile` —
    and commit the result. Every subsequent install, local and CI, is frozen.

## Expected files

```
package.json  pnpm-lock.yaml  pnpm-workspace.yaml  .npmrc  .nvmrc
tsconfig.json  next.config.ts  postcss.config.mjs
prettier.config.mjs  .prettierignore  .gitignore  README.md
```

## Implementation guidance

Pin exact versions, not ranges — S1 R3 requires that an install which would
change resolution fails, and `frozen-lockfile=true` plus exact pins is the
simplest way to hold that. **This includes every Radix package**: plan §7.2
records the exact selected versions, and no floating value may remain in the
implemented manifest. If a resolved version differs from the recorded one,
record the resolved version in implementation evidence and pin that.

The first install is the one exception to frozen mode, and only because there is
nothing yet to be frozen against. Use `--no-frozen-lockfile` explicitly, once, so
the bootstrap is a visible deliberate act rather than a silent fallback.

**Do not raise TypeScript above 5.9.3.** Plan §7.1 records the reason:
`typescript-eslint` declares `typescript >=4.8.4 <6.1.0` and `openapi-typescript`
declares `^5.x`. Raising it breaks the type-aware lint gate and the generation
pipeline — the two things 0B exists to prove. The same applies to keeping Vitest
at 4.1.11. Both are routine dependency maintenance to revisit later, not
architectural decisions.

**Node must be 24.21.0 or a later 24.x patch.** `jsdom@30.1.0` declares
`^24.15.0`; a 24.13.x baseline would break the component layer in T10.

`docs/adr/` already exists and holds ADR-024. Do not move, edit or copy it.

## Validation

```
# once, and only once, because no lockfile exists yet:
pnpm install --no-frozen-lockfile

# thereafter, and in CI, always:
pnpm install --frozen-lockfile
pnpm install --lockfile-only && git diff --exit-code pnpm-lock.yaml
node --version            # must match .nvmrc
pnpm typecheck
```

`pnpm build` is deliberately **not** run here. There is no `src/app/layout.tsx`
and no `src/app/page.tsx` until T03, so a production application build cannot
succeed and is not required. The first required successful `pnpm build` belongs
to T03.

## Required negative tests

- Hand-edit a dependency version in `package.json` without regenerating the lock;
  `pnpm install --frozen-lockfile` must **fail**. Revert and confirm it passes.

## Evidence to leave behind

The committed lockfile; a recorded transcript of the failing and then passing
frozen-install runs.

## Stop conditions

- A pinned version in plan §7.2 turns out to have an unsatisfiable peer or engine
  constraint on install → stop, report the conflict, and select the newest stable
  version inside the mutually supported range rather than guessing.
- Any dependency requires a second package manager, a vendored tree or a globally
  installed tool to obtain a working environment (S1 R2).

## What must NOT be implemented

- **No application source at all** — the root layout and page belong to T03, and
  no production application build is attempted here.
- No ESLint configuration — that is T02.
- No `src/lib/telemetry/`, no product route, no product component directory.
- No `public/`, `deploy/`, `src/hooks/`, `src/view-models/` or
  `src/lib/format/` (S1 R9, R11).
- No `contracts/knowhub-openapi.json` and no OpenAPI artifact of any kind.
- No CI workflow (T31).
- No committed secret, token, credential or `.env` file — only `.env.example`
  arrives later, in T05.

## Completion definition

The lockfile exists and is committed; `pnpm install --frozen-lockfile` and
`pnpm typecheck` succeed on a clean checkout of only `knowhub-frontend`; the
lockfile-currency check passes; the stale-manifest frozen-install failure has
been demonstrated and reverted; every dependency including each Radix package is
pinned to an exact version; `README.md` documents the workflow and the
server/client policy.

**A successful `pnpm build` is not part of this task's completion** — it is
required by T03, once the root layout and page exist.
