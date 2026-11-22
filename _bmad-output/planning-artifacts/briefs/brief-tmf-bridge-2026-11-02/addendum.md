# Addendum: TMF-Bridge

Depth that belongs in the PRD and architecture, not the brief.

## Source input

`Project Brief — TMForum Open API Adapter.md` (2026-10-21) is, by the author's own judgement, a preliminary PRD. Treat it as raw input to the PRD workflow, not as an approved spec. It holds 24 functional requirements, 11 non-functional requirements, 7 mapping scenarios, 14 acceptance criteria and 7 open questions.

## Reprioritisation the PRD must apply

- The input marks 21 of 24 FRs as Must and calls the work a PoC. The author's intent is a full platform seeded by a small slice. Re-tier: Must for the first slice, Should or Later for the rest.
- First slice drives epics: TMF620 read path, one mapping, minimal studio, no-restart configuration, runtime OpenAPI validation, mock backends.
- FR-23 (mapping export/import as text) moves from Should to Must (exit path), and is extended into configuration as code: studio authoring, export to structured text, Git as recommended source of truth, apply through a REST API. GitOps is a best practice, not an enforced rule; editing production directly is allowed but shows as drift. Needed outcomes: versioned configuration, traceability, drift detection and drift correction. This reshapes the input's in-place draft/publish cycle (FR-19): rollback becomes a Git revert plus re-apply, and the PRD decides what lifecycle features stay in the studio.
- Everything is API-based: studio, import, export and dry run are clients of the same REST APIs.
- FR-22 (field lineage) moves from Should to core. Add requirements for redacted access logs, mapping-step results and performance metrics (extends NFR-08).
- Frame epics by capability (id mapping, saga, retry, async correlation, TMF extensions, hub), using TMF620/622/642 as test examples rather than as the unit of delivery.
- Input NFRs (300 req/s, 50 orders/s, 2,000 events/s, 99.9% availability) have no stated basis. Treat as hypotheses to confirm or drop. Not first-slice concerns.
- Input acceptance criterion 14 (a fourth API with zero Java changes) is a strong test of the wedge. Honest framing needed: polymorphism, v4/v5 differences, task operations and hub semantics may still need code.

## Challenges raised and open for the PRD

- Dry-run "no side effects" conflicts with write flows hitting real backends. Needs a mechanism (stubbed backends in dry run).
- "Stateless runtime" versus durable state (id map, subscriptions, versions, idempotency, order correlation). Whether to copy the full ProductOrder is the adapter-versus-system-of-record decision.
- Studio is the riskiest component. Expression language, flow model, and non-developers authoring saga compensation are unproven.
- Build-vs-buy rationale: stakeholders want one proven product, not a patchwork of generic tools. User's inference, not evidence.

## Unverified facts (research, 2026-11-09)

- CTK licensing: whether open-source projects can use it, fees, redistribution of test descriptions. User has TM Forum portal access; check terms before publishing test descriptions in the repo.
- v5 CTK availability for TMF620, TMF622, TMF642.
- Trademark naming rules for "TMF" in a project name. User states many OSS projects do it; a disclaimer is still advisable.
- GitHub topic search for competing open-source TMF adapters.
- Why certification count is low.

## Landscape digest (snippet-based, partly unverified)

| Product | Licence | Note |
| --- | --- | --- |
| TM Forum API specs and reference implementations | Apache 2.0 | Stubs and demos. No mapping to legacy. |
| ODA Canvas (TM Forum) | Open source | Runtime for ODA components, not an adapter. |
| ETSI OpenSlice TMF API | Apache 2.0 | Spring and Camel. Own domain model. Closest overlap. |
| Commercial TMF adapters and templates | Commercial | Tied to one product or integration platform; some cover TMF622 only. |

## Stack context from the input (not decided)

Customer habits: Java 25, Spring Boot 4, Kubernetes with Helm, PostgreSQL, Kafka, OIDC, OpenTelemetry. No front-end standard. These are inputs to architecture, not prescriptions.
