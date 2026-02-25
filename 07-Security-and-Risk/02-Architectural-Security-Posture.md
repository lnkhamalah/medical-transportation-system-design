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

The system is intentionally designed so that:

- A single compromised account cannot expose unrelated tenant data
- Infrastructure disruption does not compromise transaction durability
- Abuse at the edge does not escalate into database or notification instability
- Billing-critical records cannot be retroactively manipulated

This approach reflects infrastructure-grade operational discipline while remaining proportionate to the scale of the organization.

Security is not treated as a feature.  
It is enforced as a structural property of the system.

The result is a durable, abuse-aware, and defensible operational platform aligned with long-term business continuity and enterprise value.
