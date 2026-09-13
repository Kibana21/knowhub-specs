# M00-SPEC-002 — Backend configuration and settings

- **Milestone:** M00 (§81 milestone 0A — Backend foundation)
- **Repository:** `knowhub-backend`
- **Status:** Approved
- **§81 0A item owned:** `settings`

## 1. Purpose

Define the configuration mechanism the backend uses from M00 onward: typed
settings loading, the configuration domains, the precedence chain, how
secrets are referenced rather than stored, and the Local environment
profile.

M00 establishes the mechanism and the rules. It populates only the
configuration that M00 itself needs.

## 2. Sources

**Blueprint**

- [§77 Configuration model & feature flags](../../../knowhub_master_blueprint.html#configuration-model),
  §77.1 (configuration domains) and §77.2 (precedence)
- [§76 Environment & deployment matrix](../../../knowhub_master_blueprint.html#environment-matrix)
  — the Local column — and §76.1 (promotion rules)
- [§0E.1 Technology decision matrix](../../../knowhub_master_blueprint.html#runtime-architecture)
  — Workload identity/secrets row
- [§0E.9 Explicit non-choices for V1](../../../knowhub_master_blueprint.html#runtime-architecture)
  — no model-specific business logic
- [§65.3 Security, privacy and compliance constraints](../../../knowhub_master_blueprint.html#non-functional-requirements)
  — no secret persistence in configuration
- [§61](../../../knowhub_master_blueprint.html#python-file-spec) —
  `config/settings.py`, `config/feature_flags.py`, `config/policies.py`
- [§84.3](../../../knowhub_master_blueprint.html#coding-harness-instructions)
  — `.env.example`; nothing sensitive committed

**ADRs**

- [ADR-019 — Model routing uses logical profiles](../../02-adrs/README.md)
  (canonical file in `knowhub-backend/docs/adr/`) — the reason a model
  profile is configuration rather than code
- [ADR-021](../../02-adrs/ADR-021-provider-pluggable-authentication.md),
  [ADR-022](../../02-adrs/ADR-022-hybrid-rbac-abac-authorization-default-deny.md)
  — the reason identity and security policy are configuration domains;
  their content is M1 work

## 3. Normative requirements — the settings mechanism

**R1.** Configuration lives in the `knowhub.config` package, structured as
settings, feature flags and policies.
*(§61; §77)*

**R2.** All KnowHub configuration is **typed and validated at load**.
Values arrive as declared types with explicit constraints; an invalid
value, or an unrecognised key, is a load failure rather than a silently
coerced default or a silently ignored input. R2.1 and R2.2 scope what
counts as an unrecognised key.

**R2.1.** R2's rejection of unrecognised keys is scoped to KnowHub's own
configuration namespace and configuration sources — the prefixed
environment variables, files and other inputs the settings mechanism claims
as its own. An unrecognised key inside that namespace fails validation
wherever the chosen settings mechanism can reliably identify it as
KnowHub's, because a misspelled KnowHub setting must not be silently
ignored.

**R2.2.** The backend must **not** fail startup merely because the host
process carries unrelated operating-system, container, orchestrator, CI or
platform environment variables. Everything outside KnowHub's configuration
namespace is ignored, not rejected. R2.2 narrows nothing in R2 or R2.1:
typed validation of KnowHub configuration is not weakened by it.
*(Sources for R2–R2.2: §77 — typed configuration, not scattered constants;
§84.1 — typed contracts; §76 — the same application runs in every
environment, which differ by configuration rather than by code)*

**R3.** A missing or invalid **required** setting fails process startup.
The process must not start in a partially configured state and then fail
later at the point of first use.
*(§77; §69.1 — validation errors fail immediately)*

**R4.** A configuration failure message identifies which setting is at
fault and why, and discloses nothing else: no secret value, no resolved
credential, no full environment dump, no connection string.
*(§65.3; §84.3;
[M00-SPEC-003](M00-SPEC-003-backend-observability-baseline.md) R12 for the
equivalent telemetry rule)*

**R5.** Behaviour that varies by environment, application or source is
expressed as configuration or policy, never as a scattered constant or a
branch on an environment name embedded in business logic.
*(§77)*

**R6.** Configuration must not encode model-specific or provider-specific
business logic. Model selection is a logical profile resolved through
configuration.
*(§0E.9; ADR-019)*

## 4. Normative requirements — domains

**R7.** The configuration domain set of
[§77.1](../../../knowhub_master_blueprint.html#configuration-model) —
Platform, Model profiles, Ingestion policy, Connector policy, Knowledge
policy, Identity policy, Security policy and Feature flags — is the
normative taxonomy for `knowhub-backend`. Every setting introduced by any
milestone belongs to exactly one domain.

**R8.** M00 declares the taxonomy but populates only the **Platform**
domain, and within it only what M00 needs: service identity and
environment name, the local dependency endpoints M00 actually uses, and
telemetry configuration as specified in
[M00-SPEC-003](M00-SPEC-003-backend-observability-baseline.md).
*(§77.1; §81 milestone 0A)*

**R9.** M00 does not define the content of the Model profile, Ingestion,
Connector, Knowledge, Identity or Security policy domains. Declaring a
domain name is not a licence to specify its settings ahead of the milestone
that owns the capability.
*(§81; §84.1)*

**R10.** A feature-flag mechanism exists and is typed. M00 defines no
product feature flag, because M00 delivers no product capability to gate.
*(§77.1)*

## 5. Normative requirements — precedence and scope

**R11.** The precedence chain of
[§77.2](../../../knowhub_master_blueprint.html#configuration-model) is
normative: a secure platform default is overridden by environment
configuration, which is overridden by enterprise policy, which may be
narrowed by application or source policy.

**R12.** A narrower scope may make behaviour **stricter** only. No lower
scope may silently weaken a mandatory security control.
*(§77.2; §84.1 — never weaken authorization for convenience)*

**R13.** Platform defaults are secure defaults. Where a setting has a
safe value and a convenient value, the shipped default is the safe one.
*(§77.2)*

**R14.** At M00 only the first two levels of the chain — secure platform
default and environment configuration — have any content. The mechanism
must nonetheless express the full ordering so later scopes attach without
redesign.
*(§77.2; §81.1 — preceding contracts remain stable)*

## 6. Normative requirements — secrets

**R15.** A secret is never an ordinary settings value. Configuration holds
a **reference** to a secret or credential, resolved through the
environment's approved secret manager.
*(§77.2; §65.3; §0E.1 Workload identity/secrets row)*

**R16.** The settings mechanism distinguishes a secret reference from a
plain value in its type system, so that a plain-text secret assigned to a
reference-typed setting is a load-time failure rather than a review
oversight.
*(§77.2; §65.3)*

**R17.** A resolved secret value must not be rendered by any
representation of a settings object — no log line, error message,
exception repr, debug endpoint or telemetry attribute.
*(§65.3;
[M00-SPEC-003](M00-SPEC-003-backend-observability-baseline.md) R12)*

**R18.** `.env.example` documents every setting a developer must supply,
with placeholder values only. It contains no real credential, token, key or
endpoint belonging to any deployed environment.
*(§84.3; §61)*

**R19.** M00 introduces no production secret and no enterprise secret
manager integration. Local development supplies its own non-production
values for local services.
*(§76 Local column)*

*Implementation guidance (non-normative): §0E.1 prefers passwordless
workload identity where the platform supports it, and names Azure Key Vault
as an example secret manager. Selecting and integrating a secret manager is
deployed-environment work; M00 only requires that the configuration shape
makes that integration a resolver change rather than a settings-model
change.*

## 7. Normative requirements — the Local profile

**R20.** The Local environment row of
[§76](../../../knowhub_master_blueprint.html#environment-matrix) is
realisable through configuration alone. Selecting Local must not require a
code fork, a build variant or a separate module tree — §76 states that
environments differ by configuration, scale, data class and approved
connectivity, not by separate code.

**R21.** No insecure bypass may be committed in an enabled state. §76
states this of the Local identity provider; M00 applies the same rule to
every Local-profile setting it introduces, since M00 has no identity layer
in which to scope it.
*(§76; §84.1)*

**R22.** Local configuration points at the local dependency services
specified in
[M00-SPEC-004](M00-SPEC-004-backend-build-ci-and-local-dependencies.md).
M00 configures the endpoints it uses; it does not configure clients for
services whose adapters belong to later milestones.
*(§76 Local column; §81)*

## 8. Acceptance criteria

| ID | Criterion |
|---|---|
| **M00-AC-009** | Settings load is typed and validated: an ill-typed value, and an unrecognised key inside KnowHub's configuration namespace, are rejected at load rather than coerced or ignored. Unrelated operating-system, container, orchestrator, CI or platform environment variables present in the host process do not prevent startup. *(R2, R2.1, R2.2)* |
| **M00-AC-010** | A missing required setting fails process startup, and the resulting message names the setting and its fault while disclosing no secret value, credential, connection string or environment dump. *(R3, R4)* |
| **M00-AC-011** | The precedence chain is demonstrable end to end for at least one setting: an environment value overrides a secure platform default, and a narrower scope cannot weaken a control marked mandatory. *(R11, R12, R14)* |
| **M00-AC-012** | No secret value is held as an ordinary setting; a plain-text secret assigned to a secret-reference setting fails at load; no settings representation renders a resolved secret; `.env.example` contains placeholders only. *(R15, R16, R17, R18)* |
| **M00-AC-013** | The Local environment profile is selectable by configuration alone, with no code fork or build variant, and no insecure bypass is committed in an enabled state. *(R20, R21)* |

## 9. Deferred

| Deferred from M00 | Owning milestone |
|---|---|
| Identity policy domain content — authentication mode, enabled providers, JIT provisioning, MFA, session/refresh lifetimes, bootstrap state, lock and password policy | M1 |
| Security policy domain content — classification handling, export policy, allowed model profile, retention | M1 / M12 |
| Model profile domain content — deployments, token limits, classification ceiling, timeouts | M6+ |
| Ingestion and Connector policy domain content | M3 / M10 |
| Knowledge policy domain content — freshness thresholds, authority ranking, required evidence count, incremental cutoff | M7 |
| Product feature flags | Each owning milestone |
| Enterprise secret manager and managed-identity integration | Deployed-environment work |
| DEV, SIT/UAT and PROD environment profiles and §76.1 promotion rules | Post-0B |
| Application- and source-scoped policy narrowing | M1+ |
