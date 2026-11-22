---
name: TMF-Bridge Studio
colors:
  surface: '#f9f9ff'
  surface-dim: '#d8dae2'
  surface-bright: '#f9f9ff'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#f2f3fc'
  surface-container: '#ecedf6'
  surface-container-high: '#e6e8f0'
  surface-container-highest: '#e1e2eb'
  on-surface: '#191c22'
  on-surface-variant: '#40484e'
  inverse-surface: '#2d3037'
  inverse-on-surface: '#eff0f9'
  outline: '#70787f'
  outline-variant: '#c0c7cf'
  surface-tint: '#11648e'
  primary: '#005176'
  on-primary: '#ffffff'
  primary-container: '#1c6a94'
  on-primary-container: '#c6e5ff'
  inverse-primary: '#8ccefd'
  secondary: '#585f6b'
  on-secondary: '#ffffff'
  secondary-container: '#d9e0ee'
  on-secondary-container: '#5c636f'
  tertiary: '#004f87'
  on-tertiary: '#ffffff'
  tertiary-container: '#2a67a1'
  on-tertiary-container: '#d0e3ff'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#c9e6ff'
  primary-fixed-dim: '#8ccefd'
  on-primary-fixed: '#001e2f'
  on-primary-fixed-variant: '#004c6e'
  secondary-fixed: '#dce3f1'
  secondary-fixed-dim: '#c0c7d5'
  on-secondary-fixed: '#151c26'
  on-secondary-fixed-variant: '#404753'
  tertiary-fixed: '#d2e4ff'
  tertiary-fixed-dim: '#9fcaff'
  on-tertiary-fixed: '#001d36'
  on-tertiary-fixed-variant: '#00497e'
  background: '#f9f9ff'
  on-background: '#191c22'
  surface-variant: '#e1e2eb'
typography:
  title:
    fontFamily: Segoe UI, system-ui, -apple-system, sans-serif
    fontSize: 26px
    fontWeight: '600'
    lineHeight: 34px
  body:
    fontFamily: Segoe UI, system-ui, -apple-system, sans-serif
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  body-strong:
    fontFamily: Segoe UI, system-ui, -apple-system, sans-serif
    fontSize: 16px
    fontWeight: '600'
    lineHeight: 24px
  small:
    fontFamily: Segoe UI, system-ui, -apple-system, sans-serif
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 20px
  label:
    fontFamily: Segoe UI, system-ui, -apple-system, sans-serif
    fontSize: 14px
    fontWeight: '700'
    lineHeight: 20px
    letterSpacing: 0.04em
  code:
    fontFamily: ui-monospace, Menlo, Consolas, monospace
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 20px
rounded:
  sm: 0.25rem
  DEFAULT: 0.5rem
  md: 0.75rem
  lg: 1rem
  xl: 1.5rem
  full: 9999px
spacing:
  gutter: 1rem
  gutter-desktop: 1.5rem
  margin: 1rem
  margin-desktop: 2rem
  space-xs: 0.25rem
  space-sm: 0.5rem
  space-md: 0.75rem
  space-lg: 1rem
  space-xl: 1.5rem
---

## Brand & Style

This design system delivers an exacting, low-cognitive-load environment engineered specifically for telecom integration analysts transforming complex schema architectures (such as TM Forum Open APIs, legacy billing formats, and BSS/OSS payload structures). 

The emotional signature is **Calm and Guided**:
- **Controlled & Predictable:** Complex payload field-mapping requires continuous vigilance. The visual language eliminates loud decorative ornamentation, gradient fills, and elevation drop-shadows in favor of razor-sharp structural clarity, uniform line weights, and explicit structural containment.
- **Utilitarian Elegance:** High-density enterprise tooling balanced by calm graphite surfaces and deliberate cool cyan accents. Visual priority is reserved exclusively for payload topology, schema validation states, and mapping integrity.
- **Forgiving & Informative:** Error states and transformation mismatches are communicated without punitive visual alarms. Discrepancies read as guided diagnostics, reducing cognitive fatigue during multi-hour schema reconciliation workflows.

## Colors

The palette employs a dual-tone graphite foundation illuminated by focused cool cyan anchors. Semantic colors are pair-locked with dedicated high-contrast background tints to guarantee legibility across technical telemetry.

### Light Palette (Default)
- **Canvas & Structural Surfaces:**
  - Background: `#F2F3F5` (Neutral low-strain canvas)
  - Surface: `#FFFFFF` (Primary work planes, inspect panels, mapping grids)
  - Surface Raised: `#F8F9FA` (Hover row targets, secondary table wells, header bars)
- **Typography & Boundaries:**
  - Text Primary: `#16191F`
  - Text Muted: `#505763`
  - Structural Border: `#CDD1D8`
  - Input Border: `#7A828F`
- **Brand & Interaction:**
  - Primary / Focus Ring: `#1C6A94`
  - Primary Hover: `#14557A`
  - On-Primary: `#FFFFFF`
- **Semantic Feedback States (Text on Background):**
  - Success: `#14663B` on `#E3F4EA`
  - Warning / ID-Map Warning: `#7A4A00` on `#FFF1CC`
  - Error: `#A31F1F` on `#FDECEC`
  - Info: `#1F5F99` on `#E4EEF9`

### Dark Palette
- **Canvas & Structural Surfaces:**
  - Background: `#0C0E12`
  - Surface: `#14171D`
  - Surface Raised: `#1C2028`
- **Typography & Boundaries:**
  - Text Primary: `#E8EBF0`
  - Text Muted: `#9BA3B0`
  - Structural Border: `#2E343F`
  - Input Border: `#6B7482`
- **Brand & Interaction:**
  - Primary / Focus Ring: `#5CB8E8`
  - Primary Hover: `#7CC8F0`
  - On-Primary: `#04141F`
