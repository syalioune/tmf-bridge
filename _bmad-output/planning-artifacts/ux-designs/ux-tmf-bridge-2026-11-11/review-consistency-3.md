# Consistency review 3: PRD vs UX (EXPERIENCE.md, DESIGN.md, mockups)

Scope: PRD `prds/prd-tmf-bridge-2026-11-09/prd.md` (updated 2026-11-15) against `EXPERIENCE.md`, `DESIGN.md` and the 8 mocks. Read-only review.

## Verdict

The named conflicts are resolved in the spine text. The remaining problems are stale pointers and mock/spec drift. There are 0 critical, 2 high, 5 medium and 6 low findings. The two highs are a dangling "PRD note n" system and a Flow 3 vs Flow 4 contradiction on the id map.

## A. Resolution check of the earlier conflicts

| Earlier conflict | Status | Evidence |
|---|---|---|
| Legacy upload in Pick API and WSDL vs FR-34 | Resolved in spine and mocks | EXPERIENCE step 1 says "No upload here"; step 2, Flow 1 beat 3, State "Invalid OpenAPI upload" and Backends IA row put upload on the backend. There is no Pick API mock (nothing to be stale). mock-backends shows upload as "belongs to this backend" and "WSDL and SOAP are not supported". See D-1 for the one leftover, `crm-soap`, in the list mock. |
| "Samples never reach the bridge until sub-step 3" vs FR-28 | Resolved | Local sample storage last bullet matches FR-28 and NFR-6. No leftover phrase in EXPERIENCE or DESIGN. |
| No id map surface (FR-39) | Resolved with caveats | IA row, top bar, Flow 3, State Patterns "Id map", mock-id-map. Caveats: see B-2 (edit) and B-3 (ORD-1 conflict). DESIGN top bar omits Id map (C-2). |
| Share bundle contents (FR-30, UJ-5) | Resolved in the spine | Line 282 matches FR-30 item for item. Flow 4 step 6 lists a subset (no Postman collection or timestamp). That is acceptable but not a full list. |
| FR-29 verdict rules vs unreachable, secret, overhead states | Resolved | State rows "Unreachable backend", "Secret name not found" and "Verdict overhead over budget" map to the FR-29 rules. The mock check list ("id mapped with id-map", "status is 2xx", "secret name resolves", "response matches TMF622") fits the closed list. There is no UX state for the "empty mapped fields" rule (low, E-4). |
| Apply summary diff vs FR-31 | Resolved in EXPERIENCE and the mock, not in DESIGN | EXPERIENCE says "Apply summary shows no diff" and the mock shows none. DESIGN line 411 still lists "changes" in the Apply summary definition list (C-3). |
| Disclaimer in help footer (FR-9) | Resolved | Every mock footer carries Glossary, User manual and Disclaimer. See E-6 for the "in the Help panel" claim. |
| Scope note `GET /productOrder` | Resolved | Line 75 uses `GET /productOrder/{id}`. |
| "(not in PRD)" on Flow 3 | Resolved | Flow 3 is now labeled FR-39. |
| FR range header | Resolved | Line 27 reads "FR-10 to FR-13 and FR-28 to FR-39". |
| Open-items list | NOT resolved | See A-1 below. |

## B. Findings

### High

**H-1. The "PRD note n" and "Open items for the PRD owner" system is dangling.**
- Line 27 says every `[NOTE FOR PRD]` is "listed once under Open items for the PRD owner at the end of this file". Line 39 repeats it. The file has no such section. It ends with "PRD reconciliation".
- The "(PRD note n)" pointers are still in lines 60, 77, 116, 121, 200, 210, 261, 274 and 301. Line 385 says they "point to these items", but no numbered items exist.
- Several pointers are also semantically stale because the PRD now owns the point:
  - line 77, "The PRD defines none (PRD note 8)" is now false, since FR-31 defines Draft and Live;
  - line 159, autosave "(the PRD is silent)" is false, since FR-33 covers it;
  - line 210, "Needs a server capability (PRD note 6)" is now FR-32;
  - line 301, "PRD note 9" is now NFR-2.
- Fix: replace each pointer with the real FR or NFR number and delete the open-items language.

