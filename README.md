# TMF-Bridge

A configurable adapter that exposes TM Forum Open APIs to partners and translates each call into calls on legacy backends that know nothing of TMF or SID. Mappings, backends and API definitions are data, not code: adding an API or changing a field mapping needs no rebuild.

> **Status: planning.** This repository holds the product and architecture documents for the first slice (S1). There is no application code yet, so nothing to build or run. Epics are all in `backlog` ([sprint-status.yaml](_bmad-output/implementation-artifacts/sprint-status.yaml)).

## What it does

- **Serves TMF APIs from configuration.** TMF620 (Product Catalog) and TMF622 (Product Ordering) in S1, driven by the official OpenAPI files stored in the database.
- **Translates with mappings.** Each operation has a request rule table and a response rule table built from a closed set of expression nodes (`path`, `lit`, `concat`, `coalesce`, `if`, `lookup`, `convert`, `idMap`, `map`, `filter`). TMF ids and backend keys are linked through an id map.
- **Validates at runtime.** Requests and translated responses are checked against the OpenAPI definition. An invalid response is never returned. Backend errors become a generic TMF Error with a fixed HTTP status, with no internal detail leaked.
- **Ships a Mapping Studio.** A keyboard-accessible web GUI where analysts author, test, version and apply mappings, manage backends and the id map, and get plain-language verdicts with field lineage.
- **Treats configuration as code.** Whole-config export and import are deterministic and atomic, for review in Git and promotion between environments.

The bridge is designed to be retired: when a backend goes native, the operator keeps the mappings and id map as portable text.

## Architecture in brief

- Interpreter runtime with hexagonal layering, one Maven module (`tmf-bridge-api`), Java 25, Spring Boot 4.1.x on virtual threads, PostgreSQL.
- **Data plane** serves TMF calls from published mapping versions. **Control plane** (Spring profile `control`) serves the studio API, drafts, import/export, test and preview. Both live in one deployable.
- Three images: `tmf-bridge-api`, `studio-frontend` (React, TypeScript, Vite), `tmf-mock` (WireMock).
- Studio tests and production calls run the same `CallPipeline`. Test mode allows GET only.
- Secrets are stored by name only and resolved from environment or files.

Full rules (AD-1..AD-17) and the pinned stack are in the [architecture spine](_bmad-output/planning-artifacts/architecture/architecture-tmf-bridge-2026-11-16/ARCHITECTURE-SPINE.md).

## Scope of S1

S1 is a synchronous pipeline: request table, backend call, response table. It covers REST/JSON backends only. It does not include retries, sagas, SOAP, SQL or Kafka backends, event hubs, or TMF642 alarms. Those appear in the [original brief](docs/Project%20Brief%20%E2%80%94%20TMForum%20Open%20API%20Adapter.md) and are later work. Where the brief and the spec disagree, the spec wins.

## Documents

Read in this order:

| Document | Purpose |
| --- | --- |
| [SPEC.md](_bmad-output/specs/spec-tmf-bridge/SPEC.md) | Canonical contract for S1 |
| [Architecture spine](_bmad-output/planning-artifacts/architecture/architecture-tmf-bridge-2026-11-16/ARCHITECTURE-SPINE.md) | Binding architecture decisions, stack, structure |
| [PRD](_bmad-output/planning-artifacts/prds/prd-tmf-bridge-2026-11-09/prd.md) | FR-1..FR-39, NFR-1..NFR-6 |
| [EXPERIENCE.md](_bmad-output/planning-artifacts/ux-designs/ux-tmf-bridge-2026-11-11/EXPERIENCE.md), [DESIGN.md](_bmad-output/planning-artifacts/ux-designs/ux-tmf-bridge-2026-11-11/DESIGN.md) | Studio behavior and visual design |
| [epics.md](_bmad-output/planning-artifacts/epics.md) | Eight epics, from TMF620 serving to the zero-to-hero showcase |
| [Product brief](_bmad-output/planning-artifacts/briefs/brief-tmf-bridge-2026-11-02/brief.md) | Product framing |

Planning artifacts were produced with the [BMad method](https://github.com/bmad-code-org/BMAD-METHOD). Tooling config lives in `_bmad/`, outputs in `_bmad-output/`.

## Getting started

Not available yet. The target is a working Docker Compose showcase (API, studio, mock backend, Keycloak, PostgreSQL) within ten minutes of `git clone`, planned in Epic 8. This section will be filled in when that lands.

## Disclaimer

TMF-Bridge validates payloads against the official TM Forum OpenAPI definitions at runtime. It makes no claim of conformance, compliance or certification by TM Forum. The final Disclaimer page will be linked from studio help once the studio exists.
