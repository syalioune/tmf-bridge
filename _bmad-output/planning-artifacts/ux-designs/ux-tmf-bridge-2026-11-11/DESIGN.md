---
status: final
name: TMF-Bridge Studio
description: Calm, guided, professional mapping studio for telco integration analysts. Graphite surfaces, one deep cyan-blue accent, a persistent inspector, plain-language help everywhere.
created: 2026-11-15
updated: 2026-11-15
sources:
  - ../../prds/prd-tmf-bridge-2026-11-09/prd.md
  - ./EXPERIENCE.md
  - ./.memlog.md
colors:
  background: '#F2F3F5'
  surface: '#FFFFFF'
  surface-raised: '#F8F9FA'
  text: '#16191F'
  text-muted: '#505763'
  border: '#CDD1D8'
  input-border: '#7A828F'
  primary: '#005176'
  primary-hover: '#003D59'
  on-primary: '#FFFFFF'
  focus-ring: '#005176'
  success: '#14663B'
  success-bg: '#E3F4EA'
  warning: '#7A4A00'
  warning-bg: '#FFF1CC'
  error: '#A31F1F'
  error-bg: '#FDECEC'
  info: '#1F5F99'
  info-bg: '#E4EEF9'
  code-surface: '#0C0E12'
  id-map-warning: '#7A4A00'
  id-map-warning-bg: '#FFF1CC'
  background-dark: '#0C0E12'
  surface-dark: '#14171D'
  surface-raised-dark: '#1C2028'
  text-dark: '#E8EBF0'
  text-muted-dark: '#9BA3B0'
  border-dark: '#2E343F'
  input-border-dark: '#6B7482'
  primary-dark: '#85D3FF'
  primary-hover-dark: '#A6DFFF'
  on-primary-dark: '#00344A'
  focus-ring-dark: '#85D3FF'
  success-dark: '#6FD29A'
  success-bg-dark: '#12301F'
  warning-dark: '#F2BE5C'
  warning-bg-dark: '#3A2C0C'
  error-dark: '#FF9A9A'
  error-bg-dark: '#3E1818'
  info-dark: '#8BBEF2'
  info-bg-dark: '#14283E'
  id-map-warning-dark: '#F2BE5C'
  id-map-warning-bg-dark: '#3A2C0C'
typography:
  display:
    fontFamily: system-ui, 'Segoe UI', sans-serif
    fontSize: 26px
    fontWeight: '600'
    lineHeight: '1.3'
  body:
    fontFamily: system-ui, 'Segoe UI', sans-serif
    fontSize: 16px
    fontWeight: '400'
    lineHeight: '1.5'
  body-strong:
    fontFamily: system-ui, 'Segoe UI', sans-serif
    fontSize: 16px
    fontWeight: '600'
    lineHeight: '1.5'
  small:
    fontFamily: system-ui, 'Segoe UI', sans-serif
    fontSize: 14px
    fontWeight: '400'
    lineHeight: '1.5'
  label:
    fontFamily: system-ui, 'Segoe UI', sans-serif
    fontSize: 14px
    fontWeight: '700'
    lineHeight: '1.3'
    letterSpacing: 0.04em
  code:
    fontFamily: ui-monospace, Menlo, Consolas, monospace
    fontSize: 14px
    fontWeight: '400'
    lineHeight: '1.5'
rounded:
  sm: 4px
  DEFAULT: 12px
  full: 9999px
spacing:
  '1': 4px
  '2': 8px
  '3': 12px
  '4': 16px
  '5': 24px
  '6': 32px
  inspector-width: 340px
  gutter: 16px
  target-min: 24px
  breakpoint-min: 1280px
  breakpoint-stack: 1024px
