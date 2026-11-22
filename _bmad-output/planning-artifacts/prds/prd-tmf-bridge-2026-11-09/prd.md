---
title: "PRD: TMF-Bridge"
status: final
created: 2026-11-09
updated: 2026-11-20
---

# PRD: TMF-Bridge

## Vision

TMF-Bridge is a lightweight adapter that eases TM Forum Open API adoption at telco grade during a legacy backend transformation. It lets operators expose stable TMF APIs to partners now, validated at runtime against the official OpenAPI definitions, while the backends behind them are upgraded to implement TMF natively. Its competitive advantage is configuration over code: mappings and flows are configuration, not code to build and redeploy. A zero-code new API is the goal. Polymorphism, version differences, task operations and hub semantics may still need code.

TMF-Bridge is built to be retired. When a backend exposes TMF natively, the operator drops the bridge and keeps the mappings and the id map, both as portable text.

Day-one users are telco integrators. Partners are the northbound API callers. The first slice proves, for integrators, that a TMF API, validated at runtime against the official OpenAPI definition, can be exposed from configuration without code. The mapping the bridge performs is the proof. "Stable" means a partner's contract does not change when the backend is later replaced by a native TMF one.

## User Journeys

**UJ-1: Inès adds a field to an existing mapping.** Inès, an integration analyst, is not a developer. The TMF620 product offering is already mapped to the legacy catalog. Partners ask for one more attribute. She opens the mapping in the studio. Editing starts a new version and lands on Map request, with Pick API and Declare backend still editable. Map response is one step away. There she maps the backend field to the TMF attribute in the mapping table, by drag and drop or the keyboard-friendly Connect picker. In Test she checks the mapping on her saved sample (sub-steps 1 and 2), then makes a read-only call (sub-step 3). The verdict says checks passed, and the Exchange view shows the new value and its lineage. In Apply the summary says "This replaces the live version". She confirms and sees "Version 2 is live". The running API serves the new field on the next call, with no restart and no developer.

**UJ-2: Tomás calls TMF620, then TMF622.** Tomás, a partner developer, calls `GET /productOffering/{id}`. The response is schema-valid, leaks no legacy internals, and adds under 100 ms of bridge overhead. He then calls TMF622, which is not yet mapped. He gets `501` with a TMF Error, not a crash or an empty body.

**UJ-3: Inès maps TMF622.** Inès adds the TMF622 product order API, starting with `GET /productOrder/{id}`. She works through the six studio steps. Pick API: she picks TMF622 and the operation from the official OpenAPI list. Declare backend: she selects an existing backend, or adds one with its endpoint and the name of a secret that tech people provisioned, never a value. Map request: she maps the operation's inputs (query, header and JSON body) to the backend request, and adds the id-map transform when the studio warns about the id field. Map response: she maps the backend response to the TMF payload. Test has three sub-steps: an unfired request-mapping preview on a pasted sample, an unfired response-mapping preview, then a real read-only call that opens the Exchange view. Apply: she confirms the summary and sees "Version 1 is live". The API serves with no restart and no code.

**UJ-4: Tomás calls TMF622 and it works.** Tomás repeats his `GET /productOrder/{id}` call. It now returns a schema-valid response within the overhead budget. His contract is unchanged, so a later backend swap needs no change on his side.

**UJ-5: Inès hits a 404 in the Exchange view and forwards the call to tech.** Inès runs the real read-only call (Test, sub-step 3) for order ORD-1, taken from her saved sample. The Exchange view opens with a failed verdict and the backend `404` response expanded. The verdict says the backend could not find the order, that the request sent the id without the id-map transform, to try adding it, and otherwise to ask her tech team. She jumps to the mapping row, adds the transform and re-runs. It still returns `404`: the id is not in that backend. She downloads the share bundle and copies the call as curl (or Postman). She sends them to her tech colleague. The bundle holds the mapping version, samples, request, response, verdict and checks, and secret name, never a secret value, so the colleague replays the call from the masked samples without asking her anything. There is no send integration: she forwards the file herself.

These journeys are in the first slice. The demo maps `GET /productOrder/{id}` as a read.

## Features and Requirements

