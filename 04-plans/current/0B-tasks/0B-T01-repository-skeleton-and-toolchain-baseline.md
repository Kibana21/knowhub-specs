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
3. `pnpm-workspace.yaml` — the pnpm settings file, and the **authoritative
   enforcement location** for the install policy under pnpm 12 (plan §7.4):
   - `frozenLockfile: true` — the committed lockfile is the single resolution
     path, on the default `pnpm install` path and not only when
     `--frozen-lockfile` is passed explicitly.
   - `engineStrict: true` — dependency engine ranges are enforced rather than
     advisory. See scope item 12 for what this does and does not cover.
   - `allowBuilds` — the allowlist of packages permitted to run a build step.
4. `.npmrc` — retained for registry-level and npm-compatible configuration. It
   **must not** be described or relied on as the enforcement mechanism for the
   frozen lockfile or engine strictness: verified with pnpm 12.4.2, the
   kebab-case `frozen-lockfile=true` and `engine-strict=true` keys are not read
   from this file, and with only those keys present a bare `pnpm install`
   updated the lockfile. Either omit the two keys or keep them as an
   explicitly-labelled compatibility restatement of the policy expressed in
   `pnpm-workspace.yaml` — never as its only expression.

   Note when re-checking this: `pnpm config get frozen-lockfile` returns
   `undefined` only while the setting is absent from `pnpm-workspace.yaml`. Once
   `frozenLockfile: true` is set there, the kebab-case query resolves to it and
   returns `true`. The kebab spelling is a query alias, not a way to set the
   value from `.npmrc`. See plan §7.4 for the isolated reproduction.
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
    and commit the result. Every subsequent install, local and CI, is frozen:

    ```text
    Initial repository only:   pnpm install --no-frozen-lockfile
    Once the lockfile exists:  pnpm install --frozen-lockfile
    Normal/default install:    frozenLockfile: true prevents silent drift
    ```

    The `--no-frozen-lockfile` flag overrides the configured `frozenLockfile:
    true` for exactly this one bootstrap.

12. **The build allowlist is derived mechanically, never by inspection.** A
    package is added to `allowBuilds` only after pnpm has reported its build as
    ignored and the build has been confirmed to be a genuine native compilation
    step. At 0B that is exactly one package:

    ```yaml
    allowBuilds:
      unrs-resolver: true
    ```

    `unrs-resolver` is a native Rust module resolver reached through the
    approved lint toolchain — `eslint-config-next` →
    `eslint-import-resolver-typescript` → `unrs-resolver` — whose install step
    selects the platform-native binding; the lint gate cannot resolve imports
    without it. Arbitrary dependency build execution remains disabled. **Do not
    expand the allowlist** to a package that has not been through this
    discovery.

13. **Release-age exclusions required to install the approved pinned set.**
    pnpm 12 applies a default publication cooldown and enforces it on frozen
    installs as well as on resolution. Two versions pinned by plan §7.2 —
    `jsdom@30.1.0` and `lucide-react@1.47.0` — were published inside that
    window at bootstrap time, so without exact-version exclusions
    `pnpm install --frozen-lockfile` fails with
    `ERR_PNPM_MINIMUM_RELEASE_AGE_VIOLATION` and a clean checkout cannot
    install at all:

    ```yaml
    minimumReleaseAgeExclude:
      - jsdom@30.1.0
      - lucide-react@1.47.0
    ```

    Retain the approved version pins and scope the exclusion to those exact
    versions. **Do not disable, lower or globally weaken the cooldown**, and do
    not declare a cooldown value here — **T30 owns supply-chain policy**, not
    T01. Every other package remains subject to the protection, and exact-version
    pinning plus the lockfile integrity hashes remain the resolution authority.
    The entries become inert once those versions age past the window and should
    be removed then.

## Expected files

```
package.json  pnpm-lock.yaml  pnpm-workspace.yaml  .npmrc  .nvmrc
tsconfig.json  next.config.ts  postcss.config.mjs
prettier.config.mjs  .prettierignore  .gitignore  README.md
```

## Implementation guidance

Pin exact versions, not ranges — S1 R3 requires that an install which would
change resolution fails, and `frozenLockfile: true` in `pnpm-workspace.yaml`
plus exact pins is the simplest way to hold that. **This includes every Radix
package**: plan §7.2
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
`^24.15.0`; a 24.13.x baseline would break the component layer in T10. This was
confirmed empirically: with `engineStrict: true`, installing under Node 25.2.1
fails with `ERR_PNPM_UNSUPPORTED_ENGINE` naming exactly that `jsdom` range.

