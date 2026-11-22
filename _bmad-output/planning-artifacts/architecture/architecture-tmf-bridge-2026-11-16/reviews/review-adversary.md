# Adversarial review: ARCHITECTURE-SPINE (TMF-Bridge S1)

Method: for each pair, two units one level down (module or story) that each satisfy every AD to the letter and still do not fit together. Each pair ends with the AD to add or tighten.

## Verdict

Not ready to hand to parallel agents. Layering, plane split, immutability and secret handling are solid. The seams between units are open in six places: the shared call pipeline, the evaluation context (what a pointer points at), identity and versioning of Mapping, the Redactor versus the canonicalizer, the idMap contract, and routing and error edge cases. Ten holes below, ordered by damage. The first five would each cause a failed integration.

---

## H1. The call pipeline has two homes (AD-2, AD-3, AD-5, AD-11) — CRITICAL

- Unit A: `application/runtime` `ServeCallUseCase` (match spec, read live pointer, resolve backend and secret, call, validate, apply the AD-11 error table, time `bridge.overhead`).
- Unit B: `application/control` `StudioTestCallUseCase` (sub-step c: load Draft, call the real backend through the GET-only guard, return verdict inputs and overhead).
- Obeys: AD-3 (`application/control` and `application/runtime` never import each other), AD-5 ("same domain evaluator"; the evaluator is only table-in, table-out), AD-11 (error table), AD-12.
- Clash: AD-5 unifies the evaluator, not the orchestration around it. B cannot import A, so B re-implements spec matching, request validation, the error table, timeout handling and the overhead timer. The two drift: B maps a backend 4xx to 502, A to 404 on a typo; B includes secret resolution in "backend time", A does not; B validates the response, A (lazy) skips on cache hit. The studio then shows behavior production does not have, which is exactly what AD-5 claims to prevent.
- Fix: new AD. The whole call pipeline (validate request, request table, backend port, response table, validate response, error mapping, overhead measurement) is one domain service `CallPipeline`, parameterized by a `MappingSource` (live pointer for the data plane, a draft for control) and by decorator ports (test guard, lineage observer). Both use cases are thin shells. ArchUnit: no `application/*` class may call the backend-client port or the error table directly.

## H2. Evaluation context and node semantics are undefined (AD-6) — CRITICAL

- Unit A: evaluator + node implementations in `domain` (agent 1). Unit B: studio mapping editor and Connect picker emitting mapping JSON (agent 2). A third: Unit C, the JSON Schema author.
- Obeys: AD-6 verbatim (closed tree, `{target, expr}`, JSON Pointer everywhere, `path` carries `syntax: "pointer"`).
- Clash (all legal):
  1. Pointer root. A resolves `/id` against the TMF inbound body; the operation inputs also include path params, query and headers (UJ-3 says "query, header and JSON body"). B emits `/query/id`, `/header/x-tenant`, `/body/foo`; A treats root as body only, or as `{path, query, headers, body}` with different key names (`headers` vs `header`, `pathParams`). Same for targets: where in the backend request do `/url`, `/query`, `/headers`, `/body` live, and where do the backend method and URL template come from (no field is named)?
  2. Response table source root: backend body only, or `{status, headers, body}`? Verdict (FR-29 HTTP status) and `if` on a backend status need it.
  3. `map` and `filter`: is the pointer inside them relative to the current element or absolute? What is the loop variable? Both choices pass the schema.
  4. `lookup` carries its table inline, or by name (there is no LookupTable entity in the ERD and no owner)? `convert` parameters (date pattern syntax, `java.time` vs ISO, number rounding) are unnamed.
  5. In-memory JSON representation in the domain: Jackson `JsonNode` (AD-2 bans only "Jackson-annotated web types"), `Map<String,Object>`, or a custom sealed type. Number handling (BigDecimal vs double: `1.0` vs `1`), missing vs `null` vs absent array. A "path resolves to nothing is a mapping error, not a null" in scalar slots; what is an optional non-scalar slot? What does `coalesce` treat as "empty" (missing, null, empty string)?
  6. Mapping error status. AD-6 introduces "mapping error", AD-11's fixed error table has no row for it. A returns 500, B returns 502.
- Fix: tighten AD-6 with an appendix owned by the schema: the evaluation context object (named roots), the backend request target model (method, URL template, query, headers, body), the response context, iteration scoping, the data types and null/absent semantics, one JSON value type in domain (e.g. Jackson `JsonNode` explicitly allowed as a domain dependency, or a domain `JsonValue`), and one golden-file contract test per node (already required) written from that appendix. Add a row to the AD-11 table for mapping error (suggest 502, TMF Error).

