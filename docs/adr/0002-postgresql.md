# ADR 0002 — Use PostgreSQL as the Primary Source of Truth

**Status:** Accepted
**Date:** 2026-10-05

## Context
The platform is strongly relational and requires transactions, constraints, reporting, histories, and consistent commercial state.

## Decision
Use **PostgreSQL** as the primary durable database and canonical source of truth.

Redis, queues, object storage, and external providers must never be the only source of critical business state.

## Consequences
### Positive
- ACID transactions;
- relational constraints;
- mature indexing/backup ecosystem;
- good reporting support.

### Negative
- requires controlled migrations;
- logical module ownership must be enforced despite one physical database.

## Rules
- money uses integer Toman values;
- large binary files do not live in PostgreSQL by default;
- schema changes use explicit migrations.
