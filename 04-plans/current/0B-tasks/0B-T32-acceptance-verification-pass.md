# 0B-T32 — 0B acceptance verification pass

- **Phase:** 10 — Acceptance verification
- **Depends on:** T31
- **Plan:** §27, §28, §32

## Objective

Verify all 102 allocated acceptance criteria against honest evidence, reconcile
the AT catalogue, and produce the verification report that gates milestone exit.

## Why this task exists

§81.1 is explicit that a milestone does not exit because files and classes exist.
It exits when its acceptance tests pass, failure paths are exercised and the
preceding contracts remain stable. This task is where that judgement is made and
recorded.

## Approved requirement references

S5 R6, R7, R8, R22, R27, R38; §81.1; §84.2; §84.4; and every requirement in
S1–S5 by reference through the acceptance matrix.

## Acceptance criteria advanced or satisfied

Final confirmation of **all 102**: 0B-AC-001–013, 020–036, 040–058, 060–086,
090–115. Specifically produced here: **0B-AC-005** (final inventory),
**0B-AC-010** (review), **0B-AC-013**, **0B-AC-041**, **0B-AC-090**.

## Prerequisites

T31 — a complete green CI run.

## Exact scope

1. Walk plan §27's matrix criterion by criterion, confirming each named
   verification mechanism exists, executes and produced the evidence claimed.
2. Confirm every review-based criterion is evidenced by an identified inspection
   or recorded review, and that **none is described as an automated result**.
3. Confirm the permanently unallocated identifiers — 014–019, 037–039, 059,
   087–089, 116–119 — have **not been assigned to a requirement, task,
   acceptance mapping or implementation claim**. They may legitimately appear in
   documentation that lists them as reserved and unallocated, so **a literal
   search that finds one in such a listing is not a failure**; check assignment,
   not occurrence.
4. Run the anti-scope-creep control list of plan §29.
5. Confirm every gate has been demonstrated capable of failing.
6. Author `knowhub-specs/04-plans/completed/0B-VERIFICATION-REPORT.md` — or its
   agreed location — mapping every requirement and every acceptance criterion to
   its evidence, and recording every non-claim verbatim.
7. Confirm all three working trees are clean and no secret is committed.

## Expected files

```
0B-VERIFICATION-REPORT.md
```

## Implementation guidance

**Evidence, not existence.** For each criterion ask what would have failed had the
property not held. A criterion whose only evidence is that a file exists is not
evidenced (§81.1; §84.2).

**Reconcile the AT catalogue exactly as the specifications state it**, extending
nothing:

- **AT-035 — owned by S5 and fully evidenced at 0B** by 0B-AC-107 and 0B-AC-108,
  at the production artifact and runtime boundary, subject to S5 R49's scope.
- **AT-027 — fully discharged.**
- **AT-029 — fully discharged.**
- **AT-028 — partial**; only the browser half against a synthetic authority.
- **AT-026 — partial**; only the non-leakage half.
- **AT-037 — partial**; only the negative half.
- **AT-017 — partial**; only the shell, primitives and state classes.
- **AT-025, AT-030, AT-031, AT-042** — frontend preconditions only.
- All remaining §75 entries are backend or product capability at M1 and later.

**Record the non-claims verbatim** (S5 R38): no real backend token rejection,
session invalidation, server-side token-cache invalidation, refresh-token
rotation, LOCAL credential verification, federated-token validation, OIDC tenant
integration or backend authorization is claimed; and no evidence demonstrates
compatibility with `knowhub-backend`, the existence of any backend endpoint, or a
frontend/backend version pair.

Use the M00 verification report as a **structural** precedent only.

## Validation

```
pnpm verify                     # on a clean checkout of only knowhub-frontend
<a complete green CI run>
git status                      # all three repositories clean
```

## Required negative tests

- Confirm, from the recorded transcripts, that **every** blocking gate has been
  demonstrated failing on the condition it exists to detect (S5 R6).

## Evidence to leave behind

`0B-VERIFICATION-REPORT.md`, mapping all 102 criteria to evidence, reconciling the
AT catalogue and recording every non-claim.

## Stop conditions

- **If any criterion lacks honest evidence, STOP.** Do not mark it satisfied, do
  not soften its wording, and do not substitute a weaker mechanism.
- If a review-based criterion has been recorded as an automated result, correct
  the record (S5 R22).
- If any statement implying backend compatibility is found anywhere in the
  repository, remove it (S3 R8; 0B-AC-041).

## What must NOT be implemented

- **No new capability** — this task verifies; it does not build.
- **No relaxation of a criterion** to make it pass.
- **No claim beyond the AT reconciliation above.**
- **No move of the implementation plan to `04-plans/completed/`** until every
  §32 condition holds and acceptance is agreed.
- No declaration that 0B is complete on the strength of existing files.

## Completion definition

All 102 criteria are evidenced by the mechanism plan §27 assigns; review-based
criteria are identified as such; no permanently unallocated identifier has been
assigned to a requirement, task, mapping or claim; every gate has
been shown capable of failing; CI is green with only `knowhub-frontend` checked
out; the production image is built and scanned; no M1 scope creep is present; all
working trees are clean; and `0B-VERIFICATION-REPORT.md` records every mapping and
every non-claim. **Only then** may the plan move from `04-plans/current/` to
`04-plans/completed/`.
