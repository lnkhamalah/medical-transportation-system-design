# Security Model

## Security Goals

This system handles transportation requests that may include personally identifiable information (PII) and, depending on context, protected health information (PHI). The security model is designed to:

- Prevent unauthorized access to trip and patient-related data  
- Enforce strict role-based access control (internal and facility-facing roles)  
- Protect operational and billing integrity  
- Minimize PHI/PII exposure through public intake  
- Preserve auditability for dispute-sensitive and billing-sensitive actions  
- Protect data in transit and at rest  

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

---

## Authorization Model

Authorization is enforced through:

- Role claims embedded in JWT tokens  
- Server-side validation in Lambda (defense in depth)  
- Facility scoping rules  
- Contact identity matching rules  

The system follows the **principle of least privilege**.

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
- Request cancellation of a future TripLeg  
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

---

# Facility Portal Visibility Rules

Access is scoped by `facility_id` and contact identity.

## FacilityAdmin Visibility

May view all TripRequests where:

```sql
TripRequest.facility_id = current_user.facility_id
