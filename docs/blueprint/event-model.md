# Event Model

**Phase:** 01 — System Blueprint & Architecture  
**Task:** 01.05 — Define event model  
**Document Type:** Blueprint  
**Version:** 1.0  
**Status:** Baseline  
**Related Documents:**
- `discovery/business-rules.md`
- `blueprint/domains.md`
- `blueprint/entity-relationships.md`
- `blueprint/state-machines.md`

---

# 1. Purpose

This document defines the baseline event model for the 3D Print Platform.

It establishes:

- what qualifies as a domain/integration event;
- event naming conventions;
- event metadata;
- event ownership;
- sync vs async interaction guidance;
- transactional outbox usage;
- idempotency expectations;
- consumer failure handling;
- event versioning;
- initial event catalog;
- which events are considered critical for durable delivery.

This document does not define the final queue implementation details such as exact BullMQ queue names or Redis topology.

---

# 2. Event Types

The platform uses several kinds of messages. They must not be mixed conceptually.

## 2.1 Domain Event

A Domain Event represents something meaningful that has already happened inside one domain.

Examples:

```text
PaymentSucceeded
DesignApproved
ProductionJobCompleted
ShipmentDispatched
```

Characteristics:

- past-tense meaning;
- emitted only after the owning domain has accepted the state change;
- immutable;
- may have zero or more consumers.

---

## 2.2 Command

A Command requests that something happen.

Examples:

```text
CreateProductionJob
Validate3DAsset
GenerateInvoice
SendNotification
```

Commands are not facts.

A command may fail or be rejected.

---

## 2.3 Job

A Job is an execution unit, usually async.

Examples:

```text
Generate3DModelJob
SliceModelJob
SendSmsJob
GenerateInvoicePdfJob
```

Jobs are implementation-level execution messages.

A Job may eventually produce a Domain Event.

---

## 2.4 External Provider Event / Webhook

External services may emit callbacks or webhook-like messages.

Examples:

```text
ZarinPal callback
3D provider webhook
shipping provider tracking webhook
```

These provider events must be normalized into internal state transitions and internal events.

Provider-native payloads must not become the core domain contract.

---

# 3. Naming Convention

Internal Domain Events use past-tense business language.

Preferred:

```text
OrderCreated
PaymentSucceeded
DesignApproved
ShipmentDelivered
```

Avoid:

```text
DoPayment
HandleOrder
UpdateStatus
SendSomething
```

Commands use imperative language:

```text
CreateOrder
VerifyPayment
CreateProductionJob
GenerateInvoice
```

Jobs may use an explicit `Job` suffix where useful:

```text
Generate3DModelJob
SliceModelJob
SendNotificationJob
```

---

# 4. Event Ownership

The domain that owns the state change owns the event.

Examples:

```text
Payments owns PaymentSucceeded
Orders owns OrderCreated
AI Design owns DesignApproved
Production owns ProductionJobCompleted
Shipping owns ShipmentDelivered
```

Consumers must not redefine the meaning of the event.

---

# 5. Event Envelope

Every durable integration event should use a standard envelope.

Conceptually:

```json
{
  "eventId": "uuid",
  "eventType": "PaymentSucceeded",
  "eventVersion": 1,
  "occurredAt": "ISO-8601",
  "producer": "payments",
  "aggregateType": "Payment",
  "aggregateId": "payment-id",
  "correlationId": "request-or-workflow-id",
  "causationId": "optional-parent-event-or-command-id",
  "actor": {
    "type": "USER|STAFF|SYSTEM|PROVIDER",
    "id": "optional"
  },
  "payload": {}
}
```

Exact serialization may differ, but these concepts should exist.

---

# 6. Required Event Metadata

## eventId

Globally unique identifier for idempotency and traceability.

## eventType

Stable logical event name.

## eventVersion

Schema version for payload evolution.

## occurredAt

When the business event actually happened.

## producer

Owning domain/module.

## aggregateType / aggregateId

The primary entity that changed.

## correlationId

Used to trace one business workflow across:

```text
HTTP request
database transaction
outbox
queue
worker
provider call
consumer
```

## causationId

Optional reference to the command/event that caused this event.

## actor

Who caused the business transition.

