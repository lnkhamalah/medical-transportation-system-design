# Security Model

## Security goals

This system handles transportation requests that may include personally identifiable information (PII) and, depending on the context, may involve protected health information (PHI). The security model is designed to:

- prevent unauthorized access to trip and patient-related data
- enforce role-based access control (internal + facility-facing roles)
- minimize PHI/PII exposure to external parties (especially from public intake)
- preserve auditability for billing and dispute-sensitive changes
- protect data in transit and at rest

This is a design-level security model intended to guide implementation decisions.

---

## Identity and authentication

### Identity provider
**Amazon Cognito User Pools** provides authentication for users who access protected views (internal staff and facility portal users).

- Users authenticate using email + password.
- Optional MFA (SMS or authenticator) may be enabled later for internal roles.
- Cognito returns a signed token set (JWT) after successful login.

### Authentication scope
The system supports three access modes:

1. **Internal authenticated users** (always authenticated)
   - Dispatcher/Owner
   - Driver
   - Billing/Admin

2. **Facility authenticated users** (authenticated portal access)
   - FacilityAdmin
   - FacilityUser

3. **Public/guest intake** (no authentication)
   - Anyone can submit a request
   - Public intake is treated as *untrusted input* and must not reveal internal data

---

## Authorization and roles

Authorization is enforced using role claims in authenticated tokens and enforced again server-side (defense in depth).

### Internal roles
- **Dispatcher**: full access to requests, scheduling, corrections, assignment, and patient matching/merge actions.
- **Driver**: access only to trip legs assigned to the driver and the trip details required for execution.
- **Billing**: access to completed legs, grouping views, and billing workflow flags.

### Facility roles (corrected authority boundaries)

Facility roles are intentionally limited to prevent unauthorized modification of operational or billing-sensitive records.

#### FacilityUser
Can:
- submit new trip requests
- view trips they submitted
- view trips where they are the listed contact
- cancel trips prior to execution
- submit dispute or clarification requests

Cannot:
- edit trip details after submission
- modify completed trips
- change billing status
- merge patients
- delete records

#### FacilityAdmin
Can:
- view all trips for their facility
- cancel trips prior to execution
- submit dispute or clarification requests
- manage facility portal users (add/remove/promote)

Cannot:
- modify completed trips
- edit driver-entered completion data
- change billing flags
- merge patients directly
- delete trip records
### Principle: least privilege
Each role can only view or modify what is required for its job. Facility roles do **not** receive access to internal patient records or internal autosuggest features.

---

## Facility portal access rules (explicit)

Facility access is scoped to **facility_id** and **contact identity**. The goal is to support real facility operations without exposing other facilities’ data.

### FacilityAdmin visibility
A FacilityAdmin may view all requests for their facility:

- `TripRequest.facility_id = current_user.facility_id`

### FacilityUser visibility (your chosen model)
A FacilityUser may view requests if either condition is true:

1. **Submitted-by rule**
   - `TripRequest.submitted_by_user_id = current_user.user_id`

2. **Contact-matching rule**
   - User is the designated contact for the request, even if the request was created by phone/dispatcher.
   - Matching is performed by *linked contact identity* (preferred) or by verified contact channel.

Preferred match:
- `TripRequest.contact_id = current_user.contact_id`

Fallback match (only if needed for migration from guest/phone flows):
- `TripRequest.contact_phone_used = current_user.phone` and/or `TripRequest.contact_email_used = current_user.email`

> Implementation note: the fallback match should only be enabled when the facility user’s contact channel has been verified and linked to the facility to prevent impersonation.

---

## Request and token flow (trusted path)

1. Authenticated user logs in through Cognito.
2. Cognito returns ID token + access token (JWT).
3. Frontend includes the JWT on backend requests:
   - `Authorization: Bearer <token>`
4. API Gateway validates the token (Cognito authorizer).
5. API Gateway forwards validated requests to Lambda.
6. Lambda enforces:
   - role checks (internal vs facility roles)
   - facility scoping rules
   - input validation
7. Lambda connects to RDS via private networking.
8. Response returns to the frontend.

Key point: **Protected APIs are never accessible without a valid token.**

---

## Public intake security (untrusted path)

Public/guest intake exists to support accessibility and one-time requesters. Because this path is unauthenticated, it must be designed to avoid disclosure of internal data.

Controls:

- No autosuggest for patient names or internal facility/patient history
- No endpoints that reveal existing patient records
- Public intake can only create a new TripRequest submission payload
- Dispatcher reviews and links to internal records later

This prevents probing attacks (e.g., typing names to discover who uses the service).

---

## Mandatory confirmation notifications (receipt, not disclosure)

Every request submission generates confirmations:

- Dispatcher/internal inbox receives the full request content (trusted).
- Requester receives a receipt confirmation via the provided contact channel.

### SMS default (privacy-safe)
SMS content is limited to a receipt message (avoid PHI/clinical detail):

- “We received your transportation request for SERVICE DATE. Star Mobility will confirm shortly.”

### Email (limited detail)
Email may include operational details (date/time pickup) but should avoid sensitive context. This is intentionally conservative because email forwarding and shared inboxes are common.

All notifications should be logged (delivery attempted/sent/failed) for operational accountability.

---

## Data protection controls

### Encryption in transit
- HTTPS enforced for all client traffic (TLS)
- API Gateway requires HTTPS
- Database connections use TLS where supported

### Encryption at rest
- RDS encryption enabled
- S3 buckets for PDFs/snapshots encrypted (SSE-S3 or SSE-KMS)
- Backups encrypted by default

---

## Network-level security boundaries

### Trust zones
1. **Public internet** (untrusted)
2. **API entry layer** (token-validated requests only)
3. **Application compute** (Lambda)
4. **Private data layer** (RDS; no public access)

### Database isolation
- RDS has no public endpoint
- RDS accepts connections only from backend security context
- No direct client access is possible

---

## Logging and auditability

### What gets logged (avoid PHI in logs)
- Cognito authentication events (login success/failure)
- API request metadata (endpoint, timestamp, user role), not raw request payloads containing PHI/PII
- Backend events for create/update actions affecting:
  - TripRequest
  - TripLeg
  - billing status flags

### Audit-safe rules
Dispute-sensitive fields must remain defensible:

- TripRequest stores the submitted snapshot fields as “what was submitted”
- Edits are tracked (who/when/what changed) rather than silently overwritten
- Billing status changes are tracked for accountability

---

## Common threat scenarios and defenses

### Scenario: Public user tries to discover patient names
Defense:
- no autosuggest
- no patient lookup endpoints exposed publicly
- patient linking happens only internally

### Scenario: FacilityUser attempts to view other facility data
Defense:
- strict facility_id scoping
- contact-based visibility rules
- server-side authorization enforcement

### Scenario: Driver attempts to view trips not assigned
Defense:
- role-based authorization + filtering by assignment
- backend verifies driver assignment before returning data

### Scenario: Credential compromise
Defense:
- optional MFA for internal roles
- token expiration
- least privilege reduces blast radius

---

## Summary

Security is enforced through layered controls:

- authentication (Cognito) for protected roles
- authorization (role claims + server-side enforcement)
- facility scoping (facility_id + submitted-by/contact matching rules)
- network isolation (private data layer)
- encryption (in transit + at rest)
- logging/auditability for accountability

The model is intentionally practical for a small organization while aligning with healthcare-adjacent expectations.
