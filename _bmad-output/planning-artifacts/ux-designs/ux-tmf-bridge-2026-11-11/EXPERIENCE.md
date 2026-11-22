---
name: tmf-bridge studio
status: final
created: 2026-11-15
updated: 2026-11-16
sources:
  - ../../prds/prd-tmf-bridge-2026-11-09/prd.md
  - ../../prds/prd-tmf-bridge-2026-11-09/draft-journeys.md
  - ../../prds/prd-tmf-bridge-2026-11-09/addendum.md
  - ./DESIGN.md
  - ./.memlog.md
  - ./.working/extract-prd.md
  - ./wireframes/ia-2026-11-15.excalidraw
  - ./wireframes/flow-map-request-2026-11-15.excalidraw
  - ./mockups/mock-login.html
  - ./mockups/mock-home-mappings.html
  - ./mockups/mock-map-request.html
  - ./mockups/mock-exchange-view.html
  - ./mockups/mock-apply-summary.html
  - ./mockups/mock-backends.html
  - ./mockups/mock-help-panel.html
  - ./mockups/mock-id-map.html
---

# tmf-bridge studio: Experience Spine

> Mapping Studio (PRD F4, FR-10 to FR-13 and FR-28 to FR-39, plus the studio parts of F5). Paired with `DESIGN.md`. Tags: **[S1]** marks slice 1 (tier S1 in the PRD); **[Later]** marks work planned after slice 1. No unconfirmed assumptions remain in this version. Requirements are cited by PRD number (FR-n, NFR-n). Where the PRD and this spine differ, see PRD reconciliation at the end of this file.

## Foundation

Web only, desktop browser. English first. Stakes: consumer posture (calm, forgiving, hand-holding), applied to a professional tool. No brand, no pre-existing look; `DESIGN.md` is the visual identity reference (direction "Calm and guided" with the persistent inspector from "Dense workbench"; graphite surfaces with one deep cyan-blue accent). This spine is the experience. Tokens are cross-referenced as `{path.to.token}` and resolve in `DESIGN.md`.

Primary persona: Inès, integration analyst, telco-savvy, not a developer. Tomás (partner developer) is not a studio user. Feel: easy for telecom-savvy users who are not developers; accessible; always offering help, tips and links to the embedded docs and user manual.

Terminology: technical terms stay (payload, JSON path, transform, lineage, OpenAPI) and ship with a glossary and plain explanations. Mapping model: the table is the source of truth (each row holds a TMF622 field, a transform and a backend field). Drag-and-drop wiring is a visual layer over the table. Light and dark mode follow the system setting, with a manual toggle in the top bar. The default follows the system; once the user toggles, the choice is remembered per browser.

Auth: studio users sign in via OpenID Connect (OIDC) (FR-24). Credentials to backends are never typed in the studio; the studio references a secret NAME provisioned by tech people.

Where this spine differs from the PRD, see PRD reconciliation at the end of this file.

## Responsive and platform

Desktop web only. Minimum supported width 1280px (confirmed), evergreen desktop browsers (Chrome, Firefox, Edge, Safari latest). The layout also reflows (Web Content Accessibility Guidelines (WCAG) 1.4.10): below 1024px, and at 200% and 400% zoom down to 320 CSS px, the 340px inspector and the Help panel stack under the work area as disclosures, side-by-side mode stacks editor above test, and only the mapping table scrolls sideways, inside its own labeled region. The same breakpoints are in `DESIGN.md` Layout & Spacing. No mobile or tablet design and no touch-specific patterns. Samples are per browser (see Local sample storage); dark mode follows the system, with a manual toggle.

## Information Architecture

| Surface | Reached from | Purpose | Slice |
|---|---|---|---|
| Sign in | App open when unauthenticated; session expiry | OIDC sign in; failed or denied sign-in states (see State Patterns) | [S1] |
| Mappings list | After sign in (home) | List of mappings; new, open, export, import | [S1] |
| Backends | Top bar or Mappings list | Backend declarations (endpoint, secret NAME picked from provisioned names, optional legacy OpenAPI upload; REST and JSON only): list, add, edit; delete is blocked while a mapping uses the backend. Declare backend selects from here ([mock](mockups/mock-backends.html)) | [S1] |
| Id map | Top bar | View, add and import id-map entries (TMF id and backend id, per resource) that the id-map transform translates; the same operations exist in REST (FR-39). See [mock](mockups/mock-id-map.html) | [S1] |
| Mapping workspace | New or open mapping | The six steps, mode switch, inspector | [S1] |
| Exchange view | Test step, sub-step 3 | Verdict bar, four summary tiles, three panels, share bar | [S1] |
| Apply summary | Apply step | What goes live, then the "new version is live" confirmation | [S1] |
| Help panel and Glossary | `?` anywhere, top bar | Persistent help, glossary, links to the user manual | [S1] |
| Sign out | Top bar account menu | The only account action; no settings surface in slice 1 | [S1] |
| Mapping history and rollback | Not applicable | Versions, diff, rollback | [Later] |

**The Mappings list is the home surface.** No landing on last-opened, no recency column. Columns: name, TMF API and operation, backend, status. Default order is alphabetical by name; every column header is a sort button (`aria-sort` on the header, the sort state announced), and no sort is by recency. The name is a link that opens the mapping (row click does too); row action: Export (single mapping JSON: the Live version, or the Draft when the mapping was never applied; FR-38). List actions: New mapping, Import, Export all. Import is atomic: an invalid file is rejected and the running configuration stays unchanged (FR-17); the rejection lists each problem with its location. Exports never contain secrets, only secret NAMES (FR-16). Export is deterministic (FR-18). Per-mapping export plus Export all are in the PRD (FR-15, FR-38). Import creates each imported mapping as a Draft, not applied, for review before Apply.

