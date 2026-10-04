# 3D Print Platform — Product Vision & MVP Scope

**Document:** Product Scope  
**Phase:** 00 — Project Discovery & Product Definition  
**Task:** 00.01 — Consolidate product vision and MVP scope  
**Status:** Draft complete — pending repository commit

## 1. Product Vision

The product is a Persian-first web platform for a 3D-printing/manufacturing business that combines three customer journeys:

1. **Ready Products** — browse existing products, choose supported configuration options, order, pay online, and track production/shipping.
2. **AI-Assisted Custom Design** — describe or visually reference an object, collaborate with an AI-assisted design flow, obtain a manufacturable 3D design, receive a price estimate reviewed/approved by staff, then purchase the approved design for production.
3. **Personalization Service** — submit an existing physical product for manual/custom modification; the business reviews, quotes, may collect a deposit/full payment, receives and inspects the item, performs the work, collects any remaining balance, and returns it.

The platform is not a generic marketplace or a public 3D-model repository. It is the operating and customer-facing system for the business itself.

## 2. Primary Product Goals

The MVP should:

- Present the full range of the business's existing work and portfolio rather than restricting the storefront to only a few product categories.
- Turn ready-made examples into purchasable, made-to-order products where appropriate.
- Provide controlled product configuration such as supported color, size, material, or other explicitly allowed options.
- Allow customers to request genuinely custom products through an AI-assisted workflow.
- Keep AI behavior inside business and manufacturing constraints instead of allowing unrestricted off-topic interaction.
- Use deterministic validation and business rules for hard constraints; RAG can provide contextual manufacturing/business knowledge but must not replace hard validation.
- Estimate AI-order manufacturing price using a versioned pricing engine based on real production data and measurable manufacturing metrics.
- Require staff review/approval before an AI-generated design becomes purchasable.
- Support a separate personalization workflow for customer-owned physical items.
- Provide customers with clear tracking for orders, AI designs, payments, production, personalization requests, and shipments.
- Provide staff with an admin system for catalog, production, AI review, personalization, finance, shipping, settings, and audit history.
- Be built as a custom application rather than WordPress.

## 3. Actors

### Customer
Can authenticate with mobile/OTP, browse products/portfolio, configure supported options, manage cart/checkout, create AI design sessions, upload reference images, review design versions, submit a final design for review, purchase an approved AI design, submit personalization requests, pay deposits/full/balance payments, manage addresses, download invoices, track production/shipments, and receive notifications.

### Admin
Can manage catalog/media/options/production templates, review AI designs, inspect validation/pricing data, approve/reject/request revision, lock or adjust quotes, manage personalization, orders, payments/refunds, shipping, finance, business settings, and audit history.

### Designer
Can work on assigned manual personalization jobs, update progress, record notes, and submit work for QC/review.

### Production Operator
Can work with production jobs, printers/profiles, production status, failures/rework, and QC.

## 4. MVP Flow A — Ready Products

1. Customer browses catalog/portfolio.
2. Opens a product.
3. Selects only supported options.
4. System validates configuration and price.
5. Adds to cart.
6. Authenticates if required.
7. Selects address/shipping.
8. Pays full amount.
9. Order is created.
10. Each OrderItem gets its own fulfillment lifecycle.
11. Production work is created from the internal Production Template.
12. Staff performs production and QC.
13. Item becomes ready to ship.
14. Tracking is recorded.
15. Customer sees order timeline and invoice.

### Rules
- Ready Products are made-to-order.
- Customer customization is limited to configured options.
- Internal production files/recipes stay private.
- Order items snapshot purchased product/configuration.

## 5. MVP Flow B — AI-Assisted Custom Design

