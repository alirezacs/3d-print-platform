# Open Research Register

**Phase:** 00 — Project Discovery & Product Definition  
**Task:** 00.06 — Create open-research register  
**Status:** Active  
**Purpose:** Track unresolved decisions, required investigations, dependencies, and research outcomes.

This document is the canonical register for project topics that must be investigated before the related implementation is finalized.

A topic remains **Open** until it has a documented conclusion in `research/` and, when architectural, an ADR.

---

## Status Definitions

### Open
Research has not started or is incomplete.

### In Progress
Research is actively being performed.

### Blocked
Research depends on unavailable production data, team input, credentials, budget, or another unresolved decision.

### Decision Ready
Enough evidence exists to make a decision, but the result has not yet been formally recorded.

### Resolved
The research result is documented and any required ADR/business-rule updates are complete.

---

## Priority Definitions

### Critical
Blocks or materially changes a core MVP workflow.

### High
Should be resolved before implementation of the dependent module.

### Medium
Can be deferred until the relevant implementation phase.

### Low
Does not materially block MVP architecture or early implementation.

---

# Research Register

## R-001 — Manufacturing Technologies and Actual Production Workflow

**Status:** Open  
**Priority:** Critical  
**Target Phase:** Phase 09  
**Primary Dependency:** Production team input

### Questions

- What 3D-printing technologies are currently used?
- Which production processes are manual vs machine-based?
- What is the actual step-by-step workflow from file to finished product?
- What post-processing steps are common?
- Which steps are mandatory vs product-specific?
- What are the most common production failure cases?
- When does rework/reprint happen?
- What does QC currently check?
- Which production information must operators see?

### Required Inputs

- Interview with production team.
- Examples of real completed jobs.
- Existing internal checklists/process notes if available.

### Expected Output

`research/manufacturing-workflow.md`

### Resolution Criteria

- Current production workflow is documented.
- Common failure/rework paths are documented.
- Required production states are identified.
- Required operator/QC data is identified.

---

## R-002 — Printer Models and Capabilities

**Status:** Open  
**Priority:** Critical  
**Target Phase:** Phase 09  
**Depends On:** R-001

### Questions

For every production printer:

- manufacturer/model;
- printing technology;
- build volume;
- supported materials;
- nozzle/resolution capabilities;
- practical layer-height range;
- known reliability constraints;
- preferred use cases;
- maintenance/unavailable states;
- any profile-specific limitations.

### Expected Output

`research/printer-capabilities.md`

### Resolution Criteria

Enough data exists to model `Printer`, `PrinterProfile`, and deterministic capability rules.

---

## R-003 — Materials, Colors, and Manufacturing Compatibility

**Status:** Open  
**Priority:** Critical  
**Target Phase:** Phase 09  
**Depends On:** R-001, R-002

### Questions

- Which materials are actually supported?
- Which colors exist for each material?
- Which printer/material combinations are valid?
- Are there unavailable/temporary materials?
- Are there customer-facing material names different from internal names?
- What cost data is available?
- How often do material prices change?
- What special constraints exist per material?

### Expected Output

`research/materials-and-colors.md`

### Resolution Criteria

- Supported materials/colors are known.
- Compatibility relationships are documented.
- Cost-update needs are understood.

---

## R-004 — Deterministic Manufacturing Constraints

**Status:** Open  
**Priority:** Critical  
**Target Phase:** Phase 09 / Phase 13  
**Depends On:** R-001, R-002, R-003

### Questions

Which constraints can be expressed deterministically?

Examples:

- maximum dimensions;
- minimum practical dimensions;
- minimum wall thickness;
- minimum feature size;
- tolerances;
- unsupported geometry;
- printer/material incompatibility;
- unsupported orientation/overhang thresholds;
- mandatory supports;
- product-specific restrictions.

### Expected Output

`research/manufacturing-constraints.md`

### Resolution Criteria

- Hard rules are separated from human judgment.
- Rules suitable for the Manufacturing Rules Engine are identified.
- Rules that remain advisory/manual are clearly labeled.

---

## R-005 — Existing Pricing Process and Telegram Bot Formula

