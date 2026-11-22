---
id: SPEC-tmf-bridge
companions:
  - ../../planning-artifacts/architecture/architecture-tmf-bridge-2026-11-16/ARCHITECTURE-SPINE.md
  - ../../planning-artifacts/prds/prd-tmf-bridge-2026-11-09/prd.md
  - ../../planning-artifacts/prds/prd-tmf-bridge-2026-11-09/addendum.md
  - ../../planning-artifacts/ux-designs/ux-tmf-bridge-2026-11-11/EXPERIENCE.md
  - ../../planning-artifacts/ux-designs/ux-tmf-bridge-2026-11-11/DESIGN.md
sources: []
---

> **Canonical contract.** This SPEC and the files in `companions:` are the complete, preservation-validated contract for what to build, test, and validate. Source documents listed in frontmatter are for traceability — consult them only if you need narrative rationale or prose color this contract intentionally omits.

# TMF-Bridge S1

Companions are adopted, owned by their upstream skills. The **architecture spine** is the binding build substrate: its AD-1..AD-17 rules, Stack table and Structural Seed apply as written. The **PRD** holds FR-1..FR-39 and NFR-1..NFR-6, the testable detail for each capability. **EXPERIENCE.md** and **DESIGN.md** hold studio behavior and visual design. On conflict, the PRD wins over the spine on requirements (spine "PRD Amendments" records the folds), the spine wins on architecture.

## Why

Pain and opportunity. Telco integrators must expose TM Forum Open APIs to partners now, while legacy backends are still being transformed. Today that means per-API code that is built and redeployed. TMF-Bridge exposes stable, runtime-validated TMF APIs from configuration alone, and is built to be retired: when a backend goes native, the operator keeps the mappings and the id map as portable text. "Stable" means a partner's contract does not change when the backend is replaced.

## Capabilities

- **CAP-1** Runtime serving (FR-1..FR-4)
  - **intent:** A partner can call any configured TMF operation and get a TMF response, or a TMF Error with a fixed status, with no restart after configuration.
  - **success:** `GET /productOffering/{id}` returns a schema-valid payload; an unmapped or Draft-only operation returns `501` TMF Error; an unknown id returns `404`; backend 5xx, other 4xx, transport fault and mapping error return `502`, timeout `504`, invalid request `400`; a mapping applied at runtime is served on the next call.
- **CAP-2** Mapping (FR-5..FR-7)
  - **intent:** An integrator can translate between TMF attributes and backend fields, including ids, arrays and nested objects, using only the closed transform vocabulary.
  - **success:** Each S1 node type (`path`, `lit`, `concat`, `coalesce`, `if`, `lookup`, `convert`, `idMap`, `map`, `filter`) passes its contract test; `@type`, `@baseType`, `@schemaLocation` pass through; an id-map miss in either direction yields `502`.
- **CAP-3** Runtime conformance (FR-8, FR-9)
  - **intent:** The bridge guarantees every request and translated response matches the official OpenAPI definition, and never claims certification.
  - **success:** An invalid translated response is never returned (`502`); no studio copy, API description or export says conformant, compliant or certified; the Disclaimer is reachable from studio help and README.
- **CAP-4** Mapping studio (FR-10..FR-13, FR-28..FR-35, FR-37, FR-39)
  - **intent:** A non-developer analyst can create, test, version and apply a mapping, manage backends and the id map, and get plain-language verdicts, all in a keyboard-accessible web studio.
  - **success:** UJ-1, UJ-3 and UJ-5 run end to end in the studio: six steps, three Test sub-steps with per-sub-step verdict rules, Exchange view with lineage, apply to "Version {n} is live", stale ETag blocks Apply and autosave with edits kept, backend delete blocked while referenced, Playwright plus axe-core pass on every screen.
- **CAP-5** Configuration as code (FR-14..FR-18, FR-38)
  - **intent:** An operator can export and import all configuration through REST and the studio, deterministically and atomically, as the exit path and the promotion path between environments.
  - **success:** export, import, export is byte-identical; an invalid import changes nothing and lists each error by JSON Pointer; import lands mappings as Draft and never alters Live; every studio action has a REST equivalent in the control-plane OpenAPI.
- **CAP-6** Observability and lineage (FR-19..FR-23)
  - **intent:** An operator can see what the bridge did and what it cost, while sensitive data stays out of every output.
  - **success:** Injected secret and sensitive-path values appear in no log, step result, lineage or bundle; studio test calls return step results and field lineage, production calls return neither; every runtime response carries `Server-Timing: bridge;dur=`; p95 overhead stays under 100 ms under NFR-1 conditions.
- **CAP-7** Security (FR-24..FR-26, FR-36)
  - **intent:** Only authenticated users reach the studio, config API and (by default) runtime API, and backend credentials never leave the secret store.
  - **success:** Every control-plane endpoint returns `401` without a token; the runtime chain can be disabled independently; backends hold secret names only, resolved at call time; any authenticated user can do everything.
