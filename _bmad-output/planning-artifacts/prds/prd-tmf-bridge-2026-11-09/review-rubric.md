# PRD Quality Review — TMF-Bridge (re-run, 2026-11-15 edits)

## Overall verdict
All six highs from the prior review are closed. FR-39 gives the id map a population path. FR-34 states "S1 backends are REST and JSON only". FR-19 defines sensitive data. FR-28 now says the server translates a sample and "never stores it". The demo line is `GET /productOrder/{id}`. Success Measures are now "UJ-1 to UJ-5". The thesis, closed lists, export and import rules, and addendum trail remain strong. What is left is a long tail of medium gaps. The main ones are undefined mechanisms (overhead, error status table, Draft and Live as one stored entity, import identity), the new "never stores" claim not being reconciled with logs and step results, and a UX spine that has not caught up with FR-28, FR-34 and FR-39. No high or critical findings remain. The PRD is ready for architecture and story work if the mediums are tracked.

Dimension verdicts: Decision-readiness adequate. Substance over theater adequate. Strategic coherence adequate (improved). Done-ness clarity adequate (thin on runtime FRs). Scope honesty adequate (improved). Downstream usability adequate. Shape fit adequate.

## Prior highs: status
- RESOLVED: Success Measures deferred. They are now "User journeys UJ-1 to UJ-5 are the success criteria ... using configuration only". A residual is logged as medium below.
- RESOLVED: unfired preview versus "never on the server". FR-28 and NFR-6 now say the server "processes a sample but never stores it". New follow-ons are logged below.
- RESOLVED: id map had no population path. FR-39 adds an Id map screen (view, add and import), with REST parity and demo seeding. FR-6 points at it. New follow-ons are logged below.
- RESOLVED: "sensitive data" undefined. FR-19 now reads "every secret value plus the values at a configurable list of field paths", and FR-23 reuses it. Gaps are logged below.
- RESOLVED: `GET /productOrder` collection read. UJ-4 and the closing line now use `GET /productOrder/{id}`. A low remains in the UX scope note.
- RESOLVED: S1 backend protocol. FR-34 says "REST and JSON only. WSDL and SOAP are Later", and FR-4 now says "HTTP and transport faults".
- REMAINING highs: none.

## Decision-readiness — adequate
Decisions are stated as decisions (FR-33 "A Draft is stored and never served", FR-17 "Import never changes the live configuration", FR-34 "never typed", FR-36). Rejected alternatives live in the addendum. The PRD body still has no Open Questions, `[NOTE FOR PM]` or `[ASSUMPTION]` tags, although it is marked `status: final`.

### Findings
- **medium** Open items are still only in the addendum (PRD body, no section; addendum "Adapter versus system of record", "Scale-out and consistency", "how a legitimate id-map merge or restore of a populated runtime is done") — FR-39 adds a way to add entries but not a restore path, so the exit path (FR-15) still cannot rebuild a populated runtime. *Fix:* add a short "Open for architecture" list to the PRD.
- **low** Costs are stated as consequences, not trade-offs (FR-17, FR-31 "Apply replaces the live version", FR-34) — there is no rollback in S1, so a bad Apply stays live until re-edited. *Fix:* one "given up" line per major decision.

## Substance over theater — adequate
No persona theater (Inès and Tomás both drive requirements). NFR-2 to NFR-6 carry real bounds (WCAG 2.2 AA, 1280 px, 200% and 400% zoom, 100 ms). The closed lists in FR-5 and FR-29 are real scope work.

### Findings
- **medium** "Telco grade" is still a claim with no S1 content (Vision para 1: "at telco grade"; NFR last bullet drops throughput and availability; Later "Platform: TLS ... scale-out") — S1's only quality bound is 100 ms. *Fix:* define S1 as reference-grade (single instance, mock only) or drop the phrase.
- **low** The "competitive advantage" (Vision: "configuration over code") is generic. The stronger wedge is runtime OpenAPI validation plus the exit path (FR-8, FR-15). *Fix:* lead with those.

## Strategic coherence — adequate
The thesis is clear, and the MVP is a problem-solving proof with a retire-the-bridge arc. Tying success to UJ-1 to UJ-5 closes the largest prior gap, because S1 now has a pass/fail test that is traceable to the Vision.

### Findings
- **medium** Success criteria are demo scripts, with no stress test of the wedge and no counter-measure (§ Success Measures: "Numeric metrics are deferred") — UJ-3 proves one new API (TMF622 reads), but the addendum's stronger test ("A fourth API with no Java change", original AC14) is not carried. The only bound is NFR-1 inside UJ-2 and UJ-4. *Fix:* add one line such as "a further API mapped with no code change", or record it as an explicit Later measure.

