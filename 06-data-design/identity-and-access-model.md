# Identity & Access Model (IAM + Roles + Facility Portal)

This document defines how users authenticate, how roles are assigned, and how access is enforced for a non-emergency medical transportation request system with:

- internal staff access (dispatcher, drivers, billing)
- a facility portal (facility admins + facility users)
- a public/guest intake form (no login)

This model is designed for a small business environment but uses patterns that scale safely (least privilege, server-side enforcement, and facility scoping).

---

## Goals

- Prevent unauthorized access to trip, patient, and billing data
- Support facility-facing submission **without** exposing internal data
- Support facility portal visibility rules:
  - FacilityAdmin can see all facility requests
  - FacilityUser can see requests they submitted **and** requests where they are listed as the contact
- Support guest intake with minimal friction (accessible UX)
- Preserve billing safety:
  - facilities cannot edit completed trips
  - facilities cannot modify internal completion/billing flags
- Keep implementation realistic for AWS (Cognito + API Gateway + Lambda + RDS)

---

## Architecture Decision Summary

### Identity Provider
- **Amazon Cognito User Pool (single pool)**
- **Roles stored as Cognito Groups**
- **Business scoping stored as Cognito custom attributes**
- **All authorization enforced server-side in Lambda (defense-in-depth)**

### Facility Membership Model (v1)
- **One facility per user**
- Multi-facility users are treated as a future enhancement (explicitly deferred)

Rationale:
- simplifies authorization logic
- reduces risk of cross-facility data exposure
- aligns to real workflow (facility is the billing account)

---

## User Types and Access Modes

The system supports three access modes:

1. **Internal authenticated users**
   - Dispatcher
   - Driver
   - Billing

2. **Facility authenticated users**
   - FacilityAdmin
   - FacilityUser

3. **Public / guest intake (no authentication)**
   - Anyone can submit a request
   - Treated as untrusted input
   - Must not reveal internal data

---

## Cognito Configuration

### Cognito Groups (role layer)
The user’s role is determined from Cognito Group membership:

- `Dispatcher`
- `Driver`
- `Billing`
- `FacilityAdmin`
- `FacilityUser`

Groups answer: **what kind of user is this?**

### Cognito Custom Attributes (business scope layer)
Facility portal users include custom attributes used for scoping:

- `custom:facility_id` (required for facility users)
- `custom:contact_id` (required for FacilityUsers; optional for FacilityAdmins)
- `custom:contact_phone` (optional, for migration support)
- `custom:contact_email` (optional, for migration support)

Custom attributes answer: **what account/context does this user belong to?**

> Internal users do not require `custom:facility_id` (may be null).

---

## Authorization Principles

### 1) Least Privilege
Each role can view/modify only what they need.

### 2) Server-side Enforcement
The backend (Lambda) enforces all authorization, even if the UI hides features.

### 3) Facility Scoping
Facility-facing users are always scoped by `facility_id`.

### 4) No Facility Editing of Completed Data
Facility roles cannot change completed trip documentation or billing flags.

---

## Role Definitions (Capabilities)

### Internal Roles

#### Dispatcher
Can:
- create/modify trip requests
- schedule and assign drivers
- split round trips into legs
- correct intake errors
- perform patient matching/merge actions
- approve/deny cancellations (if you choose an approval flow)
- view all system data

Cannot:
- (none in scope; dispatcher is full-access internal role)

#### Driver
Can:
- view assigned trip legs
- view required trip details for execution
- record completion data per leg (odometer/times/signature)
- mark leg status (completed/cancelled/no-show)

Cannot:
- view unassigned trips
- modify trip request intake details (facility/patient fields)
- access billing views

#### Billing
Can:
- view completed legs
- group by facility and patient
- manage billing workflow flags (ready to bill, billed, paid)
- export billing summaries

Cannot:
- modify driver-entered completion details
- modify trip scheduling/assignment

---

### Facility Roles (restricted for billing safety)

#### FacilityUser
Can:
- submit new trip requests
- view requests they submitted
- view requests where they are the listed contact
- cancel trips prior to execution (request cancellation action)
- submit dispute/clarification requests

Cannot:
- edit trip details after submission
- delete records
- modify completed trips
- modify driver completion data
- modify billing status flags
- perform patient merges directly

#### FacilityAdmin
Can:
- view all requests for their facility
- request cancellation prior to execution
- submit dispute/clarification requests
- manage facility portal users (add/remove/promote within the same facility)

