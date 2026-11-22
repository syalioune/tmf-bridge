# Reconcile: brief addendum.md vs PRD (prd.md + addendum.md)

Excludes already-decided items (see reconcile-brief.md).

## Gaps

1. **Open challenges not handed to architecture.** Brief addendum raises dry-run "no side effects" vs write flows (stub backends in dry run) and stateless vs durable state, including the adapter-vs-system-of-record question (copy full ProductOrder or not). PRD addendum records only "stateful, architecture decides what else is stored". Dry-run and the ProductOrder-copy decision are missing from "Deferred to architecture", so they can fall through when Later writes arrive.
2. **Build-vs-buy rationale dropped.** Brief addendum flags "one proven product, not a patchwork of generic tools" as an uncited user inference. PRD has neither the rationale nor an explicit statement that it was dropped. Record as architecture/positioning assumption (unverified), or mark it intentionally dropped.
3. **Honest framing on zero-code claim and unverified facts lost.** The input AC14 note (polymorphism, v4/v5, task ops, hub may still need code) is absent. PRD addendum also drops the stack context (Java 25, Spring Boot 4, K8s/Helm, Postgres, Kafka, OIDC, OTel) that architecture needs as inputs, and the trademark point survives only as the Disclaimer. Add a stack-context list and the AC14 caveat to the PRD addendum.
4. **Lifecycle reshaping unresolved.** Brief addendum says the PRD decides which lifecycle features (FR-19 draft/publish, rollback via Git revert plus re-apply) stay in the studio. PRD pushes history/diff/rollback to Later without saying how rollback works in S1 (re-import of prior export?) or whether apply is atomic/validated. Also FR-22 mock backend in S1 is fine, but input FR re-tiering (21 of 24 Must) has no traceability table from source FRs to PRD FRs, so dropped source FRs are not visible.
5. **Tone.** Brief addendum is evidence-labelled (unverified, inference). PRD addendum is terse with no unverified markers; when carrying claims forward keep the "inference / unverified" labels.

## OK
Re-tiering by capability, FR-23 export promoted, FR-22 lineage core, API-based everything, NFR hypotheses dropped, expression language deferred.
