# Security Plan

## Purpose

This document defines the security controls implemented in the system architecture, including:

- Role-based access control
- Tenant isolation
- Encryption in transit and at rest
- Network segmentation
- Audit logging
- Data retention posture
- Notification privacy controls

The goal is to protect operational integrity, billing accuracy, and sensitive scheduling information while maintaining usability for a small healthcare-adjacent organization.

This document describes architectural controls. Workforce policies, device management, and training procedures operate alongside these controls and are assumed to be part of ongoing business governance.

---

# 1. Identity and Role-Based Access Control

## Technical Design

Authentication is handled by **Amazon Cognito (single User Pool)**.

Roles are enforced using:

- Cognito Groups (Dispatcher, Driver, Billing, FacilityAdmin, FacilityUser)
- Custom attributes (`facility_id`, `contact_id`) for tenant scoping
- Server-side authorization checks inside AWS Lambda

Security principles enforced:

- Least privilege
- Server-side enforcement (never UI-only)
- Token validation at API Gateway
- JWT claim validation before database access
- No direct client-to-database communication

Facility users are always scoped by:

TripRequest.facility_id = token.custom:facility_id


FacilityUsers have additional visibility restrictions based on:
- `submitted_by_user_id`
- `contact_id`

Drivers only see assigned trip legs.

Billing roles cannot modify completion data.

Completed records cannot be modified by facility roles.

## What This Means for the Business

Access control is enforced centrally and cannot be bypassed through the interface.

Each user sees only what they are permitted to see. Facilities cannot view other facilities’ data. Drivers cannot access billing records. Billing cannot alter operational completion records.

This reduces internal risk, prevents accidental data exposure, and protects revenue workflows from unauthorized edits.

The system enforces operational boundaries automatically, reducing reliance on memory or manual discipline.

This level of structured role segmentation reflects operational maturity typically seen in larger organizations, without increasing administrative overhead.

---

# 2. Tenant Isolation Enforcement

## Technical Design

Every `TripRequest` must belong to exactly one tenant (facility or individual).

There are no NULL tenant associations.

Tenant identity is derived from validated JWT claims — never from client-supplied request data.

All queries executed by Lambda enforce tenant scoping before accessing the database.

Public intake endpoints are write-only and cannot query existing data.

## What This Means for the Business

Tenant isolation prevents cross-facility visibility and protects private-pay riders from unintended exposure.

It ensures billing clarity and clean grouping by account.

This prevents one of the most common small-business data risks: shared buckets, ambiguous ownership, and accidental mixing of records.

Over time, clean tenant isolation strengthens trust and reduces dispute exposure.

It also protects long-term enterprise value by ensuring operational data remains structured and defensible.

---

# 3. Encryption in Transit

## Technical Design

All external communication uses HTTPS with TLS encryption:

- CloudFront enforces TLS at the edge.
- API Gateway accepts only HTTPS requests.
- Communication between Lambda and RDS occurs within a VPC using secure connections.

No plaintext credentials or unsecured endpoints are exposed publicly.

JWT tokens are signed and validated before use.

## What This Means for the Business

Encryption in transit prevents interception of scheduling data during transmission.

This protects against network-based attacks and ensures that sensitive information cannot be captured through traffic monitoring.

From a business perspective, this reduces liability exposure and aligns with healthcare-adjacent best practices.

It ensures that data moving between user devices and the system remains protected by modern security standards.

---

# 4. Encryption at Rest

## Technical Design

- Amazon RDS (PostgreSQL) uses encrypted storage volumes.
- Amazon S3 buckets use server-side encryption.
- Backups are encrypted automatically.
- Database snapshots inherit encryption.

Encryption keys are managed through AWS-managed Key Management Service (KMS).

No production database is publicly accessible.

## What This Means for the Business

If physical storage were ever compromised, data would remain unreadable without encryption keys.

This protects long-term historical records, billing documentation, and scheduling data.

It reduces exposure risk and supports responsible handling of sensitive operational data.

