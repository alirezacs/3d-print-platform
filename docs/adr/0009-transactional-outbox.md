# ADR 0009 — Use Transactional Outbox for Critical Async Integration

**Status:** Accepted
**Date:** 2026-10-05

## Context
Critical workflows can fail if database state commits but queue publication is lost.

Example:

```text
Payment succeeds
→ database commits
→ app crashes before publishing production event
```

## Decision
Use a **Transactional Outbox** for critical asynchronous integration.

Business state and the outbox event are written in the same PostgreSQL transaction. A publisher later dispatches the event.

## Critical Examples
- `PaymentSucceeded`
- `OrderItemReadyForProduction`
- `Design3DGenerationRequested`
- `AIQuoteLocked`
- `BalancePaymentRequired`
- `ShipmentDispatched`

## Consequences
The design greatly reduces lost-event risk, but consumers still must be idempotent because duplicate delivery remains possible.