components:
  top-bar:
    background: '{colors.surface}'
    border-bottom: '{colors.border}'
    padding: '{spacing.3}'
    link-foreground: '{colors.primary}'
  stepper:
    background: '{colors.surface}'
    border: '{colors.border}'
    item-radius: '{rounded.full}'
    item-padding: '{spacing.2}'
    marker-size: '{spacing.target-min}'
    current-background: '{colors.info-bg}'
    current-foreground: '{colors.text}'
    current-marker-background: '{colors.primary}'
    current-marker-foreground: '{colors.on-primary}'
    current-bar: '{colors.primary}'
    done-foreground: '{colors.success}'
    todo-foreground: '{colors.text-muted}'
  mapping-table-row:
    background: '{colors.surface}'
    border: '{colors.border}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.3}'
    selected-background: '{colors.info-bg}'
    selected-border: '{colors.primary}'
    warning-border: '{colors.id-map-warning}'
    warning-background: '{colors.id-map-warning-bg}'
    empty-border: '{colors.input-border}'
  mappings-list-table:
    background: '{colors.surface}'
    border: '{colors.border}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.3}'
  field-list:
    background: '{colors.surface}'
    border: '{colors.border}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.3}'
    item-min-height: '{spacing.target-min}'
  wiring-layer:
    link-color: '{colors.primary}'
    warning-link-color: '{colors.id-map-warning}'
    warning-link-style: dashed
  connect-picker:
    background: '{colors.surface}'
    border: '{colors.input-border}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.3}'
    option-selected-background: '{colors.info-bg}'
  transform-picker:
    background: '{colors.surface}'
    border: '{colors.input-border}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.3}'
  mode-switch:
    border: '{colors.input-border}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.1}'
    selected-background: '{colors.primary}'
    selected-foreground: '{colors.on-primary}'
  field-input:
    background: '{colors.surface}'
    foreground: '{colors.text}'
    border: '{colors.input-border}'
    radius: '{rounded.DEFAULT}'
    focus-ring: '{colors.focus-ring}'
    invalid-border: '{colors.error}'
    disabled-background: '{colors.surface-raised}'
    disabled-foreground: '{colors.text-muted}'
  sample-editor:
    background: '{colors.surface}'
    foreground: '{colors.text}'
    border: '{colors.input-border}'
    radius: '{rounded.DEFAULT}'
    typography: '{typography.code}'
    invalid-border: '{colors.error}'
  inspector:
    background: '{colors.surface-raised}'
    border: '{colors.border}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.4}'
    width: '{spacing.inspector-width}'
    label-foreground: '{colors.text-muted}'
  verdict-bar:
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.4}'
    border-width: 2px
    error-background: '{colors.error-bg}'
    error-foreground: '{colors.error}'
    success-background: '{colors.success-bg}'
    success-foreground: '{colors.success}'
  exchange-panel:
    background: '{colors.surface}'
    border: '{colors.border}'
    failing-border: '{colors.error}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.4}'
    code-typography: '{typography.code}'
    code-background: '{colors.code-surface}'
    code-foreground: '{colors.text-dark}'
  summary-tile:
    background: '{colors.surface}'
    border: '{colors.border}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.3}'
    label-typography: '{typography.label}'
    label-foreground: '{colors.text-muted}'
    value-typography: '{typography.body-strong}'
    ok-foreground: '{colors.text}'
    fail-foreground: '{colors.error}'
    warning-foreground: '{colors.id-map-warning}'
    warning-background: '{colors.id-map-warning-bg}'
    breakpoint: '{spacing.breakpoint-stack}'
  share-bar:
    background: '{colors.surface-raised}'
    foreground: '{colors.text}'
    border-top: '{colors.primary}'
    border-top-width: 3px
    padding: '{spacing.3}'
  apply-summary:
    background: '{colors.surface}'
    border: '{colors.border}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.4}'
  help-panel:
    background: '{colors.surface-raised}'
    border: '{colors.border}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.4}'
    width: '{spacing.inspector-width}'
  glossary-term:
    foreground: '{colors.text}'
    underline: '{colors.primary}'
    underline-style: dotted
    popover-background: '{colors.info-bg}'
    popover-foreground: '{colors.info}'
    min-height: '{spacing.target-min}'
  inline-tip:
    background: '{colors.info-bg}'
    foreground: '{colors.info}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.3}'
  help-link:
    size: '{spacing.target-min}'
    background: '{colors.primary}'
    foreground: '{colors.on-primary}'
    radius: '{rounded.full}'
  status-badge:
    radius: '{rounded.full}'
    padding: '{spacing.2}'
    typography: '{typography.label}'
    draft-background: '{colors.surface-raised}'
    draft-foreground: '{colors.text-muted}'
    draft-border: '{colors.input-border}'
    live-background: '{colors.success-bg}'
    live-foreground: '{colors.success}'
    warning-background: '{colors.warning-bg}'
    warning-foreground: '{colors.warning}'
    error-background: '{colors.error-bg}'
    error-foreground: '{colors.error}'
  banner:
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.4}'
    border-width: 1px
    info-background: '{colors.info-bg}'
    warning-background: '{colors.warning-bg}'
    error-background: '{colors.error-bg}'
    success-background: '{colors.success-bg}'
  toast:
    background: '{colors.surface-raised}'
    foreground: '{colors.text}'
    border: '{colors.input-border}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.3}'
  confirm-dialog:
    background: '{colors.surface}'
    border: '{colors.input-border}'
    radius: '{rounded.DEFAULT}'
    padding: '{spacing.5}'
  skeleton-row:
    background: '{colors.surface-raised}'
    radius: '{rounded.DEFAULT}'
    height: '{spacing.6}'
  toggle:
    border: '{colors.input-border}'
    radius: '{rounded.DEFAULT}'
    pressed-background: '{colors.info-bg}'
    min-height: '{spacing.target-min}'
  code-chip:
    background: '{colors.surface-raised}'
    foreground: '{colors.text}'
    border: '{colors.input-border}'
    radius: '{rounded.sm}'
    typography: '{typography.code}'
  button-primary:
    background: '{colors.primary}'
    foreground: '{colors.on-primary}'
    hover-background: '{colors.primary-hover}'
    active-background: '{colors.primary-hover}'
    disabled-background: '{colors.surface-raised}'
    disabled-foreground: '{colors.text-muted}'
    disabled-border: '{colors.border}'
    radius: '{rounded.DEFAULT}'
  button-secondary:
    background: 'transparent'
    foreground: '{colors.primary}'
    border: '{colors.primary}'
    hover-background: '{colors.info-bg}'
    disabled-foreground: '{colors.text-muted}'
    disabled-border: '{colors.border}'
    radius: '{rounded.DEFAULT}'
