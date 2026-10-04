# Actors and Permissions Baseline

**Phase:** 00 — Project Discovery & Product Definition  
**Task:** 00.04 — Define actors and permissions baseline  
**Status:** Draft complete — pending repository commit

This document defines the initial actors, responsibilities, privileged operations, and authorization boundaries of the platform.

This is a baseline for future RBAC and permission design. It is intentionally implementation-independent.

---

## 1. Authorization Principles

1. Authorization must be permission-based rather than relying only on hardcoded role names.
2. Roles are convenient bundles of permissions.
3. A user may eventually hold more than one staff role if needed.
4. Customer ownership must be enforced independently from staff roles.
5. Sensitive operations must be auditable.
6. UI visibility is not security. Every protected backend operation must enforce authorization.
7. A role must receive only the permissions required for its responsibilities.
8. Internal production, finance, approval, and audit data must not be exposed to normal customers unless explicitly intended.

---

## 2. Customer

### Purpose

A Customer is the external user who purchases products or requests services.

### Primary Responsibilities

A Customer may:

- manage their own profile;
- authenticate with mobile/OTP;
- manage their own addresses;
- browse public catalog content;
- configure supported Product options;
- manage their Cart;
- complete Checkout;
- pay their own payable amounts;
- view their own Orders;
- view their own OrderItems and customer-facing status history;
- download their own Invoices;
- view their own Shipments and tracking information;
- create AI Design Sessions;
- send messages and reference images in their own AI Design Sessions;
- view their own DesignVersions;
- select and submit their own final AI Design candidate;
- view approval/rejection results for their own AI Designs;
- add an approved AI Design to Cart;
- create PersonalizationRequests;
- upload reference images for their own PersonalizationRequests;
- view their own Quotes;
- pay Deposits, full payments, or RemainingBalances associated with their own requests/orders;
- view their own Notifications.

### Customer Must Not

A Customer must not be able to:

- access another customer's private resources;
- directly change Order, Payment, ProductionJob, Inspection, Shipment, or approval states;
- view internal ProductionTemplate details;
- view internal manufacturing files;
- view internal staff notes;
- approve or reject AI Designs;
- modify locked Quotes;
- modify pricing rules;
- manage printers/materials/manufacturing rules;
- view platform-wide finance;
- access audit logs;
- assign production or design work;
- manage roles/permissions.

---

## 3. Admin

### Purpose

Admin is the primary privileged business/operations role.

### Primary Responsibilities

Admin may:

- manage Categories;
- manage Products;
- manage ProductOptions;
- manage Product media;
- manage ProductionTemplates;
- publish/unpublish Products;
- manage AI Design review queue;
- inspect AI conversations, DesignVersions, validation results, and pricing results when required for review;
- approve AI Designs;
- reject AI Designs;
- request revisions;
- create or adjust final Quotes;
- record reasons for price overrides;
- manage PersonalizationRequests;
- approve/reject PersonalizationRequests;
- request clarification;
- create Personalization Quotes;
- review item-receipt and Inspection results;
- approve revised Quotes;
- manage Orders;
- manage exceptional Order state corrections when authorized;
- view Payments;
- trigger or record approved Refund operations;
- view platform-level financial summaries;
- manage ShippingMethods;
- manage Shipment records;
- manage business settings;
- manage staff users, subject to permission boundaries;
- view AuditLogs;
- view operational diagnostics.

### Admin Privileged Actions

The following Admin actions are especially sensitive and should be permission-gated and audited:

- AI Design approval;
- AI Design rejection;
- price override;
- Quote lock/unlock where supported;
- Refund action;
- manual Order state correction;
- manual Payment reconciliation;
- manufacturing-rule changes;
- role assignment;
- permission changes;
- staff account changes;
- business-setting changes;
- privileged file access;
- audit-log access.

---

## 4. Designer

### Purpose

Designer is a staff role focused primarily on manual design/customization work, especially Personalization.

### Primary Responsibilities

Designer may:

