# ADR 0003 — Use Redis + BullMQ for Asynchronous Work

**Status:** Accepted
**Date:** 2026-10-05

## Context
AI, 3D generation, slicing, PDF generation, notifications, retries, and provider polling are slow or retryable operations.

## Decision
Use **Redis** for operational/transient infrastructure and **BullMQ** for background jobs.

Workers run independently from the API process.

## Redis May Be Used For
- BullMQ;
- OTP throttling;
- rate limiting;
- short-lived cache;
- distributed coordination where justified.

## Rules
- Redis is not canonical business storage;
- jobs reference durable PostgreSQL records;
- jobs must be idempotent or safely retryable;
- failed jobs must remain observable.

## Consequences
BullMQ provides a pragmatic Node.js queue system, but critical delivery still requires a Transactional Outbox.