---

## Brand & Style

TMF-Bridge Studio is a calm, guided workbench for Inès, an integration analyst who knows telco but is not a developer. The register is professional, calm and approachable: a quiet help desk, not a cockpit and not a toy. The visual direction is "Calm and guided", with the always-present inspector borrowed from the "Dense workbench" direction. The editorial and playful directions were rejected as wrong for a professional tool.

There is no existing brand. The identity is deliberately plain so the analyst's mapping and the product's explanations are the focus. Chrome stays neutral graphite, one cool cyan accent marks what you can act on, and everything else is information. Help (inline tips, `?` links to the manual, glossary terms) is part of the look, not an afterthought: it must be visible, readable and never feel like an error.

Visual references (rendered, illustrative; the spines win on conflict, see `EXPERIENCE.md` Visual references):
- Mockups: [Map request](mockups/mock-map-request.html), [Exchange view](mockups/mock-exchange-view.html), [Apply summary](mockups/mock-apply-summary.html), [Mappings list](mockups/mock-home-mappings.html), [Sign in](mockups/mock-login.html), [Backends](mockups/mock-backends.html), [Id map](mockups/mock-id-map.html), [Help panel](mockups/mock-help-panel.html).
- Stitch exports (raw, light and dark): [imports/stitch/](imports/stitch/). Reference only; the palette comes from them, typography stays the system stack.
- Wireframes: [IA map](wireframes/ia-2026-11-15.excalidraw), [Map request flow](wireframes/flow-map-request-2026-11-15.excalidraw).

## Colors

Defined for light and dark; dark tokens carry a `-dark` suffix. The default follows the system setting, with a manual toggle that is remembered per browser. Hex values live in the frontmatter only; this section gives roles.