## Done-ness clarity — adequate (thin on runtime FRs)
The studio, config and export FRs are testable (FR-2 `501` versus `404`, FR-17 atomic import, FR-18 deterministic export, FR-34 delete-block with listing, FR-32 exact message). Runtime and observability FRs are still adjective-led. The new FR-28, FR-19 and FR-39 edits introduce new loose ends.

### Findings
- **medium** FR-28/NFR-6 "never stores" is not reconciled with other retention paths (FR-28; FR-19 access logs, FR-20 "The result of every mapping step is available for a call", FR-21) — the preview sends a pasted sample, which may be real telco data, to the server. Nothing says that preview bodies are excluded from access logs or step results, which FR-19 only redacts and does not omit. FR-14 "test" parity, FR-25 auth and FR-13 for the preview endpoint are not stated. *Fix:* "Preview payloads are not logged or retained; the preview endpoint is part of the FR-14 test surface and requires FR-25 auth."
- **medium** The sensitive-data rule has holes (FR-19, FR-23, FR-30, FR-16) — it covers "every secret value" and "configurable ... field paths", but not the partner's inbound OIDC bearer token or `Authorization` and cookie headers (not provisioned secrets), and the path syntax (`resource.secret.value`) is undefined. Nothing says where the path list is configured or whether it is exported (FR-15). FR-30 says FR-23 applies to the Exchange view but is silent on samples and the verdict inside the share bundle. *Fix:* add credential headers to the rule, name the path syntax and its home, and state that the bundle follows the same rule.
- **medium** Id-map miss versus backend miss are not distinguishable (FR-2 "A configured operation with an unknown id returns `404`"; FR-6; FR-39; UJ-5 "the id is not in that backend") — an unknown TMF id (no id-map entry) and a backend `404` both read as `404`. UJ-3 and UJ-5 never show the id map screen or seeding, so the "add an entry" fix is not part of any journey. *Fix:* say what the bridge returns when the entry is missing, and add the entry step to UJ-5 or UJ-3.
- **medium** FR-39 is thin (FR-39) — "view, add and import" has no import format (per resource? CSV or JSON?), no behaviour for adding a duplicate (clash rule is stated only for import), no edit or delete, and no relationship to FR-15's id-map section. *Fix:* one sentence on format, duplicate rejection and what is not offered.
- **medium** The mock backend is acceptance-bearing but unspecified (FR-27) — UJ-1 needs a catalog attribute to add, UJ-2 and UJ-4 need offerings and orders, and UJ-5 needs `ORD-1` to return `404` even after the id-map transform. FR-27 says only "dedicated mock backend ... ten minutes". *Fix:* list the fixtures the UJs depend on.
- **medium** A mapping has two versions at once (FR-31 "Draft (never applied) or Live"; FR-33 "Editing a Live mapping creates a Draft of the next version"; FR-15 "mappings (Draft and Live, each with its status and version)") — it is unclear whether the stored entity is one record or two, and how export serializes it, so FR-18 determinism cannot be tested. *Fix:* define mapping as an optional live version plus an optional draft version.
- **medium** Import identity and Draft collision (FR-17, FR-18, FR-35) — "A Draft imported over an existing mapping" needs a mapping identity key (unique name?) and a rule for overwriting a colleague's unsaved Draft. *Fix:* state both.
- **medium** Overhead is still undefined and circular (FR-22, NFR-1 "measured as in FR-22", FR-30 tile) — it is not stated whether it includes OpenAPI validation (FR-8) and id-map lookups (FR-6), or excludes backend time. *Fix:* total time minus backend round-trip.
- **medium** Error mapping has no table (FR-4 "map to a TMF Error with the right HTTP status"; FR-8) — no statuses for backend 404, 5xx, timeout, request invalid or response invalid. *Fix:* a short table.
- **medium** FR-25 and FR-26 are loose (FR-25 "Runtime API calls require an OIDC bearer token ... This is configurable"; FR-26 does not name backend auth schemes; FR-34 secret picker versus FR-17 import) — "configurable" is unspecified. An imported backend can name a secret that is not provisioned, and the import check is not stated. *Fix:* say what is configurable, name the S1 backend auth scheme(s), and say whether import validates secret names.
- **medium** Draft test path versus "never served" (FR-33, FR-28(c), FR-25) — sub-step 3 sends a real call through an unserved Draft, which needs a distinct admin execution path. Its auth and logging are unstated. *Fix:* state that test calls go through an admin endpoint under the studio's OIDC context.
- **medium** Autosave and locking (FR-32, FR-33) — the stale check covers Apply only, and autosave can clobber a colleague's Draft, because it is unclear whether a Draft is per mapping or per user. *Fix:* extend the check to autosave or declare last-write-wins.
- **low** FR-20 and FR-21 do not say where step results are available or for whom. *Fix:* "on studio test calls, in the Exchange view and the share bundle".
- **low** FR-7 is not testable ("supports `@type`, `@baseType` and `@schemaLocation`") — it needs one verifiable consequence.
- **low** NFR-2 to NFR-6 name no verification method (which audit, which assistive technology).
- **low** FR-35 is a negative requirement with no rationale.

