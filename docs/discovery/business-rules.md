# Business Rules

**Phase:** 00 — Project Discovery & Product Definition  
**Task:** 00.03 — Document business rules  
**Status:** Draft complete — pending repository commit

This document defines implementation-independent business rules for the 3D Print Platform.

These rules describe what must remain true regardless of framework, database schema, provider, or UI implementation.

---

## 1. General Rules

1. The platform supports three distinct business flows:
   - Ready Products
   - AI-Assisted Custom Design
   - Personalization Service

2. These three flows may share infrastructure such as authentication, payments, files, notifications, and customer accounts, but they must not be forced into the same workflow when their business behavior differs.

3. PostgreSQL is the source of truth for business state.

4. External providers, queues, caches, and storage systems must not become the authoritative source for:
   - order state;
   - payment state;
   - approval state;
   - production state;
   - personalization state;
   - quote state;
   - financial balance.

5. Important business state changes must be explicit and auditable.

---

## 2. Ready Product Rules

### BR-PRODUCT-001 — Made-to-order

Ready Products are generally made-to-order.

A Product being visible in the catalog does not mean a finished physical unit is currently in stock.

### BR-PRODUCT-002 — Known manufacturability

A Ready Product should only be published as a purchasable product when the business has already established that it can be manufactured successfully.

### BR-PRODUCT-003 — Controlled customization

Customers may only modify Product attributes explicitly exposed as configurable options.

Unsupported properties must not become customer-editable automatically.

### BR-PRODUCT-004 — Option validation

A selected ProductOption value must be valid for that Product at checkout time.

### BR-PRODUCT-005 — Option price modifier

A ProductOption value may increase or decrease the final price.

The price effect must be deterministic and stored as part of the purchased OrderItem snapshot.

### BR-PRODUCT-006 — Commercial/manufacturing separation

Customer-facing Product data and internal ProductionTemplate data must remain separate concepts.

### BR-PRODUCT-007 — Historical snapshot

When a Ready Product is purchased, the OrderItem must preserve enough commercial/configuration data so that later catalog edits do not alter the historical purchase.

---

## 3. Cart Rules

### BR-CART-001 — Supported item types

The normal Cart may contain:

- Ready Products
- approved AI Designs

### BR-CART-002 — Personalization exclusion

Personalization Requests must not be added to the normal Cart.

They follow their own quote/payment workflow.

### BR-CART-003 — Mixed cart

Ready Products and approved AI Designs may coexist in the same Cart and become part of one Order.

### BR-CART-004 — Revalidation before checkout

Cart contents must be revalidated before Order creation.

At minimum, the system must verify:

- item is still purchasable;
- selected configuration is still valid;
- final price is valid;
- AI approval/quote is still valid where applicable.

---

## 4. Order Rules

### BR-ORDER-001 — One checkout, one Order

A successful normal checkout creates one Order.

### BR-ORDER-002 — Independent OrderItem lifecycle

Each OrderItem must be able to progress independently through production and fulfillment.

Example:

- Item A may be printing;
- Item B may be in QC;
- Item C may be ready to ship.

### BR-ORDER-003 — Order aggregation

Order-level status may be derived from OrderItem states, but it must not erase the individual lifecycle of each item.

### BR-ORDER-004 — State transition validation

Order and OrderItem states must only change through defined valid transitions.

Arbitrary status changes are not allowed.

### BR-ORDER-005 — Cancellation

Cancellation rules may depend on:

- payment state;
- production state;
- whether manufacturing has started;
- whether customer-owned physical goods are involved.

Cancellation policy must be explicit and documented before launch.

---

## 5. AI Design Rules

### BR-AI-001 — AI is not authoritative

AI output is advisory/generative and must not directly become authoritative business state.

### BR-AI-002 — AI cannot approve itself

AI-generated designs cannot approve their own manufacturability, pricing, or final purchase eligibility.

### BR-AI-003 — Versioning

Significant design revisions must create a new DesignVersion.

Existing DesignVersions must not be silently overwritten.

### BR-AI-004 — Final version selection

Only one explicitly selected DesignVersion may be submitted as the current final candidate for approval.

### BR-AI-005 — Validation required

A submitted DesignVersion must pass required validation stages before it can be approved for purchase.

### BR-AI-006 — Human approval required in MVP

Admin review is mandatory before an AI Design becomes purchasable.

### BR-AI-007 — Rejection reason

If an AI Design is rejected, the reason must be recorded and visible in the appropriate customer/admin workflow.

### BR-AI-008 — Revision after rejection

A rejected DesignVersion remains historical.

A customer must create or continue with a new DesignVersion rather than mutating the rejected version.

### BR-AI-009 — AI must stay in product scope

The AI experience must remain focused on product design/manufacturing use cases.

It must not become an unrestricted general-purpose chatbot.

### BR-AI-010 — Hard rules outside LLM

