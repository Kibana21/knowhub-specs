# 0B-T07 — The primitive layer

- **Phase:** 3
- **Depends on:** T06
- **Plan:** §11

## Objective

Build exactly the S2 R9 primitive set, each rendering every state it declares,
consuming tokens rather than literal visual values.

## Why this task exists

S2 R8 is explicit: a page is composed from primitives, and a primitive is never
extracted retroactively from a page. The shell (T09), the state classes (T08) and
the access components (T19) all compose this layer, so it comes first.

## Approved requirement references

S2 R4, R8, R9, R10, R11, R31, R32, R33, R35, R36, R37, R40, R41, R44.

## Acceptance criteria advanced or satisfied

- **0B-AC-023** — every R9 primitive renders each declared state; no
  product-specific surface built; slots rather than placeholders.
- **0B-AC-034** — modal focus behaviour (component half; browser half in T22).

## Prerequisites

T06.

## Exact scope

Exactly the S2 R9 set under `src/components/ui/`:

button · input · select · textarea · checkbox · radio group · table ·
status badge · entity pill · drawer/sheet · modal confirmation · banner/alert ·
skeleton · progress · page/workspace shell frame.

(The empty, error, forbidden, not-found and loading **state presentations** are
T08; this task builds the primitives they compose.)

Each primitive: token-derived styling only; a visible keyboard focus indicator
from the single focus token; a meaningful accessible name; where interactive,
full keyboard operability; where it carries status, a non-colour signal.

## Expected files

```
src/components/ui/*.tsx
```

## Implementation guidance

Use Radix for `dialog`, `select`, `checkbox`, `radio-group`, `slot` and `label`:
focus management, dismissal and ARIA semantics are exactly what R31–R40 require
and hand-rolling them invites accessibility defects. Use CVA for variants and
`cn` for class merging.

shadcn/ui is used **CLI-assisted, vendored-source**: scaffold locally, then own
the committed output as first-party source. `shadcn` is not a runtime dependency.
Trim anything scaffolded that no R9 role requires — S2 R10 forbids a primitive
without a foundation role, and this layer is a deliberate set, not a port of a
component library.

**Focus behaviour follows modality, not component name** (S2 R40). A modal
surface receives initial focus, contains focus while modal, restores focus to the
invoking control on close and is keyboard-dismissible. A **non-modal** drawer
remains in the normal focus order and does **not** trap focus merely because it
is a drawer. Build both behaviours into the drawer/sheet primitive explicitly.

Non-interactive content — skeletons, passive banners, status badges — stays
semantically accessible and is **not** made focusable to appear keyboard-reachable
(S2 R31).

Record every place a Radix primitive sets an inline `style` **attribute**; T27's
CSP depends on that inventory.

## Validation

```
pnpm lint && pnpm typecheck && pnpm build
```

## Required negative tests

Deferred to T10 (component) and T22 (browser); this task supplies the surfaces
they exercise.

## Evidence to leave behind

The primitive set, plus the inline-style inventory note for T27.

## Stop conditions

- If a primitive seems to need a role not in R9, stop: S2 R10 forbids creating
  one without a foundation role.
- If an inline `<style>` element (not attribute) proves unavoidable, record it and
  surface it — T27's CSP must not be widened silently (plan §30 risk 2).

## What must NOT be implemented

- **No product-specific surface**: no evidence citation component, no graph
  control, no admin permission matrix (S2 R11). Where one is anticipated, expose
  a composition slot, never a placeholder component.
- No primitive outside the R9 set.
- No dark code/evidence surface.
- No ambiguous icon-only administrative, consequential or destructive action —
  such an action carries a visible text label; an accessible name alone does not
  satisfy S2 R44 f.
- No shell composition (T09), no state presentations (T08), no access components
  (T19).
- No fabricated data of any kind.

## Completion definition

Every R9 primitive exists with its declared states, derives styling from tokens,
is keyboard-operable where interactive, carries a visible focus indicator and an
accessible name; modal and non-modal focus behaviour differ correctly; no
product-specific surface has been built.