Do not include secrets or excessive PII.

---

# 7. Event Payload Principles

Event payloads should contain enough information for consumers to act without exposing internal implementation unnecessarily.

Good payload:

```json
{
  "orderId": "order-123",
  "paymentId": "pay-456",
  "paidAmountToman": 2500000
}
```

Bad payload:

```json
{
  "entirePaymentDatabaseRow": "...",
  "providerSecret": "...",
  "rawCustomerObject": "..."
}
```

Rules:

- include stable IDs;
- include essential business facts;
- avoid secrets;
- avoid giant object snapshots unless there is a specific reason;
- do not expose provider-specific structures as the internal contract;
- consumers may query the owning module/database read API when they need more context.

---

# 8. Synchronous vs Asynchronous Interaction

Not every cross-module operation should be an event.

## 8.1 Prefer synchronous interaction when

- the caller requires an immediate answer;
- validation must complete before the transaction can proceed;
- the operation is fast;
- the business invariant crosses a public module contract.

Examples:

```text
Cart → Catalog.validateConfiguration()
AI Design → Manufacturing.getCapabilities()
Checkout → Orders.createOrder()
Pricing → Manufacturing.getMaterialPricingInputs()
```

---

## 8.2 Prefer asynchronous event/job when

- work is slow;
- external provider is involved;
- operation is retryable;
- execution can continue after request returns;
- side effect should not block core transaction.

Examples:

```text
Design3DGenerationRequested → 3D generation worker
PaymentSucceeded → notification
OrderItemReadyForProduction → production job creation
ShipmentDispatched → SMS
InvoiceRequested → PDF worker
```

---

# 9. Transactional Outbox

Critical integration events must use a Transactional Outbox pattern.

## Problem

Without an outbox:

```text
1. DB transaction commits
2. app tries to publish queue message
3. app crashes
4. queue message is lost
```

Business state is now committed but downstream workflow never starts.

## Required pattern

```text
BEGIN TRANSACTION

update domain state
insert status history
insert outbox event

COMMIT
```

Then a publisher/worker reads unpublished outbox rows and dispatches them.

---

# 10. Outbox Event Record

Conceptually:

```text
OutboxEvent
  id
  eventType
  eventVersion
  aggregateType
  aggregateId
  payload
  correlationId
  causationId
  occurredAt
  publishedAt
  publishAttempts
  lastError
```

Exact fields are implementation details.

---

# 11. Events That Must Be Durable

At minimum, the following should be considered outbox-worthy.

## Payments

- `PaymentSucceeded`
- `PaymentRefunded`
- `PaymentReconciled`

## Orders

- `OrderCreated`
- `OrderItemReadyForProduction`
- `OrderCancelled`

## AI Design

- `Design3DGenerationRequested`
- `DesignSubmittedForReview`
- `DesignApproved`
- `DesignRejected`

## Pricing

- `AIQuoteLocked`
- `AIQuoteExpired`

## Personalization

- `PersonalizationRequestApproved`
- `PersonalizationQuoteAccepted`
- `CustomerItemReceived`
- `InspectionPassed`
- `InspectionFailed`
- `BalancePaymentRequired`
- `PersonalizationReadyForReturn`

## Production

- `ProductionJobCompleted`
- `QCApproved`
- `QCRejected`

## Shipping

- `ShipmentDispatched`
- `ShipmentDelivered`

---

# 12. Idempotency Rules

Every async consumer must assume an event may be delivered more than once.

Consumers should use:

```text
eventId
consumer identity
processed-event store / unique constraint
```

or equivalent idempotent business logic.

Example:

`PaymentSucceeded` delivered twice must not create two ProductionJobs.

---

# 13. Consumer Processing Record

For high-value events, a consumer may keep:

```text
ProcessedEvent
  eventId
  consumerName
  processedAt
```

with uniqueness:

```text
(eventId, consumerName)
```

This is one possible implementation.

The exact strategy may vary by domain.

---

# 14. Retry Strategy

Transient failures should retry with backoff.

Examples:

- temporary SMS outage;
- OpenAI 5xx;
- 3D provider timeout;
- temporary object-storage failure.

Non-retryable failures should not loop forever.

Examples:

