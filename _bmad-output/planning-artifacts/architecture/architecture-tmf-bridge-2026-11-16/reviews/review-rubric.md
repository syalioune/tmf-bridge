# Review: ARCHITECTURE-SPINE.md against PRD (good-spine checklist)

Reviewed: spine (updated 2026-11-20) vs PRD prd-tmf-bridge-2026-11-09 (final). The addendum named in the spine sources was not reviewed. Tech versions were checked only for plausibility, with no live lookup.

## Verdict

**Revise before handing to the level below.** The spine is strong on layering, the config-as-data core, secrets and the control/data split. Most ADs have a testable Rule. The gaps are:
- the operational and environmental envelope, which is almost absent;
- two Rules that contradict the PRD or each other;
- unnamed core technology in the engine (OpenAPI validator, JSON model);
- the whole studio unit, which is bound only by name.

## Findings

### F-1 [HIGH] Operational and environmental envelope is mostly undecided
The spine gives one deployment paragraph (compose demo, Helm, operator-provided Postgres) and one diagram edge `API -.OTLP / Prometheus.-> Obs`. No AD decides, defers or opens the following:
- **Environments and promotion.** There is no dev/test/prod model, per-environment config, or image tagging/registry/release policy. The deferred "Id-map restore" and "environment promotion" show the need, but nothing frames it.
- **Observability.** OTLP and Prometheus appear only in a diagram. No AD names the metrics set beyond `bridge.overhead`, structured-log field schema, trace propagation, or the health, liveness and readiness probes. Helm cannot be written without the probes.
- **Rollout ordering (AD-3).** Data-plane pods fail fast if the schema is older, and only control pods or a pre-upgrade Job migrate. The spine does not say how rolling upgrades work. Data pods can start before the migration and crash-loop. Nothing requires expand/contract compatibility between old data pods and a new schema.
- **Backup and DR.** Postgres holds the id map, which is runtime state, and the PRD says the exit path is export. The spine says "single-region HA" and nothing about backup, RPO/RTO, or id-map recovery. Its restore story is deferred additive-only import.
- **Compose ingress.** AD-4 requires one origin through an ingress. The compose deployment names no reverse proxy.
- **Studio runtime config.** Nothing says how the static SPA image receives its OIDC authority, client id and API base URL at deploy time. This is a real divergence point.
- **CI/CD and sizing.** Nothing decides the pipeline shape, per-instance sizing, DB connection pool, or capacity assumptions behind the NFR-1 "one instance" condition.

Fix: add an operations AD (or an explicit Deferred or Open entry per item above).

### F-2 [HIGH] AD-5 and AD-6 Rules are unsatisfiable or leave gaps
- **Preview vs. id map.** AD-5 says `POST /preview` has "no repository dependency (ArchUnit)". But `idMap` and `lookup` nodes need id-map and lookup data, and FR-28(a/b) must preview mappings that use them. Either preview takes the id map as input, or a read-only port is allowed. As written, the ArchUnit check either fails or forces the preview to silently skip `idMap`.
- **Mapping errors have no status.** AD-6 says an unresolved path in a scalar slot "is a mapping error". The AD-11 fixed error table does not map it (400, 500 or 502?). Two implementers will diverge. It also interacts with sparse legacy backends, since `coalesce` is the only escape.
- **FR-20 "result of every mapping step for a call".** AD-5 turns lineage off in production. AD-6 defines step results as the two table outputs plus the call. The spine never says where step results are exposed on non-studio calls, or whether FR-20 is studio-only.
- **AD-7 "constraint or trigger".** The Rule leaves the mechanism open, so it is not enforceable as one thing.

### F-3 [HIGH] Core engine technology and a domain boundary are unnamed
- **No OpenAPI validator.** FR-8 (runtime validation of every request and response) is the product's central promise, and AD-11 only says "domain validator port". The Stack names no JSON Schema or OpenAPI validation library, no OpenAPI 3.0 vs 3.1 handling (TMF specs are 3.0, so there are nullable and discriminator semantics), and no JSON Pointer library. Validators differ on `nullable`, `readOnly`/`writeOnly` and `additionalProperties`, so this is a behavioral divergence point, not a detail.
- **JSON model in domain.** AD-2 forbids "Jackson-annotated web types" in domain, yet the evaluator must operate on a JSON tree. Whether Jackson 3 `JsonNode` is allowed in `domain` is unstated. Boot 4 moved to Jackson 3, and this is a likely source of unit drift.
- **Mapping schema evolution.** AD-6 calls the schema "published, versioned", but there is no rule for stored MappingVersions written against an older schema version, or for adding a node type later.

