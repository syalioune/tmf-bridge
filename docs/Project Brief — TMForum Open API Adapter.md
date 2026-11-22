# Project Brief — TMForum Open API Adapter

Oct 21, 2026 · @Alioune

## 1. Purpose and context

Build **TMF-Bridge**, a configurable adapter that exposes any chosen set of TMForum Open APIs northbound and translates each call into calls on legacy backends that know nothing of TMForum or SID. Mapping and orchestration are configuration, not code: adding a TMF API or a backend must not require a rebuild.

The proof of concept covers three APIs with different load profiles: TMF620 Product Catalog (read-heavy), TMF622 Product Ordering (write-heavy, long-running) and TMF642 Alarm Management (event-driven). Integration analysts must be able to author mappings and flows through a GUI. The component must scale horizontally while keeping the state it needs durably.

**Actors**

- **Stakeholder:** Claire, Head of BSS Transformation at a mid-size mobile and fixed-line operator.
- **Architect:** Marc, Telecom BSS/OSS Solution Architect.
- **Integration analysts:** author and publish mappings in the Mapping Studio.
- **Northbound consumers:** digital channels, partner portals, an ODA-aligned order orchestrator, the NOC dashboard.
- **Southbound backends:** a home-grown product catalog (REST/JSON over an Oracle schema), a pricing engine (SOAP), an order management system or OMS (REST with asynchronous status callbacks), a billing system (SOAP), and a fault-management system (REST + Kafka events).

&#91;embedded content: TMF-Bridge target architecture · 4 layers\]

Consumers only ever see TMF APIs. Mappings that analysts publish through the GUI turn each call into one or more backend calls, and some state must outlive any single instance.

## 2. Discussion transcript

**Claire:** Our partners and the new order orchestrator all expect TMForum APIs. Our catalog and fault systems are fifteen years old and speak their own dialect. Rewriting them is off the table for at least three years. What can we do?

**Marc:** Put an adapter in front of them. It exposes TMF APIs northbound, conformant to the spec, and translates every call into whatever the backend understands. Consumers see ODA; the legacy stays untouched.

**Claire:** Which APIs first?

**Marc:** Three, each stressing a different pattern. TMF620 Product Catalog Management is read-heavy, deeply nested, with polymorphic resources such as ProductOffering, ProductSpecification and ProductOfferingPrice. TMF622 Product Ordering is write-heavy: orders are created, patched and cancelled, run for hours or days, and each one writes to several backends. TMF642 Alarm Management is event-driven, with task operations like acknowledge and clear, and notifications pushed to subscribers. If the adapter handles all three, TMF641 or TMF637 later is just configuration.

**Claire:** "Just configuration" is exactly what I want. I don't want a Java release every time a field moves.

**Marc:** Agreed. The TMF API definitions should come from the official OpenAPI files. Each operation is then bound to a mapping and an orchestration flow that can be changed, versioned and published without a redeploy. Code should only be needed for a genuinely new kind of backend, for example a new protocol.

**Claire:** What does a mapping actually do?

**Marc:** Three things. It transforms payloads both ways, TMF JSON to backend format and back, with an expression language for computed fields. It maps identifiers, because TMF ids and backend keys rarely match. And it fills TMF conventions the backend lacks: `href`, `@type`, `@baseType`, `lifecycleStatus`, `validFor`.

**Claire:** And orchestration?

**Marc:** One TMF call often needs several backend calls. A ProductOffering read needs the catalog for the offer and the pricing engine for its prices, merged into one response. A flow is a small graph of steps: sequential, parallel, conditional, with retries and timeouts. Writes are harder: a ProductOrder creates the order in the OMS and opens a charge context in billing. There is no distributed transaction, so each write step declares its compensation, saga-style.

**Claire:** What about queries? Partners filter offerings by category and status.

**Marc:** The adapter supports TMF query conventions: `fields`, `offset`, `limit`, attribute filters, sorting. Where the backend can filter, we push the filter down. Where it can't, we filter in memory with a configured hard limit, and we say so in the response headers. `X-Total-Count` and `X-Result-Count` must be right.

**Claire:** Ordering is where partners hurt us today. Duplicate orders, lost status updates.

