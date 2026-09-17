# 0B-T23 — Authoritative accessibility pass and recorded review

- **Phase:** 6
- **Depends on:** T22
- **Plan:** §19

## Objective

Run the authoritative axe pass in Chromium — where layout-dependent rules can
actually be evaluated — and record the review evidence for criteria automation
cannot decide.

## Why this task exists

`color-contrast` and WCAG 2.2 AA `target-size` (2.5.8), both required by
0B-AC-031, cannot be evaluated in jsdom, which has no layout engine. Running them
only in the T10 component layer would produce a green result that proves nothing.
S5 R22 then requires that everything automation cannot decide be a targeted
assertion or a **recorded review**, never presented as an automated result.

## Approved requirement references

S5 R21, R22, R23; S2 R4, R29, R30, R33, R34, R35, R36, R37, R39, R44.

## Acceptance criteria advanced or satisfied

- **0B-AC-031** — contrast, accessible names, labels, error association, target
  size, text alternatives, landmark structure.
- **0B-AC-032** — no state signalled by colour alone.
- **0B-AC-036** — the R44 prohibition checklist.
- **0B-AC-097** — the gate blocks; review evidence identified as such; nothing
  fabricated; AT-017 not presented as discharged.

## Prerequisites

T22.

## Exact scope

1. `tests/e2e/accessibility/` — `@axe-core/playwright` over the shell, the
   primitive layer and the state presentations, with tags `wcag2a`, `wcag2aa`,
   `wcag21a`, `wcag21aa`, **`wcag22aa`** enabled. Blocking.
2. Targeted assertions where axe cannot decide: non-colour status signalling,
   text alternatives for meaningful visuals, and error-to-control programmatic
   association.
3. `docs/ux/0B-accessibility-review.md` — the recorded review. Dated and
   attributed; one row per criterion with **criterion · method
   (`inspection` | `targeted assertion`) · surface · outcome · reviewer**.
4. The R44 prohibition checklist worked through row by row in that document.
5. An `a11y-report.json` output for CI publication.

## Expected files

```
tests/e2e/accessibility/**
docs/ux/0B-accessibility-review.md
```

## Implementation guidance

Enabling `wcag22aa` is what brings `target-size` into scope — it is not in the
default tag set, and 0B-AC-031 names the criterion explicitly.

Creating `docs/ux/` is justified here: S1 R26 says it is created by the change
recording the first real UX decision, and a recorded accessibility review is
exactly that.

**Label every review row honestly.** A row is `inspection` or `targeted
assertion`; none is described as an automated result (S5 R22). This is the
requirement most easily broken by well-meant summarising.

**Scope the claim.** Evidence covers only the surfaces 0B delivers — shell,
primitives, state classes. The primary Ask, evidence and source workflows AT-017
names do not exist, and **no evidence here may be presented as discharging
AT-017** (S5 R23; S2 §9). State that in the document.

## Validation

```
pnpm test:e2e --project=chromium-desktop -- tests/e2e/accessibility
```

## Required negative tests

- Lower a status token's contrast below threshold → the axe pass **fails**;
  restore and confirm green.
- Remove a control's accessible name → fails; restore.
- Shrink an interactive control below the target-size criterion with no
  applicable exception → fails; restore.

## Evidence to leave behind

`a11y-report.json`; the committed recorded review with every row labelled.

## Stop conditions

- If a criterion can only be made green by disabling a rule, stop and fix the
  surface.
- If a criterion cannot be decided, record it as a review row — **never fabricate
  a numeric or automated metric for it**, and never express an aesthetic or
  design-quality judgement as a test threshold (S5 R22).

## What must NOT be implemented

- **No layout-dependent axe rule enabled in jsdom** (T10's boundary).
- **No fabricated metric** and no aesthetic judgement expressed as a threshold.
- **No claim that AT-017 is discharged** (S5 R23).
- No accessibility evidence for surfaces 0B does not deliver.
- No coverage or score threshold — no source establishes one.
- No second accessibility engine alongside axe-core.

## Completion definition

The axe pass runs on Chromium with `wcag22aa` enabled, blocks, and is green; all
three negative cases have been demonstrated failing and restored; the recorded
review is committed with every row labelled by method; the document states that
AT-017 is not discharged.
