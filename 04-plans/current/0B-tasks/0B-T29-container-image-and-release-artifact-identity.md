# 0B-T29 — Container image and release-artifact identity

- **Phase:** 8 — Container and supply chain
- **Depends on:** T28
- **Plan:** §22

## Objective

Produce the production `knowhub-frontend` container image from the repository,
bound to the repository name and the exact commit.

## Why this task exists

§0E.1's Packaging row and §0E.8 make the Next.js image the frontend's primary
release artifact, and S5 R66 makes its build a blocking gate. Building a release
artifact is **not** designing a deployment platform, and this task is scoped
accordingly.

## Approved requirement references

S5 R66, R67, R69, R70.

## Acceptance criteria advanced or satisfied

- **0B-AC-114** — the image builds as a blocking gate; identity binds repository
  and commit; no deployment work; no backend container detail imported.

## Prerequisites

T28 — the production build and its inspection gate.

## Exact scope

1. `Dockerfile` — three stages on `node:24-bookworm-slim` pinned by digest:
   - `deps`: `corepack enable`; `pnpm install --frozen-lockfile`;
   - `build`: `pnpm build` with `output: 'standalone'`;
   - `runtime`: copy `.next/standalone` and `.next/static`; `USER 10001:10001`;
     `EXPOSE 3000`; `CMD ["node","server.js"]`.
2. `.dockerignore` — excludes `tests/`, `.next`, `.git`, `node_modules`,
   `playwright-report`, `test-results`.
3. The image tagged `knowhub-frontend:<git-sha>`.

## Expected files

```
Dockerfile  .dockerignore
```

## Implementation guidance

**Every choice here is an implementation choice, not a specification
requirement**, and must be labelled as such in the Dockerfile's comments. S5 R70
is explicit: a non-root user identifier, a health check, a multi-stage structure,
image labels and an exposed-port choice are **not fixed by any source**, and
`knowhub-backend`'s image decisions bind only `knowhub-backend`. Justify each on
its own merits:

- digest-pinned base matching the declared Node baseline → reproducibility;
- three stages → build tooling never reaches the runtime layer, which is also what
  keeps S5 R65 easy to hold;
- `output: 'standalone'` → Next's own minimal server, no wholesale
  `node_modules`;
- non-root `10001` → least privilege at no cost;
- `EXPOSE 3000` → Next's default; no port policy is asserted.

**Add no `HEALTHCHECK` and no labels.** No source requires either, and importing
them from the backend image is exactly what S5 R70 forbids. Add them when a
deployment target asks for them.

`.dockerignore` excluding `tests/` is a containment aid as well as a speed one.

## Validation

```
docker build -t knowhub-frontend:$(git rev-parse HEAD) .
docker run --rm -p 3000:3000 knowhub-frontend:$(git rev-parse HEAD)
# confirm the root layout is served and the runtime user is not root
```

## Required negative tests

- Confirm `tests/` is absent from the image filesystem.
- Confirm the runtime user is not UID 0.

## Evidence to leave behind

A successful image build tagged `knowhub-frontend:<git-sha>`; the runtime-user and
absent-tests confirmations.

## Stop conditions

- If the image cannot start without a live backend, a real identity provider or a
  product contract, stop: that contradicts S5 R64 and the repository-independence
  requirement.

## What must NOT be implemented

- **No registry publishing and no push step** (S5 R69).
- **No `deploy/` content, deployment definition, Kubernetes or ingress
  configuration, cloud runtime design, Terraform or Bicep, or environment
  promotion** (S5 R69).
- No production identity-provider registration; no TLS, CDN or WAF design.
- **No backend container implementation detail imported** — and no claim that
  non-root UID, health check, multi-stage structure, labels or exposed port are
  specification requirements (S5 R70).
- No further tag scheme beyond `knowhub-frontend:<git-sha>` (S5 R67).
- No container scanning (T30).

## Completion definition

The image builds from the repository as a blocking gate, is tagged
`knowhub-frontend:<git-sha>`, starts and serves without a backend, runs as a
non-root user, contains no `tests/`, and every Docker choice is labelled an
implementation choice in the file.
