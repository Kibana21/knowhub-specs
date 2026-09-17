# 0B-T12 — Contract freshness, drift detection and offline generation

- **Phase:** 4
- **Depends on:** T11
- **Plan:** §12, §24

## Objective

Make the freshness property a wired, blocking gate that detects every drift case
S3 R24 enumerates, and prove generation needs no backend, no network and no
workstation state.

## Why this task exists

S3 R23 makes stale generated output a defect rather than a tolerated state, and
S5 R25 requires the gate to be wired and blocking. Generation that cannot be
trusted to be reproducible makes the whole contract boundary unverifiable.

## Approved requirement references

S3 R15, R16, R17, R23, R24, R25; S5 R24, R25, R26, R27.

## Acceptance criteria advanced or satisfied

- **0B-AC-046** — regeneration is byte-identical.
- **0B-AC-047** — malformed input fails clearly, leaving nothing partial.
- **0B-AC-048** — generation completes with no backend and no network.
- **0B-AC-049** — generated output is derived, not authoritative.
- **0B-AC-050 / 0B-AC-100** — all four drift cases detected; regeneration
  restores.
- **0B-AC-099** — the generation gates execute and block.

## Prerequisites

T11.

## Exact scope

1. `scripts/verify-contract-freshness.mjs` — regenerate into a temporary
   directory and byte-compare against the committed output, printing a readable
   diff on mismatch. Wired as `pnpm contract:verify`.
2. `tests/contract/` — the contract test layer, created because it now holds real
   tests.
3. Tests for all four S3 R24 drift cases:
   - generated output stale with respect to its input;
   - generated output that has been hand-edited;
   - generator or configuration drift that changes output;
   - invalid contract input.
4. The offline proof: generation with the network namespace removed.
5. `scripts/write-contract-evidence.mjs` — emits `contract-evidence.json`
   recording the input, its `sha256`, what it demonstrates, that **no product
   backend contract is bound**, and that **no compatibility is claimed**.

## Expected files

```
scripts/verify-contract-freshness.mjs
scripts/write-contract-evidence.mjs
tests/contract/generation.test.ts
```

## Implementation guidance

Byte-identity is the assertion (S3 R15) — compare bytes, not parsed structures.
Normalise line endings at write time so the check is stable across platforms.

Invalid input must fail **loudly**, with a message identifying the problem, and
must leave no partial, empty or silently stale output behind (S3 R16). Generate
to a temporary location and move into place only on success.

For the offline case use
`sudo unshare -n sudo -u "$USER" pnpm contract:verify`, with
`docker run --network none` on the digest-pinned Node image as the recorded
fallback. **Install dependencies before removing the namespace** — S5 R5 imposes
no general offline requirement; the single offline obligation is generation
itself.

`contract-evidence.json` must **fabricate no backend contract revision**. Where
only the fixture exists it records the absence of a bound product contract
explicitly, identifies the fixture as test and pipeline evidence only, and claims
no backend compatibility (S5 R27). This is what makes it impossible to confuse
pipeline verification with compatibility verification.

## Validation

```
pnpm contract:verify
pnpm test --project node
sudo unshare -n sudo -u "$USER" pnpm contract:verify
```

## Required negative tests

Four, each demonstrated failing and then restored by regeneration:

1. delete a line from the committed generated output → stale detected;
2. hand-edit the generated output → detected;
3. change the generator configuration → output drift detected;
4. point the pipeline at `malformed-fixture.openapi.json` → clear failure, no
   partial output left behind.

## Evidence to leave behind

`contract-evidence.json`; a transcript of the offline run; transcripts of all
four drift cases.

## Stop conditions

- If regeneration is not byte-stable, stop: 0B-AC-046 fails and the freshness
  property is unverifiable. Pin the generator harder or normalise output; do not
  relax the assertion to a structural comparison.
- If generation reaches the network, stop and remove the cause — a remote `$ref`
  in the fixture is the likely culprit.

## What must NOT be implemented

- No compatibility test against `knowhub-backend`, and **no claim of
  compatibility anywhere** (S3 R8; S5 R7).
- No fabricated backend contract revision in the evidence artifact (S5 R27).
- No general offline requirement imposed on other gates (S5 R5).
- No hand-editing of generated output as a fix — a change originates in the input
  or the configuration (S3 R20).
- No product contract, no product operation.

## Completion definition

`pnpm contract:verify` is green and blocking; all four drift cases have been
demonstrated failing and restored; generation succeeds with the network removed;
`contract-evidence.json` records the absence of a bound product contract and
claims no compatibility.