**Status:** Open  
**Priority:** Critical  
**Target Phase:** Phase 13  
**Primary Dependency:** Production/business team input

### Questions

- What exact formula does the current Telegram bot use?
- What inputs does it receive?
- Does it use volume, weight, print time, supports, material, or manual values?
- Where do rate values come from?
- Are there minimum charges?
- Are there quantity discounts?
- How are failures/reprints accounted for?
- How are post-processing and labor priced?
- Are prices different by printer/material?
- How accurate is the current estimate compared with actual production?

### Required Evidence

- Current bot logic/formula if accessible.
- Several historical order examples.
- Estimated vs actual production cost/time.

### Expected Output

`research/current-pricing-process.md`

### Resolution Criteria

The existing business pricing logic is understood well enough to compare it with a new Price Engine.

---

## R-006 — AI Order Price Estimation Model

**Status:** Blocked  
**Priority:** Critical  
**Target Phase:** Phase 13  
**Depends On:** R-001, R-003, R-005, R-010

### Important Constraint

The pricing formula is intentionally **not finalized during Discovery**.

The architecture should expose a pluggable Price Estimation Engine, but no arbitrary formula should be implemented before this research is complete.

### Questions

- Which slicer/production metrics correlate with actual cost?
- Should price use material weight, print duration, both, or additional metrics?
- How should machine-hour cost be calculated?
- How should labor and post-processing be represented?
- How should support material be priced?
- How should expected failure/reprint risk be included?
- How should minimum order charge work?
- What margin model is appropriate?
- Should price rules vary by printer/material/profile?
- How should pricing rules be versioned?
- How should Admin overrides work?
- How should actual vs estimated cost be measured after launch?

### External Market Research

Compare practical approaches used by Iranian 3D-printing businesses/calculators.

### Expected Output

`research/price-engine.md`

### Follow-up

If architectural decisions are made, create/update the relevant ADR.

### Resolution Criteria

- A versioned pricing model is documented.
- Inputs and formula are explicit.
- Example calculations using real jobs are included.
- Accuracy is acceptable to the business/production team.
- Admin override policy is documented.

---

## R-007 — 3D Generation Provider Benchmark

**Status:** Open  
**Priority:** Critical  
**Target Phase:** Phase 12

### Candidate Providers

Initial candidates include:

- Meshy
- Rodin / Hyper3D
- other viable provider discovered during research

### Evaluation Criteria

- Text-to-3D quality.
- Image-to-3D quality.
- Geometry quality.
- Printability.
- Mesh cleanliness.
- Output formats.
- Texture/material output if relevant.
- API quality.
- Async job support.
- Webhook/polling support.
- Reliability.
- Rate limits.
- Latency.
- Cost.
- Commercial usage rights.
- Provider lock-in risk.
- Ability to retrieve raw 3D files.
- Suitability for Persian-input workflow after prompt normalization.

### Test Set

Create a fixed set of representative product requests to benchmark providers consistently.

### Expected Output

`research/3d-generation-providers.md`

### Resolution Criteria

- At least two serious providers are tested.
- Results use the same benchmark cases.
- Cost/quality/latency trade-offs are documented.
- One primary provider is recommended.
- One fallback/replacement strategy is documented.

---

## R-008 — Automatic 3D Printability Validation

**Status:** Open  
**Priority:** High  
**Target Phase:** Phase 13  
**Depends On:** R-004, R-007

### Questions

Which checks can be performed automatically?

Possible checks:

- supported file format;
- corrupt file detection;
- polygon/mesh integrity;
- watertight/manifold checks;
- inverted normals;
- bounding-box dimensions;
- mesh scale/unit consistency;
- minimum wall thickness;
- disconnected components;
- degenerate faces;
- self-intersections;
- excessive complexity/polycount.

### Additional Questions

- Which tools/libraries are reliable enough server-side?
- Which checks should be warnings vs blockers?
- Which checks remain human-review responsibilities?
- How should validation reports be persisted?

### Expected Output

`research/printability-validation.md`

### Resolution Criteria

A clear automated-validation baseline exists, including its limits.

---

## R-009 — Server-Side Slicer Choice

**Status:** Open  
**Priority:** Critical  
**Target Phase:** Phase 13  
**Depends On:** R-002, R-003

