# Review: Stack verification (ARCHITECTURE-SPINE.md)

Date: 2026-11-20. Method: Maven Central metadata/POMs (repo1.maven.org), npm registry, endoflife.date, one web search. Spine not edited.

## Verdict
CONDITIONAL PASS. Most pins are real and current-ish, but one committed value is wrong (Hibernate 7.3), the Boot patch is stale (4.1.2 vs 4.1.3, and 4.2.0 now exists), and the memlog shows most versions were taken from secondary sources ("likely", "third-party blog", "unverified") and never confirmed against official registries. Fixable in minutes.

## Stack table, row by row

| Name | Spine | Registry fact (2026-11-20) | Status |
| --- | --- | --- | --- |
| Java | 25 | Boot 4.1 supports 17-26 (endoflife.date). 25 is an LTS. | OK |
| Spring Boot | 4.1.2 | Maven Central metadata: 4.1.3 and 4.0.10 exist; `latest`/`release` = 4.2.0 and the 4.2.0 BOM is published. endoflife.date (stale) shows 4.1.2 as latest. 4.1 OSS EOL 2027-07-31. | STALE: patch behind (4.1.3). 4.2.0 GA exists, not considered. Memlog only says "4.1.x ... pin exact patch", source secondary. |
| Hibernate ORM (Boot-managed) | 7.3 | spring-boot-dependencies-4.1.2.pom manages `hibernate.version` = 7.4.13.Final. 7.3 is a prior cycle (7.3.13 latest). | WRONG. Memlog: "Hibernate 7.3.x ... matrix unverified". Never checked. |
| Liquibase | 5.0.4 | liquibase-core 5.0.4 is latest on Central; Boot 4.1.2 BOM sets liquibase.version = 5.0.4. | OK. New risk: POM license is FSL-1.1-ALv2 (Functional Source License), not Apache. Not noted anywhere in spine or memlog. Check against "open source project" goal. |
| springdoc (webmvc-api) | 3.2.0 | 3.2.0 is latest on Central. Its parent POM is spring-boot-starter-parent 4.1.2 (swagger-core 2.2.55). | OK, and compatibility with Boot 4.1.2 confirmed from the POM. Compat with 4.2.x is unknown (springdoc bumps Boot per minor). |
| ArchUnit | 1.5.0 | 1.5.1 is latest on Central. | Minor stale. |
| WireMock standalone | 3.13.x (4.x beta only) | org.wiremock:wiremock-standalone: last 3.x is 3.13.2; 4.0.0-beta.44 is newest, no 4.0.0 GA. | OK. Claim holds. Pin 3.13.2. Note 4.x is a long-running beta (44 betas), so the "beta only" statement is correct but revisit later. |
| Keycloak (demo) | 26.8.2 | keycloak-core latest 26.8.2 on Central. Image tag not checked. | OK (verify the quay.io image tag exists at scaffold). |
| PostgreSQL | operator-provided | n/a | OK but no min supported major stated (Testcontainers image and Helm need one). |
| Node.js | 22.12+ or 20.19+ | Vite 8.3.8 engines `^20.19.0 \|\| >=22.12.0`. Spectral CLI 6.17.1 engines `^22.0 \|\| >=24.0`. | CONFLICT: Node 20.19 satisfies Vite but not Spectral. Spec should say Node 22.12+ only. Also Node 20 is EOL (April 2026). |
| Vite | 8.3.x | npm latest = 8.3.8. | OK (memlog got it from a blog, number happens to be right). |
| react-aria-components | 1.22.x | npm latest 1.22.1; peer react ^19 accepted. | OK |
| React | 19.x, verify at scaffold | npm latest 19.3.1. | OK, can be pinned now. |
| Spectral CLI | 6.17.x | npm latest 6.17.1. | OK, but see Node conflict. |
| Maven | 3.9.x, verify | Not checked by me or memlog. | UNVERIFIED. Boot 4.1 plugin needs Maven 3.6.3+; fine, but state it. Maven 4 GA status not checked. |

