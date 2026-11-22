---
title: "Product Brief: TMF-Bridge"
status: complete
created: 2026-11-02
updated: 2026-11-09
---

# Product Brief: TMF-Bridge

## Executive Summary

TMF-Bridge is an open-source, configuration-driven accelerator that lets integrators expose TM Forum Open APIs over legacy BSS/OSS backends without writing, rebuilding or redeploying code. Mappings and flows are configuration. They are authored in a Mapping Studio, exported as structured text, versioned in Git, and applied to any environment through a REST API. Changes need no restart.

It is a bridge designed to be retired. When operators and vendors expose TMF natively, they drop TMF-Bridge and keep what mattered: the mappings, which are portable, readable text.

The project starts as a personal Apache-2.0 effort. The aim is recognition in the TM Forum community and conversations with integrators and vendors who will build mappings for real systems with it.

## The Problem

Operators and partners increasingly expect TM Forum Open APIs. The systems behind them (catalogs, order managers, fault systems) are often fifteen years old, speak their own dialects, and will not be rewritten for years. Today, exposing TMF over them means a custom adapter per API and per backend. Each field change is a code, test, deploy and restart cycle owned by developers, not by the analysts who understand the data. When something goes wrong at runtime, it is hard to see where a TMF field came from or why a call was slow.

Few vendors hold TM Forum Open API certification (about 100 companies, per a secondary, undated source). We infer, without a source, that mapping effort and conformance cost are the main bottlenecks.

## The Solution

A runtime that serves whichever TMF APIs are configured, built from the official OpenAPI definitions. Each operation binds to a mapping and an orchestration flow. The runtime supplies the capabilities real TMF APIs demand: id mapping, saga compensation, retry and timeout, asynchronous correlation, and TMF extensions (`@type`, `@baseType`, `@schemaLocation`, polymorphism).

Three behaviors define the product:

- **Starts empty, answers on configuration.** An unconfigured resource returns `501` with a TMF Error. Once a configuration is imported, the same endpoint answers, with no restart.
- **Configuration as code.** Git is the recommended source of truth, a best practice rather than a hard rule. Analysts can author in the studio, export structured text, and push it to Git. A pipeline then applies it through the REST API with no manual step. What this buys: versioned configuration, a full audit trail, drift detection (is the running configuration what Git says?) and drift correction (re-apply Git). Direct edits in any environment remain possible, and become visible as drift.
- **Fully API-based.** Configuration, import, export, dry run and the studio itself all use the same REST APIs. Anything the GUI does, a pipeline can do.

Every request and response is validated at runtime against the official OpenAPI definition of the API.

## What Makes This Different

- **Configuration over code is the wedge.** The runtime is the easy part. The cost today is the change cycle around it, and the lack of visibility into what the adapter did.
- **Traceable by design.** Redacted access logs, the result of every mapping step, and field-level lineage (each TMF field traced to its backend source and transformation) are first-class. Performance is measured and reported with the same weight.
- **Vendor-neutral and legacy-aware.** Commercial platforms offer TMF templates tied to their own stack. The closest open-source project found, ETSI OpenSlice, implements TMF on its own domain model rather than mapping to arbitrary backends. No general open-source, config-driven TMF adapter turned up in research; a GitHub search should confirm.
- **Honest conformance.** We promise runtime schema validation against the spec. We do not claim "certified" or "CTK-conformant". The CTK is not open source, so public CI cannot run it.
- **Built to leave.** Portable mappings and a stated exit path are a feature, not an apology.

The moat is focus and speed of execution. It is not technical.

## Who This Serves

- **Telco integrators (primary).** Teams who must put TMF in front of legacy backends and want to cut mapping effort and keep full visibility of runtime behavior. Success: an analyst adds a backend field to a TMF attribute, tests it, and carries it through Git to production without developer help.
- **TM Forum community and vendors (secondary).** People who want a working reference for TMF over a non-native backend, and vendors who may contribute mappings for well-known COTS products.

The operator personas in the source input are simulated, used to shape realistic requirements.

## Success Criteria

- Open-sourced under Apache-2.0 and publicly visible in the TM Forum community.
- Contact initiated with integrators or vendors to build real mappings together. Target count to be set in the PRD.
- Demo moment 1: an unconfigured TMF620 returns `501`, then answers after configuration is applied without restart.
- Demo moment 2: a mapping change authored in the studio is exported, applied through the REST API, and visible in the next TMF620 response, with its field lineage shown.
- Every demo response validates against the official OpenAPI definition.
- Performance and traceability figures are observable for every demo call. Targets to be set in the PRD.

## Scope

TMF620, TMF622 and TMF642 serve as **test examples**. What matters is the set of capabilities they exercise, and the scope is framed by capability.

**First slice:** TMF620 read path with one mapping and a minimal studio, to prove the wedge. It includes id mapping, TMF extensions, configuration without restart, configuration export and import through the REST API with drift detection and correction, runtime OpenAPI validation, redacted access logs, mapping results and field lineage, and basic performance metrics. Backends are mocks.

**Capabilities planned after the first slice:** saga compensation, retry and timeout, asynchronous correlation (callbacks, events), notification hub with delivery guarantees, idempotency, query and filter delegation, multi-tenancy, and the full mapping lifecycle with history, diff and rollback. TMF622 and TMF642 are the test beds for these.

**Out:** changes to backends, business logic beyond translation and orchestration, any claim of TM Forum certification, and real backends in v1 (mocks only).

## Vision

If it works, TMF-Bridge is a shared, open library of mappings for common legacy and COTS systems, and an accelerator integrators reach for first. Its success is measured partly by how cleanly operators can retire it. The PRD must size the platform ambition; this brief does not.
