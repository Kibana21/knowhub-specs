# 0B-T11 — Synthetic contract fixture and the generation entry point

- **Phase:** 4 — Contract pipeline and API/query boundary
- **Depends on:** T05
- **Plan:** §12

## Objective

Create the governed `contracts/` directory, the sanctioned synthetic OpenAPI
fixture, and the single deterministic generation entry point that takes its
document as an explicit input.

## Why this task exists

On the normal 0B path **no backend product contract exists** — verified: the
backend publishes none. S3 is deliberately built so the pipeline can be completed
and proved without one, and S3 R4 forbids manufacturing a substitute. The fixture
is the mandatory demonstration input.

## Approved requirement references

S3 R1, R2, R3, R4, R5, R6, R7, R8, R9, R10, R11, R12, R13, R14, R15, R17, R18,
R19, R20, R21, R22.

## Acceptance criteria advanced or satisfied

- **0B-AC-040** — `contracts/` holds only the governance note; no product
  contract manufactured.
- **0B-AC-042** — the fixture carries no KnowHub vocabulary or product-like path.
- **0B-AC-043** — the fixture is test-only and unreachable from production.
- **0B-AC-044** — nothing generated from it ships as a product client.
- **0B-AC-045** — generation accepts an explicitly specified local document.
- **0B-AC-049** — generated output is derived, in a distinct location.

## Prerequisites

T05 (the configuration boundary exists), T04 (the boundary gate can enforce
fixture containment).

## Exact scope

1. `contracts/README.md` — the governance note recording S3 R2–R4 and the
   `/api/v1` versioning rule, and stating plainly that **no product contract
   artifact exists at 0B** and none may be hand-authored.
2. `tests/fixtures/contract/synthetic-pipeline-fixture.openapi.json` — an
   obviously synthetic surface.
3. `tests/fixtures/contract/malformed-fixture.openapi.json` — for T12's
   invalid-input case.
4. `contract-generation.config.json` — names `input` and `output` explicitly;
   records `src/lib/api/generated/` as the **product** output path, which is
   **not created at 0B**.
5. `scripts/generate-api-client.mjs` — the one entry point, wired as
   `pnpm generate:api` (the §0E.8 and §30A.2 name). It fails if no input is
   specified; it discovers nothing implicitly.
6. `tests/fixtures/contract/generated/` — the committed fixture output.
7. A boundary rule confirming no production module can import the fixture.

## Expected files

```
contracts/README.md
contract-generation.config.json
scripts/generate-api-client.mjs
tests/fixtures/contract/synthetic-pipeline-fixture.openapi.json
tests/fixtures/contract/malformed-fixture.openapi.json
tests/fixtures/contract/generated/**
```

## Implementation guidance

**The fixture must be unmistakably synthetic** (S3 R10). No KnowHub resource
vocabulary, no M1 endpoint name, no `/api/v1` or other product-like path adopted
to resemble KnowHub, no authentication schema, no business schema. Nothing about
it may be read as describing what the backend has, will have, or is being asked
to have. Use a deliberately unrelated domain and say so in a description field.

**The input is explicit** (S3 R14): pointing the pipeline at a different document
is a configuration change, never a code change — which is exactly what makes
substituting a genuine artifact later a config edit rather than a rewrite. Give
`input` no default.

**Do not create `src/lib/api/generated/`.** No product contract exists, so the
directory would be empty and S1 R9 forbids that. The path is recorded in the
config; the directory is created by the change that pins a real artifact.

**Both R7 paths must stay valid.** Write the script so the presence of
`contracts/knowhub-openapi.json` is a branch, not a rewrite — if a legitimate
backend-generated artifact arrives through §30A.2 before 0B completes, only the
config `input` changes.

The fixture must contain **no remote `$ref`**, or T12's offline case cannot pass.

## Validation

```
pnpm generate:api --config contract-generation.config.json
pnpm generate:api            # must FAIL: no input specified
pnpm lint && pnpm typecheck
```

## Required negative tests

- Invoke generation with no input → fails rather than discovering a document.
- Attempt to import the fixture from `src/**` → lint fails.

## Evidence to leave behind

The committed fixture, the committed generated output, and the governance note.

## Stop conditions

- If a legitimate backend-generated artifact has appeared through the §30A.2
  workflow, follow the R7 exception path: pin it under S3 R2–R4, record its
  backend revision, and **still** claim no compatibility. Do not treat its
  arrival as permission to claim more.
- If the fixture cannot avoid product-like vocabulary, stop and redesign it.

## What must NOT be implemented

- **No `contracts/knowhub-openapi.json`** and no hand-authored product OpenAPI
  document, path or schema of any kind (S3 R4, R7).
- No promotion, copy or rename of the fixture into `contracts/` (S3 R12).
- No `src/lib/api/generated/` directory.
- No product operation, no authentication schema, no business schema in the
  fixture.
- No claim, in any file or comment, that anything here demonstrates compatibility
  with `knowhub-backend` (S3 R8).
- No API boundary code — T13.

## Completion definition

`contracts/` holds only the governance note; the fixture is synthetic, contained
and unreachable from production; generation runs from an explicit input and fails
without one; the generated output is committed in a location distinct from
handwritten source.