Tier **S1** is the first slice, Later is planned after it. Slices follow capability, not API set. TMF620 and TMF622 reads are the test beds.

### F1. Runtime serving
- **FR-1 (S1)** The runtime serves each configured TMF API from its official OpenAPI definition.
- **FR-2 (S1)** An unconfigured operation returns `501` with a TMF Error body. A configured operation with an unknown id returns `404` (the backend's `404`, FR-4). An operation whose mapping has no Live version, including a Draft-only mapping, returns `501`.
- **FR-3 (S1)** Configuring a resource makes it answer without a restart.
- **FR-4 (S1)** Backend failures (not found, HTTP and transport faults, validation errors) map to a TMF Error with a fixed HTTP status. Backend `404` returns `404`. Any other backend `4xx` or `5xx` returns `502`. A transport fault returns `502`. A backend timeout returns `504`; the default timeout is 5 seconds, with an optional per-backend value, and there are no retries in S1. A translated response that fails OpenAPI validation returns `502` and is never returned to the caller. A mapping error at runtime returns `502`, including a missing id-map entry in either direction (FR-5). An invalid request returns `400`. Error bodies carry no backend internals. Configurable status mapping is Later.

### F2. Mapping
- **FR-5 (S1)** An operation binds to a mapping from backend fields to TMF attributes, including value transformation. S1 supports a closed list: direct copy or rename, constant, concatenation, conditional or default value, lookup table, type or format conversion (dates, numbers), id-map (translate a TMF id to a backend id and back, using the id map of FR-6), and map and filter over arrays and nested objects (map transforms each item; filter keeps the items that match a condition). The studio shows an array rule as a nested sub-table, and lineage and step results trace nested rules per item. Anything else is Later.
- **FR-6 (S1)** The runtime maps ids between TMF resources and backend records. It keeps this id map in its own database, so the runtime is stateful. The studio seeds it through the id map screen (FR-39). The id-map transform (FR-5) is how a mapping uses it: the map is runtime state, the transform is mapping configuration. The studio warns, without blocking, when an id field is mapped without the transform. An id-map transform with no matching entry, forward or reverse, is a mapping error and returns `502` (FR-4); the verdict says the entry is missing, so it is not mistaken for a backend `404`.
- **FR-7 (S1)** The runtime supports the TMF extension attributes `@type`, `@baseType` and `@schemaLocation`. Full polymorphism is out of S1.

### F3. Conformance
- **FR-8 (S1)** Every request and response is validated at runtime against the official OpenAPI definition.
- **FR-9 (S1)** No certification claim is made. Conformance means runtime schema validation only. The studio never claims conformance: the verdict (FR-29) says only that checks passed against the sample and the schema, and the Disclaimer is reachable from the studio help.

### F4. Mapping Studio
- **FR-10 (S1)** An analyst creates and edits a mapping in a minimal studio, with no code. Steps: Pick API (TMF API and operation from the official OpenAPI list), Declare backend, Map request, Map response, Test (three sub-steps, FR-28), Apply. The mapping is a table (TMF field, transform, backend field); drag and drop is only a visual layer over it. Editing an existing mapping lands on Map request, with earlier steps editable.
- **FR-11 (S1)** An analyst tests a call from the studio and sees the response with its field lineage, in the exchange view (FR-30).
- **FR-12 (S1)** An analyst applies a mapping to the running bridge from the studio. Apply shows three states: Ready (summary of what goes live, "This replaces the live version"), concurrent divergence (FR-32), and live confirmation ("Version {n} is live").
- **FR-13 (S1)** Test calls from the studio never change backend state. In S1 they are reads against the backend or the mock.
- **FR-28 (S1)** Unfired translation preview. Test has three sub-steps: (a) request-mapping sample test, showing the translated output, which is NOT sent to the backend; (b) response-mapping sample test; (c) a real read-only call to the backend or the mock, which looks the same from the bridge's view. This is distinct from the Later dry-run (write flows, stubbed backends). Sample request and response data is kept in the browser only. The studio sends a sample to the server to translate it, and the server never stores it.
- **FR-29 (S1)** Verdict bar. Every studio test shows a plain-language, rules-based verdict: what happened, likely meaning, what to try, who to ask. The S1 rule set is closed: HTTP status, id-map transform present, id-map entry found, empty mapped fields, OpenAPI validation outcome, backend reachable (unreachable or timed out), secret name provisioned (FR-26), and bridge overhead against the NFR-1 target. A failed backend-reachable or secret check gives a failed verdict. Overhead above target is an info note beside a pass: the studio reports it and never blocks. Rules apply per Test sub-step (FR-28). Id-map transform present, id-map entry found and empty mapped fields apply to (a), (b) and (c). OpenAPI validation applies to the request side in (a), the response side in (b), and both in (c). HTTP status, backend reachable, secret name provisioned and overhead apply to (c) only. A rule that does not apply is skipped, not shown as passed. A pass reads "Checks passed. The output matches your sample and the TMF622 schema." and never claims conformance (FR-9).
- **FR-30 (S1)** Exchange view. For each studio test call the analyst sees four summary tiles (bridge overhead, FR-22, shown as the exchange latency tile; target endpoint; fields translated; id-map status), the mapping result, the HTTP request sent to the backend, the HTTP response, and the resulting TMF payload with field lineage (FR-21), in three panels. A failed verdict (for example a backend `404`, FR-4) opens the HTTP response panel and links to the mapping row at fault. FR-23 redaction applies. Secret names are shown, values never. The analyst can copy as curl, copy as Postman (collection JSON on the clipboard), and download a share bundle. The bundle holds the mapping version (deterministic JSON), the sample request and response with the FR-19 sensitive field paths masked, the Exchange view record (backend request, backend response, resulting TMF payload), the verdict and its checks, the secret name, the Postman collection and a timestamp. The exact file layout is an architecture decision. Exports use a `$SECRET_VALUE` placeholder, never a secret. Copy and download only, no send integration.
- **FR-31 (S1)** A mapping has a version number and a status of Draft (never applied) or Live (an applied version exists). S1 applies by overwrite: Apply replaces the live version. History, diff and rollback are Later.
- **FR-32 (S1)** Optimistic locking. The configuration and apply API checks a version or entity tag. Apply on a stale version is blocked: "Someone else changed this mapping. Reload the latest version before you apply." Autosave of a draft is refused in the same way when its version or entity tag is stale. The user's unsaved edits are kept, and the studio shows the same message. Diff and history are Later (FR-31).
- **FR-33 (S1)** The studio autosaves mapping drafts on change and offers "Save now". The configuration API stores drafts in the runtime database. A Draft is stored and never served. Live is the applied version, which the runtime serves. Editing a Live mapping creates a Draft of the next version, and Apply promotes it.
- **FR-34 (S1)** The studio manages named backends: endpoint, secret name reference, authentication type (bearer, basic or api-key; not secret), and an optional legacy OpenAPI upload. The upload belongs to the backend, not to a mapping. S1 backends are REST and JSON only. WSDL and SOAP are Later. The secret name is chosen from a picker of the names provisioned on the server (FR-26), never typed; the picker lists names only. A backend is configuration of its own, decoupled from mappings: a mapping holds only the backend's name. Editing a backend's endpoint or secret name creates no mapping version and needs no Apply, and the runtime resolves the backend by name at call time, so mappings that use it pick up the new values on their next call. The edit shows which mappings use the backend. Backends are stored in the runtime database. A backend cannot be deleted while mappings use it, and the block lists those mappings.
- **FR-35 (S1)** The mappings list is sorted A-Z by name by default and every column is sortable. It has no recency column and no last-opened behavior. A mapping is one per TMF operation, and its name is unique in the deployment. Import matches a mapping to an existing one by TMF API definition, method and path, not by name.
- **FR-37 (S1)** Documentation. The studio ships static user-manual docs, served by the studio itself, with a glossary of technical terms (payload, field path (JSON Pointer), transform, lineage, OpenAPI, id map, secret name, verdict, unfired preview, mapping version). Every help marker in the studio links to the matching manual page (NFR-5). Writing the content is an S1 deliverable. Content owner: the project team, delivered as a story.

### F5. Administration and configuration as code
- **FR-14 (S1)** Everything the studio does is available through the same REST APIs: configure, apply, import, export and test. Import and export are available in the studio as well as through REST.
- **FR-15 (S1)** The whole configuration exports as structured JSON and imports back through the REST API and the studio ("Export all", "Import"). This is the exit path. It covers mappings (Draft and Live, each with its status and version), the TMF OpenAPI definitions in use with their full text, backends and the id map entries.
- **FR-38 (S1)** One mapping exports on its own, from the studio and the REST API, in addition to FR-15 and never replacing it. A per-mapping export carries the Live version when one exists, so a Draft of a later version is not exported with it; a mapping never applied exports its Draft. It carries the mapping with its status and version, including its id-map transform (FR-5), and never id-map entries: entries are runtime state (FR-6) and travel only in the whole-configuration export. It names the backend it uses and carries no backend definition.
- **FR-39 (S1)** Id map screen. The studio has a screen to view, add, edit, delete and import id-map entries (TMF id and backend id, per resource), so the transform of FR-5 has data to translate. A clash is the same resource and TMF id with a different backend id, or two different TMF ids of the same resource mapped to the same backend id, so a reverse lookup has one answer. Clashes are checked on add, edit and import. Adding or importing an identical entry is accepted and changes nothing. Imported entries are additive, and a clash rejects the whole file (FR-17). Edit and delete take effect on the next call, with no Apply, because entries are runtime state (FR-6); delete asks for confirmation. The same operations exist through REST (FR-14). For the demo, infrastructure provisioning may seed the entries.
- **FR-16 (S1)** Exports contain secret names (references), never secret values. The same holds for every export: whole, per mapping and share bundles. Sensitive field paths (FR-19) are masked in samples and bundles. FR-18 determinism applies to the configuration exports only; a share bundle carries a timestamp. Copied curl and Postman use a `$SECRET_VALUE` placeholder (FR-30). A backend endpoint with credentials embedded in the URL, in its user-info part or in credential-like query parameters, is rejected when the backend is saved or imported; credentials are declared as a secret name.
- **FR-17 (S1)** Import is atomic. An invalid file is rejected as a whole and the running configuration stays unchanged, and the rejection lists each problem with its location. Imported mappings always land as Draft, whatever status the file carries, and are never applied. The analyst reviews them, then applies (FR-12). Import never changes the live configuration: it never overwrites or removes a Live mapping, a backend or an id-map entry. When a mapping already has a Draft, the import request states `replace` or `skip` for it; without that choice the import is refused with a conflict listing those mappings. `replace` overwrites that Draft; `skip` leaves it and does not import that mapping. Backends and id-map entries are added only when new. An identical backend (same name and definition) or an identical id-map entry is skipped, so re-importing an export into the same runtime succeeds. A backend with the same name and a different definition, or an id-map clash as defined in FR-39, makes the file invalid and it is rejected. A mapping that names a backend found neither in the file nor in the runtime also makes the file invalid. A mapping with no Draft, including a Live one, imports as the Draft of its next version (FR-33).
- **FR-18 (S1)** Export is deterministic, whole and per mapping: identical stored state yields identical text, with no export timestamps or generated ids, so exports diff cleanly. Drafts are covered. A draft edit changes the state, and therefore the text.

### F6. Observability and lineage
- **FR-19 (S1)** Access logs are redacted of sensitive data. Sensitive data is every secret value plus the values at a configurable list of field paths written as JSON Pointers (for example `/password`, `/resource/secret/value`).
- **FR-20 (S1)** On a studio test call, the result of every mapping step is available. Normal runtime calls return no step results.
- **FR-21 (S1)** On a studio test call, each TMF field traces to its backend source and transformation. Lineage is not produced on normal runtime calls.
- **FR-22 (S1)** Bridge overhead is measured and reported for every call. It is the time the bridge adds: total call time minus the time of the outbound backend call as the bridge sees it, and excluding the lineage and step-result capture of studio test calls (FR-20, FR-21). Runtime responses carry the overhead in a `Server-Timing` header, and the studio reads the same value. Conditions are in NFR-1.
- **FR-23 (S1)** Step results and lineage follow the same redaction rule as access logs (FR-19), including the configurable field paths.

### F7. Security
- **FR-24 (S1)** Studio and admin API users authenticate with OIDC.
- **FR-25 (S1)** Runtime API calls require an OIDC bearer token, whatever the access path. This is configurable.
- **FR-26 (S1)** The bridge manages authentication to legacy backends. A backend declaration holds only a secret name. Tech people provision the secret value outside the studio, and the bridge resolves the name to the value at call time. The runtime lists provisioned secret names to the studio, never values. Studio, exports and logs never hold values.
- **FR-36 (S1)** No roles or permission states in S1. Any authenticated OIDC user can do everything. Roles are Later.

### F8. Mock backend
- **FR-27 (S1)** A dedicated mock backend ships with the product for demos. It backs a zero-to-hero promise to open-source users: after cloning from Git, they get a working showcase in ten minutes. This is a promise, not a hard metric.

### Later capabilities
Saga compensation, retry and timeout, asynchronous correlation, notification hub, idempotency, query and filter delegation, multi-tenancy, and mapping history, diff and rollback. Also:
- **Mapping studio extensions.** WSDL import and extending the verdict rule set.
- **Safe publishing.** A validate gate that blocks invalid mappings, dry-run (write-side: write flows run against stubbed backends, so nothing reaches a real backend; distinct from the S1 unfired translation preview, FR-28), roles and permissions (author and publisher; S1 has none, FR-36), and an audit trail.
- **API coverage.** TMF622 writes, PATCH and cancel, and TMF642.
- **Generated TMF conventions.** `id`, `href`, `lifecycleStatus` and similar fields.
- **Platform.** TLS, trace and correlation ids, and scale-out with consistency (no lost or duplicated order).

## Non-Functional Requirements

- **NFR-1** Bridge overhead is under 100 ms per API call, measured as in FR-22, at the 95th percentile. Measurement conditions: warm compiled-mapping and API-definition caches, single bridge instance, zero-latency mock backend, payload up to 50 KB, lineage and step-result capture excluded.
- **NFR-2 (Accessibility)** The studio meets WCAG 2.2 AA. Every function works by keyboard alone, and every drag action has a keyboard alternative (the Connect picker). No status or signal relies on color alone.
- **NFR-3 (Platform)** The studio is desktop web only, on current evergreen browsers (Chrome, Firefox, Edge, Safari). The supported minimum width is 1280 px, and the layout reflows at 200% and 400% zoom down to 320 CSS px (WCAG 1.4.10). English first.
- **NFR-4 (Themes and preferences)** The studio has light and dark themes. The theme follows the operating system, with a manual toggle. The theme choice and the Guided or Side-by-side mode are remembered per browser.
- **NFR-5 (Contextual help)** The studio offers inline tips, a glossary, and a link to the user manual (FR-37) from every help marker.
- **NFR-6 (Sample data)** Sample request and response data is saved in the browser only. The server processes a sample but never stores it (FR-28).
- Throughput, availability and event-rate targets from the source brief are dropped. They had no stated basis. Architecture elicits further NFRs, such as idempotency and confidentiality.

## Out of Scope

- Changes to backends.
- Business logic beyond translation and orchestration.
- Any claim of TM Forum certification.
- Real backends in v1. Mocks only. The studio can declare a backend, and a real one has the same shape as the mock from the bridge's view, but S1 claims and tests the mock only.
- A textual mapping expression language and free-form scripting. S1 transforms are the closed list of FR-5.
- Drift detection, drift correction and Git workflows. These belong to an outer process, which uses FR-14 to FR-18, FR-38 and FR-39 to read and write configuration. Because import never changes live configuration (FR-17), restoring a whole configuration into a populated runtime is Later.

## Success Measures

User journeys UJ-1 to UJ-5 are the success criteria. S1 succeeds when each runs end to end through the studio and the REST API, using configuration only, with no code written, built or redeployed. Numeric metrics are deferred.

## Disclaimer

TMF-Bridge is an independent open-source project. It is not affiliated with, endorsed by or certified by TM Forum. "TMF" and "TM Forum" are used only to describe API compatibility.