1. Customer starts an AI Design Session.
2. Describes the object and may upload references.
3. AI structures the request and collects missing constraints.
4. Conversation and design intent are persisted.
5. Significant revision/generation creates an immutable Design Version.
6. Selected 3D provider creates the model.
7. Assets are stored in object storage.
8. Deterministic validation and manufacturing rules run.
9. Slicer/manufacturing analysis extracts measurable metrics where appropriate.
10. Pricing engine calculates an estimate from versioned rules.
11. Customer submits a selected version for staff review.
12. Staff reviews summary, spec, assets, preview, validation, constraints, and estimate.
13. Staff approves, rejects, or requests changes.
14. If approved, a final locked quote is created; any override requires a recorded reason.
15. Only the approved version/quote can become a cart item.
16. Customer pays through normal checkout.
17. Production job(s) are created.
18. Production, QC, shipping, and tracking follow normal fulfillment.

### AI Guardrails

AI behavior has three layers:

1. **System/Product Policy** — scope, allowed behavior, refusal/redirection, business rules.
2. **RAG/Knowledge Retrieval** — business/manufacturing knowledge.
3. **Deterministic Rules/Validation** — hard constraints such as dimensions, supported combinations, approvals, and payment conditions.

RAG is a knowledge layer, not the enforcement mechanism for critical rules.

### Rules
- AI is not the source of truth for pricing.
- AI cannot bypass manufacturing validation.
- AI output is not directly purchasable.
- Admin approval is mandatory.
- Final price is locked before payment.
- Design versions are immutable.
- Expensive AI operations are usage/credit controlled.
- Provider integrations use replaceable adapters.

## 6. MVP Flow C — Personalization Service

1. Customer describes the physical product and requested customization and uploads references.
2. Request is submitted.
3. Admin reviews it.
4. Admin may reject, ask for clarification, or approve with a quote.
5. Customer pays deposit or full amount per policy.
6. Request becomes Awaiting Physical Item.
7. Customer sends/delivers the item.
8. Staff records receipt.
9. Staff performs physical inspection.
10. Item can still be rejected or re-quoted after inspection.
11. Accepted work is assigned to a designer/operator.
12. Work proceeds through manual work and QC states.
13. Remaining balance is collected when required.
14. Item is returned and tracking is recorded.

### Rules
- Personalization does not use the standard cart.
- A quote is required.
- Deposit and full payment are supported.
- Customer pays transport costs unless policy changes.
- Pre-approval does not guarantee physical acceptance.
- Customer-owned items need clear receipt/inspection/work/QC/payment/return history.

## 7. Shared MVP Capabilities

- Customer OTP authentication.
- Stronger staff authentication.
- RBAC and resource ownership checks.
- ZarinPal behind a payment-provider abstraction.
- Full payments, deposits, balances, refunds.
- S3-compatible storage abstraction; MinIO locally.
- Redis/BullMQ background jobs with retries/idempotency/failure visibility.
- SMS provider abstraction and in-app notifications.
- Audit logging for sensitive/admin actions.
- Customer dashboard for orders, AI designs, personalization, payments, invoices, shipments, and notifications.
- Admin panel for catalog, production, AI review, personalization, finance, shipping, users/RBAC, settings, and audits.

## 8. Technical Direction

- Frontend: Next.js / TypeScript
- Backend: NestJS / TypeScript
- Architecture: Modular Monolith
- Database: PostgreSQL
- Queue/Cache: Redis
- Jobs: BullMQ or equivalent
- Storage: S3-compatible abstraction; MinIO for local
- AI: OpenAI behind provider adapter
- 3D Generation: provider selected after benchmark, behind adapter
- 3D Validation/Slicing: isolated worker/process after research
- Payments: provider abstraction, initially ZarinPal
- Customer Auth: mobile/OTP-first
- UI: Persian-first, RTL, responsive
- Documentation: Docs-as-Code

## 9. Explicit MVP Boundaries

Included:

