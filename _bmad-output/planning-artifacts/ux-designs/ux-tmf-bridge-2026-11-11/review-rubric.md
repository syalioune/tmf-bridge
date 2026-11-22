# Spine Pair Review — TMF-Bridge Studio

## Overall verdict
The pair is a strong, decision-dense contract. Every colour token has a hex and a light/dark pair, every token reference resolves, section order and required defaults are correct, and the key flows carry named protagonist, climax and failure paths. It is not yet safe to hand downstream. The id-map transform contradicts the PRD's closed transform list. The responsive rules conflict between the two files. DESIGN.md specifies only about half the components EXPERIENCE.md names, under different names. The five rendered mocks and the wireframes are not linked from either spine. Severity counts: critical 0, high 6, medium 11, low 12.

Scope notes: Pass 1 and Pass 2 done on DESIGN.md, EXPERIENCE.md, .memlog.md, `.working/` mocks and the PRD. Spines untouched. No `mockups/`, `wireframes/` or `imports/` folders exist; visual artifacts sit in `.working/`.

## 1. Flow coverage — adequate
Sources frontmatter and PRD journeys: UJ-1 (Flow 2) and UJ-3 (Flow 1) are the studio journeys, both with Inès, numbered steps, a climax and a failure path. UJ-2 and UJ-4 (Tomás, northbound API caller) get no flow. Flow 3 is a UX-originated flow, tagged "not in PRD".
### Findings
- **[high]** Flow 1 interleaves Test sub-step 1 (step 5) between Map request and Map response (step 6), but the IA table makes Test step 5, after Map response step 4, with sub-steps 1 and 2 inside it. A consumer cannot tell whether Test is one step or split across the mapping steps. (EXPERIENCE.md > Key Flows Flow 1 steps 4-7 vs Information Architecture step table). *Fix:* align Flow 1 to the 6-step model, or state that sub-steps 1 and 2 are reachable from the Map steps.
- **[medium]** Where the operation (`GET /productOrder`) is chosen is ambiguous. IA puts "TMF622 OpenAPI" in Pick API and "TMF622 inputs" in Map request, with no step owning operation selection. Flow 1 step 4 selects it inside Map request. The Mappings list column "TMF API + operation" depends on it. (EXPERIENCE.md > IA step table, Flow 1 step 4). *Fix:* name the step that binds the operation and add it to the step table.
- **[low]** Flow 1 failure path omits the 404 and id-map case, which is covered only in Flow 3. (Flow 1 "Failure:" line). *Fix:* cross-reference Flow 3.
- **[low]** UJ-2 and UJ-4 are neither covered nor stated as out of the studio's scope in the sources or flows. Foundation only says Tomás is not a studio user. (EXPERIENCE.md > Foundation). *Fix:* one line in Key Flows: UJ-2 and UJ-4 are not studio flows.

## 2. Token completeness — adequate
All 21 light colour tokens have hex and a matching `-dark` hex. Typography, rounded and spacing are defined. All `{...}` references in DESIGN.md components resolve. EXPERIENCE.md refs (`{colors.focus-ring}`, `{colors.error}`, `{colors.input-border}`) resolve. Contrast is stated only for focus-ring and border.
### Findings
- **[medium]** `status-badge` defines no per-status colour mapping. The spine says Draft, Live, Error and Warning use tints, but not which token pair Draft and Live take. They are the only two S1 statuses. (DESIGN.md > components.status-badge, Components > Status badge). *Fix:* add `draft-*` and `live-*` token pairs, or state the mapping.
- **[medium]** No error, invalid, disabled or hover/active state tokens for `field-input` and buttons. EXPERIENCE requires inline invalid-JSON errors and `aria-disabled` actions with visible reasons. (DESIGN.md > components.field-input, button-primary, button-secondary). *Fix:* add `error-border` and `disabled-*` tokens, or state the rule in Components.
- **[medium]** Contrast targets for load-bearing pairs are missing. Only focus-ring and border have figures. Primary on surface (link text), status foreground on `-bg`, `on-primary` on primary, and muted text in dark are claimed "AA" without ratios. The memlog notes the computed values existed in the theme render. (DESIGN.md > Colors). *Fix:* add a contrast table of the 8-10 key pairs for light and dark.
- **[medium]** `help-link` is 20px, below the WCAG 2.2 AA 24px target minimum (2.5.8). `label` is 13px, yet Typography says "Never set essential text below 14px" and column headers are essential. (DESIGN.md > components.help-link, typography.label, Typography). *Fix:* 24px hit area, and set the label to 14px or restate the floor.
- **[low]** `gutter: 18px` is off the stated 4/8/12/16/24/32 scale. Raw px values (20px, 24px, 2px, 3px) bypass tokens. (DESIGN.md > spacing, components). *Fix:* snap 18 to 16 or 24, and tokenize widths.
- **[low]** Focus ring uses the same hex as primary. The memlog flags this as unconfirmed by the user and needing an offset on filled buttons. The spine handles it with the 2px gap. (DESIGN.md > Colors Focus-ring; .memlog.md last entry). *Fix:* record the confirmation, or add a ring-on-primary-fill rule.

