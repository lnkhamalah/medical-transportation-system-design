# Validation and Matching Rules

This document defines cross-field validation logic, identity matching behavior,
and tenant isolation rules that support accurate scheduling, billing integrity,
and secure multi-tenant operation.

The system handles transportation requests that may include PII and healthcare-adjacent data.
Validation rules must preserve billing defensibility and prevent cross-tenant exposure.

---

## 1. Tenant Isolation Rules (Critical)

Every TripRequest must be associated with exactly one Facility record.

- `TripRequest.facility_id` must never be NULL.
- `Facility.facility_type` must be either:
  - `healthcare`
  - `individual`

All facility-facing queries must include tenant scoping:

TripRequest.facility_id = current_user.facility_id

Failure to scope by facility_id is considered a security defect.

---

## 2. Public Intake Behavior (No Authentication)

Public intake supports accessibility and one-time requesters.

Public intake rules:

- No login required
- No autosuggest
- No patient lookup
- No facility lookup
- No ability to read existing records

Public intake is strictly **write-only**.

### Public submission flow

1. User selects Requester Type:
   - Healthcare Facility
   - Individual / Private Pay

2. If Healthcare Facility:
   - Dispatcher later links to or creates a Facility record (`facility_type = healthcare`)

3. If Individual:
   - System creates or reuses a Facility record (`facility_type = individual`)

4. TripRequest is created with:
   - `created_via = guest`
   - `submitted_by_user_id = NULL`

`created_via` must be one of:

- `guest`
- `facility_portal`
- `dispatcher_phone`

Public intake must not expose patient history, contact records, or mobility history.

---

## 3. Facility Portal Validation Rules

Authenticated Facility users are scoped by:

- `custom:facility_id`
- `custom:contact_id` (for FacilityUser)

All facility portal queries must enforce tenant isolation first:

TripRequest.facility_id = current_user.facility_id
---

### FacilityAdmin Visibility Rule

FacilityAdmin may view all TripRequests where:

TripRequest.facility_id = current_user.facility_id
---

---

### FacilityUser Visibility Rule

FacilityUser may view TripRequests where:

TripRequest.facility_id = current_user.facility_id
AND
(
    TripRequest.submitted_by_user_id = current_user.user_id
    OR
    TripRequest.contact_id = current_user.contact_id
)

Facility users must never see requests from other facilities,
even if patient names match.

---

## 4. Request Submission Validation Rules

### Required Fields

The system must block submission if required fields are missing:

- service_date
- pickup_time
- appointment_time
- pickup_address
- dropoff_address
- patient first and last name
- wheelchair_type
- request_signature

### Conditional Rules

- If `trip_type = round_trip`, system must create two TripLeg records.
- If `will_call = true`, `back_time` must be NULL.
- If `escort_present = true`, `escort_identifier` must not be empty.
- If `direct_billing_applies = true`, authorization signature and date are required.

### Logical Consistency Rules

- pickup_time must not be after appointment_time.
- odometer_end must be ≥ odometer_start.
- TripLeg completion fields required when `completed_flag = true`.

All validation must be enforced server-side in Lambda,
even if UI validation exists.

---

## 5. Patient Matching (Internal Only)

Patient matching occurs only in internal workflows.

Facility portal users and public users do not receive autosuggest.

### Matching Trigger

When a TripRequest is created or reviewed internally:

- System checks for existing Patient records with:
  - Similar first name
  - Similar last name
  - Optional weighting by prior tenant association

If similarity threshold is met:
- Flag for dispatcher review.

### Allowed Internal Actions

Only Dispatcher role may:

- Link TripRequest to existing Patient
- Create new Patient
- Merge duplicate Patient records
- Undo merges

Facility roles cannot merge patients directly.

---

## 6. Patient Merge and Unmerge Rules (Audit-Safe)

Patient merges must preserve historical accuracy.

### Merge Behavior

- Duplicate patient record sets:
  - `active_patient_id` → canonical record
- Merge event recorded in `PatientMergeLog`
- Historical TripRequests remain unchanged

### Unmerge Behavior

- Restore duplicate as active
- Record undo event in merge log
- No deletion of historical data

No destructive updates are allowed.

---

## 7. Contact Matching Rules

When dispatcher selects a FacilityContact:

- Snapshot fields on TripRequest must store:
  - `contact_name_used`
  - `contact_phone_used`
  - `contact_ext_used`

If contact info is edited during intake:

- Changes apply only to snapshot fields
- FacilityContact record is updated only if explicitly chosen

Historical truth always lives on TripRequest snapshot fields.

---

## 8. Trip Status Transition Rules

### TripRequest.status Allowed Transitions

- `draft → scheduled`
- `scheduled → completed`
- `scheduled → cancelled`

### TripLeg.completed_flag

- False by default
- Can only be set True by assigned Driver
- Cannot be unset by Facility roles

### Billing Flags

- `invoiced_flag` can be set only by Billing role
- `paid_flag` can be set only by Billing role
- Facility roles cannot modify billing flags

---

## 9. Cancellation and Dispute Handling

Facility roles cannot delete or modify TripRequests.

Instead:

- Cancellation requests must:
  - Change TripRequest.status to `cancelled` (if before execution)
  - Preserve original data
- Disputes must:
  - Be recorded as notes or separate DisputeRequest entity
  - Never overwrite driver completion data

All cancellation and dispute events must log:

- user_id
- role
- timestamp
- reason

---

## 10. Individual / Private-Pay Rules

If `facility_type = individual`:

- The Facility record represents a single billing account.
- TripRequests are grouped under that individual tenant.
- Portal users (if created later) are linked to that `facility_id`.

Individual tenants must never be grouped together.

Each private-pay rider is isolated in their own Facility record.

---

## 11. Logging Constraints

System logs must:

- Avoid storing PHI in plain-text logs.
- Log event metadata only:
  - user_id
  - role
  - action
  - timestamp
  - entity_id

Audit-sensitive events include:

- Patient merges
- Trip completion
- Billing status changes
- Cancellation requests

---

## Summary

These validation and matching rules enforce:

- Strict tenant isolation
- Least-privilege access
- Non-destructive audit safety
- Public intake safety
- Billing defensibility
- Secure facility portal visibility

All authorization and validation rules must be enforced server-side in Lambda.


