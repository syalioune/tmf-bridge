# Extract: Original Brief (TMF-Bridge, 2026-10-21, @Alioune)

Source: "Project Brief — TMForum Open API Adapter.md"

## Purpose
Configurable adapter exposing chosen TMForum Open APIs northbound, translating calls to legacy backends unaware of TMF/SID. Mapping and orchestration are configuration, not code (no rebuild to add a TMF API or backend). PoC: TMF620 (read-heavy), TMF622 (write-heavy, long-running), TMF642 (event-driven). Analysts author mappings via GUI. Horizontal scale with durable state.

## Personas / Actors
- Claire: Head of BSS Transformation (stakeholder), mid-size mobile and fixed-line operator. Wants "just configuration", no Java release per field move.
- Marc: Telecom BSS/OSS Solution Architect.
- Integration analysts: author/publish mappings in Mapping Studio (not developers; no hand-edited config files).
- Northbound consumers: digital channels, partner portals, ODA-aligned order orchestrator, NOC dashboard.
- Southbound backends: home-grown catalog (REST/JSON over Oracle), pricing engine (SOAP), OMS (REST + async status callbacks), billing (SOAP), fault-management (REST + Kafka).

## Functional Requirements (id | text | original priority)
- FR-01 | Expose any TMF Open API selected by config from official OpenAPI (v4.x baseline, v5 readiness); PoC TMF620/622/642 | Must
- FR-02 | Serve only enabled resources/operations; others return 501 with TMF Error | Must
- FR-03 | Requests/responses conform to TMF schema incl. polymorphic @type/@baseType/@schemaLocation | Must
- FR-04 | Bidirectional per-operation mapping: fields, computed, conditions, defaults, code lookups, arrays, nested | Must
- FR-05 | Stable TMF id <-> backend key correspondence surviving restarts | Must
- FR-06 | Produce absent TMF conventions: id, href, @type, lifecycleStatus, validFor, lastUpdate | Must
- FR-07 | One TMF op combines backend calls: sequence, parallel+merge, conditional, per-item; per-call timeout/retry | Must
- FR-08 | Multi-backend write failing part-way is compensated; progress survives instance failure and is traceable | Must
- FR-09 | TMF query: fields, offset, limit, attribute filter, sort; backend pushdown else bounded fallback; correct X-Total-Count/X-Result-Count | Must
- FR-10 | PATCH with application/merge-patch+json; json-patch+json optional per API | Must
- FR-11 | TMF622 lifecycle: create, retrieve, list, PATCH, delete where allowed, cancelProductOrder; states driven by backend responses/callbacks | Must
- FR-12 | Repeated create with same idempotency key or externalId returns original, no new backend write | Must
- FR-13 | Async backend updates (callbacks, Kafka) correlated to TMF resources, produce state changes and TMF events | Must
- FR-14 | TMF642 tasks: acknowledge, unacknowledge, clear, comment, group, ungroup | Must
- FR-15 | Event hub (/hub register/unregister) per API; subscriptions survive restarts; at-least-once delivery, retry, dead-letter | Must
- FR-16 | PoC southbound: REST/JSON, SOAP/XML, read-only SQL, Kafka, inbound webhooks; new endpoints = config, new protocol may need code | Must
- FR-17 | Backend errors mapped to TMF Error + HTTP status by config; business validation to 400/422; no internal leak | Must
- FR-18 | Analyst mapping GUI: TMF schema beside backend schema (OpenAPI, WSDL, JSON sample, table), visual mapping, assisted expressions, flow design incl. compensation, lookup tables and endpoint editing | Must
- FR-19 | Mapping lifecycle: draft, validate, test, publish, rollback; history and diff; publication reaches all instances without restart | Must
- FR-20 | Try TMF request against draft: shows each backend call and TMF response, no side effects on real backends | Must
- FR-21 | GUI via company OIDC; roles viewer/author/publisher minimum; audit trail of every change | Must
- FR-22 | Field-level lineage in GUI: backend source and transformation per TMF field | Should
- FR-23 | Export/import mapping sets in readable text for Git review and env promotion | Should
- FR-24 | Multi-tenancy: tenant from token or header; mapping sets per tenant | Should

## Non-Functional Requirements
- NFR-01 Read perf | TMF620 300 req/s sustained; adapter overhead p95 < 300 ms (backend excluded)
- NFR-02 Write perf | TMF622 create 50 orders/s peak; sync `acknowledged` p95 < 1 s (backend excluded)
- NFR-03 Event throughput | TMF642 2,000 backend events/s peak; e2e notification p95 < 2 s
- NFR-04 Consistency | No lost/duplicated order under instance failure; no lost notification; listeners dedupe on eventId
- NFR-05 Scalability | Horizontal, interchangeable instances; publication reaches every instance within 30 s
- NFR-06 Availability | 99.9 % monthly; failing backend degrades only dependent operations
- NFR-07 Security | OAuth2/OIDC northbound, scopes per API+operation; backend creds never in mappings/exports; encrypted inter-service traffic
- NFR-08 Observability | One trace per northbound call covering all backend calls; rate/error/latency metrics per API, operation, backend; correlation id in logs; visibility of pending compensations and notifications
- NFR-09 Conformance | TMForum CTK for TMF620/622/642 passes in CI for enabled operations
- NFR-10 Testability | Each mapping auto-testable against recorded backend responses; coverage visible to analysts
- NFR-11 Usability | Analyst adds backend field to TMF attribute, tests, publishes in < 30 min without developer help

