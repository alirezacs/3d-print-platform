# System Overview

**Status:** Draft  
**Version:** 0.1  
**Project Type:** 3D Printing Commerce & Design Platform

---

## 1. Purpose

This platform provides customers with multiple ways to order physical products produced or customized by the business.

The platform combines:

- e-commerce
- AI-assisted product design
- 3D model generation
- manufacturing workflows
- manual product personalization
- payments
- production tracking
- order fulfillment
- administrative management

The system must be designed as a long-term product rather than a temporary online store.

---

## 2. Main Business Models

The platform supports three primary sales flows.

### 2.1 Ready-to-Order Products

These are products that have already been successfully produced before.

Examples:

- phone cases
- AirPods cases
- glasses-related products
- keychains
- decorative objects
- other previously manufactured products

These products are not necessarily kept in physical inventory.

Instead, they are generally produced after the customer places an order.

Because they have already been manufactured successfully, the business already knows that they are technically manufacturable.

Each product may expose a limited set of customizable options such as:

- color
- size
- text
- selected attributes

Not every property of the product can necessarily be customized.

Some options may affect the final price.

### High-Level Flow

Customer

→ Browse Products

→ Select Product

→ Configure Available Options

→ Add to Cart

→ Checkout

→ Payment

→ Production

→ Quality Control

→ Shipment

→ Delivery

---

## 3. AI-Assisted Product Design

Customers can create a new product by interacting with an AI system.

The AI experience must support both:

- text input
- image input

The customer communicates their desired product characteristics to the AI.

The system may generate:

- concept images
- design proposals
- 3D models

The final goal is to generate a real 3D model that can eventually enter the manufacturing workflow.

### AI Responsibilities

OpenAI will primarily be responsible for:

- conversation
- understanding customer requirements
- vision analysis
- extracting structured design requirements
- helping refine the design
- suggesting valid design decisions
- interacting with internal platform tools
- applying business and manufacturing knowledge

A specialized 3D generation provider may be responsible for converting design requirements or images into actual 3D models.

Potential providers include services such as:

- Meshy
- Rodin / Hyper3D
- other providers selected during implementation research

The 3D provider must remain replaceable.

---

## 4. AI Design Versioning

Every AI design session may contain multiple design versions.

A customer may repeatedly refine a product.

Example:

Design Version 1

→ Change dimensions

→ Design Version 2

→ Change color

→ Design Version 3

→ Change geometry

→ Design Version 4

Each version must be stored independently.

A version may contain information such as:

- customer requirements
- AI-generated structured specification
- reference images
- generated concept images
- generated 3D model
- selected material
- selected color
- dimensions
- generation metadata
- validation result
- creation timestamp

Previous versions must not be silently overwritten.

---

## 5. AI Usage Limits

AI usage must not be unlimited.

Some AI operations may have significantly different costs.

For example:

- text conversation
- image generation
- image analysis
- 3D generation
- 3D regeneration
- model refinement

The platform should therefore support an AI usage or credit system.

Example:

Customer receives a limited number of free design operations.

Additional generations may require payment or additional credits.

The final pricing strategy for AI credits will be determined later.

---

## 6. AI Manufacturing Constraints

AI must not independently decide whether a product is manufacturable.

Manufacturing constraints must exist outside the AI model.

Examples include:

- available materials
- available colors
- printer capabilities
- maximum dimensions
- minimum dimensions
- supported manufacturing methods
- wall thickness requirements
- production limitations

These constraints must be manageable from the Admin Panel.

AI can read and use these constraints, but the platform must independently validate generated designs.

The system must therefore separate:

AI reasoning

from

Manufacturing validation.

---

## 7. AI Knowledge and RAG

RAG may be used to provide AI with business-specific knowledge.

Possible knowledge sources include:

- production guidelines
- material documentation
- printer documentation
- design guidelines
- internal manufacturing instructions
- frequently asked questions
- product examples
- business policies

RAG is not considered a replacement for deterministic validation rules.

