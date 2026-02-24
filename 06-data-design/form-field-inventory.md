# Form Field Inventory

This document lists all fields from the existing paper-based
Non-Emergency Medical Transportation Request Form in their original order,
with minimal structural additions to support digital intake and tenant isolation.

The purpose is to preserve familiarity while enabling structured data storage
and secure multi-tenant operation.

No database structure decisions are made here.

---

## Section: Requester Type (New – Required for Digital Intake)

| Field Name | Field Type | Required | Notes |
|------------|------------|----------|------|
| Requester Type | enum | yes | Healthcare Facility / Individual (Private Pay). Controls facility logic. |

If **Healthcare Facility** is selected:
- Facility Information section behaves normally.

If **Individual (Private Pay)** is selected:
- Facility Name field is hidden.
- System auto-creates or reuses an Individual-type Facility record internally.

---

## Section: Facility Information

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| Facility Name | text | conditional | Required if Requester Type = Healthcare Facility. Hidden if Individual. |
| Contact Name | text | yes | If Individual, this is the person requesting the ride. If unknown, enter ‘UNKNOWN’. |
| Phone Number | text | yes | Validate as 10 digits (ignore punctuation). Used for confirmations. |
| Phone Extension | text | no | Optional; mostly relevant for healthcare facilities. |
| Fax Number | text | no | Legacy support; typically unused for individuals. |

---

## Section: Trip Information

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| Order Date | date | yes | Date request was created. |
| Service Date | date | yes | Date of transport. |
| Pickup Time | time | yes | Expected pickup. |
| Appointment Time | time | yes | Required for scheduling and punctuality. |
| Back Time | time | no | Used for planned return times. |
| Will Call | boolean | no | Indicates return time unknown at request time. |
| Trip Type | enum | yes | One-way or Round trip. |
| Wheelchair Type | enum | yes | Regular, Wide, Extra Wide, Ambulatory. |
| Pickup Address | text | yes | Full address. |
| Drop-off Address | text | yes | Full address. |
| Escort Present | boolean | no | If yes and name unknown, Escort Identifier may be 'UNKNOWN'. |
| Escort Name / Identifier | text | no | Name if known; may be 'UNKNOWN'. |
| Request Signature | text | yes | Authorization indicator; typed name, checkbox acknowledgment, captured signature image later. |

---

## Section: Patient Information

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| First Name | text | yes | |
| Middle Initial | text | no | Optional. |
| Last Name | text | yes | |
| Gender | enum | yes | Male / Female. |
| Payment Method | enum | yes | Private, Check, Cash. Facility billing handled as monthly invoicing. |

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
| Total Mileage | number | yes | Calculated. |

---

## Digital Logic Notes (Non-Visible to End User)

- TripRequest.facility_id must never be NULL.
- If Requester Type = Individual:
  - System creates or reuses Facility where facility_type = individual.
- If Requester Type = Healthcare Facility:
  - Facility is selected or created from healthcare facility records.
- Individuals may later create portal accounts and will be linked to their individual Facility record.