- Public catalog and portfolio.
- Ready-product configuration and purchase.
- Cart/checkout for Ready Products and approved AI designs.
- OTP auth and addresses.
- ZarinPal payments.
- Orders and per-item fulfillment.
- Production jobs and QC.
- AI sessions with text/image context.
- Design version history.
- 3D provider integration and preview.
- Baseline 3D validation.
- Slicing/metric extraction if validated by research.
- Versioned price estimation.
- Mandatory admin AI review.
- Approved AI quote/purchase flow.
- AI usage/credit controls.
- Personalization request/quote/deposit/inspection/work/balance/return.
- Shipping/tracking.
- PDF invoices.
- SMS and in-app notifications.
- Customer dashboard.
- Admin panel.
- RBAC.
- Audit logs.
- Monitoring, tests, CI/CD, backups, and launch hardening.

## 10. Deferred / Post-MVP

- Native mobile apps.
- Multi-vendor marketplace.
- Customer/community 3D sharing.
- Public 3D asset marketplace.
- Fully automated AI approval.
- Fully automated feasibility decisions where human judgment is still needed.
- Full ERP/accounting replacement.
- Advanced warehouse/inventory.
- Multiple payment gateways unless needed.
- Deep integration with every shipping carrier.
- Real-time printer-farm orchestration unless proven necessary.
- Microservices without evidence.
- Recommendation engine.
- Loyalty/gamification.
- Multi-language beyond Persian-first.
- Advanced marketing automation.
- Complex coupon/campaign engine unless launch requires it.
- General-purpose off-topic chatbot behavior.

## 11. Assumptions

1. The business can manufacture a broad range of products and is not limited to three categories.
2. Existing portfolio examples can seed the public catalog/inspiration gallery.
3. Most ready products are made after purchase.
4. Ready-product customization can be represented as finite supported options.
5. The business already has a repeatable pricing concept, partly represented by an existing Telegram bot/process.
6. Pricing logic must be formalized only after reviewing the actual formula, historical examples, exceptions, and production practices.
7. AI/3D output may not always be printable and needs validation/human review.
8. Staff can review AI orders before purchase/production.
9. Personalization can involve a customer-owned physical item.
10. Persian/RTL is the primary launch UX.
11. Integrations are initially Iran-focused.
12. A modular monolith is sufficient for MVP/early production.

## 12. Known Unknowns

### Manufacturing
- Printing technologies.
- Printer models/build volumes.
- Materials/colors.
- Nozzle/layer/profile capabilities.
- Post-processing.
- Failure/reprint patterns.
- QC procedure.

### Pricing
- Existing Telegram-bot formula.
- Material cost source/update process.
- Machine-time cost.
- Labor/post-processing.
- Support material.
- Packaging.
- Failure/risk allowance.
- Minimum charge.
- Margin policy.
- Quote validity.
- Override rules.

### AI / 3D
- Final OpenAI model choices.
- AI free allowance / credit economics.
- Final 3D-generation provider.
- Licensing/commercial terms.
- Printability quality.
- Mesh-validation tooling.
- Slicer choice.
- Slicer resource/security limits.
- 3D preview strategy.

### Iranian Integrations
- SMS/OTP provider.
- Shipping carriers/API availability.
- Hosting topology.
- Production object storage.

### Operations / Policy
- Refund/cancellation policy.
- Personalization deposit rules.
- Quote expiry.
- Rejected-item return procedure.
- Partial-shipment policy.
- Lead-time communication.

## 13. MVP Success Criteria

MVP is launch-ready when:

- Ready Product purchase works end-to-end.
- AI design works from session → generation → validation → estimate → staff approval → locked quote → payment → production.
- Personalization works from request → quote → payment → receipt/inspection → work/QC → balance → return.
- Staff can operate all flows without direct DB manipulation.
- Payments are idempotent/auditable.
- AI/provider usage is controlled.
- Manufacturing/pricing rules are versioned/traceable.
- Critical flows have automated tests.
- Monitoring, backups, security controls, and runbooks exist.
- Persian/RTL customer flows work on mobile and desktop.

## 14. Scope Control Rule

A new feature belongs in MVP only if it is required to make one of the three primary business flows safe, operable, payable, manufacturable, trackable, or launch-ready.

Anything else defaults to the post-MVP backlog unless a documented dependency proves otherwise.
