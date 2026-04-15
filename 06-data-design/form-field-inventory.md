# Form Field Inventory

This document lists all fields from the existing paper-based  
Non-Emergency Medical Transportation Request Form in their original order,  
with minimal structural additions to support digital intake, tenant isolation,  
and downstream operational and billing workflows.

The purpose is to preserve familiarity while enabling structured data storage,  
secure multi-tenant operation, and accurate billing preparation.

No database structure decisions are made here.

---

## Section: Requester Type (New – Required for Digital Intake)

| Field Name | Field Type | Required | Notes |
|------------|------------|----------|------|
| Requester Type | enum | yes | Healthcare Facility / Individual (Private Pay). Controls facility logic and billing grouping. |

If **Healthcare Facility** is selected:
- Facility Information section behaves normally.
- Billing is typically aggregated and invoiced monthly.

If **Individual (Private Pay)** is selected:
- Facility Name field is hidden.
- System auto-creates or reuses an Individual-type Facility record internally.
- Billing is typically per-trip.

---

## Section: Facility Information

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| Facility Name | text | conditional | Required if Requester Type = Healthcare Facility. Hidden if Individual. |
| Contact Name | text | yes | If Individual, this is the person requesting the ride. If unknown, enter ‘UNKNOWN’. |
| Phone Number | text | yes | Validate as 10 digits (ignore punctuation). Used for confirmations and billing follow-up if needed. |
| Phone Extension | text | no | Optional; mostly relevant for healthcare facilities. |
| Fax Number | text | no | Legacy support; typically unused for individuals. |

---

## Section: Trip Information

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| Order Date | date | yes | Date request was created. |
| Service Date | date | yes | Date of transport. |
| Pickup Time | time | yes | Expected pickup. |
| Appointment Time | time | yes | Required for scheduling and punctuality validation. |
| Back Time | time | no | Used for planned return times. |
| Will Call | boolean | no | Indicates return time unknown at request time. |
| Trip Type | enum | yes | One-way or Round trip. Determines number of TripLeg records. |
| Wheelchair Type | enum | yes | Regular, Wide, Extra Wide, Ambulatory. Impacts service classification. |
| Pickup Address | text | yes | Full address. Used for routing and billing reference. |
| Drop-off Address | text | yes | Full address. Used for routing and billing reference. |
| Escort Present | boolean | no | If yes and name unknown, Escort Identifier may be 'UNKNOWN'. |
| Escort Name / Identifier | text | no | Name if known; may be 'UNKNOWN'. |
| Request Signature | text | yes | Authorization indicator; typed name, checkbox acknowledgment, or captured signature image. |

---

## Section: Patient Information

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| First Name | text | yes | |
| Middle Initial | text | no | Optional. |
| Last Name | text | yes | |
| Gender | enum | yes | Male / Female. |
| Payment Method | enum | yes | Private, Check, Cash. Facility billing handled as aggregated invoicing. |

Note:
- If Requester Type = Individual, Payment Method defaults to Private unless otherwise selected.
- Direct billing section applies only when authorized.

---

## Section: Direct Billing Authorization

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| Billing Authorization Signature | text | conditional | Required only when direct billing applies. Typically healthcare facilities. |
| Authorization Date | date | conditional | Required only when direct billing applies. |

---

## Section: Official Use Only – Trip Legs

This section is repeated for each trip leg (outbound and return).

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| Driver Name | text | yes | Assigned per leg. |
| Driver Signature | text | yes | Completion confirmation. |
| Odometer Start | number | yes | |
| Start Time | time | yes | |
| Odometer End | number | yes | |
| End Time | time | yes | |
| Total Mileage | number | yes | Calculated. Used for billing reference. |

---

## Section: Operational Outcome Tracking (New – System Driven)

These fields are not part of the original paper form but are required for
accurate operational tracking and billing validation.

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| Trip Status | enum | yes | draft, scheduled, completed, cancelled, no_show. |
| Trip Leg Status | enum | yes | scheduled, completed, cancelled, no_show. |
| Completion Flag | boolean | yes | Indicates whether the trip leg was successfully completed. |

Notes:
- These fields determine billing eligibility.
- They are typically set by drivers or dispatchers, not public users.

---

## Section: Billing Preparation Fields (System Derived – Not User Input)

These fields are generated by the system to support billing workflows.
They are not manually entered during intake.

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| Billing Eligibility | boolean | yes | True if trip meets criteria for billing. |
| Billing Review Status | enum | yes | not_ready, pending_review, approved, edited, exported. |
| Billing Notes | text | no | Optional notes for billing review or clarification. |
| Billing Translation Output | text | no | System-generated invoice-ready description. |

Notes:
- Billing output must be reviewed before use.
- These fields support assisted billing workflows and reduce manual reconstruction.

---

## Digital Logic Notes (Non-Visible to End User)

- TripRequest.facility_id must never be NULL.

- If Requester Type = Individual:
  - System creates or reuses Facility where facility_type = individual.

- If Requester Type = Healthcare Facility:
  - Facility is selected or created from healthcare facility records.

- Individuals may later create portal accounts and will be linked to their individual Facility record.

- Trip outcomes (completed, cancelled, no_show) directly influence billing eligibility.

- Billing preparation is derived from structured trip and trip leg data,
  not from manual interpretation of paper forms.

- System-generated billing outputs must remain editable and require human approval
  prior to entry into accounting systems.