Critical manufacturing/business constraints must be enforced outside the LLM through deterministic rules or application logic.

### BR-AI-011 — RAG is contextual, not authoritative

RAG may provide business/manufacturing knowledge, but must not replace deterministic validation.

### BR-AI-012 — Usage control

Costly AI operations must be measurable, rate-limited, and subject to usage/credit rules.

---

## 6. AI Pricing Rules

### BR-PRICE-001 — AI does not determine final price

The LLM must not invent or directly determine the final customer price.

### BR-PRICE-002 — Dedicated pricing engine

AI-order price estimation must be produced by a dedicated deterministic Price Engine.

### BR-PRICE-003 — Real manufacturing inputs

The Price Engine should use measurable production inputs where available.

Potential inputs include:

- material usage;
- material cost;
- print duration;
- machine usage;
- support material;
- labor;
- post-processing;
- packaging;
- failure/risk allowance;
- minimum charge;
- margin.

### BR-PRICE-004 — Research before final algorithm

The final pricing formula must not be considered complete until real production data and the existing pricing process are reviewed.

### BR-PRICE-005 — Admin review

The estimated price must be reviewable by Admin before purchase.

### BR-PRICE-006 — Admin override

If Admin changes the calculated estimate, the final amount and adjustment reason must be recorded.

### BR-PRICE-007 — Price lock

The customer must only pay after a final Quote has been approved and locked.

### BR-PRICE-008 — Pricing version traceability

A stored estimate should be traceable to the pricing-rule version used to calculate it.

---

## 7. Personalization Rules

### BR-PERS-001 — Independent workflow

Personalization is a ServiceRequest workflow and does not use the normal Cart.

### BR-PERS-002 — Request first

The customer must first submit a PersonalizationRequest with sufficient details and reference images.

### BR-PERS-003 — Admin review before sending item

Admin must review the request before the workflow reaches the "Awaiting Physical Item" stage.

### BR-PERS-004 — Quote required

Admin must define the quoted price before the customer sends the physical item under the normal workflow.

### BR-PERS-005 — Awaiting item state

Once the request is approved and priced, its status changes to a state equivalent to "Awaiting Physical Item".

### BR-PERS-006 — Deposit or full payment

After approval/quote, the customer may pay:

- a deposit; or
- the full quoted amount.

The exact deposit policy may be configurable.

### BR-PERS-007 — Customer transport responsibility

The customer is responsible for the transportation/shipping cost of sending the physical item and receiving it back unless the business explicitly changes this policy.

### BR-PERS-008 — Physical inspection required

Receiving a physical item does not automatically mean work can begin.

A physical Inspection must occur first.

### BR-PERS-009 — Post-receipt rejection allowed

The business may reject the PhysicalItem after inspection.

Possible causes include:

- unsuitable material;
- existing damage;
- technical limitation;
- safety risk;
- mismatch with submitted information;
- unacceptable requested content.

### BR-PERS-010 — Existing condition must be recordable

The business must be able to record the condition of the received physical item, including pre-existing damage.

### BR-PERS-011 — Revised quote allowed after inspection

If inspection reveals additional complexity, the business may issue a revised quote before work begins.

### BR-PERS-012 — Work starts only after acceptance

Manual customization work must not begin until the received item has been accepted after Inspection.

### BR-PERS-013 — Remaining balance before return

If the customer initially paid only a deposit, the remaining balance must be fully paid before the item is returned/shipped back.

### BR-PERS-014 — No cash on delivery

The platform does not support COD for Personalization completion.

---

## 8. Payment Rules

### BR-PAY-001 — Supported payment types

The platform must support:

- full payment;
- deposit;
- remaining-balance payment.

### BR-PAY-002 — No COD

Cash on delivery is outside MVP scope.

### BR-PAY-003 — Payment state separation

Payment status and Order/Service status are separate concerns.

A successful payment may cause a business-state transition, but they must not be represented as the same field/state.

### BR-PAY-004 — Multiple attempts

A failed or canceled PaymentAttempt must not prevent a customer from retrying payment.

### BR-PAY-005 — Idempotent callback

Payment-provider callbacks and verification must be idempotent.

The same successful payment must never be applied twice.

### BR-PAY-006 — Amount validation

The amount verified with the payment provider must match the expected payable amount before the business state is updated.

### BR-PAY-007 — Overpayment protection

Deposit/balance workflows must prevent accidental overpayment.

### BR-PAY-008 — Refund history

Refund activity must be recorded even if the external refund operation is initially performed manually.

---

## 9. Invoice Rules

### BR-INVOICE-001 — Internal business invoice

The platform must generate a customer-facing business Invoice.

### BR-INVOICE-002 — No tax-system integration required

Iranian electronic tax-system integration is not required for the current MVP.

### BR-INVOICE-003 — Invoice traceability

