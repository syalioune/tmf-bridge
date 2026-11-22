# PRD Quality Review — TMF-Bridge (pass 3, after 2026-11-16 update)

Scope: prd.md and addendum.md. Focus: consistency of FR-16/FR-18/FR-30, FR-29/FR-22/NFR-1, FR-28/NFR-6, FR-34/FR-26, FR-39/FR-17.

## Overall verdict
Good, with fixes needed before architecture. The thesis (config over code, built to be retired) is sharp. Scope honesty is strong: closed lists, explicit Later, Out of Scope. The FR-16/FR-18/FR-30 split is now clean (determinism for configuration exports only, bundle carries a timestamp). No critical findings. The risk sits in seams between the newly extended FRs: backend edits bypass versioning, the share bundle's redaction is incomplete, the overhead metric is undefined, and the id-map import rules leave dead ends. The 2026-11-16 update is not reflected in the frontmatter or the addendum.

## Decision-readiness — adequate
Decisions are stated as decisions, and the addendum records rejected alternatives (per-mapping id-map entries, restore-into-empty, Live-only determinism). Good. There is no Open Questions section; the one real open item (how a legitimate id-map merge or restore of a populated runtime is done) lives only in the addendum. The 2026-11-16 changes (FR-29 verdict rules extended with overhead as info note, FR-30 bundle contents, FR-16 scoping) have no addendum entry, so the rationale is lost.

### Findings
- **low** Update not logged (frontmatter, addendum) — `updated: 2026-11-15` while the PRD changed 2026-11-16. The addendum has no entry for the FR-29, FR-30, FR-16 or Out of Scope range changes. *Fix:* bump `updated`, add a "2026-11-16" addendum block with the decisions (overhead is info-only, bundle holds timestamp, determinism scoped).
- **low** Open item buried (addendum, Export and import update) — The id-map merge/restore gap is stated only in the addendum. *Fix:* add a one-line Open Question or `[NOTE FOR PM]` in F5.

## Substance over theater — strong
Four personas at most, and each drives decisions. NFRs have thresholds (100 ms, WCAG 2.2 AA, 1280 px, 200% and 400% zoom). The dropped throughput NFRs are honestly called out. No innovation theater. One soft spot: NFR-1's percentile and conditions are deferred to architecture, so the one hard performance number is not yet testable (see Done-ness).

### Findings
- none

## Strategic coherence — adequate
The thesis ("zero-code new API", portable exit) holds together, and features follow it (export, import, id map, no certification claim). Success Measures are the UJs only, with numeric metrics deferred and no counter-metric. The wedge test from the addendum (a fourth API with no Java change, original AC14) is not in Success Measures, so the headline claim is never tested by the S1 definition of success.

### Findings
- **medium** Wedge not tested (Success Measures; addendum "Zero-code honesty") — UJ-3 proves one API (TMF622 GET) can be mapped without code, but TMF620 is also "already mapped". Nothing requires an unseen API, so the zero-code claim is not falsifiable. *Fix:* add a Success Measure: a further read operation of a third TMF API is mapped from configuration alone.
- **medium** Source of official OpenAPI definitions unspecified (FR-1, FR-10) — "Pick API ... from the official OpenAPI list": which TMF versions (v4 or v5, which the Vision admits differ), and how definitions are supplied or updated. *Fix:* state the bundled set for S1 or mark the source as an architecture decision.

## Done-ness clarity — adequate (weakest area; most findings here)
Most FRs carry a testable consequence. The newly changed ones have gaps.

