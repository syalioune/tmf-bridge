# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status

Planning-only repo. **No application code, build files, or tests yet** (no commits either). No build/lint/test commands exist. Add them here once the Maven + studio scaffolding lands (epics.md Epic 1 / Epic 8).

Contents:
- `docs/Project Brief — TMForum Open API Adapter.md` — original customer brief (TMF620/622/642 adapter, 2026-10-21).
- `_bmad/` — BMad method install (`config.toml`: agents, output folders). Tooling, not product.
- `_bmad-output/` — BMad planning artifacts. Source of truth for what to build.

## Where truth lives (read in this order)

1. `_bmad-output/specs/spec-tmf-bridge/SPEC.md` — canonical contract for slice **S1**. Says: the architecture spine, PRD, EXPERIENCE.md and DESIGN.md (its `companions:`) are the complete contract. On conflict: PRD wins on requirements, spine wins on architecture.
2. `_bmad-output/planning-artifacts/architecture/architecture-tmf-bridge-2026-11-16/ARCHITECTURE-SPINE.md` — binding AD-1..AD-17 rules, Stack table (pinned versions), Structural Seed. `solution-design.md` is narrative.
3. `_bmad-output/planning-artifacts/prds/prd-tmf-bridge-2026-11-09/prd.md` — FR-1..FR-39, NFR-1..NFR-6.
4. `_bmad-output/planning-artifacts/ux-designs/ux-tmf-bridge-2026-11-11/{EXPERIENCE,DESIGN}.md` — studio behavior and visuals.
5. `_bmad-output/planning-artifacts/epics.md` (8 epics) and `_bmad-output/implementation-artifacts/sprint-status.yaml` (all epics `backlog`) — work breakdown and tracking.

Note: the brief's scope (saga flows, Kafka, SOAP, hub, TMF642) is broader than S1. S1 is TMF620/TMF622 over REST/JSON with a synchronous request-table → backend call → response-table pipeline; no retries, no resource state beyond config + id map (AD-15). Follow SPEC/spine over the brief.

## Architecture (big picture)

- **Interpreter runtime, hexagonal.** One generic engine evaluates config stored in Postgres. TMF OpenAPI specs, mappings, backends are data. No class/route/bean may be named after a TMF resource (AD-1, ArchUnit-enforced).
- Single Maven module `tmf-bridge-api`, package-by-layer under `com.tmfbridge`: `domain` (pure; only Jackson `JsonNode` allowed as third-party) / `application/{runtime,control}` / `adapter/in` / `adapter/out` / `bootstrap`. `application/runtime` and `application/control` never import each other (AD-2, AD-3).
- **Data plane** (serve TMF calls via one catch-all handler, reads *published* versions only) vs **control plane** (studio API, drafts, import/export, test, preview; beans gated by Spring profile `control`). Only control instances run Flyway.
- Studio tests and production calls share one domain `CallPipeline` (AD-5). Test mode wraps the backend port to allow GET only. `POST /preview` is stateless and writes nothing.
- Mapping = canonical JSON doc: request rule table + response rule table, rules are `{target, expr}`, expr is a closed node tree (`path, lit, concat, coalesce, if, lookup, convert, idMap, map, filter`), all addresses JSON Pointer (AD-6). Numbers are `BigDecimal`, never `double`.
- Applied MappingVersions/ApiSpecs are immutable (DB trigger). ETag = Mapping `lockRevision`; `If-Match` on autosave and Apply, stale → `412` (AD-7).
- Export/import go only through the domain canonicalizer; export→import→export must be byte-identical; imports are atomic and land as Draft (AD-8).
- Secrets stored by name only, resolved from env `TMFBRIDGE_SECRET_<NAME>` or file; all output passes one `Redactor` (AD-10).
- Fixed error table: invalid request 400, backend 404 → 404, other 4xx/5xx/transport/invalid response/mapping error (incl. id-map miss) → 502, timeout → 504. Fail closed, no internal detail in TMF Error bodies (AD-11).
- Three images: `tmf-bridge-api`, `studio-frontend` (React 19 + TS + Vite, React Aria Components; calls only the public config API), `tmf-mock` (WireMock). One immutable image per env (AD-4, AD-16).

## Constraints that affect any copy/docs you write

- Never say "conformant", "compliant" or "certified" in studio copy, API descriptions or exports (AD-17). Fixed pass-verdict text is in the spine.
- Stack is pinned in the spine's Stack table (Java 25, Spring Boot 4.1.x, Postgres, Node 22+). Hibernate/Flyway/Jackson versions come from the Boot BOM, never pinned by hand.

## Working with BMad

Planning/build workflows run via `bmad-method:*` and `bmad-toolbox:*` skills (enabled in `.claude/settings.local.json`). Each artifact folder has a `.memlog.md` of decisions; check it before revisiting a settled decision.