RAG provides knowledge.

Validation rules enforce constraints.

---

## 8. AI Design Validation

Before an AI-generated design can become an order, it must pass several checks.

Expected high-level pipeline:

AI Conversation

→ Design Specification

→ 3D Generation

→ Technical Validation

→ Manufacturing Validation

→ Price Estimation

→ Admin Review

→ Customer Payment

→ Production

During the initial versions of the platform, human approval is mandatory.

The system must be designed so that human approval can potentially become optional in the future.

---

## 9. AI Order Approval

The customer does not immediately pay after generating a design.

The intended flow is:

Customer finalizes a design

→ Automated validation runs

→ Estimated pricing is calculated

→ Admin reviews the design

→ Admin approves or adjusts the final price

→ Final price is locked

→ Customer can pay

→ Production begins

If the design is rejected, the Admin must be able to provide a reason.

The customer may then return to the AI design process and create another version.

---

## 10. AI Price Estimation

The AI ordering flow requires a future Price Estimation Engine.

The exact pricing formula has not yet been defined.

This component must therefore remain modular.

Potential future inputs may include:

- material usage
- material type
- print duration
- machine usage
- support material
- post-processing
- labor
- complexity
- failure risk
- packaging
- profit margin
- minimum order amount

A possible future pipeline is:

3D Model

→ Slicer

→ Manufacturing Metrics

→ Pricing Rules

→ Estimated Price

The exact pricing algorithm is intentionally deferred.

A research task must be created before implementation of this component.

---

## 11. Product Personalization Service

The platform also supports customization of physical products owned by customers.

Example:

A customer wants artwork manually applied to their shoes.

This service is fundamentally different from normal 3D printed products.

The customer owns the physical item and sends it to the business.

The work is manually performed by a designer or artist.

---

## 12. Personalization Request Flow

The intended high-level flow is:

Customer submits request form

→ Customer uploads reference images

→ Admin reviews request

→ Admin approves or rejects request

→ Admin determines price

→ Request becomes Awaiting Item

→ Customer pays deposit or full amount

→ Customer ships physical product

→ Business receives product

→ Product inspection

→ Request accepted or rejected after inspection

→ Manual customization work

→ Quality control

→ Remaining payment if required

→ Return shipment

→ Delivery

Personalization requests do not use the normal shopping cart.

They are managed through a separate service-request workflow.

---

## 13. Personalization Shipping

The customer is responsible for shipping costs related to the personalization service.

The platform does not need to automatically purchase inbound shipping.

The customer sends the product using their preferred shipping method.

The platform should allow relevant shipment information to be recorded.

---

## 14. Product Inspection

After receiving a customer's physical product, a staff member must inspect it.

Possible outcomes include:

- accepted
- rejected
- needs customer clarification
- needs revised quote
- canceled

Reasons for rejection may include:

- unsuitable material
- existing damage
- technical limitations
- unacceptable requested content
- inability to safely perform the work

Inspection history must be recorded.

---

## 15. Cart Model

The shopping cart is used for:

- ready-to-order products
- approved AI-designed products

Personalization requests do not use the shopping cart.

A cart may contain multiple types of purchasable items.

Example:

Order

- Ready Product A
- Ready Product B
- AI Design C

These items may have different production timelines.

---

## 16. Order and Fulfillment Model

A customer checkout creates one Order.

Each Order contains multiple Order Items.

Each Order Item may have its own independent production or fulfillment lifecycle.

Example:

Order #1001

- Item 1 → printing
- Item 2 → quality control
- Item 3 → awaiting production

This allows mixed orders without forcing every item to share the same production status.

---

## 17. Customer Dashboard

Customers must have access to a dashboard.

The dashboard should eventually allow customers to manage or view:

- profile
- addresses
- orders
- order details
- payments
- invoices
- AI design sessions
- design versions
- AI-generated products
- personalization requests
- personalization status
- production progress
- shipment status
- payment balance

Detailed progress tracking is a core platform feature.