- **Semantic Feedback States (Text on Background):**
  - Success: `#6FD29A` on `#12301F`
  - Warning / ID-Map Warning: `#F2BE5C` on `#3A2C0C`
  - Error: `#FF9A9A` on `#3E1818`
  - Info: `#8BBEF2` on `#14283E`

## Typography

The type scale strictly enforces that no essential interface text renders below `14px`, ensuring fatigue-free scanning of multi-level JSON structures and TMF specifications over prolonged analytical sessions.

- **Prose & Layout Hierarchy:**
  - `title` (`26px / 600`): Studio view headers, schema transformation stage titles, payload summaries.
  - `body` (`16px / 400`): General instruction text, descriptive mapping walkthroughs, modal dialog explanations.
  - `body-strong` (`16px / 600`): Critical field validations, key schema entities, section dividers.
  - `small` (`14px / 400`): Secondary descriptions, helper metadata, timestamp logs.
  - `label` (`14px / 700`, uppercase, `0.04em` tracking): Structural table headers, status pills, panel group labels, protocol flags.
- **Monospace Hierarchy:**
  - `code` (`14px / 400`, `ui-monospace, Menlo, Consolas`): Field path nodes (`tmf.serviceCandidate.id`), cURL command snippets, transformation scripts (JSLT/Liquid), schema keys, and masked secrets.

## Layout & Spacing

The layout is anchored on an asymmetric workspace architecture optimized for source-to-target field mapping:

- **Desktop Workbench (>= 1200px):**
  - Fixed-width contextual **Inspector Panel** (`340px`) pinned to the right edge.
  - Main schema workbench occupies the flexible remaining viewport, split evenly between Source Payload (Left) and Target TMF Spec (Center-Right).
  - Sticky utility footer anchored along the base with minimal z-index interference.
- **Responsive Stacking (< 1200px):**
  - Inspector Panel unpins from the side column and shifts into an expandable bottom drawer or stacked tab below the mapping table.
  - Mapping views shift from side-by-side comparison into a unified lineage table with togglable source/target columns.
- **Physical Touch & Mouse Targets:**
  - All interactive elements uphold a minimum clickable target dimension of `24px` for tight inline controls (expand arrows, help markers) and `44px` for primary workflows (action buttons, row selector hitboxes, stepper triggers).

## Elevation & Depth

This design system deliberately eschews standard ambient shadows, blurs, and gradient fills. Visual layering is achieved exclusively through **planar tone separation** and **crisp 1px boundary lines**.

- **Planar Tiering:**
  - Base level: `background`
  - Working containers, tables, and sidebars: `surface`
  - Active rows, hovered inputs, and headers: `surface-raised`
- **Border Specifications:**
  - Standard structural partitions: `1px solid border`
  - Input and form control boundaries: `1px solid input-border`
  - Focus indication: `2px solid focus-ring` with `2px` offset, no diffuse glow.
  - Drop targets and unconfirmed/ambiguous ID mapping nodes: `2px dashed border` (or `2px dashed id-map-warning` for identity mapping caveats).

## Shapes

Shape rules follow strict functional categories rather than uniform rounding:

- **12px Radius:** Major structural containers, workspace cards, mapping table rows, inspector panel frame, inputs, dropdown triggers, and modal dialogs.
- **4px Radius:** Inline keyboard shortcuts (`kbd`), chip indicators, code badges, and script syntax containers.
- **Fully Rounded (`9999px`):** Sequential steppers, circular status indicator badges, and inline help markers (`?`).

## Components

### Stepper
- Flow: Horizontal header tracking transformation phases (e.g., Schema Ingest -> Path Mapping -> Value Transform -> Validation).
- Nodes: Fully rounded circular indicators with 1px border. Completed steps use `primary` fill with `on-primary` icons. Active step uses a `2px solid primary` border with a centered dot. Step titles use `label` uppercase formatting.

### Mapping Table Row
- Geometry: `12px` border radius on row groups, separated by `space-xs` vertical gaps.
- Borders: `1px solid border`. On focus or hover: transitions background to `surface-raised`.
- States:
  - **Mapped & Validated:** Inline connection indicator with `success` token icon.
  - **ID-Map Warning:** Highlighted with a `2px dashed id-map-warning` border, indicating potential synthetic key or reference discrepancies.
  - **Unresolved Target:** Subtle dotted baseline with `text-muted` placeholder.

### Inspector Panel
- Dimensions: Fixed `340px` width on desktop; full width stacked on responsive screens.
- Anatomy: Anchored to the right. Header includes contextual field schema details, validation rules, TMF resource link, sample payload preview in `code` typography, and field transformation expression inputs.

### Status Pills & '?' Markers
- Status Pills: Fully rounded (`9999px`) badges using paired semantic background fills (e.g., `success` text on `success` background). Typography is strictly `label` (14px/700 uppercase).
- Help Markers: `24px` diameter fully circular buttons containing `?`, neutral borders, offering low-profile contextual popovers on click.

### Sticky Share & Export Bar
- Fixed bottom container spanning the workspace with `12px` top border radius or full edge-to-edge docking.
- Background: `surface` with a `1px solid border` top edge. Contains payload test runners, schema export cURL actions, and mapping verification summary.

### Verdict Bars & Informative Error Blocks
- Verdict Bars: Compact banner summarizing transformation health (e.g., "142 Mapped / 0 Errors / 3 Warnings"). Solid `surface-raised` background with a left accent border corresponding to the most severe state.
- Error Blocks: Non-alarmist, high-clarity containers. Built with `12px` radius, `1px solid border`, and semantic `error` text on `error` background. Error blocks must display the JSON path in `code` font, the failed rule, and actionable remediation steps.