**Marc:** Three rules. Every create is idempotent: a retried POST with the same idempotency key or `externalId` returns the original order, never a second one. The adapter answers fast with the order in `acknowledged` state, then follows the OMS asynchronously; OMS callbacks move the TMF state through `inProgress` to `completed`, `failed` or `partial` and fire a `ProductOrderStateChangeEvent` to subscribers. Cancellation goes through the TMF `cancelProductOrder` task, mapped to the OMS cancel call, with compensation in billing.

**Claire:** Alarms come from the fault system in real time. How do subscribers get them?

**Marc:** TMF642 uses the hub pattern: consumers register a callback through `/hub`. The adapter consumes the fault system's Kafka topic, maps each backend event to an `AlarmCreateEvent`, `AlarmAttributeValueChangeEvent` and so on, and posts it to every matching listener, with retries and a dead-letter queue.

**Claire:** Errors. Last time a vendor leaked Oracle stack traces to a partner.

**Marc:** Every backend error is mapped, by configuration, to the TMF Error object with the right HTTP status. Nothing internal leaks. The original error is logged with the correlation id.

**Claire:** Security and multi-tenancy?

**Marc:** OAuth2 / OIDC on the northbound side, scopes per API and operation. Southbound credentials come from a secrets store, never from mapping files. One deployment can serve several tenants, each with its own mapping set if needed.

**Claire:** Volumes?

**Marc:** Catalog reads peak around 300 requests per second, p95 under 300 ms excluding backend time. Order creation peaks at 50 per second, each fanning out to two or three backend writes. Alarms peak at 2,000 events per second during an outage storm. It runs on our Kubernetes platform and scales horizontally.

**Claire:** How do I know it's really TMForum-compliant?

**Marc:** We run the TMForum Conformance Test Kit for all three APIs in CI, and every mapping ships with contract tests against recorded backend responses.

**Claire:** Last thing. Who maintains the mappings?

**Marc:** Integration analysts, not developers. Hand-editing configuration files won't work for them, so they need a GUI: see the TMF schema next to the backend schema, map fields visually, write expressions with help, design flows, and try a mapping on sample payloads before it goes live. Changes need a draft, validate, publish cycle with history, diff and rollback, and they want to see where each TMF field comes from.

**Claire:** So the adapter keeps state? I was told adapters should be stateless.

**Marc:** Runtime instances should be interchangeable, so any instance can serve any call. But some state must survive restarts and be shared across instances: the id map between TMF ids and backend keys, hub subscriptions, draft and published mapping versions, order correlation and compensation progress, idempotency keys, and pending notifications. How and where that is stored is a design decision; we already operate PostgreSQL and Kafka, so a proposal reusing them is easier to run, but it has to justify the fit.

**Claire:** Stack?

**Marc:** Our teams build on Java 25 and Spring Boot 4 and deploy containers to Kubernetes with Helm. Anything else must earn its place, especially for the GUI, where we have no house standard yet.

