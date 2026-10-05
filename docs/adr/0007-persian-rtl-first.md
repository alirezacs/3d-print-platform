# ADR 0007 — Build Persian-First and RTL-First

**Status:** Accepted
**Date:** 2026-10-05

## Context
The launch product is for Persian-speaking users.

## Decision
Design and implement the application **Persian-first and RTL-first** from the beginning.

Multi-language support is not an MVP requirement, but the architecture should avoid unnecessarily blocking it later.

## Consequences
Critical customer flows must be designed/tested in RTL instead of treating RTL as a late styling patch.
