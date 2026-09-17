# 0B-T09 — The application shell

- **Phase:** 3
- **Depends on:** T07
- **Plan:** §11

## Objective

Build the data-agnostic application shell: its regions, its persistent-context
slots and the global command affordance — rendering correctly with nothing
supplied.

## Why this task exists

S2 R12 fixes the shell's regions and S2 R18 makes the 0B shell **data-agnostic**:
it must render with no Application record, no permission data, no freshness data,
no product navigation data, no backend call and no authenticated session. Later
milestones compose screens into this frame rather than inventing one per screen.

## Approved requirement references

S2 R12, R13, R14, R15, R16, R17, R18, R19, R20, R42, R43, R44.

## Acceptance criteria advanced or satisfied

- **0B-AC-024** — renders with nothing; workspace uses available width; empty
  slots render correctly; the top bar carries global affordances only.
- **0B-AC-025** — no fabricated data in production source; no product navigation
  entry or route target.
- **0B-AC-026** — the command affordance is present, focusable, not disabled, and
  opens a surface stating the capability is unavailable.
- **0B-AC-027** — shell navigation and controls are keyboard-operable.
- **0B-AC-035** — the tablet collapse strategy (browser half in T22).

## Prerequisites

T07.

## Exact scope

`src/components/shell/`:

1. a stable left navigation region;
2. an application-switcher **slot** within or adjacent to it;
3. a compact top bar carrying the global command/search affordance and slots for
   environment, help and identity;
4. a full-width workspace region;
5. a page-header region for title, context and actions;
6. persistent-context slots for application, scope and freshness;
7. the ⌘/Ctrl-K keyboard entry point and the affordance's surface;
8. the responsive collapse strategy: a pane that cannot remain side-by-side
   becomes a sheet or slide-over, with every control still reachable;
9. `src/app/page.tsx` updated to compose the shell as the foundation
   demonstration surface.

Every navigation, permission and context concern enters the shell as a
**presentation input supplied by the caller**. The shell supplies no values.

## Expected files

```
src/components/shell/*.tsx
src/app/page.tsx            (composes the shell)
```

## Implementation guidance

The workspace region uses the available width intelligently — enterprise tables
and wide content are not centred in a narrow column (S2 R13).

Every persistent-context slot must render correctly **while empty** (S2 R14).
Test that path first; it is the only path 0B exercises.

The command affordance is present, focusable and reachable by its keyboard entry
point, and **is not rendered in a disabled state** (S2 R15). Activating it opens
a surface stating the capability is not yet available. It performs no search and
executes no command.

The top bar carries a compact set of global affordances only — it does not become
a second navigation structure (S2 R16).

**Navigation rendering contract (S2 R17):** an item the shell is *told* is
unauthorized is **absent** from the rendered navigation — never rendered and then
visually disabled. This task owns only the rendering of that determination; the
permission semantics and their source are T19. Hiding an item is **not** a
security control.

Choose breakpoints now (the blueprint states none) and keep desktop the primary
productivity surface with the shell usable at tablet width.

## Validation

```
pnpm lint && pnpm typecheck && pnpm build
```

## Required negative tests

Authored here, executed by T10 and T22:

- render the shell with every slot empty and no session → it renders;
- a fixture reporting an item unauthorized → the item is **absent**, not disabled.

## Evidence to leave behind

The shell; the empty-slot and unauthorized-item fixtures under
`tests/fixtures/shell/`.

## Stop conditions

- If the shell cannot render without some datum, stop: S2 R18 makes
  data-agnosticism a requirement, not a preference.

## What must NOT be implemented

- **No product navigation entry and no product route target** (S2 R20).
- **No fabricated application, user, permission or navigation data in production
  source** (S2 R19) — demonstrations use test-only fixtures under `tests/`.
- No Portfolio implementation; `page.tsx` remains the foundation demonstration
  surface and says so.
- No global search and no command execution (S2 R15).
- No backend call, no session dependency, no permission logic (T19 owns the port).
- No second navigation structure in the top bar.
- No chat-style primary frame, no dashboard-card scatter, no gradient hierarchy
  (S2 R44 a, c, d).

## Completion definition

The shell renders with no Application record, no permission data, no freshness
data, no navigation data, no backend call and no session; every empty slot
renders; the command affordance is present, focusable, enabled and states its
unavailability on activation; no product navigation entry exists.
