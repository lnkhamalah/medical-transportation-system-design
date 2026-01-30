# Form Field Inventory

This document lists all fields from the existing paper-based
Non-Emergency Medical Transportation Request Form in their original order.
The purpose is to preserve familiarity while enabling structured data storage
and later querying.

No database structure decisions are made here.

---

## Section: Facility Information

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| Facility Name | text | yes | Reused across many trips |
| Contact Name | text | yes | Facility point of contact, “If unknown, enter ‘UNKNOWN’.”  |
| Phone Number | text | yes | Validate as 10 digits (ignore punctuation) |
| Phone Extension | text | no | Optional; stored separately from phone |
| Fax Number | text | no | Legacy support |

---

## Section: Trip Information

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| Order Date | date | yes | Date request was created |
| Service Date | date | yes | Date of transport |
| Pickup Time | time | yes | Expected pickup |
| Appointment Time | time | yes | Required for scheduling and punctuality |
| Back Time | time | no | Used for planned return times |
| Will Call | boolean | no | Indicates return time unknown at request time |
| Trip Type | enum | yes | One-way or Round trip |
| Wheelchair Type | enum | yes | Regular, Wide, Extra Wide, Ambulatory |
| Pickup Address | text | yes | Full address |
| Drop-off Address | text | yes | Full address |
| Escort Present | boolean | no | If yes and name unknown, Escort Identifier may be 'UNKNOWN' |
| Escort Name / Identifier | text | no | Name if known; may be 'UNKNOWN' |
| Request Signature | text | yes | Authorization indicator; typed name, checkbox acknowledgment, captured signature image later |

---

## Section: Patient Information

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| First Name | text | yes | |
| Middle Initial | text | no | Optional |
| Last Name | text | yes | |
| Gender | enum | yes | Male / Female |
| Payment Method | enum | yes | Private, Check, Cash. Facility billing handled as monthly invoicing. |

---

## Section: Direct Billing Authorization

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| Billing Authorization Signature | text | conditional | Required only when direct billing applies |
| Authorization Date | date | conditional | Required only when direct billing applies |


---

## Section: Official Use Only – Trip Legs

This section is repeated for each trip leg (outbound and return).

| Field Name | Field Type | Required | Notes |
|-----------|-----------|----------|------|
| Driver Name | text | yes | Assigned per leg |
| Driver Signature | text | yes | Completion confirmation |
| Odometer Start | number | yes | |
| Start Time | time | yes | |
| Odometer End | number | yes | |
| End Time | time | yes | |
| Total Mileage | number | yes | Calculated |
