# Architectural Security Posture

## Purpose

This document defines the security controls, governance mechanisms, and architectural safeguards implemented in the transportation platform.

It consolidates:

- Identity and access control
- Encryption standards
- Network isolation
- Abuse mitigation controls
- Notification governance
- Data retention strategy
- Audit and logging controls
- Billing workflow integrity controls
- Immutable record protection

The objective is to protect operational integrity, billing reliability, tenant isolation, and sensitive scheduling information while maintaining usability for a small healthcare-adjacent organization.

This document defines architectural controls. Workforce policies, device management, and operational training function alongside these controls as part of broader governance.

---

# 1. Security Objectives

The system is designed to:

1. Prevent unauthorized access to trip, tenant, and billing data  
2. Enforce strict role-based access control  
3. Preserve tenant isolation  
4. Protect data in transit and at rest  
5. Prevent destructive edits and silent record manipulation  
6. Mitigate automated abuse and denial-of-service attempts  
7. Preserve outbound notification reputation and operational continuity  
8. Maintain auditable records for dispute and billing defensibility  
9. Enforce controlled lifecycle state transitions  
10. Protect billing outputs from unauthorized or silent modification  

Security is implemented as a layered model spanning:

- Edge protection  
- Application enforcement  
- Network isolation  
- Data-layer safeguards  
- Operational governance  

---

## 1.1 Threat Model Overview

The architectural security posture is designed in response to realistic threat categories for a healthcare-adjacent operational system:

- **External automated abuse** (bot-driven intake flooding, denial-of-service attempts)
- **Cross-tenant data exposure**
- **Unauthorized internal access**
- **Credential misuse**
- **Infrastructure failure**
- **Outbound notification reputation degradation**
- **Accidental destructive edits impacting billing integrity**
- **Silent modification of completed or billed records**
- **Operational-to-billing mismatch caused by weak workflow controls**

Controls described in this document are intentionally mapped to these threat categories.

Security is not implemented as a single barrier, but as layered containment designed to prevent escalation across trust boundaries.

---

# 2. Identity and Access Control

## Identity Provider

The system uses:

- **Amazon Cognito (Single User Pool)**

Cognito provides:

- Secure authentication
- Signed JWT token issuance
- Role grouping
- Custom attribute support for tenant scoping

---

## Role-Based Access Control (RBAC)

Roles include:

- Dispatcher
- Driver
- Billing
- FacilityAdmin
- FacilityUser

Role permissions are enforced:

- In Cognito (group membership)
- In API Gateway (JWT validation)
- In Lambda (server-side authorization logic)

Authorization decisions are derived exclusively from validated token claims.

Client-supplied role or tenant identifiers are never trusted.

---

## Tenant Scoping Enforcement

Every `TripRequest` must include exactly one `facility_id` (tenant).

Tenant access is enforced server-side:

```sql
TripRequest.facility_id = token.custom:facility_id
```

There are no NULL tenant states and no shared “unscoped” records.

FacilityUser visibility is further restricted by:
- `submitted_by_user_id`
- `contact_id`

Facility roles must never see data outside their own tenant scope, even if names or trip details appear similar.

---

# 3. Workflow Integrity and Record Immutability

## Controlled State Transitions

Operational and billing records are protected through controlled lifecycle state transitions.

Examples include:

- `scheduled → assigned → in_progress → completed`
- `scheduled → cancelled`
- `scheduled → no_show`
- `ready_to_bill → billed → paid`

Transitions are:

- Explicit
- Role-restricted
- Application-enforced
- Audit-logged

No role may arbitrarily overwrite state without validation.

## Immutable Completion Records

Once trip completion data is submitted:

- Drivers cannot edit it
- Facility users cannot edit it
- Billing users cannot alter it
- Dispatcher cannot silently overwrite it

This protects dispute defensibility and ensures that billing logic is derived from stable operational truth.

## Billing Output Protection

Billing-ready output may be reviewed and, where allowed, edited through controlled workflows, but:

- edited output must not overwrite original operational records
- approval / edit actions must be logged
- exported or billed records must not be changed silently

This supports a human-in-the-loop billing model while preserving accountability.

---

# 4. Encryption Standards

## Encryption in Transit

All external communication uses HTTPS with TLS encryption.

Controls include:

- CloudFront TLS enforcement at the edge
- API Gateway HTTPS-only access
- Signed and validated JWT tokens
- Secure service communication within the VPC

No plaintext credentials or open public endpoints are permitted.

## Encryption at Rest

At-rest encryption applies to:

