# Addendum: TMF-Bridge PRD

## Deferred to architecture
- Mapping expression language and flow model (studio risk). User ruled it an architecture concern, 2026-11-11.
- Runtime is stateful with a database. What it stores beyond the id map (subscriptions, versions, idempotency) is for architecture to settle.

## Carried from inputs
- **Dry run versus real backends.** "No side effects" conflicts with write flows hitting real backends. Mechanism needed: stubbed backends in dry run. PRD: Later (safe publishing). S1 test calls are read-only (FR-13).
- **Adapter versus system of record.** The runtime keeps a database. Whether to copy a full ProductOrder is open for architecture.
- **Zero-code honesty (original AC14).** A fourth API with no Java change is a strong test of the wedge. Polymorphism, v4/v5 differences, task operations and hub semantics may still need code.
- **Build versus buy.** Stakeholders want one proven product, not a patchwork of generic tools. This is the user's inference, not evidence.
- **Scale-out and consistency.** The runtime is stateful, so instance interchangeability and no lost or duplicated orders are for architecture.
- **Stack context (inputs, not decisions).** Java 25, Spring Boot 4, Kubernetes with Helm, PostgreSQL, Kafka, OIDC, OpenTelemetry. No front-end standard.

## Studio update, 2026-11-15 (from UX spines)
- **Unfired preview vs dry-run.** Preview translates a sample and sends nothing. Dry-run is write-side with stubbed backends. Kept distinct so S1 never implies write safety (FR-28, Later).
- **Storage.** Backends and drafts in the runtime database (user choice). Config-as-code was rejected: autosave and entity-tag locking need a writable store, and it splits FR-14 REST parity. Exports include drafts (see below).
- **WSDL.** Later. Avoids SOAP parsing and protocol-fault scope in S1. UX copy must drop WSDL.
- **Verdict rules.** Closed list in S1, matching the closed transform list. Extension Later.
- **Real backends.** Out of Scope line kept (mocks only). Real shape equals mock, so declare and read-only call work, but only the mock is claimed and tested.
- **No-conformance placement.** Verdict wording and Disclaimer via studio help. UX note 7 asked the PRD owner to confirm; assumed, confirm.
- **id-map.** Transform is configuration (FR-5); map is runtime state (FR-6). Warning is non-blocking.
- **Secrets.** Names only in studio, export, exchange export (`$SECRET_VALUE`). Value provisioning is deployment, mechanism for architecture.
- **UX notes 3, 4, 9, 10 applied** on user's request: studio export/import, per-mapping export, import as Draft (FR-15, FR-17); WCAG/platform/i18n/dark mode (NFR-2); user manual (FR-37).
- **Exports include drafts** with status (user). Confirmed no-conformance placement: verdict wording plus Disclaimer via studio help (FR-9).
- **FR-37 content owner:** integration analyst role (user, 2026-11-15). Superseded 2026-11-20: the project team writes it as a story (user).

## Export and import update, 2026-11-15 (UX notes 3, 4, 11)
- **Split.** FR-15 stays the whole-configuration exit path. Per-mapping export is a separate S1 requirement (FR-38), so the exit path is not weakened by a partial export.
- **Per-mapping and id map (user).** Transform only, no entries. Entries are runtime state, may expose backend record ids, and only make sense whole. Rejected: entries for the mapping's ids (larger, leaks ids); reference-only (needs a dependency check on import).
- **Import never touches live (user).** Id map entries and backends import additively. A collision rejects the whole file (FR-17). Rejected: restore into empty runtime only (blocks merging environments); mappings only (breaks FR-15 round trip). Open for architecture: how a legitimate id-map merge or restore of a populated runtime is done, which S1 import does not cover.
- **Drafts (user).** Exports include drafts with status; FR-18 determinism covers them. Rejected: Live-only determinism (weakens diff workflows); excluding drafts (unapplied work lost on exit). Autosave changes stored state, so it changes the export.
- **Import status.** The file's status is ignored: everything imports as Draft. A Live mapping round trips through Draft and needs an Apply.
- **Secrets.** Names, never values, in every export. `$SECRET_VALUE` in curl, Postman and share bundles.

## Studio NFRs and docs update, 2026-11-15 (UX notes 9, 10)
- **NFR split.** NFR-2 became NFR-2 to NFR-6 so each is testable alone. NFR-1 untouched.
- **Docs owner.** Superseded 2026-11-20: the project team writes the docs as a story (user).
- **Dry-run.** Defined write-side only, Later. Unfired preview stays S1.

## 2026-11-16 decisions

- Id map: edit and delete in S1; clash means same resource and TMF id with a different backend id; identical entries are skipped. (Clash definition extended by FR-39 on 2026-11-20.)
- Share bundle samples are masked with the FR-19 field paths.
- Backend definition is decoupled from mappings: no mapping version or Apply on backend edit. Consequence: a backend edit changes live calls on their next call. Revisit if backend versioning is wanted (Later).
- Per-mapping export (FR-38) carries the Live version when one exists; a never-applied mapping exports its Draft. Whole-configuration export (FR-15) still carries both.
- Backend edits changing live calls on their next call confirmed by user: not a mapping concern.
- FR-29 per-sub-step applicability for unfired sub-steps goes to architecture.

## 2026-11-20 architecture amendments

- **Source.** Architecture spine (final 2026-11-20), section PRD Amendments. Seven spine amendments (FR-5, FR-16, FR-17, FR-20/21, FR-22, FR-32, FR-39) plus two PRD-side fills (FR-37 glossary term, NFR-1 percentile) are folded in.
- **map/filter (user).** Added because arrays and nested objects cannot be mapped with the original closed list. Still closed; no free-form expressions. Address syntax, expression tree and engine rules live in architecture.
- **Import over a Draft (user).** Replace or skip is chosen per import, not fixed. Rejected: always replace (silent loss of work), always reject (blocks re-import).
- **NFR-1.** p95 chosen; conditions are measurement conditions, not load targets. Sizing stays Later.
- **Rationale, other amendments.** FR-32 autosave answers stale too, so a lost-update cannot slip in between Apply attempts. FR-39 reverse clash keeps reverse lookup single-valued. FR-16 embedded credentials rejected so a secret cannot hide in a URL. FR-20 step results are test-only because production calls must stay lean. FR-22 `Server-Timing` lets the studio and callers read one value.

## 2026-11-20 review follow-ups

- **Id-map miss is `502` in both directions (user).** Architecture said forward miss `404`, reverse miss mapping error. User chose one rule: any miss is a mapping error, `502`. The architecture spine needs the same change (conflict logged, not yet applied there).
- **Backend re-import (user).** Identical backend skipped, same name with different definition is a clash, so re-importing a whole export into the same runtime works.
- **FR-32 wording.** "Review their changes" replaced by "Reload the latest version", since diff and history are Later.
- **Deferred.** Splitting compound FRs (17, 30, 34, 39) is left to epics and stories. Per-sub-step rule applicability, status table and mapping identity came from architecture AD-12 to AD-14.
- **Left in architecture.** Null versus absent, `op: expr` rejection, `schemaVersion`, secret-name regex.