Even if infrastructure were accessed improperly, encrypted storage prevents direct data extraction.

---

# 5. Network Segmentation and Database Isolation

## Technical Design

The system is deployed inside a Virtual Private Cloud (VPC).

Segmentation includes:

- Public Edge Layer (CloudFront + API Gateway)
- Private Application Subnet (Lambda attached to VPC)
- Private Database Subnet (RDS)

Database controls:

- No public endpoint
- Accepts connections only from Lambda security group
- No inbound internet access
- No direct client communication

RDS Proxy manages database connections and prevents overload during spikes.

## What This Means for the Business

The database cannot be reached from the public internet.

Even if someone discovers an API endpoint, they cannot connect directly to stored data.

This dramatically reduces the attack surface compared to traditional shared-hosting website models.

It prevents a common small-business vulnerability: exposed databases accessible through misconfiguration.

This level of network segmentation signals infrastructure discipline and protects core business data from preventable exposure.

---

# 6. Logging and Audit Controls

## Technical Design

Logging occurs at:

- API Gateway (request metadata)
- Lambda (application logic events)
- Database (transaction and audit context)
- CloudTrail (administrative actions)

Key audit-relevant events include:

- Trip creation
- Cancellation request
- Cancellation approval/denial
- Completion submission
- Billing state transition

Logs avoid storing unnecessary PHI in log payloads.

Logs are retained according to configured lifecycle policies.

## What This Means for the Business

Audit logs provide traceability.

If disputes arise, the system can demonstrate:

- Who made a change
- When it occurred
- What state transition was triggered

This protects the dispatcher from “he said / she said” scenarios and strengthens defensibility.

It also supports long-term documentation integrity and dispute resolution.

Audit clarity increases operational confidence and reduces reputational risk.

---

# 7. Notification Privacy Controls

## Technical Design

Notification workflow:

Main Lambda → SQS → Notification Lambda → SNS / SES


Controls include:

- Decoupled messaging (SQS buffer)
- Retry handling for failed notifications
- Minimal data exposure in SMS messages
- Operational details limited in email
- No clinical context transmitted via SMS

Notification failure does not interrupt trip creation.

## What This Means for the Business

The system does not depend on email or SMS availability to function.

Trip scheduling continues even if notification services are temporarily delayed.

SMS messages contain receipt-level content only, reducing exposure of sensitive information.

This reduces operational fragility and protects privacy.

It also demonstrates thoughtful separation between operational workflow and communication channels.

---

# 8. Data Retention and Record Preservation

## Technical Design

- Relational records are preserved long-term.
- Completed trip data is immutable.
- Cancellations are recorded as workflow events.
- S3 lifecycle policies support retention management.
- No destructive deletion by facility roles.

Retention aligns with long-term billing and dispute needs.

## What This Means for the Business

The system preserves operational history rather than overwriting it.

This supports:

- Billing audits
- Payment reconciliation
- Dispute defense
- Long-term operational transparency

Record preservation strengthens trust and improves organizational continuity.

Over time, structured retention becomes a business asset rather than a liability.

---

# 9. Regulatory Context

This system implements architectural controls consistent with modern security best practices:

- Role-based access control
- Encryption in transit and at rest
- Tenant isolation
- Network segmentation
- Immutable operational records
- Audit logging

Formal regulatory compliance programs (e.g., workforce training, device policy enforcement, documentation policies, and business associate agreements) operate outside the scope of system architecture and remain part of organizational governance.

The architecture supports secure handling of sensitive scheduling data when paired with appropriate operational policies.

---

# Summary

The security posture of this system is structured around:

- Enforced tenant isolation  
- Immutable operational records  
- Role-based segmentation  
- Private network boundaries  
- Encrypted storage and transport  
- Audit-friendly workflows  
- Decoupled notification reliability  

Security is not implemented as a surface feature.

It is embedded into data structure, workflow design, and infrastructure boundaries.

This approach reduces operational risk, protects revenue integrity, and supports long-term business durability while remaining manageable for a small organization.

