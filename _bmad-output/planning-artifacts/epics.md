---
stepsCompleted: ['step-01-validate-prerequisites', 'step-02-design-epics', 'step-03-create-stories', 'step-04-final-validation']
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-tmf-bridge-2026-11-09/prd.md
  - _bmad-output/planning-artifacts/prds/prd-tmf-bridge-2026-11-09/addendum.md
  - _bmad-output/planning-artifacts/architecture/architecture-tmf-bridge-2026-11-16/ARCHITECTURE-SPINE.md
  - _bmad-output/planning-artifacts/architecture/architecture-tmf-bridge-2026-11-16/solution-design.md
  - _bmad-output/planning-artifacts/ux-designs/ux-tmf-bridge-2026-11-11/DESIGN.md
  - _bmad-output/planning-artifacts/ux-designs/ux-tmf-bridge-2026-11-11/EXPERIENCE.md
  - _bmad-output/specs/spec-tmf-bridge/SPEC.md
---

# TMF-Bridge - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for TMF-Bridge (slice S1), decomposing the requirements from the PRD, UX Design and Architecture into implementable stories.

Source precedence (from SPEC.md): the PRD wins over the spine on requirements, the spine wins on architecture. Where UX copy differs from the PRD, the PRD wins (see "Known source conflicts" under Additional Requirements).

## Requirements Inventory

### Functional Requirements

