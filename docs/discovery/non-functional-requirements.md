# Non-Functional Requirements

**Phase:** 00 — Project Discovery & Product Definition  
**Task:** 00.05 — Define non-functional requirements  
**Status:** Draft complete — pending repository commit

This document defines the baseline non-functional requirements (NFRs) for the 3D Print Platform.

Functional requirements describe **what** the system does.  
Non-functional requirements describe **how well, how safely, and under what operational constraints** the system must do it.

These requirements are intentionally implementation-independent where possible.

---

## 1. Security Requirements

### NFR-SEC-001 — Authentication security

Customer authentication must use secure OTP practices.

At minimum:

- OTP values must expire.
- OTP values must be one-time use.
- OTP values must not be stored in plaintext where avoidable.
- OTP requests must have resend cooldowns.
- OTP verification attempts must be rate-limited.
- Abuse protections should consider both account/mobile number and network/IP context.

### NFR-SEC-002 — Staff authentication

Staff authentication must provide stronger protection than customer authentication.

The design must support:

- secure password hashing;
- login rate limiting;
- account/session revocation;
- future 2FA support;
- stronger policies for privileged accounts.

### NFR-SEC-003 — Authorization

All protected backend operations must enforce authorization.

UI hiding is not an authorization mechanism.

Authorization must support:

- permission-based access;
- resource ownership checks;
- assignment-based staff access where appropriate;
- least privilege.

### NFR-SEC-004 — Secrets management

Secrets must never be committed to Git.

This includes:

- database passwords;
- payment gateway credentials;
- SMS credentials;
- OpenAI credentials;
- 3D provider credentials;
- object-storage credentials;
- signing secrets.

Secrets must be supplied through environment/configuration systems appropriate to each environment.

### NFR-SEC-005 — File-upload security

All uploads must be validated.

Validation should include:

- allowed MIME/type;
- file size;
- file extension consistency where applicable;
- safe filename handling;
- image decoding/validation for image uploads;
- access control;
- storage isolation.

### NFR-SEC-006 — Private assets

Private customer and internal production assets must not be publicly accessible by default.

Private files should use controlled access such as signed URLs or authenticated application access.

### NFR-SEC-007 — Payment security

Payment callbacks must be verified server-side with the payment provider before any business state becomes paid.

Duplicate callbacks must not produce duplicate financial effects.

### NFR-SEC-008 — Input validation

All external API inputs must be validated.

Invalid input must fail safely with a consistent error format.

### NFR-SEC-009 — Abuse protection

The platform must support rate limiting and usage controls for high-cost/high-risk operations, including:

- OTP;
- login;
- file upload;
- AI calls;
- 3D generation;
- expensive background jobs.

### NFR-SEC-010 — Auditability

Sensitive administrative actions must create auditable records.

---

## 2. Reliability Requirements

### NFR-REL-001 — PostgreSQL as source of truth

Durable business state must be persisted in PostgreSQL.

Redis, queues, AI providers, and external APIs must not become the only source of critical business truth.

### NFR-REL-002 — Idempotent operations

Critical operations must be idempotent where duplicate execution is possible.

Examples:

- payment callbacks;
- payment verification;
- background jobs;
- provider webhooks;
- order-creation side effects;
- notification dispatch where duplication is harmful.

### NFR-REL-003 — Retry-safe background jobs

Background jobs must support controlled retries.

Retries must not corrupt business state or create duplicate artifacts.

### NFR-REL-004 — Failure visibility

Failed jobs must be visible to operators.

The system must make it possible to identify:

- failed job type;
- related business entity;
- retry count;
- failure reason;
- last attempt time.

### NFR-REL-005 — External provider failure isolation

Failure of OpenAI, the 3D provider, SMS provider, storage provider, or payment provider must not corrupt existing business data.

### NFR-REL-006 — Explicit state transitions

Operational workflows must use defined state transitions rather than arbitrary status field updates.

### NFR-REL-007 — Transactional integrity

Operations that must succeed or fail together should use database transactions.

Examples:

- checkout → order creation;
- payment application;
- balance updates;
- quote lock;
- critical state transitions with related records.

### NFR-REL-008 — No silent data loss

Important customer/business records must not be silently discarded when a provider or background process fails.

---

## 3. Performance Requirements

### NFR-PERF-001 — Responsive customer experience

Normal public and authenticated pages should feel responsive on typical mobile and desktop connections.

Performance should be measured rather than guessed.

### NFR-PERF-002 — Non-blocking expensive work

Expensive operations must not block normal HTTP requests unnecessarily.

Examples:

- AI generation;
- 3D generation;
- slicing;
- model validation;
- invoice generation;
- large file processing;
- bulk notifications.

These should use background processing where appropriate.

### NFR-PERF-003 — Pagination

Potentially large lists must support pagination or bounded result sets.

