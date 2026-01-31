# User Flows – Non-Emergency Medical Transportation System

This document describes how information moves through the system over time, based on the existing paper intake form. The goal is to preserve the familiar workflow while enabling structured, queryable data behind the scenes.

The system supports multiple user roles interacting with the same trip request at different stages.

---

## Primary User Roles

- Facility Staff / Requester
- Dispatcher / Business Owner
- Driver
- Billing Function

Each role interacts with the same trip record but contributes different information.

---

## Flow 1: Trip Request Creation (Facility → Dispatcher)

**Actor:** Facility staff or authorized requester  
**Purpose:** Request transportation for a patient

Steps:
1. Facility staff completes the top portion of the form:
   - Facility name
   - Contact name
   - Phone, extension (if applicable), fax
2. Trip information is provided:
   - Order date
   - Service date
   - Pickup time, appointment time
   - One-way or round-trip selection
   - Wheelchair and mobility needs
3. Pickup and drop-off addresses are recorded.
4. Attendant or escort information is included if applicable.
5. Patient information is entered:
   - Name
   - Gender
   - Payment method
6. Authorization signature is provided when required.

Output:
- A new trip request exists in the system.
- The request is pending scheduling and assignment.

---

## Flow 2: Scheduling and Dispatch (Dispatcher / Owner)

**Actor:** Dispatcher or business owner  
**Purpose:** Plan and assign transportation

Steps:
1. Dispatcher reviews incoming trip requests.
2. Confirms trip type:
   - One-way (single leg)
   - Round-trip (two separate legs)
3. Assigns service dates and expected times.
4. Assigns a driver to each trip leg.
5. Prepares the trip for execution.

Output:
- Trip legs are scheduled.
- Driver assignments are visible.
- Trip is ready for execution.

---

## Flow 3: Trip Execution – Outbound Leg (Driver)

**Actor:** Driver  
**Purpose:** Document completion of the trip leg

Steps:
1. Driver receives assigned trip details.
2. Driver records:
   - Odometer start
   - Start time
3. Driver completes the trip leg.
4. Driver records:
   - Odometer end
   - End time
   - Total mileage
5. Driver signs to confirm accuracy.

Output:
- Outbound trip leg is documented.
- Mileage and timing are recorded for accountability.

---

## Flow 4: Trip Execution – Return Leg (Driver, if applicable)

**Actor:** Driver  
**Purpose:** Document return trip when round-trip applies

Steps:
1. Driver completes a separate trip leg later in the day.
2. Odometer start/end and times are recorded independently.
3. Driver signs for the return leg.

Notes:
- The return leg may occur hours later.
- The return leg may be canceled if the patient is admitted.
- Each leg is treated as its own record for accuracy and dispute protection.

Output:
- Return trip leg is documented or marked canceled.

---

## Flow 5: Billing Preparation (Billing Function)

**Actor:** Billing staff  
**Purpose:** Prepare accurate invoices

Steps:
1. Billing reviews completed trip legs.
2. Trips are grouped by:
   - Facility
   - Patient
3. Completion status is verified.
4. Billing status is tracked (ready, submitted, paid).

Output:
- Billing packets are prepared.
- Data is ready for invoicing and record retention.

---

## Key Design Principles Reflected in the Flow

- Familiar paper workflow is preserved.
- Information is added incrementally.
- Accountability is role-specific.
- Round trips are modeled as separate legs.
- The system supports real-world exceptions (cancellations, delays).

##Design decisions deferred
- Final database schema
- Billing workflow depth
- Contact lifecycle edge cases