- view PersonalizationRequests assigned to them;
- view relevant customer reference images;
- view accepted PhysicalItem information relevant to their work;
- update status of assigned ManualWorkJobs;
- record internal work notes;
- upload work-progress/reference images where supported;
- mark work as ready for QC;
- respond to rework requests;
- view only the minimum customer information required to perform the assigned task.

### Designer Must Not

Designer must not normally be able to:

- approve their own pricing;
- change locked Quotes;
- issue Refunds;
- manage Payments;
- manage platform finance;
- manage roles/permissions;
- change manufacturing rules;
- publish Products;
- approve AI Designs unless separately granted that permission;
- access unrelated PersonalizationRequests;
- access unrelated customer financial information;
- bypass QC.

---

## 5. Production Operator

### Purpose

Production Operator is a staff role responsible for actual manufacturing and production workflow.

### Primary Responsibilities

Production Operator may:

- view assigned ProductionJobs;
- view relevant ProductionTemplate details;
- access required internal manufacturing files;
- select or record printer/profile usage;
- update ProductionJob state;
- record production start/completion;
- record failures;
- record reprint/rework reasons;
- record actual production notes;
- perform QC actions if separately permitted;
- mark work as ready for QC;
- view only necessary customer/order information for production.

### Production Operator Must Not

Production Operator must not normally be able to:

- alter commercial pricing;
- alter Quotes;
- issue Refunds;
- view platform-wide finance;
- approve AI Designs unless explicitly granted;
- manage customer accounts;
- manage staff roles/permissions;
- publish Products;
- arbitrarily change Order payment state;
- bypass required QC.

---

## 6. Future Optional Roles

The permission model should support future roles without redesigning authorization.

Possible future roles:

### Finance Operator
May:
- view Payments;
- manage reconciliation;
- initiate/record Refunds;
- view financial reports;
- manage cost records.

Should not automatically receive:
- product-management permissions;
- AI approval permissions;
- manufacturing permissions.

### QC Operator
May:
- perform QC;
- pass/fail work;
- request rework;
- record QC notes.

Should be independent from production execution where the business later requires separation of duties.

### Content Manager
May:
- manage public content;
- edit Products/Categories;
- manage media;
- manage SEO/public pages.

### Support Agent
May:
- view limited customer/order information;
- respond to customer issues;
- record internal support notes;
- escalate operational issues.

---

## 7. Permission Naming Convention

Permissions should use stable domain/action names.

Recommended format:

`domain.action`

Examples:

- `catalog.read`
- `catalog.manage`
- `product.publish`
- `order.read`
- `order.manage`
- `order.correct_state`
- `payment.read`
- `payment.reconcile`
- `refund.create`
- `ai_design.read`
- `ai_design.approve`
- `ai_design.reject`
- `ai_design.override_quote`
- `personalization.read`
- `personalization.review`
- `personalization.quote`
- `personalization.inspect`
- `production.read`
- `production.update`
- `production.assign`
- `qc.perform`
- `shipment.manage`
- `finance.read`
- `finance.manage_costs`
- `user.read`
- `staff.manage`
- `role.manage`
- `permission.manage`
- `settings.manage`
- `audit.read`
- `file.internal.read`

The exact permission catalog will be finalized in the implementation phase.

---

## 8. Initial Permission Matrix