- Amazon RDS storage volumes
- RDS backups and snapshots
- Amazon S3 object storage

Encryption is managed through AWS-managed KMS-backed services.

These controls reduce exposure risk if infrastructure storage is ever accessed improperly.

---

# 5. Network Isolation and Trust Boundaries

## Trust Zones

The platform enforces three trust zones:

1. **Public Internet (untrusted)**  
2. **Application Layer (controlled compute)**  
3. **Data Layer (restricted persistence)**  

## Segmentation Controls

The system uses:

- CloudFront and API Gateway as controlled public entry points
- Lambda in controlled compute boundaries
- RDS in private subnets
- No direct client-to-database connectivity
- RDS Proxy for controlled database access patterns

This ensures that even if public endpoints are attacked, core stored data remains isolated behind multiple barriers.

---

# 6. Edge Protection and Abuse Mitigation

Public intake is the main external attack surface.

Controls include:

- AWS WAF
- API Gateway throttling
- Per-IP request controls
- CAPTCHA for unauthenticated intake
- Payload size limits
- Schema-level validation before application execution

These controls are designed to:

- prevent bot-driven form flooding
- reduce denial-of-service risk
- prevent write amplification into the database
- protect notification services from abuse-driven flooding

This is especially important because abuse at intake could otherwise degrade operational reliability and notification reputation.

---

# 7. Notification Governance

Notification workflows are intentionally decoupled from primary trip creation and database transactions.

Architecture:

`Main Lambda → SQS → Notification Lambda → SNS / SES`

Controls include:

- No direct client access to SNS or SES
- IAM-restricted access to queues and notification services
- Private connectivity through VPC endpoints
- Retry logic and buffered delivery

Notification content is segmented:

- requesters receive limited confirmation content
- dispatcher receives fuller operational detail
- canonical intake snapshots may be archived in encrypted S3

This preserves operational continuity while reducing unnecessary external exposure of sensitive information.

---

# 8. Logging, Audit, and Defensibility

Logging occurs at:

- API Gateway
- Lambda
- Database audit layer
- CloudTrail

Audit-relevant events include:

- Trip creation
- Assignment
- Completion submission
- Cancellation request / approval
- Billing state transition
- Billing approval
- Billing edit / correction
- Export / invoicing event

Logs intentionally exclude unnecessary PHI in payload bodies.

Audit logs are used to support:

- dispute resolution
- billing defensibility
- operational accountability
- forensic review

This ensures that key business events are attributable and historically traceable.

---

# 9. Retention and Preservation Strategy

The system is designed to preserve business-critical history.

Controls include:

- long-term retention of relational records
- immutable completed trip records
- workflow-event recording for cancellations and disputes
- retention-friendly S3 lifecycle policies
- no destructive deletion by facility roles

This allows the business to preserve records necessary for:

- billing review
- invoice defense
- reconciliation
- historical operational reference

Retention is treated as a governance and defensibility requirement, not merely a storage setting.

---

# 10. Governance Posture

This architectural security posture is designed to enforce structural integrity across identity, data, workflow, and infrastructure layers.

Security is embedded into:

- Data modeling (tenant isolation and immutability)
- Workflow design (controlled state transitions and cancellation governance)
- Network boundaries (private subnets and segmented trust zones)
- Edge protection (WAF, throttling, CAPTCHA)
- Notification governance (rate controls and content segmentation)
- Audit logging and retention strategy
- Billing workflow controls (review, approval, export protection)

This design assumes that operational governance outside the system — such as training, policy, and device management — complements these controls.

---

# Executive Summary

This architectural security posture is designed to enforce structural integrity across identity, data, workflow, and infrastructure layers.

Security is embedded into:

- Data modeling (tenant isolation and immutability)
- Workflow design (controlled state transitions and cancellation governance)
- Network boundaries (private subnets and segmented trust zones)
- Edge protection (WAF, throttling, CAPTCHA)
- Notification governance (rate controls and content segmentation)
- Audit logging and retention strategy
- Billing workflow controls (review, approval, export protection)

The system is intentionally designed so that:

- A single compromised account cannot expose unrelated tenant data
- Infrastructure disruption does not compromise transaction durability
- Abuse at the edge does not escalate into database or notification instability
- Billing-critical records cannot be retroactively manipulated
- Invoice-ready outputs cannot be silently changed without accountability

This approach reflects infrastructure-grade operational discipline while remaining proportionate to the scale of the organization.

Security is not treated as a feature.  
It is enforced as a structural property of the system.

The result is a durable, abuse-aware, and defensible operational platform aligned with long-term business continuity and enterprise value.