**Mapping workspace**: guided steps by default; editing an existing mapping creates a new version and lands on the Map request step, with earlier steps still editable.

| Step | Name | Content |
|---|---|---|
| 1 | Pick API | TMF API (TMF622 OpenAPI) from a select list, then the operation to map (for example `GET /productOrder/{id}`); this step owns the API and operation choice and feeds the Mappings list column "TMF API and operation". No upload here: the legacy OpenAPI belongs to the backend (Declare backend) |
| 2 | Declare backend | Select an existing backend from the Backends surface (endpoint and secret NAME shown read-only), or use the inline "Add a backend" action (endpoint, secret NAME from a picker, optional legacy OpenAPI upload); the endpoint is never re-typed (WCAG 3.3.7). REST and JSON backends only; WSDL and SOAP are [Later] (FR-34). The step name is exactly "Declare backend" (confirmed) |
| 3 | Map request | Inputs of the operation chosen in Pick API (query, JSON body, header), mapped to backend request fields |
| 4 | Map response | Backend response, mapped to the TMF622 payload |
| 5 | Test | Sub-step 1: request-mapping sample test (unfired). Sub-step 2: response-mapping sample test. Sub-step 3: real read-only backend call, opening the Exchange view |
| 6 | Apply | Summary of what goes live, then confirmation that the new version is live |

Persistent on every step: mode switch (Guided steps and Side-by-side), inspector, top bar (Backends, Id map, Help, Dark mode toggle, account). In Side-by-side mode the matching sample test runs beside each Map step, so Test sub-steps 1 and 2 can also be seen while mapping. Steps can be revisited in any order after the first pass. Warnings never block Next.

Scope note: TMF622 `GET /productOrder/{id}` read and TMF620 productOffering read are the S1 test beds; closed transform list (FR-5) only. Writes, PATCH and cancel are [Later].

**Versioning and status.** Mappings are versioned in the design model and carry a status. Slice 1 statuses are **Draft** (never applied, including freshly imported) and **Live** (an applied version exists); there is no other status in slice 1 (no "Needs attention"). The PRD defines them (FR-31).
- **[S1] Overwrite only:** editing an existing mapping creates a new version, lands on the Map request step (earlier steps editable), and Apply replaces the live version. No history, diff or rollback; none of it is shown in the UI. Concurrent edits are guarded by optimistic locking: Apply blocks on a stale version (State Patterns, Stale version; FR-32).
- The Apply summary shows what goes live; confirmation: "Version {n} is live."
- **[Later]** history, diff, rollback; validate gate; roles; audit trail.
- Data-model rules beyond what the UI shows belong to architecture.

### Visual references

Rendered, illustrative mocks and wireframes; the spines (`DESIGN.md` and this file) win on any conflict with a mock or wireframe, and this is the only place that rule is stated. Google Stitch exports (light and dark, same five surfaces) are kept as raw imports in [imports/stitch/](imports/stitch/): login, home-mappings, map-request, exchange-view, apply-summary, plus its own DESIGN.light.md and DESIGN.dark.md. They are references only; known deviations from these spines, all resolved in the spines' favour: left Workbench sidebar, wrong steppers on Exchange and Apply, red id-map warning, Namespace-integrity banner and info cards, Production-environment label, scenario and verdict demo toggles. Mocks: [mock-login](mockups/mock-login.html), [mock-home-mappings](mockups/mock-home-mappings.html), [mock-map-request](mockups/mock-map-request.html), [mock-exchange-view](mockups/mock-exchange-view.html), [mock-apply-summary](mockups/mock-apply-summary.html), [mock-backends](mockups/mock-backends.html), [mock-help-panel](mockups/mock-help-panel.html), [mock-id-map](mockups/mock-id-map.html). Wireframes: [flow-map-request](wireframes/flow-map-request-2026-11-15.excalidraw) (table rows are placeholders), [ia](wireframes/ia-2026-11-15.excalidraw).

## Voice and Tone

Microcopy. Plain, reassuring, specific: names the object, says what was sent, says what likely went wrong. Brand voice and aesthetic posture live in `DESIGN.md`.

| Do | Don't |
|---|---|
| "The backend couldn't find order ORD-1. Your request sent it as orderId without the id-map transform." | "Error 404" |
| "Request mapped. Nothing was sent to the backend." | "Dry run succeeded" (term reserved for PRD Later) |
| "Uses secret `crm-prod-readonly`. Ask your tech team to change the value." | Any prompt to type a password or token |
| "Version 3 is live. The next call uses it." | "Deployed successfully!" |
| Keep the technical term, explain it once on first use with `?`: "payload (the body of the message)". | Replace the term with a vague synonym, or leave it unexplained |
| Verbs on buttons: "Map request", "Test", "Apply". | "Submit", "OK" |

Never blame the user. No exclamation marks, no celebration copy. Every error follows the error pattern (see State Patterns).

## Component Patterns

Behavioral. Visual specs live in `DESIGN.md` Components.