- invalid 3D input;
- unsupported manufacturing constraint;
- permanently malformed payload;
- missing required business entity.

The job/consumer must distinguish:

```text
retryable
non-retryable
manual intervention required
```

---

# 15. Dead-Letter / Failed Work

Failed events/jobs that exhaust retries must remain visible.

The operational view should expose:

```text
event/job type
entity ID
attempt count
last error
first failure time
last failure time
manual retry availability
```

A failed message must not silently disappear.

---

# 16. Event Versioning

Event payloads evolve over time.

Use an explicit version:

```text
PaymentSucceeded v1
PaymentSucceeded v2
```

Prefer additive backward-compatible changes where possible.

Breaking changes require:

- new event version;
- compatible consumer migration;
- documented rollout strategy.

---

# 17. Initial Event Catalog

---

# 17.1 Identity Events

## UserRegistered

Producer: Identity

Meaning:
A new User identity was successfully created.

Possible consumers:

- Customer Profile → create default profile
- Notifications → welcome message if later required
- Audit

Payload:

```text
userId
registrationType
```

---

## UserDeactivated

Producer: Identity

Possible consumers:

- session revocation logic
- security/audit projections

Payload:

```text
userId
reasonCode
```

---

## RoleAssigned

Producer: Identity

Payload:

```text
userId
roleId
assignedBy
```

Audit-sensitive.

---

## RoleRemoved

Producer: Identity

Payload:

```text
userId
roleId
removedBy
```

Audit-sensitive.

---

# 17.2 Catalog Events

## ProductPublished

Producer: Catalog

Payload:

```text
productId
publishedAt
```

Possible consumers:

- search/index projection
- cache invalidation

---

## ProductUnpublished

Producer: Catalog

Possible consumers:

- cart invalidation hint
- search/index update

Important:
Existing OrderItems remain valid historical records.

---

## ProductUpdated

Producer: Catalog

Should not carry entire Product record unless necessary.

Possible consumers:

- cache/search projection
- storefront revalidation

---

# 17.3 Order Events

## OrderCreated

Producer: Orders

Meaning:
Durable Order and OrderItems exist.

Payload:

```text
orderId
customerId
totalToman
itemIds[]
```

Possible consumers:

- Payments
- Billing
- Notifications
- analytics/reporting

---

## OrderPlaced

Producer: Orders

Meaning:
Checkout is complete and Order is awaiting/has entered payment flow.

This event may be merged with OrderCreated during implementation if semantics are identical.

---

## OrderItemReadyForProduction

Producer: Orders

Payload:

```text
orderId
orderItemId
sourceType
sourceReferenceId
```

Consumer:

- Production

Critical durable event.

---

## OrderCancelled

Producer: Orders

Possible consumers:

- Payments/refund workflow
- Production cancellation review
- Notifications
- Finance projection

---

## OrderCompleted

Producer: Orders

Possible consumers:

- Finance reporting
- customer notification
- post-launch analytics

---

# 17.4 Payment Events

## PaymentInitiated

Producer: Payments

Payload:

```text
paymentId
payableType
payableId
amountToman
```

Usually informational.

---

## PaymentSucceeded

Producer: Payments

Payload:

```text
paymentId
payableType
payableId
verifiedAmountToman
providerReference
```

Provider secret/raw payload excluded.

Consumers:

- Orders
- Personalization
- Billing
- Notifications
- Finance

Critical durable event.

---

## PaymentFailed

Producer: Payments

Payload:

```text
paymentId
attemptId
failureCategory
```

Do not expose sensitive provider internals.

---

## PaymentRefunded

Producer: Payments

Payload:

```text
paymentId
refundId
refundAmountToman
```

Consumers:

- Orders/Personalization
- Billing
- Finance
- Notifications

---

## PaymentReconciled

Producer: Payments

Meaning:
An ambiguous/manual payment state was resolved.

Audit-sensitive.

---

# 17.5 AI Design Events

## DesignSessionStarted

Producer: AI Design

Payload:

```text
designSessionId
customerId
```

---

## DesignVersionCreated

Producer: AI Design

Payload:

```text
designSessionId
designVersionId
versionNumber
previousVersionId?
```

---

## Design3DGenerationRequested

Producer: AI Design

