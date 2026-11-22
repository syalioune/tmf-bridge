# TMF-Bridge S1: Solution Design

Companion to `ARCHITECTURE-SPINE.md` (the rules). This document explains the shape so a team can build without rereading every decision. If the two disagree, the spine wins.

## 1. What we are building

TMF-Bridge serves TM Forum Open APIs from configuration. Partners call a TMF API. The bridge translates the call into a REST/JSON call on a legacy backend, translates the answer back, and validates both sides against the official OpenAPI definition. Analysts author the translation in a studio, with no code and no restart.

S1 proves this for reads: `GET /productOffering/{id}` (TMF620) and `GET /productOrder/{id}` (TMF622), against a mock backend.

## 2. The shape

Two images that matter, one runtime database.

| Image | What it is |
| --- | --- |
| `tmf-bridge-api` | Spring Boot 4.1, Java 25, one Maven module. Serves TMF calls (data plane) and the config/studio API (control plane). |
| `studio-frontend` | React, TypeScript, Vite SPA with React Aria Components. A pure client of the config API. |
| `tmf-mock` | WireMock with legacy-shaped JSON stubs. Demo and tests. |

PostgreSQL holds everything the bridge keeps: specs, mappings, backends, the id map. Secrets are never stored. One ingress gives one origin for studio, config API and TMF paths.

## 3. Inside the API

Hexagonal. The domain is pure Java and knows nothing of Spring, HTTP or JPA.

```
domain/        evaluator, expression nodes, canonicalizer, redactor, CallPipeline, ports
application/   runtime/ (serve a call)   control/ (studio, config, import/export, test, preview)
adapter/in/    web/runtime, web/control, security
adapter/out/   persistence (JPA), backend (RestClient), secrets, observability
```

Rules that keep it honest, all checked by ArchUnit in CI: domain imports no framework, runtime and control never import each other, and the runtime path works with the `control` profile off.

The two planes share one thing: the domain `CallPipeline`. Production calls and studio test calls both go through it, so what the analyst sees is what partners get.

## 4. One call, step by step

1. Partner calls `GET /tmf-api/.../productOrder/ORD-1` with a bearer token.
2. The runtime chain checks the token (this chain can be switched off by property, the control chain cannot).
3. The catch-all handler matches the path against the stored specs. No spec or no Live mapping gives `501`.
4. The pipeline reads the Live pointer from the database, then loads the compiled mapping from cache (keyed by mapping and number, so it never goes stale).
5. The request rule table turns the TMF request into a backend request. An `idMap` node translates `ORD-1` to the backend id (read from the DB on every call).
6. The backend client resolves the secret by name and calls the backend. Timeout 5 s by default.
7. The response rule table builds the TMF payload.
8. The payload is validated against the OpenAPI schema. Invalid means `502`, never returned.
9. The response carries a `Server-Timing` header with the overhead.

Errors follow a fixed table: backend 404 gives 404, other 4xx or 5xx gives 502, timeout gives 504, transport failure gives 502. A missing id-map entry, forward or reverse, is a mapping error and gives 502, so it is never confused with a backend 404. Bodies are generic, the original error goes to redacted logs.

## 5. The mapping document

One JSON document per operation version, validated by a published JSON Schema, with `schemaVersion: 1`. Two rule tables, request and response. A rule is a target pointer and an expression tree.

```json
{ "target": "/name",
  "expr": { "op": "concat",
            "args": [ {"path": "/body/code"}, {"lit": " - "}, {"path": "/body/label"} ] } }
```

Nodes in S1: `path`, `lit`, `concat`, `coalesce`, `if`, `lookup`, `convert`, `idMap`, `map`, `filter`. There is no textual expression language. A reserved `expr` node leaves room for CEL later. Addresses are JSON Pointer, with a reserved `syntax` field for JSONPath later.

Studio test calls add two observers to the same evaluator: step results and lineage. They are off in production.

## 6. Persistence

| Entity | Rule |
| --- | --- |
| ApiSpec | Immutable, content-hashed. TMF620 and TMF622 v4 are seeded on first boot. |
| Mapping | One per operation. Holds the live pointer and at most one Draft. ETag is its lock revision. |
| MappingVersion | Draft is mutable. Applied versions are immutable (trigger plus test). Apply deletes nothing. |
| Backend | Mutable. Referenced by name. Deleting a used backend gives 409 with the list. |
| IdMapEntry | Unique on resource plus TMF id, and on resource plus backend id. |

JPA is used strictly: lazy associations, explicit JPQL fetches, no entity graphs, entities never leave the persistence adapter, UTC in the database. Flyway owns the schema. Only `control` instances (or a Helm job) migrate. Data-plane pods wait on readiness until the schema is current.

## 7. Studio

Six steps: Pick API, Declare backend, Map request, Map response, Test, Apply. Test has three sub-steps: request preview, response preview, then a real read-only call. The real call runs in test mode, where the backend client allows GET only and refuses anything else before the network. Preview is stateless and stores nothing, and sample data stays in the browser.

Each test ends in a rules-based verdict computed by the server. The studio never claims conformance.

Export and import: one canonical serializer, deterministic bytes, no generated ids. Import validates the whole file first, then commits in one transaction. Mappings land as Draft only.

## 8. Security

Spring Security with one filter chain per plane. The control and config chain is always authenticated by OIDC. The runtime chain requires a bearer token unless `tmfbridge.runtime.auth.enabled` is false. Secrets are names in config and values in environment variables or mounted files (file wins). One redactor sees every resolved value.

## 9. Delivery and operations

One image per component for every environment, differing only by configuration. Promotion is export then import. Actuator probes, Prometheus and OTLP. Postgres backup and HA belong to the operator, and the whole-configuration export is a second restore path. The demo is `docker compose up` with Postgres, API, studio, mock and Keycloak, working within ten minutes of cloning.

CI builds the three images and runs: ArchUnit, Testcontainers, per-node contract tests, export round trip, secret-leak grep, Spectral on the generated config OpenAPI, Playwright with axe-core.

## 10. Build order suggestion

1. Domain core: nodes, evaluator, canonicalizer, redactor, with contract tests.
2. Persistence adapter and Flyway schema, with the immutability trigger.
3. CallPipeline and runtime handler against the mock, TMF620 read.
4. Config API and import/export.
5. Studio shell and the six steps.
6. Test mode, preview, verdict, exchange view, share bundle.
7. Compose demo with Keycloak and seeder, TMF622 mapped through the studio.

## 11. Deliberately not decided

Textual expression language, JSONPath, configurable error mapping, retries and saga, idempotency, hub, Kafka, history and rollback, id-map restore, secret rotation, roles and audit, AI-assisted mapping, WSDL and other protocols, multi-tenancy, sizing. Each has a revisit condition in the spine.
