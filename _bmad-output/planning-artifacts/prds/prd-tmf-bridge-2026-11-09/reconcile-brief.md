# Reconcile: brief.md vs prd.md

Excludes decisions already made (drift/GitOps out of scope, expression language to architecture, throughput/availability dropped, success measures deferred, CTK/competitor items dropped, 10-minute zero-to-hero, all FRs S1, OIDC).

## Gaps

1. **Built-to-leave / exit path lost.** Brief makes portable mappings and a stated exit path a feature, and calls retirability a success measure. PRD keeps FR-13/14 export but never states the purpose. Worse, FR-5 makes the runtime stateful with an id map in its own DB. Retiring the bridge then breaks partner-visible ids unless the id map is exportable. Does "whole configuration" (FR-13) include the id map? Add: exit-path requirement (export is portable, human-readable, includes or exposes id map) and the Vision should say "designed to be retired", not only "stable".
2. **Honest-conformance wording drift.** FR-8 is right, but Vision says "a TMF-conformant API can be exposed". That overclaims against FR-8. Also dropped: brief's caveat that a fourth API with zero code is a wedge test where polymorphism, v4/v5 differences, task operations and hub semantics may still need code. Vision/UJ promise "no code" without that honesty. Reword Vision to "schema-valid"; add caveat in Later or Out of Scope.
3. **Redaction undefined, and lineage not covered.** FR-15 says "sensitive data" with no rules (what fields, configurable?). FR-16/17 expose step results and lineage with real backend values through studio/API and are not stated to be redacted or access-controlled. Brief treats redacted logs as first-class. Define the redaction scope and whether lineage output is covered.
4. **Dry-run / test-call side effects unaddressed.** Brief lists dry run as an API capability; PRD FR-10/12 have "test a call" but say nothing on whether test calls hit the (mock) backend, or what happens for writes later. S1 is read-only so low risk, but the rule (test calls never mutate; stubbed backends for writes) belongs as a requirement or an explicit Later note, else Later saga work inherits an unstated contradiction.
5. **Dropped smaller items.** (a) Secondary audience (TM Forum community, vendors contributing mappings) and Apache-2.0/open-source positioning absent from PRD (disclaimer only). (b) 501 scope: FR-2 covers an unconfigured resource only; unspecified for an unconfigured operation on a configured API, and 501 vs 404 for a missing id. (c) Invalid config on import/apply: atomicity and rejection behaviour not stated though FR-3/13 depend on it. (d) Build-vs-buy / differentiation vs commercial templates and OpenSlice is gone from the PRD, leaving "competitive advantage" as an unsupported assertion.

## Tone
PRD Vision says "Telco-grade" and "competitive advantage"; brief is deliberately modest ("moat is focus, not technical", "unverified"). Soften to match.

## OK (no gap)
501 + no restart (FR-2/3), runtime validation (FR-7), lineage and overhead metrics (FR-17/18), API-first (FR-12), stateful decision (FR-5, addendum).
