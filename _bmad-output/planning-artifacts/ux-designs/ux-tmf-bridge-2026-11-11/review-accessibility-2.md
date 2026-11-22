# Accessibility review 2: palette swap and summary tiles (WCAG 2.2 AA)

Scope: 2026-11-15 changes only. DESIGN.md, EXPERIENCE.md, mockups/mock-exchange-view.html. Read-only on spines and mocks. Ratios computed in Python (WCAG relative luminance) from the actual hex values; script at `.../scratchpad/c.py`. Supersedes the focus-ring section of review-accessibility.md, which assumed ring #1C6A94 / #5CB8E8 (the ring is now `primary`).

Counts: Critical 0, Major 2, Moderate 5, Low 6.

## 1. Contrast verification

Every ratio claimed in the DESIGN.md Colors table recomputes exactly (light / dark): text 17.60/15.85 and 15.02/16.16; muted 7.28/6.56/6.91 and 7.06/7.60/6.42; primary on surface 8.61 / 10.90; primary on info-bg 7.34 / 9.10; on-primary on primary 8.61 / 8.02 and on hover 11.59 / 9.19; status pairs 6.14 to 7.96; input-border 3.88/3.49/3.68 and 3.80/4.09/3.46; input-border on info-bg 3.30 / 3.17; focus-ring on surface/bg 8.61/7.76 and 10.90/11.74. No claim is wrong.

Pairs the table does not list, all pass:
- primary on raised 8.17 / 9.92; on warn-bg 7.67 / 8.26; on err-bg 7.54 / 9.46; on ok-bg 7.54 / 8.69. This is the focus ring against every tinted background it can sit on (min 7.34 L, 8.26 D). Needed 3:1.
- error on surface 7.57 / 8.84 (tile `.bad .tv`); on info-bg 6.45 / 7.37. Muted label on info-bg 6.20 / 5.89, on warn-bg 6.48 / 5.34.
- hover text primary-hover on surface 11.59 / 12.50, on info-bg 9.88 / 10.42.

