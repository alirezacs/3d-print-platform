# Domain Glossary

**Phase:** 00 — Project Discovery & Product Definition  
**Task:** 00.02 — Create domain glossary  
**Status:** Draft complete — pending repository commit

This glossary defines canonical terms used across code, UI, documentation, Trello tasks, and Codex prompts.

## Core Commerce

### Product
A customer-facing item that has already been produced successfully and can be ordered again. It contains commercial data such as title, description, media, category, base price, and supported configuration options.

### ProductOption
A customer-selectable configuration dimension for a Product, such as color, size, custom text, selected material, or another explicitly supported option. An option may affect price.

### ProductionTemplate
The internal manufacturing recipe for a Ready Product. It may include manufacturing files, printer/profile references, material defaults, print settings, expected duration/material use, post-processing instructions, QC instructions, and internal notes.

### Cart
The customer's temporary collection of purchasable Ready Products and approved AI Designs before checkout. Personalization Requests do not use the normal Cart.

### CartItem
A single configured purchasable entry in a Cart, including item type, quantity, configuration, and price context.

### Order
A durable purchase record created from checkout. It belongs to a Customer and contains one or more OrderItems plus payment, invoice, and shipping context.

### OrderItem
A single purchased item inside an Order. It snapshots purchased configuration and can have an independent production/fulfillment lifecycle.

### Fulfillment
The process of moving an OrderItem from confirmed purchase through production, QC, packaging, shipping, and delivery.

### ProductionJob
A concrete manufacturing unit of work. It can reference an OrderItem, ProductionTemplate or approved AI Design, assigned operator, printer/profile, status, notes, failures/rework, and QC.

## AI Design

### DesignSession
A customer-owned AI-assisted workspace for one custom product idea. It contains the conversation and evolving context and may contain many DesignVersions.

### DesignVersion
An immutable snapshot of a design at a specific point in time. It may include structured specification, reference images, generated previews, 3D assets, dimensions/material/color choices, provider metadata, validation results, and manufacturing metrics. Revisions create new versions instead of overwriting old ones.

### AI Design
A custom product created through the AI-assisted design workflow. It is not purchasable until a final DesignVersion passes required validation, Admin review, and receives an approved locked Quote.

### AIUsage
A recorded AI/provider operation such as text generation, image analysis/generation, 3D generation, or regeneration. It is used for cost analysis, limits, abuse protection, and credit accounting.

### AICredit
An application-level allowance used to control access to costly AI operations. One credit does not necessarily equal one model token.

### RAG Knowledge
Business/manufacturing knowledge retrieved and supplied to AI for context. It helps reasoning but does not enforce hard production rules.

## Pricing & Payments

### PriceEstimate
A calculated manufacturing price produced by the Price Engine from measurable production inputs and versioned pricing rules. It is not necessarily the final customer price.

### Quote
A proposed or approved price for work before final purchase/payment. For AI Designs and Personalization Requests, an approved Quote may include the estimate, Admin adjustment, reason, expiry, and final locked amount.

### Deposit
A partial payment toward a larger quoted amount.

### RemainingBalance
The unpaid portion of a Quote, Order, or service after previous payments/deposits are applied.

### Payment
A financial obligation/transaction associated with an Order, Quote, Deposit, or Remaining Balance. Payment state is separate from operational workflow state.

### PaymentAttempt
A specific interaction with a payment provider. Multiple attempts may exist for the same payment obligation.

### Refund
A recorded return of money related to a previous Payment.

### Invoice
A customer-facing business document summarizing seller/customer data, line items, discounts/shipping, paid amount, remaining balance, and invoice number. MVP requires a normal PDF/business Invoice.

## Personalization

### PersonalizationRequest
A service request to modify/customize a physical item owned by the Customer. It follows its own workflow and does not use the normal Cart.

### PhysicalItem
The actual customer-owned object received by the business for a PersonalizationRequest.

### Inspection
A staff evaluation of a received PhysicalItem before work begins. It may result in accepted, rejected, clarification required, or revised quote required.

### ManualWorkJob
A unit of manual customization work assigned to a Designer or staff member for a PersonalizationRequest.

## Manufacturing

### ManufacturingRule
A deterministic rule that defines a hard production constraint, such as maximum dimensions, supported material/printer combinations, or minimum wall thickness.

### ManufacturingValidation
The process of applying deterministic checks and ManufacturingRules to a design/configuration/3D asset. It produces pass/fail, warnings, violations, and measurable metadata.

### QualityControl / QC
A review step that decides whether manufactured or manually customized work meets required standards. QC may pass, fail, or trigger rework.

## Shipping

### ShippingMethod
A configured delivery option available at checkout or used by staff during fulfillment.

### Shipment
A record representing movement of goods between the business and Customer, including carrier/method, tracking code, status, sent timestamp, and delivered timestamp.

## Identity & Authorization

### Customer
The end user purchasing products or requesting services. A Customer owns addresses, Orders, DesignSessions, and PersonalizationRequests.

### Admin
A privileged staff role responsible for management and approvals. Admin is a Role, not a separate duplicated domain model.

### Designer
A staff role primarily responsible for manual personalization/design work.

### ProductionOperator
A staff role primarily responsible for manufacturing/production work.

### Role
A named grouping of permissions. Initial roles are Customer, Admin, Designer, and Production Operator.

### Permission
A specific authorization capability, such as `product.manage`, `ai_design.approve`, `production.update`, or `personalization.inspect`.

## Platform Concepts

### StorageAsset
A logical file record managed through the storage abstraction. Examples: product image, reference image, generated image, 3D model, production file, and invoice PDF.

### Notification
A message delivered through a supported channel such as in-app or SMS.

### AuditLog
An append-oriented record of a sensitive or privileged action such as AI approval/rejection, quote override, refund action, permission change, or manual state correction.

### State Machine
The explicit definition of valid states and transitions for a workflow. Important state machines include Order, OrderItem, Payment, AI Review, PersonalizationRequest, ProductionJob, Inspection, and Shipment.

## Canonical Naming Rule

Use these terms consistently in backend modules/entities/services, frontend feature naming, API contracts, database naming where appropriate, documentation, Trello, and Codex prompts.

When a new domain concept is introduced, update this glossary before multiple competing names enter the codebase.