| Component | Use | Behavioral rules |
|---|---|---|
| Top bar | All authenticated surfaces | Logo and home, Backends, Id map, Help `?`, Dark mode toggle (button, `aria-pressed`), account menu (Sign out only). Same order everywhere. |
| Mappings list table | Mappings list (see [mock](mockups/mock-home-mappings.html)) | The name is a link and opens the mapping; row action Export, named per row ("Export {name}"). Columns: name, TMF API and operation, backend, status. No recency sort. Dense rows. |
| Status badge | Mappings list, workspace header | Mapping status (Draft or Live only in slice 1). Always text and icon, never color alone. |
| Stepper | Mapping workspace (guided mode) | Six numbered steps; the current step is marked (`aria-current="step"`); completed steps are links and reachable; warnings are shown as an icon and a count. Wraps instead of scrolling. |
| Mapping table | Map request, Map response (see [mock](mockups/mock-map-request.html)) | Native table (not an ARIA grid). Each row holds a TMF field, a transform and a backend field, with the field name as a row header holding one row-select button. Selecting a row selects its link in the wiring layer and fills the inspector; the selected row is exposed with `aria-current="true"` and hidden text "selected", and the link is described in text ("id to orderId, no transform, warning") through `aria-describedby`. Rows can be added and deleted; no reordering. Sits in a labeled, focusable horizontal scroll region at narrow widths. Keyboard: see Interaction Primitives, Keyboard. |
| Field list | Both sides of Map steps | Legacy fields (from the backend's OpenAPI or the sample) and TMF attributes; draggable and focusable; shows type; each item has a Connect button. |
| Wiring layer | Map steps | Visual overlay of links; see Interaction Primitives. aria-hidden. |
| Connect picker | Map steps | The keyboard and pointer alternative to drag. A popover with a labeled search field and a listbox (`aria-activedescendant`). It chooses source and target and creates a row. A Connect button is bound to the selected row ("Connect a field to row id"); item-level Connect buttons exist in the Field lists. Esc and focus return: see Keyboard. |
| Transform picker | Mapping table row | Closed list (FR-5, including id-map): direct copy or rename, constant, concatenation, conditional or default value, lookup table, type or format conversion, id-map. Each has an inline tip and `?` link. |
| Id-map warning | Mapping table row | Non-blocking warning when an id field is mapped without the id-map transform. Dashed link, warning icon and a message with an "Add id-map transform" action. |
| Inspector | Right side, every step | Selected-mapping detail (source, transform, target, lineage when available) and contextual help (inline tip, `?` to the manual). Collapsible; state remembered per session. Below the stack breakpoint it is a disclosure under the table. |
| Mode switch | Workspace header | Switches between Guided steps and Side-by-side (editor left, live test and lineage right). Two toggle buttons (`aria-pressed`). Keeps selection, scroll and unsaved edits. |
| Sample editor | Test sub-steps 1 and 2 | A plain multi-line text field (not a code editor), so Tab is never captured; paste or edit sample JSON; Save locally; Run. Invalid JSON is linked with `aria-invalid` and `aria-describedby` to the line and column message; a disabled Run has its reason associated the same way. |
| Verdict bar | Exchange view top (see [mock](mockups/mock-exchange-view.html)) | Plain-language result from rules-based checks (FR-29); icon, heading and text; every check item says Passed, Failed or Skipped in words; links to the failing check. |
| Summary tiles | Exchange view, under the verdict bar | Four read-only tiles in a description list: Exchange latency (ms), Target endpoint (host only), Fields translated (count of backend fields produced, so it never contradicts a failed call), Id-map status (OK, or Missing key with a link). Label and value are programmatically paired. The Id-map link uses the same name everywhere on the page, "Go to the mapping row: id", and moves focus to that row's Transform control in the Map request step. Tiles are hidden before a run. A re-run moves focus to the verdict heading, so tiles need no live region. |
| Exchange panel | Exchange view (three panels) | Mapping result, HTTP request to backend, HTTP response and TMF622 payload, each with a heading. The failing step is auto-expanded; the others are collapsed with a one-line summary. Code blocks are focusable labeled regions (`tabindex="0"`, `aria-labelledby`); masked values carry hidden text "masked". |
| Share bar | Exchange view (sticky) | Copy as curl, Copy as Postman, Download bundle. Copy and download only; no send integration. A labeled region with a skip link. Page scroll padding equals its height; it un-sticks when the viewport is under 600px tall. See Share bundle and redaction. |
| Apply summary | Apply step (see [mock](mockups/mock-apply-summary.html)) | Lists what goes live (mapping name, operation, backend, secret NAME); primary "Apply"; secondary "Back to Test". |
| Help panel | All surfaces | Non-modal complementary region, dockable; context-aware to the selected field, transform or step; search; Glossary tab; manual links; lists the shortcuts. Opened from the top bar `?` or by the Help shortcut (see Keyboard). See [mock](mockups/mock-help-panel.html). |
| Inline tip | Per field and transform | One sentence and `?`. Dismissible per tip; can be restored from the Help panel. |
| Glossary term | Inline in copy | A real button (dotted underline) opening a popover with the definition and a Glossary link on hover and on focus; the pointer can enter the popover; the popover persists until dismissed (Esc, see Keyboard). Never a hover-only title tooltip. See [mock](mockups/mock-help-panel.html). |
| Help link (`?`) | Beside titles, fields, transforms, checks | 24px target; accessible name "Help: {topic}". External manual links add "(opens in a new tab)". |
| Banner | Unfired preview, offline, session ended, stale version, id-map warning, sign-in failure | Inline, never modal; severity by icon and text. |
| Toggle | Top bar, Wiring layer | Dark mode and "Show all links"; button with `aria-pressed`; has a name and state. |
| Skeleton row | Loading lists | Announced once ("Loading mappings"); no shimmer under reduced motion. |
| Toast | Global | Only confirmations such as "Copied."; `role="status"`, visible at least six seconds, paused on hover and focus, dismissible; never the only carrier of essential information. "Saved." is the persistent indicator in the workspace, not a toast. Errors never toast-only; see State Patterns. |
| Confirm dialog | Destructive actions, Import, Apply over live | States what changes, with the named object. Default focus on the safe action. One level, never stacked. |

Skip links on every surface, in this order: Skip to main content, Skip to inspector (workspace), Skip to share bar (Exchange view).

## State Patterns

**Error pattern (fixed, every error).** Four parts, in this order, each present:
1. **What happened**: names the object ("The backend couldn't find order ORD-1.").
2. **Likely meaning**: the best explanation ("Your request sent it as orderId without the id-map transform.").
3. **What to try**: one concrete action, as a button when possible.
4. **Who to ask**: a role and what to hand over ("Your tech team: send them the bundle."), with the Share bundle action when relevant.

Errors carry an icon and text (`{colors.error}`) and are placed inline where the action happened. Announcement rule, one per case: (a) an error that follows a user action which replaces or reloads the view (Run test, Apply, sign-in return, opening a mapping) puts focus on the error heading inside a labeled region (`tabindex="-1"`) and has no `role="alert"`; (b) an error that appears in place without a user action (autosave failure, offline, session ended, stale version noticed in the background) uses `role="alert"` and does not move focus. Never both. No backend internals, stack traces or secret values (FR-4, FR-23).

### Lists and empty states

| State | Surface | Treatment |
|---|---|---|
| Empty list | Mappings list | "No mappings yet." with a primary "New mapping", a secondary "Import a configuration" and a `?` link to the first-mapping guide. |
| Loading list | Mappings list | Five skeleton rows matching the columns; announced "Loading mappings". |
| Loading OpenAPI | Pick API | Skeleton select and "Loading TMF API definitions." |
| Empty mapping table | Map request, Map response | "Nothing mapped yet. Drag a field, or use Connect." No auto-suggestion (no auto-map is in scope). |
| Inspector, nothing selected | Inspector | "Select a row to see its source, transform and target." and an inline tip. |
| Disabled action | Anywhere | Visible, `aria-disabled` (not the `disabled` attribute, so it stays focusable), with the reason as text beside or under it, linked by `aria-describedby` ("Run is off until a sample request is saved."). Never a silent grey-out. |
| Next with warnings | Steps | Allowed; shows the warning count and list. Warnings are non-blocking. |
| Saved | Workspace | Persistent "Saved." indicator (`role="status"`, not a toast) for the autosaved draft. The draft autosaves on change (FR-33), and a "Save now" button sits beside the indicator for reassurance; it saves immediately and the indicator confirms. |
| Mapping fails to open or is gone | Mappings list, workspace | Error pattern (case a): "We couldn't open {name}." or "{name} no longer exists. Someone may have deleted it." Try: back to the Mappings list. Ask: your integration lead. |
| Import rejected | Mappings list | "Import rejected. Nothing changed." and a list of problems with their location; running configuration unchanged (FR-17). Ask: whoever produced the file. |
| Import success | Mappings list | "Imported {n} mappings as Draft. Review them, then apply." Imported mappings are not applied and nothing goes live until Apply; each shows the Draft status. |
| Export failed | Mappings list, Exchange view | Error pattern: "We couldn't create the export." Try: retry; the Copy as curl fallback in the Exchange view. Ask: your tech team. |
| Invalid OpenAPI upload | Declare backend, Backends | What: "We couldn't read {filename} as an OpenAPI file." Likely: the file is not OpenAPI, is incomplete, or references files that were not uploaded. Try: re-upload; a link shows what a valid OpenAPI file looks like. Ask: whoever owns the legacy system. Names the first error and its line. The prior selection is unchanged. WSDL and SOAP files are not accepted in slice 1 ([Later]). |
| Connect type mismatch | Connect picker | A non-blocking note beside the choice: "{source} is a number and {target} expects text." Try: add a type or format conversion (offered as a button). The row can still be created. |

### Backends

| State | Surface | Treatment |
|---|---|---|
| Empty Backends | Backends | "No backends declared yet." and "Declare backend". With no secret names listed, a tip: "Secret names are provisioned by your tech team." See [mock](mockups/mock-backends.html). |
| No secret names available | Declare backend | A disabled picker with the reason in text: "No secret names are provisioned. Ask your tech team to create one." |
| Backend form invalid | Backends (add, edit, inline Add a backend) | Field errors per `field-input` (bad URL: "This doesn't look like a web address. Try https://…"), `aria-invalid`, a summary at the top; Save is `aria-disabled` with the reason; saving shows a loading label. |
| Delete backend blocked | Backends | What: "{backend} can't be deleted. It is used by {n} mappings: {list of mapping names, each a link}." Likely: removing it would break those mappings. Try: open each mapping and pick another backend in Declare backend, then delete. Ask: nobody if you own those mappings, otherwise the owner of the mapping. Delete stays visible, `aria-disabled`, with the reason shown as text. See [mock](mockups/mock-backends.html). |
| Delete backend allowed | Backends | Confirm dialog: "Delete {backend}? No mapping uses it." Default focus on Cancel. Success: "{backend} deleted." Focus returns to the Backends list. See [mock](mockups/mock-backends.html). |
| Unreachable backend | Test sub-step 3 (verdict rule "backend reachable", FR-29) | What: "We couldn't reach {endpoint}." Likely: the backend is down, the URL is wrong, or the bridge can't reach that network. Try: check the endpoint in Declare backend, then retest. Ask: your tech team, with the bundle. |
| Secret name not found | Test sub-step 3 (verdict rule "secret name provisioned", FR-29) | What: "Secret `{name}` isn't provisioned." Try: pick another name. Ask: your tech team to provision it. The value is never shown; the name is. |

### Backend edits

| State | Surface | Treatment |
|---|---|---|
| Backend edited | Backends | A backend is configuration of its own, decoupled from mappings (FR-34): a mapping holds only the backend's name, so editing the endpoint or secret NAME creates no new mapping version and needs no Apply. The edit form shows the note "{n} mappings use this backend. They use the new values on their next call." and lists them as links. Save is the only action; no stale-version block applies to mappings. |

### Id map

| State | Surface | Treatment |
|---|---|---|
| Empty id map | Id map | "No id-map entries yet." with "Add an entry" and "Import entries". A tip: "The id-map transform translates a TMF id to a backend id and back. Without entries it finds nothing." For the demo, infrastructure may seed entries. |
| Entry form invalid | Id map (add, edit) | Field errors per `field-input`: a TMF id, a backend id and a resource are required (marked required in the label); Save is `aria-disabled` with the reason. A clash is checked on blur and announced in a polite `role="status"` line; on Save with an error, focus moves to the first invalid field, whose `aria-describedby` points at the one-line "what happened" sentence. |
| Entry clash | Id map (add) | A clash is the same resource and TMF id with a different backend id (FR-39). What: "TMF id {id} already maps to {backend id} for {resource}." Try: edit that entry from the list, or use a different TMF id. Ask: nobody if you own the mapping, otherwise the entry's owner. Adding an identical entry is accepted and changes nothing. The form never overwrites. |
| Edit entry | Id map | Row action "Edit {TMF id}" opens the same form prefilled; saving changes the backend id (or the TMF id) in place. Runtime state: a Live mapping uses the new value on its next call, no Apply needed. |
| Delete entry | Id map | Row action "Delete {TMF id}"; confirm dialog: "Delete the entry {TMF id} to {backend id} for {resource}? Calls with this id will return 404 until it is added again." Default focus on Cancel. Success: "Entry deleted." Focus returns to the list. |
| Import entries rejected | Id map | Error pattern, case (a): focus moves to the banner heading (`tabindex="-1"`, no `role="alert"`). "Import rejected. Nothing changed." with each problem and its location. A clash (same resource and TMF id, different backend id) rejects the whole file; identical entries are skipped (FR-17, FR-39). Ask: whoever produced the file. |
| Import entries success | Id map | "Imported {n} entries, {m} already present." Imports are additive and never change Live mappings. The message is a polite `role="status"` line. |
| Sortable list | Id map | Every column header is a sort button with `aria-sort`; activation sets one polite status line ("Sorted by TMF id, ascending") and the table caption does not name the sort. Default sort is resource, then TMF id. |
| Whole-configuration export | Mappings list | Id-map entries travel only in Export all (FR-38); a per-mapping export carries the id-map transform but never entries. |

### Test and Exchange

| State | Surface | Treatment |
|---|---|---|
| Live test announcements | Side-by-side | A re-run after an edit is debounced and summarized in one polite `role="status"` line ("Test re-ran: 7 fields mapped, 1 warning"); the result is not read out field by field. Lineage is available as a nested list (value, then each step that produced it) as well as any graphic. |
| Lineage unavailable | Inspector, side-by-side | "Lineage isn't available for this value yet." with the reason (no sample run yet, or an unmapped field); the list form of lineage below still shows what is known. |
| Invalid sample JSON | Sample editor | Inline: the line and column of the problem, linked with `aria-invalid` and `aria-describedby`; Run is `aria-disabled` with the reason. |
| Lost browser sample | Test sub-steps 1 and 2, Exchange view | Samples are browser-only (see Local sample storage). Empty slot: "No saved sample in this browser. Samples are kept per browser and are lost when site data is cleared." with "Paste a sample" and "Import from share bundle". |
| Unfired preview | Test sub-steps 1 and 2 | Banner "Nothing was sent to the backend." persistent while viewing. |
| Test running | Test sub-step 3 (the sample id is prefilled from the saved sample request; never re-typed) | The button becomes "Calling backend…" and disabled; cancel is available; a timeout message appears after the limit (the limit is set in architecture). |
| Verdict pass | Exchange view | "Checks passed. The output matches your sample and the TMF622 schema." (no conformance claim, FR-9), plus overhead in ms (FR-22) and the schema validation result (FR-8). |
| Verdict overhead over budget | Exchange view (verdict rule "overhead against the NFR-1 target", FR-29) | If overhead exceeds the 100 ms target of non-functional requirement NFR-1, an info-level note "Bridge overhead was {n} ms, above the 100 ms target." shows beside the pass; the studio only reports, it does not block. |
| Verdict fail | Exchange view | Error pattern in the verdict bar; the failing panel is auto-expanded; field-level jump to the mapping row. |
| 404 from backend | Exchange view | A verdict fail; the full error pattern content is in Key Flows, Flow 4. |
| 501 on bridge | Exchange view, Apply step | "This operation isn't configured on the bridge yet." Try: Apply the mapping. Ask: nobody (self-serve). |

### Apply and versions

| State | Surface | Treatment |
|---|---|---|
| Stale version (concurrent edit) | Apply step (and on open) | Apply detects that the mapping changed since it was opened and blocks. What: "Someone else changed this mapping. Review their changes before you apply." Likely: a colleague saved a newer version while you were editing, so applying could overwrite it. Try: "Review their changes" (opens the latest version; your edits stay in this browser until you decide).  Ask: your integration lead or the colleague who edited it. Apply stays visible, `aria-disabled`, with the reason as text. FR-32. See [mock](mockups/mock-apply-summary.html). |
| Apply in progress | Apply step | The button becomes "Applying…" and `aria-disabled` (a double submit is ignored); status announced; on finish see Apply success or Apply failed. |
| Apply failed | Apply step | Error pattern; mapping state unchanged; "Back to Test". |
| Apply success | Apply step | Confirmation: "Version {n} is live. The next call uses it." (FR-3, no restart) and a link "Open Mappings list". |

### Auth

| State | Surface | Treatment |
|---|---|---|
| Session expiry | Any | The draft is kept in the browser. Non-modal banner "Your session ended." (`role="alert"`, focus stays; "Sign in again" is the next Tab stop); after sign in, return to the same surface, step and selection. Unsaved work is restored. If restore fails: error pattern. |
| Sign in in progress | Sign in | "Taking you to your company sign-in…" status; the button is `aria-disabled` until the redirect returns. |
| Sign-in failed or denied | Sign in | Error pattern (case a): "Sign-in didn't complete." Try: sign in again; the Sign-in help link. Ask: IT. See [mock](mockups/mock-login.html). |
| Signed in, no access | Sign in | What: "Your account signed in, but it doesn't have access to TMF-Bridge Studio." Try: nothing yourself. Ask: your integration lead or IT, naming the account shown. There are no roles in slice 1, so this is an auth-server outcome. |
| Permission denied | Any | Out of scope (user confirmed); no roles in slice 1 (PRD: roles Later). A server 403 uses the error pattern. |

### Help and system

| State | Surface | Treatment |
|---|---|---|
| Help search, no result | Help panel | "Nothing matched {query}." with "Browse the glossary" and a manual link. |
| Manual unreachable | Help panel, `?` links | "We couldn't open the user manual." Try: retry; the Help panel still shows the built-in tip and glossary. Ask: your tech team if it persists. |
| Offline or network lost | Any | Banner "You're offline. Edits are kept in this browser." No auto-retry loops; a "Retry" button. |
| Focus | All | The `{colors.focus-ring}` ring with its halo, never removed or clipped. Focus return rules: see Keyboard. |
| Dark mode | All | Follows the system; the manual toggle persists per browser. Status colors keep icon and text. |

## Interaction Primitives

**Wiring layer (eight points, confirmed).**
1. No lines at rest.
2. Selecting or focusing a row draws one labeled link (source, transform, target).
3. Hovering a row or field previews its link faintly.
4. Warning links are dashed with an icon (never color alone).
5. Dragging a legacy field onto a TMF622 attribute (or the reverse) creates a table row.
6. The Connect picker is the keyboard alternative; it is also available by pointer.
7. A "Show all links" toggle, off by default; when more than eight links exist, show a count instead of drawing them.
8. The layer is `aria-hidden`; the table is the truth and announces the link in text.

Never a mesh of arrows between two tables. Drag directions: for a request, from TMF622 inputs to backend request fields; for a response, from backend response fields to the TMF622 payload. Legacy fields and TMF622 attributes can be dragged either way.

**Guided and side-by-side.** The switch is in the workspace header. Guided: one step at a time with the stepper, Back and Next. Side-by-side: editor (current Map step) left, live test and lineage right; the test re-runs on edit against the saved sample. The switch preserves selection, scroll position and unsaved edits; no confirmation is needed. The mode choice is remembered per browser.

**Inspector.** Persistent right panel on every step. Content follows selection: mapping detail (source, transform, target, lineage, warnings) and contextual help (inline tip, `?` to the matching user-manual section). Keyboard: see Keyboard.

**Contextual help.** An inline tip per field and transform; a `?` link beside every step title, field and transform opens the matching Help panel section; a persistent Help panel with search; the Glossary tab covers payload, JSON path, transform, lineage and OpenAPI (and more, see Help and glossary system).

**Keyboard (the single rule set; other sections point here).**
- Every drag action has a keyboard path via the mapping table and the Connect picker. Nothing traps focus: the sample editor is a plain text field.
- The mapping table is fully operable by keyboard: Tab moves through each row's controls (row-select button, Edit); `Enter` or `Space` activates. Tab order matches reading order.
- `Esc` closes the topmost panel, popover, picker or glossary popover and returns focus to its opener. After a dialog closes, focus returns to the trigger. Inside the inspector, `Esc` returns focus to the table.
- Shortcuts always need a modifier and never fire on a bare character (WCAG 2.1.4). `Alt+C` opens the Connect picker for the selected row. `Alt+H` opens the Help panel at the current focus. These key combinations still need a browser, OS and screen-reader conflict check during build; the build may change the keys, and the Help panel always lists the current ones. They work inside inputs because they are modified, and the Help panel lists them.

**Id mapping.** Mapping an id field without the id-map transform shows the non-blocking warning; it never blocks Next or Apply. The id-map transform is a real slice-1 transform (FR-5).

**Banned:** drag-only actions; single-character shortcuts; color-only signals; auto-fired backend calls (every real backend call needs an explicit button press); typing secrets anywhere; send integrations (email, chat) for sharing; modal stacks deeper than one level; hover-only help; auto-dismissing errors; celebratory animation.

## Help and glossary system

*Invented section: a cross-cutting requirement from the design decision log ("contextual everywhere"). Component behavior lives in Component Patterns; these are the rules that cut across it.*

- **Coverage:** every field, transform and step has an inline tip (one sentence, shown under the control, dismissible, restorable with "Restore tips" in the Help panel). Every step title, field, transform and verdict check has a `?` link.
- **Manual links:** links to the external manual open in a new tab with "(opens in a new tab)" in the accessible name.
- **Consistent help (WCAG 3.2.6):** a help block (Glossary, User manual, Disclaimer, "Need a hand? Ask your integration lead.") sits in the footer of every surface, in the same order, including the Exchange view (below the share bar) and Sign in.
- **Glossary terms:** payload, JSON path, transform, lineage, OpenAPI; plus id map, secret NAME, verdict, unfired preview and mapping version. Each entry has a plain definition, an example and a manual link. Terms in the UI are the Glossary term component.
- **Disclaimer:** the Disclaimer link opens the PRD disclaimer page (independent open-source project, not affiliated with or certified by TM Forum; FR-9). It is in every footer and in the Help panel.
- **Manual:** the user manual ships with the studio, as static docs served by it, so every `?` and Read link resolves inside the product. Writing and bundling it is a slice-1 deliverable (FR-37); anchors match the `?` links used in the mocks.

## Share bundle and redaction

*Invented section: derived from the design decision log (Exchange view share bar; redaction).*

- **Share bar** (sticky in the Exchange view): Copy as curl, Copy as Postman, Download bundle. Copy and download only. Forwarding to tech means the analyst sends the artifact.
- **curl and Postman** replay the HTTP request to the backend as shown in panel 2. Postman is a collection JSON copied to the clipboard and also included in the downloaded bundle.
- **Bundle** (download, FR-30): the mapping version (deterministic JSON), sample request and response, the Exchange view record (backend request, backend response, TMF payload), the verdict and checks, the secret NAME, the Postman collection and a timestamp. Sensitive field paths (FR-19) are masked in the samples too, so the bundle never carries more than the Exchange view shows. The exact file layout is an architecture hand-off. Importable as samples (see Local sample storage).
- **Redaction:** the secret value is never shown or exported; the secret NAME is shown. Sensitive data in step results, lineage and shared artifacts is masked per the same rule as access logs (FR-19, FR-23). Masked values display as `••••` with the text "masked" (hidden text for screen readers). A note under the share bar: "Secrets are never included. Only the secret name is. Sensitive fields are masked in samples too."
- curl and Postman use a placeholder for the credential (`$SECRET_VALUE`).
- Export failure: see State Patterns.

## Local sample storage

*Invented section: derived from the design decision log.*

- Sample request and sample response are saved in the browser (per browser, not shared, not synced); they survive refresh and sign-out, but are lost when site data is cleared.
- Re-runnable after edits to the mapping, in both modes.
- Surfaces carry a standing note: "Samples are saved in this browser only." with a link to Share bundle as the backup and hand-off route.
- Lost-sample state: see State Patterns. Download bundle and "Import from share bundle" restore them.
- The studio sends a sample to the bridge server only to translate it in sub-steps 1 and 2; the server never stores it (FR-28, NFR-6). Nothing reaches the backend until the analyst runs sub-step 3.

## Accessibility Floor

Behavioral. Visual contrast lives in `DESIGN.md`. Each line is a checklist rule; where another section owns the detail, it is named.

- WCAG 2.2 AA across all surfaces (NFR-2); contrast and dark mode meet the same ratios (`DESIGN.md` Colors).
- Full keyboard operation, drag alternatives, no traps, modified shortcuts: Interaction Primitives, Keyboard.
- Screen readers: links between fields are announced in text from the table (Interaction Primitives, Wiring layer). A step change moves focus to the surface `h1` (`tabindex="-1"`, text "Step {n} of 6: {name}") and updates the document title ("{Step} (step {n} of 6) - {mapping} - TMF-Bridge Studio"), with no separate live region. Errors follow the announcement rule (State Patterns). Pass and saved use `role="status"`.
- No color-only signals: statuses, warnings, links and masked values carry icon and text or a pattern such as dashed links (Component Patterns, Status badge and Id-map warning).
- Focus ring from `{colors.focus-ring}`, always visible; the input border uses the dedicated 3:1 token `{colors.input-border}` (`DESIGN.md` Colors).
- Focus handling: after a step change focus lands on the step `h1`; after a dialog closes, on the trigger (Keyboard); after an action-driven error, on the error heading (State Patterns, announcement rule).
- Reduced motion respected: no transitions on step change, panel expand, save indicator or skeleton shimmer. Forced-colors mode: selection, pills and banners carry borders or outlines, not background alone.
- Dense tables keep a readable row height and header association (`<th scope>`, row headers): Component Patterns, Mapping table.
- The sticky share bar must not cover focused elements (page `scroll-padding-bottom` equals its height; it un-sticks under 600px viewport height): Component Patterns, Share bar.
- One `h1` per surface; panels are `h2`. Targets are 24px or more; essential text 14px or more (`DESIGN.md` Typography and Layout & Spacing). Every button name is unique in context.
- Skip links: main, inspector, share bar (Component Patterns).

## Inspiration and anti-patterns

- **Rejected: mesh of arrows between two tables.** Wiring is unreadable and inaccessible; links only for the selected field.
- **Rejected: editorial and playful visual directions.** They felt wrong for a professional tool. The register is professional, calm, approachable.
- **Rejected: the analyst's external mapping import.** No import of Word, Markdown or Excel mapping docs; she re-implements in the studio, and the studio does not mirror her formalism.
- **Rejected: send-to-tech integration.** Copy and download only.
- **Lifted: the persistent inspector** from the "Dense workbench" direction.

## Key Flows

Flows are labeled with the PRD's user journeys (UJ-n). The id map is seeded in the Id map surface before Flow 1's mapping is tested (see Flow 3).

### Flow 1: Inès maps TMF622 (UJ-3)

Inès, integration analyst, has her mapping doc (Word or Excel) done elsewhere and wants TMF622 `GET /productOrder/{id}` live. The flow follows the six-step model; Test is one step with three sub-steps. The list numbers below count the flow's beats, not the workspace steps.

1. Inès signs in and lands on the Mappings list (home). She selects "New mapping".
2. **Pick API:** she picks the TMF622 OpenAPI from the select list and chooses the operation `GET /productOrder/{id}`; there is no upload in this step.
3. **Declare backend:** she selects the existing `legacy-oms` backend (endpoint and secret NAME shown); if it is missing she uses "Add a backend" with its endpoint, a secret NAME chosen from the picker, and an optional legacy OpenAPI upload. She never types a secret or re-types an endpoint.
4. **Map request:** she sees the operation's inputs (query, JSON body, header), then drags legacy fields onto TMF622 inputs (or uses Connect). Table rows appear; a link is drawn for the selected row only. She maps the id field; the non-blocking id-map warning shows and she adds the id-map transform.
5. **Map response:** she maps the backend response fields to the TMF622 payload, the same way, by dragging or using Connect.
6. **Test, sub-step 1:** she pastes a sample request; the studio shows the translated backend request, unfired ("Nothing was sent to the backend."). In Side-by-side mode she could already see this while mapping.
7. **Test, sub-step 2:** she pastes a sample response and sees the TMF622 payload it produces.
8. **Climax:** the unfired preview matches her sample, field for field. This is how she knows the mapping is about 99% right before touching the real backend.
9. **Test, sub-step 3:** she runs a read-only call to the backend as if called through the REST API (the sample id is prefilled). The Exchange view opens: the verdict bar passes ("Checks passed. The output matches your sample and the TMF622 schema."); the panels show the mapping result, the HTTP request to the backend, and the HTTP response and TMF622 payload.
10. **Apply:** the summary lists what goes live; she confirms; "Version 1 is live. The next call uses it."

Failure (item numbers refer to the list above): backend unreachable or secret missing at item 9 gives the error pattern, and the work from items 1 to 8 is retained; a 404 at item 9 leads to Flow 4; an invalid OpenAPI upload at item 3 gives the error pattern with the prior selection kept; missing samples give the lost-sample state; a stale version at item 10 gives the Stale version state.

### Flow 2: Inès adds a field to an existing mapping (UJ-1)

Partners asked Inès to add one more attribute to the TMF620 offering.

1. Inès opens the Mappings list and opens the TMF620 mapping.
2. Editing creates a new version and the workspace lands directly on the Map request step. Earlier steps remain editable via the stepper; Map response is one step away.
3. She drags the backend field onto the TMF attribute, or uses Connect; a row is created and selected; the inspector shows source, transform, target and the matching tip.
4. She switches to **Side-by-side**: editor left, live test and lineage right; the saved sample re-runs.
5. The right side shows the new value with its field lineage and schema validation result (FR-11, FR-21, FR-8).
6. She goes to **Apply**; the summary shows what goes live (mapping name, operation, backend, secret NAME) and "This replaces the live version."
7. **Climax:** she confirms and sees "Version 2 is live. The next call uses it.", with no restart and no developer.

Failure: a lost saved sample gives the lost-sample state; failed validation gives the verdict bar fail with a jump to the mapping row; a failed apply gives the error pattern with the version unchanged. Slice 1 is overwrite only: there is no rollback, so the summary says "This replaces the live version."

### Flow 3: Inès seeds the id map (FR-39)

Before testing, the id-map transform needs data.

1. Inès opens **Id map** from the top bar. She picks the resource (for example `productOrder`).
2. The list shows TMF id, backend id and resource, sortable like other lists. She selects "Add an entry", types a TMF id and its backend id, and saves. A clash with an existing entry is shown as an error, never overwritten.
3. For many entries she selects "Import entries" and chooses a file. A file with any clash or invalid row is rejected whole, with each problem and its location.
4. **Climax:** she returns to Test, sub-step 3. The id-map status tile says OK for order ORD-2, which the demo backend knows. ORD-1 is deliberately not in the demo backend, which is why Flow 4 still returns 404.

Failure: rejected import and entry clash: see State Patterns, Id map. For the demo, infrastructure may seed the entries, and the screen then shows them.

### Flow 4: 404 forwarded to tech (UJ-5)

Inès, in the Exchange view for an order id she picked from her docs.

1. In Test sub-step 3 she runs the real read-only call for ORD-1 (taken from her saved sample request, not re-typed).
2. The Exchange view opens; the verdict bar fails and the HTTP response panel is auto-expanded, showing 404.
3. The bar reads: "The backend couldn't find order ORD-1. Your request sent it as orderId without the id-map transform. Try adding the id-map transform. If it still fails, ask your tech team." Error pattern parts: what happened (the backend found nothing for the given id); likely meaning (the id was sent without the id-map transform, or it is the wrong record); what to try (jump to the mapping row, add the id-map transform, retest); who to ask (your tech team, with the bundle).
4. She uses the field-level jump to the mapping row, adds the id-map transform, and re-runs. It still returns 404: the id is not in that backend.
5. She uses the sticky share bar: Download bundle (and Copy as curl).
6. **Climax:** she sends her tech colleague the bundle; it holds the mapping version, samples, the three panels, the verdict and checks, and the secret NAME, but no secret value, so they can replay it in curl or Postman without asking her anything.
7. The tech person fixes the backend or provisions access; Inès re-opens the mapping and re-runs from the Exchange view.

Failure: a bundle download blocked by the browser gives the error pattern with a "Copy as curl" fallback.

Not studio flows: UJ-2 and UJ-4 (Tomás, a northbound API caller, who does not use the studio) are covered by the bridge's REST behavior, not by this spine.

## PRD reconciliation

The PRD was updated to match this spine on 2026-11-16, and the UX was updated to match the PRD. Where they still differ, the PRD wins. Resolved: verdict rules (FR-29), real backend versus mock and unfired preview versus dry-run (FR-28), export and import in the studio (FR-14 to FR-18, FR-38), id-map transform (FR-5, FR-6, FR-39), optimistic locking (FR-32), disclaimer (FR-9), status and version (FR-31), accessibility, platform, themes and help (NFR-2 to NFR-6), user manual (FR-37), draft autosave (FR-33). Apply summary shows no diff, because history and diff are [Later] (FR-31).
