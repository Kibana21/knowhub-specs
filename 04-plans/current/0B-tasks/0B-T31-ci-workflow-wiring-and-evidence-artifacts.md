# 0B-T31 — CI workflow wiring and evidence artifacts

- **Phase:** 9 — CI wiring
- **Depends on:** T01 – T30
- **Plan:** §24

## Objective

Wire every 0B gate into one blocking CI workflow that runs with only
`knowhub-frontend` checked out, and publish the required evidence artifacts.

## Why this task exists

S5 R71 lists sixteen required, blocking gates and S5 R72 forbids any of them
being advisory. S5 R73 requires the complete path to run with only this
repository present — the property that makes ADR-016's independence real rather
than asserted.

## Approved requirement references

S5 R7, R18, R27, R71, R72, R73, R74, R75, R76, R77; S1 R5, R29; ADR-016, ADR-017.

## Acceptance criteria advanced or satisfied

- **0B-AC-003** — gates execute on the declared Node baseline.
- **0B-AC-011** — lint and format block in CI.
- **0B-AC-013** — the full sequence with only this repository cloned.
- **0B-AC-041** — no document or gate result implies backend compatibility.
- **0B-AC-048** — no-network generation, in CI.
- **0B-AC-091** — determinism and independence; the scanner-data caveat recorded.
- **0B-AC-093** — the complete validation sequence, repository-alone.
- **0B-AC-115** — the whole CI contract.

## Prerequisites

T01 – T30. Every gate exists and has been demonstrated failing.

## Exact scope

1. `.github/workflows/ci.yml` — seven jobs per plan §24: `quality`, `test`,
   `contract`, `browser`, `security`, `build`, `image`.
2. Every action pinned by commit SHA; every container image by digest.
3. Node from `.nvmrc`, so CI and local tooling share the declared baseline.
4. The no-network generation step:
   `sudo unshare -n sudo -u "$USER" pnpm contract:verify`, with
   `docker run --network none` as the recorded fallback — dependencies installed
   **before** the namespace is removed.
5. Published evidence: `playwright-report/`, Vitest and Playwright JUnit XML,
   `a11y-report.json`, `contract-evidence.json`,
   `security-headers-report.json`, `bundle-inspection-report.json`, the Trivy
   report and the container image.
6. Each artifact labelled a **CI evidence artifact**, not runtime telemetry.

## Expected files

```
.github/workflows/ci.yml
```

## Implementation guidance

**Name jobs by outcome**, as the backend workflow does, so a later move to another
provider is a translation rather than a redesign (S5 R76). GitHub Actions is
selected now because §0E.1 permits it and both repositories are hosted there —
the choice is reversible and fixes nothing architectural.

**Constrain ordering only where it is architecturally meaningful** (S5 R77):
artifact inspection and header verification run against the artifact `build`
produced, and the container scan runs after the image exists. Everything else is
free to parallelise, and over-serialising is a cost with no requirement behind it.

**No job, step or script may fetch from or depend on a `knowhub-backend`
checkout, source tree, Python package or runtime** (S5 R73). The one preserved
exception is a legitimate pinned backend-generated OpenAPI artifact as contract
**input**, which may retain its recorded provenance — that is an artifact input,
not a source or runtime dependency.

**Record the determinism caveat** (0B-AC-091): determinism is not claimed for
mutable external security intelligence. A vulnerability database may change a
scan result between runs; those gates stay blocking and no requirement to pin or
snapshot that data is created. Put this in the workflow comments so it is not
later mistaken for flakiness.

**CI evidence is not runtime telemetry** (S5 R75). §81.1's telemetry condition
rests on the backend runtime telemetry M00-SPEC-003 established plus the
correlation propagation S3 R34–R36 builds into it. Say so where the artifacts are
uploaded.

## Validation

A complete green run on a clean checkout of only `knowhub-frontend`.

## Required negative tests

- Introduce a violation for each gate class in turn and confirm CI **fails** —
  reusing the demonstrations recorded in T02, T04, T12, T23, T27, T28 and T30.
- Confirm no job references `knowhub-backend`.

## Evidence to leave behind

A green CI run; every listed artifact published and labelled.

## Stop conditions

- **If any gate cannot run without a backend checkout, a running backend, a real
  identity provider, a sibling repository or workstation state, STOP** — that
  breaches S5 R73 and ADR-016.
- If a gate can only pass when made advisory, stop: S5 R72 forbids it.

## What must NOT be implemented

- **No advisory gate, no placeholder job, no empty test layer created in order to
  have a gate** (S5 R72).
- **No dependency on `knowhub-backend`** in any job, step or script.
- No SBOM, provenance, signing or licence step (S5 R61).
- No registry push or deployment step (S5 R69).
- No browser telemetry SDK, vendor integration or metrics framework (S5 R75).
- **No fabricated backend contract revision** in the published evidence, and no
  statement implying backend compatibility (S5 R27; S3 R8).
- No unpinned action and no untagged container image.

## Completion definition

One workflow runs every S5 R71 gate as blocking with only `knowhub-frontend`
checked out; the no-network generation step passes; every evidence artifact is
published and labelled as CI evidence; the determinism caveat and the
no-compatibility statement are recorded; no job references the backend.
