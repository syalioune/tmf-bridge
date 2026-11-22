# Google Stitch handoff: TMF-Bridge Studio

Paste everything between the two horizontal rules into Stitch (https://stitch.withgoogle.com). Choose "Web" (desktop) as the device.

---

## Product and user

Design **TMF-Bridge Studio**, a web mapping studio for telecom integration analysts. The studio lets an analyst map TMForum TMF622 (product ordering) API fields to a legacy backend's fields, test the mapping, and put a new version live without a developer.

The user is **Inès**, an integration analyst. She knows telco well but is not a developer. The product must feel really easy, calm, forgiving and always offer help. Register: professional, calm, approachable. A quiet help desk, not a cockpit, not a toy. Keep technical terms (payload, JSON path, transform, lineage, OpenAPI) and explain them with glossary terms and tips.

Platform: **desktop web, designed for 1280px wide minimum**; below 1024px the inspector stacks under the main content, and the layout still works down to 320px at high zoom (only the mapping table scrolls sideways, in its own region). English. Design every screen in **light and dark**.

## Visual identity ("Graphite and cool cyan", direction "Calm and guided")

Use only these values. No gradients, no shadows on cards or rows (depth comes from tone and 1px borders), one accent only.

**Light:** background #F2F3F5, surface #FFFFFF, surface-raised #F8F9FA, text #16191F, text-muted #505763, border #CDD1D8 (decorative separators only), input-border #7A828F (every control edge), primary #1C6A94, primary-hover #14557A, on-primary #FFFFFF, focus-ring #1C6A94, success #14663B on #E3F4EA, warning #7A4A00 on #FFF1CC, error #A31F1F on #FDECEC, info #1F5F99 on #E4EEF9, id-map-warning #7A4A00 on #FFF1CC.

**Dark:** background #0C0E12, surface #14171D, surface-raised #1C2028, text #E8EBF0, text-muted #9BA3B0, border #2E343F, input-border #6B7482, primary #5CB8E8, primary-hover #7CC8F0, on-primary #04141F, focus-ring #5CB8E8, success #6FD29A on #12301F, warning #F2BE5C on #3A2C0C, error #FF9A9A on #3E1818, info #8BBEF2 on #14283E, id-map-warning #F2BE5C on #3A2C0C.

**Type:** system-ui / Segoe UI for the interface; ui-monospace / Menlo / Consolas for field paths, JSON, curl, secret names. Display 26/600 (page titles), body 16/400, body-strong 16/600, small 14/400 (dense tables, inspector, tips), label 14/700 uppercase, letter-spacing 0.04em (column headers, badges), code 14 mono. Nothing essential below 14px, including table headers, captions, badges and footers.

**Shape and spacing:** radius 12px for rows, panels, inputs, tips, bars, buttons; 4px for keyboard-key and inline-code chips; fully round only for stepper items, status badges and the `?` marker. Spacing scale 4/8/12/16/24/32. Interactive targets are at least 24px. Table row padding 12px. Inspector is a persistent **340px** right column with a 16px gutter. Drop targets and warnings use a 2px dashed border so they never rely on colour.

**Components:**
- Stepper: wrapping row of pill items with 24px numbered markers on a surface bar. Current = info-bg pill, bold text, primary filled marker and a 3px primary bar along the bottom (no outline, so it never looks like a focus ring); done = success marker plus check icon, as a link; to do = text-muted. Step name always shown as text.
- Mapping table row: surface, 1px border, 12px radius. Columns: TMF field | transform | backend field; the first cell is a row header with the field name. Selected row has an info-bg fill, a 4px primary bar, a 2px primary border and the word "selected". Empty target has a dashed input-border. Id warning row has a dashed id-map-warning border, id-map-warning-bg fill, warning icon and a text badge.
- Status badge: pill, label type, icon plus word. Draft = surface-raised fill, text-muted text, input-border outline, hollow circle icon. Live = success on success-bg with a check icon. Also Error and Warning.
- Inspector: surface-raised, 1px border, 12px radius, 16px padding. Title, a definition list (labels text-muted, values body or code), then an inline tip and a `?` link.
- Inline tip: info-bg panel, info text, icon plus text, 12px padding, link to the manual section. Help `?` link: 24px round marker with a primary outline. Glossary term: a focusable button with a dotted primary underline (links use a solid underline) that opens an info-bg popover with the definition on hover and focus, dismissed with Esc.
- Mode switch: two toggle buttons in a group with inner padding so focus rings are never clipped.
- Banner: 12px radius, status border and tint, icon plus text (unfired preview, offline, session ended, stale version).
- Buttons: primary (primary fill) and secondary (transparent, primary text, 2px primary border). One primary per view.
- Focus: 3px focus-ring outline with a 2px offset on every focusable element, the gap filled with a surface-coloured halo. Disabled controls: surface-raised fill, text-muted text, border outline, with the reason shown as text. Invalid fields: error border plus an inline message.

## Screens to generate (each in light and dark)

All authenticated screens share a top bar in this order: logo/home, Backends, Help `?`, Dark mode toggle, account menu (Sign out only). Every screen, including Login and the Exchange view, ends with the same footer help block: Glossary, User manual, "Need a hand? Ask your integration lead."

1. **Login.** OIDC sign-in only. Brand, one primary "Sign in" button. No username or password fields.
2. **Mappings list (home).** Dense table with columns: Name, TMF API + operation, Backend, Status (Draft or Live badge). Alphabetical, no recency columns. The name is a link that opens the mapping; the row action is Export. List actions: New mapping (primary), Import, Export all. Show the table populated (for example TMF622 GET /productOrder, TMF620 productOffering read).
3. **Map request (workspace step 3 of 6).** Stepper across the top: Pick API (where the TMF API and operation are chosen), Declare backend (pick an existing backend, with an inline "Add a backend"), Map request, Map response, Test, Apply. Mode switch in the header: Guided steps / Side-by-side. The **mapping table** is the main content, with a few rows (one selected, one empty target, one **id-map warning row**: an id field mapped without the id-map transform, with the message and an "Add id-map transform" action). Field lists for TMF622 inputs and backend request fields sit below, with Connect buttons; one Connect button is bound to the selected row (shortcut Alt+C). Show **exactly one labelled link** (source, transform, target) for the selected row, as a thin primary line over the layout. The persistent **inspector** on the right shows the selected mapping (source, transform, target, lineage) plus a help tip and `?` link. A "Saved." indicator and a Back/Next footer.
4. **Exchange view (Test, sub-step 3).** A full-width **verdict bar** at top, in the failing state: error icon, bold headline, one sentence, short checks list. Below it, three panels: 1) Mapping result, 2) HTTP request to backend, 3) HTTP response + TMF622 payload. The failing panel (HTTP response, 404) is auto-expanded with a 2px error border; the others are collapsed with a one-line summary. A **sticky share bar** at the bottom (3px primary top border): Copy as curl, Copy as Postman, Download bundle, secret name in code style, and a note "Secrets are never included. Only the secret name is." The footer help block sits below it. Checks in the verdict list say Passed, Failed or Skipped in words. Also include a passing-verdict variant.
5. **Apply summary + confirmation.** Summary of what goes live: mapping name, operation, backend, secret name, and "This replaces the live version." Primary "Apply", secondary "Back to Test". Then the live confirmation state: "Version 2 is live. The next call uses it." with a link "Open Mappings list". A single calm confirmation, no celebration. Also include a blocked variant: error banner "Someone else changed this mapping. Review their changes before you apply." with "Review their changes" and a visibly disabled Apply with the reason as text.

