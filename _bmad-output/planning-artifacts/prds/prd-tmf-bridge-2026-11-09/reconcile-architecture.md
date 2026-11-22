# Reconcile: ARCHITECTURE-SPINE.md vs prd.md + addendum.md

Input: architecture/architecture-tmf-bridge-2026-11-16/ARCHITECTURE-SPINE.md (final 2026-11-20). PRD not edited.

## (a) Amendment check

The spine's PRD Amendments table has 7 rows: FR-5, FR-17, FR-32, FR-39, FR-16, FR-20/FR-21, FR-22. All are faithfully reflected in prd.md. The PRD adds FR-37 and NFR-1 as amendments, and the spine does not list them.

| Ref | Status | Note |
| --- | --- | --- |
| FR-5 | OK | map and filter, nested sub-table. |
| FR-16 | OK, narrower | PRD rejects only credentials "embedded in the URL". The spine (AD-10) also rejects credential-like query parameters. |
| FR-17 | OK | replace or skip, and a Live mapping imports as the next-number Draft. |
| FR-20 | OK | Studio-test only. |
| FR-21 | OK | Studio-test only. |
| FR-22 | OK | Server-Timing header. |
| FR-32 | OK | Autosave also returns a stale-version error. The PRD does not say the studio shows the FR-32 message on a stale autosave (AD-7 does). |
| FR-39 | OK | Reverse-uniqueness clash on add, edit and import. |
| FR-37 | Not in the spine table | PRD FR-37 glossary has "field path (JSON Pointer)", consistent with AD-6. |
| NFR-1 | Not in the spine table | The PRD conditions match AD-12. |

The counts do not match. The spine table has 8 FR refs including FR-21, with no FR-37 and no NFR-1. The addendum says "nine changes" in FR-5, 16, 17, 20, 22, 32, 37, 39 and NFR-1, and omits FR-21. The addendum or the spine table should be aligned.

## (b) Unlisted user-visible behaviour in the spine

1. **Error statuses (FR-4).** The PRD says only "the right HTTP status". The spine fixes:
   - backend 404 returns 404;
   - other backend 4xx returns 502;
   - backend 5xx returns 502;
   - timeout returns 504;
   - transport failure returns 502;
   - invalid translated response returns 502 (the payload is never returned);
   - a runtime mapping error returns 502 (marked [ASSUMPTION, confirm]).

   Also not in the PRD: a global timeout default of 5 s, an optional per-backend timeout, and no retries in S1. None of this is an amendment.
2. **501 and 404 rules (FR-2).**
   - The spine returns 501 when an operation is in a spec but has no Live mapping, so a Draft-only mapping returns 501. The PRD says "unconfigured", which does not make that explicit.
   - A forward id-map miss returns 404. A reverse miss is a mapping error (502). The PRD does not state either.
3. **FR-29 sub-step rules (AD-14).** The PRD defers them to architecture, and AD-14 now defines them:
   - id-map-transform-present and empty-mapped-fields apply to sub-steps a, b and c.
   - OpenAPI validation is request-side in a, response-side in b, and both in c.
   - HTTP status, backend reachable, secret provisioned and overhead apply to c only.
   - Non-applicable rules are skipped, never reported as passed.

   The FR-29 text has not been updated with these rules.
4. **FR-30 bundle contents and FR-15 export scope.**
   - Bundle format: one .zip with manifest.json (timestamp, formatVersion), mapping.json, sample-request.json, sample-response.json, exchange.json, verdict.json and postman.json.
   - The mask token is `***`.
   - The server assembles the bundle from the exchange record the studio posts, and stores nothing.
   - The PRD leaves the layout open, which is consistent. But the bundle has no explicit secret-name file, and the PRD requires "the secret name" in the bundle. Confirm it sits in exchange.json or the mapping.
   - AD-9 says the whole-configuration export carries full ApiSpec text. The PRD FR-15 lists only mappings, backends and id-map entries. This is a silent extension. How specs are handled on import is also unspecified. Import is additive and specs are keyed by hash.
5. **Mapping identity and naming (FR-31, FR-35).** The spine has one Mapping per operation, with natural key (spec hash, method, path template). The mapping name is a label and is unique per deployment. The PRD does not state that names are unique or what happens on an import name collision. Import matches mappings by natural key, not by name.
6. **Mapping-document semantics that users see (FR-5 / FR-7).**
   - A `path` to an absent location in a scalar slot is a mapping error (502), not a null. A `path` to JSON `null` yields null.
   - `op: expr` is rejected at validation.
   - Numbers are BigDecimal.
   - Every mapping document carries `schemaVersion`, and an unknown value is rejected on import.
   - Secret names must match `[A-Za-z0-9_-]{1,64}`, which constrains the FR-26 picker and any imported backend.
   - `authType` (bearer, basic, api-key) is non-secret backend config. FR-34 lists only endpoint, secret name and optional spec.
   - The studio has a mapping-lint endpoint (FR-6 warning).

## Suggested follow-up

- Fold the FR-4 and FR-2 statuses into the PRD, and confirm the 502 assumption.
- Add the FR-29 sub-step rules to FR-29.
- Mention spec text in FR-15.
- Add a mapping-name uniqueness line to FR-35.
- Add `authType` to FR-34.
- Reconcile the amendment count and list between the spine and the addendum.