## Mapping Scenarios
1. Read offering: GET /productOffering/{id} <- catalog GET /offers/{code} + pricing SOAP GetOfferPrices + catalog GET /families/{id}; id translation, parallel merge, status lookup ACT->Active, date conversion, price list -> productOfferingPrice[], backend 404 -> TMF Error.
2. List offerings: ?lifecycleStatus=Active&fields=id,name; backend filters status not category; pushdown else bounded fallback, field selection, correct counts.
3. Create order: POST /productOrder -> OMS POST /orders then billing SOAP OpenChargeContext; ordered writes, cancel OMS if billing fails, idempotency, acknowledged response.
4. Track order: ProductOrderStateChangeEvent from OMS callback IN_PROGRESS/DONE/REJECTED; correlate by OMS order number; map to inProgress/completed/failed.
5. Cancel order: POST /cancelProductOrder -> OMS cancel + billing CloseChargeContext; multi-step with compensation, TMF task state.
6. Publish alarms: AlarmCreateEvent/AlarmAttributeValueChangeEvent from Kafka fm.alarms.v1, kind RAISED/CLEARED; route by content.
7. Acknowledge alarms: POST /ackAlarms -> fault REST ack per alarm; loop, partial failures reported in TMF response.

## Acceptance Criteria
1. GET productOffering/{id} returns schema-valid offering merged from catalog+pricing.
2. List with lifecycleStatus=Active&fields=id,name&limit=20 returns only requested fields, correct count headers.
3. POST productOrder returns 201 state acknowledged; order in OMS, charge context in billing.
4. Same POST with same idempotency key returns original; nothing new in OMS.
5. Billing fails after OMS success: OMS order cancelled via compensation, client gets TMF Error, each compensation step traceable.
6. OMS completion callback moves order to completed; ProductOrderStateChangeEvent reaches every listener.
7. POST cancelProductOrder cancels OMS order and closes billing charge context.
8. Backend 404 or SOAP fault returns TMF Error with no internal detail.
9. Listener receives AlarmCreateEvent within 2 s of backend Kafka event.
10. POST ackAlarms / clearAlarms reach fault system; alarm reflects new state.
11. Studio: analyst maps new field, dry-run tests, publishes; all instances serve within 30 s no restart; invalid mapping unpublishable; rollback restores previous.
12. Killing a pod during order creation loses no order, duplicates no backend write, loses no event.
13. TMF CTK suites for 620/622/642 pass for enabled operations.
14. Enabling fourth API (TMF637 Product Inventory read ops, mock backend) needs zero Java changes.

## Out of Scope
Backend changes; business logic beyond translation/orchestration (pricing rules, eligibility, order decomposition); other TMF APIs in PoC (TMF637 must be config-only); multi-region DR for adapter datastore (PoC single-region HA).

## Open Questions
1. How to store mapping versions, id maps, subscriptions, in-flight orders; one store for all?
2. How are mappings/flows represented; which expression language for analysts?
3. Keep full copy of each ProductOrder for reads, or rebuild from OMS per GET?
4. Four-eyes approval on publish (author != publisher)?
5. What bounds in-memory filter fallback: fixed cap or per-operation setting?
6. Expose TMF v4 and v5 side by side, or v4 only in PoC?
7. Event ordering: per resource id only, or global?

## Technology Context (customer habits, not prescriptions)
Java 25, Spring Boot 4; Kubernetes, containers, Helm; PostgreSQL, Kafka (reuse preferred but must justify fit); central OIDC, secrets vault; OpenTelemetry; no front-end house standard (anything else must earn its place).

## Tone / Voice / Qualitative Ideas
- "Just configuration" is Claire's mantra: no Java release when a field moves; code only for a genuinely new protocol.
- "Consumers see ODA; the legacy stays untouched."
- Instances interchangeable; state is minimal but durable (id map, hub subs, mapping versions, order correlation/compensation, idempotency keys, pending notifications). Adapter "stateless runtime, stateful where necessary".
- Nothing internal leaks (past vendor leaked Oracle stack traces to partner).
- Partner pain: duplicate orders, lost status updates. Answer fast with acknowledged, follow OMS async.
- Filtering fallback must be honest: say so in response headers.
- Analysts want transparency: see where each TMF field comes from; draft/validate/publish with history, diff, rollback; try on sample payloads.
- Pragmatic, declarative, trust-oriented voice; design decisions (storage, language, GUI stack) deliberately left to the solution to justify.
- Glossary: TMF Open API, SID, ODA, CTK, northbound/southbound, Saga.
