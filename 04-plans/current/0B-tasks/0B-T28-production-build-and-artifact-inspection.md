# 0B-T28 — Production build and artifact inspection

- **Phase:** 7
- **Depends on:** T27, T21
- **Plan:** §21

## Objective

Make the production build a blocking gate, and inspect the produced bundle and
every emitted source map for secrets, server-only configuration values and
test-only material.

## Why this task exists

S5 R52–R53 require inspection of the produced artifact, and S5 R65 requires that
no development-only or test-only code reach the bundle. This is the gate that
turns the four containment mechanisms from a design claim into evidence.

## Approved requirement references

S5 R9, R10, R12, R52, R53, R54, R62, R63, R64, R65; S4 R7, R9, R73; S3 R37.

## Acceptance criteria advanced or satisfied

- **0B-AC-044** — no fixture-derived client ships.
- **0B-AC-055** — no credential exposure; no telemetry.
- **0B-AC-061** — no server-only value, resolved secret, provider credential or
  token in a bundle or source map.
- **0B-AC-086** — the double's containment contract, against the artifact.
- **0B-AC-092** — test-only material structurally unreachable, proved.
- **0B-AC-110** — inspection finds no secret and no test-only material.
- **0B-AC-113** — the build blocks; type and lint errors block it; no backend,
  IdP, product service or product contract required.

## Prerequisites

T27 (the production build is the artifact under inspection), T21 (the harness
sentinels this task asserts the absence of).

## Exact scope

1. `scripts/inspect-build-artifact.mjs` — scans `.next/static/**`,
   `.next/standalone/**` and every `*.map` for:
   - the **values** of server-only configuration present in the build
     environment;
   - forbidden patterns — `client_secret`, private-key headers, session-key
     material;
   - test-only sentinels — `KNOWHUB_TEST_ONLY_SENTINEL`, harness module names,
     fixture operation identifiers.
2. Wired as `pnpm inspect:artifact` and made blocking.
3. A `bundle-inspection-report.json` output for CI publication.
4. Confirmation that a strict type error and a lint violation each **block the
   production build**.
5. Confirmation that the build needs no live backend, no real identity provider,
   no product runtime service and **no product OpenAPI contract**.

## Expected files

```
scripts/inspect-build-artifact.mjs
package.json                       (script wiring)
```

## Implementation guidance

Inspect the **values**, not just the key names — a leaked secret appears as its
value. Read the server-only values from the configuration boundary's declared
set at inspection time so the check cannot drift from the schema.

Scan the browser-reachable output (`.next/static`) for server-only values and
sentinels, and scan **every** `*.map` regardless of where it sits: S5 R54 requires
no source-map policy but subjects whatever is emitted to R52 and R53.

The production build is the profile with the **unbound** authority binding. This
is what makes the absence of the harness sentinels meaningful — and it is the same
artifact T27 verified headers against and T29 will containerise.

Make the report machine-readable so T31 can publish it and T32 can cite it.

## Validation

```
pnpm build && pnpm inspect:artifact
```

## Required negative tests

Each demonstrated failing and restored:

- place a server-only configuration value in a client component → inspection
  fails;
- import a harness module into `src/**` (bypassing lint temporarily) → the
  sentinel appears in the bundle and inspection fails;
- introduce a type error → the production build fails;
- introduce a lint violation → the production build fails.

## Evidence to leave behind

`bundle-inspection-report.json`; transcripts of all four negative cases.

## Stop conditions

- **If any test-only material reaches the production bundle, STOP.** S5 R10(b) is
  absolute, and the cause is a containment failure, not a reporting one.
- If a server-only value appears in a browser bundle, stop: S4 R7 admits no
  exception.

## What must NOT be implemented

- No source-map policy imposed or forbidden (S5 R54) — the Next default is a
  reversible choice and whatever is emitted is inspected.
- No allowlist or suppression added to make the inspection pass.
- No product OpenAPI contract introduced to make the build succeed (S5 R64).
- No container work (T29), no scanning (T30).
- No test-only route, endpoint, authority or profile selector in the production
  runtime (S5 R10 c).

## Completion definition

The production build blocks on type and lint errors and needs no backend, IdP,
product service or product contract; inspection finds no secret, server-only
value or test-only material; all four negative cases have been demonstrated
failing and restored.