- **background / surface / surface-raised**: page, work surface, and raised panels (inspector, share bar, help panel). Tone separates them, not shadow.
- **text / text-muted**: body and secondary text. Muted text meets WCAG AA contrast for body text in both themes. Never use it for essential content that could look disabled.
- **primary** with **primary-hover** and **on-primary**: the single accent. Primary action, selected row, links, the `?` help marker. Not used for status.
- **border**: decorative separators only (about 1.5:1, below the 3:1 that UI components need). Nothing essential relies on it.
- **input-border**: the edge of any control a user must find (inputs, selects, secondary buttons, toggles, empty drop targets, draft badge, popovers). Exists because `border` fails 3:1.
- **focus-ring**: the keyboard focus indicator, a 3px outline with a 2px offset. It is never a fill or a 2px border, so shape tells it apart from selection. Halo recipe: see Components. Same hex as primary; the 2px surface halo (mandatory) keeps it distinct on filled buttons.
- **success / warning / error / info** with matching `-bg` tints: shared status set, always paired with an icon and word. Slice 1 (the first delivery slice, tier S1 in the PRD) values are verified for contrast below.
- **id-map-warning** and `-bg`: the non-blocking highlight for an id field mapped without the id-map transform. Same hue family as warning, but a named token so it can diverge.
- **code-surface**: the always-dark background for code blocks in the Exchange view, in both themes. It reuses the `background-dark` hex; text on it uses `text-dark`.
- **Invalid and disabled**: an invalid field has an `error` border plus an inline message, never red alone. A disabled button or field has a `surface-raised` fill, `text-muted` text and `border` outline, is `aria-disabled`, and always shows the reason as text (disabled controls are exempt from contrast, but the reason text is not).

Contrast ratios are computed in Python from the frontmatter hex values. WCAG 2.2 AA requires 4.5:1 for text and 3:1 for UI components.

| Pair | Light | Dark |
|---|---|---|
| text on surface / on background | 17.60 / 15.85 | 15.02 / 16.16 |
| text-muted on surface / on background / on surface-raised | 7.28 / 6.56 / 6.91 | 7.06 / 7.60 / 6.42 |
| primary on surface (links, secondary button text) | 8.61 | 10.90 |
| primary on info-bg (selected row, current step) | 7.34 | 9.1 |
| on-primary on primary / on primary-hover | 8.61 / 11.59 | 8.02 / 9.19 |
| success / warning / error / info on own `-bg` | 6.14 / 6.66 / 6.63 / 5.67 | 7.74 / 7.96 / 7.67 / 7.66 |
| input-border on surface / background / surface-raised | 3.88 / 3.49 / 3.68 | 3.80 / 4.09 / 3.46 |
| input-border on info-bg | 3.30 | 3.17 |
| focus-ring on surface / on background | 8.61 / 7.76 | 10.90 / 11.74 |

Forced-colors mode: borders, `outline` and system colors replace background-only cues (selected row, pills, banners); wiring strokes use `currentColor`.

## Typography

System font stack (`system-ui`) for the interface and `ui-monospace` for payloads, JSON paths and curl. No web font is loaded. Technical terms stay (payload, JSON path, transform, lineage, OpenAPI), so type must make them legible and recognizable.

- **display** 26/600: page titles (mapping name, step title). One `h1` per surface.
- **body** 16/400 and **body-strong** 16/600: default reading text and row labels.
- **small** 14/400: inspector detail, tips, secondary lines.
- **label** 14/700 tracked 0.04em: column headers, status badges and group headings only. The token sets weight and tracking, not letter case, so text is rendered as written.
- **code** 14 mono: field paths, sample values, HTTP exchange, secret names.

Never set essential text below 14px, including table headers, captions, badges, chips and footer text. Dense tables use `small`, not smaller. Text must resize to 200% without loss (see Layout & Spacing for reflow).

## Layout & Spacing

Desktop web. Scale 4 / 8 / 12 / 16 / 24 / 32 (`{spacing.1}` to `{spacing.6}`); sizes in components reference spacing tokens, not raw px. Minimum interactive target is `{spacing.target-min}` (24px; WCAG 2.5.8).

The default workspace is guided single-column, with the stepper across the top. Side-by-side mode puts the editor left and live test and lineage right. The inspector is a persistent `{spacing.inspector-width}` (340px) column with an `{spacing.gutter}` gap, showing the selected mapping detail and its contextual help.

Breakpoints (one rule, shared with `EXPERIENCE.md` Responsive and platform): the designed and supported minimum is `{spacing.breakpoint-min}` (1280px). Below `{spacing.breakpoint-stack}` (1024px), and at 200% and 400% zoom (down to 320 CSS px), the layout reflows: the inspector and Help panel stack under the work area as disclosures, side-by-side mode stacks editor above test, and the mapping table keeps its columns inside its own labelled horizontal scroll region (`tabindex="0"`). Nothing else scrolls sideways.

Mapping tables are dense but never cramped: rows keep `{spacing.3}` padding, and group headings sit above, not inside, rows.

## Elevation & Depth

