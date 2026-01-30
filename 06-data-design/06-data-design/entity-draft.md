# Entity Draft (Proposed)

This document identifies the core entities required to support the current paper form workflow while enabling structured, queryable data. These entities are not final tables yet; they represent the real-world objects the system must track.

The primary design constraint is that the digital form must remain familiar in order and language, while the back end supports reuse (patients/facilities) and leg-level documentation (outbound and return trips).

---

## Entity 1: Facility

**Definition:** An organization (clinic, hospital, care home) that requests rides and receives invoices.

**Why it exists:** Facilities are repeated across trips and are used for billing grouping and contact reference.

**Likely core attributes**
- facility_id
- facility_name
- facility_main_phone
- facility_fax (optional)


## Entity 2: FacilityContact

**Definition:** A contact point at a facility (a person, role, or department) used for scheduling or trip coordination.

**Why it exists:** Facilities may have multiple contacts and different phone numbers (scheduler vs nurse vs after-hours). These may change over time.

**Likely core attributes**
- contact_id
- facility_id (FK)
- contact_name (person name or role label, e.g., "Scheduling", "Nurse Station")
- phone_number
- phone_extension (optional)
- contact_type (optional: scheduling / nursing / after_hours / billing)


Notes:
- Phone number is stored as text and validated (e.g., 10 digits ignoring punctuation).
- Extension is stored separately and optional.

---

## Entity 3: Patient

**Definition:** A person being transported.

**Why it exists:** Patients are frequently repeated, and reuse prevents retyping and reduces errors.

**Likely core attributes**
- patient_id
- first_name
- middle_initial (optional)
- last_name
- gender
Notes:
- Mobility needs are captured per trip request for accuracy.
- A patient may optionally store a "default mobility" value as a convenience, but the trip value is authoritative.

---

## Entity 4: TripRequest

**Definition:** The ride request created from the paper form. A single request may produce one leg (one-way) or two legs (round trip).

**Why it exists:** It stores the request context and connects facility, patient, and trip details.

**Likely core attributes**
- trip_request_id
- facility_id (FK)
- contact_id (FK, optional depending on workflow)
- patient_id (FK)
- order_date
- service_date
- pickup_time
- appointment_time
- back_time (optional)
- will_call (optional)
- trip_type (one-way or round trip)
- wheelchair_type (regular / wide / extra wide / ambulatory)
- pickup_address
- dropoff_address
- escort_identifier (optional; may be 'UNKNOWN')
- request_signature
- payment_method (private / check / cash)
- direct_billing_signature (optional based on billing type)
- direct_billing_date (optional based on billing type)
Notes:
- Mobility needs (wheelchair_type) are stored on TripRequest because they do not differ by leg in normal operations.

---

## Entity 5: TripLeg

**Definition:** A single leg of travel that must be documented by the driver. A round trip produces two legs (outbound and return). Legs may occur hours apart and may be cancelled independently.

**Why it exists:** The official-use-only section on the form is leg-level, not request-level.

**Likely core attributes**
- trip_leg_id
- trip_request_id (FK)
- leg_type (outbound / return)
- scheduled_pickup_time (optional if will-call)
- driver_id (FK, assigned by dispatcher)
- leg_status (scheduled / completed / cancelled / no-show)

Driver documentation fields (captured during execution):
- odometer_start
- start_time
- odometer_end
- end_time
- total_mileage
- driver_signature

---

## Entity 6: Driver

**Definition:** A driver who completes trips and provides documentation.

**Why it exists:** Drivers are reused across days and are required for scheduling visibility and accountability.

**Likely core attributes**
- driver_id
- driver_name

---

## Entity 7 (Optional Later): BillingRecord

**Definition:** A billing workflow record to track invoicing state over time.

**Why it exists:** Billing needs “checkbox” status and grouping support (ready to bill / billed / paid). This can be added later if needed.

**Likely core attributes**
- billing_record_id
- facility_id (FK)
- billing_period_start
- billing_period_end
- status (draft / submitted / paid)

Notes:
- This may be implemented as a status field on TripLeg instead of a separate table, depending on design decisions later.