### F-4 [MEDIUM] Studio unit and several FRs are bound weakly or not at all
Binds says FR-1..FR-39 and NFR-1..6. The map covers most, but these are bound to an AD that does not govern them:
- **NFR-2, NFR-3, NFR-4, NFR-5, NFR-6** map to AD-4, which decides only images and API-only access. No decision covers frontend state, API client generation (springdoc contract to TS), the keyboard Connect picker and drag layer, the theme/mode persistence store, or the browser-only sample store (NFR-6). The server half of NFR-6 is in AD-5 only. Test tooling (Playwright and axe) is listed but nothing binds it as a gate.
- **FR-33 autosave** (drafts on change, "Save now") has no rule. Interaction between autosave writes, integer-`version`/ETag, and the FR-32 stale-apply block (user's unsaved edits kept) is unspecified, and it is a classic two-unit mismatch between studio and API.
- **FR-7** (`@type`, `@baseType`, `@schemaLocation`): FR-7 appears in the F2 row but no AD says how these are handled (passed through, validated, or required).
- **FR-9, FR-35, FR-37**: unbound by any AD. `docs/` appears in the tree for FR-37, but the delivery mechanism is not decided. Low risk, but the checklist asks that each FR be bound or deferred.

### F-5 [MEDIUM] Identity and referencing in config export/import is underdefined
- AD-7 says ids are server-generated opaque. FR-18 requires exports with "no generated ids". The spine does not say how exported mappings reference ApiSpec (by content hash? by name+version?), Backend (by name, per FR-34), or each other.
- Mapping identity and name uniqueness is not in the ERD or entity owners, although FR-35 sorts by name and FR-17 needs "a Draft imported over an existing mapping becomes the Draft of its next version". Import matching rule (by name? by spec+operation?) is a divergence point.
- Draft versus applied row model: how "next version" numbering and the Draft/Live status derive from rows and the live pointer is implied but not stated.

### F-6 [MEDIUM] Stack currency and Deferred items
- Versions that look plausible but were not independently verified here: Java 25, Spring Boot 4.1.2, Hibernate 7.3, Liquibase 5.0.4, springdoc 3.2.0, ArchUnit 1.5.0, Keycloak 26.8.2, Vite 8.3, react-aria-components 1.22, Spectral 6.17, WireMock 3.13. "React 19.x" and "Maven 3.9.x" are marked "verify at scaffold", so they are not pinned. PostgreSQL is "operator-provided" with no minimum version (jsonb and constraint features matter for AD-7).
- Missing from the Stack: OpenAPI/JSON Schema validator (F-3), Micrometer/OTel exporter choice, Testcontainers, Playwright/axe versions.
- Deferred mostly sound. Items that could let units diverge: "TLS" (bearer tokens in clear unless the ingress terminates TLS, which is not stated) and "trace/correlation ids beyond trace id in logs". Add one line each saying who owns them, ingress or app, in S1.
- AD-13: runtime-chain bearer validation says nothing about whether the runtime and control chains share one issuer and audience. The FR-25 "whatever the access path" reading is satisfied only if this is fixed.

## FR/NFR coverage summary

| Status | Items |
| --- | --- |
| Bound and enforceable | FR-1..6, 8, 10..32 (except as noted), 34, 36, 38, 39, NFR-1 |
| Weak or misattributed | FR-7, FR-20, FR-33, NFR-2..6 |
| Unbound | FR-9, FR-35, FR-37 |
| Deferred correctly | Later list in PRD (history, roles, WSDL/SOAP, multi-tenancy, scale-out, and others) |

## What is good
Layering and the control/data split are machine-checkable. AD-8 (canonicalizer and atomic import), AD-10 (single Redactor) and AD-14 (per-sub-step verdict) are tight, with concrete tests. The Deferred table is honest and ties each item to a revisit trigger.