Cannot:
- modify completed trips
- modify driver completion data
- change billing flags
- merge patients directly
- delete trip records

---

## Facility Portal Visibility Rules (Formal)

All facility portal access is scoped to:

- `TripRequest.facility_id = token.custom:facility_id`

Then visibility differs by role.

### FacilityAdmin visibility
FacilityAdmin may view all facility requests:

- `TripRequest.facility_id = current_user.facility_id`

### FacilityUser visibility (your chosen model)
FacilityUser may view a request when BOTH:

**Facility match**
- `TripRequest.facility_id = current_user.facility_id`

AND either:

**Submitted-by rule**
- `TripRequest.submitted_by_user_id = current_user.user_id`

OR

**Contact rule**
- `TripRequest.contact_id = current_user.contact_id`

> This ensures FacilityUsers can see:  
> - their own submissions  
> - trips where they are listed as the contact  
> even if submitted by phone/dispatcher.

---

## Guest Intake UX Rules (No Login)

Guest intake is supported for accessibility and one-time requesters.

Rules:
- no login
- no account creation
- no passwords
- one clear form with large buttons and simple sectioning
- confirmation screen: “We received your request.”

Optional:
- offer “Enter email/phone to receive a copy” (but do not require)

### Mandatory notifications (operational requirement)
Every submitted request triggers:
1. Internal copy to dispatcher inbox (full details)
2. Receipt to the requester’s provided contact channel (privacy-safe)

**SMS (safe default):** receipt only  
Example:
- “We received your transportation request for [SERVICE DATE]. Star Mobility will confirm shortly.”

**Email:** may include limited operational details (date/time), but avoid sensitive context.

---

## Guest → FacilityUser Conversion

If a facility wants portal access later, guest requests can be linked safely.

### Conversion steps
1. Dispatcher creates or verifies Facility record
2. Dispatcher creates FacilityContact record (if not already present)
3. Dispatcher creates Cognito user and assigns:
   - Group: FacilityUser or FacilityAdmin
   - `custom:facility_id`
   - `custom:contact_id`

### Linking historical guest requests
Historical TripRequests can be linked to the new FacilityContact by:
- setting `TripRequest.contact_id` once verified
- optionally setting `submitted_by_user_id` for future filtering (not required)

---

## Data Model Requirements (ERD Impact)

This Identity & Access model implies the database needs a minimal concept of:

### A) Portal users (system representation)
Even if Cognito is the identity provider, the database must reference user identity for:
- “submitted by” visibility
- audit logs
- cancellation/dispute workflow tracking

**Minimum fields referenced on TripRequest**
- `submitted_by_user_id` (nullable)

**Recommended**
- `submitted_by_role` (optional snapshot)
- `created_via` enum (guest, facility_portal, phone_dispatcher)

### B) Facility portal linkage
TripRequest should include:
- `facility_id` (required)
- `contact_id` (nullable but preferred)

### C) Cancellation and dispute workflow (to prevent destructive edits)
Instead of allowing facilities to delete or edit records, capture requests as separate records:

- `CancellationRequest`
- `DisputeRequest`

(Entities can be added later; for capstone design, documenting intent is sufficient.)

---

## Threats Addressed by This Model

### Cross-facility data exposure
Prevented by `facility_id` scoping on every facility request query.

### Facility deleting trips to avoid billing
Prevented by:
- no delete permission for facility roles
- cancellation as a request/action with audit log
- no editing completed trips

### Public probing for PHI
Prevented by:
- no autosuggest
- no lookup endpoints in public intake
- public can only create new TripRequest submissions

---

## Deferred Decisions (Explicit)

The following are intentionally deferred until post-capstone implementation:

- Multi-facility membership per user (UserFacilityMembership table)
- Full HIPAA compliance program (policies, BAAs, audits)
- Advanced MFA enforcement rules
- Automated patient identity matching using DOB/insurance identifiers
- Full dispute/cancellation workflow UI

---

## Summary

This Identity & Access model is intentionally practical and secure:

- Single Cognito User Pool
- Roles via Cognito Groups
- Facility scoping via custom attributes
- Guest intake allowed without login
- Facility portal supports admin vs user visibility
- Facilities can request cancellations and disputes without editing billing-critical records
- Authorization is enforced server-side in Lambda for safety and auditability
