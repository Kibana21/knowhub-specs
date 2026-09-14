# 0B-SPEC-002 — Application shell, design system and accessibility

- **Milestone:** 0B (§81 milestone 0B — Frontend foundation)
- **Repository:** `knowhub-frontend`
- **Status:** Approved
- **§81 0B items owned:** `bright design system`

## 1. Purpose

Establish the reusable visual and interaction foundation of
`knowhub-frontend` before any M1 product screen exists: the design-token
system, the reusable primitive layer, the application shell as a composition
mechanism, the common UI state classes, safe error presentation, accessibility
and responsive behaviour.

This specification delivers **mechanism and slots**. It delivers no product
functionality, consumes no backend data and depends on no authenticated
session. Later capabilities compose screens from what it establishes rather
than reinventing visual language, state handling or accessibility per screen.

## 2. Sources

**Blueprint**

- [§0E.17 Premium bright UI/UX visual specification](../../../knowhub_master_blueprint.html#runtime-architecture),
  §0E.17.1 (visual direction), §0E.17.2 (application shell), §0E.17.6
  (interaction polish), §0E.17.7 (accessibility and responsive behaviour),
  §0E.17.8 (prohibited anti-patterns), §0E.17.9 (design-system deliverables)
- [§0E.2 UI/UX implementation baseline](../../../knowhub_master_blueprint.html#runtime-architecture)
- [§0E.15 Frontend authentication, authorization & administration implementation](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**, for the navigation-rendering contract in R17 and the
  access-denied surface in R25 only
- [§61A.3 UI state classes](../../../knowhub_master_blueprint.html#frontend-file-spec)
- [§61A.4 Frontend quality gates](../../../knowhub_master_blueprint.html#frontend-file-spec)
  — accessibility and security rows
- [§65.4 Accessibility, browser and UX constraints](../../../knowhub_master_blueprint.html#non-functional-requirements)
- [§0E.1 Technology decision matrix](../../../knowhub_master_blueprint.html#runtime-architecture)
  — Design system row
- [§71.3 Cross-repository rules](../../../knowhub_master_blueprint.html#package-dependency-rules)
  — "No frontend privilege", constraining R17
- [§84.1 Mandatory working style](../../../knowhub_master_blueprint.html#coding-harness-instructions)
- [§81 Implementation dependency graph & execution order](../../../knowhub_master_blueprint.html#implementation-dependency-graph)
- [§75 Acceptance criteria & executable test catalogue](../../../knowhub_master_blueprint.html#acceptance-tests)
  — AT-017, cited as a forward target in §9

**ADRs** — referenced by number and link only; decision text lives in the
canonical files.

- **ADR-024 — Premium bright evidence-first UI design language.** Canonically
  owned by `knowhub-frontend` at
  [`docs/adr/ADR-024-premium-bright-evidence-first-ui.md`](https://github.com/Kibana21/knowhub-frontend/blob/main/docs/adr/ADR-024-premium-bright-evidence-first-ui.md),
  as recorded in the [ADR register](../../02-adrs/README.md). This
  specification is its implementation; no copy of it exists in
  `knowhub-specs`.
- [ADR-002 — Application is the primary enterprise scope](../../02-adrs/ADR-002-application-is-primary-scope.md)
  (shapes the switcher and context slots in R13 and R15; supplies no data at 0B)
- [ADR-022 — Hybrid RBAC/ABAC authorization with default deny](../../02-adrs/ADR-022-hybrid-rbac-abac-authorization-default-deny.md)
  (constrains R17 only; permission semantics and source are owned by 0B-SPEC-004)

**Dependency.** This specification builds on
[0B-SPEC-001](0B-SPEC-001-frontend-repository-toolchain-and-application-architecture.md),
which owns the repository, the App Router root layout, the
provider-composition point and the `src/components/` and `src/styles/`
directories.

## 3. Normative requirements — design tokens and theme policy

**R1.** A single design-token system is the source of visual values. Product
UI consumes approved tokens rather than ad-hoc visual values, and every token
has exactly one authoritative definition.
*(§0E.17.9 — "color/spacing/type tokens"; ADR-024)*

**R2.** The token system covers, at minimum: canvas/background, primary
surface, primary text, secondary text, accent, border, semantic status,
typography scale, spacing scale, radius, elevation, icon family, and a
tabular-numeral treatment for metrics.
*(§0E.17.1)*

**R3.** Token values follow the visual direction of §0E.17.1. That section
states its values as approximations and they are reproduced here as
approximations, not as exact fixed values:

| Token area | §0E.17.1 direction |
|---|---|
| Canvas | Bright background around `#F7F9FC` or a very light neutral; primary surfaces pure white |
| Primary text | Deep neutral around `#0F172A`/`#111827`; washed-out gray body copy avoided |
| Accent | One confident blue/azure accent around `#2563EB` for primary actions and links; an optional restrained teal secondary for knowledge/relationship states |
| Borders | Fine neutral borders around `#E5EAF1`; boundaries and whitespace preferred to heavy boxes |
| Radius | Consistent 8–12px for surfaces; pills reserved for filters and status, not every control |
| Typography | Geist/Inter or an enterprise-approved equivalent; crisp 14–16px body, 20–32px page titles, compact metadata typography, tabular numerals for metrics |
| Icons | One icon family such as Lucide; 16–20px, consistent stroke weight |
| Elevation | Very subtle shadows on floating drawers and panels only; no dashboard-tile shadow clutter |

No additional brand or accent palette is introduced, and every value stays
within this bright visual direction. Accessible semantic values required by R5
may be defined where the direction above does not already supply them; they
remain generic presentation semantics and create no business-state taxonomy.
Where the blueprint is approximate, exact resolved values are fixed by the
implementation plan within the stated direction.
*(§0E.17.1)*

**R4.** Icons are paired with text wherever their meaning is not universally
obvious.
*(§0E.17.1 icon row; §0E.17.8 — no ambiguous icon-only actions)*

**R5.** Semantic status tokens express **generic presentation semantics only**
— neutral, informational, success, warning and error. No KnowHub business-state
taxonomy — run status, source status, claim status, freshness category or
governance decision — is introduced by this specification; each belongs to the
capability specification that owns the state it describes.
*(§0E.17.1 status row; §84.1 — do not create empty abstractions ahead of the
capability)*

**R6.** The application is bright and light. Dark-mode-first styling is not
used, no user-facing theme switcher exists, and no alternate
full-application dark theme is built for V1.
*(§0E.17.1 — "Avoid a dark default theme for V1"; §0E.17.8; ADR-024)*

**R7.** One exception to R6 is permitted by §0E.17.1: a **contained** dark
code/evidence surface within the otherwise bright application. It is not built
at 0B unless real 0B content requires it, and it never becomes a second
application theme.
*(§0E.17.1 code/evidence row)*

*Implementation guidance (non-normative): how tokens are expressed — Tailwind
theme extension, CSS custom properties, a token pipeline, or the shadcn/ui
theming mechanism — and how ad-hoc values are discouraged in review or by
tooling, are implementation-plan decisions. R1 requires the single-source
outcome, not a mechanism.*

## 4. Normative requirements — reusable primitives and the application shell

### 4.1 The primitive layer

**R8.** Reusable primitives are built **before** page composition. A page is
composed from primitives; a primitive is not extracted retroactively from a
page.
*(§0E.17.9 — "Build these as reusable primitives before creating dozens of
one-off page components")*

**R9.** The primitive layer materialized at 0B is:

| Primitive | Foundation role | Source |
|---|---|---|
| Button | Primary, secondary and destructive actions, including disabled and busy treatment | §0E.17.9 |
| Input, select, textarea | Form controls the login shell and later administrative forms compose | §0E.17.9 |
| Checkbox, radio | Selection controls the scope and filter regions compose | §0E.17.9 |
| Table | The enterprise table treatment: grouping, sticky header, readable density | §0E.17.9; §0E.17.8 |
| Status badge | Generic status presentation under R5 | §0E.17.9 |
| Entity pill/tag | Compact labelled reference treatment | §0E.17.9 |
| Drawer/sheet | The side surface used by detail panes and by the responsive collapse in R33 | §0E.17.9; §0E.17.7 |
| Modal confirmation | Explicit confirmation of consequential actions | §0E.17.9; §0E.17.5 |
| Banner/alert | Persistent inline messaging near the object it concerns | §0E.17.6 |
| Skeleton, empty, error, forbidden, not-found, progress | The state-class presentations required by §5 | §0E.17.9; §61A.3 |
| Page/workspace shell | The page-level frame the shell composes | §0E.17.9 |

**R10.** No primitive is created without a foundation role. This layer is a
deliberate set, not a port of a component library.
*(§84.1 — deliver working vertical slices, not a large set of empty
interfaces)*

**R11.** The product-specific surfaces §0E.17.9 also names — evidence
citations, graph controls and the admin permission matrix — require product
semantics that 0B does not have and are **not built at 0B**. Where the shell
or a primitive must accommodate one later, it exposes a composition slot
rather than a placeholder component.
*(§0E.17.9; §84.1)*

### 4.2 The application shell

**R12.** The shell provides these regions:

- a stable left navigation region;
- an application-switcher slot within it or adjacent to it;
- a compact top bar carrying the global command/search affordance and slots
  for environment, help and identity;
- a full-width workspace region;
- a page-header region for title, context and actions.

*(§0E.17.2; §0E.2)*

**R13.** The workspace region uses the available width intelligently.
Enterprise tables, workspaces and wide content are not centred inside a narrow
column.
*(§0E.17.2 — "avoid centering enterprise tables/graphs inside a narrow blog
column")*

**R14.** The shell provides persistent-context slots so that application,
scope and freshness remain visible without reopening filters. The shell
supplies no values for them; it renders correctly when a slot is empty.
*(§0E.17.2; §0E.2;
[ADR-002](../../02-adrs/ADR-002-application-is-primary-scope.md))*

**R15.** The shell provides the global command/search **affordance** and its
keyboard entry point. The affordance is present, focusable and reachable by
that entry point; activating it opens a surface that states the capability is
not yet available. 0B implements no global search and executes no command. The
affordance is not rendered in a disabled state.
*(§0E.17.6 — ⌘/Ctrl+K global command/search; §0E.17.2)*

**R16.** The top bar carries a compact set of global affordances only. It does
not become a second navigation structure.
*(§0E.17.2 — "not a second navigation jungle")*

**R17.** **Navigation rendering contract.** A navigation item that the shell
is told is unauthorized is **absent** from the rendered navigation. It is
never rendered and then visually disabled. This specification owns only the
rendering of that determination; the permission semantics and their source are
owned by 0B-SPEC-004, and hiding an item is **not** a security control — the
backend authorizes every operation independently.
*(§0E.17.8 — "render unauthorized modules then merely disable their buttons"
is prohibited; §0E.15 — "PermissionGate is not security"; §71.3 — "No frontend
privilege";
[ADR-022](../../02-adrs/ADR-022-hybrid-rbac-abac-authorization-default-deny.md))*

### 4.3 Shell data-agnosticism

**R18.** The 0B shell is **data-agnostic**. It requires no real Application
record, no permission data, no freshness data, no product navigation data, no
backend call and no authenticated session in order to render. Each of those
concerns enters the shell as a presentation input supplied by its caller.
*(§81 — 0B precedes the milestone that delivers identity and the application
registry; §0E.8 parallelization rule)*

**R19.** No fabricated application, user, permission or navigation data exists
in the production source tree to make the shell appear populated. Any
demonstration of application, navigation or permission-dependent rendering
uses test-only fixtures or doubles that are not reachable from production
application code. Where those fixtures live is owned by 0B-SPEC-005 and the
implementation plan.
*(§84.1; §84.3 — repository hygiene; §0E.17.8 — no trust decisions from hidden
UI controls)*

**R20.** 0B introduces no product navigation entry and no product route target
in the shell.
*(§61A.1 — the route map is delivered by M1 and later; §81)*

## 5. Normative requirements — UI state classes and error presentation

**R21.** Each §61A.3 state class has a reusable presentation: loading,
skeleton, empty, partial, error, forbidden, not-found, retry, disabled action
and asynchronous progress.
*(§61A.3; §0E.17.9)*

**R22.** Where the resulting layout is known, a skeleton is used in preference
to a generic spinner.
*(§0E.17.6 — "Prefer skeletons over spinners for known layouts")*

**R23.** An empty state presents one obvious next action where an action
exists, and explains the emptiness rather than showing a bare absence.
*(§0E.17.6 — "thoughtfully written empty states with one obvious next action")*

**R24.** An important error is presented near the object that failed, with a
remediation path where one exists. Toasts carry transient confirmation, not
important errors.
*(§0E.17.6 — "Toasts are for transient confirmation; important errors live
near the failed object and include a remediation path/run ID")*

**R25.** The asynchronous-progress presentation renders an externally supplied
stage, progress value where measurable, last event and actionable error. It
derives none of them and models no run.
*(§0E.17.6; §0E.2 — long-running work shows durable run status; §65.4)*

**R26.** Error presentation covers these surfaces: the App Router error
boundary, route-level error, not-found, recoverable inline failure, and
forbidden/access-denied.
*(§61A.3; §0E.15 — a polished Access Denied experience)*

**R27.** **Safe error content.** A user-facing error presents safe language
only. It never renders raw backend exception text, a stack trace, a sensitive
implementation or diagnostic detail, sensitive or internal configuration, a
credential, a secret or a token. The frontend does not undo the backend's
safe-error model.
*(§61A.4 security row; §0E.13.9 — no secrets in telemetry or diagnostics;
§0E.17.8 — no bearer tokens, debug claims or secrets in developer UI)*

**R28.** Where a support or correlation identifier is supplied with an error,
the presentation displays it so a user can quote it. This specification defines
only its presentation; its production, propagation and the normalized error
representation are owned by 0B-SPEC-003.
*(§0E.17.6 — errors "include a remediation path/run ID")*

**R29.** Freshness, authorization problems and missing evidence are never
discoverable only on hover. Where such state is present it is visible in the
rendered layout.
*(§0E.17.6 — "Never bury freshness, authorization problems or missing evidence
behind tooltips only"; §0E.17.8)*

## 6. Normative requirements — accessibility and responsive behaviour

**R30.** **WCAG 2.2 AA** is the normative accessibility target for primary
KnowHub workflows. At 0B this binds the shell, the primitive layer and the
state-class presentations — the surfaces that exist.
*(§0E.17.7 — "Target WCAG 2.2 AA for primary workflows"; §65.4; §61A.4
accessibility row)*

**R31.** The shell's interactive navigation and controls, and every
interactive primitive, are fully operable by keyboard. Non-interactive content
— a skeleton, a status badge, a passive banner, a static state presentation —
remains semantically accessible and is not made focusable merely to appear
keyboard-reachable.
*(§0E.17.7; §65.4 — keyboard navigation for primary workflows; §0E.2)*

**R32.** Every control has a visible keyboard focus indicator, and focus order
follows the visual and logical reading order.
*(§0E.17.7 — "All controls have visible keyboard focus")*

**R33.** Every control has a meaningful accessible name, and the shell uses
semantic landmark structure.
*(§0E.17.7; §0E.2 — semantic landmarks; §61A.4 — labels)*

**R34.** Contrast meets accessible thresholds, including for secondary text
and for status colours.
*(§0E.17.7 — "Contrast meets accessible thresholds even for secondary text/
status colors")*

**R35.** State is never communicated by colour alone. Every status treatment
carries a non-colour signal — text, icon, shape or position.
*(§0E.17.1 status row; §65.4 — "Color must never be the only indicator";
§0E.2)*

**R36.** Form controls are labelled, and a validation or submission error is
programmatically associated with the control it concerns.
*(§61A.4 accessibility row — labels; §0E.17.7)*

**R37.** Interactive controls satisfy the applicable WCAG 2.2 AA target-size
criterion, including its defined exceptions. No KnowHub-specific size is
introduced; the criterion referenced by R30 governs.
*(§0E.17.7 — "minimum practical hit targets"; R30)*

**R38.** `prefers-reduced-motion` is respected.
*(§0E.17.7 — "Respect prefers-reduced-motion")*

**R39.** Where a visual introduced by this specification conveys meaning, a
meaningful text alternative is provided.
*(§0E.2 — text alternatives for diagrams; §0E.17.7)*

**R40.** Focus behaviour follows interaction **modality**, not component
name. A modal surface — a modal dialog, or a sheet or drawer presented modally
— receives initial focus on open, contains focus while it is modal, restores
focus to the invoking control on close, and is dismissible by keyboard wherever
dismissal is permitted at all. A non-modal drawer or pane remains keyboard
reachable and operable in the normal focus order and does **not** trap focus
merely because it is a drawer.
*(§0E.17.7 — visible focus and keyboard operability; §0E.17.9 — drawers and
modal confirmation as design-system deliverables)*

**R41.** Transitions for drawers, filters and context changes are brief —
§0E.17.6 states 120–180ms — and no decorative animation runs while a user is
reading.
*(§0E.17.6)*

**R42.** Desktop is the primary productivity surface, and the shell remains
usable at tablet width.
*(§0E.17.7; §0E.2 — responsive philosophy)*

**R43.** Navigation and side regions have a defined collapse strategy. A pane
that cannot remain side-by-side becomes a sheet or slide-over surface, and
every control remains reachable and operable after collapse.
*(§0E.17.7 — "three-pane Ask collapses evidence into a slide-over/drawer at
narrower widths")*

*Implementation guidance (non-normative): the blueprint states no breakpoint
values, so none are fixed here. Breakpoint selection and its Tailwind
expression belong to the implementation plan and may change without amending
this specification, so long as R42 and R43 hold.*

## 7. Normative requirements — prohibited patterns

**R44.** The following are prohibited. Each is stated so that a reviewer can
decide it by inspection rather than by taste.

| # | Prohibition |
|---|---|
| a | The product is not presented as a chat interface with a sidebar: no conversational surface is the application's primary frame, and answers are not rendered as oversized chat bubbles |
| b | No dark-mode-first styling and no alternate full-application dark theme (R6) |
| c | No page is composed as a scatter of unrelated dashboard cards; a page presents content organised for one task |
| d | Gradients and multi-hue decorative effects are not used to create hierarchy; hierarchy comes from typography, spacing, structure and restrained accent use |
| e | No critical state — freshness, authorization problems, missing evidence, errors — is discoverable only on hover (R29) |
| f | No administrative, consequential or destructive action is presented as an ambiguous icon-only control. Such an action carries a visible text label, or is otherwise visually unambiguous; an accessible name alone does not satisfy this (R4) |
| g | An unauthorized navigation item is absent, never rendered-and-disabled (R17) |
| h | Tables are not made unreadably dense: grouping, sticky headers and filtering are available where a table carries volume (R9) |
| i | No bearer token, debug claim, credential, secret or sensitive diagnostic or internal value appears in any rendered surface. An identifier that its owning capability explicitly designates as user-visible — a support, correlation or run identifier — is permitted; this specification renders such an identifier when supplied and decides none of them (R27, R28) |

*(§0E.17.8; §0E.17.3; §0E.17.6)*

## 8. Forward constraints recorded, not implemented at 0B-SPEC-002

- **Permission semantics and their source.** R17 owns only the rendering of an
  authorization determination. The permissions port, route guards and the
  statement that the backend authorizes independently are owned by
  **0B-SPEC-004**. *(§0E.15; ADR-022)*
- **Normalized API errors.** R27 and R28 own presentation. The safe normalized
  error representation, its correlation identifier and its production are owned
  by **0B-SPEC-003**. This specification defines no API error schema.
  *(§0E.8; ADR-017)*
- **Automated accessibility checks and every other gate.** R30–R43 state
  requirements. The accessibility test layer, its tooling and its CI wiring are
  owned by **0B-SPEC-005**. Verification by 0B-SPEC-005 does not transfer
  normative ownership of any requirement in this specification.
  *(§0E.8 frontend CI gate list; §61A.4)*
- **Evidence, graph and traceability presentation.** §0E.17.3 and §0E.17.4
  describe the Ask workspace, graph, traceability and impact surfaces, and
  §0E.17.4 requires every visualization to have an accessible table or list
  equivalent. Those surfaces are delivered by M1 and later; R39 binds only
  visuals this specification introduces. §0E.17.8's prohibition on a
  visualization animating continuously while a user reads it is likewise
  deferred: no visualization exists at 0B, so the prohibition is not
  objectively reviewable here and belongs to the capability that introduces
  one. R41 already bars decorative animation during reading on the surfaces
  0B does deliver. *(§0E.17.3; §0E.17.4; §0E.17.8; §81)*
- **Administration UX.** §0E.17.5 describes user, role, group-mapping and
  session administration screens. All are delivered by M1 and later.
  *(§0E.17.5; §80 identity row)*
- **Performance treatment.** §61A.4's performance row — virtualizing large
  evidence, search and graph lists — presupposes volume that does not exist at
  0B. *(§61A.4; §81)*

## 9. Acceptance criteria

| ID | Criterion |
|---|---|
| **0B-AC-020** | Every token has exactly one authoritative definition, and changing a token's value changes every primitive that consumes it. *(R1, R2)* |
| **0B-AC-021** | A representative sample of primitives renders with computed values derived from tokens rather than from literal visual values, and the token set covers the areas in R2 within the §0E.17.1 direction; no additional brand or accent palette has been introduced; and the status tokens express only generic presentation semantics, introducing no run, source, claim, freshness or governance taxonomy. *(R1, R2, R3, R5)* |
| **0B-AC-022** | No full-application dark theme and no user-facing theme switcher exists, and no component path selects an alternate application theme; and no contained dark code/evidence surface has been built. *(R6, R7)* |
| **0B-AC-023** | Every primitive in R9 renders in each state it declares; no product-specific surface named in R11 has been built, and where one is anticipated the shell or primitive exposes a slot rather than a placeholder component. *(R8, R9, R10, R11)* |
| **0B-AC-024** | The shell renders with no Application record, no permission data, no freshness data, no product navigation data, no backend call and no authenticated session; the workspace region uses the available width; every persistent-context slot renders correctly while empty; and the top bar carries global affordances only rather than a second navigation structure. *(R12, R13, R14, R16, R18)* |
| **0B-AC-025** | The production source tree contains no fabricated application, user, permission or navigation data, and no product navigation entry or product route target; any such data used to demonstrate the shell is test-only and unreachable from production application code. *(R19, R20)* |
| **0B-AC-026** | The global command/search affordance is present, focusable and reachable by its keyboard entry point, is not rendered disabled, and on activation opens a surface stating the capability is not yet available while performing no search and executing no command. *(R15)* |
| **0B-AC-027** | The shell's interactive navigation and controls and every interactive primitive are fully operable by keyboard; every control that receives focus shows a visible focus indicator; focus order follows the logical reading order; and non-interactive presentation content has not been made focusable. *(R31, R32)* |
| **0B-AC-028** | Given a test double reporting an item as unauthorized, that item is absent from the rendered navigation rather than rendered and disabled. *(R17)* |
| **0B-AC-029** | Every state class in R21 has a representative rendered state; a known-layout loading case renders a skeleton rather than a generic spinner; and an empty state presents one next action where an action exists; and the asynchronous-progress presentation renders externally supplied stage, progress and error values and derives none of them. *(R21, R22, R23, R25)* |
| **0B-AC-030** | Given an error carrying backend exception text, a stack trace, a credential or a sensitive internal value, the rendered output contains none of them; a supplied support or correlation identifier is displayed; and an important error renders near the failed object rather than only as a toast. *(R24, R26, R27, R28)* |
| **0B-AC-031** | Automated WCAG-oriented checks over the shell, the primitive layer and the state-class presentations, together with targeted assertions and inspection where automation cannot decide a criterion, collectively demonstrate that: applicable contrast thresholds are met including secondary and status text; every control carries a meaningful accessible name; form and other controls are labelled; a validation or submission error is programmatically associated with the control it concerns; interactive controls satisfy the applicable WCAG 2.2 AA target-size criterion or a defined exception; every visual that conveys meaning carries a text alternative; and the shell uses semantic landmark structure. *(R30, R33, R34, R36, R37, R39)* |
| **0B-AC-032** | Every status treatment is distinguishable without colour, and no state is signalled by colour alone. *(R35)* |
| **0B-AC-033** | Transitions for drawers, filters and context changes follow the brief duration direction of R41; with `prefers-reduced-motion` set, motion is reduced or removed rather than merely shortened; and no decorative animation runs while content is being read. *(R38, R41)* |
| **0B-AC-034** | A modal surface receives initial focus on open, contains focus while modal, restores focus to the invoking control on close, and is dismissible by keyboard wherever dismissal is permitted; and a non-modal drawer or pane remains keyboard reachable in the normal focus order without trapping focus. *(R40)* |
| **0B-AC-035** | At tablet width the shell remains usable: the collapse strategy operates, a pane that cannot remain side-by-side becomes a sheet or slide-over, and every control remains reachable and operable. *(R42, R43)* |
| **0B-AC-036** | A review against the R44 checklist finds no prohibited pattern in the shell, the primitive layer or the state-class presentations; and no critical state is discoverable only on hover; and no administrative, consequential or destructive action is presented as an ambiguous icon-only control without a visible text label. *(R4, R29, R44)* |

**Verification dependency.** 0B-AC-020, 0B-AC-021, 0B-AC-027 and 0B-AC-031
require the component and accessibility test layers, and 0B-AC-024, 0B-AC-028,
0B-AC-033, 0B-AC-034 and 0B-AC-035 require a rendering or browser harness.
Those layers and their CI wiring are owned by **0B-SPEC-005**; the requirements
they check remain owned by this specification. This is a verification
dependency at milestone level, not a transfer of ownership.

**Forward catalogue target.** [§75](../../../knowhub_master_blueprint.html#acceptance-tests)
AT-017 requires primary Ask, evidence and source workflows to pass automated
accessibility checks and manual keyboard criteria. None of those workflows
exists at 0B. 0B-AC-027 and 0B-AC-031 evidence AT-017 **only** for the shell,
the primitives and the state classes; AT-017 is satisfied by the milestone that
delivers the workflows it names.

## 10. Deferred

| Deferred from 0B-SPEC-002 | Owner |
|---|---|
| Permission semantics, permissions source, route guards, access-denied behaviour beyond its presentation | 0B-SPEC-004 |
| Normalized API error representation, correlation-identifier production and propagation | 0B-SPEC-003 |
| Accessibility test layer, component test layer, browser harness and all CI wiring | 0B-SPEC-005 |
| Real application-switcher data, real navigation modules, real permission data | M1 |
| Login, authentication, identity-provider UI, user and role management, administration screens | M1 |
| Ask/enquiry workspace, evidence viewers, evidence citation components | M1 and later |
| Graph, traceability and impact presentation; Cytoscape, React Flow, Mermaid, Monaco | M9 and later |
| Jira, document upload and structured-parsing screens | M10 |
| The contained dark code/evidence surface permitted by R7 | The capability that introduces real code or evidence content |
| Virtualization and other volume-driven performance treatment | M9 and later |
| Browser telemetry SDK, vendor or UX-metrics framework | Deferred; a future ADR when a concrete requirement and technology choice exist |