primary-container (#1C6A94 L, #5CB8E8 D), non-text use (needs 3:1):
- vs surface 5.93 / 8.10, vs background 5.35 / 8.72, vs raised 5.63 / 7.37, vs info-bg 5.06 / 6.76. Pass everywhere it can be a bar or marker.
- on-primary on container 5.93 / 5.96, so it even passes as a text background. Text colour on it fails: `text` 2.97 L / 1.85 D. The "never for text" rule holds, and nothing may put body text on it.
- Container vs primary is 1.45 L / 1.35 D, so the two are the same colour to a user. They cannot encode a state difference (see L2).

Focus ring (primary) vs adjacent colours:
- Ring vs everything outside it is 7.3 to 11.7 (see above). Ring vs the 2px halo (surface) is 8.61 / 10.90. The 3px ring and 3:1 requirement pass for 2.4.7 and 2.4.11.
- On a primary-filled button the ring equals the fill (1.00), but the 2px offset is filled by the `surface` halo (`box-shadow: 0 0 0 2px` matches `outline-offset:2px` exactly, mock line 57), so the ring is separated from the fill by an 8.61 / 10.90 gap. Passes. The halo is what makes this work, so it must stay mandatory (DESIGN Components, Focus halo says so; keep it).
- Light-theme ring vs the code-block surface is only 2.24 (dark theme 11.74). It passes because the ring is drawn outside the `pre` over the halo and card surface. If anyone changes the `pre` focus to a negative offset or inside the dark block, light mode fails 3:1.
- Seg control (mock line 22): ring extent is 2px halo + 2px gap + 3px stroke = 5px, but the group gap is 4px and padding 3px, so the focused ring overlaps the neighbouring button by 1px, and the pressed neighbour is the same hue as the ring. Still readable (the halo separates), so Low (L3).

## 2. Findings

### Major

- **MA1. Tiles break reflow at 320 CSS px (1.4.10).** `mockups/mock-exchange-view.html:75` `.tiles{grid-template-columns:repeat(4,1fr)}` and the 2-column rule `repeat(2,1fr)` use `1fr` (min `auto`), and `.tv` has no wrapping rule. `legacy-oms.internal` in 16px mono is about 182px, but a 2-column tile at 320px has about 100px of content width. Unbreakable hostnames force the track wider and the page scrolls sideways at 400% zoom. It also bites at 4 columns between 1101 and 1279px: main is about 713px wide, so a tile has about 143px, narrower than that host. Real hosts are longer. Fix: `repeat(auto-fit,minmax(min(100%,11rem),1fr))` or `minmax(0,1fr)`, plus `overflow-wrap:anywhere` on `.tv`, and one column at 320 to 480px. Spec the same in DESIGN (summary-tile has tokens but no prose, see MO1) and in the EXPERIENCE row.
- **MA2. Page-level overflow risk at the stacked breakpoint.** Mock line 50 `.layout{grid-template-columns:1fr}` below 1024px (same `1fr` problem; the desktop rule at line 28 correctly uses `minmax(0,1fr)`). The `pre` request line (about 630px) can widen the single track, so the "own scroll region" promise in DESIGN Layout fails at 320px even though `pre` has `overflow:auto`. Also `.appbar` (line 16) has no `flex-wrap`, so brand + crumb + 4 nav items cannot fit 320px. Fix: `minmax(0,1fr)`, `flex-wrap:wrap` on `.appbar`. Verify in a browser at 320px; this is from CSS reading, not a rendered test.

### Moderate

- **MO1. Spine gap: summary-tile has tokens but no component prose or contrast rows.** DESIGN.md:206 defines the token block, but Components (lines 393 to 420) has no "Summary tile" entry and the Colors table has no tile pairs (muted label on surface 7.28 is listed generically; error value on surface 7.57 / 8.84 is not). EXPERIENCE.md:120 is the only behaviour source. Add: layout rule (MA1), responsive columns, wrapping, what the failed value looks like. Also say what the OK state shows.
- **MO2. Tile semantics are weak (1.3.1).** Mock line 82: `ul[aria-label]` > `li` > two sibling `span`s. (a) `list-style:none` removes list semantics in Safari/VoiceOver unless `role="list"` is added. (b) Label and value are not programmatically paired, so a screen reader reads "Exchange latency 142 ms" with no separator and, in browse mode by element, as unrelated text. Better: `<dl>` with `div` groups (`dt` label, `dd` value) inside a labelled region, or keep `li` and add `aria-labelledby`/visually hidden ": ". (c) The id-map tile mixes status text and a link in one value; wrap the link so the status word is the first thing read ("Missing key. Jump to ..."). (d) `ul` name "Exchange summary" is fine but add a visible or hidden heading level consistent with the page (`h2` siblings are panels; tiles sit between the verdict `h2` and the panel headings with no heading). Optional.
- **MO3. Link purpose and consistent identification (2.4.4, 3.2.4, 4.1.2).** Three controls do the same thing with three names: button "Go to the mapping row: id" (mock line 80), tile link "Jump to id → orderId" (line 82), panel link "Jump to id → orderId in the mapping" (line 93). (a) 3.2.4 Consistent Identification (AA): same function, different labels. Use one label, for example "Go to mapping row: id to orderId". (b) The verdict "Go to the mapping row" is a `button` that navigates; it should be a link (or the spec must say it moves focus in place). (c) The destination is on Map request (step 3), not on this page, and the link text does not say so; EXPERIENCE does not define the return path or where focus lands ("field-level jump", EXPERIENCE.md:190, 350). Specify: opens the step, focus on the row's select button, and the Exchange view result stays reachable. (d) `→` is read as "rightwards arrow"; put it in an `aria-hidden` span. Link text itself is distinguishable in context (inside the Id-map status item), so 2.4.4 passes; the issue is consistency and wording.
- **MO4. Colour-only/semantic mismatch on the Id-map tile.** Mock line 75 `.tile.bad .tv{color:var(--err)}` with `&#9888;` (a warning triangle). It has an icon and the words "Missing key", so 1.4.1 passes. But DESIGN Do/Don't (line 430) says never use error red for the id-map advisory, and `id-map-warning` tokens exist; the tile token set (DESIGN.md:214) only has `fail-foreground: error`. The icon says "warning", the colour says "error", and the verdict bar already treats it as the cause of a failure. Pick one meaning: if blocking-cause, use the error icon (&#10007;) and word "Failed"/"Missing key"; if advisory, use `id-map-warning`. Add the OK state with the same pattern (EXPERIENCE.md:120 only requires an icon for failure; success shows plain "OK").
- **MO5. Sticky share bar vs focus (2.4.11) unresolved by the palette change.** Mock line 53 `scroll-padding-bottom:140px` is a fixed number; at 2-column tiles, narrow widths and 200% zoom the bar wraps (3 buttons plus a sentence) well above 140px. Carry-over of Mo5 in review-accessibility.md, but the new tile link and "Go to mapping row" targets scroll into this zone. Derive padding from the bar height (a CSS variable set from `ResizeObserver`) or un-stick below a width as well as a height.

### Low

- **L1. Tile label casing drift.** Mock line 75 `.tile .tl{text-transform:uppercase}`; DESIGN.md:364 says the label token sets weight and tracking, not case, "rendered as written". Uppercase is also harder to read for the cognitively-load-sensitive audience and screen readers may spell short all-caps strings. Remove the transform (same for `th`, mock line 36).
- **L2. primary-container spec is internally inconsistent and unused.** DESIGN.md:332 assigns container to "the selected-row bar and focused stepper marker", but DESIGN.md:394 says the current step marker and bar are `primary` and "no outline", DESIGN.md:395 says the selected-row bar is `primary`, and the stepper token (line 121) is `primary`. The mock uses `primary` everywhere; `#1C6A94`/`#5CB8E8` appear nowhere in the mocks. "Focused stepper marker" is also ambiguous (focus is the ring). Either delete the container token or name its single use. Since it is 1.35 to 1.45 from primary it cannot signal a state difference.
- **L3. Seg control ring overlap.** Gap 4px < 5px ring extent (see section 1). Make the gap 6px or use `outline-offset:-3px` inside the group. DESIGN.md:401 says the 4px gap keeps rings from being clipped; the arithmetic says it only avoids clipping, not overlap.
- **L4. Help marker drift (`.q`).** Mock line 20 and 59 draw an outlined circle (primary border and glyph on transparent); DESIGN.md:412 says filled primary with on-primary glyph. Both pass (8.61 / 10.90, 24px). Align the mock or the spec.
- **L5. Coverage tile vs verdict can read as contradictory.** "Field coverage: 3 of 3 fields mapped" beside "Missing key" and a 404. Not a WCAG failure; for the plain-language audience add "mapped" meaning (a field mapped without the id transform still counts). Same for "Exchange latency" on a 404 (time to failure).
- **L6. Tile announcement policy not specified.** After a user-initiated Run, focus goes to the verdict heading (EXPERIENCE.md:144), so tiles are read in order and need no live region; good. For live re-runs in side-by-side mode, tile values change silently (carry-over of Mo8). One line in EXPERIENCE.md:120: tiles are not live regions; they are part of the verdict region that is announced once.

## 3. Passes worth keeping

- Palette swap is safe: every text pair at or above 5.06 (primary-container on info-bg) and every UI pair at or above 3.17 (dark input-border on info-bg, as before). Light on-primary on primary rose from 5.93 to 8.61; dark on-primary on primary 8.02.
- Ring and halo recipe is now stronger than before (ring 7.3 to 11.7 vs adjacent, not 5.1 to 5.9). The "primary equals ring equals selection bar" caveat (previous review b) now applies to the selected bar and the focused ring being one hue; shape (3px offset ring vs 4px inset bar) still separates them.
- Tiles: visible text label and text value, the failed state has icon plus words, not colour alone; tiles are read-only; hidden-not-zeroed before a run (EXPERIENCE.md:120) avoids a misleading "0 ms". Muted label 7.28 / 7.06 passes. The link is underlined by default, primary on surface 8.61 / 10.90.
- The tile link gets the same 3px ring plus halo (`a:focus-visible`, mock line 57) on a `surface` background, passing 2.4.7.

## 4. Suggested fix order

1. MA1 and MA2 (one CSS pass plus a 320px test).
2. MO3 (one name and one control type for the jump; define landing focus).
3. MO1 and MO2 (spine prose and tile markup).
4. MO4 (resolve warning vs error meaning).
5. L2 (retire or define primary-container).