## 3. Component coverage — thin
EXPERIENCE.md names about 22 components. DESIGN.md specifies 13 in its Components section and frontmatter. Several are the same thing under a different name.
### Findings
- **[high]** Name drift and missing visual specs. Name drift: Step rail vs Stepper, Mapping table vs mapping-table-row, Inline tip vs Help tip, Status chip vs Status badge, Wiring layer vs Wiring link. No DESIGN spec at all: Top bar, Field list, Connect picker, Transform picker, Mode switch, Sample editor, Exchange panel (x3), Apply summary, Help panel (with tabs), Toast, Confirm dialog, banners (session, offline, unfired), skeleton rows, the "Show all links" and dark-mode toggles. (EXPERIENCE.md > Component Patterns; DESIGN.md > Components). *Fix:* one canonical name per component across both files, and a DESIGN row for each behavioural component.
- **[high]** The Transform picker lists the six FR-5 closed transforms, but the id-map warning, Flow 1 step 4, Flow 3 and Id mapping all require an "id-map transform", which is not in that list. The PRD's closed list excludes it, and FR-6 treats the id map as runtime state. This is a capability gap that architecture must resolve. (EXPERIENCE.md > Component Patterns Transform picker, Id-map warning; Interaction Primitives > Id mapping). *Fix:* add a `[NOTE FOR PRD]`, or define the id-map transform as a seventh item.
- **[low]** "Mapping table" (workspace) and "Mappings table" (list) are near-identical names for different components. (EXPERIENCE.md > Component Patterns). *Fix:* rename the list one to "Mappings list table".
- **[low]** The mapping-table-row columns are hard-coded to "TMF622 field", but Flow 2 maps TMF620. (DESIGN.md > Components Mapping table row). *Fix:* "TMF field".
- **[low]** `rounded.sm` is reserved for keyboard-key and inline code chips, which have no component spec. (DESIGN.md > Shapes). *Fix:* add a `code-chip` entry or drop the claim.

## 4. State coverage — adequate
State Patterns is thorough on errors, empties, loading and the 404/501/unreachable/WSDL cases. Gaps are on surfaces with less attention.
### Findings
- **[high]** No concurrent-edit or stale-version state. Studio users sign in via OIDC and so are several. Slice 1 is overwrite-only and Apply "replaces the live version". Two analysts can silently clobber each other. (EXPERIENCE.md > Mapping versioning, State Patterns). *Fix:* a "changed since you opened it" state, or an explicit `[NOTE FOR PRD]` accepting last-write-wins.
- **[medium]** The Sign in surface has no states: OIDC redirect, failed or denied login, account with no access. Only session expiry is covered, and no sign-in flow exists. (EXPERIENCE.md > IA Sign in; State Patterns). *Fix:* add Sign in rows.
- **[medium]** Other missing states: Backends create/edit validation (bad URL) and loading; Inspector with nothing selected; Help panel search with no result or manual link unreachable (targets undefined); opening a mapping that fails or no longer exists; Apply in-progress (double submit); export failure; Connect with a type mismatch; lineage unavailable. (EXPERIENCE.md > State Patterns, Component Patterns). *Fix:* add one row each; most reuse the error pattern.
- **[medium]** The single-key shortcuts `C` and `?` are global-feeling character-key shortcuts (WCAG 2.1.4). `?` also collides with typing in text fields and with Esc handling in the inspector. (EXPERIENCE.md > Interaction Primitives > Keyboard). *Fix:* scope to table focus only, or require a modifier, and document the turn-off.
- **[low]** The verdict pass shows overhead ms, but there is no state for exceeding the NFR-1 100 ms budget. (EXPERIENCE.md > State Patterns Verdict pass). *Fix:* a warning-level note, or state that the studio only reports.