---

## 18. Authentication

The primary customer authentication method will be:

Mobile Number + OTP

Email may remain optional.

Staff and Admin authentication may use stronger security mechanisms such as:

- password
- OTP
- two-factor authentication

The exact staff authentication implementation will be specified separately.

---

## 19. User Roles

Initial system roles include:

### Customer

Can:

- browse products
- manage cart
- place orders
- make payments
- use AI design
- create personalization requests
- view order progress

### Admin

Can manage the overall platform.

Responsibilities include:

- products
- pricing
- orders
- AI requests
- approvals
- manufacturing rules
- personalization requests
- payments
- financial reports
- users
- system configuration

### Designer

Primarily works with personalization requests and design-related tasks.

### Production Operator

Primarily manages manufacturing and production jobs.

The authorization system must support adding new roles and permissions later.

---

## 20. Admin Panel

The Admin Panel must provide management capabilities for major business concepts.

Expected areas include:

- products
- categories
- product options
- orders
- order items
- customers
- AI design sessions
- AI design versions
- AI approvals
- manufacturing constraints
- printers
- materials
- colors
- pricing configuration
- production jobs
- personalization requests
- inspections
- payments
- deposits
- remaining balances
- refunds
- invoices
- shipping
- users
- roles
- permissions
- financial reports
- AI usage
- AI credits
- system settings

---

## 21. Financial Scope

The platform requires lightweight financial management.

The goal is not to build full accounting software.

The financial module should support concepts such as:

- payments
- deposits
- remaining balances
- refunds
- revenue
- costs
- basic profit calculations
- transaction history
- financial reports

---

## 22. Payment

The primary payment gateway is:

ZarinPal

Supported payment scenarios include:

- full payment
- deposit payment
- remaining balance payment

Cash on delivery is not required.

---

## 23. Invoice

The platform must generate customer invoices.

The required invoice is an internal business invoice.

Integration with Iranian tax systems or electronic tax invoice systems is currently outside the project scope.

Invoices should eventually be available through the Customer Dashboard.

---

## 24. Shipping

Ready-to-order products and AI products may support multiple shipping providers.

Shipping architecture must remain extensible.

Personalization inbound shipping is handled independently by the customer.

The platform must support recording shipment and tracking information.

---

## 25. Product Data Separation

Customer-facing product information must be separated from manufacturing information.

### Product

Contains commercial information such as:

- title
- description
- images
- category
- price
- available options
- customization options

### Production Template

Contains internal manufacturing knowledge such as:

- manufacturing files
- material
- printer profile
- print configuration
- layer height
- infill
- support settings
- expected duration
- expected material usage
- post-processing instructions
- quality-control instructions
- internal notes

This information is generally not exposed directly to customers.

---

## 26. File Storage

The application must use an abstraction for file storage.

Example:

StorageService

The business logic must not directly depend on a specific storage provider.

Possible environments:

Local Development

→ MinIO

Testing / Staging

→ S3-compatible Object Storage

Production

→ S3-compatible Object Storage

Potential production providers may include ArvanCloud or another compatible provider.

Expected stored files include:

- product images
- user uploads
- AI reference images
- generated images
- 3D models
- production files
- invoices
- personalization images

---

## 27. Core Technology Stack

### Frontend

Next.js

### Backend

NestJS

### Primary Database

PostgreSQL

### Cache / Queue Infrastructure

Redis

### Background Jobs

BullMQ or an equivalent job-processing solution

### File Storage

S3-compatible Object Storage

### Development Infrastructure

Docker

---

## 28. Backend Architecture Direction

The initial backend architecture will follow a Modular Monolith approach.

Business domains must remain logically separated.

Possible domains include:

- Identity
- Customer
- Catalog
- Cart
- Ordering
- Payment
- AI Design
- Manufacturing
- Pricing
- Production
- Personalization
- Shipping
- Storage
- Notification
- Finance
- Administration

The architecture should allow individual components to be extracted into independent services later if required.