### Candidate

PrusaSlicer CLI is an initial candidate and must be evaluated rather than assumed.

### Questions

- Which slicer best matches actual printers/workflow?
- Can it run reliably headlessly?
- What metrics can be extracted?
- Can profiles be versioned?
- How are printer/material profiles supplied?
- What resources does slicing consume?
- How should malicious/complex files be isolated?
- What CPU/memory/time limits are required?
- Can the slicer run in an isolated worker/container?
- How stable is the output format for parsing?

### Expected Output

`research/slicer-options.md`

### Resolution Criteria

- Slicer choice is documented.
- Server execution model is documented.
- Required security/resource controls are known.
- Relevant output metrics are documented.

---

## R-010 — Manufacturing Metrics Available from Slicing

**Status:** Blocked  
**Priority:** Critical  
**Target Phase:** Phase 13  
**Depends On:** R-009

### Questions

Determine which reliable metrics can be extracted for pricing/production:

- estimated print time;
- material length;
- material weight;
- support usage;
- number of layers;
- tool changes;
- estimated filament volume;
- profile identifiers;
- other useful G-code/slicer outputs.

### Expected Output

May be included in:

`research/slicer-options.md`

or:

`research/slicer-metrics.md`

### Resolution Criteria

Metrics required by the Price Engine can be obtained consistently from representative models.

---

## R-011 — AI Credit and Usage Economics

**Status:** Open  
**Priority:** High  
**Target Phase:** Phase 11  
**Depends On:** R-007 partially

### Questions

- Which operations should consume credits?
- Should normal text conversation be free or limited?
- What should concept-image generation cost?
- What should 3D generation/regeneration cost?
- How should failed provider jobs affect credits?
- Should first-time users receive free credits?
- What abuse patterns must be prevented?
- How should provider USD cost translate into internal credits?
- How should package pricing be adjusted without changing historical usage records?

### Expected Output

`research/ai-cost-control.md`

### Resolution Criteria

- Internal credit unit is defined.
- Operation costs are defined.
- Free allowance policy is defined.
- Failure/refund behavior is defined.
- Admin adjustment behavior is defined.

---

## R-012 — OpenAI Operation/Model Strategy

**Status:** Open  
**Priority:** Medium  
**Target Phase:** Phase 10

### Questions

Choose appropriate OpenAI capability/model per operation:

- customer conversation;
- image/reference analysis;
- structured requirement extraction;
- tool calling;
- moderation/safety support;
- optional concept-image generation;
- conversation summarization.

### Important Rule

Model choice must remain configurable and should not be hardcoded throughout business logic.

### Expected Output

`research/openai-model-strategy.md`

### Resolution Criteria

Model/capability choices are documented by operation and cost/quality requirements.

---

## R-013 — AI Content and Product Policy

**Status:** Open  
**Priority:** High  
**Target Phase:** Phase 10

### Questions

- Which requested products/content should the service reject?
- Which requests need mandatory Admin review?
- How should copyright/trademark-sensitive requests be handled?
- How should unsafe or inappropriate requested designs be handled?
- What should the AI say when a request is outside service scope?
- What information should be logged for rejected generations?

### Expected Output

`research/ai-product-policy.md`

### Resolution Criteria

A clear application-level AI policy exists independently from the provider prompt.

---

## R-014 — RAG Knowledge Sources and Maintenance

**Status:** Open  
**Priority:** Medium  
**Target Phase:** Phase 10  
**Depends On:** R-001, R-003, R-004

### Questions

- Which documents should be included in RAG?
- Who updates them?
- Which knowledge belongs in RAG vs deterministic rules?
- What metadata is needed?
- How should outdated production documentation be retired?
- Is a vector database necessary initially, or is PostgreSQL-based retrieval sufficient?
- How should source/version traceability work?

### Expected Output

`research/rag-knowledge-design.md`

### Resolution Criteria

Knowledge sources, ingestion, retrieval, ownership, and update process are documented.

---

## R-015 — SMS / OTP Provider

**Status:** Open  
**Priority:** High  
**Target Phase:** Phase 05

### Evaluation Criteria