## Scope honesty — adequate
Improved. S1 now states its backend protocol (FR-34), the Draft and Live model, the "no certification" rule, and the Later groupings with an explicit no-conformance placement. The body still carries zero assumption tags.

### Findings
- **medium** TLS is Later while OIDC tokens and backend credentials are required (Later "Platform: TLS"; FR-24 to FR-26) — S1 should say whether the platform terminates TLS. *Fix:* one line, tagged `[ASSUMPTION]`.
- **medium** TMF versions and the OpenAPI source are unspecified (FR-1, FR-8, FR-10 "official OpenAPI list"; addendum "v4/v5 differences") — nothing says which TMF620 and TMF622 versions are bundled or how the list is maintained. *Fix:* name them and say whether they are bundled.
- **low** No `[ASSUMPTION]` tags or index in a `final` PRD (TLS, per-mapping Draft, backend auth). *Fix:* tag and index.
- **low** Collection reads (`GET /productOrder`) are no longer in any journey but are not named as Later; "query and filter delegation" is the only hint. *Fix:* one line in Out of Scope or Later.

## Downstream usability — adequate
IDs are unique and contiguous (FR-1 to FR-39, NFR-1 to NFR-6, UJ-1 to UJ-5), and every in-PRD reference resolves. Each UJ has a named protagonist. There is still no Glossary, the PRD still embeds UX detail, and the UX spine is out of step with several FRs (see the cross-document section).

### Findings
- **medium** No Glossary, and terms are undefined (whole PRD) — "Guided or Side-by-side mode" (NFR-4) is defined nowhere in the PRD, and "slice", "S1", "mock", "tech people" versus "tech team", "id map" versus "id-map", "resource" and "operation" are used loosely. FR-37 promises a studio glossary, not a PRD one. *Fix:* a short Glossary.
- **medium** The PRD embeds UX design (FR-12 "This replaces the live version", FR-29 pass wording, FR-30 tiles and panels, FR-32 message) — two sources of truth, and they already disagree. *Fix:* keep behaviour and testable consequences in the PRD and leave strings and layout to EXPERIENCE.md.
- **low** UJs cite no FRs, and FR-19 to FR-27 and FR-39 have no journey. *Fix:* add "(FR-..)" tags.

## Shape fit — adequate
UJs with named protagonists suit a multi-role, UX-heavy B2B tool. UJ-2 and UJ-4 are a deliberate before-and-after pair for Tomás. The operator and adopter are still unrepresented.

### Findings
- **low** No journey for the tech colleague who provisions secrets, seeds the id map (FR-39) and eventually retires the bridge (Vision para 2; FR-26; FR-27). *Fix:* a short UJ-6, or an explicit note that these are deployment concerns for architecture.

## Cross-document checks — PRD versus UX spine (EXPERIENCE.md)
Rule applied: FR-30 and FR-34 are authoritative. **[UX]** means the UX spine should change, **[PRD]** means the PRD should change, and **[decide]** means the owner must choose.

Resolved since the prior run: the NFR-3 zoom gap (now 200% and 400% down to 320 CSS px, matching the UX reflow rule); the secret-name catalogue (FR-34 picker, FR-26 listing, matching the UX "No secret names available" state); the legacy upload placement (FR-34 now says it belongs to the backend, so the decision is made; the UX now needs to follow); the demo line `GET /productOrder/{id}`.

