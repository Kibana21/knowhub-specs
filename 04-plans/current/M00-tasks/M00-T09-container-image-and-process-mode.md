# M00-T09 — Container image and process mode

- **Status:** Completed
- **Depends on:** T05
- **Blocks:** T10
- **Plan:** [M00-PLAN-backend-foundation](../M00-PLAN-backend-foundation.md) §3.1, §3.7, T9

## Owning specification requirements

M00-SPEC-004 R17, R18, R19, R20, R21

## Acceptance criteria contributed to

M00-AC-025

## Files created or modified

| Path | Action |
|---|---|
| `Dockerfile` | Create |
| `docker-entrypoint.sh` | Create |

## Implementation steps

1. Two-stage build: `ghcr.io/astral-sh/uv:python3.12-bookworm-slim` as
   builder, `python:3.12-slim-bookworm` as runtime. Pin both by tag **and**
   SHA256 digest.
2. Install from the committed lockfile — the image must not resolve
   dependencies afresh.
3. Produce **one** image per commit, startable in different modes.
4. `docker-entrypoint.sh` dispatches on `$1`: `api` starts uvicorn. Any other
   value exits non-zero with `unknown process mode: <x>`.
5. **No `worker` or `scheduler` branch exists.** Adding one later is a new
   command backed by real implementation, not a change to the image contract.
   Taskiq worker and scheduler implementations are M2.
6. Run as a non-root user.
7. The running image serves the T05 liveness and readiness endpoints and
   emits telemetry per the T04 baseline.
8. No infrastructure-as-code or deployed-environment definition. `deploy/`
   is where those will live; its contents are later work.

## Tests and evidence

- The image builds.
- Started in `api` mode, it serves `/healthz` and `/readyz`.
- A request against the containerised process emits a span.
- An unknown process mode exits non-zero with the stated message.
- A repository search confirms no worker or scheduler implementation exists.

## Failure paths to exercise

Unknown process mode exits non-zero. A container started with a required
setting missing fails at startup with a safe message.

## Telemetry expectations

The containerised process emits spans identically to the local process. No
separate telemetry path for containers.

## Out of scope

Any worker or scheduler mode, stub or no-op. `deploy/` contents, Terraform,
Bicep, Container Apps or AKS topology. Image publication to a registry.
SBOM generation. Multi-architecture build matrices.

## Completion checklist

- [x] One image, two-stage, both bases pinned by tag and digest
- [x] Installs from the committed lockfile
- [x] Entrypoint accepts a process-mode selection
- [x] `api` is the only implemented mode; unknown modes exit non-zero
- [x] No placeholder worker or scheduler anywhere in the repository
- [x] Runs as non-root
- [x] Serves both probes and emits telemetry