- reliability inside Iran;
- OTP support;
- delivery status;
- API quality;
- rate limits;
- pricing;
- documentation;
- SDK quality;
- sender/pattern requirements;
- operational support.

### Expected Output

`research/sms-provider.md`

### Resolution Criteria

At least two providers are compared and one is selected with fallback considerations.

---

## R-016 — Iranian Shipping Strategy

**Status:** Open  
**Priority:** Medium  
**Target Phase:** Phase 16

### Questions

- Which shipping methods will actually be offered at launch?
- Which are configured manually vs integrated by API?
- Is automatic price lookup required?
- Is label creation required?
- Is tracking API integration practical?
- How should Tehran/local delivery differ if applicable?
- How are shipping prices maintained?
- Is partial shipment required for mixed Orders?

### Expected Output

`research/shipping-strategy.md`

### Resolution Criteria

MVP shipping methods and integration depth are explicitly defined.

---

## R-017 — Production Hosting Topology

**Status:** Open  
**Priority:** Medium  
**Target Phase:** Phase 25

### Questions

- Where will Next.js be hosted?
- Where will NestJS/API run?
- Where will worker containers run?
- Managed or self-hosted PostgreSQL?
- Managed or self-hosted Redis?
- Reverse proxy strategy?
- CDN needs?
- Network restrictions?
- Backup solution?
- Deployment method?
- Expected monthly cost?

### Constraints

The solution should be appropriate for early-stage traffic and should avoid unnecessary infrastructure complexity.

### Expected Output

`research/production-hosting.md`

### Resolution Criteria

A production topology, cost estimate, backup approach, and scaling path are documented.

---

## R-018 — Production Object Storage

**Status:** Open  
**Priority:** Medium  
**Target Phase:** Phase 25

### Candidate Direction

S3-compatible object storage.

### Questions

- Which provider offers sufficient reliability/cost/access from the target market?
- Does it support signed URLs and expected S3 semantics?
- What are storage/traffic/request costs?
- Is CDN integration needed?
- What backup/versioning options exist?
- What are size/file-count limitations?

### Expected Output

`research/object-storage-provider.md`

### Resolution Criteria

One production storage provider is selected without changing the `StorageService` business interface.

---

## R-019 — Personalization Deposit Policy

**Status:** Open  
**Priority:** High  
**Target Phase:** Phase 15

### Questions

- Is deposit optional or mandatory?
- Percentage or fixed amount?
- Can Admin choose per request?
- Is there a minimum deposit?
- What happens if the PhysicalItem is rejected after receipt?
- What happens to the deposit if the customer cancels?
- What happens when Inspection results in a revised Quote?
- Can work begin before full payment in all cases?

### Expected Output

`research/personalization-payment-policy.md`

### Resolution Criteria

Deposit, cancellation, revised quote, and balance policies are explicit.

---

## R-020 — Quote Expiry and Repricing Policy

**Status:** Open  
**Priority:** Medium  
**Target Phase:** Phase 14 / Phase 15

### Questions

- Do AI Quotes expire?
- Do Personalization Quotes expire?
- How long are prices valid?
- What happens if material prices change?
- Can an expired quote be regenerated automatically?
- Does Admin need to reapprove?
- Can a customer pay an already-started payment after quote expiry?

### Expected Output

`research/quote-policy.md`

### Resolution Criteria

Quote validity and repricing behavior are deterministic.

---

## R-021 — Refund and Cancellation Policy

**Status:** Open  
**Priority:** High  
**Target Phase:** Phase 08 / Before Launch

### Questions

Rules may differ by:

- Ready Product before production;
- Ready Product after production starts;
- AI Design before manufacturing;
- AI generation/credit consumption;
- Personalization before receiving item;
- Personalization after receiving item;
- Personalization after manual work begins.

### Expected Output

`research/refund-cancellation-policy.md`

### Resolution Criteria

Customer-facing and operational cancellation/refund rules are defined for all core flows.

---

## R-022 — Mixed Order / Partial Shipment Policy

**Status:** Open  
**Priority:** Medium  
**Target Phase:** Phase 16

### Questions

When one Order contains several OrderItems with different completion times:

- must all items ship together?
- can items ship separately?
- who pays additional shipping?
- can customer choose?
- what happens if one item is delayed/reworked?

### Expected Output

`research/partial-shipment-policy.md`

### Resolution Criteria

Fulfillment behavior for mixed Orders is explicitly defined.

---

## R-023 — Customer-Facing Production Status Vocabulary

**Status:** Open  
**Priority:** Medium  
**Target Phase:** Phase 01 / Phase 18  
**Depends On:** R-001

### Questions

Internal workflow may contain technical states, but customers need understandable Persian statuses.

Define mappings such as:

- awaiting approval;
- payment pending;
- preparing;
- production/printing;
- post-processing;
- quality control;
- ready to ship;
- shipped;
- delivered;
- rework/delay.

### Expected Output

May be recorded in:

`blueprint/order-lifecycle.md`

and/or:

`research/status-language.md`

### Resolution Criteria

Internal-to-customer status mapping is documented.

---

## R-024 — Customer Data Retention and Deletion

**Status:** Open  
**Priority:** Medium  
**Target Phase:** Before Production Launch

### Questions

Define retention expectations for:

- customer accounts;
- AI conversations;
- uploaded images;
- generated models;
- Personalization photos;
- payment records;
- invoices;
- audit records;
- failed/abandoned AI sessions.

### Expected Output

`research/data-retention-policy.md`

### Resolution Criteria

Operational/privacy retention periods and deletion exceptions are documented.

---

## R-025 — 3D Preview MVP Complexity

**Status:** Open  
**Priority:** Low  
**Target Phase:** Phase 12

### Context

3D preview is desirable but must not create disproportionate MVP complexity.

### Questions

- Can provider GLB output be displayed directly?
- Is conversion required?
- Is browser performance acceptable?
- What fallback preview is needed?
- Does the preview need measurement tools or only rotate/zoom?
- Should preview be customer-facing in MVP or Admin-only if implementation becomes costly?

### Expected Output

May be part of:

`research/3d-generation-providers.md`

### Resolution Criteria

A minimal preview scope is chosen.

---

# Dependency Summary

The most important dependency chain is:

`R-001 Manufacturing Workflow`
→ `R-002 Printers`
→ `R-003 Materials`
→ `R-004 Manufacturing Constraints`

and:

`R-009 Slicer`
→ `R-010 Slicer Metrics`

combined with:

`R-005 Existing Pricing`

which then unlocks:

`R-006 AI Price Estimation Model`

For AI generation:

`R-007 3D Provider`
→ `R-008 Printability Validation`

For AI cost control:

`R-007 3D Provider`
+ actual OpenAI usage
→ `R-011 AI Credit Economics`

---

# Critical Research Before Dependent Implementation

The following research items must not be silently skipped:

1. **R-001** — Manufacturing workflow
2. **R-002** — Printer capabilities
3. **R-003** — Materials/colors
4. **R-004** — Manufacturing constraints
5. **R-005** — Existing pricing process
6. **R-006** — AI Price Engine
7. **R-007** — 3D provider benchmark
8. **R-008** — Printability validation
9. **R-009** — Server-side slicer
10. **R-010** — Slicer metrics

---

# Research Result Template

Every completed research item should answer:

## Decision / Recommendation

What is recommended?

## Evidence

What was tested, measured, compared, or learned?

## Alternatives Considered

What other options were evaluated?

## Trade-offs

What are the benefits and disadvantages?

## Impact on Architecture

Which modules, interfaces, data models, queues, providers, or infrastructure are affected?

## Impact on Business Rules

Do any existing Business Rule IDs change?

## Follow-up Tasks

What implementation or further research is required?

## ADR Required?

Yes / No.

---

# Maintenance Rule

Whenever a new unknown is discovered that could materially affect:

- architecture;
- business rules;
- cost;
- production feasibility;
- security;
- legal/operational policy;
- provider choice;

it must be added to this register rather than left as an undocumented assumption.

When an item is resolved:

1. update its status to `Resolved`;
2. link/reference the research document;
3. create/update an ADR if necessary;
4. update Blueprint/Business Rules if affected;
5. update implementation tasks where required.
