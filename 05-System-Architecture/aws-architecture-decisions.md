# AWS Architecture Decisions (Design-Level)

This document explains the AWS services selected for the Non-Emergency Medical Transportation system and why each choice supports business realism, security boundaries, and long-term maintainability.

The design goal is a believable architecture that a small organization can operate: minimal infrastructure management, clear trust boundaries, and a relational database foundation.

---

## Architecture goals

- Support **public guest intake** without revealing internal data (no PHI disclosure via lookups).
- Support **facility portal access** (FacilityUser / FacilityAdmin) with strict facility scoping.
- Support **internal operational access** (Dispatcher / Driver / Billing) with role-based controls.
- Preserve data integrity for billing and disputes (auditability, no silent overwrites).
- Use a **relational database** for joins, grouping, and long-term reporting.
- Keep operating overhead low (serverless where reasonable).

---

## High-level components

### 1) Frontend (static web UI)
**Services**
- Amazon S3 (static website hosting)
- Amazon CloudFront (CDN + TLS termination)
- Amazon Route 53 (DNS)

**Why this fits**
- Low cost and low maintenance.
- CloudFront provides HTTPS, caching, and a safer public edge.
- Separates UI deployment from backend access controls.

---

### 2) API entry layer (controlled access)
**Services**
- Amazon API Gateway (REST API)

**Why this fits**
- Single front door for both public and authenticated API calls.
- Enables authentication enforcement and request routing.
- Supports throttling and basic abuse control.

---

### 3) Application layer (business logic)
**Services**
- AWS Lambda

**Why this fits**
- No server management for a small business.
- Scales automatically with demand spikes.
- Central place to enforce authorization and business rules:
  - trip creation + validation
  - round-trip → leg generation
  - facility scoping rules
  - billing grouping logic
  - patient matching/merge workflows (internal only)

---

### 4) Database layer (relational system of record)
**Services**
- Amazon RDS (PostgreSQL preferred)

**Why RDS (not DynamoDB)**
- The data model is relational (Facility → TripRequest → TripLeg).
- Billing and reporting require joins and grouping by facility/patient/month.
- Referential integrity is important for dispute defensibility.

**Core controls**
- Private access only (no public endpoint exposure).
- Encryption at rest enabled.
- Automated backups enabled.

---

### 5) Identity and access
**Services**
- Amazon Cognito User Pool

**Design choice**
- Single Cognito User Pool for all authenticated users.
- Roles stored as Cognito Groups.
- Facility scoping stored as custom attributes (e.g., facility_id, contact_id where applicable).
- All authorization enforced server-side in Lambda (defense in depth).

**User categories**
- Internal: Dispatcher, Driver, Billing
- Facility portal: FacilityAdmin, FacilityUser
- Public/guest intake: no authentication (create-only)

---

### 6) Storage (snapshots and exports)
**Services**
- Amazon S3 (private bucket)

**Use cases**
- PDF exports (billing packets)
- form snapshot archives
- signature image storage (future enhancement only)

**Core controls**
- Block public access.
- Encryption at rest enabled.
- Access restricted to backend roles only.

---

### 7) Notifications (submission receipts + dispatcher copy)
**Services**
- Amazon SNS (SMS receipt notifications)
- Amazon SES (email receipts and internal copies)

**Business requirement supported**
- Every request submission sends a confirmation to the requester (receipt-level content).
- Every request submission sends a full copy to internal dispatcher/billing inbox (trusted path).

**Privacy default**
- SMS contains a receipt message only (avoid PHI/clinical details).
- Email may include operational details but remains conservative by design.

---

### 8) Logging and audit
**Services**
- Amazon CloudWatch (application logs)
- AWS CloudTrail (AWS account-level audit)

**Rules**
- Log metadata, not PHI payloads.
- Track create/update actions on dispute-sensitive records (TripRequest/TripLeg) and billing flags.

---

## Trust boundaries (what makes this secure and professional)

### Public guest intake (untrusted)
- Can submit new trip requests only.
- No read access.
- No autosuggest or patient lookup endpoints exposed publicly.

### Facility portal (trusted but limited)
- FacilityUser sees: requests they submitted OR where they are the contact.
- FacilityAdmin sees: all facility requests.
- Facility roles can cancel pre-execution and submit disputes, but cannot edit completed trips or billing flags.

### Internal operations (most trusted)
- Dispatcher schedules and assigns drivers.
- Drivers document completion per leg.
- Billing groups trips and manages billing workflow flags.

---

## Summary

This AWS design is intentionally minimal but realistic:

- Static frontend with secure backend APIs
- Serverless business logic for low operational overhead
- Relational database foundation for integrity and billing workflows
- Cognito-based identity with strict server-side authorization
- Clear separation between public intake, facility portal, and internal operations
