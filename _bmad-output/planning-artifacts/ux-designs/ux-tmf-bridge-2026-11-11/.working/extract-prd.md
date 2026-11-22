# UX extract: TMF-Bridge PRD (final, updated 2026-11-11)
Sources: prd.md, draft-journeys.md (older draft, superseded by PRD journeys), addendum.md, brief.md.

## Product in 3 lines
- TMF-Bridge: lightweight, open-source (Apache-2.0) adapter exposing stable TM Forum Open APIs over legacy BSS/OSS backends, validated at runtime against official OpenAPI.
- Wedge: configuration over code. Mappings and flows are config, applied without restart, exported as portable text.
- Built to be retired: operator keeps mappings and id map when backend goes TMF-native. Independent project, no TM Forum certification claim (FR-9, disclaimer).

## The studio (F4 "Mapping Studio")
- Minimal, no-code UI for analysts: create/edit a mapping, test a call, see response + field lineage, apply to running bridge.
- S1 scope IN: TMF620 productOffering and TMF622 `GET /productOrder` reads; closed transformation list; mock backend only; read-only test calls.
- OUT / Later: writes, dry-run with stubbed backends, validate gate, author/publisher roles, audit trail, mapping history/diff/rollback, TMF622 writes/PATCH/cancel, TMF642, full polymorphism, TLS, multi-tenancy, drift detection/Git workflows (outer process using FR-14..18).
- Brief's earlier demo moments: unconfigured TMF620 -> 501, then answers after config; studio change exported, applied via REST, visible in next response with lineage.

## Personas / protagonists
- Ines: integration analyst, not a developer; studio user (UJ-1, UJ-3). Primary persona (telco integrators).
- Tomas: partner developer, northbound API caller, NOT a studio user (UJ-2, UJ-4).
- Draft only (brief source input, simulated): Claire (Head of BSS Transformation, wants "just configuration"), Marc (solution architect, reviews PRs). Names marked invented/inferred in draft.
- Secondary: TM Forum community and vendors (contribute mappings).

## Journeys (verbatim PRD names)
- UJ-1: Ines adds a field to an existing mapping. (TMF620 offering already mapped; maps backend field to TMF attribute, tests, sees new value + lineage + OpenAPI validation pass, applies; next call serves the field, no restart.)
- UJ-2: Tomas calls TMF620, then TMF622. (schema-valid, no legacy leak, <100 ms overhead; unmapped TMF622 -> 501 TMF Error.)
- UJ-3: Ines maps TMF622. (adds productOrder API starting with GET /productOrder, creates mapping to legacy order system, tests, applies; live no restart.)
- UJ-4: Tomas calls TMF622 and it works.
- Draft UJ-1 title (older, includes Git/PR/drift steps now out of S1): "Ines adds a backend field to a TMF620 attribute and carries it to production". Draft example: `family_name` -> product family label [INFERRED].

## FRs implying a UI surface
- FR-10 create/edit mapping, no code.
- FR-5 per-operation mapping, fields -> TMF attrs; closed transforms: direct copy/rename, constant, concatenation, conditional/default, lookup table, type/format conversion (dates, numbers).
- FR-11 test a call, see response with field lineage.
- FR-21 lineage: each TMF field -> backend source + transformation (studio test calls only).
- FR-20 result of every mapping step viewable per call.
- FR-12 apply mapping to running bridge.
- FR-13 test calls never change backend state (reads vs backend or mock).
- FR-8 / FR-9 validation result shown (runtime OpenAPI validation only, no certification claim).
- FR-22 overhead measured and reported per call (show in test result).
- FR-2, FR-4 error states: 501 / 404 / TMF Error bodies, no backend internals.
- FR-14..18 same REST API as UI: configure/apply/import/export/test. Export JSON, secret-free (FR-16), deterministic (FR-18); import atomic, invalid rejected, running config unchanged (FR-17). Whether the studio exposes import/export UI is not stated.
- FR-24 studio/admin authenticate via OIDC. FR-23 redaction applies to step results and lineage shown in UI.
- FR-3 no-restart effect (UI should reflect applied state). FR-27 mock backend for demo; 10-minute zero-to-hero promise (not a metric).

## NFRs touching UX
- NFR-1: bridge overhead <100 ms per call (percentile/conditions set in architecture).
- Accessibility, i18n, browser/platform: NOT specified anywhere. Success measures deferred.
- Other NFRs (idempotency, confidentiality) to be elicited in architecture.

## Data concepts the UI must show
- Mapping: operation binding, backend field -> TMF attribute, transformation type (closed list), constants, lookup tables.
- Test call: request, response payload, per-step mapping results, field lineage (TMF field <- backend source + transformation), OpenAPI validation outcome, overhead ms.
- Errors: TMF Error body, HTTP 501 (unconfigured op), 404 (unknown id), mapped backend failures; no internals leaked.
- TMF entities: TMF620 productOffering (`GET /productOffering/{id}`), TMF622 productOrder (`GET /productOrder`); extension attrs `@type`, `@baseType`, `@schemaLocation`.
- Id map (TMF id <-> backend record id); config export/import JSON; applied/running config state; backend = mock (catalog, order).
- Redaction: sensitive data masked in logs, steps, lineage; no secrets in exports.

## Open questions / assumptions relevant to UX
- Mapping expression language and flow model deferred to architecture (user ruling 2026-11-11): determines studio editing paradigm.
- Dry run vs real backends: S1 read-only only; write dry-run Later.
- Runtime stateful (DB); what else it stores (versions, subscriptions) open.
- Zero-code honesty: polymorphism, v4/v5 diffs, task ops, hub semantics may need code.
- Draft journeys' Git/PR/drift steps are not in PRD S1 (outer process).
- Persona names Ines/Tomas flagged as invented in draft.

## Form-factor hints
- "Minimal studio", web-style GUI for an analyst. Stack context (inputs, not decisions): Java 25, Spring Boot 4, Kubernetes/Helm, PostgreSQL, Kafka, OIDC, OpenTelemetry; "No front-end standard". Nothing on mobile, responsive, or desktop-only.
