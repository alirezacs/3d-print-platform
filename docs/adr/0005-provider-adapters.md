# ADR 0005 — Isolate External Providers Behind Internal Adapters

**Status:** Accepted
**Date:** 2026-10-05

## Context
The platform depends on OpenAI, 3D providers, ZarinPal, SMS, object storage, and possibly shipping APIs.

## Decision
All provider integrations must sit behind application-owned interfaces such as:

- `AIProvider`
- `ThreeDGenerationProvider`
- `PaymentProvider`
- `SmsProvider`
- `ObjectStorageProvider`
- `ShippingProvider`

Provider-specific payloads and statuses are translated into stable internal models.

## Consequences
### Positive
- less vendor lock-in;
- easier testing and provider replacement;
- provider terminology does not leak into the domain.

### Negative
- adapter code adds some implementation overhead.