## Compatibility claims

- Boot 4.1.2 with springdoc 3.2.0: CONFIRMED from Central (springdoc parent = spring-boot-starter-parent 4.1.2). Spine claim is correct; memlog recorded it only from secondary sources.
- Liquibase 5.0.4 with Boot 4.1.2: CONFIRMED (BOM-managed). Spine should say "Boot-managed" as it does for Hibernate rather than pinning independently, to avoid drift. Memlog also mentions liquibase-hibernate7; the spine uses `ddl-auto=none`, so that plugin is not needed and not listed. Fine.
- Hibernate: spine says 7.3, BOM says 7.4.13.Final. Any text or config that pins 7.3 will conflict with the BOM. Fix to "7.4.x (Boot-managed)" or drop the row. Spine's JPA rules (lazy associations, no entity graphs, JPQL) are version-agnostic, so no design impact.
- Java 25 with Boot 4.1: CONFIRMED supported (17-26). Virtual threads on 25: no known issue; the spec of pinning-free JDK 24+ behavior on `synchronized` is a plus (not verified against sources here, general knowledge).
- Vite 8 + React: Vite engines confirmed. React 19.3.1 is not tied to Vite. Vite 8 plugin compatibility (`@vitejs/plugin-react` version) not checked; add to scaffold checklist.
- WireMock 3.13 vs 4 beta: CONFIRMED (no 4.0.0 GA; 3.13.2 current).
- Boot 4.2.0 vs springdoc 3.2.0: unknown; one search found no statement. Not a defect while 4.1.x is chosen, but then the 4.1 line must stay pinned.

## Memlog assessment
Memlog entries marked as unverified or secondary: Boot 4.1.x (spring.io blog + secondary), Hibernate "7.3.x ... matrix unverified", springdoc 3.2.0 and Keycloak (no source), ArchUnit (no source), Vite ("third-party blog"), react-aria ("likely, npm snapshots"), React unconfirmed, Spectral ("socket.dev"), Maven none. The memlog itself says "reverify at scaffold". The spine then presents these in a clean table with no confidence marker, except React and Maven. The table reads as verified when it was not. Spine has no "verified on date / source" line.

## Named technologies other than the table
- Spring Modulith: correctly rejected, not in spine.
- Micrometer, OTLP/Prometheus, Spring Security, RestClient, Testcontainers, Playwright, axe-core: no versions, BOM-managed or unpinned; fine at this altitude. Playwright/axe-core/Testcontainers versions not listed and not verified.
- Memlog says "Kafka" in the stack context as an input, spine defers it. Fine.
- JSON Schema validator and OpenAPI request/response validator library: memlog says "validator lib TBD (must handle OpenAPI 3.x dialect + @type polymorphism)". AD-11 and AD-14 depend on it and the spine names none and records no decision. This is the largest unverified technology risk (library fit for TMF @type/discriminator, OpenAPI 3.0 vs 3.1 dialect), more than any version pin.
- JSON canonicalization (stable key order): Jackson is BOM-managed as 3.1.7 (Jackson 3, `tools.jackson` packages). Spine AD-2 says "Jackson-annotated web types". No mention that Boot 4 moved to Jackson 3; harmless but worth noting for builders.
- Keycloak as OIDC demo provider with PKCE and `issuer-uri`: fits.
- WireMock standalone as separate image: fits.

## Recommended fixes (for the spine owner)
1. Hibernate row: "7.4.x (Boot-managed, 7.4.13 in Boot 4.1.2)".
2. Boot: pin 4.1.3 (or consciously choose 4.2.0 after confirming springdoc and Liquibase support); record why.
3. Node: "22.12+" only (Spectral requires 22+).
4. Add a "verified <date> against Maven Central / npm" note and mark Maven and the validator library as open.
5. Decide and name the OpenAPI/JSON Schema validator library, and note the Liquibase FSL license implication.
6. ArchUnit 1.5.1; WireMock pin 3.13.2; React 19.3.1.