**H-2. Flow 3 contradicts Flow 4 and UJ-5 on the id map.**
- Flow 3's climax says the id-map status tile is OK "and the call finds order ORD-1". mock-id-map seeds `productOrder ORD-1 -> 1000234`.
- PRD UJ-5, Flow 4 and mock-exchange-view say ORD-1 still returns 404 after the transform, because "the id is not in that backend". The validation report also notes that FR-27 mock fixtures are undefined and `ORD-1` returns 404.
- The mocks disagree with each other on the backend id. mock-exchange-view says the backend knows the order as `40021877`. mock-id-map says `1000234`.
- Fix: pick a fixture story. For example, the demo id map holds ORD-2 or ORD-3 for the success path, and ORD-1 is deliberately absent for the 404 path. Make Flow 3, Flow 4 and both mocks agree.

### Medium

**M-1. mock-home-mappings contradicts FR-35 and the spine.**
- The rows are not A-Z: Product orders lookup, Create product order, Customer lookup, Trouble ticket status, Product catalog read.
- The headers are plain `<th>`. The spec requires every column header to be a sort button with `aria-sort`, and there is no sort control or `aria-sort` in the mock.
- The empty-state link reads "Import a mapping file" and the toolbar reads "Import mapping...". The spec says "Import a configuration" and "Import". FR-15 import is whole-configuration, so a mapping-only label under-describes it.

**M-2. The Id map entry-clash state offers "edit that entry instead", which FR-39 does not provide.**
- FR-39 covers view, add and import only. The validation report already flagged "no edit or delete".
- EXPERIENCE line 185 says "Try: edit that entry instead". The mock text is safer ("use a different TMF id, or leave it").
- Fix: align the spine to the mock, or have the PRD add edit.

**M-3. mock-home-mappings shows items outside S1 scope and outside FR-34.**
- `Create product order / POST /productOrder` is a write mapping. The scope note and the PRD say writes are Later, and FR-13 allows read tests only.
- `crm-soap` is a SOAP backend, but FR-34 says S1 is REST and JSON only.
- TMF629 and TMF621 rows are outside the TMF620 and TMF622 S1 test beds. This is arguably only a visual illustration.
- Fix: use read operations and REST backends.

**M-4. The "Exchange latency" tile is not defined as FR-22 bridge overhead.**
- FR-30 says the latency tile "is the FR-22 bridge overhead".
- EXPERIENCE calls it "Exchange latency (ms)", and the mock shows 142 ms next to a 100 ms NFR-1 target. The verdict pass copy separately says "overhead in ms (FR-22)".
- A reader cannot tell whether the tile is end-to-end latency or bridge overhead. The 142 ms value also silently exceeds the target while no info note is shown (the verdict is fail in that mock).
- Fix: label the tile "Bridge overhead", or define the distinction.

**M-5. Stale-version copy in the Apply mock suggests the user's edits are lost.**
- FR-32 says "The user's unsaved edits are kept". EXPERIENCE says edits stay in the browser.
- The mock says "open their version, check it, then make your edit again if it is still needed". That reads as lost edits.

### Low

- **L-1. DESIGN.md top bar** (line 395) lists "Backends link, Help" and omits Id map. EXPERIENCE and all mocks have Backends, Id map, Help, dark mode and account.
- **L-2. DESIGN.md mockup list** (line 324) omits mock-id-map, while EXPERIENCE lists 8 mocks.
- **L-3. DESIGN.md Apply summary** (line 411) lists "changes" in the definition list. FR-31 makes diff Later, and EXPERIENCE says no diff. Remove "changes".
- **L-4. EXPERIENCE line 385** says "the PRD was updated to match this spine on 2026-11-16" and "this spine wins on zoom". The PRD is dated 2026-11-15 and already carries zoom in NFR-3, so there is no remaining difference. The frontmatter `updated` is also 2026-11-15. Update the dates and drop the zoom exception.
- **L-5. Fixture inconsistencies across mocks.**
  - The secret name is `OMS_API_KEY` in the exchange and apply mocks, but `OMS_API_TOKEN` in mock-backends for the same `legacy-oms`.
  - The endpoint is `https://oms.internal.example/api` in mock-backends and `legacy-oms.internal/api/orders/lookup` elsewhere.
  - Backend names differ between the list mock (`catalog-db-api`, `crm-soap`, `ticketing-v2`) and mock-backends (`catalog-db`, `billing-sandbox`).