An Invoice must remain traceable to the related Order/service and financial records.

---

## 10. Manufacturing Rules

### BR-MFG-001 — Admin-managed capabilities

Manufacturing capabilities and constraints must be manageable as business data/configuration where practical.

They must not be embedded only inside AI prompts.

### BR-MFG-002 — Deterministic validation

Hard manufacturing constraints must be validated deterministically.

### BR-MFG-003 — Production data privacy

Internal production instructions, manufacturing files, and operator notes must not be publicly exposed by default.

### BR-MFG-004 — ProductionJob required

Actual manufacturing work should be represented by explicit ProductionJob records rather than only Order status fields.

### BR-MFG-005 — QC required

Manufactured output must have a QC step before being considered ready for fulfillment.

### BR-MFG-006 — Rework support

QC failure must support a rework/failure path rather than forcing the job directly to completion.

---

## 11. Shipping Rules

### BR-SHIP-001 — Ready/AI shipping

Ready Product and AI Design Orders may use platform-configured ShippingMethods.

### BR-SHIP-002 — Tracking

Where tracking information exists, it must be recordable and visible to the customer.

### BR-SHIP-003 — Personalization inbound transport

The platform does not need to purchase or manage inbound Personalization transportation on behalf of the customer.

### BR-SHIP-004 — Personalization return blocking

A Personalization PhysicalItem must not be dispatched for return when a required RemainingBalance is unpaid.

---

## 12. Authentication Rules

### BR-AUTH-001 — Customer authentication

The primary customer login method is mobile number + OTP.

### BR-AUTH-002 — Optional email

Email may be associated with a customer account but is not the primary authentication requirement for MVP.

### BR-AUTH-003 — Stronger staff authentication

Staff accounts require stronger authentication than normal customer OTP-only access.

### BR-AUTH-004 — Permission-based authorization

Authorization must be permission-based and not rely only on role-name checks.

### BR-AUTH-005 — Ownership enforcement

Customers may access only resources they own unless explicitly shared by business logic.

---

## 13. File Rules

### BR-FILE-001 — Storage abstraction

Business modules must use a StorageService abstraction instead of depending directly on one object-storage provider.

### BR-FILE-002 — No large binary files in PostgreSQL

Large user media, generated images, 3D models, invoices, and manufacturing files must not be stored as database blobs by default.

### BR-FILE-003 — Image-only customer upload where specified

For Personalization MVP, customer uploads are limited to supported image formats.

### BR-FILE-004 — AI image reference support

AI Design sessions must support customer image references.

### BR-FILE-005 — No customer STL upload service in MVP

Direct customer "upload a 3D file and print it" service is outside current MVP scope.

---

## 14. Finance Rules

### BR-FIN-001 — Lightweight operational finance

The platform needs operational financial tracking, not a full accounting system.

### BR-FIN-002 — Required financial concepts

The system should support:

- payments;
- deposits;
- balances;
- refunds;
- revenue;
- basic costs;
- basic profit reporting.

### BR-FIN-003 — No double-entry accounting requirement

General ledger, journals, debit/credit accounting, and full accounting software behavior are outside MVP scope.

---

## 15. Audit Rules

### BR-AUDIT-001 — Sensitive changes are auditable

The following actions must be auditable:

- AI approval/rejection;
- price override;
- refund action;
- role/permission changes;
- important manual state corrections;
- manufacturing-rule changes;
- important business-setting changes.

### BR-AUDIT-002 — Audit history is append-oriented

Audit history should not be silently overwritten.

---

## 16. Customer Status Visibility Rules

### BR-STATUS-001 — Detailed status tracking

Customers must receive meaningful progress visibility rather than a single generic Order status.

### BR-STATUS-002 — Internal vs customer status

Internal workflow states may be more detailed than customer-facing statuses.

The system may map several internal states into a simpler customer-facing status where appropriate.

### BR-STATUS-003 — No false completion

A customer-facing flow must not be shown as complete before all required payment, production/QC, and fulfillment conditions are satisfied.

---

## 17. Scope Rules

### BR-SCOPE-001 — MVP includes all three core flows

Ready Products, AI Design, and Personalization are all MVP requirements.

### BR-SCOPE-002 — Architecture must allow future growth

Deferred features do not need to be implemented now, but current design should avoid unnecessarily blocking future addition of:

- mobile/PWA;
- more providers;
- additional roles;
- additional manufacturing processes;
- advanced shipping integrations.

### BR-SCOPE-003 — No premature microservices

The system should remain a Modular Monolith until real scaling or organizational needs justify extracting services.

---

## 18. Change Rule

Any future decision that contradicts one of these rules must:

1. explicitly identify the affected Business Rule ID;
2. explain why the rule is changing;
3. update this document;
4. update affected Blueprint/ADR documentation;
5. update implementation and tests accordingly.

Business rules must not drift silently.
