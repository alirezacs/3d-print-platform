# ADR 0008 — Use Docs-as-Code as the Project Source of Truth

**Status:** Accepted
**Date:** 2026-10-05

## Context
The project has many business, architecture, research, and implementation decisions and will use Codex during development.

## Decision
Keep canonical project documentation inside Git.

Main categories:

```text
discovery/
blueprint/
architecture/
adr/
research/
tasks/
```

## Rules
- meaningful product/architecture changes update relevant docs;
- ADRs preserve decision rationale;
- research documents resolve unknowns with evidence;
- Codex prompts should point to relevant repository documents.

## Consequences
Documentation becomes versioned and reviewable, but Definition of Done must prevent docs from drifting behind the code.