## 3. Functional requirements

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-01 | Expose any TMF Open API selected by configuration, based on its official OpenAPI definition (v4.x baseline, v5 readiness). PoC set: TMF620 Product Catalog, TMF622 Product Ordering, TMF642 Alarm Management. | Must |
| FR-02 | Serve only the resources and operations enabled for a deployment; others return 501 with a TMF Error. | Must |
| FR-03 | Requests and responses conform to the TMF schema, including polymorphic `@type` / `@baseType` / `@schemaLocation`. | Must |
| FR-04 | Bidirectional mapping per operation: field mapping, computed fields, conditions, defaults, code-to-code lookups, arrays and nested objects. | Must |
| FR-05 | Stable correspondence between TMF ids and backend keys, surviving restarts. | Must |
| FR-06 | TMF conventions absent from backends are produced: `id`, `href`, `@type`, `lifecycleStatus`, `validFor`, `lastUpdate`. | Must |
| FR-07 | One TMF operation can combine several backend calls: in sequence, in parallel with merged results, conditionally, or per item of a collection, with per-call timeout and retry. | Must |
| FR-08 | A write spanning several backends that fails part-way is compensated; progress survives an instance failure and is traceable. | Must |
| FR-09 | TMF query support: `fields`, `offset`, `limit`, attribute filtering, `sort`; filtering delegated to the backend when it can, with a bounded fallback otherwise; correct `X-Total-Count` / `X-Result-Count`. | Must |
| FR-10 | PATCH with `application/merge-patch+json`; `json-patch+json` optional per API. | Must |
| FR-11 | TMF622 order lifecycle: create, retrieve, list, PATCH, delete where allowed, `cancelProductOrder` task; TMF order states driven by backend responses and callbacks. | Must |
| FR-12 | Repeated creates with the same idempotency key or `externalId` return the original resource and cause no new backend write. | Must |
| FR-13 | Asynchronous backend updates (callbacks, Kafka) are correlated to TMF resources and produce state changes and TMF events. | Must |
| FR-14 | TMF642 task operations: acknowledge, unacknowledge, clear, comment, group, ungroup alarms. | Must |
| FR-15 | Event hub (`/hub` register / unregister) per API; subscriptions survive restarts; notifications delivered at least once, with retry and dead-letter handling. | Must |
| FR-16 | Southbound protocols needed for the PoC: REST/JSON, SOAP/XML, read-only SQL, Kafka, inbound webhooks. New backend endpoints are configuration; a new protocol may need code. | Must |
| FR-17 | Backend errors are mapped to the TMF Error object and HTTP status by configuration; business validation errors to 400 / 422; no internal detail leaks. | Must |
| FR-18 | Mapping GUI for analysts: TMF schema beside the backend schema (OpenAPI, WSDL, JSON sample or table), visual field mapping, assisted expression editing, flow design including compensation, editing of lookup tables and backend endpoints. | Must |
| FR-19 | Mapping lifecycle: draft, validate, test, publish, rollback; version history and diff; a publication reaches every running instance without restart. | Must |
| FR-20 | Trying a TMF request against a draft mapping shows each backend call and the resulting TMF response, without side effects on real backends. | Must |
| FR-21 | GUI access through the company's OIDC login, with at least viewer, author and publisher roles, and an audit trail of every change. | Must |
| FR-22 | Field-level lineage visible in the GUI: for each TMF field, its backend source and transformation. | Should |
| FR-23 | Mapping sets can be exported and imported in a readable text form, for review in Git and promotion between environments. | Should |
| FR-24 | Multi-tenancy: tenant resolved from token or header; mapping sets selectable per tenant. | Should |

## 4. Non-functional requirements

| ID | Area | Target |
| --- | --- | --- |
| NFR-01 | Read performance | TMF620: 300 req/s sustained, adapter overhead p95 < 300 ms (backend time excluded). |
| NFR-02 | Write performance | TMF622 create: 50 orders/s peak, synchronous `acknowledged` response p95 < 1 s (backend time excluded). |
| NFR-03 | Event throughput | TMF642: 2,000 backend events/s peak, end-to-end notification latency p95 < 2 s. |
| NFR-04 | Consistency | No lost or duplicated order under instance failure; no lost notification; listeners can deduplicate on `eventId`. |
| NFR-05 | Scalability | Horizontal scaling with interchangeable instances; a publication reaches every instance within 30 s. |
| NFR-06 | Availability | 99.9 % monthly; a failing backend degrades only the operations that depend on it. |
| NFR-07 | Security | OAuth2/OIDC northbound, scopes per API and operation; backend credentials never stored in mappings or exports; encrypted traffic between services. |
| NFR-08 | Observability | One trace per northbound call covering every backend call; rate, error and latency metrics per API, operation and backend; correlation id in all logs; visibility of pending compensations and notifications. |
| NFR-09 | Conformance | TMForum CTK for TMF620, TMF622 and TMF642 passes in CI for the enabled operations. |
| NFR-10 | Testability | Each mapping can be tested automatically against recorded backend responses; coverage visible to analysts. |
| NFR-11 | Usability | An analyst can add a backend field to a TMF attribute, test and publish it in under 30 minutes, without developer help. |

**Technology context (customer habits, not prescriptions)**

| Area | What the customer runs today |
| --- | --- |
| Language and framework | Java 25, Spring Boot 4 |
| Runtime | Kubernetes, container images, Helm |
| Data and messaging | PostgreSQL, Kafka |
| Security | Central OIDC provider, secrets vault |
| Observability | OpenTelemetry-based tracing and metrics |
| Front end | No house standard |

## 5. Representative mapping scenarios

An analyst must be able to express each scenario below through the GUI, without code. How mappings and flows are represented and stored is left to the solution.

