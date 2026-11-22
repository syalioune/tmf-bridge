---
name: Graphite Guided
colors:
  surface: '#101319'
  surface-dim: '#101319'
  surface-bright: '#363940'
  surface-container-lowest: '#0b0e14'
  surface-container-low: '#191c22'
  surface-container: '#1d2026'
  surface-container-high: '#272a30'
  surface-container-highest: '#32353b'
  on-surface: '#e1e2eb'
  on-surface-variant: '#bec8cf'
  inverse-surface: '#e1e2eb'
  inverse-on-surface: '#2d3037'
  outline: '#899299'
  outline-variant: '#3f484e'
  surface-tint: '#7cd0ff'
  primary: '#85d3ff'
  on-primary: '#00344a'
  primary-container: '#5cb8e8'
  on-primary-container: '#004762'
  inverse-primary: '#00668a'
  secondary: '#99cbff'
  on-secondary: '#003354'
  secondary-container: '#074c7a'
  on-secondary-container: '#8abdf1'
  tertiary: '#7adda4'
  on-tertiary: '#003920'
  tertiary-container: '#5ec18a'
  on-tertiary-container: '#004c2d'
  error: '#ffb4ab'
  on-error: '#690005'
  error-container: '#93000a'
  on-error-container: '#ffdad6'
  primary-fixed: '#c4e7ff'
  primary-fixed-dim: '#7cd0ff'
  on-primary-fixed: '#001e2c'
  on-primary-fixed-variant: '#004c69'
  secondary-fixed: '#cfe5ff'
  secondary-fixed-dim: '#99cbff'
  on-secondary-fixed: '#001d33'
  on-secondary-fixed-variant: '#024a77'
  tertiary-fixed: '#93f7bc'
  tertiary-fixed-dim: '#77daa1'
  on-tertiary-fixed: '#002111'
  on-tertiary-fixed-variant: '#005231'
  background: '#101319'
  on-background: '#e1e2eb'
  surface-variant: '#32353b'
typography:
  display:
    fontFamily: Geist
    fontSize: 26px
    fontWeight: '600'
    lineHeight: 32px
    letterSpacing: -0.01em
  body:
    fontFamily: Geist
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  body-strong:
    fontFamily: Geist
    fontSize: 16px
    fontWeight: '600'
    lineHeight: 24px
  small:
    fontFamily: Geist
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 20px
  label:
    fontFamily: Geist
    fontSize: 14px
    fontWeight: '700'
    lineHeight: 20px
    letterSpacing: 0.04em
  code:
    fontFamily: JetBrains Mono
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 20px
  code-strong:
    fontFamily: JetBrains Mono
    fontSize: 14px
    fontWeight: '600'
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
  space-md: 1rem
  space-lg: 1.5rem
  space-xl: 2rem
---

## Brand & Style

The design system establishes a high-precision, low-cognitive-load developer workspace tailored for integration architects, API engineers, and enterprise system administrators managing complex data transformations. The primary posture is "Calm and Guided": an interface that reduces anxiety around mission-critical field mappings, payload transformations, and runtime synchronizations.

Drawing heavily from technical minimalism and low-glare dark productivity surfaces, the visual philosophy deliberately resists gratuitous decorations, ambient glowing accents, and non-semantic animations. Visual hierarchy is established strictly through tonal surface layering, purposeful contrast deltas, and sharp architectural borders. Interactions feel stable, deliberate, and linear, transforming dense configuration pipelines into transparent, sequential flows.

## Colors

The palette is rooted in deep graphite tones paired with an intentional, cool cyan accent to anchor key actions and active state focus. Every hue meets strict WCAG 2.2 AA contrast requirements against adjacent dark surfaces.

### Surface Tiers
- **Canvas / Background (`#0C0E12`)**: Foundational structural void; houses base framing.
- **Surface Layer 1 (`#14171D`)**: Standard surface for working panels, sidebars, and inactive canvas containers.
- **Surface Layer 2 (`#1C2028`)**: Elevated panels, active drawer states, tables, and modal cards.

### Text & Iconography
- **Text Main (`#E8EBF0`)**: High-contrast, clean off-white for primary data points, headlines, and critical path values.
- **Text Muted (`#9BA3B0`)**: Secondary metadata, inactive states, and structural hints (exceeds 4.5:1 on `#14171D`).

### Borders
- **Decorative Border (`#2E343F`)**: Structural segmentation lines, dividers, and passive grid separators.
- **Input Border (`#6B7482`)**: Deliberate, high-visibility perimeter applied to all interactive controls (inputs, dropdowns, code editors, check controls) in neutral rest states.

### Core Action & Interactive Tokens
- **Primary (`#5CB8E8`)**: Action anchors, active segment fills, primary triggers, and key highlights.
- **Primary Hover (`#7CC8F0`)**: Interactive feedback state for primary buttons.
- **On-Primary (`#04141F`)**: Deep ink tint for high-legibility text over primary cyan fills.
- **Focus Ring (`#5CB8E8`)**: Standardized 2px solid ring with 2px offset for accessible keyboard navigation.

### Semantic Status Pairs (Foreground on Background Container)
- **Success**: `#6FD29A` on `#12301F`
- **Warning / ID-Map Warning**: `#F2BE5C` on `#3A2C0C`
- **Error**: `#FF9A9A` on `#3E1818`
- **Info**: `#8BBEF2` on `#14283E`

## Typography