Examples:

- products;
- orders;
- AI sessions;
- messages;
- audit logs;
- payments;
- production jobs;
- personalization requests.

### NFR-PERF-004 — Database indexing

Frequently queried fields must be indexed based on actual query patterns.

Likely examples include:

- user/mobile lookup;
- order number;
- payment references;
- status fields;
- foreign keys;
- creation timestamps;
- product slug;
- customer ownership fields.

### NFR-PERF-005 — Efficient media delivery

Public images should be optimized for web delivery.

Large internal assets should not be transferred unless required.

### NFR-PERF-006 — AI concurrency control

The platform must be able to limit concurrent expensive AI/3D jobs to protect:

- cost;
- provider quotas;
- CPU/memory;
- queue stability.

---

## 4. Scalability Requirements

### NFR-SCALE-001 — Start simple

The initial architecture should optimize for development speed and operational simplicity.

A Modular Monolith is preferred over premature microservices.

### NFR-SCALE-002 — Clear module boundaries

Business modules must remain sufficiently separated so that high-load components may be extracted later if evidence justifies it.

### NFR-SCALE-003 — Stateless application services

Web/API application instances should remain stateless where practical.

Durable state must live in shared infrastructure such as PostgreSQL, object storage, or explicitly designed state services.

### NFR-SCALE-004 — Independent worker scaling

Background workers should be scalable independently from HTTP/API processes.

### NFR-SCALE-005 — Object storage

Binary assets must use object storage rather than relying on local container/application filesystem persistence.

### NFR-SCALE-006 — Provider abstraction

External providers must be integrated through abstractions/adapters to reduce vendor lock-in and allow future replacement.

### NFR-SCALE-007 — Evidence-based scaling

New infrastructure complexity must be introduced based on observed load, cost, reliability, or organizational needs.

---

## 5. Observability Requirements

### NFR-OBS-001 — Structured logging

Backend and worker logs must be structured.

Logs should support fields such as:

- timestamp;
- service/process;
- environment;
- request/correlation ID;
- user ID where safe;
- business entity ID;
- job ID;
- provider request ID;
- error information.

### NFR-OBS-002 — Correlation IDs

Requests and related background operations should be traceable through correlation/request identifiers where practical.

### NFR-OBS-003 — Secret and PII redaction

Logs must not expose:

- passwords;
- OTP values;
- tokens;
- provider API keys;
- full sensitive payment data;
- unnecessary personal data.

### NFR-OBS-004 — Error monitoring

Production errors from frontend, backend, and workers must be centrally observable.

### NFR-OBS-005 — Health checks

The platform must expose operational health/readiness information for core internal dependencies.

Likely checks include:

- API process;
- PostgreSQL;
- Redis;
- workers.

Health checks should avoid expensive external-provider calls unless specifically required.

### NFR-OBS-006 — Job visibility

Operators must be able to inspect failed, retried, and stalled background jobs.

### NFR-OBS-007 — Payment diagnostics

Payment flows must record enough non-sensitive diagnostic information to investigate failed or ambiguous transactions.

---

## 6. Backup and Recovery Requirements

### NFR-BACKUP-001 — Database backups

Production PostgreSQL must have a documented backup strategy.

### NFR-BACKUP-002 — Restore procedure

A backup is not considered sufficient unless a restore procedure is documented and periodically validated.

### NFR-BACKUP-003 — Object-storage durability

Production file storage must use a durability/backup approach appropriate for important customer and production assets.

### NFR-BACKUP-004 — Recovery priorities

The recovery plan should prioritize:

1. PostgreSQL business data;
2. payment/order state;
3. private customer and production assets;
4. AI/3D artifacts that cannot be regenerated safely;
5. configuration and deployment state.

### NFR-BACKUP-005 — No single local-disk dependency

Critical production data must not exist only on one application server/container disk.

---

## 7. RTL and Responsive UX Requirements

### NFR-UX-001 — Persian-first

The initial customer-facing platform language is Persian.

### NFR-UX-002 — RTL-first

Customer-facing layout must be designed and tested in RTL from the beginning.

RTL must not be treated as a final visual patch.

### NFR-UX-003 — Mobile usability

Critical customer workflows must be usable on common mobile screen sizes.

Critical workflows include:

- authentication;
- catalog browsing;
- product configuration;
- checkout;
- payment return;
- AI chat;
- file/image upload;
- Personalization request;
- order tracking.

### NFR-UX-004 — Desktop usability

Admin and operational interfaces must remain efficient on desktop/laptop layouts.

### NFR-UX-005 — Clear async states

Long-running operations must expose clear UI states such as:

- queued;
- processing;
- completed;
- failed;
- retry available.

The user must not be left unsure whether a request is still running.

### NFR-UX-006 — Error clarity