**What enforces the Node baseline, precisely.** Four mechanisms with distinct
responsibilities — do not claim that any one of them does another's job:

| Mechanism | Responsibility |
|---|---|
| `.nvmrc` | defines the developer Node baseline (`24.21.0`) |
| `package.json#engines.node` | **declares** the supported application range (`>=24.21.0 <25`) |
| `engineStrict: true` | rejects incompatible **dependency** engine/runtime combinations |
| T31 CI | explicitly selects Node `24.21.0` from `.nvmrc`, giving automated repository-level enforcement |

`engineStrict: true` enforces the engine ranges **declared by dependencies**. It
does **not** by itself enforce the root project's own `engines.node` range: an
install with a deliberately impossible root range of `>=99.0.0` was observed to
succeed. In this dependency set the declared baseline is nonetheless enforced in
practice, because the dependencies' own ranges reject anything outside Node
24.15+, but the `<25` upper bound is not itself machine-checked at T01.
Repository-level enforcement arrives with **T31**, which owns the CI half of
0B-AC-003. Do not move that work into T01, and do not claim T01 fully satisfies
0B-AC-003 — T01 **advances** it.

`docs/adr/` already exists and holds ADR-024. Do not move, edit or copy it.

## Validation

```
# once, and only once, because no lockfile exists yet:
pnpm install --no-frozen-lockfile

# thereafter, and in CI, always:
pnpm install --frozen-lockfile
pnpm install --lockfile-only && git diff --exit-code pnpm-lock.yaml
node --version            # must match .nvmrc
pnpm --version            # must match packageManager
pnpm typecheck
```

Until the lockfile is committed, `git diff --exit-code pnpm-lock.yaml` has no
tracked baseline to compare against. Evidence the currency check either after
committing the lockfile, or by confirming the file is byte-identical across
repeated regeneration — both forms were used at implementation.

`pnpm build` is deliberately **not** run here. There is no `src/app/layout.tsx`
and no `src/app/page.tsx` until T03, so a production application build cannot
succeed and is not required. The first required successful `pnpm build` belongs
to T03.

## Required negative tests

Hand-edit a dependency version in `package.json` without regenerating the lock,
then run **both** install paths. Revert and confirm each passes again.

1. **Explicit frozen install.** `pnpm install --frozen-lockfile` must **fail**
   with `ERR_PNPM_OUTDATED_LOCKFILE`, naming the mismatched dependency.
2. **The configured default path.** A bare `pnpm install`, with no CLI flag,
   must **also fail** with the same error and must not rewrite the lockfile.

The second case is what proves `frozenLockfile: true` is actually in force. It is
required precisely because it is the case that failed when the policy was
expressed only in `.npmrc`: the bare install silently updated the lockfile, which
is the drift S1 R3 and 0B-AC-001 forbid. A gate that only holds when a developer
remembers a flag is not a gate.

## Evidence to leave behind

The committed lockfile; a recorded transcript of the failing and then passing
runs for **both** install paths; and the effective values of `frozenLockfile`
and `engineStrict` as reported by `pnpm config get`.

## Stop conditions

- A pinned version in plan §7.2 turns out to have an unsatisfiable peer or engine
  constraint on install → stop, report the conflict, and select the newest stable
  version inside the mutually supported range rather than guessing.
- Any dependency requires a second package manager, a vendored tree or a globally
  installed tool to obtain a working environment (S1 R2).
- A package-manager control named by this task or by plan §7 turns out not to
  enforce what it is relied on to enforce → stop, record the verified behaviour,
  and express the policy through the mechanism that **is** effective on the
  pinned pnpm rather than leaving an inert setting in place. This is a
  plan-level tool-configuration refinement (plan §7.4), not a requirement
  change; it is a §84.4 stop condition only if no available mechanism can hold
  the requirement.

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
lockfile-currency check passes; the stale-manifest failure has been demonstrated
and reverted **on both the explicit frozen path and the configured default
path**; `frozenLockfile` and `engineStrict` are in force from
`pnpm-workspace.yaml` and `.npmrc` is not relied on for either; the `allowBuilds`
allowlist names only mechanically-discovered native build steps; every dependency
including each Radix package is pinned to an exact version; `README.md` documents
the workflow and the server/client policy.

**A successful `pnpm build` is not part of this task's completion** — it is
required by T03, once the root layout and page exist.