### Findings
- **medium** Legacy upload and WSDL in the UX spine conflict with FR-34 (Pick API step table "Optional upload of a legacy OpenAPI or ... (WSDL) file"; Flow 1 item 2; Field list "OpenAPI, WSDL or sample"; State Patterns "Invalid WSDL", "Invalid OpenAPI upload"; Pick API loading state) — FR-34 says the upload belongs to the backend, WSDL and SOAP are Later, and the Backends form has no upload. **[UX]** Move the upload to Declare backend or Backends, and drop WSDL.
- **medium** "Samples never reach the bridge or backend until the analyst runs sub-step 3" (UX Local sample storage; also "Unfired preview" and Sample editor) contradicts FR-28 and NFR-6, where the studio sends the sample to the server to translate it. **[UX]** Say "not stored" and "nothing is sent to the backend".
- **medium** UX has no surface for FR-39 (the IA table has no Id map screen; the Component and State sections have no id-map screen or empty state). The UX spine only knows the id-map transform and warning. **[UX]** Add the surface, or confirm that S1 UX leaves it to the PRD.
- **medium** Share bundle contents differ across three documents (UJ-5 "mapping version, samples, request, response, verdict and secret name"; FR-30 "request, response, mapping version, timestamp, Postman collection"; UX "mapping (deterministic JSON), sample request and response, the Exchange view record (three panels), the verdict and checks, and the secret NAME") — FR-30 omits samples, verdict and secret name, and carries a timestamp that the UX lacks. **[PRD]** Align FR-30 with UJ-5, and decide deliberately on the timestamp.
- **medium** Verdict rules (FR-29 "closed: HTTP status, id-map transform present, empty mapped fields, OpenAPI validation outcome") versus the UX states overhead over budget, unreachable backend, secret not found and the bridge `501` — the last three have no HTTP status the rule list could use. **[PRD]** Add those to the closed list or declare them errors outside the verdict.
- **medium** Apply summary diff (UX Apply summary "changes since the last version"; Flow 2 "what changes versus the live version" versus FR-31 "History, diff and rollback are Later" and FR-12 "summary of what goes live") — S1 has either a small diff or none. **[decide]**
- **low** UX scope note still says "TMF622 `GET /productOrder` read" (collection) where the PRD says `GET /productOrder/{id}`. **[UX]**
- **low** Exchange view wording — FR-30 "latency, which is the FR-22 bridge overhead" versus the UX tile "Exchange latency (ms)"; FR-30 lists four items "in three panels" without naming the panels. **[PRD]** Name the panels.
- **low** FR-9 says the Disclaimer is reachable from studio help, but the UX help footer (Glossary, User manual, "Need a hand?") does not list it. **[UX]**
- **low** UX staleness: the header "FR-10 to FR-13" (now FR-10 to FR-13, FR-28 to FR-35, FR-37 to FR-39); the "Open items for the PRD owner" list (11 notes, mostly applied); Flow 3 labelled "(not in PRD)" though UJ-5 exists; "Not studio flows: UJ-2 and UJ-4" omits UJ-5; the Transform picker note "(FR-5, plus id-map, PRD note 5)" is now inside FR-5. **[UX]**
- **low** UX Flow 2 skips the real call that UJ-1 includes (UJ-1 runs sub-step 3 before Apply). **[UX]**
- Consistent, no action: the closed transform list, Draft and Live only, import as Draft, delete-blocked-with-listing, secret picker and names-only rule, optimistic-locking text, atomic import, the glossary term list (FR-37), static manual, per-mapping export, browser-only samples (storage), list sorted A-Z with no recency (FR-35), NFR-3 reflow.

## Mechanical notes
- **ID continuity.** FR-1 to FR-39, NFR-1 to NFR-6 and UJ-1 to UJ-5 are present, unique and contiguous. FR-37 to FR-39 are appended out of order (F4 reads 10 to 13, 28 to 35, 37; F5 reads 14, 15, 38, 39, 16 to 18), which is acceptable for stable append-only IDs.
- **Cross-references.** All resolve (FR-6 to FR-39, FR-39 to FR-17 and FR-14, NFR-6 to FR-28, FR-9 to FR-29, FR-30 to FR-22, FR-21 and FR-23, FR-34 to FR-26, FR-38 to FR-5, FR-6 and FR-15). Out of Scope's "FR-14 to FR-18 and FR-38" omits FR-39 (id-map import), and the FR-14 range also covers the FR-39 surface, so decide whether FR-39 belongs there. NFR-1 and FR-22 are still circular (see done-ness).
- **Glossary drift.** "Exchange view" and "exchange view" (FR-11 versus FR-30); "id map" versus "id-map"; "v1" (Out of Scope) versus "S1"; "telco integrators" (Vision) versus "integration analyst" (UJ-1); "tech people" (FR-26, UJ-3) versus "tech team" (UJ-5); "mapping version" versus "Version {n}".
- **Assumptions Index.** None, and no inline tags. The addendum holds the assumption trail as a dated log, with some superseded entries ("assumed, confirm", "(NFR-2)" before the split).
- **Sections present.** Vision, User Journeys, FRs, NFRs, Out of Scope, Success Measures, Disclaimer. No Glossary, Open Questions or Assumptions Index.
- **UJ protagonists.** All five named (Inès, Tomás) with context inline.

## Finding counts
- critical: 0
- high: 0
- medium: 25 (including 6 cross-document)
- low: 15 (including 5 cross-document, 4 of them UX-side)
