# 0B-T30 — Supply-chain gates

- **Phase:** 8
- **Depends on:** T29
- **Plan:** §23

## Objective

Make lockfile integrity, dependency vulnerabilities, committed secrets and
container vulnerabilities blocking gates, each demonstrated capable of failing.

## Why this task exists

§0E.16's Supply chain row and §0E.8's frontend CI gate list require these
controls, and S5 R58 specifically requires the secret scanner to be demonstrated
against an **ephemeral synthetic credential** rather than assumed to work.

## Approved requirement references

S5 R55, R56, R57, R58, R59, R60, R61, R68; S1 R2, R3, R4, R28; S4 R38.

## Acceptance criteria advanced or satisfied

- **0B-AC-001, 002** — frozen install and lockfile currency, as blocking gates.
- **0B-AC-012** — no committed secret, credential, token, snapshot or key.
- **0B-AC-076** — no hard-coded privileged credential; no committed enabled
  bypass.
- **0B-AC-111** — frozen install, currency and dependency vulnerability gates;
  zero findings accepted as a pass; no SBOM, provenance, signing or licence work.
- **0B-AC-112** — the secret scanner demonstrated against an ephemeral synthetic
  credential.
- **0B-AC-114** — the container vulnerability scan (scan half).

## Prerequisites

T29 — the image to scan.

## Exact scope

1. Frozen install: `pnpm install --frozen-lockfile`.
2. Lockfile currency, as an **independent** assertion:
   `pnpm install --lockfile-only` then `git diff --exit-code pnpm-lock.yaml`.
3. Dependency vulnerabilities: `pnpm audit --audit-level=high`.
4. Secrets: gitleaks, container pinned by digest, over the **working tree and the
   full git history**, with the ephemeral synthetic-credential demonstration.
5. Container vulnerabilities: Trivy, pinned by digest,
   `--severity HIGH,CRITICAL --ignore-unfixed --exit-code 1`.
6. An inventory confirming no hard-coded privileged credential — no default
   administrator, no universal username and password, no development password, no
   committed enabled insecure bypass.

## Expected files

```
(CI step definitions land in T31; this task establishes and proves each gate,
 plus tests/fixtures/secret-scan/ used transiently by the demonstration)
```

## Implementation guidance

**Keep the currency check separate from the frozen install.** They fail on
different conditions and the M00 backend workflow documents exactly why a lock
that is stale relative to the manifest can otherwise pass silently.

**The secret-scan demonstration must be ephemeral** (S5 R58): create a verification
input containing a **synthetic** credential, run the scan and observe it fail,
then delete the input and observe it pass. No real or persistent credential is
committed, and **no detectable test credential remains** once verification
completes. Scan the full history — a credential removed in a later commit is still
a leaked credential.

**Zero findings is a pass** (S5 R60). A scan that reports nothing has done its
job; it is not evidence that a gate is missing or misconfigured. Say so in the
step's comment so a future reader does not "fix" it.

`--ignore-unfixed` on Trivy keeps the gate actionable — it fails on what can be
remediated. **Never widen the severity threshold to get a pass**; bump the base
image instead.

## Validation

```
pnpm install --frozen-lockfile
pnpm install --lockfile-only && git diff --exit-code pnpm-lock.yaml
pnpm audit --audit-level=high
<gitleaks over tree and history>
<trivy over knowhub-frontend:<sha>>
```

## Required negative tests

- Stale lockfile → the frozen install fails; restore.
- An ephemeral synthetic credential in a verification input → the secret scan
  fails; delete it and confirm the scan passes and nothing detectable remains.
- A base image with a known HIGH finding → Trivy fails; restore.

## Evidence to leave behind

Transcripts of each gate passing and each negative case failing; the Trivy report;
confirmation that the ephemeral credential is gone.

## Stop conditions

- If a base-image CVE policy cannot be met, bump the base image. **Never widen
  the severity threshold or add a blanket ignore to pass.**
- If a real credential is found in history, stop and escalate — rotation is
  required, not suppression.

## What must NOT be implemented

- **No SBOM generation, provenance generation or artifact signing**, and no
  tooling, format or registry integration for them (S5 R61).
- **No licence scanning** — not a KnowHub requirement at 0B.
- No blanket allowlist, ignore file or suppression added to make a scan pass.
- No scanning product treated as part of the architectural contract — each is a
  reversible plan choice.
- No registry publishing.

## Completion definition

All five gates run and block; every negative case has been demonstrated failing
and restored; the ephemeral synthetic credential has been created, detected and
removed with nothing detectable remaining; no SBOM, provenance, signing or licence
tooling has been introduced.