## H3. "version" is three different things; Mapping has no identity (AD-7, AD-8, FR-18, FR-31, FR-32) — CRITICAL

- Unit A: config API mapping CRUD and autosave (story: "save Draft"). Unit B: Apply use case. Unit C: import/export (FR-15, 17, 18, 38). Unit D: runtime compiled-mapping cache.
- Obeys: AD-7 (immutable applied versions, integer `version` column surfaced as ETag, cache keyed `(mappingId, version)`), AD-8, FR-31.
- Clash:
  1. AD-7 says "integer `version` column" for optimistic locking, and the cache key `(mappingId, version)` and the PRD ("Version 2 is live") use a business version number. A maps the JPA `@Version` on `MappingVersion` to the business number, so every autosave bumps it, "Version 2" becomes "Version 7", and D's cache key (still correct, since applied rows are frozen) shows a gap in the numbers. B keeps a separate `versionNumber` and makes `version` the lock only on `Mapping`. Which resource's ETag does the studio hold for stale-apply detection (FR-32): Mapping (live pointer moves), MappingVersion (draft edits), or both?
  2. Draft cardinality: one open Draft per Mapping, or many? A autosave updates in place; C's import "becomes the Draft of its next version" inserts a new row. Two Drafts of the same next version, or an overwrite of the analyst's unsaved work (violates FR-32's "unsaved edits are kept")?
  3. Natural key. FR-18 forbids generated ids in exports, AD conventions say ids are server-generated opaque strings, and no field identifies a Mapping across environments. A matches by name, C by (spec hash, operation), the config API allows duplicate names. Round trip export/import creates duplicates or clashes. The same hole exists for ApiSpec in import (AD-8's clash list omits specs) and Backend references.
  4. Status is derived (live pointer non-null) or stored? Stored drifts from the pointer.
  5. Whole-config export content: AD-7 preserves old applied versions; FR-15 says "Draft and Live". Does the export carry superseded versions? And ordering of arrays: the canonicalizer fixes key order only, not collection order (mappings, id-map entries, specs), so determinism (FR-18) is not guaranteed.
- Fix: new AD "Identities and versions". Define: `Mapping` natural key (`apiSpecKey` + operation, unique), `MappingVersion.number` (business, monotonic per Mapping) distinct from `etagVersion`/lock column, ETag scope for each resource, at most one open Draft per Mapping (partial unique index), status derived from the pointer, export sets (Live + Draft only) and a mandatory sort for every collection in the canonicalizer.

## H4. Redactor versus canonicalizer, and who builds the bundle (AD-8, AD-10, AD-4) — HIGH

- Unit A: export use case (whole and per mapping). Unit B: share-bundle use case. Unit C: studio client (curl/Postman copy, download bundle).
- Obeys: AD-8 (only through the canonicalizer; bundle layout), AD-10 ("logs, step results, lineage, exports and bundles all pass through [the Redactor]"; Redactor knows resolved values and configurable sensitive paths), AD-4 (studio is a pure client).
- Clash:
  1. A passes the mapping export through the Redactor as AD-10 demands. The Redactor's registry contains whatever secrets happened to be resolved in this JVM, plus the configurable sensitive paths. Result: the same stored state exports differently on a fresh instance versus a warm one, or a mapping literal containing `password` is masked and the exit-path export is corrupted. FR-18 determinism broken. B does not redact `mapping.json` (it is config, not a sample). Same artifact, two behaviors.
  2. Mask token undefined: logs `***`, bundle `$SECRET_VALUE` (specified only for secrets), masked sample fields in a bundle `"[REDACTED]"`, `null`, or removed? A and B choose differently, round-trip and gold files diverge.
  3. The bundle is assembled by server (canonicalizer in Java domain) or by the browser (JSON.stringify, zip.js)? Samples exist only in the browser (FR-28), the server must not store them, the canonicalizer is server-side. C builds the zip client-side with its own key sort; B builds server-side from a posted sample. Both comply with AD-4/AD-8 text. The same question for curl and Postman generation.
- Fix: tighten AD-8/AD-10. Config exports (whole, per mapping) are never redacted, they contain only names (assert by schema); the Redactor applies to runtime-derived data (logs, step results, lineage, exchange, samples). Name one mask token per surface. State that bundle, curl and Postman are produced by a stateless control endpoint that receives the samples in the request body, uses the canonicalizer and the Redactor, and returns bytes; the studio assembles nothing.

## H5. idMap and the "preview has no repository" rule (AD-5, AD-6, AD-7, AD-15) — HIGH

- Unit A: `idMap` node implementation plus the `IdMapPort` (JPA adapter). Unit B: `POST /preview` use case. Unit C: the Id map screen API (FR-39). Unit D: verdict rule "id-map-transform-present".
- Obeys: AD-5 (preview "no repository dependency", ArchUnit-checked on that use case), AD-6 (`idMap` node), AD-7 (IdMapEntry unique `resource + tmfId`).
- Clash:
  1. B needs `idMap` to translate the sample, but has no repository. B1 injects `IdMapPort` (a port, not a "repository": ArchUnit passes by name) and so reads the DB, contradicting "persists nothing" in spirit. B2 injects a no-op that passes ids through, so the preview shows output that production will not produce. Both obey the letter.
  2. Direction and miss behavior. The node name `idMap` does not say tmf-to-backend vs backend-to-tmf (FR-6: "and back"). The unique constraint is on `(resource, tmfId)` only, so the reverse lookup `backendId -> tmfId` is not unique and not indexed. On a miss: mapping error, 404 (AD-9 "unknown id" is 404), pass-through? A chooses 404, A' (response side) chooses a mapping error 502.
  3. `resource` key: the ERD ties it to an `API_SPEC` row, which is immutable per hash. A new spec version of the same API orphans the entries. Unit A keys by resource name string (`productOrder`), the node carries the spec row id or a different casing. AD-1 forbids code naming TMF resources, so it must be data, but its source is not stated.
  4. Caching: AD-7's "no invalidation protocol" covers compiled mappings. Id-map edits must apply on the next call (FR-39) and Backends are resolved by name at call time; a unit may add a Caffeine cache for id-map lookups to meet the overhead target, which breaks FR-39.
- Fix: new AD for the `idMap` node contract: `{direction, resource}` fields, resource defined as an API-level key independent of spec hash, unique index both ways (or declare reverse lookups unsupported in S1 and reject), miss behavior fixed (suggest: request side 404, response side mapping error), no caching of id-map or backend rows in S1, and preview receives the id-map and lookups through a read-only `IdMapReadPort` explicitly allowed (or the sample payload carries the entries), stating which.

## H6. 501 versus 404, and the routing table (AD-9, AD-13, AD-1) — HIGH

- Unit A: routing table derivation and catch-all handler. Unit B: seeder / spec upload. Unit C: Spring Security chain for the runtime.
- Obeys: AD-9 ("An unmatched operation returns 501, a matched operation with an unknown id returns 404"), AD-13 (chains selected by `securityMatcher` on path prefix), AD-1.
- Clash:
  1. TMF622 is seeded on first boot, so its operations are matched by the routing table, yet UJ-2 requires 501 before it is mapped. A: no live mapping gives 501. B (reading AD-9 literally, "matched" means in the spec): 404. Draft-only mapping: 501 or 404? No wording covers "operation in spec, no live mapping".
  2. Two stored spec rows of the same API and version (a re-upload with a changed hash): AD-9 says v4 and v5 may coexist but nothing prevents two identical `(api, version)` entries, giving ambiguous routing or silent shadowing. "Version collisions" is claimed as prevented but has no rule.
  3. `securityMatcher` uses a static path prefix while the routing table is dynamic from each spec's `servers`. A spec whose server path starts with the control prefix (or `/studio`) is shadowed by the control chain or, worse, served under the control chain. Unmatched paths: does the runtime chain apply (401 first) or is an unknown path 501? Unit C protects `/**` minus control; Unit A returns 501 for anything unmatched; unauthenticated callers see 401 for a typo.
- Fix: tighten AD-9: define precisely "configured" as "has a live mapping", the order of checks (auth, route match, live mapping, validation, backend), one active spec per `(api, version)`, upload rejects reserved prefixes (control prefix, studio, actuator), and the runtime chain's matcher is the complement of reserved prefixes.

## H7. Verdict has no owner, and test-mode output shape is open (AD-4, AD-5, AD-14, FR-29/30) — MEDIUM

- Unit A: `application/control` test use case emitting a result. Unit B: studio verdict component.
- Obeys: AD-4 (studio calls only public API, logic the REST API lacks is forbidden), AD-14 (rules per sub-step).
- Clash: AD-4 forbids studio logic that the REST API lacks, yet the verdict is rules-based plain language and also lands in `verdict.json` in the bundle. A computes verdict server-side (a `verdict` object with check ids, outcomes and text keys). B recomputes it in TypeScript from raw outputs to render the wording and "who to ask". The two disagree on the closed rule set (skipped vs passed, AD-14) and the info-versus-fail severity for overhead. The response format of the test endpoint (tiles, lineage, steps) is not an AD either, so the studio and the API are built against different shapes.
- Fix: new AD: the control plane computes the verdict (rule ids, outcome enum `passed|failed|info|skipped`, parameters), the studio only renders text from i18n keys; the test and preview response is one published schema (steps, lineage, exchange, verdict, overhead), added to the Spectral-linted OpenAPI.

## H8. Error body shape for the control plane (Conventions, AD-8, AD-11) — MEDIUM

- Unit A: import endpoint. Unit B: global `@RestControllerAdvice` for the control plane.
- Obeys: Conventions ("Error bodies are the TMF Error object, no internal detail") and AD-8 ("reports every error with a JSON Pointer location").
- Clash: the TMF Error object has no list of located problems, A returns `{errors:[{pointer,message}]}`, B wraps everything as TMF Error and drops the list or squeezes it into `message`. Also 412/409 for ETag failures and the stale-apply message (FR-32) have no stated body. Also the 400 on the runtime: do consumers get violation details (AD-11 gives "details to logs only" for 502 only)?
- Fix: tighten Conventions: runtime errors use TMF Error; control plane errors use one `ProblemDetail`-like shape (with `violations[{pointer,message}]`) declared in the OpenAPI. State what a runtime 400 may carry.

## H9. Overhead boundary (AD-12, AD-5, AD-10) — MEDIUM

- Unit A: runtime timer placement. Unit B: test-mode guard/backend decorator.
- Obeys: AD-12 (total minus backend call time, one timer, lineage excluded), AD-5.
- Clash: "backend call time" is not delimited. A measures around `RestClient.exchange` only. B measures inside the guard decorator, which includes secret resolution (a file read) and the GET-only check. The DB reads of the live pointer, backend row and id map are counted as overhead (honest) in A, hidden in B's warm cache. The 100 ms claim is thus not comparable between the studio value and production. Timeout cases: when the backend times out, is the 5 s counted as backend time?
- Fix: tighten AD-12: backend time is the span between the first byte written and last byte read by the HTTP client, excluding secret resolution and guards, DB reads count as overhead, and the timer sits in `CallPipeline` (H1) only.

## H10. Runtime-only deployment has no seeder and no migration owner (AD-3, AD-9, Deployment) — LOW

- Unit A: Helm chart values for runtime-only. Unit B: seeder.
- Obeys: AD-3 (data plane never migrates, fails fast on old schema), AD-9 ("the image seeds TMF620 and TMF622 on first boot"), seeder "under `control`".
- Clash: the image seeds, but the seeder bean is control-only; a runtime-only deployment against a fresh database has no schema (no migration), no specs, and fails fast forever. "Required" schema version is defined nowhere (Liquibase changeset id? count?). Two control replicas booting together run the seeder concurrently with no stated lock.
- Fix: say that the Helm pre-upgrade Job runs Liquibase and the seeder (same `control` image/profile in one-shot mode), define the required-schema check (latest changeset id compiled in), and make the seeder idempotent with an advisory lock or unique natural keys.

---

## Suggested new or changed ADs (summary)

| Hole | Action |
| --- | --- |
| H1 | New AD: `CallPipeline` domain service shared by both planes |
| H2 | Tighten AD-6: evaluation context, backend request model, iteration scope, value type, null/absent, `lookup`/`convert` params; add mapping-error row to AD-11 |
| H3 | New AD: identities and versions (Mapping natural key, business version vs lock column, ETag scope, one Draft, derived status, export set and collection ordering) |
| H4 | Tighten AD-8/AD-10: config exports unredacted, mask token per surface, bundle/curl/Postman built server-side by a stateless endpoint |
| H5 | New AD: `idMap` node contract, resource key, miss behavior, no caching, preview access |
| H6 | Tighten AD-9/AD-13: define "configured", check order, one active spec per api/version, reserved prefixes |
| H7 | New AD: verdict computed by control plane, one published test/preview response schema |
| H8 | Tighten Conventions: control-plane error shape with located violations |
| H9 | Tighten AD-12: boundaries of backend time |
| H10 | Tighten AD-3/AD-9: who migrates and seeds in runtime-only deployments |
