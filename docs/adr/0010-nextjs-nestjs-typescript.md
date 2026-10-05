# ADR 0010 — Use Next.js + NestJS + TypeScript

**Status:** Accepted
**Date:** 2026-10-05

## Context
The platform needs SEO-friendly public pages, rich customer/admin UI, modular backend architecture, worker reuse, and efficient full-stack development.

## Decision
Use:
- **Next.js** for the frontend;
- **NestJS** for the backend/API;
- **TypeScript** across application layers.

The Worker may reuse backend/domain code.

## Important Boundary
CPU-heavy 3D tooling does not need to be implemented in TypeScript. Native/CLI tools may run in isolated worker/container processes behind stable internal interfaces.

## Consequences
The stack improves developer consistency and type sharing, while runtime validation and module-boundary discipline are still required.