Typography relies on a crisp system-grade grotesque pairing (`Geist`, falling back to standard `system-ui` / `Segoe UI`) alongside an engineering monospace stack (`JetBrains Mono`, falling back to `ui-monospace` / `Menlo` / `Consolas`). 

### Core Constraints
- **Strict Size Floor**: Never render text below `14px` (`0.875rem`). Every label, status badge, secondary timestamp, tooltip, and code snippet remains at or above this threshold to prevent visual strain.
- **Monospace Deployment**: Dedicated exclusively to runtime payloads, JSON structures, cURL requests, field path transformations (e.g., `payload.billing.id`), and secret variable handles.
- **Labels**: Styled strictly uppercase with explicit tracking (`letter-spacing: 0.04em`) to establish visual separation from neighboring data values without reducing font size.

## Layout & Spacing

The system leverages a fluid-responsive multi-panel grid framework optimized for 16:9 and 16:10 workspace displays, breaking down into single-column vertical flows on narrow screens.

### Grid Structure
- **Desktop (≥ 1280px)**: 12-column dynamic workbench with fixed-width contextual toolbars (`320px` to `400px`), `1.5rem` gutters, and `2rem` outer frame padding. 
- **Tablet (768px - 1279px)**: Dual-pane split or tabbed primary stack; gutters clamp to `1rem` with `1.5rem` margins.
- **Mobile (< 768px)**: Single stacked layout; `1rem` margins and uniform `1rem` vertical stacking.

### Spacing Principles
Padding within controls respects internal touch boundaries while preserving density. Form controls utilize `0.5rem` (`space-sm`) vertical rhythm with `1rem` (`space-md`) horizontal inset. Section containers leverage `1.5rem` (`space-lg`) internal breathing room to cleanly separate mapping configurations.

## Elevation & Depth

This system forbids elevation drop shadows, soft ambient blurs, and diffuse colored halos. Spatial depth is communicated exclusively through two physical properties: **Tonal Stepping** and **High-Contrast Perimeter Framing**.

### Layering Hierarchy
1. **Root Bed (`#0C0E12`)**: Lowest stratum. Structural foundation.
2. **Work Surface (`#14171D`)**: Standard containers, mapping panels, and workspace canvases. Bound by a `1px` border of `#2E343F`.
3. **Elevated Surface (`#1C2028`)**: Active workflow states, selected mapping rows, tooltips, dialogs, and slide-over inspectors. Bound by a `1px` border of `#6B7482` to cleanly pop out against base surfaces.

### Interaction Depth
Hover and active states do not lift elements on a Z-axis. Instead, state transitions change the background tone incrementally (e.g., `#14171D` → `#1C2028`) or activate the primary border/ring treatment.

## Shapes

The design uses a clean, predictable hierarchy of corner radiuses, balancing tactile engineering precision with rounded ergonomics.

### Standard Geometry (12px / 0.75rem)
Applied ubiquitously across standard interactors:
- Action buttons, tabs, and panel headers.
- Input fields, text areas, and code editor wrappers.
- Mapping cards, callout tip boxes, and notification banners.

### Micro Geometry (4px / 0.25rem)
Reserved strictly for micro-chips:
- Inline keyboard shortcuts (`<kbd>Ctrl + K</kbd>`).
- Inline code tags and payload key fragments embedded in body copy.

### Fully Circular Geometry (`9999px`)
Used exclusively for state indicators and circular reference anchors:
- Stepper sequence circles (`1`, `2`, `3`).
- Standalone status dot pills and compact status badges.
- Help and diagnostic marker targets (`?`).

## Components

### Buttons
- **Primary**: Solid `#5CB8E8` fill with `#04141F` text, `font-weight: 600`, 12px radius. Hover shifts fill to `#7CC8F0`. Focus applies a 2px ring of `#5CB8E8` offset by 2px `#0C0E12`.
- **Secondary / Ghost**: Background transparent or `#14171D`, bounded by a 1px border of `#6B7482`, text `#E8EBF0`. Hover elevates surface to `#1C2028`.
- **Destructive**: Background `#3E1818`, border 1px `#FF9A9A`, text `#FF9A9A`.

### Input Fields & Controls
- **Rest State**: `#14171D` surface background, 1px perimeter border of `#6B7482`, 12px radius, text `#E8EBF0`, placeholder `#9BA3B0`.
- **Focus State**: Border switches to `#5CB8E8` accompanied by the standard 2px cyan focus ring.
- **Error State**: Border shifts to `#FF9A9A` with `#3E1818` background highlight.

### Checkboxes & Radios
- Size: 18x18px, 4px radius for checkboxes; circular for radios.
- Rest state bounded by 1.5px `#6B7482`. Checked state filled with `#5CB8E8` with `#04141F` checkmark.

### Cards & Mapping Panels
- Background `#14171D`, 12px radius, enclosed in a 1px `#2E343F` outline.
- Active/Selected mapping row: switches to `#1C2028` background with an active 1px `#6B7482` outline.

### Status Badges & Chips
- Fully rounded (`9999px`) for system states, or 12px for semantic banners.
- Rendered using the semantic pairs: text colored via foreground token, resting on the corresponding dark tint container (e.g., `#6FD29A` on `#12301F`).

### Code Inspector & Terminal Bars
- Background `#0C0E12` with 12px outer radius and `#2E343F` divider lines.
- Field paths, JSON payloads, and cURL commands styled in `14px` JetBrains Mono text with `#5CB8E8` syntax markers.