| Scenario | TMF side | Backend side | The mapping must express |
| --- | --- | --- | --- |
| Read an offering | `GET /productOffering/{id}` | Catalog `GET /offers/{code}`, pricing SOAP `GetOfferPrices`, catalog `GET /families/{id}` | Id translation, parallel calls merged into one resource, status lookup (`ACT` → `Active`), date conversion, price list to `productOfferingPrice[]`, backend 404 to TMF Error |
| List offerings | `GET /productOffering?lifecycleStatus=Active&fields=id,name` | Catalog filters by status, not by category | Filter delegation where supported, bounded fallback otherwise, field selection, correct counts |
| Create an order | `POST /productOrder` | OMS `POST /orders`, then billing SOAP `OpenChargeContext` | Ordered writes, cancel the OMS order if billing fails, idempotency, `acknowledged` response |
| Track an order | `ProductOrderStateChangeEvent` to listeners | OMS callback with status `IN_PROGRESS`, `DONE` or `REJECTED` | Correlation by OMS order number, status to TMF state (`inProgress`, `completed`, `failed`) |
| Cancel an order | `POST /cancelProductOrder` | OMS cancel, billing SOAP `CloseChargeContext` | Multi-step write with compensation, TMF task state |
| Publish alarms | `AlarmCreateEvent`, `AlarmAttributeValueChangeEvent` | Kafka topic `fm.alarms.v1`, `kind` = `RAISED` or `CLEARED` | Routing by event content, backend event to TMF Alarm |
| Acknowledge alarms | `POST /ackAlarms` | Fault REST ack, one call per alarm | Loop over a collection, partial failures reported in the TMF response |

## 6. Scope, acceptance and open points

**Out of scope**

- Changes to backend systems.
- Business logic beyond translation and orchestration, such as pricing rules, eligibility or order decomposition.
- TMF APIs other than TMF620, TMF622 and TMF642 in the PoC (but adding TMF637 must be configuration only).
- Multi-region disaster recovery for the adapter datastore (PoC: single-region HA).

**Acceptance criteria**

1. `GET /productOffering/{id}` returns a schema-valid ProductOffering merged from the catalog and pricing backends.
2. `GET /productOffering?lifecycleStatus=Active&fields=id,name&limit=20` returns only the requested fields and correct count headers.
3. `POST /productOrder` returns 201 with state `acknowledged`; the order exists in the OMS and a charge context in billing.
4. Repeating the same `POST /productOrder` with the same idempotency key returns the original order and creates nothing new in the OMS.
5. If billing fails after the OMS succeeded, the OMS order is cancelled by compensation, the client gets a TMF Error, and each step of the compensation is traceable.
6. An OMS completion callback moves the TMF order to `completed` and a `ProductOrderStateChangeEvent` reaches every registered listener.
7. `POST /cancelProductOrder` cancels the OMS order and closes the billing charge context.
8. A backend 404 or SOAP fault returns a TMF Error object with no internal detail.
9. A registered TMF642 listener receives an `AlarmCreateEvent` within 2 s of the backend Kafka event.
10. Task operations such as `POST /ackAlarms` and `POST /clearAlarms` reach the fault system and the alarm reflects the new state.
11. In the studio, an analyst maps a new backend field to a TMF attribute, tests it in dry run and publishes it; every instance serves it within 30 s without restart. An invalid mapping cannot be published, and rollback restores the previous version.
12. Killing a pod during order creation loses no order, duplicates no backend write and loses no event.
13. TMF CTK suites for TMF620, TMF622 and TMF642 pass for the enabled operations.
14. Enabling a fourth API (TMF637 Product Inventory, read operations against a mock backend) needs zero Java changes.

**Open questions**

- How should mapping versions, id maps, subscriptions and in-flight orders be stored, and is one store right for all of them?
- How are mappings and flows represented, and which expression language do analysts use?
- Does the adapter keep a full copy of each TMF ProductOrder for reads, or rebuild it from the OMS on every GET?
- Should publication require four-eyes approval (author and publisher must differ)?
- What bounds the in-memory filtering fallback: a fixed cap or a per-operation setting?
- Do we expose TMF v4 and v5 side by side, or v4 only in the PoC?
- Event ordering: per resource id only, or global?

**Glossary**

| Term | Meaning |
| --- | --- |
| TMF Open API | TMForum's standard REST APIs for telco business and operations domains. |
| SID | TMForum's Information Framework, the shared data model behind the APIs. |
| ODA | Open Digital Architecture, TMForum's component-based reference architecture. |
| CTK | Conformance Test Kit published by TMForum for each Open API. |
| Northbound / southbound | Facing API consumers / facing backend systems. |
| Saga | Sequence of local transactions with compensating actions instead of a distributed transaction. |