Flat. Depth comes from tone (background, surface, surface-raised) and 1px borders, not shadow. The only stacking cue is the sticky share bar, marked by a 3px primary top border. Modals, popovers and menus may use a single subtle shadow if the platform requires one. The focus halo is the one other permitted `box-shadow`.

## Shapes

`{rounded.DEFAULT}` for rows, panels, inputs, tips, bars, dialogs, popovers and buttons: friendly but not toy-like. `{rounded.sm}` for the code chip (keyboard keys and inline code). `{rounded.full}` for stepper items, status badges and the `?` marker only. Drop targets and warnings use a 2px dashed border so they never depend on color.

## Components

Component names are canonical and identical in `EXPERIENCE.md` Component Patterns. Behavior lives there; this section is the look. Values come from the component tokens in the frontmatter; this section names the tokens and adds what tokens cannot express.

Focus halo (all focusable elements): the `focus-ring` outline, with its 2px offset filled by a `surface` halo (`box-shadow: 0 0 0 2px surface`) so the ring reads on tinted backgrounds and on primary-filled buttons.

- **Top bar**: `surface`, bottom `border`. Logo and home, Backends link, Id map link (the current page link carries `aria-current="page"` and a bold weight with a 3px `primary` underline, never color alone), Help `?`, Dark mode toggle, account menu (Sign out only). Same order on every authenticated surface.
- **Stepper**: pill items in a wrapping row on `surface`, bottom `border`. Each item has a `{spacing.target-min}` numbered marker. Done: `success` marker and check icon, a link. Current: `info-bg` pill, bold text, `primary` filled marker, 3px `primary` bar along the bottom; no outline, so it never mimics the focus ring. To do: `text-muted`. Step name is always text. Items are links or buttons with `aria-current="step"` on the current one.
- **Mapping table row**: `surface`, 1px `border`. Columns: TMF field, transform, backend field. Selected row: `info-bg` fill, 4px `primary` bar and 2px `primary` border, plus the word "selected" and `aria-current`. Empty target: dashed `input-border`. Id warning: dashed `id-map-warning` border, `id-map-warning-bg` fill, warning icon plus text badge. The first cell is a row header (`th scope="row"`). The table is the source of truth; the wiring layer is an overlay. See [Map request mock](mockups/mock-map-request.html).
- **Mappings list table**: same look as the mapping table, columns Name (a link), TMF API and operation, Backend, Status. See [Mappings list mock](mockups/mock-home-mappings.html).
- **Field list**: two bordered lists (TMF inputs, backend fields); each item shows name and type and has a Connect button of at least `{spacing.target-min}`.
- **Wiring layer**: a single labeled line in `primary` (solid), warnings dashed with an icon, hover preview faint. No lines appear at rest. The layer is aria-hidden.
- **Connect picker**: a popover with a labeled search field and a listbox; selected option `info-bg`; `input-border` edge; Esc closes.
- **Transform picker**: same shell as the Connect picker; each option shows name, one-line explanation and a `?`.
- **Mode switch**: two-button group with `{spacing.1}` inner padding and a 4px gap so focus rings are not clipped; selected is `primary` fill with `on-primary` text; `aria-pressed`. Never inside an `overflow:hidden` parent.
- **Field input**: `surface` fill, `input-border`, `{rounded.DEFAULT}`, `focus-ring` on focus, label above in `body-strong`. Invalid: `error` border plus message. Disabled: `surface-raised` fill plus reason text.
- **Sample editor**: a plain multi-line field in `code` type with `field-input` styling; invalid shows `error` border and the line and column message beneath.
- **Inspector**: `surface-raised`, `border`, `{rounded.DEFAULT}`, `{spacing.4}` padding, `{spacing.inspector-width}` wide. Title, a definition list (label in `text-muted`, value in `body` or `code`), then inline tip and `?` link. Empty state: one sentence and a tip.
- **Verdict bar**: full-width block at the top of the Exchange view, 2px border, `{rounded.DEFAULT}`. Failing: `error` on `error-bg`, an icon, a headline (a heading), one explanatory sentence, and a short checklist using the words Passed, Failed or Skipped. Passing: `success` on `success-bg`. See [Exchange view mock](mockups/mock-exchange-view.html).
- **Summary tile**: four tiles in a row under the verdict bar, `surface`, `border`, `{rounded.DEFAULT}`. Label in `label` type, value in `body-strong` (endpoint in `code` type); the label keeps the case it is written in. States: normal (`text` value); failed value (`error` text plus icon and word, for latency timeout or no response); id-map advisory (the Id-map status tile uses `id-map-warning` on its `-bg` with the warning icon and the words "Missing key", never `error`); not run yet (tiles hidden, not zeroed). Two columns below `{spacing.breakpoint-stack}`, one column below 480px; values wrap and never force sideways scroll. See [Exchange view mock](mockups/mock-exchange-view.html).
- **Exchange panel**: three panels, `surface`, `border`; the failing panel has a 2px `error` border and is expanded; others collapse to a one-line summary. Code blocks use `code` type on `code-surface`, keyboard-focusable.
- **Share bar**: sticky bottom, `surface-raised`, 3px `primary` top border. Copy as curl, Copy as Postman, Download bundle, and a code-styled secret name. Stops being sticky when the viewport is under 600px tall; the page keeps matching scroll padding so focus is never hidden behind it. The secret value is never rendered.
- **Apply summary**: `surface` card with a definition list (API and operation, backend, secret NAME) and a warning banner "This replaces the live version." See [Apply summary mock](mockups/mock-apply-summary.html).
- **Help panel**: non-modal complementary region on `surface-raised`, same width as the inspector, with tabs (Help, Glossary) and a search field; dockable.
- **Glossary term**: a button with a dotted `primary` underline (distinct from links' solid underline), `{spacing.target-min}` minimum height. Hover and focus open a popover (`info-bg`, `info` text, `{rounded.DEFAULT}`) with the definition and a link; Esc dismisses; the pointer can enter the popover.
- **Inline tip**: `info-bg` panel, `info` text, `{rounded.DEFAULT}`, icon plus text, with a link to the matching manual section.
- **Help link (`?`)**: `{spacing.target-min}` round marker, filled `primary` with an `on-primary` glyph, accessible name "Help: {topic}"; with visible text where space allows.
- **Status badge**: pill, `label` type, icon plus word. Draft: `surface-raised` fill, `text-muted` text, `input-border` outline, hollow circle icon. Live: `success-bg` fill, `success` text, check icon. Warning and Error use their own pairs. Slice 1 has only Draft and Live as mapping statuses.
- **Banner**: `{rounded.DEFAULT}`, 1px status-colored border, status tint, icon plus text; used for a preview that has not run, offline, session ended, stale version and id-map warnings.
- **Toast**: `surface-raised`, `input-border`, bottom-left; for non-error confirmations only ("Copied.").
- **Confirm dialog**: `surface`, `{rounded.DEFAULT}`, `{spacing.5}` padding, backdrop dimmed; one level only.
- **Skeleton row**: `surface-raised` bar, no shimmer when reduced motion is on.
- **Toggle**: bordered button, `info-bg` when pressed; used for Dark mode and "Show all links".
- **Code chip**: `{rounded.sm}`, `surface-raised`, `input-border`, `code` type.
- **Buttons**: primary (`primary` fill; `primary-hover` on hover and when pressed) and secondary (transparent, `primary` text and 2px border, `info-bg` on hover). Disabled buttons follow Colors, Invalid and disabled. 2px border on both, so sizes match. One primary per view. Every button has a unique accessible name in context (for example "Edit id row", "Export Product orders lookup").

## Do's and Don'ts

| Do | Don't |
|---|---|
| Use `input-border` for every control edge | Use `border` for inputs, selects or drop targets |
| Pair every status color with an icon and text | Signal state by color alone |
| Show the `focus-ring` (with its halo) on every focusable element | Remove or recolor the focus outline |
| Keep technical terms and attach a glossary term or inline tip | Replace terms with vague words or leave them unexplained |
| Keep warnings (id-map) non-blocking in look: dashed, amber, with explanation | Use error red for an advisory warning |
| Let tone and 1px borders create hierarchy | Add shadows (except for overlays), gradients or a second accent color |
| Show a secret by name only | Render or pre-fill a secret value anywhere |
| Treat the wiring layer as decoration over the table | Draw a mesh of arrows between the two tables |
| Give focusable controls room inside any scroll or clip region (at least 5px) | Put a focusable control against an `overflow:hidden` edge, or clip a focus ring |
| Size targets 24px or more; keep essential text at 14px or more | Ship 20px help markers or 12-13px headers |
| Mark the current step with a bold word, filled marker and bottom bar | Outline the current step like a focus ring |