- **L-6. Smaller spine-versus-mock and PRD items.**
  - mock-map-request transform labels "Map values" and "Copy as is" differ from the FR-5 picker names (lookup table, direct copy or rename).
  - EXPERIENCE line 273 says the Disclaimer is "in the Help panel", but mock-help-panel has the link only in the page footer. FR-9 requires "reachable from help", so add it inside the panel.
  - Line 273 says the Disclaimer "opens the PRD disclaimer page". The studio should serve a page of its own.
  - mock-exchange-view panel 2 says "Shown in full in panel 3 below", and the HTTP request block sits under panel 3. The panels are not cleanly split as FR-30 and the spine describe.
  - EXPERIENCE "Import success" says "Imported {n} mappings as Draft", but whole-configuration import (FR-15, FR-17) also adds backends and id-map entries. It is also silent on FR-17's "Draft over an existing mapping becomes next version's Draft".
  - There is no UX state for the FR-29 "empty mapped fields" rule.
  - "Signed in, no access" and "Permission denied" sit uneasily with FR-36 (any authenticated user can do everything). EXPERIENCE frames them as auth-server outcomes, which is acceptable.
  - PRD UJ-3 and UJ-5 still never mention the id map screen, though UX Flow 3 does. This is a PRD-side gap already noted in the validation report.
  - The mocks claim Pick API as step 1 in the stepper, but there is no Pick API or Test sub-step 1 and 2 mock. They are untested surfaces, not contradictions.

## C. FR and NFR sweep (F4, F5, F7, NFR-2..6)

| Requirement | Result |
|---|---|
| FR-10 | OK. Six steps; table is the source of truth; edit lands on Map request. |
| FR-11, 12 | OK. Apply has three states (Ready, stale, live), plus an extra "Apply failed". |
| FR-13 | OK in the spine; M-3 for the mock. |
| FR-28 | OK. Three sub-steps, banner, browser-only samples, server translates and never stores. |
| FR-29 | OK. See the table above and L-6. |
| FR-30 | OK, with M-4 (tile) and the L-6 panel split. |
| FR-31 | OK in EXPERIENCE; stale "PRD defines none" (H-1) and "changes" (L-3). |
| FR-32 | OK in the spine; M-5 in the mock. |
| FR-33 | OK. Autosave, Save now and Draft naming agree. The "PRD is silent" text is stale (H-1). |
| FR-34 | OK. The secret picker is names-only, delete is blocked and lists the mappings, and upload is on the backend. The `crm-soap` row is the only conflict (M-3). The picker itself is not mocked; mock-backends shows only the read-only fields and an "Add a backend" button. |
| FR-35 | Spine OK; mock fails (M-1). |
| FR-37 | OK. The glossary list matches all ten terms. |
| FR-14, 15, 16, 17, 38 | OK. Export all, per-mapping export, atomic import, imports land as Draft, and secret names only all appear. See the L-6 import-success wording. |
| FR-39 | OK except edit (M-2) and the ORD-1 fixture (H-2). |
| FR-24, 25, 26 | OK. OIDC sign-in, secret name only. FR-25 is not a studio concern. |
| FR-36 | OK. Only the auth-server "no access" state touches it. |
| NFR-2 | OK. Keyboard alternative (Connect picker), no color-only signals, and non-drag shortcuts. |
| NFR-3 | OK. 1280 px minimum and 200% and 400% reflow down to 320 px agree across PRD, EXPERIENCE and DESIGN. |
| NFR-4 | OK. Theme follows OS with a manual toggle, remembered per browser; the mode is remembered per browser. |
| NFR-5 | OK. `?` markers link to the manual; the glossary is available. |
| NFR-6 | OK. Samples are browser-only and the server never stores them. |

## D. Navigation and numbering

- The flow numbering is consistent. Flow 1 is UJ-3, Flow 2 is UJ-1, Flow 3 is FR-39 and Flow 4 is UJ-5. Cross-references (Flow 1 failure to Flow 4, State "404 from backend" to Flow 4) resolve.
- Flow 3 is a non-UJ flow placed between UJ flows. Line 323 explains why, so this is fine.
- The top bar order in EXPERIENCE (Backends, Id map, Help, Dark mode, account) matches every mock except DESIGN.md (L-1).
- The IA lists "Backends: Top bar or Mappings list". The Mappings list mock has no Backends entry point other than the top bar, so "or Mappings list" is unmocked.
- The Stitch references in `imports/stitch` are explicitly non-authoritative and were not reviewed here.