---

## 29. Source of Truth

PostgreSQL is the primary source of truth for business data.

Redis, message queues, AI providers and storage providers must not become authoritative sources for business state.

Examples of authoritative business state include:

- order status
- payment status
- AI design approval
- production status
- personalization status
- financial balance

---

## 30. Background Processing

Some operations should not block normal HTTP requests.

Examples include:

- AI processing
- image generation
- 3D generation
- model validation
- slicing
- price estimation
- invoice generation
- notifications
- file processing

These operations should be processed through background jobs when appropriate.

---

## 31. Scalability

The platform should be engineered correctly from the beginning but should not prematurely introduce unnecessary distributed-system complexity.

Initial expected order volume is relatively low.

However, the system should support future growth through:

- stateless application services
- background workers
- queue-based processing
- object storage
- modular domain boundaries
- database indexing
- caching
- horizontal scaling where necessary

---

## 32. Language and Interface

The initial platform language is Persian.

The customer-facing interface must support RTL layouts.

Multi-language support is not an MVP requirement.

---

## 33. MVP Requirements

The MVP must include all three core business models:

1. ready-to-order products
2. AI-assisted product design
3. physical product personalization

The MVP must also include the core supporting systems required to operate them.

This includes:

- authentication
- customer dashboard
- admin panel
- cart
- checkout
- payments
- invoices
- production tracking
- shipping tracking
- file management
- AI design history
- design versioning
- approvals
- personalization workflows
- basic finance management

---

## 34. Deferred Features

The following items are currently not required for the initial implementation:

- PWA
- native mobile application
- direct customer upload of STL / 3MF files
- full accounting software
- Iranian electronic tax invoice integration
- automatic removal of human AI approval
- advanced recommendation engine
- loyalty system

The architecture should avoid preventing these features from being added later.

---

## 35. Open Research Items

The following subjects require additional research before implementation:

### Manufacturing Details

Exact printing technology, machines, materials, colors and manufacturing limitations.

### 3D Generation Provider

Evaluate available providers based on:

- output quality
- printability
- API reliability
- pricing
- supported file formats
- latency
- commercial usage limitations

### AI Price Estimation Engine

Determine a practical pricing model based on actual production data.

### Slicing Infrastructure

Determine which slicer and execution architecture should be used for server-side manufacturing analysis.

### AI Credit Model

Determine how AI operations should be measured and monetized.

---

## 36. Core Design Principle

The system must keep the following concerns separate:

Customer Experience

AI Reasoning

Business Rules

Manufacturing Validation

Pricing

Production

Payments

Keeping these concerns separated is necessary to ensure that an AI mistake cannot directly create an invalid production order.

---

## 37. High-Level System Map

```text
Customer
   │
   ├── Product Catalog
   │      └── Ready Product
   │
   ├── AI Design
   │      ├── Conversation
   │      ├── Design Versions
   │      ├── 3D Generation
   │      ├── Validation
   │      └── Admin Approval
   │
   └── Personalization
          ├── Request
          ├── Quote
          ├── Physical Item
          ├── Inspection
          └── Manual Work

                 │
                 ▼

              Commerce
                 │
          ┌──────┼──────┐
          ▼      ▼      ▼

        Order  Payment Shipping

                 │
                 ▼

             Production

                 │
          ┌──────┴──────┐
          ▼             ▼

       3D Print      Manual Work

                 │
                 ▼

            Quality Control

                 │
                 ▼

              Delivery
```

---

## 38. Document Relationship

This document provides the high-level system definition.

Detailed behavior will be defined in separate blueprint documents.

Future documents should include:

- `domains.md`
- `user-flows.md`
- `order-lifecycle.md`
- `ai-design-flow.md`
- `personalization-flow.md`
- `production-flow.md`
- `pricing-architecture.md`

Implementation-specific decisions belong under:

`docs/architecture/`

Unresolved technical investigations belong under:

`docs/research/`

Architectural decisions belong under:

`docs/adr/`