Payload:

```text
designSessionId
designVersionId
generationRequestId
inputAssetIds[]
```

Consumer:

- 3D generation application/worker

Critical durable event.

---

## Design3DGenerationCompleted

Producer:
Internal 3D provider adapter / AI Design orchestration after provider normalization.

Payload:

```text
designVersionId
generationJobId
assetId
provider
```

Provider is metadata, not business state.

Consumers:

- 3D Processing
- AI Design

---

## Design3DGenerationFailed

Payload:

```text
designVersionId
generationJobId
failureCategory
retryable
```

Consumers:

- AI Design
- AI Credits reconciliation
- Notifications if appropriate

---

## DesignSubmittedForReview

Producer: AI Design

Payload:

```text
designSessionId
designVersionId
customerId
```

Consumer:

- Admin review projection
- Notifications

Critical durable event.

---

## DesignApproved

Producer: AI Design

Payload:

```text
designSessionId
designVersionId
reviewId
reviewedBy
```

Consumers:

- Pricing/quote finalization where needed
- Notifications
- Audit

Critical durable event.

---

## DesignRejected

Producer: AI Design

Payload:

```text
designSessionId
designVersionId
reviewId
reasonCode
```

Customer-visible explanation should be retrieved from AI Design, not necessarily fully embedded.

---

## DesignRevisionRequested

Producer: AI Design

Consumers:

- Notifications
- dashboard projections

---

# 17.6 AI Credits Events

## AICreditsGranted

Payload:

```text
userId
creditTransactionId
amount
reason
```

---

## AICreditsConsumed

Payload:

```text
userId
creditTransactionId
usageRecordId
amount
operationType
```

---

## AICreditsReversed

Used when policy refunds/reverses failed AI/provider usage.

---

# 17.7 3D Processing Events

## ThreeDValidationStarted

Usually operational.

---

## ValidationPassed

Producer: 3D Processing

Payload:

```text
designVersionId
validationResultId
assetId
warningsCount
```

Consumers:

- AI Design
- Pricing orchestration
- Admin review projection

---

## ValidationFailed

Payload:

```text
designVersionId
validationResultId
failureType
ruleViolationCodes[]
```

Must distinguish:

```text
TECHNICAL_FAILURE
MODEL_INVALID
```

---

## SliceCompleted

Producer: 3D Processing

Payload:

```text
designVersionId
sliceResultId
metricsId
profileId
```

Consumers:

- Pricing

---

## SliceFailed

Payload:

```text
designVersionId
sliceJobId
failureType
retryable
```

---

## ManufacturingMetricsCreated

Payload:

```text
designVersionId
metricsId
sliceResultId
```

Pricing retrieves the actual metrics via domain read contract.

---

# 17.8 Pricing Events

## PriceEstimateCreated

Producer: Pricing

Payload:

```text
priceEstimateId
designVersionId
pricingRuleVersionId
estimatedTotalToman
```

Consumers:

- AI Design
- Admin review projection

---

## AIQuoteCreated

Payload:

```text
quoteId
designVersionId
priceEstimateId
amountToman
```

---

## AIQuoteAdjusted

Audit-sensitive.

Payload:

```text
quoteId
previousAmountToman
newAmountToman
adjustedBy
reasonCode
```

---

## AIQuoteApproved

Meaning:
Admin accepted commercial pricing.

---

## AIQuoteLocked

Producer: Pricing

Payload:

```text
quoteId
designVersionId
finalAmountToman
lockedAt
expiresAt?
```

Consumers:

- AI Design → mark purchasable
- Notifications
- Cart validity/read model

Critical durable event.

---

## AIQuoteExpired

Consumers:

- AI Design
- Cart invalidation
- Notifications

---

# 17.9 Production Events

## ProductionJobCreated

Payload:

```text
productionJobId
orderItemId
sourceType
```

---

## ProductionJobAssigned

Payload:

```text
productionJobId
operatorUserId
```

---

## ProductionJobStarted

Consumers:

- Orders → customer fulfillment projection
- Notifications

---

## ProductionJobFailed

Payload:

```text
productionJobId
failureCategory
reworkPossible
```

Consumers:

- Orders
- Admin operational dashboard
- Notifications when customer-visible action is required