- **CAP-8** Mock backend and showcase (FR-27)
  - **intent:** An open-source user can clone the repo and see TMF620 served and TMF622 mapped end to end against a shipped mock.
  - **success:** Compose demo brings up Postgres, API, studio, mock and Keycloak; TMF620 is served; TMF622 returns `501` until mapped; a user reaches a working showcase in ten minutes following the README (a promise, not a metric).

## Constraints

- Hexagonal layering is machine-enforced: domain imports no Spring, JPA, HTTP or adapters; `application/runtime` and `application/control` never import each other (AD-2, ArchUnit fails the build).
- Config is data: no class, route or bean named after a TMF resource, and adding an API or mapping never needs a rebuild (AD-1).
- Control plane is profile-gated (`control`); the data plane reads published versions only, never a Draft, never migrates schema, and reports not-ready on an old schema (AD-3).
- Studio is a pure client of the public config API on one origin; three images (`tmf-bridge-api`, `studio-frontend`, `tmf-mock`) (AD-4).
- Studio tests and production run one `CallPipeline`; test mode is GET-only and fails before network I/O; preview persists nothing and sample bodies reach no log or DB (AD-5, FR-13, FR-28).
- Mapping is canonical JSON with a closed expression tree, JSON Pointer addressing, `schemaVersion` 1, `BigDecimal` numbers; absent differs from `null`; free-form scripts are rejected (AD-6).
- PostgreSQL is the single store, holding config and id map only, never resource state; applied versions and specs are immutable by DB trigger; Flyway owns the schema; JPA entities never cross ports (AD-7, AD-15).
- Exports, bundles and hashes go only through the domain canonicalizer, carry no generated ids or secret values; import is atomic (AD-8, FR-16..FR-18).
- Secrets are referenced by name only, resolved from env or file at call time, and every output passes the single Redactor; endpoints with embedded credentials are rejected (AD-10).
- Runtime is blocking Spring MVC on Java 25 virtual threads, no reactive types, fail closed, fixed error table, 5 s default timeout, no retries (AD-11).
- NFR-1: p95 bridge overhead under 100 ms (warm caches, one instance, zero-latency mock, payload up to 50 KB, capture excluded).
- Studio: WCAG 2.2 AA, keyboard-only parity with a Connect picker for every drag, no status by color alone, desktop evergreen browsers at 1280 px minimum with reflow to 320 CSS px, light and dark themes, English first (NFR-2..NFR-6, AD-17).
- S1 claims and tests the mock backend only; REST and JSON backends only.
- Stack and versions per the spine Stack table (Java 25, Spring Boot 4.1.3, React 19.3.1, Vite 8.3.8, etc.), with these pins overriding the spine: Maven 3.10.1, networknt json-schema-validator 3.0.9, Spring Boot 4.1.3 (4.2.0 not adopted).
- FR-37 user manual and glossary are built by the project team as a story of this spec, not delegated to an external role.

## Non-goals

- Changes to backends, or business logic beyond translation and orchestration.
- Any TM Forum certification or conformance claim.
- Real backends (mocks only), WSDL/SOAP, SQL and Kafka backends.
- A textual expression language (CEL), free-form scripting, JSONPath syntax.
- Write operations: TMF622 writes, PATCH, cancel, retries, saga, idempotency, hub, async correlation, notifications.
- Mapping history, diff and rollback; roles, four-eyes publication and audit trail; configurable error mapping.
- Drift detection and Git workflows; restoring a whole configuration into a populated runtime.
- Multi-tenancy, TLS, trace and correlation ids beyond the log trace id, scale-out consistency, sizing and resource limits.
- AI-assisted mapping generation (Later; the published mapping schema keeps it possible).

## Success signal

UJ-1 to UJ-5 (PRD) each run end to end through the studio and the REST API using configuration only, with no code written, built or redeployed: an analyst adds a field and maps TMF622, a partner's TMF620 call is schema-valid and then TMF622 goes from `501` to working, and a failed call is diagnosed and forwarded as a share bundle.

## Assumptions

- Production calls return no step results or lineage; FR-20 and FR-21 are studio-test features (spine AD-5 marks this an assumption).
- The addendum's note that the spine still needed "id-map miss is `502` both ways" is stale; the spine already states it (AD-6, AD-7, AD-11).
- Sizing, resource limits and replica counts are ignored for now (user), consistent with the Non-goals.
- PRD, addendum, EXPERIENCE.md and DESIGN.md are adopted whole as the detail layer; this kernel does not restate their FRs.

## Open Questions

- Where are compound FRs 17, 30, 34 and 39 split? Left to epics and stories.