FR-1: The runtime serves each configured TMF API from its official OpenAPI definition.
FR-2: An unconfigured operation returns `501` with a TMF Error body. A configured operation with an unknown id returns `404` (the backend's `404`, FR-4). An operation whose mapping has no Live version, including a Draft-only mapping, returns `501`.
FR-3: Configuring a resource makes it answer without a restart.
FR-4: Backend failures map to a TMF Error with a fixed HTTP status. Backend `404` returns `404`. Any other backend `4xx` or `5xx` returns `502`. A transport fault returns `502`. A backend timeout returns `504` (default 5 s, optional per-backend value, no retries in S1). A translated response that fails OpenAPI validation returns `502` and is never returned to the caller. A mapping error at runtime returns `502`, including a missing id-map entry in either direction (FR-5). An invalid request returns `400`. Error bodies carry no backend internals. Configurable status mapping is Later.
FR-5: An operation binds to a mapping from backend fields to TMF attributes, including value transformation. S1 supports a closed list: direct copy or rename, constant, concatenation, conditional or default value, lookup table, type or format conversion (dates, numbers), id-map (translate a TMF id to a backend id and back, using the id map of FR-6), and map and filter over arrays and nested objects. The studio shows an array rule as a nested sub-table, and lineage and step results trace nested rules per item. Anything else is Later.
FR-6: The runtime maps ids between TMF resources and backend records. It keeps this id map in its own database (stateful runtime). The studio seeds it through the id map screen (FR-39). The id-map transform (FR-5) is how a mapping uses it: the map is runtime state, the transform is mapping configuration. The studio warns, without blocking, when an id field is mapped without the transform. An id-map transform with no matching entry, forward or reverse, is a mapping error and returns `502` (FR-4); the verdict says the entry is missing, so it is not mistaken for a backend `404`.
FR-7: The runtime supports the TMF extension attributes `@type`, `@baseType` and `@schemaLocation`. Full polymorphism is out of S1.
FR-8: Every request and response is validated at runtime against the official OpenAPI definition.
FR-9: No certification claim is made. Conformance means runtime schema validation only. The studio never claims conformance: the verdict (FR-29) says only that checks passed against the sample and the schema, and the Disclaimer is reachable from the studio help.
FR-10: An analyst creates and edits a mapping in a minimal studio, with no code. Steps: Pick API, Declare backend, Map request, Map response, Test (three sub-steps, FR-28), Apply. The mapping is a table (TMF field, transform, backend field); drag and drop is only a visual layer over it. Editing an existing mapping lands on Map request, with earlier steps editable.
FR-11: An analyst tests a call from the studio and sees the response with its field lineage, in the exchange view (FR-30).
FR-12: An analyst applies a mapping to the running bridge from the studio. Apply shows three states: Ready (summary of what goes live, "This replaces the live version"), concurrent divergence (FR-32), and live confirmation ("Version {n} is live").
FR-13: Test calls from the studio never change backend state. In S1 they are reads against the backend or the mock.
FR-14: Everything the studio does is available through the same REST APIs: configure, apply, import, export and test. Import and export are available in the studio as well as through REST.
FR-15: The whole configuration exports as structured JSON and imports back through the REST API and the studio ("Export all", "Import"). It covers mappings (Draft and Live, each with status and version), the TMF OpenAPI definitions in use with their full text, backends and the id map entries.
FR-16: Exports contain secret names (references), never secret values, for every export: whole, per mapping and share bundles. Sensitive field paths (FR-19) are masked in samples and bundles. FR-18 determinism applies to configuration exports only; a share bundle carries a timestamp. Copied curl and Postman use a `$SECRET_VALUE` placeholder (FR-30). A backend endpoint with credentials embedded in the URL (user-info or credential-like query parameters) is rejected when the backend is saved or imported.
FR-17: Import is atomic. An invalid file is rejected as a whole, running configuration unchanged, and the rejection lists each problem with its location. Imported mappings always land as Draft, whatever status the file carries, and are never applied. Import never changes live configuration: it never overwrites or removes a Live mapping, a backend or an id-map entry. When a mapping already has a Draft, the import request states `replace` or `skip` for it; without that choice the import is refused with a conflict listing those mappings. Backends and id-map entries are added only when new. An identical backend or id-map entry is skipped, so re-importing an export into the same runtime succeeds. A backend with the same name and a different definition, or an id-map clash (FR-39), makes the file invalid. A mapping naming a backend found neither in the file nor in the runtime makes the file invalid. A mapping with no Draft, including a Live one, imports as the Draft of its next version (FR-33).
FR-18: Export is deterministic, whole and per mapping: identical stored state yields identical text, no export timestamps or generated ids. Drafts are covered.
FR-19: Access logs are redacted of sensitive data. Sensitive data is every secret value plus the values at a configurable list of field paths written as JSON Pointers.
FR-20: On a studio test call, the result of every mapping step is available. Normal runtime calls return no step results.
FR-21: On a studio test call, each TMF field traces to its backend source and transformation. Lineage is not produced on normal runtime calls.
FR-22: Bridge overhead is measured and reported for every call (total call time minus outbound backend call time, excluding lineage and step-result capture). Runtime responses carry the overhead in a `Server-Timing` header, and the studio reads the same value.
FR-23: Step results and lineage follow the same redaction rule as access logs (FR-19), including the configurable field paths.
FR-24: Studio and admin API users authenticate with OIDC.
FR-25: Runtime API calls require an OIDC bearer token, whatever the access path. This is configurable.
FR-26: The bridge manages authentication to legacy backends. A backend declaration holds only a secret name. Tech people provision the secret value outside the studio; the bridge resolves the name to the value at call time. The runtime lists provisioned secret names to the studio, never values. Studio, exports and logs never hold values.
FR-27: A dedicated mock backend ships with the product for demos, backing a zero-to-hero promise: after cloning from Git, a working showcase in ten minutes (a promise, not a hard metric).
FR-28: Unfired translation preview. Test has three sub-steps: (a) request-mapping sample test showing translated output, NOT sent to the backend; (b) response-mapping sample test; (c) a real read-only call to the backend or the mock. Sample request and response data is kept in the browser only; the studio sends a sample to the server to translate it, and the server never stores it.
FR-29: Verdict bar. Every studio test shows a plain-language, rules-based verdict: what happened, likely meaning, what to try, who to ask. Closed S1 rule set: HTTP status, id-map transform present, id-map entry found, empty mapped fields, OpenAPI validation outcome, backend reachable, secret name provisioned, and bridge overhead against the NFR-1 target. A failed backend-reachable or secret check gives a failed verdict. Overhead above target is an info note beside a pass, never blocks. Rules apply per Test sub-step: id-map transform present, id-map entry found and empty mapped fields apply to (a), (b) and (c); OpenAPI validation is request side in (a), response side in (b), both in (c); HTTP status, backend reachable, secret name provisioned and overhead apply to (c) only. A rule that does not apply is skipped, not shown as passed. A pass reads "Checks passed. The output matches your sample and the TMF622 schema." and never claims conformance.
FR-30: Exchange view. For each studio test call the analyst sees four summary tiles (bridge overhead as exchange latency, target endpoint, fields translated, id-map status), the mapping result, the HTTP request sent to the backend, the HTTP response, and the resulting TMF payload with field lineage, in three panels. A failed verdict opens the HTTP response panel and links to the mapping row at fault. FR-23 redaction applies. Secret names shown, values never. Copy as curl, copy as Postman (collection JSON on the clipboard), download a share bundle (mapping version as deterministic JSON, sample request and response with FR-19 paths masked, Exchange view record, verdict and checks, secret name, Postman collection, timestamp). Exports use a `$SECRET_VALUE` placeholder. Copy and download only, no send integration.
FR-31: A mapping has a version number and a status of Draft (never applied) or Live (an applied version exists). S1 applies by overwrite: Apply replaces the live version. History, diff and rollback are Later.
FR-32: Optimistic locking. The configuration and apply API checks a version or entity tag. Apply on a stale version is blocked: "Someone else changed this mapping. Reload the latest version before you apply." Autosave of a draft is refused in the same way when stale. The user's unsaved edits are kept, and the studio shows the same message.
FR-33: The studio autosaves mapping drafts on change and offers "Save now". The configuration API stores drafts in the runtime database. A Draft is stored and never served. Live is the applied version, which the runtime serves. Editing a Live mapping creates a Draft of the next version, and Apply promotes it.
FR-34: The studio manages named backends: endpoint, secret name reference (picked from provisioned names, never typed), authentication type (bearer, basic or api-key), and an optional legacy OpenAPI upload (belongs to the backend). REST and JSON only. A backend is configuration of its own, decoupled from mappings: a mapping holds only the backend's name. Editing a backend's endpoint or secret name creates no mapping version and needs no Apply; the runtime resolves the backend by name at call time. The edit shows which mappings use the backend. Backends are stored in the runtime database. A backend cannot be deleted while mappings use it, and the block lists those mappings.
FR-35: The mappings list is sorted A-Z by name by default and every column is sortable. No recency column, no last-opened behavior. A mapping is one per TMF operation, and its name is unique in the deployment. Import matches a mapping to an existing one by TMF API definition, method and path, not by name.
FR-36: No roles or permission states in S1. Any authenticated OIDC user can do everything.
FR-37: Documentation. The studio ships static user-manual docs, served by the studio itself, with a glossary of technical terms (payload, field path (JSON Pointer), transform, lineage, OpenAPI, id map, secret name, verdict, unfired preview, mapping version). Every help marker links to the matching manual page (NFR-5). Writing the content is an S1 deliverable delivered as a story by the project team.
FR-38: One mapping exports on its own, from the studio and the REST API, in addition to FR-15. A per-mapping export carries the Live version when one exists (a Draft of a later version is not exported with it); a never-applied mapping exports its Draft. Carries status and version including its id-map transform, never id-map entries. Names the backend it uses, carries no backend definition.
FR-39: Id map screen. The studio has a screen to view, add, edit, delete and import id-map entries (TMF id and backend id, per resource). A clash is the same resource and TMF id with a different backend id, or two different TMF ids of the same resource mapped to the same backend id. Clashes are checked on add, edit and import. Adding or importing an identical entry is accepted and changes nothing. Imported entries are additive, and a clash rejects the whole file (FR-17). Edit and delete take effect on the next call, with no Apply; delete asks for confirmation. The same operations exist through REST (FR-14). For the demo, infrastructure provisioning may seed the entries.

### NonFunctional Requirements

NFR-1: Bridge overhead is under 100 ms per API call, measured as in FR-22, at the 95th percentile. Conditions: warm compiled-mapping and API-definition caches, single bridge instance, zero-latency mock backend, payload up to 50 KB, lineage and step-result capture excluded.
NFR-2 (Accessibility): The studio meets WCAG 2.2 AA. Every function works by keyboard alone, every drag action has a keyboard alternative (the Connect picker). No status or signal relies on color alone.
NFR-3 (Platform): Desktop web only, current evergreen browsers (Chrome, Firefox, Edge, Safari). Supported minimum width 1280 px; layout reflows at 200% and 400% zoom down to 320 CSS px (WCAG 1.4.10). English first.
NFR-4 (Themes and preferences): Light and dark themes. Theme follows the operating system, with a manual toggle. The theme choice and the Guided or Side-by-side mode are remembered per browser.
NFR-5 (Contextual help): Inline tips, a glossary, and a link to the user manual (FR-37) from every help marker.
NFR-6 (Sample data): Sample request and response data is saved in the browser only. The server processes a sample but never stores it (FR-28).

### Additional Requirements

**Starter template / scaffold.** No starter template is named. Greenfield repo. The architecture fixes the layout, which drives Epic 1 Story 1:

- Repo layout: `api/` (single Maven module, package-by-layer under `com.tmfbridge`), `studio/` (React + TS + Vite), `mock/` (WireMock stubs), `deploy/` (Helm chart, docker compose, Keycloak realm seed), `docs/` (user manual).
- Stack pins (spine Stack table, SPEC overrides): Java 25, Spring Boot 4.1.3, Hibernate/Flyway/Jackson via BOM (never pinned), springdoc-openapi webmvc-api 3.2.0, ArchUnit 1.5.1, WireMock standalone 3.13.2, Keycloak 26.8.2, PostgreSQL operator-provided, Node 22+, Vite 8.3.8, react-aria-components 1.22.1, React 19.3.1, Spectral CLI 6.17.1, networknt json-schema-validator 3.0.9, Maven 3.10.1.

**Architecture rules (AD-n):**

- AR-1 (AD-1): Config is data, engine is generic. No class, route or bean named after a TMF resource (ArchUnit). A test adds a spec and mapping at runtime and serves them with no restart.
- AR-2 (AD-2): Hexagonal layering machine-enforced by one ArchUnit suite in CI: domain imports nothing from Spring, Jakarta, JPA or adapters (Jackson `JsonNode` is the only allowed third-party type). Domain ports end in `Port`; JPA entities end in `Entity`, live only in `adapter/out/persistence`.
- AR-3 (AD-3): Control plane beans active only under Spring profile `control`. `application/runtime` and `adapter/in/web/runtime` never depend on control code. CI boots with `control` off and asserts the TMF call path works. Data plane reads published versions only, never a Draft, read-only on config tables. Only `control` instances (or a Helm pre-upgrade Job) run Flyway; a data-plane pod with an older schema reports not-ready and retries.
- AR-4 (AD-4): Three images: `tmf-bridge-api`, `studio-frontend` (static SPA), `tmf-mock` (WireMock). Studio calls only the public config API. Test asserts every studio request path and method exists in the springdoc-generated control-plane OpenAPI. One origin via ingress/proxy.
- AR-5 (AD-5): One domain `CallPipeline` (spec matching, request validation, request table, backend call, response table, response validation, error table, overhead timer, evaluator) used by both runtime and studio test. Lineage and step results are an optional observer, on only for studio tests. Backend-client port wrapped in test mode guard: GET only, fails before network I/O. `POST /preview` is stateless: no write port, no draft or mapping repository (ArchUnit), sample bodies never reach any log, DB row or mapping document; may use read-only `IdMapReadPort`. Drafts loaded by control plane and handed to the pipeline.
- AR-6 (AD-6): Mapping document is canonical JSON validated by a published, versioned JSON Schema, `schemaVersion: 1`. Request and response rule tables. Rule `{target, expr}`; closed node vocabulary: `path`, `lit`, `concat`, `coalesce`, `if`, `lookup`, `convert`, `idMap`, `map`, `filter`, each with a schema entry and a contract test. All addresses are JSON Pointer (RFC 6901); `path.syntax` is `"pointer"`. `{"op":"expr"}` rejected at validation. Evaluation contexts as specified (request root, backend request, response root, `/$root/...` inside `map`/`filter`). Absent vs `null` differ: path to absent is a mapping error, to `null` yields `null`. Mapping error is `502`. Numbers are `BigDecimal`.
- AR-7 (AD-7): PostgreSQL single store. Applied MappingVersions and ApiSpecs immutable (Flyway DB trigger rejecting UPDATE/DELETE, plus test). Mapping natural key `(apiSpec content hash, HTTP method, path template)`; `name` is a unique label. `MappingVersion.number` is the business version, distinct from `lockRevision`. At most one Draft per Mapping (unique partial index). ETag = Mapping `lockRevision`; Autosave and Apply send `If-Match`; stale gives `412`. Compiled-mapping cache keyed by `(mappingId, number)`, spec cache by content hash, live pointer read from DB every call. Backends and id-map entries read from DB on every call, never cached. `IdMapReadPort` forward and reverse lookups, indexed; unique index on `resource + backendId`. Backend delete with references gives `409` listing mappings; `GET` Backend returns `usedBy[]`. Spring Data JPA: LAZY associations, explicit JPQL fetch, no entity graphs, `open-in-view=false`, `ddl-auto=none`, Flyway owns schema, UTC.
- AR-8 (AD-8): One domain canonicalizer for exports, bundles and hashes (stable key order, fixed indent, collections sorted by natural key). Test: export, import, export is byte-identical. Exports carry no generated ids; mapping references spec by content hash plus `(method, path)` and backend by name. Import validates whole file first (schema, references, id-map clashes, backend clashes), reports each error with a JSON Pointer location, commits in one transaction; `onDraftConflict=replace|skip` else `409`. Share bundle is one `.zip`: `manifest.json` (timestamp, formatVersion), `mapping.json`, `sample-request.json`, `sample-response.json`, `exchange.json`, `verdict.json`, `postman.json`; samples masked with `"***"`; secrets as `$SECRET_VALUE`; server assembles bundle, curl and Postman from the posted exchange record and stores nothing.
- AR-9 (AD-9): ApiSpecs stored in DB, content-hashed, uploaded via config API. Image seeds TMF620 and TMF622 v4 on first boot; DB wins afterwards. Routing table derived from each spec's `servers` and `paths`. Absent operation or no Live mapping gives `501`.
- AR-10 (AD-10): Secret name regex `[A-Za-z0-9_-]{1,64}`. `SecretResolver` port resolves from env `TMFBRIDGE_SECRET_<NAME>` or file `<secrets-dir>/<name>` (file wins). One `Redactor` registers every resolved value; logs, step results, lineage and bundles pass through it (exports do not). Configurable sensitive JSON Pointer paths, `***` token. Test: inject known secret, run every flow, grep all outputs. "List provisioned names" API returns names only. Backend endpoints validated: no userinfo or credential-like query params.
- AR-11 (AD-11): Spring MVC on Java 25 virtual threads, `RestClient` for backends, no reactive types. One catch-all handler dispatches into `CallPipeline`. Validator behind a domain port (networknt; 3.0 `nullable` etc. converted). Fixed error table (400, 404, 502, 504). Timeout default 5 s, optional per-backend, no retries.
- AR-12 (AD-12): Overhead measured by one Micrometer timer `bridge.overhead`; `Server-Timing: bridge;dur=<ms>` on every runtime response.
- AR-13 (AD-13): Separate `SecurityFilterChain`s per plane by path prefix. Control/config chain always authenticated (OIDC JWT resource server, `issuer-uri`), no toggle. Runtime chain bearer by default, `tmfbridge.runtime.auth.enabled` disables independently. Studio is a public SPA client (auth code with PKCE). Audiences per chain as separate properties. Test: enumerate springdoc control-plane endpoints, assert `401` without token.
- AR-14 (AD-14): Verdict computed server-side per sub-step rules; studio only renders. Every verdict has four parts.
- AR-15 (AD-15): DB holds configuration and the id map only. No resource state; `GET` rebuilds from the backend every call.
- AR-16 (AD-16): One immutable image per component across environments; promotion is export then import. Actuator liveness and readiness (DB reachability and schema version). Prometheus and OTLP. Studio image generates `config.json` (issuer, client id, API base) from env at container start. Compose demo with reverse proxy, working showcase within ten minutes of `git clone`, documented in README. CI builds three images and runs ArchUnit (AD-2, AD-3, AD-4, AD-13 suites), Testcontainers, Spectral, Playwright with axe-core.
- AR-17 (AD-17): Studio is React, TypeScript, Vite; React Aria Components for table, combobox (Connect picker) and drag and drop. Sample data in browser storage only. User manual is static content in the studio image. No studio copy, API description or export says conformant, compliant or certified. Studio exposes an apply-summary response and a mapping-lint endpoint (non-blocking warnings such as an id field mapped without `idMap`). Strings externalized, English first. Playwright reflow test. Axe-core in CI on every studio screen.
- AR-18: Deployment. `docker compose` demo: Postgres, API (`control` on), studio, mock, Keycloak with seeded user. Idempotent seeder (under `control`) loads TMF620 and TMF622 specs, mock backend declaration, id-map entries and the TMF620 mapping on first boot. The TMF622 mapping is deliberately NOT seeded (TMF622 answers `501` until mapped, UJ-2 and UJ-3). Helm chart deploys API (values select profile) and studio.
- AR-19: Test strategy. ArchUnit suite, Testcontainers Postgres, per-node contract tests, export round-trip, secret-leak grep, Playwright with axe-core, Spectral on generated control-plane OpenAPI. No CTK in S1. Config API described by springdoc OpenAPI, linted by Spectral with a ruleset in the repo.
- AR-20: Consistency conventions. Config properties prefix `tmfbridge.`. All stored and exported config is canonical JSON. Timestamps UTC ISO-8601. Ids are server-generated opaque strings except TMF and backend ids in the id map. Error bodies are the TMF Error object. Logs are structured JSON with trace id, through the Redactor. ETag and `If-Match` on mutable config resources.
- AR-21: Build order suggestion (solution-design section 10): domain core, persistence and Flyway, CallPipeline and runtime handler with TMF620 read, config API and import/export, studio shell and six steps, test mode / preview / verdict / exchange / share bundle, compose demo with Keycloak and seeder with TMF622 mapped through studio.

**Known source conflicts (resolved in favour of the PRD):**

- CONFLICT-1: EXPERIENCE.md Stale-version copy says "Review their changes before you apply" / "Review their changes" action; PRD FR-32 says "Someone else changed this mapping. Reload the latest version before you apply." Use the PRD text, action is "Reload the latest version".
- CONFLICT-2: EXPERIENCE.md Id-map delete confirm says "Calls with this id will return 404 until it is added again"; PRD FR-4/FR-6 and the spine make a missing id-map entry `502`. Use `502` wording.
- CONFLICT-3: EXPERIENCE.md Id-map entry clash defines only same resource and TMF id; PRD FR-39 and AD-8 also cover two TMF ids to one backend id. Use the PRD definition.
- CONFLICT-4: EXPERIENCE.md/DESIGN.md reference `imports/stitch/` and `wireframes/` paths and a `mock-id-map.html`; actual folder has `.working/` and `mockups/`. Treat mocks as illustrative only; spines win.

### UX Design Requirements

UX-DR1: Implement the design tokens from DESIGN.md as CSS custom properties for light and dark (colors incl. `-dark` variants, typography display/body/body-strong/small/label/code, rounded sm/DEFAULT/full, spacing 1..6, inspector-width 340px, gutter 16px, target-min 24px, breakpoints 1280/1024). Theme follows OS by default, manual Dark mode toggle in the top bar (`aria-pressed`), choice remembered per browser.
UX-DR2: Meet the DESIGN.md contrast table: text 4.5:1, UI components 3:1 using the dedicated `input-border` token (not `border`) for every control edge. Focus indicator is a 3px `focus-ring` outline with 2px offset and a 2px `surface` halo (`box-shadow`), never removed or clipped. Forced-colors mode: borders/outline/system colors replace background-only cues, wiring strokes use `currentColor`. Verified with axe-core and a contrast check in CI.
UX-DR3: Typography: system-ui stack and ui-monospace for payloads/paths/curl, no web font; no essential text below 14px (tables, badges, chips, footers); text resizes to 200% without loss.
UX-DR4: Layout and reflow: default guided single column with stepper on top; Side-by-side puts editor left and live test/lineage right; inspector persistent 340px column. Below 1024px and at 200%/400% zoom down to 320 CSS px the inspector and Help panel stack under the work area as disclosures, Side-by-side stacks editor above test, and only the mapping table scrolls sideways inside its own labeled `tabindex="0"` region. Flat elevation, no shadows except overlays and focus halo. Targets at least 24px.
UX-DR5: Top bar component on every authenticated surface in fixed order: logo/home, Backends, Id map, Help `?`, Dark mode toggle, account menu (Sign out only). Current page link carries `aria-current="page"`, bold weight and a 3px primary underline.
UX-DR6: Skip links on every surface in order: Skip to main content, Skip to inspector (workspace), Skip to share bar (Exchange view).
UX-DR7: Stepper component (six numbered steps): pill items in a wrapping row, 24px markers, done = success marker + check icon and a link, current = `info-bg` pill, bold text, primary filled marker, 3px primary bottom bar (no outline) and `aria-current="step"`, to-do = muted. Warnings shown as icon and count. Steps revisitable in any order after the first pass; warnings never block Next.
UX-DR8: Mapping table (Map request, Map response): native `<table>` (not ARIA grid) with columns TMF field, transform, backend field; first cell is `th scope="row"` containing one row-select button; selected row gets `info-bg`, 4px primary bar, 2px primary border, hidden text "selected" and `aria-current="true"`; link described via `aria-describedby`; rows can be added and deleted, no reordering; array rules render as a nested sub-table; empty target is dashed `input-border`; rows keep 12px padding; labeled focusable horizontal scroll region at narrow widths.
UX-DR9: Field lists (TMF inputs/attributes and backend fields) draggable and focusable, show type, each item has a Connect button (24px min). Fields come from the backend's OpenAPI or the saved sample.
UX-DR10: Wiring layer (eight rules): no lines at rest; selecting/focusing a row draws one labeled link; hover previews faintly; warning links dashed with icon; dragging legacy field onto TMF attribute (or reverse) creates a table row; Connect picker is the keyboard alternative; "Show all links" toggle off by default, count instead of lines when more than eight; layer is `aria-hidden`, table announces the link in text. Never a mesh of arrows. Drag directions per request/response.
UX-DR11: Connect picker (React Aria combobox/listbox popover): labeled search field, listbox with `aria-activedescendant`, selected option `info-bg`, chooses source and target and creates a row; Connect button bound to the selected row ("Connect a field to row id"); `Alt+C` opens it for the selected row; Esc closes and returns focus to the opener. Type mismatch shows a non-blocking note "{source} is a number and {target} expects text" with an "add conversion" button; row can still be created.
UX-DR12: Transform picker with the closed S1 list (direct copy or rename, constant, concatenation, conditional or default value, lookup table, type or format conversion, id-map, map/filter for arrays), each option showing name, one-line explanation and a `?` link.
UX-DR13: Id-map warning on a mapping row: dashed `id-map-warning` border, `-bg` fill, warning icon plus text and an "Add id-map transform" action. Non-blocking, never error red, never blocks Next or Apply.
UX-DR14: Inspector: persistent right panel on every step, collapsible with state remembered per session, content follows selection (source, transform, target, lineage when available, warnings) plus inline tip and `?` link; empty state "Select a row to see its source, transform and target."; Esc inside returns focus to the table; below 1024px it is a disclosure under the table.
UX-DR15: Mode switch (Guided steps / Side-by-side) in the workspace header: two toggle buttons with `aria-pressed`, 4px gap so focus rings are not clipped, never inside `overflow:hidden`; preserves selection, scroll and unsaved edits; choice remembered per browser. In Side-by-side the matching sample test re-runs on edit (debounced) beside each Map step.
UX-DR16: Sample editor: plain multi-line text field (not a code editor, so Tab is never captured) in `code` type; paste/edit JSON; Save locally; Run; invalid JSON shows line and column with `aria-invalid`/`aria-describedby`; Run is `aria-disabled` with reason text. Samples stored per browser (survive refresh and sign-out); standing note "Samples are saved in this browser only." with link to Share bundle; lost-sample empty state "No saved sample in this browser..." with "Paste a sample" and "Import from share bundle" (restores samples from a downloaded bundle).
UX-DR17: Test sub-step banners and states: unfired preview banner "Nothing was sent to the backend." persistent on sub-steps 1 and 2; "Request mapped. Nothing was sent to the backend."; sub-step 3 prefills the sample id from the saved sample request (never re-typed), button becomes "Calling backend…" and `aria-disabled`, cancel available, timeout message after the limit; every real backend call needs an explicit button press.
UX-DR18: Verdict bar at the top of the Exchange view: 2px border, icon, heading, one explanatory sentence, checklist with each check worded Passed, Failed or Skipped, links to the failing check; error state `error` on `error-bg`, pass state `success` on `success-bg`. Fixed pass text "Checks passed. The output matches your sample and the TMF622 schema." plus overhead ms and schema validation result. Overhead over budget shows info note "Bridge overhead was {n} ms, above the 100 ms target." beside the pass. Four-part error pattern rendered in the bar; 404 flow copy per Flow 4.
UX-DR19: Summary tiles (four, description list, label paired with value): Exchange latency (ms), Target endpoint (host only, code type), Fields translated (count of backend fields produced), Id-map status (OK, or "Missing key" in `id-map-warning` styling with link "Go to the mapping row: id" that moves focus to that row's Transform control in Map request). Tiles hidden before a run; two columns below 1024px, one column below 480px; a re-run moves focus to the verdict heading.
UX-DR20: Exchange panels (three): mapping result, HTTP request to backend, HTTP response and TMF payload with lineage; failing panel auto-expanded with 2px `error` border, others collapsed with a one-line summary; code blocks on `code-surface`, focusable labeled regions (`tabindex="0"`, `aria-labelledby`); masked values show `••••` with hidden text "masked"; secret name in code styling, value never rendered.
UX-DR21: Share bar sticky at the bottom of the Exchange view: Copy as curl, Copy as Postman, Download bundle, secret NAME; 3px primary top border; page `scroll-padding-bottom` equals its height; un-sticks under 600px viewport height; labeled region with skip link; note "Secrets are never included. Only the secret name is. Sensitive fields are masked in samples too."; "Copied." toast; export failure uses error pattern with Copy as curl fallback.
UX-DR22: Apply summary: card with definition list (mapping name, API and operation, backend, secret NAME), warning banner "This replaces the live version." (when a Live version exists), primary "Apply", secondary "Back to Test"; "Applying…" with `aria-disabled` (double submit ignored); success "Version {n} is live. The next call uses it." with link "Open Mappings list"; apply failed uses error pattern, state unchanged; stale version uses the Reload action (CONFLICT-1), Apply `aria-disabled` with reason text.
UX-DR23: Help panel: non-modal complementary region, 340px, dockable, context-aware to selected field/transform/step, search with "Nothing matched {query}." empty state, Help and Glossary tabs, manual links, lists current shortcuts, "Restore tips"; opened from top-bar `?` or `Alt+H`; keyboard combinations get a browser/OS/screen-reader conflict check during build and the panel always lists the current ones. "We couldn't open the user manual." fallback keeps built-in tip and glossary.
UX-DR24: Inline tip (per field and transform, one sentence, `?`, dismissible per tip, restorable), Help link `?` (24px round, accessible name "Help: {topic}", external manual links add "(opens in a new tab)"), Glossary term (real button with dotted primary underline, popover on hover and focus that the pointer can enter, persists until Esc, never a hover-only title tooltip). Glossary covers payload, field path (JSON Pointer), transform, lineage, OpenAPI, id map, secret name, verdict, unfired preview, mapping version, each with plain definition, example and manual link.
UX-DR25: Consistent help footer on every surface including Sign in and below the Exchange share bar (WCAG 3.2.6), same order: Glossary, User manual, Disclaimer, "Need a hand? Ask your integration lead." Disclaimer page states independent open-source project, not affiliated with or certified by TM Forum.
UX-DR26: Status badge pill with icon plus word: Draft (hollow circle, `surface-raised`, muted, `input-border` outline), Live (check icon, `success`). Only Draft and Live in S1.
UX-DR27: Banner (inline, never modal, severity by icon and text, 1px status border): unfired preview, offline ("You're offline. Edits are kept in this browser." with Retry, no auto-retry loops), session ended, stale version, id-map warning, sign-in failure. Toast only for confirmations (`role="status"`, at least 6 s, pauses on hover/focus, dismissible). Confirm dialog one level only, states what changes with the named object, default focus on the safe action. Skeleton rows announced once, no shimmer under reduced motion. Toggle buttons with `aria-pressed`.
UX-DR28: Four-part error pattern for every error (what happened, likely meaning, what to try as a button where possible, who to ask), icon plus text, inline where the action happened. Announcement rule: (a) error after a user action that replaces/reloads the view moves focus to the error heading inside a labeled region (`tabindex="-1"`, no `role="alert"`); (b) error appearing in place without user action (autosave failure, offline, session ended, background stale) uses `role="alert"` without moving focus; never both. No backend internals, stack traces or secret values.
UX-DR29: Mappings list (home): columns name (link, row click also opens), TMF API and operation, backend, status; every column header is a sort button with `aria-sort` and a polite sort announcement; default A-Z by name; no recency column or last-opened behavior; row action "Export {name}"; list actions New mapping, Import, Export all; empty state "No mappings yet." with New mapping, "Import a configuration" and a `?` link; five skeleton rows with "Loading mappings"; import rejected "Import rejected. Nothing changed." with each problem and location; import success "Imported {n} mappings as Draft. Review them, then apply."; per-draft-conflict prompt asking replace or skip; open failure "We couldn't open {name}." / "{name} no longer exists...".
UX-DR30: Backends surface: list, add, edit, delete; empty state "No backends declared yet." + "Declare backend"; no secret names state (disabled picker with reason "No secret names are provisioned. Ask your tech team to create one."); form invalid (bad URL message, `aria-invalid`, summary at top, Save `aria-disabled` with reason, loading label); edit note "{n} mappings use this backend. They use the new values on their next call." with links; delete blocked (what/likely/try/ask, mapping links, Delete visible and `aria-disabled` with reason) and allowed (confirm "Delete {backend}? No mapping uses it.", default focus Cancel, success "{backend} deleted."); auth type select (bearer, basic, api-key); secret name only from picker; invalid OpenAPI upload error with first error and line, prior selection unchanged, WSDL/SOAP not accepted.
UX-DR31: Id map surface: sortable list (default sort resource then TMF id, polite "Sorted by TMF id, ascending"); add/edit form with required fields, clash check on blur with polite status line, focus to first invalid field on Save error; clash message "TMF id {id} already maps to {backend id} for {resource}." (and the reverse clash per CONFLICT-3); identical entry accepted silently; edit row action "Edit {TMF id}"; delete row action "Delete {TMF id}" with confirm (502 wording per CONFLICT-2, default focus Cancel, "Entry deleted."); import entries rejected (case a, whole file rejected with locations) and success "Imported {n} entries, {m} already present."; empty state with "Add an entry" and "Import entries" and tip.
UX-DR32: Sign-in surface and auth states: "Taking you to your company sign-in…" with `aria-disabled` button; failed or denied "Sign-in didn't complete." (error pattern a); signed in no access with the account shown; session expiry banner "Your session ended." (`role="alert"`, focus stays, "Sign in again" next Tab stop) with draft kept in browser and return to the same surface, step and selection; server 403 uses error pattern.
UX-DR33: Step navigation accessibility: a step change moves focus to the surface `h1` (`tabindex="-1"`, text "Step {n} of 6: {name}") and updates document title "{Step} (step {n} of 6) - {mapping} - TMF-Bridge Studio", no separate live region. One `h1` per surface, panels `h2`. Every button has a unique accessible name in context.
UX-DR34: Pick API step (TMF API from official OpenAPI list, then operation; skeleton select with "Loading TMF API definitions."; this step owns the API/operation choice that feeds the list column; no upload) and Declare backend step (select existing backend with endpoint and secret NAME read-only, or inline "Add a backend"; endpoint never re-typed, WCAG 3.3.7; step named exactly "Declare backend").
UX-DR35: Autosave UX: persistent "Saved." indicator (`role="status"`, not a toast) with "Save now" button beside it; empty mapping table text "Nothing mapped yet. Drag a field, or use Connect." (no auto-suggestion); disabled actions are `aria-disabled` (focusable) with reason text via `aria-describedby`, never silent grey-out.
UX-DR36: Live test announcements in Side-by-side: debounced single polite `role="status"` line ("Test re-ran: 7 fields mapped, 1 warning"), lineage available as a nested list (value, then each step that produced it) as well as any graphic, "Lineage isn't available for this value yet." with reason; step results and lineage trace nested `map`/`filter` rules per item.
UX-DR37: Voice and tone: plain, specific, reassuring; verbs on buttons ("Map request", "Test", "Apply"); never "Submit"/"OK"; no exclamation marks, no celebration copy; terms like "Dry run" reserved for Later; technical terms kept and explained once with `?`; never blame the user; no studio copy says conformant, compliant or certified (AD-17). All strings externalized, English first.
UX-DR38: Reduced motion: no transitions on step change, panel expand, save indicator or skeleton shimmer. Keyboard rule set: Esc closes topmost popover/panel and returns focus to opener; shortcuts always need a modifier (WCAG 2.1.4); mapping table fully keyboard operable. Banned: drag-only actions, single-character shortcuts, color-only signals, auto-fired backend calls, typing secrets anywhere, send integrations, modal stacks deeper than one level, hover-only help, auto-dismissing errors, celebratory animation.

### FR Coverage Map

FR-1: Epic 1 - Serve each configured TMF API from its official OpenAPI definition
FR-2: Epic 1 - 501 for unconfigured or no-Live operation, 404 for unknown id
FR-3: Epic 2 - Configuration takes effect without restart
FR-4: Epic 1 - Fixed backend and validation error table
FR-5: Epic 1 (engine, 10 node types) and Epic 5 (studio transform table, nested sub-tables)
FR-6: Epic 2 - Id map storage, lookup, clash rules (REST); warning in Epic 5
FR-7: Epic 1 - `@type`, `@baseType`, `@schemaLocation` pass-through
FR-8: Epic 1 - Runtime request/response OpenAPI validation
FR-9: Epic 6 (verdict wording, no conformance claim) and Epic 7 (Disclaimer page)
FR-10: Epic 5 - Six-step studio authoring
FR-11: Epic 6 - Studio test with lineage in Exchange view
FR-12: Epic 2 (apply API) and Epic 5 (Apply UI, three states)
FR-13: Epic 6 - Test calls are read-only (GET-only test mode)
FR-14: Epic 2 (REST parity) with studio import/export in Epic 3 and Epic 4
FR-15: Epic 3 - Whole-configuration export and import
FR-16: Epic 3 (exports, credential-in-URL rejection) and Epic 6 (bundles, curl, Postman)
FR-17: Epic 3 - Atomic import, Draft-only landing, `onDraftConflict`
FR-18: Epic 3 - Deterministic export
FR-19: Epic 1 - Redacted access logs, configurable sensitive paths
FR-20: Epic 6 - Step results on studio test calls
FR-21: Epic 6 - Field lineage on studio test calls
FR-22: Epic 1 - Bridge overhead and `Server-Timing`
FR-23: Epic 6 - Redaction of step results and lineage
FR-24: Epic 2 (control-plane OIDC chain) and Epic 4 (studio sign-in)
FR-25: Epic 1 - Runtime bearer token (configurable)
FR-26: Epic 1 (secret resolver) and Epic 2 (provisioned names list)
FR-27: Epic 8 - Mock backend showcase (mock itself seeded from Epic 1 Story 1.1)
FR-28: Epic 6 - Unfired preview and three Test sub-steps
FR-29: Epic 6 - Rules-based verdict per sub-step
FR-30: Epic 6 - Exchange view, curl, Postman, share bundle
FR-31: Epic 2 - Mapping version number and Draft/Live status
FR-32: Epic 2 - Optimistic locking (ETag, `412`)
FR-33: Epic 2 (draft storage, autosave API) and Epic 5 (autosave UI, Save now)
FR-34: Epic 2 (backend API) and Epic 4 (Backends screen)
FR-35: Epic 4 - Mappings list sorted A-Z, all columns sortable
FR-36: Epic 2 - No roles in S1
FR-37: Epic 7 - User manual and glossary content
FR-38: Epic 3 - Per-mapping export
FR-39: Epic 2 (id map REST, clash rules) and Epic 4 (Id map screen)

NFR coverage: NFR-1 verified in Epic 8 (benchmark), NFR-2 to NFR-4 built in Epic 4 and Epic 5 and gated in Epic 7, NFR-5 in Epic 7, NFR-6 in Epic 6.

## Epic List

### Epic 1: Partners call TMF620 and get a valid answer
A partner calls `GET /productOffering/{id}` and gets a schema-valid TMF response, or a fixed TMF Error. Includes the repo scaffold, a minimal docker compose dev stack (Postgres, API, mock) in Story 1.1, the generic domain engine, persistence, runtime handler and a first-boot seeder.
**FRs covered:** FR-1, FR-2, FR-4, FR-5 (engine), FR-7, FR-8, FR-19, FR-22, FR-25, FR-26 (resolver)

### Epic 2: Operators manage live configuration through the REST API
An operator creates, drafts, applies and changes mappings, backends and the id map by REST, with no restart and with optimistic locking.
**FRs covered:** FR-3, FR-6, FR-12 (API), FR-14, FR-24 (API), FR-26 (names), FR-31, FR-32, FR-33 (API), FR-34 (API), FR-36, FR-39 (API)

### Epic 3: Configuration as code (export, import, promote)
An operator exports everything, or one mapping, as deterministic JSON and imports it atomically. This is the exit path and the promotion path between environments.
**FRs covered:** FR-14 (import/export), FR-15, FR-16 (exports), FR-17, FR-18, FR-38

### Epic 4: Studio shell and configuration management
Inès signs in, sees her mappings, and manages backends, the id map, export and import in an accessible studio.
**FRs covered:** FR-24 (studio), FR-34 (UI), FR-35, FR-39 (UI), FR-14/15/38 (UI); NFR-2 to NFR-4 foundations

### Epic 5: Inès authors and applies a mapping in the studio
Inès goes from Pick API to "Version {n} is live" with no code.
**FRs covered:** FR-5 (UI), FR-10, FR-12 (UI), FR-33 (UI), FR-6 (warning)

### Epic 6: Test, diagnose and share (verdict, Exchange view, lineage)
Inès knows a mapping works before it goes live, and can forward a failure to tech as a share bundle.
**FRs covered:** FR-9 (verdict wording), FR-11, FR-13, FR-16 (bundles), FR-20, FR-21, FR-23, FR-28, FR-29, FR-30

### Epic 7: Help, glossary and user manual
A non-developer is never stuck: contextual help, glossary and the user manual ship inside the product, with CI accessibility gates.
**FRs covered:** FR-9 (Disclaimer), FR-37; NFR-2 and NFR-5 verification

### Epic 8: Zero-to-hero showcase and delivery
An open-source user clones the repo and has TMF620 served and TMF622 mapped in ten minutes; an operator can deploy it.
**FRs covered:** FR-27; NFR-1 verification

## Epic 1: Partners call TMF620 and get a valid answer

A partner calls `GET /productOffering/{id}` and gets a schema-valid TMF response, or a fixed TMF Error. Includes the repo scaffold, a minimal docker compose dev stack, the generic domain engine, persistence, the runtime handler and a first-boot seeder.

### Story 1.1: Scaffold the repo and a minimal compose dev stack

As a developer,
I want the repo layout, a booting API skeleton, a mock backend and a minimal `docker compose` stack,
So that every later story builds on the agreed structure and I can run the system locally from day one.

**Acceptance Criteria:**

**Given** a fresh clone
**When** I look at the root
**Then** `api/` (single Maven module, packages `domain`, `application/runtime`, `application/control`, `adapter/in`, `adapter/out`, `bootstrap` under `com.tmfbridge`), `studio/` (placeholder), `mock/`, `deploy/` and `docs/` exist
**And** the build uses Java 25, Spring Boot 4.1.3 and Maven 3.10.1, with Hibernate, Flyway and Jackson versions managed by the BOM and never pinned.

**Given** the `api/` module
**When** I run the build
**Then** an ArchUnit suite enforces: domain imports no Spring, Jakarta, JPA or adapters (Jackson `JsonNode` is the only allowed third-party type); `application/runtime` and `application/control` never import each other; no package or class under `main` is named after a TMF resource
**And** the build fails on a violation, proven by a deliberately failing fixture test.

**Given** `docker compose up` in `deploy/`
**When** the stack starts
**Then** Postgres, `tmf-bridge-api` and `tmf-mock` (WireMock 3.13.2) run
**And** the API reports healthy on its Actuator liveness endpoint
**And** this minimal stack has no Keycloak and no studio, which arrive in later epics.

**Given** the mock backend
**When** I call it directly
**Then** it serves legacy-shaped JSON (not TMF-shaped) for a product-offering record and for orders, where ORD-2 is known and ORD-1 returns `404`.

### Story 1.2: Evaluate basic mapping expressions

As an integrator,
I want a generic engine that evaluates `path`, `lit`, `concat`, `coalesce` and `if` rules,
So that field-level translation is data, not code.

**Acceptance Criteria:**

**Given** a rule table `[{target, expr}]` and an input document
**When** the evaluator runs
**Then** every address (read and write) is a JSON Pointer (RFC 6901) and `path.syntax` is `"pointer"`
**And** the output document is built by writing each `target` pointer.

**Given** a `path` to an absent location in a scalar slot
**When** evaluated
**Then** it raises a mapping error
**And** a `path` to a JSON `null` yields `null`.

**Given** numeric values
**When** they pass through any node
**Then** they use `BigDecimal`, never `double`.

**Given** a mapping document
**When** validated
**Then** the published JSON Schema (`schemaVersion: 1`) accepts the nodes of this story, rejects an unknown `schemaVersion` and rejects `{"op":"expr","text":"..."}`
**And** each node has a schema entry and a contract test (AR-6).

### Story 1.3: Evaluate lookup, convert, idMap, map and filter

As an integrator,
I want lookup tables, type and format conversion, id-map translation, and map and filter over arrays,
So that I can translate ids, dates, numbers and nested structures.

**Acceptance Criteria:**

**Given** a `lookup`, `convert` (dates, numbers) or `idMap` node
**When** evaluated
**Then** each has a schema entry and a contract test
**And** `idMap` calls only the read-only `IdMapReadPort` (forward `(resource, tmfId)`, reverse `(resource, backendId)`).

**Given** an `idMap` node with no matching entry, forward or reverse
**When** evaluated
**Then** it raises a mapping error (never a not-found).

**Given** a `map` node over an array and a `filter` node with a condition
**When** evaluated
**Then** `path` inside is relative to the current item and `/$root/...` addresses the table root
**And** `filter` keeps only matching items and `map` transforms each item.

**Given** a `map` nested in a `map`
**When** evaluated
**Then** results are correct per item, with no iteration expressed in path syntax.

**Given** the evaluation contexts of AD-6
**When** the request and response tables run
**Then** the request root is `{path, query, headers, body}` and the response root is `{request, status, headers, body}`.

### Story 1.4: Store specs and mappings

As an operator,
I want specs and mappings persisted in PostgreSQL with immutable applied versions,
So that the runtime serves stable versions and nothing live can be mutated by accident.

**Acceptance Criteria:**

**Given** a Flyway migration
**When** it runs
**Then** it creates only `api_spec` (content-hashed), `mapping` and `mapping_version`, the tables needed to route and serve (the `backend` table is added by Story 1.7 and `id_map_entry` by Story 1.8, each in its own migration)
**And** `ddl-auto=none`, `open-in-view=false` and timestamps are UTC.

**Given** an applied `mapping_version` or an `api_spec` row
**When** an UPDATE or DELETE is attempted
**Then** a DB trigger rejects it, proven by a test.

**Given** a Mapping
**When** a second Draft is inserted
**Then** the unique partial index rejects it
**And** the natural key is `(api_spec hash, method, path template)` and `name` is unique per deployment.

**Given** the persistence adapter
**When** entities cross a port
**Then** they are mapped to domain types, entities stay in `adapter/out/persistence` and associations are LAZY (ArchUnit)
**And** readiness reports not-ready when the schema is older than required and retries instead of crash-looping (AR-3).

### Story 1.5: Route TMF calls from stored OpenAPI specs

As a partner,
I want unconfigured TMF operations to answer cleanly,
So that I get a TMF Error, not a crash or an empty body.

**Acceptance Criteria:**

**Given** first boot
**When** the API starts
**Then** the TMF620 and TMF622 v4 specs are seeded once, content-hashed, and on later boots the DB wins and nothing is overwritten.

**Given** stored specs
**When** a request reaches the catch-all handler
**Then** the routing table is derived from each spec's `servers` and `paths`, and no class or route is named after a TMF resource.

**Given** an operation absent from every spec, or present with no Live mapping (including Draft-only)
**When** called
**Then** it returns `501` with a TMF Error body with no internals.

**Given** a spec and mapping inserted while the app runs
**When** the next call arrives
**Then** it is served with no restart, because the live pointer is read from the DB every call (AR-1, AR-7).

### Story 1.6: Resolve secrets by name and redact sensitive data

As an operator,
I want secret values resolved at call time and kept out of every output,
So that credentials never leak through logs.

**Acceptance Criteria:**

**Given** a secret name matching `[A-Za-z0-9_-]{1,64}`
**When** the `SecretResolver` resolves it
**Then** it reads env `TMFBRIDGE_SECRET_<NAME>` or file `<secrets-dir>/<name>`, file wins
**And** a missing secret yields a "not provisioned" outcome.

**Given** a resolved value
**When** registered with the single `Redactor`
**Then** anything passed through the Redactor shows `***` in its place.

**Given** a configurable list of sensitive JSON Pointer paths (dotted input normalized)
**When** access logs are written
**Then** values at those paths are redacted, and logs are structured JSON with a trace id.

**Given** a known injected secret and sensitive-path value
**When** every flow runs
**Then** a grep of all outputs finds neither (FR-19).

### Story 1.7: Call backends with timeouts and a fixed error table

As a partner,
I want backend failures turned into predictable TMF Errors,
So that my client can handle them without seeing backend internals.

**Acceptance Criteria:**

**Given** a Flyway migration adding the `backend` table (name, endpoint, authType, secret name, optional timeout)
**And** a REST and JSON backend declaration (authType bearer, basic or api-key)
**When** the backend client calls it
**Then** it uses `RestClient` on virtual threads, resolves the secret by name at call time and reads the backend from the DB every call
**And** the default timeout is 5 s with an optional per-backend value, and there are no retries.

**Given** each backend outcome
**When** mapped to a TMF Error
**Then** backend `404` gives `404`, other `4xx` and `5xx` give `502`, transport fault gives `502`, timeout gives `504`
**And** bodies are generic and the original error goes to logs through the Redactor.

**Given** each row of the error table
**When** tests run against WireMock
**Then** each is proven (FR-4).

### Story 1.8: Serve TMF620 reads through one validated pipeline

As a partner (Tomás),
I want `GET /productOffering/{id}` to return a schema-valid TMF payload,
So that I can integrate against a stable contract.

**Acceptance Criteria:**

**Given** a Live TMF620 read mapping and a reachable mock
**When** I call `GET /productOffering/{id}`
**Then** the domain `CallPipeline` (spec matching, request validation, request table, backend call, response table, response validation) returns `200` with the TMF payload
**And** `@type`, `@baseType` and `@schemaLocation` pass through as mapped fields and are checked by the validator (FR-7).

**Given** a Flyway migration adding `id_map_entry` with unique indexes on `(resource, tmfId)` and `(resource, backendId)` and an indexed adapter for `IdMapReadPort`
**When** the pipeline runs an `idMap` node
**Then** it translates forward and reverse from the DB on every call, never cached.

**Given** a malformed request
**When** called
**Then** it returns `400` (the fixed error table of Story 1.7 plus invalid request).

**Given** a translated response that fails OpenAPI validation
**When** produced
**Then** it returns `502`, the bad payload is never returned and details go to logs only
**And** OpenAPI 3.0 specs validate correctly, the domain port converting `nullable` etc. for networknt 3.0.9.

**Given** any runtime response
**When** returned
**Then** it carries `Server-Timing: bridge;dur=<ms>`, with overhead = total time minus the outbound backend call, measured by one Micrometer timer `bridge.overhead`
**And** runtime calls return no step results or lineage.

**Given** a mapping error at runtime (including an id-map miss in either direction)
**When** called
**Then** it returns `502`, never `404`.

### Story 1.9: Require OIDC bearer tokens on the runtime API

As an operator,
I want the runtime API to require an OIDC token, with the option to turn that off,
So that partner access is controlled by default.

**Acceptance Criteria:**

**Given** the runtime filter chain, selected by path prefix
**When** a call has no valid token
**Then** it returns `401`, with tokens validated against `issuer-uri` and a runtime-specific audience property.

**Given** `tmfbridge.runtime.auth.enabled=false`
**When** a call has no token
**Then** it is served, and the toggle affects only the runtime chain.

**Given** profile `control` is off
**When** the app boots in CI
**Then** the TMF call path works with no control-plane beans, and an ArchUnit rule forbids runtime code depending on control code (AR-3).

**Given** no roles in S1
**When** any authenticated token calls a configured operation
**Then** it is served.

### Story 1.10: Seed the demo data on first boot

As an open-source user,
I want the mock backend, id-map entries and a TMF620 mapping seeded on first boot,
So that TMF620 answers immediately and TMF622 stays unmapped for the studio journey.

**Acceptance Criteria:**

**Given** profile `control` and an empty database
**When** the API starts
**Then** an idempotent seeder loads the mock backend declaration, the id-map entries and a Live TMF620 mapping, and a second start changes nothing.

**Given** the seeded system
**When** I call TMF620 `GET /productOffering/{id}`
**Then** it returns a schema-valid payload (first half of UJ-2)
**And** TMF622 `GET /productOrder/{id}` returns `501` with a TMF Error, because the TMF622 mapping is deliberately not seeded.

**Given** the compose dev stack of Story 1.1
**When** the seeder has run
**Then** the above is reproducible there.

**Given** an id-map entry seeded for ORD-2
**When** the mapping is evaluated
**Then** forward and reverse lookups resolve, and ORD-1 stays unmapped.

## Epic 2: Operators manage live configuration through the REST API

An operator creates, drafts, applies and changes mappings, backends and the id map by REST, with no restart and with optimistic locking. Every studio action in later epics maps to an endpoint built here.

### Story 2.1: Secure the control plane and publish its OpenAPI

As an operator,
I want a config API that always requires OIDC and is described by a linted OpenAPI,
So that nothing administrative is ever open and clients can be generated from the contract.

**Acceptance Criteria:**

**Given** profile `control` is on
**When** any control-plane endpoint is called without a token
**Then** it returns `401`, proven by a test that enumerates every springdoc-collected endpoint
**And** the control chain has no toggle and uses its own audience property (AR-13).

**Given** a valid OIDC JWT from the configured `issuer-uri`
**When** any control-plane endpoint is called
**Then** it is allowed, with no roles in S1 (FR-24, FR-36).

**Given** the build
**When** the springdoc OpenAPI is generated
**Then** Spectral 6.17.1 lints it with a ruleset stored in the repo, and the build fails on violations
**And** springdoc 3.2.0 (webmvc-api) is used (AR-19).

**Given** profile `control` is off
**When** the app boots
**Then** no control-plane beans or endpoints exist (AR-3).

### Story 2.2: Create, list and autosave a mapping Draft

As an integrator,
I want to create a mapping for a TMF operation and save its Draft as I edit,
So that work in progress is stored safely and never served.

**Acceptance Criteria:**

**Given** a stored ApiSpec and an operation (method and path)
**When** I `POST` a new mapping with a unique name
**Then** the Mapping is created with natural key `(spec hash, method, path template)` and a Draft of number 1
**And** a duplicate name or a duplicate operation returns `409`.

**Given** a Draft
**When** I `PUT` a mapping document
**Then** it is validated against the published JSON Schema and invalid documents return `400` with the error locations
**And** the response carries an `ETag` equal to the Mapping `lockRevision`.

**Given** a Draft exists
**When** a runtime call for that operation arrives
**Then** it returns `501`, because a Draft is never served (FR-33).

**Given** `GET /mappings`
**When** I pass sort parameters for name, operation, backend or status
**Then** the list is sorted accordingly with A-Z by name as the default, and each item shows status Draft or Live and its version number (FR-31, FR-35)
**And** a single `GET` returns the Draft, the Live version if any, and the ETag.

**Given** a PUT with `If-Match` carrying a stale ETag
**When** it is sent
**Then** it returns `412` and the stored Draft is unchanged (FR-32).

### Story 2.3: Apply a Draft and edit a Live mapping

As an integrator,
I want to apply a Draft so it serves, and edit a Live mapping safely,
So that a new version goes live with no restart and no one's work is overwritten.

**Acceptance Criteria:**

**Given** a valid Draft
**When** I `POST` apply with a current `If-Match`
**Then** the Draft becomes an immutable applied MappingVersion, the Mapping live pointer moves to it, nothing is deleted
**And** the next runtime call uses it with no restart (FR-3, FR-12).

**Given** a stale `If-Match`
**When** I apply
**Then** it returns `412` with the message "Someone else changed this mapping. Reload the latest version before you apply." and nothing changes.

**Given** a Live mapping
**When** I edit it
**Then** a Draft of number+1 is created, and the Live version keeps serving until Apply
**And** Apply replaces the live version (overwrite only, no history in S1, FR-31).

**Given** the Mapping references a backend name that does not exist
**When** I apply
**Then** it is refused with `400` naming the backend.

**Given** an apply-summary request
**When** called before apply
**Then** it returns what goes live (mapping name, operation, backend, secret name) and whether a Live version would be replaced
**And** after apply the response includes "Version {n}" (AR-17).

### Story 2.4: Manage backends over REST

As an integrator,
I want named backends with endpoint, secret name and auth type,
So that mappings reference a backend by name and credentials never enter configuration.

**Acceptance Criteria:**

**Given** a backend with endpoint, authType (bearer, basic or api-key), secret name and optional timeout
**When** I create or edit it
**Then** it is stored in the runtime database and the secret name must match `[A-Za-z0-9_-]{1,64}`
**And** editing creates no mapping version and needs no Apply, so Live mappings use the new values on their next call (FR-34).

**Given** an endpoint with credentials in user-info or credential-like query parameters
**When** I save it
**Then** it is rejected with the location of the problem (FR-16, AR-10).

**Given** a backend used by any Draft or applied MappingVersion
**When** I `GET` it
**Then** the response lists `usedBy[]`
**And** deleting it returns `409` listing those mappings.

**Given** an unused backend
**When** I delete it
**Then** it is removed.

**Given** the list endpoint
**When** called
**Then** it is sortable and each backend shows its name and endpoint (no secret value ever, only the name).

### Story 2.5: List provisioned secret names and upload a backend's legacy OpenAPI

As an integrator,
I want to pick a provisioned secret name and attach a legacy OpenAPI to a backend,
So that I never type a secret and the studio can list backend fields.

**Acceptance Criteria:**

**Given** secrets provisioned via env or file
**When** I call the provisioned-names endpoint
**Then** it returns names only, never values, and the file source wins over env for the same name (FR-26).

**Given** a backend with a secret name that is not provisioned
**When** it is saved
**Then** it is accepted (names are validated by pattern, not presence) and the later test verdict will report "not provisioned".

**Given** a legacy OpenAPI file uploaded for a backend
**When** it is valid REST and JSON
**Then** it is stored with the backend and its operations and schemas are returned for field lists.

**Given** an invalid, incomplete or WSDL/SOAP file
**When** uploaded
**Then** it is rejected with the first error and its line, and the previous upload is unchanged (WSDL and SOAP are Later).

### Story 2.6: Manage the id map over REST

As an integrator,
I want to add, edit, delete and import id-map entries with clash rules,
So that the id-map transform has data to translate and reverse lookup has one answer.

**Acceptance Criteria:**

**Given** an entry `(resource, tmfId, backendId)`
**When** I add it
**Then** it is stored, and an identical entry is accepted and changes nothing
**And** a clash returns `409` with what happened: same resource and TMF id with a different backend id, or two TMF ids of the same resource sharing a backend id (FR-39).

**Given** an existing entry
**When** I edit or delete it
**Then** the change takes effect on the next call with no Apply (FR-6)
**And** clashes are checked on edit too.

**Given** an entries file
**When** I import it
**Then** entries are additive, identical entries are skipped, and any clash or invalid row rejects the whole file listing each problem with its location
**And** the response reports how many were added and how many were already present.

**Given** a call that hits a missing entry after a delete
**When** the pipeline runs
**Then** it returns `502`, not `404` (FR-4, FR-6).

**Given** the list endpoint
**When** called
**Then** it is sortable with default order resource then TMF id.

### Story 2.7: Upload TMF specs and list APIs and operations

As an integrator,
I want to load a TMF OpenAPI and list the APIs and operations I can map,
So that a new API needs no rebuild and Pick API has real data.

**Acceptance Criteria:**

**Given** a valid TMF OpenAPI file
**When** I upload it
**Then** it is stored content-hashed and immutable, and routing includes it on the next call with no restart (AR-9)
**And** v4 and v5 can coexist when both are loaded.

**Given** the list endpoint
**When** called
**Then** it returns each stored API (title, version, hash) and its operations (method, path, operationId).

**Given** an invalid OpenAPI file
**When** uploaded
**Then** it is rejected with the first error and its line.

**Given** an operation in a stored spec with no Live mapping
**When** called at runtime
**Then** it returns `501` (FR-2).

### Story 2.8: Lint a mapping for non-blocking warnings

As an analyst,
I want warnings when an id field is mapped without the id-map transform,
So that I notice the most common mistake without being blocked.

**Acceptance Criteria:**

**Given** a Draft where a rule targets or reads an id field without an `idMap` node
**When** I call the mapping-lint endpoint
**Then** it returns a warning naming the rule location (JSON Pointer) and an "add idMap" hint (FR-6, AR-17).

**Given** a Draft with an `idMap` on every id field
**When** linted
**Then** it returns no warnings.

**Given** warnings exist
**When** I apply the Draft
**Then** Apply still succeeds (warnings never block).

**Given** lint output
**When** returned
**Then** it contains no conformance wording (AR-17).

## Epic 3: Configuration as code (export, import, promote)

An operator exports everything, or one mapping, as deterministic JSON and imports it atomically. This is the exit path and the promotion path between environments.

### Story 3.1: Export the whole configuration deterministically

As an operator,
I want to export all configuration as canonical JSON,
So that I have a portable exit path and clean diffs.

**Acceptance Criteria:**

**Given** stored state
**When** I `GET` the whole export
**Then** it contains Draft and Live mappings (each with status and number), the TMF OpenAPI definitions in use with full text, backends and id-map entries (FR-15)
**And** it is produced only by the domain canonicalizer (stable key order, fixed indent, collections sorted by natural key: mappings by operation key, backends by name, id-map entries by `(resource, tmfId)`, specs by hash).

**Given** identical stored state
**When** exported twice
**Then** the output is byte-identical, with no timestamps and no generated ids (FR-18)
**And** a mapping references its spec by content hash plus `(method, path)` and its backend by name.

**Given** any export
**When** inspected
**Then** it contains secret names and never values, by construction (the schema has no value field)
**And** a draft edit changes the exported text.

### Story 3.2: Export one mapping on its own

As an analyst,
I want to export a single mapping,
So that I can hand one mapping to a colleague or another environment.

**Acceptance Criteria:**

**Given** a Mapping with a Live version
**When** I export it
**Then** the Live version is exported, even if a Draft of a later number exists (FR-38).

**Given** a Mapping never applied
**When** I export it
**Then** its Draft is exported.

**Given** a per-mapping export
**When** inspected
**Then** it carries status, version and its id-map transform, names the backend without carrying a backend definition, and never contains id-map entries
**And** it uses the same canonicalizer and is deterministic.

### Story 3.3: Validate an import file completely before touching anything

As an operator,
I want an invalid file rejected as a whole with every problem listed,
So that a bad import can never leave the running configuration half changed.

**Acceptance Criteria:**

**Given** an import file
**When** I submit it
**Then** the whole file is validated first: schema, references, id-map clashes, backend clashes
**And** every problem is reported with a JSON Pointer location, and nothing is committed (FR-17).

**Given** a backend with the same name and a different definition, or an id-map clash (including two TMF ids to one backend id)
**When** imported
**Then** the file is invalid.

**Given** a mapping naming a backend found neither in the file nor in the runtime
**When** imported
**Then** the file is invalid.

**Given** a backend endpoint with embedded credentials
**When** imported
**Then** the file is invalid (FR-16).

**Given** a valid file
**When** committed
**Then** it is one DB transaction, and a failure rolls everything back.

### Story 3.4: Import mappings as Drafts, additively, without touching Live

As an operator,
I want imported mappings to land as Drafts and live configuration to stay untouched,
So that I can review before anything goes live.

**Acceptance Criteria:**

**Given** a valid file
**When** imported
**Then** every mapping lands as Draft whatever status the file carries, and is never applied
**And** a mapping with no Draft (including one whose Mapping is Live) becomes the Draft of its next number.

**Given** mappings are matched to existing ones
**When** imported
**Then** matching is by TMF API definition (spec hash), method and path, not by name (FR-35).

**Given** backends and id-map entries in the file
**When** imported
**Then** they are added only when new, identical ones are skipped, and no Live mapping, backend or entry is overwritten or removed.

**Given** a mapping whose Mapping already has a Draft
**When** the request has no `onDraftConflict`
**Then** it returns `409` listing those mappings and nothing changes
**And** with `onDraftConflict=replace` the Draft is overwritten, with `skip` it is left and that mapping is not imported.

**Given** an export of a runtime
**When** imported into the same runtime and exported again
**Then** the import succeeds and, with Drafts handled, the round trip is proven by a test (export, import, export is byte-identical for unchanged state).

## Epic 4: Studio shell and configuration management

Inès signs in, sees her mappings, and manages backends, the id map, export and import in an accessible studio.

### Story 4.1: Scaffold the studio with design tokens, themes and test harness

As a developer,
I want a React, TypeScript and Vite studio with the design system and a Playwright plus axe harness,
So that every screen is built on the same accessible foundation.

**Acceptance Criteria:**

**Given** `studio/`
**When** I build it
**Then** it uses React 19.3.1, Vite 8.3.8 and react-aria-components 1.22.1 on Node 22+, and builds to the `studio-frontend` static image (AR-17)
**And** the image generates `config.json` (issuer, client id, API base) from environment at container start.

**Given** DESIGN.md
**When** tokens are implemented
**Then** all light and `-dark` colors, typography, radius and spacing tokens exist as CSS custom properties, using the system font stacks with no web font and no essential text below 14px (UX-DR1, UX-DR3)
**And** controls use `input-border`, and the focus ring is a 3px outline, 2px offset, with a 2px surface halo (UX-DR2).

**Given** the OS color scheme
**When** the studio loads
**Then** it follows the OS, a manual toggle overrides it, and the choice is remembered per browser (NFR-4)
**And** forced-colors mode uses borders and outline, not background alone.

**Given** the layout
**When** viewed at 1280px or wider
**Then** a 340px inspector column with a 16px gutter is supported
**And** below 1024px and at 200% and 400% zoom (down to 320 CSS px) the inspector and Help panel stack as disclosures, with only the mapping table scrolling sideways in a labeled `tabindex="0"` region (UX-DR4).

**Given** all user-facing strings
**When** authored
**Then** they come from an externalized English catalog, never hard-coded, with the voice rules of UX-DR37 (verbs on buttons, no exclamation marks)
**And** a Playwright test with axe-core runs against the shell in CI.

### Story 4.2: Sign in with OIDC and handle session states

As an analyst,
I want to sign in through my company OIDC and keep my work if my session ends,
So that access is secure and I never lose edits.

**Acceptance Criteria:**

**Given** an unauthenticated visit
**When** the app opens
**Then** the Sign in surface shows and signing in uses authorization code with PKCE against the configured issuer (FR-24)
**And** while redirecting it shows "Taking you to your company sign-in…" with the button `aria-disabled`.

**Given** a failed or denied sign-in
**When** the user returns
**Then** the four-part error "Sign-in didn't complete." shows with focus on the error heading and a Sign-in help link (UX-DR32).

**Given** a signed-in account without access
**When** the auth server denies
**Then** the surface shows "Your account signed in, but it doesn't have access to TMF-Bridge Studio." naming the account.

**Given** an expired session while editing
**When** it ends
**Then** a non-modal banner "Your session ended." with `role="alert"` appears, focus stays, "Sign in again" is the next Tab stop
**And** after sign in the user returns to the same surface, step and selection with unsaved work restored from the browser.

**Given** a server `403`
**When** received
**Then** the four-part error pattern is used.

### Story 4.3: Build the app shell and shared components

As an analyst,
I want a consistent top bar, skip links and clear error, banner and dialog behavior,
So that every surface feels the same and is operable by keyboard.

**Acceptance Criteria:**

**Given** any authenticated surface
**When** it renders
**Then** the top bar shows in fixed order: logo and home, Backends, Id map, Help `?`, Dark mode toggle (`aria-pressed`), account menu (Sign out only); the current page link has `aria-current="page"`, bold weight and a 3px primary underline (UX-DR5).

**Given** every surface
**When** I press Tab first
**Then** skip links appear in order: main content, inspector (workspace), share bar (Exchange view) (UX-DR6).

**Given** a navigation or step change
**When** it completes
**Then** focus moves to the surface `h1` (`tabindex="-1"`) and the document title follows "{Step} (step {n} of 6) - {mapping} - TMF-Bridge Studio" on workspace steps (UX-DR33)
**And** each surface has exactly one `h1` and panels use `h2`.

**Given** the shared components
**When** used
**Then** Banner (inline, never modal), Toast (`role="status"`, at least 6 s, pauses on hover and focus, dismissible, confirmations only), Confirm dialog (one level, default focus on the safe action), Skeleton row (announced once, no shimmer under reduced motion) and Toggle exist (UX-DR27).

**Given** an error after a user action that replaces or reloads a view
**When** shown
**Then** it uses the four-part pattern with focus moved to the error heading in a labeled region and no `role="alert"`; an error appearing in place without a user action uses `role="alert"` and focus does not move; never both (UX-DR28).

**Given** reduced motion and offline
**When** active
**Then** no transitions run, and offline shows the banner "You're offline. Edits are kept in this browser." with a Retry button and no retry loops
**And** modified shortcuts only, no single-character shortcuts (UX-DR38).

### Story 4.4: List and open mappings

As an analyst,
I want a Mappings list as my home screen,
So that I can find, sort and open any mapping.

**Acceptance Criteria:**

**Given** signed in
**When** I land
**Then** the Mappings list shows columns name (a link), TMF API and operation, backend, status, sorted A-Z by name by default (FR-35)
**And** every header is a sort button with `aria-sort` and a polite announcement, and there is no recency column or last-opened behavior (UX-DR29).

**Given** the status column
**When** rendered
**Then** the Status badge shows Draft (hollow circle icon) or Live (check icon) with text, never color alone (UX-DR26).

**Given** no mappings
**When** the list loads
**Then** it shows "No mappings yet." with New mapping, "Import a configuration" and a `?` link
**And** while loading it shows five skeleton rows announced as "Loading mappings".

**Given** a mapping that cannot be opened or no longer exists
**When** I open it
**Then** the four-part error "We couldn't open {name}." or "{name} no longer exists. Someone may have deleted it." appears with a back-to-list action.

**Given** each row
**When** rendered
**Then** the name link or row click opens the mapping (the Export row action is added in Story 4.7).

### Story 4.5: Manage backends

As an analyst,
I want to declare, edit and delete backends,
So that mappings can select one and I never type a secret.

**Acceptance Criteria:**

**Given** the Backends screen
**When** empty
**Then** it shows "No backends declared yet." with "Declare backend" and, when no secret names exist, the tip "Secret names are provisioned by your tech team." (UX-DR30).

**Given** the add and edit form
**When** I fill it
**Then** fields are name, endpoint, auth type (bearer, basic, api-key), secret name from a picker listing provisioned names only, optional timeout and optional legacy OpenAPI upload
**And** with no names the picker is disabled with "No secret names are provisioned. Ask your tech team to create one."

**Given** an invalid form
**When** I submit
**Then** field errors use `aria-invalid`, a summary shows at the top, a bad URL reads "This doesn't look like a web address. Try https://…", Save is `aria-disabled` with the reason and shows a loading label while saving.

**Given** a backend used by mappings
**When** I edit it
**Then** the note "{n} mappings use this backend. They use the new values on their next call." lists them as links
**And** delete is blocked, Delete stays visible and `aria-disabled` with the reason, and the four-part message lists the mappings.

**Given** an unused backend
**When** I delete it
**Then** a confirm dialog "Delete {backend}? No mapping uses it." has default focus on Cancel, and success reads "{backend} deleted." with focus back on the list.

**Given** an invalid legacy OpenAPI upload
**When** rejected
**Then** the message names the first error and its line, the prior selection is unchanged and WSDL and SOAP are not accepted.

### Story 4.6: Manage the id map

As an analyst,
I want to view, add, edit, delete and import id-map entries,
So that the id-map transform has data.

**Acceptance Criteria:**

**Given** the Id map screen
**When** empty
**Then** it shows "No id-map entries yet." with "Add an entry", "Import entries" and the tip about the id-map transform (UX-DR31).

**Given** the list
**When** shown
**Then** it is sortable by every column with `aria-sort`, defaults to resource then TMF id, and announces "Sorted by TMF id, ascending" in a polite status line.

**Given** the add and edit form
**When** fields are empty or a clash exists
**Then** required fields are marked, the clash is checked on blur and announced politely, on Save error focus moves to the first invalid field, Save is `aria-disabled` with a reason
**And** the clash message reads "TMF id {id} already maps to {backend id} for {resource}." and the reverse clash (two TMF ids to one backend id) is explained the same way, never overwriting (FR-39).

**Given** an identical entry
**When** added
**Then** it is accepted and changes nothing.

**Given** Edit {TMF id} and Delete {TMF id} row actions
**When** used
**Then** edit saves in place and takes effect on the next call, and delete asks "Delete the entry {TMF id} to {backend id} for {resource}? Calls with this id will return 502 until it is added again." with default focus on Cancel and success "Entry deleted."

**Given** an entries import
**When** it has any clash or invalid row
**Then** "Import rejected. Nothing changed." shows with each problem and its location and focus on the banner heading
**And** success reads "Imported {n} entries, {m} already present." in a polite status line.

### Story 4.7: Export and import configuration from the studio

As an analyst,
I want Export all, per-mapping Export and Import in the studio,
So that I can back up, hand over and promote configuration without a developer.

**Acceptance Criteria:**

**Given** the Mappings list
**When** I choose Export all or a row's "Export {name}"
**Then** the file downloads from the same REST endpoints, is deterministic, and contains secret names only (FR-14, FR-15, FR-38)
**And** a failure shows the four-part "We couldn't create the export." with retry.

**Given** an import file
**When** I choose Import
**Then** a confirm dialog states what changes, and a rejected file shows "Import rejected. Nothing changed." with each problem and its location, focus on the heading, running configuration unchanged
**And** a success shows "Imported {n} mappings as Draft. Review them, then apply." with each imported mapping showing the Draft badge.

**Given** the import response is `409` for existing Drafts
**When** shown
**Then** the dialog lists those mappings and asks the user to choose replace or skip, then resends with `onDraftConflict`.

**Given** Id-map entries
**When** a per-mapping export is made
**Then** the UI explains entries travel only in Export all (UX-DR29).

## Epic 5: Inès authors and applies a mapping in the studio

Inès goes from Pick API to "Version {n} is live" with no code, in an accessible mapping workspace. Test (Epic 6) is optional here: a mapping can be applied without it.

### Story 5.1: Open the mapping workspace with stepper, modes, inspector and autosave

As an analyst,
I want a workspace with the six steps, two modes and a persistent inspector,
So that I can move through a mapping at my own pace and never lose work.

**Acceptance Criteria:**

**Given** a new or existing mapping
**When** the workspace opens
**Then** a stepper shows six numbered steps (Pick API, Declare backend, Map request, Map response, Test, Apply), the current one marked `aria-current="step"` with a filled marker and 3px bottom bar (no outline), completed steps as links, warnings as icon and count, wrapping instead of scrolling (UX-DR7)
**And** an existing mapping lands on Map request, with earlier steps editable (FR-10).

**Given** the mode switch
**When** I toggle Guided steps and Side-by-side
**Then** two `aria-pressed` buttons switch the layout, selection, scroll and unsaved edits are preserved, and the choice is remembered per browser (UX-DR15, NFR-4)
**And** Side-by-side stacks editor above test below 1024px (UX-DR4).

**Given** the inspector
**When** nothing is selected
**Then** it shows "Select a row to see its source, transform and target." with a tip
**And** it is collapsible with state remembered per session, and below 1024px it is a disclosure (UX-DR14).

**Given** edits
**When** I change the Draft
**Then** it autosaves with `If-Match`, a persistent "Saved." indicator (`role="status"`, not a toast) shows beside a "Save now" button (FR-33, UX-DR35)
**And** on `412` the stale-version banner shows with the PRD text "Someone else changed this mapping. Reload the latest version before you apply.", unsaved edits are kept in the browser and the action is "Reload the latest version" (FR-32).

**Given** the Test step before Epic 6 ships
**When** I open it
**Then** it shows a short note that testing is not available yet and Next still reaches Apply (warnings and absent tests never block).

**Given** a disabled action
**When** shown
**Then** it is `aria-disabled` (still focusable) with the reason as text linked by `aria-describedby`
**And** each step change moves focus to the `h1` "Step {n} of 6: {name}" (UX-DR33).

### Story 5.2: Pick the API and operation

As an analyst,
I want to choose a TMF API and one of its operations,
So that the mapping is bound to the right contract.

**Acceptance Criteria:**

**Given** step 1
**When** it loads
**Then** a skeleton select and "Loading TMF API definitions." show, then the stored APIs and operations from the official OpenAPI list (FR-10)
**And** there is no upload in this step.

**Given** I pick an API and an operation (for example `GET /productOrder/{id}`)
**When** I continue
**Then** the Mapping is created via the API with a name I enter, and the choice feeds the Mappings list column "TMF API and operation" (UX-DR34)
**And** a duplicate name or operation shows the four-part error and nothing is created.

**Given** an existing mapping
**When** I revisit this step
**Then** the choice is shown and can be changed with a warning that mapped rows may no longer fit.

### Story 5.3: Declare the backend

As an analyst,
I want to select an existing backend or add one inline,
So that the mapping knows where to call without me retyping an endpoint.

**Acceptance Criteria:**

**Given** step 2
**When** it loads
**Then** existing backends are listed with endpoint and secret name read-only, and I select one (UX-DR34)
**And** the endpoint is never re-typed (WCAG 3.3.7).

**Given** the inline "Add a backend" action
**When** I use it
**Then** it offers endpoint, auth type, secret name from the picker and optional legacy OpenAPI upload, reusing the Backends form behavior (REST and JSON only, WSDL and SOAP not accepted).

**Given** an invalid OpenAPI upload
**When** rejected
**Then** the four-part error names the first error and line and the prior selection is unchanged.

**Given** the step name
**When** shown
**Then** it is exactly "Declare backend".

### Story 5.4: Show the mapping table and field lists

As an analyst,
I want a mapping table with field lists on both sides,
So that I can see and edit every row as the source of truth.

**Acceptance Criteria:**

**Given** Map request or Map response
**When** it loads
**Then** a native `<table>` (not an ARIA grid) shows columns TMF field, transform, backend field, with the first cell a `th scope="row"` holding one row-select button (UX-DR8)
**And** an empty table reads "Nothing mapped yet. Drag a field, or use Connect." with no auto-suggestion.

**Given** a selected row
**When** chosen
**Then** it gets `info-bg`, a 4px primary bar, 2px primary border, hidden text "selected" and `aria-current="true"`, and the inspector fills with source, transform, target and tip
**And** the link is described in text via `aria-describedby`.

**Given** rows
**When** I add or delete one
**Then** it works with no reordering, rows keep 12px padding and the table sits in a labeled focusable scroll region at narrow widths.

**Given** the field lists
**When** shown
**Then** TMF inputs or attributes (from the spec) and backend fields (from the legacy OpenAPI or the saved sample) are focusable, show their type and each has a Connect button of at least 24px (UX-DR9).

**Given** Tab order
**When** I tab through a row
**Then** it matches reading order and Enter or Space activates controls (UX-DR38).

### Story 5.5: Connect fields with the Connect picker

As an analyst using a keyboard or pointer,
I want a Connect picker that creates mapping rows,
So that every mapping action works without dragging.

**Acceptance Criteria:**

**Given** a selected row or a field item
**When** I press its Connect button or `Alt+C`
**Then** a popover opens with a labeled search field and a listbox (`aria-activedescendant`) built with React Aria (UX-DR11)
**And** the Connect button is named "Connect a field to row {name}".

**Given** I choose a source and a target
**When** I confirm
**Then** a row is created and selected, the Draft autosaves
**And** Esc closes the picker and returns focus to its opener.

**Given** a source number and a target text
**When** I choose them
**Then** a non-blocking note "{source} is a number and {target} expects text." offers an "add conversion" button, and the row can still be created.

**Given** shortcuts
**When** used
**Then** `Alt+C` works inside inputs, the key is defined in one shared shortcut registry (listed in the Help panel by Story 7.1) and a browser, OS and screen-reader conflict check is recorded in the story notes.

### Story 5.6: Drag and drop with the wiring layer

As an analyst using a pointer,
I want to drag a field onto an attribute and see the link for the selected row,
So that mapping feels visual while the table stays the truth.

**Acceptance Criteria:**

**Given** a legacy field and a TMF attribute
**When** I drag one onto the other (either direction)
**Then** a table row is created, using React Aria drag and drop, with drag directions per request and response (UX-DR10)
**And** the same result is achievable via Connect (NFR-2).

**Given** no row selected
**When** the layer renders
**Then** no lines draw at rest, selecting or focusing a row draws one labeled link (source, transform, target), hover previews faintly
**And** warning links are dashed with an icon.

**Given** the "Show all links" toggle (off by default, `aria-pressed`)
**When** more than eight links exist
**Then** a count shows instead of drawing them, and never a mesh of arrows.

**Given** the layer
**When** exposed to assistive tech
**Then** it is `aria-hidden` and the table announces the link in text.

### Story 5.7: Choose and edit transforms, including arrays

As an analyst,
I want to pick a transform from the closed list and configure it,
So that I can translate values, ids and nested arrays without code.

**Acceptance Criteria:**

**Given** a row
**When** I open the Transform picker
**Then** it lists the closed S1 list: direct copy or rename, constant, concatenation, conditional or default value, lookup table, type or format conversion, id-map, and map and filter for arrays, each with name, one-line explanation and a `?` link (UX-DR12, FR-5)
**And** each choice writes the matching node (`path`, `lit`, `concat`, `coalesce`/`if`, `lookup`, `convert`, `idMap`, `map`, `filter`) into the Draft.

**Given** an array rule
**When** shown
**Then** it renders as a nested sub-table inside the row, with item-relative fields, and `/$root/...` references are shown readably.

**Given** `@type`, `@baseType` and `@schemaLocation`
**When** mapped
**Then** they behave as ordinary attributes (FR-7).

**Given** a free-form expression
**When** attempted
**Then** the studio offers no such option and the API rejects `{"op":"expr"}`.

### Story 5.8: Warn when an id field is mapped without id-map

As an analyst,
I want a non-blocking warning when I map an id without the id-map transform,
So that I catch the most common cause of a 404.

**Acceptance Criteria:**

**Given** the mapping-lint endpoint returns a warning for a row
**When** the row renders
**Then** it has a dashed `id-map-warning` border, `-bg` fill, a warning icon plus text and an "Add id-map transform" action (UX-DR13, FR-6)
**And** it never uses error red.

**Given** the action
**When** pressed
**Then** the row's transform becomes `idMap` and the warning clears.

**Given** warnings exist
**When** I press Next or Apply
**Then** they are not blocked, the stepper shows the warning count, and the warning list is reachable.

### Story 5.9: Apply the mapping and confirm it is live

As an analyst,
I want to review what goes live and apply it,
So that my change serves on the next call with no restart.

**Acceptance Criteria:**

**Given** the Apply step
**When** it loads
**Then** a summary card shows mapping name, API and operation, backend and secret name, and a primary "Apply" with a secondary Back action to the previous step, labelled "Back to Test" (UX-DR22, FR-12)
**And** if a Live version exists a warning banner reads "This replaces the live version."

**Given** I press Apply
**When** in progress
**Then** the button reads "Applying…" and is `aria-disabled` (double submit ignored) with status announced.

**Given** success
**When** done
**Then** it shows "Version {n} is live. The next call uses it." with a link "Open Mappings list"
**And** a runtime call for that operation now uses the new version with no restart.

**Given** a stale version
**When** Apply is attempted
**Then** it is blocked with the FR-32 text, Apply stays visible and `aria-disabled` with the reason, and the action is "Reload the latest version"
**And** an apply failure shows the four-part error with state unchanged and the Back action.

### Story 5.10: Prove UJ-1 end to end

As a product owner,
I want an automated run of Inès adding a field to an existing mapping,
So that the authoring promise is verified.

**Acceptance Criteria:**

**Given** the seeded TMF620 mapping
**When** a Playwright test opens it, lands on Map request, maps one more backend field to a TMF attribute through Connect and applies
**Then** the Apply summary says "This replaces the live version" and the result reads "Version 2 is live"
**And** the next runtime call returns the new attribute with no restart and no developer.

**Given** the same run
**When** executed
**Then** it uses only the keyboard for the mapping actions and axe-core passes on each screen visited.

## Epic 6: Test, diagnose and share (verdict, Exchange view, lineage)

Inès knows a mapping works before it goes live, and can forward a failure to tech as a share bundle.

### Story 6.1: Preview a translation without sending anything

As an analyst,
I want to translate a pasted sample request or response without calling the backend,
So that I can check my mapping before touching a real system.

**Acceptance Criteria:**

**Given** a stateless preview endpoint
**When** I post a mapping (Draft) and a sample request
**Then** it returns the translated backend request and sends nothing to the backend (FR-28 a)
**And** posting a sample backend response returns the resulting TMF payload (FR-28 b).

**Given** the preview use case
**When** inspected
**Then** it has no write port and no draft or mapping repository (ArchUnit), may use only the read-only `IdMapReadPort`, and the mapping schema has no sample field (AR-5)
**And** after preview calls a grep of logs and the DB finds none of the sample values (NFR-6).

**Given** an id-map transform in the mapping
**When** previewed
**Then** it reads the id map without writing it.

### Story 6.2: Run a real read-only test call in guarded test mode

As an analyst,
I want to run the mapping against the backend safely,
So that the test can never change backend state.

**Acceptance Criteria:**

**Given** a studio test call
**When** it runs
**Then** it executes the same domain `CallPipeline` as production, with the backend-client port wrapped in a guard allowing GET only that fails before any network I/O (FR-13, AR-5)
**And** a test proves a non-GET mapping in test mode fails without sending.

**Given** a Draft
**When** tested
**Then** the control plane loads the Draft and hands it to the pipeline, and the data plane never loads a Draft.

**Given** the test response
**When** returned
**Then** it includes the HTTP request sent to the backend, the backend response, the resulting TMF payload and the overhead (FR-22, excluding lineage capture)
**And** secret values never appear, only the secret name.

### Story 6.3: Capture step results and field lineage on test calls

As an analyst,
I want to see each mapping step's result and where each TMF field came from,
So that I can find a wrong value quickly.

**Acceptance Criteria:**

**Given** a studio test call
**When** it runs
**Then** step results of both rule tables and the backend call are returned (FR-20) and each TMF field traces to its backend source and transformation (FR-21)
**And** nested `map` and `filter` rules are traced per item (FR-5).

**Given** a production runtime call
**When** it runs
**Then** it returns neither step results nor lineage (observer off).

**Given** step results and lineage
**When** produced
**Then** they pass through the Redactor with the same configurable sensitive paths as logs (FR-23)
**And** a secret-leak test injects a known secret and sensitive-path value and greps every output.

### Story 6.4: Compute the verdict server-side

As an analyst,
I want a plain-language verdict with what happened, likely meaning, what to try and who to ask,
So that I know what to do next without reading HTTP.

**Acceptance Criteria:**

**Given** a test sub-step (a, b or c)
**When** the verdict is computed
**Then** the closed rules apply per sub-step: id-map transform present, id-map entry found and empty mapped fields apply to a, b and c; OpenAPI validation is request side in a, response side in b, both in c; HTTP status, backend reachable, secret name provisioned and overhead apply to c only (FR-29, AR-14)
**And** a rule that does not apply is Skipped, never Passed.

**Given** a failed backend-reachable or secret-provisioned check
**When** computed
**Then** the verdict is failed.

**Given** overhead above 100 ms in sub-step c
**When** computed
**Then** an info note "Bridge overhead was {n} ms, above the 100 ms target." sits beside a pass and never blocks.

**Given** a pass
**When** computed
**Then** the text is "Checks passed. The output matches your sample and the TMF622 schema." with the API name substituted and never says conformant, compliant or certified (FR-9)
**And** every verdict has all four parts, and an id-map miss says the entry is missing so it is not mistaken for a backend `404`.

**Given** a backend `404`
**When** the verdict is computed
**Then** it says the backend could not find the order, that the request sent the id without the id-map transform if so, and to try adding it or ask the tech team.

### Story 6.5: Assemble curl, Postman and the share bundle on the server

As an analyst,
I want curl, Postman and a share bundle built from the exchange,
So that I can forward a failed call without exposing secrets.

**Acceptance Criteria:**

**Given** an exchange record posted by the studio
**When** I request curl and Postman
**Then** both replay the backend request with a `$SECRET_VALUE` placeholder, never a secret (FR-30, FR-16)
**And** Postman is collection JSON.

**Given** a bundle request
**When** built
**Then** it is one `.zip` with `manifest.json` (timestamp, formatVersion), `mapping.json`, `sample-request.json`, `sample-response.json`, `exchange.json`, `verdict.json` and `postman.json`, all but the manifest canonical JSON (AR-8)
**And** samples are masked at the FR-19 paths with `"***"` and the secret name is included, never a value.

**Given** the server
**When** the bundle is built
**Then** it stores nothing (NFR-6).

**Given** a leak test
**When** all outputs are grepped for an injected secret and sensitive values
**Then** none are found.

### Story 6.6: Test sub-steps 1 and 2 in the studio

As an analyst,
I want to paste samples and see the unfired previews, also while mapping,
So that I get fast feedback with nothing sent to the backend.

**Acceptance Criteria:**

**Given** Test sub-step 1 or 2
**When** I use the sample editor
**Then** it is a plain multi-line text field in code type that never traps Tab, with Save locally and Run (UX-DR16)
**And** invalid JSON shows line and column linked by `aria-invalid` and `aria-describedby`, and Run is `aria-disabled` with its reason.

**Given** a saved sample
**When** I refresh or sign out and back in
**Then** it persists in this browser only, never in the server, with the standing note "Samples are saved in this browser only." and a link to the share bundle (NFR-6)
**And** a lost sample shows "No saved sample in this browser..." with "Paste a sample" and "Import from share bundle".

**Given** a run
**When** it finishes
**Then** the banner "Nothing was sent to the backend." persists, with "Request mapped. Nothing was sent to the backend." for sub-step 1 (UX-DR17)
**And** the verdict bar renders the server verdict for that sub-step.

**Given** Side-by-side mode
**When** I edit a Map step
**Then** the matching preview re-runs debounced against the saved sample and one polite `role="status"` line summarizes it ("Test re-ran: 7 fields mapped, 1 warning")
**And** lineage is available as a nested list with "Lineage isn't available for this value yet." plus the reason when missing (UX-DR36).

### Story 6.7: Run sub-step 3 and read the Exchange view

As an analyst,
I want to run a real read-only call and see the verdict, tiles and three panels,
So that I can see exactly what happened and jump to the row at fault.

**Acceptance Criteria:**

**Given** sub-step 3
**When** it opens
**Then** the sample id is prefilled from the saved sample request and never retyped, nothing is called until I press the button, the button becomes "Calling backend…" and `aria-disabled`, cancel is available and a timeout message shows after the limit (UX-DR17).

**Given** the result
**When** the Exchange view opens
**Then** the verdict bar shows icon, heading, one sentence and a checklist with Passed, Failed or Skipped in words, with links to the failing check (UX-DR18)
**And** a re-run moves focus to the verdict heading.

**Given** the four summary tiles
**When** shown
**Then** Exchange latency (ms), Target endpoint (host only), Fields translated and Id-map status are a description list with label and value paired, hidden before a run, two columns below 1024px and one below 480px (UX-DR19)
**And** a missing id-map key shows "Missing key" in id-map-warning styling with a link "Go to the mapping row: id" moving focus to that row's Transform control.

**Given** the three panels
**When** rendered
**Then** the failing one is auto-expanded with a 2px error border and the others collapse to a one-line summary, code blocks are focusable labeled regions on `code-surface`, masked values show `••••` with hidden text "masked" and only the secret name shows (UX-DR20)
**And** a backend `404` expands the HTTP response panel.

### Story 6.8: Share from the Exchange view

As an analyst,
I want Copy as curl, Copy as Postman and Download bundle,
So that I can forward a failure to my tech colleague.

**Acceptance Criteria:**

**Given** the Exchange view
**When** it renders
**Then** a sticky share bar with a 3px primary top border shows Copy as curl, Copy as Postman, Download bundle and the secret name in code styling, in a labeled region with a skip link (UX-DR21)
**And** it un-sticks under 600px viewport height and page `scroll-padding-bottom` equals its height so focus is never hidden.

**Given** a copy action
**When** it succeeds
**Then** a toast "Copied." shows and no secret value is in the clipboard
**And** the note "Secrets are never included. Only the secret name is. Sensitive fields are masked in samples too." sits under the bar.

**Given** a download blocked or an export failure
**When** it happens
**Then** the four-part error appears with "Copy as curl" as the fallback.

**Given** a downloaded bundle
**When** I choose "Import from share bundle"
**Then** the samples are restored into this browser's sample storage and nothing is sent to the server for storage.

**Given** there is no send integration
**When** I look for one
**Then** none exists (copy and download only).

### Story 6.9: Prove UJ-5 end to end

As a product owner,
I want an automated run of the 404 diagnosis and hand-off,
So that the diagnostic promise is verified.

**Acceptance Criteria:**

**Given** a mapped TMF622 read without the id-map transform and sample order ORD-1
**When** a Playwright test runs sub-step 3
**Then** the Exchange view shows a failed verdict with the HTTP response panel expanded showing `404`, and the verdict says the backend could not find the order and suggests the id-map transform.

**Given** the same run
**When** the test jumps to the mapping row, adds the transform and re-runs
**Then** it still returns `404` because ORD-1 is not in the backend.

**Given** the test downloads the bundle and copies curl
**When** their contents are inspected
**Then** they hold the mapping version, samples, three panels, verdict and checks and the secret name, and no secret value.

## Epic 7: Help, glossary and user manual

A non-developer is never stuck: contextual help, glossary and the user manual ship inside the product, with CI accessibility gates.

### Story 7.1: Help panel with search and shortcuts

As an analyst,
I want a persistent Help panel that follows what I am doing,
So that I can get an answer without leaving my work.

**Acceptance Criteria:**

**Given** the top-bar `?` or `Alt+H`
**When** I open it
**Then** a non-modal complementary region of 340px shows, dockable, context-aware to the selected field, transform or step, with Help and Glossary tabs and a search field (UX-DR23)
**And** below 1024px it stacks as a disclosure.

**Given** a search with no match
**When** submitted
**Then** it shows "Nothing matched {query}." with "Browse the glossary" and a manual link.

**Given** the panel
**When** it lists shortcuts
**Then** it shows the current keys (`Alt+C`, `Alt+H`) and offers "Restore tips"
**And** Esc closes it and returns focus to the opener.

**Given** the manual cannot be opened
**When** a link fails
**Then** "We couldn't open the user manual." shows with retry and the built-in tip and glossary still work.

### Story 7.2: Inline tips, help links and glossary terms

As an analyst,
I want a tip and a `?` link beside every field, transform, step and check,
So that technical terms are explained once, in place.

**Acceptance Criteria:**

**Given** every field, transform and step
**When** rendered
**Then** it has an inline tip (one sentence, icon plus text, dismissible per tip, restorable) (UX-DR24, NFR-5)
**And** every step title, field, transform and verdict check has a `?` help link of 24px with accessible name "Help: {topic}" linking to the matching manual page, external links adding "(opens in a new tab)".

**Given** a glossary term in copy
**When** I hover or focus it
**Then** it is a real button with a dotted primary underline opening a popover with the definition and a Glossary link, the pointer can enter the popover and it persists until Esc, never a hover-only title tooltip.

**Given** the glossary
**When** opened
**Then** it covers payload, field path (JSON Pointer), transform, lineage, OpenAPI, id map, secret name, verdict, unfired preview and mapping version, each with a plain definition, an example and a manual link (FR-37).

### Story 7.3: Consistent help footer, Disclaimer and no-conformance guard

As an analyst,
I want the same help links on every screen and a clear disclaimer,
So that I always know where to go and what the product does not claim.

**Acceptance Criteria:**

**Given** every surface including Sign in and below the share bar on the Exchange view
**When** it renders
**Then** the footer shows in the same order Glossary, User manual, Disclaimer and "Need a hand? Ask your integration lead." (UX-DR25, WCAG 3.2.6).

**Given** the Disclaimer
**When** opened from the footer, the Help panel or the README
**Then** it states TMF-Bridge is an independent open-source project, not affiliated with, endorsed by or certified by TM Forum, and that "TMF" and "TM Forum" are used only to describe API compatibility (FR-9).

**Given** the repository
**When** CI runs
**Then** a check fails if studio copy, API descriptions or exports contain "conformant", "compliant" or "certified" (AR-17, UX-DR37).

### Story 7.4: Write the user manual and glossary content

As an analyst,
I want a static user manual served by the studio,
So that every help link resolves inside the product.

**Acceptance Criteria:**

**Given** `docs/`
**When** the studio image builds
**Then** the manual is static content bundled in the image and served by the studio, written by the project team (FR-37)
**And** anchors match every `?` link used in the studio, proven by a link-check in CI.

**Given** the content
**When** reviewed
**Then** it covers the six steps, Backends, Id map, Test sub-steps and verdict, Exchange view and share bundle, export and import, keyboard shortcuts, and the first-mapping guide linked from the empty Mappings list
**And** it contains the glossary entries with examples.

**Given** the manual
**When** read
**Then** it uses plain language, explains technical terms on first use and makes no conformance claim.

### Story 7.5: Gate accessibility and reflow in CI

As a maintainer,
I want automated accessibility and reflow gates,
So that WCAG 2.2 AA regressions fail the build.

**Acceptance Criteria:**

**Given** CI
**When** it runs
**Then** Playwright with axe-core runs on every studio screen in light and dark (NFR-2)
**And** a contrast check verifies the DESIGN.md pairs.

**Given** a Playwright reflow test
**When** it runs at 1280px, below 1024px, and at 200% and 400% zoom down to 320 CSS px
**Then** inspector and Help panel stack, only the mapping table scrolls sideways in its labeled region, and nothing else overflows (NFR-3, UX-DR4).

**Given** keyboard-only runs
**When** executed
**Then** every function in the six steps, Backends, Id map and Exchange view is completed without a pointer, with focus never lost or clipped (UX-DR38)
**And** forced-colors mode keeps selection, badges and banners distinguishable.

**Given** the banned list
**When** tests run
**Then** they assert no single-character shortcut, no auto-fired backend call, no modal deeper than one level and no auto-dismissing error.

## Epic 8: Zero-to-hero showcase and delivery

An open-source user clones the repo and has TMF620 served and TMF622 mapped in ten minutes; an operator can deploy it.

### Story 8.1: Full compose demo with Keycloak, studio and one origin

As an open-source user,
I want one `docker compose up` that starts everything on one origin,
So that I can try the product without any setup.

**Acceptance Criteria:**

**Given** `deploy/`
**When** I run `docker compose up`
**Then** Postgres, `tmf-bridge-api` (profile `control`), `studio-frontend`, `tmf-mock` and Keycloak 26.8.2 with a seeded user start, behind a reverse proxy giving studio, config API and TMF paths one origin (AR-4, AR-16, AR-18)
**And** the studio image writes `config.json` from environment so no rebuild is needed.

**Given** the seeded user
**When** I sign in to the studio
**Then** OIDC works with PKCE and the runtime accepts a token from the same issuer.

**Given** the stack
**When** it is up
**Then** TMF620 is served and TMF622 returns `501` until mapped.

### Story 8.2: README and the ten-minute showcase path

As an open-source user,
I want a README that walks me from clone to a mapped TMF622,
So that I see the product work in about ten minutes.

**Acceptance Criteria:**

**Given** the README
**When** followed from `git clone`
**Then** it covers prerequisites, `docker compose up`, signing in, calling TMF620, seeing `501` for TMF622, mapping TMF622 `GET /productOrder/{id}` in the studio and calling it successfully (FR-27)
**And** it states the ten-minute goal is a promise, not a metric.

**Given** the README
**When** it describes the product
**Then** it links the Disclaimer and makes no conformance claim.

**Given** the walkthrough
**When** executed by a fresh machine test
**Then** each documented command works as written.

### Story 8.3: Helm chart, probes and one image per environment

As an operator,
I want a Helm chart and proper probes,
So that I can run the bridge on Kubernetes and promote configuration by export and import.

**Acceptance Criteria:**

**Given** the chart
**When** installed
**Then** it deploys the API (values select the `control` profile) and the studio, with PostgreSQL operator-provided and one immutable image per component for all environments, differing only by values (AR-16)
**And** Flyway runs only on `control` instances or a pre-upgrade Job.

**Given** Actuator
**When** probed
**Then** liveness and readiness exist, readiness including DB reachability and schema version, and an older schema keeps data-plane pods not-ready without crash-looping
**And** Prometheus and OTLP exports are configurable.

**Given** a runtime-only deployment
**When** configured with `control` off and no studio
**Then** it serves the TMF path (AR-4).

### Story 8.4: CI pipeline for images and suites

As a maintainer,
I want CI to build the three images and run every suite,
So that architecture rules and quality gates are enforced on every change.

**Acceptance Criteria:**

**Given** a push
**When** CI runs
**Then** it builds `tmf-bridge-api`, `studio-frontend` and `tmf-mock`
**And** it runs the ArchUnit suites (AD-2, AD-3, AD-4, AD-13), Testcontainers Postgres tests, per-node contract tests, the export round trip, the secret-leak grep, Spectral on the generated control-plane OpenAPI and Playwright with axe-core (AR-19).

**Given** the AD-4 suite
**When** it runs
**Then** every studio request path and method is asserted to exist in the generated control-plane OpenAPI.

**Given** any suite failing
**When** CI finishes
**Then** the build fails.

### Story 8.5: Verify the 100 ms overhead target

As a maintainer,
I want a repeatable benchmark of bridge overhead,
So that NFR-1 is verified under its stated conditions.

**Acceptance Criteria:**

**Given** warm compiled-mapping and API-definition caches, a single bridge instance, a zero-latency mock and payload up to 50 KB
**When** the benchmark runs
**Then** it reports the p95 of `bridge.overhead` and passes when under 100 ms (NFR-1, AR-12)
**And** lineage and step-result capture are excluded.

**Given** the result
**When** published
**Then** the conditions are documented beside the number and no load or sizing claim is made.

**Given** runtime responses in the benchmark
**When** inspected
**Then** each carries `Server-Timing: bridge;dur=<ms>` consistent with the timer.

### Story 8.6: Prove UJ-2, UJ-3 and UJ-4 end to end

As a product owner,
I want an automated run of the whole showcase journey,
So that the zero-code promise is verified against the shipped demo.

**Acceptance Criteria:**

**Given** the compose stack with seed data
**When** a test calls TMF620 `GET /productOffering/{id}` and then TMF622 `GET /productOrder/{id}`
**Then** TMF620 is schema-valid with overhead under budget and TMF622 returns `501` with a TMF Error (UJ-2).

**Given** the studio
**When** a Playwright run picks TMF622, declares the existing backend, maps request and response (adding the id-map transform), tests and applies
**Then** it ends on "Version 1 is live" with no code written, built or redeployed (UJ-3).

**Given** the same call as before
**When** repeated
**Then** TMF622 returns a schema-valid response within the overhead budget and the partner contract is unchanged (UJ-4).