## 5. Visual reference coverage — broken
Files in `.working/`: 5 mocks (`mock-map-request`, `mock-exchange-view`, `mock-apply-summary`, `mock-home-mappings`, `mock-login`), `ia-2026-11-15.excalidraw`, `flow-map-request-2026-11-15.excalidraw`, `directions-1.html`, `color-themes-1.html`, `extract-prd.md`.
### Findings
- **[high]** The five key-screen mocks are linked from neither spine. EXPERIENCE says they are "planned; links to be added when `mockups/` exists", but the memlog records them as rendered. No `mockups/` or `wireframes/` folder exists. The two excalidraw files are referenced once, in prose under IA, with the caveat that table rows are placeholders. DESIGN.md has no visual references at all. "Spine wins on conflict" appears once, in EXPERIENCE only. (EXPERIENCE.md > IA composition reference; DESIGN.md > none). *Fix:* move or symlink the mocks into `mockups/`, link each at its section (Map request, Exchange view, Apply summary, Mappings list, Sign in), name what each illustrates, and verify them in a browser (the memlog says unverified). Mock text was not verified against current spine copy.
- **[low]** `directions-1.html` and `color-themes-1.html` are not marked superseded. The memlog says the theme render carried a magenta focus ring that the spine later overrode. (.memlog.md; .working/). *Fix:* note them as exploration only.

## 6. Bloat & overspecification — adequate
Verdict: adequate. DESIGN.md is lean. EXPERIENCE.md at 28.7K is the dense one, justified by the user's "take care of tiny details" request, but restates things.
### Findings
- **[medium]** Repeated content. The wiring-layer rules appear in Component Patterns (Wiring layer), Interaction Primitives (8 points), Key Flows, DESIGN Components (Wiring link) and Do's and Don'ts (mesh). Contextual help is described in Foundation, Component Patterns (Inline tip, Help panel), Interaction Primitives and the Help & Glossary system section. Drift risk. (EXPERIENCE.md, multiple). *Fix:* keep one canonical home, with others pointing to it.
- **[low]** DESIGN Colors prose restates hex values already in frontmatter. They can drift. (DESIGN.md > Colors). *Fix:* reference by token name.

## 7. Inheritance discipline — thin
UJ-1 and UJ-3 names match the PRD verbatim. FR numbers cited in the spines match the PRD (FR-3, 4, 5, 8, 11, 16-19, 21-24). Glossary and component naming is inconsistent (see 3).
### Findings
- **[high]** The responsive rules conflict. DESIGN Layout says the page "must remain usable at 200% zoom", that the inspector stacks under the work area below about 900px, and that breakpoints are unspecified. EXPERIENCE Responsive says minimum width 1280px, no responsive reflow, and the inspector collapses to icons. At 200% zoom a 1280px viewport is 640 CSS px, below the stated minimum and in conflict with WCAG reflow (1.4.10). (DESIGN.md > Typography, Layout & Spacing; EXPERIENCE.md > Responsive & Platform, Accessibility Floor). *Fix:* decide one rule, and state the zoom behaviour explicitly.
- **[medium]** FR-9 says no conformance claim. The verdict copy "Looks right. Response matches TMF622 and your mapping." and the "OpenAPI validation pass" read as a conformance judgement. The spines never mention FR-9 or the disclaimer. (EXPERIENCE.md > State Patterns Verdict pass). *Fix:* reword to "passes schema validation", and add an FR-9 note.
- **[medium]** `sources` frontmatter. EXPERIENCE uses unresolved `{planning_artifacts}` placeholders, while DESIGN uses relative paths. It lists working files (`.memlog.md`, excalidraw) but not the mocks. Neither lists `extract-prd.md` consistently. (EXPERIENCE.md > frontmatter). *Fix:* use relative paths, add the mocks.
- **[low]** `[NOTE FOR PRD]` handling. Foundation says "Three conflicts" then lists four, one separately tagged. The memlog says "PRD notes x5". More notes sit in Mapping versioning and Accessibility Floor. (EXPERIENCE.md > Foundation). *Fix:* correct the count and consolidate into one list.

## 8. Shape fit — strong
DESIGN.md sections follow canonical order (Brand & Style, Colors, Typography, Layout & Spacing, Elevation & Depth, Shapes, Components, Do's and Don'ts). EXPERIENCE.md has all eight required defaults, plus Inspiration (triggered by memlog rejects) and Responsive (triggered by min width). Invented sections (Help & Glossary, Share bundle & redaction, Local sample storage, Mapping versioning) each trace to the memlog and mostly earn their place.
### Findings
- **[low]** The Mapping versioning / status model is a data-model decision that sits close to architecture territory, and several of its S1 and Later rules also appear in Foundation notes. (EXPERIENCE.md > Mapping versioning). *Fix:* keep the UI-visible rules, and move model rules to architecture.

## Mechanical notes
- No Mermaid in either spine. No broken `{token}` references found.
- Dark tokens use a `-dark` suffix convention, consistently for all 21 light tokens. Check against `references/design-md-spec.md` for the preferred pairing shape.
- Component names must match across files: see 3.
- Memlog says renderer invented a "Needs attention" status pill. A scan of the five mocks found no such text (not otherwise verified).
- EXPERIENCE "Saved." appears both as a state indicator and a toast. Pick one.
