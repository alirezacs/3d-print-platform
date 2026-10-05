# ADR 0001 — Use a Modular Monolith

**Status:** Accepted
**Date:** 2026-10-05

## Context
The platform contains many domains, but the initial team is one developer and early-scale requirements do not justify distributed-system complexity.

## Decision
Use a **Modular Monolith** with explicit domain boundaries, public application interfaces, domain-owned repositories, and controlled cross-module dependencies.

API and Worker processes may reuse the same domain/application code while running separately.

## Consequences
### Positive
- simpler deployment and local development;
- easier transactions;
- faster delivery;
- future extraction remains possible.

### Negative
- boundaries require discipline;
- one codebase/database makes accidental coupling easy;
- no independent deployment per domain initially.

## Alternatives
Microservices were rejected for MVP because they add unnecessary network, deployment, tracing, and consistency complexity.

## Revisit When
Revisit only with evidence such as different scaling needs, independent team ownership, deployment isolation, or operational bottlenecks.