Customer-facing errors must be understandable and actionable without exposing internal technical details.

### NFR-UX-007 — Status consistency

The same business status must use consistent naming and visual treatment throughout the application.

---

## 8. Testing Requirements

### NFR-TEST-001 — Domain logic unit tests

Deterministic business logic must be covered by unit tests.

Priority areas:

- state transitions;
- pricing rules;
- balance calculations;
- permission rules;
- manufacturing rules;
- AI credit accounting.

### NFR-TEST-002 — Integration tests

Persistence and infrastructure boundaries should have integration tests.

Priority areas:

- database repositories;
- order creation;
- payment application;
- storage adapters;
- background jobs;
- audit logging.

### NFR-TEST-003 — External provider isolation

Automated tests must not depend on live external providers by default.

Mocks/fakes/test adapters should exist for:

- OpenAI;
- 3D provider;
- ZarinPal;
- SMS;
- storage.

### NFR-TEST-004 — End-to-end critical paths

The project must include E2E coverage for at least:

1. Ready Product purchase;
2. AI Design purchase;
3. Personalization service flow.

### NFR-TEST-005 — Payment edge cases

Payment tests must cover:

- successful callback;
- failed verification;
- duplicate callback;
- retry after failure;
- delayed callback;
- amount mismatch protection.

### NFR-TEST-006 — Authorization tests

Protected resources must have automated tests for:

- valid access;
- cross-customer access denial;
- missing permission;
- privileged staff access.

### NFR-TEST-007 — CI enforcement

Lint, type checks, tests, and production builds should run automatically in CI.

---

## 9. Maintainability Requirements

### NFR-MAINT-001 — Modular code

Code must be organized around clear application/domain modules.

### NFR-MAINT-002 — Provider isolation

Provider-specific SDK/API details must remain inside adapters/infrastructure code and not leak throughout business logic.

### NFR-MAINT-003 — Docs as Code

Architecture, business rules, discovery, ADRs, and relevant technical documentation must live in the repository and evolve with the code.

### NFR-MAINT-004 — Explicit migrations

Database schema changes must use controlled migrations.

Production schema changes must not rely on automatic destructive synchronization.

### NFR-MAINT-005 — Consistent conventions

The project should define and enforce conventions for:

- formatting;
- linting;
- naming;
- API errors;
- configuration;
- module boundaries;
- tests.

### NFR-MAINT-006 — No hidden critical rules

Critical business rules must not exist only inside:

- prompts;
- frontend code;
- informal operator knowledge;
- scattered conditionals.

They should be represented in documented application/domain logic.

---

## 10. Privacy and Data Handling Requirements

### NFR-PRIV-001 — Minimum necessary access

Staff should only access customer information required for their responsibilities.

### NFR-PRIV-002 — Private conversations/assets

AI design conversations, customer reference images, and Personalization images are private customer/business data unless explicitly designated otherwise.

### NFR-PRIV-003 — Internal/customer separation

Internal notes and customer-visible notes must be separable.

### NFR-PRIV-004 — Data deletion/retention policy

A formal retention/deletion policy is not yet finalized and must be defined before production launch where legally/operationally required.

---

## 11. Cost Control Requirements

### NFR-COST-001 — AI usage visibility

AI and 3D provider usage must be measurable per operation/user/session where practical.

### NFR-COST-002 — Expensive operation limits

Expensive operations must support quotas, credits, rate limits, or concurrency limits.

### NFR-COST-003 — Provider cost diagnostics

Provider metadata should allow the business to understand cost drivers and evaluate alternative providers later.

### NFR-COST-004 — Avoid unnecessary regeneration

The system should preserve generated assets and versions so the same expensive generation does not need to be repeated unnecessarily.

---

## 12. Availability Expectations

The project does not currently require enterprise-grade multi-region/high-availability infrastructure.

However:

- normal application restarts must not lose durable business data;
- worker restarts must not silently lose queued/durable work;
- temporary provider outages should be recoverable through retry/manual recovery;
- planned deployments should minimize unnecessary disruption.

Exact production SLA targets may be defined later after launch requirements and hosting choices are known.

---

## 13. Definition of Done Impact

A feature is not considered complete only because its happy-path functionality works.

Relevant non-functional requirements must also be considered.

Depending on the feature, Definition of Done may require:

- authorization;
- validation;
- error handling;
- audit logging;
- tests;
- observability;
- retry/idempotency;
- responsive/RTL UI;
- documentation updates;
- secure file handling.

---

## 14. Change Rule

If a future implementation intentionally violates or relaxes one of these requirements:

1. the affected NFR must be identified;
2. the reason must be documented;
3. relevant ADR/architecture documentation must be updated;
4. security/reliability consequences must be reviewed;
5. tests and operational documentation must be updated where applicable.

Non-functional requirements must not drift silently.