| Capability | Customer | Admin | Designer | Production Operator |
|---|---|---|---|---|
| Browse public catalog | Yes | Yes | Yes | Yes |
| Manage own profile | Yes | Own account | Own account | Own account |
| Manage own addresses | Yes | No default need | No default need | No default need |
| Manage Products | No | Yes | No | No |
| Publish Products | No | Yes | No | No |
| View internal ProductionTemplate | No | Yes | Only if needed | Yes |
| Manage Cart | Own only | No default need | No | No |
| Create Order | Own only | No | No | No |
| View customer Orders | Own only | Yes | Limited if assigned | Limited if assigned |
| Change Order state | No | Yes, controlled | No | Limited production-related only |
| View Payments | Own only | Yes | No | No |
| Refund | No | Yes, permissioned | No | No |
| Create AI Design Session | Yes | Optional test/support | No | No |
| View AI Designs | Own only | Yes | Only if assigned/needed | Only if production-related |
| Approve AI Design | No | Yes | No by default | No |
| Override AI Quote | No | Yes | No | No |
| Create PersonalizationRequest | Yes | No | No | No |
| Review PersonalizationRequest | No | Yes | Limited | No |
| Create Personalization Quote | No | Yes | No | No |
| Perform Physical Inspection | No | Yes or delegated | Optional if delegated | Optional if delegated |
| Perform Manual Work | No | Monitor | Yes | No |
| Update ProductionJob | No | Yes | No | Yes |
| Perform QC | No | Yes or delegated | Optional | Optional |
| Manage Shipping | No | Yes | No | Limited operational use |
| View Finance Reports | Own payments only | Yes | No | No |
| Manage Roles/Permissions | No | Yes, highly privileged | No | No |
| View AuditLogs | No | Yes, permissioned | No | No |

---

## 9. Ownership Rules

### Customer-owned resources

The following are customer-owned unless explicitly transferred/shared by business logic:

- Address;
- Cart;
- Order;
- Payment view;
- Invoice;
- DesignSession;
- DesignVersion;
- PersonalizationRequest;
- customer-facing Shipment;
- Notification.

A Customer request must always validate that the authenticated customer owns the requested private resource.

### Staff access

Staff access should be based on:

- explicit permission;
- assignment;
- operational need;
- least-privilege principle.

Assignment-based access should be preferred where broad read access is unnecessary.

---

## 10. Sensitive Data Boundaries

The following data should be considered internal by default:

- ProductionTemplate details;
- manufacturing files;
- printer configuration;
- internal costing;
- margin rules;
- provider API metadata/secrets;
- internal AI prompt/policy configuration where exposure is unsafe;
- internal staff notes;
- audit metadata;
- fraud/security diagnostics;
- staff-only rejection/decision notes where not intended for customer display.

Customer-facing explanations should be stored separately where necessary.

---

## 11. Audited Actions Baseline

At minimum, the following actions should create AuditLog entries:

### Identity / Access
- staff account creation/deactivation;
- role assignment/removal;
- permission change;
- privileged authentication/security change.

### Catalog / Manufacturing
- Product publish/unpublish;
- ProductionTemplate change;
- Material/Printer capability change;
- ManufacturingRule change.

### AI
- AI Design approval;
- AI Design rejection;
- Admin-requested revision;
- Quote override;
- final Quote lock/change.

### Orders / Payments
- manual Order state correction;
- manual Payment reconciliation;
- Refund action;
- exceptional financial correction.

### Personalization
- request approval/rejection;
- Inspection acceptance/rejection;
- revised Quote;
- manual lifecycle correction.

### Platform
- important business-setting change;
- privileged internal-file access where required by policy;
- critical operational override.

---

## 12. Privileged Action Rules

1. Privileged actions must be enforced in the backend.
2. Privileged actions should require a specific permission rather than only an `Admin` role check.
3. High-impact actions should require a reason where appropriate.
4. High-impact actions should create an AuditLog.
5. A user should not silently gain new capabilities because a UI element becomes visible.
6. Future separation-of-duties requirements must be possible without redesigning the whole authorization model.

---

## 13. Initial Role Baseline

### Customer
Default external-user role.

Primary scope:
- own data;
- own purchasing;
- own AI sessions;
- own PersonalizationRequests.

### Admin
Broad operational-management role.

Primary scope:
- business administration;
- approval;
- pricing;
- financial oversight;
- users/settings/audit.

### Designer
Restricted operational role.

Primary scope:
- assigned manual customization/design work.

### Production Operator
Restricted operational role.

Primary scope:
- assigned manufacturing work.

---

## 14. Rule for Future Permission Changes

When adding a new privileged capability:

1. define the new permission;
2. document which role receives it by default;
3. document whether it is auditable;
4. define customer ownership impact;
5. add authorization tests;
6. update this document if the capability changes actor responsibilities.