---

## ProductionJobReworkRequested

Payload:

```text
productionJobId
qcRecordId?
reasonCode
```

---

## QCApproved

Producer: Production

Payload:

```text
productionJobId
qcRecordId
approvedBy
```

Consumers:

- Orders → OrderItem ready state
- Shipping readiness
- Notifications

Critical durable event.

---

## QCRejected

Consumers:

- Production rework flow
- Orders projection
- Admin dashboard

---

## ProductionJobCompleted

Meaning:
Manufacturing + required QC are complete.

Consumers:

- Orders
- Finance reporting

---

# 17.10 Personalization Events

## PersonalizationRequestSubmitted

Producer: Personalization

Payload:

```text
personalizationRequestId
customerId
```

Consumers:

- Admin review projection
- Notifications

---

## PersonalizationRequestApproved

Critical durable event.

Possible consumers:

- Notifications
- quote workflow

---

## PersonalizationRequestRejected

Payload includes reason code/reference.

---

## PersonalizationQuoteCreated

Payload:

```text
personalizationRequestId
quoteId
amountToman
depositRequiredToman?
```

Consumers:

- Notifications
- Customer Dashboard

---

## PersonalizationQuoteAccepted

Meaning:
Customer accepted commercial quote conditions.

This does not necessarily mean fully paid.

---

## CustomerItemReceived

Producer: Personalization

Payload:

```text
personalizationRequestId
physicalItemReceiptId
receivedBy
```

Consumers:

- Admin/inspection queue
- Notifications

Critical durable event.

---

## InspectionPassed

Payload:

```text
personalizationRequestId
inspectionId
```

Consumer:

- Personalization workflow → manual work readiness
- Notifications

Critical durable event.

---

## InspectionFailed

Payload:

```text
personalizationRequestId
inspectionId
reasonCode
```

Consumers:

- return/cancellation workflow
- Notifications

---

## PersonalizationRequoteRequired

Payload:

```text
personalizationRequestId
inspectionId
reasonCode
```

---

## ManualWorkStarted

Possible consumers:

- dashboard projection
- notifications

---

## ManualWorkCompleted

Consumer:

- QC workflow

---

## BalancePaymentRequired

Payload:

```text
personalizationRequestId
quoteId
remainingBalanceToman
```

Consumers:

- Payments
- Notifications

Critical durable event.

---

## PersonalizationReadyForReturn

Consumers:

- Shipping
- Notifications

Critical durable event.

---

# 17.11 Shipping Events

## ShipmentCreated

Payload:

```text
shipmentId
shipmentType
orderId?
personalizationRequestId?
```

---

## ShipmentDispatched

Producer: Shipping

Payload:

```text
shipmentId
trackingCode?
carrierCode?
dispatchedAt
```

Consumers:

- Orders
- Personalization
- Notifications

Critical durable event.

---

## ShipmentTrackingUpdated

Usually lower-priority operational event.

---

## ShipmentDelivered

Payload:

```text
shipmentId
deliveredAt
```

Consumers:

- Orders
- Personalization
- Notifications

Critical durable event.

---

# 17.12 Billing Events

## InvoiceGenerated

Producer: Billing

Payload:

```text
invoiceId
sourceType
sourceId
storageAssetId
```

Consumers:

- Customer Dashboard
- Notifications if desired

---

## InvoiceGenerationFailed

Operational event.

---

# 17.13 Notification Events

## NotificationDelivered

Usually operational/reporting only.

---

## NotificationFailed

Payload:

```text
notificationId
channel
retryable
failureCategory
```

Must not mutate core business state.

---

# 18. Example Workflow — Ready Product Purchase

```text
HTTP Checkout Command
        ↓
Orders.createOrder()
        ↓
OrderCreated
        ↓
Payments.createPayment()
        ↓
customer pays
        ↓
provider callback
        ↓
Payments.verify()
        ↓
PaymentSucceeded
        ↓
Orders payment consumer
        ↓
OrderItemReadyForProduction
        ↓
Production consumer
        ↓
ProductionJobCreated
        ↓
ProductionJobStarted
        ↓
QCApproved
        ↓
Orders marks item ready
        ↓
Shipping creates shipment
        ↓
ShipmentDispatched
        ↓
Notifications sends SMS
```