### Findings
- **high** Overhead is undefined and circular (NFR-1, FR-22, FR-29, FR-30) — NFR-1 says "measured as in FR-22". FR-22 says "measured and reported" without defining it (total minus backend time? includes validation, lineage, redaction?). On a studio test call, lineage and step results (FR-20, FR-21) add cost absent from normal runtime calls, so the test-call figure is not the NFR-1 figure. NFR-1 also has a percentile left open, while FR-29 compares a single call to the target. FR-30's tile is labelled "latency" but is overhead. *Fix:* define overhead once (FR-22), say whether it excludes lineage cost in the studio, and state that the FR-29 comparison is single call against the 100 ms target, informational only. Rename the tile "bridge overhead" or keep the glossary link.
- **medium** Verdict rules lack pass/fail mapping and sub-step applicability (FR-29, FR-28) — Only backend-reachable and secret failures are said to give a failed verdict, and overhead is info. For HTTP status, empty mapped fields, OpenAPI outcome, id-map missing, the severity is unspecified. Sub-steps (a) and (b) are unfired: reachability, secret, status and overhead are inapplicable, but "every studio test shows" a verdict. The pass text hard-codes "TMF622". *Fix:* a small table: rule, applies in sub-steps a/b/c, pass/warn/fail. Parameterise the API name.
- **medium** "Read-only" not enforceable as written (FR-13, FR-28c, FR-10) — "Test calls never change backend state" is a guarantee the bridge cannot give for arbitrary backends. Pick API does not restrict S1 to GET, so an analyst can map a POST and then reach sub-step (c). *Fix:* state the mechanism (only GET-bound operations enable sub-step c, or the studio blocks it) and soften the guarantee to "the bridge sends only GET".
- **medium** Autosave versus optimistic lock (FR-32, FR-33) — The lock is checked at Apply. Autosave writes Drafts on change; with two editors, a stale autosave overwriting the other's draft is not addressed. "Unsaved edits are kept" does not say what the stored draft holds. *Fix:* say whether autosave carries the entity tag, and what happens on a stale autosave.

## Scope honesty — strong
Out of Scope, Later, closed transform and verdict lists, and mock-only claims are explicit. The Out of Scope FR range (FR-14 to FR-18, FR-38, FR-39) is accurate and resolves. There are no inline `[ASSUMPTION]` tags; the addendum's "assumed, confirm" on no-conformance placement was later confirmed.

### Findings
- **medium** Backend edits bypass versioning and locking (FR-34, FR-31, FR-32, FR-12) — Versioning, Draft/Live and entity-tag locking exist only for mappings. A backend is a stored, editable object, with no Draft state and no stated lock. Editing its endpoint or secret name silently changes the behaviour of every Live mapping that uses it, without an Apply or confirmation. Delete is guarded, edit is not. This also contradicts the spirit of FR-17 ("import never changes the live configuration"). *Fix:* either state that backend edits take effect immediately and show the affected mappings, with entity-tag locking, or mark as a `[NOTE FOR PM]`.
- **low** Legacy OpenAPI upload has no stated S1 purpose (FR-34) — The upload is "optional" and belongs to the backend, but no FR uses it (picker, validation, field list?). *Fix:* name its use or mark it as informational.

## Downstream usability — adequate
IDs FR-1 to FR-39 are all present with no gaps or duplicates (ordered by feature group, not numerically: 28-35 sit under F4, 37 after 35, 14/15/38/39/16-18 under F5, 36 under F7). All cross-references checked resolve: FR-9 to FR-29, FR-17 to FR-33, FR-30 to FR-21/FR-22/FR-23, NFR-6 to FR-28, Out of Scope range. UJs have named protagonists. No PRD-level Glossary exists (the glossary is an S1 deliverable under FR-37). Terms drift: "exchange view" (FR-11) vs "Exchange view" (FR-30), "verdict bar" vs "verdict", "resource" (FR-3, FR-6) vs "mapping" and "operation", "share bundle" vs "bundle".

