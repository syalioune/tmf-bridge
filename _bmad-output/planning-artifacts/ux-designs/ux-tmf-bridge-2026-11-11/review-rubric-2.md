# Spine Pair Review (round 2, post-Stitch update) — TMF-Bridge Studio

Scope: 2026-11-15 Stitch update (primary swap, primary-container, summary-tile, Summary tiles pattern, imports/stitch refs). Spines not edited. Contrast re-computed in Python from frontmatter hex.

## Overall verdict
The update landed cleanly where it is mechanical: every `{token}` ref in DESIGN.md components and EXPERIENCE.md resolves, the recomputed contrast table matches my numbers for every row, and all seven mocks carry the new primary/hover/focus hex in both themes. The weak spots are the new pieces themselves: `primary-container` is defined and described but no component uses it, the summary-tile spec is thinner than its neighbours (no ok state, no failed-run or n/a states, no responsive rule), and the mock contradicts DESIGN.md on label letter case. 0 critical, 2 high, 9 medium, 11 low.

## 1. Flow coverage — strong
Not changed by the update. Flow 1 step for Test/Exchange does not mention tiles, which is acceptable (the tiles are read-only). No new misses.
### Findings
- **low** Flow 1 / Flow 3 never mention the Id-map status tile, although Flow 3 (404) and the id-map warning story are where it matters most (EXPERIENCE.md:312-358). *Fix:* optionally note the tile link in the id-map failure beat.

