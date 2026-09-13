# M00-T03 — Configuration package

- **Status:** Completed
- **Depends on:** T01
- **Blocks:** T02b, T04, T05, T06, T07
- **Plan:** [M00-PLAN-backend-foundation](../M00-PLAN-backend-foundation.md) §3.5, T3

Establishes the **canonical** KnowHub configuration mechanism. Every later
task that needs a configured value consumes this and defines no parser of its
own.

## Owning specification requirements

M00-SPEC-002 R1–R22 inclusive, including R2.1 and R2.2, **except R18**,
which is completed by T02b: `.env.example` is T02b's file, so T03 cannot
satisfy it. `policies.py` is deferred — see Out of scope.

M00-SPEC-001 R4, R5, R6, R7 as they apply to `knowhub.config`.

## Acceptance criteria contributed to

M00-AC-009, M00-AC-010, M00-AC-011, M00-AC-013 in full.

**M00-AC-012 in part.** T03 completes its three code-side clauses — no secret
held as an ordinary setting, a plain-text secret assigned to a secret
reference fails at load, and no settings representation renders a resolved
secret. The fourth clause, `.env.example` containing placeholders only, is
completed by T02b together with SPEC-002 R18.

## Files created or modified

| Path | Action |
|---|---|
| `src/knowhub/config/__init__.py` | Create |
| `src/knowhub/config/settings.py` | Create |
| `src/knowhub/config/secrets.py` | Create |
| `src/knowhub/config/feature_flags.py` | Create |

## Implementation steps

1. Root `Settings` on `pydantic-settings` v2 with `env_prefix="KNOWHUB_"`,
   `env_nested_delimiter="__"` and `extra="forbid"`, **plus** an explicit
   validation of the KnowHub-owned namespace before settings are accepted:
   - keep `extra="forbid"` — it catches an unknown key inside a *known*
     section;
   - additionally scan the `KNOWHUB_` namespace and reject any key the model
     does not recognise;
   - ignore everything outside that namespace, so unrelated host, container,
     orchestrator and CI variables never block startup;
   - leave the `KNOWHUB_FEATURE_` namespace to its owning mechanism in
     `feature_flags.py`.

   > **Factual correction, found during implementation.** This step previously
   > relied on `extra="forbid"` alone. That is insufficient: pydantic-settings
   > filters environment variables matching no known field *before* model
   > validation, so an unrecognised **top-level** key such as
   > `KNOWHUB_DATABSE__HOST` is silently discarded and never reaches
   > `extra="forbid"`. Measured: `KNOWHUB_NONSENSE`, `KNOWHUB_DATABSE__HOST`
   > and `KNOWHUB_A__B__C` were all accepted; only `KNOWHUB_DATABASE__NOPE`
   > (unknown key in a known section) was rejected. M00-SPEC-002 R2.1 and R2.2
   > are unchanged — only the mechanism needed to be stronger than this task
   > assumed.
2. Populate the **Platform** domain only: service identity and environment
   name, the dependency endpoints M00 uses, and telemetry settings —
   exporter `otlp` / `console` / `none`, plus an OTLP endpoint.
3. Database settings as discrete fields — host, port, name, user and a
   password **reference** — composed into a DSN internally. Never a
   credential-bearing connection string as one setting.
4. `secrets.py`: a `SecretRef` type carrying `scheme:identifier` with an
   `env:` scheme at M00, a `SecretResolver` protocol, and an env-backed
   resolver. A raw value assigned to a `SecretRef` field fails the grammar at
   load. Wrap resolved values in `SecretStr`.
5. Give every secret-bearing type a redacting representation so `repr`,
   `str`, f-strings, structured logging and exception rendering are safe by
   construction.
6. Constrain at the type — bounded integers, enumerated choices, URLs as
   URLs. Defaults live on the setting, and the shipped default is the safe
   value.
7. Express the full precedence ordering so later scopes attach without
   redesign, even though only the first two levels have content at M00.
8. `feature_flags.py`: a typed mechanism with a typed accessor and **zero
   flags defined**.
9. Configuration failure messages name the setting and the fault and nothing
   else.

## Tests and evidence

Unit tests:

- Ill-typed value rejected at load.
- Unknown `KNOWHUB_*` key rejected at load.
- Unrelated OS, container, orchestrator and CI environment variables present
  in the process do **not** prevent startup.
- Missing required setting fails startup.
- Failure message names the setting and leaks nothing else.
- Environment value overrides a secure platform default.
- A narrower scope cannot weaken a control marked mandatory.
- Raw plain-text secret assigned to a `SecretRef` fails at load.
- `repr`, `str` and JSON of a settings object render no resolved secret.
- Local profile is selectable by configuration alone.

## Failure paths to exercise

All rejection and startup-failure cases above. These are the failure paths
SPEC-004 R25 requires for configuration.

## Telemetry expectations

None. Configuration loads before observability is initialised, so this module
must not depend on telemetry.

## Out of scope

`src/knowhub/config/policies.py` — **deferred**; M00 has no policy content
and the no-empty-structure rule forbids an empty module. Identity, security,
model-profile, ingestion, connector and knowledge domain **content**. Any
product feature flag. Enterprise secret manager integration. Any client for a
dependency service.

## Completion checklist

- [x] Unknown `KNOWHUB_*` keys rejected; foreign environment variables ignored
- [x] Every setting typed and constrained; validated at load
- [x] Only the Platform domain populated
- [x] Secrets held as references, never as values; representations redact
- [x] `.env.example` clause of M00-AC-012 and SPEC-002 R18 left to T02b
- [x] Precedence ordering expressed in full
- [x] Feature-flag mechanism typed, zero flags defined
- [x] `policies.py` not created
- [x] No failure message leaks a value, credential or environment dump