---

# 19. Example Workflow — AI Design

```text
Customer requests generation
        ↓
AI Design commits generation request
        ↓
Design3DGenerationRequested
        ↓
3D worker/provider
        ↓
Design3DGenerationCompleted
        ↓
3D Processing
        ↓
ValidationPassed
        ↓
SliceCompleted
        ↓
ManufacturingMetricsCreated
        ↓
Pricing
        ↓
PriceEstimateCreated
        ↓
Customer submits final version
        ↓
DesignSubmittedForReview
        ↓
Admin approves
        ↓
DesignApproved
        ↓
AIQuoteLocked
        ↓
AI Design becomes purchasable
```

---

# 20. Example Workflow — Personalization

```text
PersonalizationRequestSubmitted
        ↓
Admin review
        ↓
PersonalizationRequestApproved
        ↓
PersonalizationQuoteCreated
        ↓
PaymentSucceeded (deposit/full)
        ↓
Await Physical Item
        ↓
CustomerItemReceived
        ↓
InspectionPassed
        ↓
Manual Work
        ↓
QC
        ↓
BalancePaymentRequired (if needed)
        ↓
PaymentSucceeded
        ↓
PersonalizationReadyForReturn
        ↓
ShipmentDispatched
        ↓
ShipmentDelivered
```

---

# 21. Correlation Strategy

One business workflow should preserve a common `correlationId`.

Examples:

## Order workflow

```text
checkout request
OrderCreated
PaymentSucceeded
ProductionJobCreated
ShipmentDispatched
```

may share one order/workflow correlation ID.

## AI workflow

```text
DesignVersion
generation
validation
slicing
pricing
review
```

may share a DesignVersion or generated workflow correlation ID.

Correlation IDs are observability metadata, not domain identity.

---

# 22. Causation Strategy

Example:

```text
PaymentSucceeded event
  causes
OrderItemReadyForProduction event

OrderItemReadyForProduction
  causes
CreateProductionJob command
```

`causationId` helps reconstruct why a downstream action occurred.

---

# 23. External Provider Normalization

## Payment provider

External:

```text
ZarinPal authority/status
```

Internal:

```text
PaymentAttempt
PaymentSucceeded
PaymentFailed
```

## 3D provider

External:

```text
provider-specific queued/generating/completed codes
```

Internal:

```text
QUEUED
PROCESSING
SUCCEEDED
FAILED
```

and internal Domain Events.

## SMS provider

External response remains delivery metadata.

Internal Notification status stays provider-neutral.

---

# 24. Event Security

Events must not contain:

- passwords;
- OTP codes;
- JWT/session tokens;
- API secrets;
- payment credentials;
- unnecessary personal data;
- full sensitive provider payloads.

Sensitive reference data should be fetched by authorized consumers when necessary.

---

# 25. Event Audit Relationship

Not every event is an AuditLog.

Example:

```text
ProductionJobStarted
```

is a business event.

```text
Admin changed locked quote from X to Y
```

should produce both:

- a domain event such as `AIQuoteAdjusted`;
- an AuditLog record.

Audit requirements remain separate from event transport.

---

# 26. Event Retention

Outbox and processed-event retention policies are implementation/operations decisions.

However:

- unpublished outbox events must never be deleted;
- failed durable events must remain diagnosable;
- processed-event deduplication data must be retained long enough to prevent duplicate side effects.

Exact retention periods are defined later.

---

# 27. Event Ordering

Do not assume global event ordering.

Consumers may only rely on ordering where explicitly guaranteed for one aggregate/workflow.

Consumer logic should validate current domain state before applying transitions.

Example:

If an old `ProductionJobStarted` event arrives after a job is already `COMPLETED`, the consumer should not move it backward.

---

# 28. Eventual Consistency

Async consumers introduce eventual consistency.

Examples:

- Payment becomes `SUCCEEDED`;
- Order may update milliseconds/seconds later;
- notification may arrive later still.

UI should tolerate short-lived transitional states.

Core invariants that require immediate consistency should use synchronous/local transactions instead of async events.

---

# 29. Commands That Should Remain Explicit

Important commands/use cases include:

```text
CreateOrder
InitiatePayment
VerifyPayment
SubmitDesignForReview
ApproveDesign
RejectDesign
LockAIQuote
CreateProductionJob
RecordQCResult
SubmitPersonalizationRequest
ApprovePersonalizationRequest
RecordPhysicalItemReceipt
RecordInspection
CreateShipment
```

Commands should not be replaced by generic:

```text
UpdateEntity
SetStatus
```

---

# 30. Recommended Event Infrastructure Shape

Conceptually:

```text
Domain Transaction
      ↓
PostgreSQL Outbox
      ↓
Outbox Publisher
      ↓
BullMQ / Internal Queue
      ↓
Consumer Worker
      ↓
Domain Application Service
```

Not every event needs Redis/BullMQ.

Pure in-process events may be used for low-risk synchronous concerns, but must not be used where losing the event would break a critical workflow.

---

# 31. Critical Delivery Matrix

| Event | Durable Outbox | Async Consumer | Idempotency Required |
|---|---:|---:|---:|
| OrderCreated | Yes | Often | Yes |
| PaymentSucceeded | Yes | Yes | Yes |
| PaymentRefunded | Yes | Yes | Yes |
| Design3DGenerationRequested | Yes | Yes | Yes |
| DesignApproved | Yes | Yes | Yes |
| AIQuoteLocked | Yes | Yes/Optional | Yes |
| OrderItemReadyForProduction | Yes | Yes | Yes |
| QCApproved | Yes | Yes | Yes |
| CustomerItemReceived | Yes | Yes/Optional | Yes |
| InspectionPassed | Yes | Yes | Yes |
| BalancePaymentRequired | Yes | Yes | Yes |
| PersonalizationReadyForReturn | Yes | Yes | Yes |
| ShipmentDispatched | Yes | Yes | Yes |
| ShipmentDelivered | Yes | Yes | Yes |
| NotificationDelivered | No/Optional | No | Usually |
| ProductUpdated | Optional | Optional | Usually |

---

# 32. Testing Requirements

Event-related tests must include:

## Producer tests

- state commit and outbox event are in one transaction;
- payload contains expected IDs/facts;
- event version is correct.

## Consumer tests

- normal processing;
- duplicate delivery;
- missing target entity;
- already-applied state;
- transient failure/retry;
- non-retryable failure.

## Integration tests

- outbox publish flow;
- worker consumption;
- idempotent side effects;
- crash/retry simulation for critical flows.

---

# 33. Anti-Patterns

Avoid:

## 33.1 Generic event

```text
EntityUpdated
```

for all business actions.

It loses semantic meaning.

---

## 33.2 Event as remote method call

Events represent facts, not hidden synchronous RPC.

---

## 33.3 Giant payloads

Do not serialize entire aggregates without need.

---

## 33.4 Provider-specific events as domain contracts

Bad:

```text
ZarinPalStatus100Received
MeshyJobState3
```

Good:

```text
PaymentSucceeded
Design3DGenerationCompleted
```

---

## 33.5 No idempotency

Async delivery must always assume duplicates are possible.

---

## 33.6 Queue as source of truth

Business state belongs in PostgreSQL.

---

# 34. Open Implementation Decisions

Deferred to architecture/implementation:

- BullMQ queue partitioning;
- number of queues;
- outbox polling vs notification strategy;
- event serialization format;
- processed-event table design;
- retry/backoff defaults;
- dead-letter implementation;
- queue observability UI;
- event retention duration;
- whether some low-risk events remain purely in-process.

These choices must preserve the business guarantees defined here.

---

# 35. Completion Criteria

Task `01.05 — Define event model` is complete when:

- Domain Events, Commands, Jobs, and provider callbacks are distinguished;
- event ownership is defined;
- envelope metadata is defined;
- naming/versioning rules are documented;
- sync vs async guidance is explicit;
- Transactional Outbox is defined for critical workflows;
- idempotency and retry rules are documented;
- major initial events across all domains are listed;
- critical delivery events are identified;
- external provider normalization is documented;
- correlation/causation strategy is established;
- event testing requirements are documented.

The next task, `01.06 — Record baseline ADRs`, records the major architectural decisions and their rationale.
