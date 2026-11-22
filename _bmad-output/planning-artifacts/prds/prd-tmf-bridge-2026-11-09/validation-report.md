# Validation Report — PRD: TMF-Bridge (re-run)

- **PRD:** `_bmad-output/planning-artifacts/prds/prd-tmf-bridge-2026-11-09/prd.md`
- **Rubric:** `review-rubric.md`
- **Run at:** 2026-11-15
- **Grade:** Good (0 critical, 0 high, 25 medium, 15 low; no thin dimension)

## Overall verdict
Adequate, close to ready. All six prior highs are closed. Remaining risk: a long tail of mediums and a UX spine that lags FR-28, FR-34, FR-39. All seven dimensions adequate; strategic coherence and scope honesty improved.

## Prior highs: all resolved
Success measures (UJ-1..5), samples processed-not-stored, id-map population (FR-39), sensitive data (FR-19), `/productOrder/{id}`, REST+JSON.

## Medium findings (new or regressions)
- FR-28 "never stores" not reconciled with access logs, step results, lineage (FR-19/20/21). Auth and REST parity of the preview endpoint unstated.
- FR-19/23 omit the partner bearer token and `Authorization` headers. Field-path syntax and config home undefined. Share-bundle redaction not explicit.
- FR-2 `404` cannot tell an id-map miss from a backend miss. UJ-3/UJ-5 never show the id map screen or seeding.
- FR-39: no import format, no duplicate handling on add, no edit or delete.
- FR-27: mock fixtures undefined (e.g. `ORD-1` returns `404`) though UJ-1/4/5 depend on them.
- FR-34 secret picker vs FR-17 import: does import check the secret is provisioned?
- Success measures are demo scripts: no wedge stress test, no counter-measure.
- Carried: Draft and Live as one entity, import identity, overhead definition (NFR-1/FR-22 circular), FR-4 error table, FR-25 "configurable", Draft test path, autosave locking, TLS, TMF versions, no Glossary, embedded UX detail, open items only in addendum.
- Out of Scope "FR-14 to FR-18 and FR-38" omits FR-39.

## PRD vs UX conflicts (S1 only; FR-30/FR-34 authoritative)
- UX Pick API legacy upload and WSDL conflict with FR-34. Fix in UX.
- UX "Samples never reach the bridge until sub-step 3" contradicts FR-28. Fix in UX.
- UX has no id map surface for FR-39. Fix in UX.
- Share bundle contents differ across FR-30, UJ-5, UX. Fix in PRD.
- FR-29 closed rules cannot produce UX unreachable, secret-missing, overhead states. Fix in PRD.
- Apply summary diff: UX "changes since last version" vs FR-31 diff Later. Undecided.
- UX staleness (low): scope note `GET /productOrder`, FR range header, "(not in PRD)" on Flow 3, open-items list, Disclaimer missing from help footer.
- Resolved: zoom, secret-name catalogue, upload placement.

## Numbering
FR-1..39, NFR-1..6, UJ-1..5 contiguous; cross-references resolve.

## Reviewer files
- `review-rubric.md`