### Findings
- **high** Share bundle can carry unredacted sensitive data (FR-30, FR-16, FR-23, FR-19, NFR-6) — FR-30 applies FR-23 redaction to the Exchange view, but the bundle also holds the "sample request and response", which come from the browser and are not covered by FR-19/FR-23 (server-side redaction of configured field paths). FR-16 promises "never secret values or other credentials" in every export. A sample pasted from production with a password field, or a backend response with a configured sensitive path, breaks that promise in a file explicitly meant to be forwarded. *Fix:* state that the bundle applies the FR-19 field-path redaction to samples and to all recorded exchange payloads, or warn the analyst at download.
- **medium** Export of a mapping with both Live and Draft versions is ambiguous (FR-15, FR-38, FR-33, FR-18) — Editing a Live mapping creates a Draft of the next version, so one mapping can have Live v1 and Draft v2 at once. FR-15 says "Draft and Live, each with its status and version" and FR-38 "the mapping with its status and version" (singular). Which does a per-mapping export contain, and how does the import (always Draft, "next version") treat two? *Fix:* state that an export carries every stored version of the mapping, or the Draft if it exists.
- **medium** Id-map import and edit rules leave dead ends (FR-39, FR-17, FR-6) — "Clash" is not defined: same TMF id with a different backend id, same pair re-imported, or same backend id mapped from a second TMF id. A re-import of the same file would fail entirely. There is also no edit or delete: a mistyped entry cannot be corrected in S1, since import never overwrites and nothing removes. The studio "add" of a single entry has no stated clash behaviour. *Fix:* define clash (key uniqueness, one-to-one), say whether an identical pair is a no-op, and either add edit/delete or state the workaround.
- **low** No Glossary section — FR-37 lists terms for the user manual, but the PRD itself has none. *Fix:* a short glossary (Draft, Live, verdict, unfired preview, share bundle, Exchange view, id map, secret name) feeds UX and stories.

## Shape fit — strong
Capability-spec plus journeys suits a two-audience tool (analyst in the studio, partner calling the runtime). UJs are not overhead because UX is a load-bearing deliverable. Brownfield and legacy concerns are accurately scoped.

### Findings
- **medium** Sample storage versus server processing (FR-28, NFR-6, FR-20, FR-30) — "The server never stores" a sample, but the server handles it for translation and (sub-step c) for a real call; FR-20 makes step results available, and FR-30's Exchange record and the bundle hold backend payloads. The PRD does not say whether the Exchange record or step results persist server-side, nor that samples are excluded from logs (FR-19 redacts only configured paths). A strict reader could say the real call (c) violates NFR-6. *Fix:* state that samples, step results and the Exchange record are transient (memory-only, not logged, not persisted) and are returned to the browser only. Note the consequence: samples are lost on switching browser, and are absent from FR-15/FR-38 exports.
- **medium** Import of a backend and secret-name provisioning (FR-34, FR-26, FR-17) — The studio picker lists only provisioned names, but import (FR-17) accepts a backend carrying any secret name, including one not provisioned in the target environment. FR-29 catches this at test time, but no FR says an imported or unprovisioned name is allowed, flagged, or shown as such in the backend list. *Fix:* add a clause to FR-34 or FR-17 (allowed, shown as "not provisioned").

## Mechanical notes
- IDs: FR-1 to FR-39 complete, no duplicates, ordered by group rather than number; NFR-1 to NFR-6 contiguous; UJ-1 to UJ-5 contiguous.
- Cross-refs resolved: FR-9 to FR-29, FR-12 to FR-32, FR-16 to FR-18 and FR-30, FR-17 to FR-33 and FR-12, FR-29 to FR-22, FR-26 and NFR-1, FR-34 to FR-26, FR-39 to FR-17 and FR-14, NFR-6 to FR-28.
- FR-16/FR-18/FR-30 are consistent with each other: determinism for whole and per-mapping exports only; the bundle embeds deterministic mapping JSON plus a timestamp; `$SECRET_VALUE` in curl, Postman and bundle. The only gap is redaction of samples (see above).
- FR-29/FR-22/NFR-1: consistent in direction (info note, never blocks) but see the overhead definition finding.
- FR-28/NFR-6: consistent wording ("browser only", "never stores"); transient-data guarantee needs tightening.
- FR-34/FR-26: consistent on names-only picker; edit and import paths not covered.
- FR-39/FR-17: consistent on additive, reject-whole-file; clash undefined.
- Frontmatter `updated` stale (2026-11-15); addendum lacks a 2026-11-16 entry.
- No `[ASSUMPTION]` or `[NOTE FOR PM]` tags anywhere; the Assumptions Index round trip is n/a. Open-items density is low for a final-status PRD, arguably too low given the findings above.
- Case drift: "exchange view" vs "Exchange view"; "resource" vs "mapping".
- Grade by the validate rules: Fair (high findings present, no broken dimension).
