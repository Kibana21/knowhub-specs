# 0B-T27 — Production CSP and security headers

- **Phase:** 7 — Production hardening and build-artifact verification
- **Depends on:** T09
- **Plan:** §20

## Objective

Emit the application-wide production HTTP security-header baseline, including a
restrictive CSP, and verify it against the production artifact.

## Why this task exists

0B-SPEC-004 §1.1, §14 and §16 **delegate** this subject matter and ownership of
AT-035 to 0B-SPEC-005. AT-035 is the one §75 catalogue entry 0B fully evidences,
so the gate must be real and must be verified against the production runtime.

## Approved requirement references

S5 R39, R40, R41, R42, R43, R44, R45, R46, R47, R48, R49.

## Acceptance criteria advanced or satisfied

- **0B-AC-107** — restrictive CSP: `object-src 'none'`, no unsafe eval, no
  arbitrary third-party script source, controlled `connect-src`.
- **0B-AC-108** — HSTS, nosniff, strict referrer policy, `Permissions-Policy`,
  anti-clickjacking; no control the sources do not establish; scope recorded.

## Prerequisites

T09 — the shell exists, so the CSP can be validated against real rendering.

## Exact scope

1. `src/proxy.ts` — the **Next 16 Proxy** convention (the renamed `middleware`
   entry; Next 16 warns when migrating `middleware` → `proxy` without renaming
   the exported function). Emits the per-request nonce and the full baseline
   under matcher `/((?!_next/static|_next/image|favicon.ico).*)`. The Proxy is a
   request and header boundary **only** — never the authentication or
   authorization guard, which stays at the server boundary (T19).
2. `next.config.ts` — HSTS and `nosniff` only, for `/_next/static/:path*`.
3. The CSP of plan §20.1 and the header values of plan §20.2.
4. `scripts/verify-security-headers.mjs` — starts the **production** build,
   fetches a document response and a static asset, asserts every header and CSP
   directive, and asserts the negatives.
5. **Executed production-runtime dynamic-rendering verification**, asserting
   that: a document protected by the nonce CSP is **dynamically rendered**; the
   nonce differs between two requests for the same document; every framework and
   page `<script>` carries that request's nonce; **no static-rendered document is
   served under the nonce CSP**; and the policy still contains no
   `'unsafe-eval'`.
6. `tests/e2e/headers/` — the same assertions exercised through the browser.
7. A `security-headers-report.json` output for CI publication.

## Expected files

```
src/proxy.ts
next.config.ts                      (extended)
scripts/verify-security-headers.mjs
tests/e2e/headers/**
```

## Implementation guidance

**Verify nonce propagation before wiring the gate** (plan §30 risk 1). Next
propagates a nonce found in the CSP header to its own scripts; confirm that on
this exact version rather than assuming it.

**The nonce strategy requires dynamic rendering.** A per-request nonce can only
reach framework and page scripts if the document response is dynamically
rendered: a statically rendered document is produced at build time, before any
nonce exists, and would fall outside the nonce model while still appearing to
pass a header-only check. This is an implementation consequence of the CSP
strategy selected in plan §20, not a new product requirement. Make it explicit
in the surface's own code, and assert it at runtime rather than assuming the
framework's default.

**`style-src-attr 'unsafe-inline'` is necessary and permitted.** Radix positions
floating surfaces with inline `style` **attributes**, which `style-src 'self'`
alone blocks. S5 R40 restricts unsafe *eval* and third-party *script*, not style
attributes; scoping to `style-src-attr` keeps inline `<style>` elements blocked.
Use the inline-style inventory T07 recorded. Record this reasoning in the
Proxy module's comment.

**`connect-src 'self'` and nothing else.** S5 R41 is emphatic: the directive must
not create or imply direct browser → KnowHub-backend access, must not expose a
server-only backend origin merely to populate it, and **no backend origin is
added automatically**. Where configuration participates, only values explicitly
permitted through the browser-exposable boundary may do so.

**The strict policy binds the production runtime.** `next dev` needs
`'unsafe-eval'`; S5 R39 scopes the baseline to production, so apply it there and
do not attempt a single policy for both.

Each header value is justified in plan §20.2 — carry that reasoning into the code
comments, particularly the omission of HSTS `preload` (a deployment decision) and
the choice of `no-referrer`.

Header placement is split so **no path receives a header from both sources**,
avoiding duplicate or conflicting values.

## Validation

```
pnpm build && pnpm verify:headers
pnpm test:e2e -- tests/e2e/headers
```

## Required negative tests

Each demonstrated failing and restored:

- remove `object-src 'none'` → the gate fails;
- add `'unsafe-eval'` → fails;
- add a third-party script source → fails;
- widen `connect-src` beyond `'self'` → fails;
- remove HSTS, `nosniff`, the referrer policy, the `Permissions-Policy` or the
  anti-clickjacking control → each fails;
- force a document under the nonce CSP to render statically → the
  dynamic-rendering assertion **fails**; restore and confirm green.

## Evidence to leave behind

`security-headers-report.json`; transcripts of each weakening demonstrated
failing; the recorded scope statement.

## Stop conditions

- **If a CSP that Next, Radix and Tailwind can actually run cannot meet S5 R40
  without `'unsafe-eval'`, STOP and surface it.** Never weaken the CSP to make the
  application run.
- If inline `<style>` elements prove unavoidable, surface the trade-off rather
  than widening `style-src` silently.

## What must NOT be implemented

- **No COOP, COEP, CORP, Subresource Integrity, CSP reporting endpoint or CORS
  policy** — S5 R47 forbids introducing a control the sources do not establish.
- **No backend origin in `connect-src`**, automatically or otherwise; no
  server-only origin exposed (S5 R41).
- No direct browser → KnowHub-backend connectivity.
- No `'unsafe-eval'` in production.
- No claim that deployment-platform configuration is demonstrated — the scope
  statement of S5 R49 accompanies the evidence.
- **No security decision in the Proxy**: it is a request and header boundary
  only, and never the authentication or authorization guard, which stays at the
  server boundary (T19).
- No `middleware.ts`: Next 16's convention is `proxy.ts`.

## Completion definition

The production runtime emits the full baseline from `src/proxy.ts`; every
directive and header is asserted; the executed dynamic-rendering verification
passes — documents under the nonce CSP are dynamically rendered, scripts carry
the per-request nonce, no static-rendered document bypasses the model, and no
`'unsafe-eval'` is present; every weakening has been demonstrated failing and
restored; the S5 R49 scope statement accompanies the evidence; no forbidden
control has been added.
