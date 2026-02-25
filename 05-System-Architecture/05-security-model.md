# Security Model

## Security Goals

This system handles transportation requests that may include personally identifiable information (PII) and, depending on context, protected health information (PHI). The security model is designed to:

- Prevent unauthorized access to trip and patient-related data  
- Enforce strict role-based access control (internal and facility-facing roles)  
- Protect operational and billing integrity  
- Minimize PHI/PII exposure through public intake  
- Preserve auditability for dispute-sensitive and billing-sensitive actions
- Prevent automated abuse from degrading operational reliability or notification reputation    
- Protect data in transit and at rest
- Enforce encryption using TLS for all external and internal service communication  
- Ensure database and object storage encryption at rest using managed encryption keys

The application connects to the database exclusively through Amazon RDS Proxy to:

- Prevent connection exhaustion during Lambda concurrency spikes
- Centralize credential management
- Reduce database exposure risk
    

This is a layered security model designed for a small healthcare-adjacent organization.

---

## Identity and Authentication

### Identity Provider

**Amazon Cognito User Pools** provides authentication for all protected system access.

- Users authenticate using email + password  
- Optional MFA may be enabled for internal roles  
- Cognito returns signed JWT tokens upon successful authentication  

Public intake does **not** use authentication and is treated as untrusted input.

---

## Authentication Scope

The system supports three access categories:

### 1. Internal Authenticated Users
- Dispatcher / Owner  
- Driver  
- Billing  

### 2. Facility Authenticated Users (Portal Access)
- FacilityAdmin  
- FacilityUser  

### 3. Public / Guest Intake (Unauthenticated)
- Anyone may submit a transportation request  
- No login required  
- No patient lookup or data exposure permitted  

Public intake is strictly write-only.

Public endpoints are protected by layered edge defenses:

- AWS WAF with rate-based rules and IP reputation filtering
- API Gateway throttling and burst limits
- CAPTCHA enforcement for unauthenticated intake submissions

These controls mitigate automated abuse, bot-driven form flooding, and denial-of-service attempts before application logic is invoked.

API Gateway enforces:
- JWT validation for authenticated routes
- Payload size limits
- Rate limiting
- Schema-level input validation before Lambda execution
  

---

## Authorization Model

Authorization is enforced through:

- Role claims embedded in JWT tokens  
- Server-side validation in Lambda (defense in depth)  
- Facility scoping rules  
- Contact identity matching rules  

The system follows the **principle of least privilege**.

Authorization decisions are derived exclusively from validated Cognito token claims and never from client-supplied request data.

Tenant scoping (`facility_id`) is enforced server-side before any database query is executed.

---

## Notification Security Model

Notifications are dispatched asynchronously through Amazon SQS and a dedicated Notification Lambda.

Security controls include:

- No direct client access to SNS or SES
- All outbound notifications originate from controlled Lambda execution
- SQS access restricted via IAM roles
- SNS and SES accessed through VPC Interface Endpoints (no public internet path)

This prevents notification systems from exposing sensitive operational data or becoming an attack surface.

### Notification Content Segmentation

Outbound notifications follow a dual-layer strategy:

- Requesters receive structured confirmation summaries with limited operational details.
- Dispatcher receives full intake copy for operational redundancy.
- Canonical intake snapshots are stored in encrypted S3 for archival integrity.

This approach:

- Preserves legal defensibility
- Reduces external PII/PHI exposure
- Maintains operational continuity if application access is temporarily unavailable
---

## Abuse Containment and Deliverability Protection

The system is designed to prevent automated intake abuse from escalating into outbound notification flooding.

Controls include:

- API Gateway rate limits per IP
- AWS WAF rate-based blocking
- CAPTCHA validation for public intake
- SQS decoupling to buffer outbound notifications
- Notification Lambda concurrency controls
- SES sending quotas and bounce/complaint monitoring

These layered controls ensure that malicious or automated submissions cannot generate uncontrolled email or SMS volume.

The primary risk in abuse scenarios is deliverability degradation, not infrastructure cost.  
The architecture prioritizes preserving notification reputation and operational signal clarity.

---

# Role Definitions and Authority Boundaries

## Internal Roles

### Dispatcher

**Can:**
- Create and edit TripRequests
- Assign drivers
- Modify scheduling details
- Merge patients
- Approve or reject cancellation requests
- Resolve disputes
- Modify billing workflow flags

**Cannot:**
- Bypass audit logging

---

### Driver

**Can:**
- View assigned TripLegs only
- Record completion data
- Mark completed or no-show

**Cannot:**
- View unassigned trips
- Modify scheduling
- Modify billing
- Edit data after completion is finalized

---

### Billing

**Can:**
- View completed TripLegs
- Group by facility/patient
- Manage billing flags (`ready → billed → paid`)

**Cannot:**
- Modify intake data
- Modify completion data
- Modify scheduling

---

# Facility Roles (Strictly Limited)

Facility roles are intentionally restricted to prevent unauthorized data modification and billing risk.

## FacilityUser

**Can:**
- Submit new TripRequests
- View:
  - TripRequests they submitted  
  - TripRequests where they are the listed contact  
- Request cancellation of a future TripLeg (subject to same-day restriction rules)  
- Submit dispute or clarification requests  

**Cannot:**
- Edit TripRequest data after submission  
- Modify completed trips  
- Modify TripLeg completion data  
- Change billing flags  
- Merge patients  
- Delete records  

---

## FacilityAdmin

**Can:**
- View all TripRequests for their facility  
- Request cancellation of future TripLegs  
- Submit dispute or clarification requests  
- Manage facility portal users (add/remove/promote FacilityUsers)  

**Cannot:**
- Modify completed trips  
- Edit driver-entered data  
- Change billing flags  
- Merge patient records directly  
- Delete TripRequests or TripLegs  

FacilityAdmin is an oversight role, not an operational role.

### Cancellation Enforcement Rules

- Cancellation requests initiated by facility users are treated as state-transition requests and require dispatcher approval.
- Same-day cancellation attempts are automatically denied and instruct the facility to contact dispatch by phone.
- Approved cancellations generate notification events to both the dispatcher and the listed facility contact.
  
---

# Facility Portal Visibility Rules

Access is scoped by `facility_id` and contact identity.

## FacilityAdmin Visibility

May view all TripRequests where:

```sql
TripRequest.facility_id = current_user.facility_id

---

## Logging and Audit Controls

Logging is enabled at:

- API Gateway
- Lambda execution
- Database audit layer

Logs capture:

- User ID
- Role
- Timestamp
- Entity reference
- Action performed

Logs intentionally exclude PHI and sensitive payload content.

Audit logs are immutable and support dispute resolution, billing defensibility, and forensic review.