## Critical layout rules

- **The table is the source of truth.** The wiring layer is only a visual overlay on top of it.
- **No lines at rest.** Select or focus a row to draw its single labelled link. Hover shows a faint preview. Warning links are dashed with an icon. A "Show all links" toggle is off by default; if more than 8 links exist, show a count instead.
- **Never draw a mesh or web of arrows between two tables.** Do not use one-line-per-field connector diagrams.
- Every drag action also has a visible keyboard alternative (the Connect button and picker). Shortcuts always use a modifier (Alt+C, Alt+H), never a bare key.
- The inspector stays visible on every workspace step.

## Voice (use these strings)

- "The backend couldn't find order ORD-1. Your request sent it as orderId without the id-map transform."
- "Request mapped. Nothing was sent to the backend."
- "Uses secret `crm-prod-readonly`. Ask your tech team to change the value."
- "Version 3 is live. The next call uses it."
- "Checks passed. The output matches your sample and the TMF622 schema."
- "Samples are saved in this browser only."
- Payload shown with explanation: "payload (the body of the message)".
- Error blocks have four parts: what happened, likely meaning, what to try (a button), who to ask.
- Buttons are verbs: Map request, Test, Apply. Never "Submit" or "OK". No exclamation marks, never blame the user.

## Accessibility constraints (WCAG 2.2 AA)

- Never signal state by colour alone: every status, warning, link and masked value carries icon plus text, or a dashed pattern.
- Warnings (id-map) are advisory: amber and dashed, never error red. Red is for real failures only.
- Use input-border (not border) for every control edge. Visible 3px focus ring everywhere.
- Disabled actions stay visible with the reason as text beside them.
- Masked values display as `••••` with the text "masked". Never show or prefill a secret value; show the secret name only.
- Dark mode meets the same contrast ratios as light.
- No hover-only help (glossary terms are focusable buttons), no auto-dismissing errors, no motion-heavy effects.
- Every button has a distinct name in context (for example "Edit id row", "Export Product orders lookup"); one h1 per screen; status badges and verdict checks always include a word, not just a glyph.

---

## After you run it

1. Generate the five screens, each in light and dark. Re-prompt for anything that draws arrow meshes, adds shadows or gradients, or uses a second accent colour.
2. Save outputs into `imports/` in this workspace (`_bmad-output/planning-artifacts/ux-designs/ux-tmf-bridge-2026-11-11/imports/`):
   - Stitch's exported `DESIGN.md` as `imports/DESIGN.stitch.md` (do not overwrite the spine `DESIGN.md`).
   - The HTML for each screen as `imports/<screen>-light.html` and `imports/<screen>-dark.html` (login, mappings-list, map-request, exchange-view, apply-summary).
3. Compare against the spines and the promoted `mockups/` and `wireframes/`. `DESIGN.md` and `EXPERIENCE.md` win on any conflict.