## 2. Token completeness — adequate
All 40+ colors have hex (light and `-dark` pairs, except intentionally single-valued `code-surface`). Every component token ref resolves (checked colors, spacing, rounded, typography). Swap values match imports/stitch/DESIGN.light.md and DESIGN.dark.md (#005176/#1c6a94, #85d3ff/#5cb8e8, on-primary-dark #00344a).
### Findings
- **high** `primary-container` and `primary-container-dark` are orphans: defined (DESIGN.md:20, 43) and described as used for "selected-row bar and focused stepper marker" (DESIGN.md:332), but no component token references them. The selected row bar and stepper marker use `{colors.primary}` (DESIGN.md:119, 130, 164) and the component prose still says "4px `primary` bar" (DESIGN.md:395) and "`primary` filled marker" (DESIGN.md:394). A consumer cannot tell which colour the bar is. *Fix:* either wire `selected-bar` / `current-marker-background` to `{colors.primary-container}` and update lines 394-395, or drop the tokens and the sentence at 332.
- **medium** `primary-container` has no stated contrast row and no on-container pair. Recomputed: container on surface 5.93 light / 8.10 dark, on info-bg 5.06 / 6.76, on background 5.35 light, `on-primary` on container 5.93. All pass 3:1 for large accents, but the contrast table (DESIGN.md:343-353) omits them, and Stitch's `on-primary-container` (#c6e5ff / #004762) was not carried over. *Fix:* add one table row (primary-container on surface / info-bg) so the "never for text" rule is justified (it would pass 4.5 on surface, so the rule is a design choice, say so).
- **medium** summary-tile (DESIGN.md:206-214) defines only `fail-foreground`. No ok/success value colour, no failed-tile background or border, no label case. Mock `.tile.bad` only colours the value text. Other status components (verdict-bar, status-badge, banner) define bg + fg pairs. *Fix:* add `ok-foreground` (or state "ok uses `text`") and state fail has no tint by design.
- **low** Contrast row "text on code-surface" is absent for the exchange-panel code block (`text-dark` on `code-surface` = 16.16 computed). Pre-existing; not introduced by update.

## 3. Component coverage — adequate
Summary tile has a DESIGN row? **No.** The DESIGN.md frontmatter has `summary-tile`, but the Components prose list (DESIGN.md:393-420) has no "Summary tile" bullet, while EXPERIENCE.md has a "Summary tiles" row (EXPERIENCE.md:120). Name is also singular in DESIGN vs plural in EXPERIENCE.
### Findings
- **high** DESIGN.md Components prose lacks a Summary tile entry (between Exchange panel DESIGN.md:406 and Share bar :407). Rubric requires a row in both spines. Token block exists but no visual statement (grid of four, label above value, label 14/700, value 16/600, host rendered in code type as in the mock, failed value icon plus word). *Fix:* add a bullet; reference [Exchange view mock](mockups/mock-exchange-view.html).
- **low** Name mismatch: `summary-tile` (DESIGN token) / "Summary tiles" (EXPERIENCE pattern) / "summary tiles" (IA row EXPERIENCE.md:52, lowercase). Pick one canonical component name ("Summary tile" singular, with the group described in the row).
- **low** Mock renders Target endpoint in mono (`.mono`); neither spine says endpoint host uses `code` type. State it.

## 4. State coverage — thin (for the new component)
Summary tiles pattern covers "Before a run, tiles are hidden, not zeroed" (EXPERIENCE.md:120). Exchange view only exists after a run, so this case is largely moot, and the real gaps are elsewhere.
### Findings
- **medium** No states for tiles on a failed or aborted call: backend 404, timeout, cancelled "Calling backend…" (EXPERIENCE.md:178-193). Latency may be absent, field coverage may not be computable. Flow 3 (404) is a named flow. *Fix:* one row in Test and Exchange: tile value "Not available" plus reason for each missing metric, never "0 ms".
- **medium** Id-map status tile: only "OK" and "Missing key" are defined. Missing: no id field in this operation (not applicable), id mapped with the wrong transform (id-map warning is non-blocking, amber per spine, but the tile fail style is red `error`; the Stitch red/amber conflict that the spine resolved at the warning row is reintroduced on the tile). *Fix:* "Missing key" should use `id-map-warning` colour or be defined as advisory, or explain why red here.
- **low** Latency tile vs existing "Verdict pass" copy already reports overhead in ms (FR-22) and the 100 ms over-budget note (EXPERIENCE.md:187-188). Does the tile show end-to-end exchange latency or bridge overhead? "Exchange latency" is ambiguous against NFR-1 "overhead". *Fix:* define it in one clause.

## 5. Visual reference coverage — adequate
Mocks: all 7 linked in EXPERIENCE.md:83. mock-exchange-view now renders 4 tiles (mockups/mock-exchange-view.html:75, 82); primary/hover/focus hex match in all 7 mocks, light and dark. imports/stitch linked as a folder.
### Findings
- **medium** DESIGN.md visual-reference list (DESIGN.md:322) links only 5 mocks; mock-backends and mock-help-panel are missing there. EXPERIENCE.md links all 7. Also imports are linked only as a folder (DESIGN.md:323, EXPERIENCE.md:83), so no per-surface file naming what it illustrates; the "known deviations" list covers this adequately in prose. *Fix:* add the two mocks to DESIGN.md:322.
- **low** Neither spine's `sources` frontmatter lists imports/stitch/DESIGN.light.md / DESIGN.dark.md, though both are cited as the source of the palette (DESIGN.md:7-10, EXPERIENCE.md:6-21). *Fix:* add them.
- **low** "Spines win on conflict" is stated twice: DESIGN.md:321 and EXPERIENCE.md:83 (which says "this is the only place that rule is stated").
- **low** Wireframes (flow-map-request, ia) predate the tiles and do not show them. EXPERIENCE says table rows are placeholders; add that the Exchange view surface is not wireframed, or ignore (mock covers it).

## 6. Bloat & overspecification — adequate
### Findings
- **medium** Change-history narration inside the spine: "(from the Stitch export)", "(user adopted the Stitch palette on 2026-11-15, primary `#005176` light, `#85D3FF` dark)" (DESIGN.md:332, 335), "From the Stitch import (user kept on 2026-11-15)" (EXPERIENCE.md:120). Decision provenance belongs in `.memlog.md`. Colors section also states "Hex values live in the frontmatter only" (DESIGN.md:328) and then repeats hexes at 323, 332, 335. *Fix:* delete the provenance clauses and the repeated hexes.
- **low** DESIGN.md:332 is a run-on: the primary-container sentence is spliced into the middle of the primary bullet, ending "Primary action, selected row, links, the `?` help marker. Not used for status." as an orphan fragment. Split into two bullets.

## 7. Inheritance discipline — adequate
### Findings
- **medium** Stale terminology after the palette swap. DESIGN.md:4 description still says "one cool cyan accent"; #005176 is a deep blue-teal. EXPERIENCE.md:30 says theme "Graphite and cool cyan". DESIGN.md:336 refers to "theme 5" (rendered in .working/color-themes-1.html, which is not in sources or linked) and ".memlog.md:70" still records the old cyan focus ring. *Fix:* reword to "one deep teal-blue accent" (or "cool blue"); replace "theme 5" with plain statement.
- **low** Glossary check: "Summary tiles" / "Id-map status" / "Missing key" copy appears in EXPERIENCE.md:120 only; the mock says "Missing key" with "Jump to id -> orderId". Consistent.
- Token refs from EXPERIENCE.md: `{colors.error}` (:144), `{colors.focus-ring}` (:220, :292), `{colors.input-border}` (:292); all resolve.

## 8. Shape fit — strong
DESIGN.md section order canonical; EXPERIENCE.md has all required defaults. Responsive present.
### Findings
- **medium** Responsive for tiles unspecified in spines. The mock collapses to 2 columns at max-width 1100px (mock-exchange-view.html:75), a breakpoint the spines never define (DESIGN breakpoints are 1280 and 1024, DESIGN.md:103-104, 375). At 200%/400% zoom (reflow rule) tile behavior is undefined. *Fix:* state "four across; wraps to 2 then 1 column under `breakpoint-stack`" and align the mock to 1024px.
- **medium** Mock vs spine: mock label has `text-transform:uppercase` (mock-exchange-view.html:75), but DESIGN.md:364 says the label token "sets weight and tracking, not letter case, so text is rendered as written". Spines win, but the mock is the only visual of tiles. *Fix:* remove uppercase in the mock (or state case rule in spine).
- **low** Accessibility structure of tiles not in the pattern: the mock uses `<ul aria-label="Exchange summary">`; the spine row does not say list semantics, a group name, or where the tiles sit in the skip-link and reading order (after verdict bar). Failed Id-map tile carries a link inside a read-only tile: say it is the only tab stop.

## Mechanical notes
- All `{path.to.token}` refs resolve; no dangling refs; no missing hex. Tokens defined but unreferenced anywhere: `primary-container`, `primary-container-dark`, `warning` / `warning-dark` are used only by status-badge (ok), `breakpoint-*` used in prose only (acceptable).
- Contrast table values in DESIGN.md:343-353 all reproduce within 0.01 (primary/surface 8.61, primary/info-bg 7.34, on-primary/hover 11.59, dark 10.90 / 9.1 / 8.02 / 9.19, focus-ring dark on background 11.74).
- Mock `--code:#0F1319` vs `code-surface` `#0C0E12` (DESIGN.md:32): pre-existing mismatch (mocks/*.html line 5-7). Mock dark-theme variables also use own short names, fine.
- Mock palette scope: sed-updated hex confirmed for primary, hover, onp, focus in all 7 mocks; primary-container not present in any mock (consistent with it being unused).
- Memlog is internally consistent with spines on the update (decisions at .memlog.md:93-98); last entry :101 says the review-accessibility-2.md is in flight.
- No Mermaid in spines.

## Counts
critical 0, high 2, medium 9, low 11 (the two highs are the ones to fix first: orphan primary-container tokens, and no Summary tile bullet in DESIGN.md Components).
