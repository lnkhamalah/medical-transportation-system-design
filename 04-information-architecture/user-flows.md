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
1. Requester selects requester type:
   - Healthcare Facility
   - Individual / Private Pay
2. Requester completes intake:
   - Facility name (if applicable)
   - Contact name
   - Phone, extension (if applicable), fax
3. Trip information is provided:
   - Order date
   - Service date
   - Pickup time, appointment time
   - One-way or round-trip selection
   - Wheelchair and mobility needs
4. Pickup and drop-off addresses are recorded.
5. Attendant or escort information is included if applicable.
6. Patient information is entered:
   - Name
   - Gender
   - Payment method
7. Authorization signature is provided when required.

Output:
- A new **TripRequest** is created.
- TripRequest is assigned to a **tenant** (facility or individual).
- Initial status: **Scheduled (unassigned)**.

---

## Flow 2: Scheduling and Dispatch (Dispatcher / Owner)

**Actor:** Dispatcher or business owner  
**Purpose:** Plan and assign transportation

Steps:
1. Dispatcher reviews incoming trip requests.
2. Confirms trip structure:
   - One-way (single leg)
   - Round-trip (two separate legs)
3. System generates one or two **TripLeg** records.
4. Dispatcher assigns:
   - Service times
   - Driver for each leg
5. Dispatcher resolves conflicts (timing, routing, capacity).

Output:
- Trip legs are scheduled and assigned.
- Status transitions:
  - **Scheduled → Assigned**

---

## Flow 3: Trip Execution – Outbound Leg (Driver)

**Actor:** Driver  
**Purpose:** Document completion of the trip leg

Steps:
1. Driver views assigned trips for the day.
2. Driver begins trip:
   - Records odometer start
   - Records start time
   - Status transitions to **In Progress**
3. Driver completes the trip:
   - Records odometer end
   - Records end time
   - Confirms total mileage
4. Driver signs to confirm accuracy.

Output:
- Trip leg marked **Completed** OR:
  - **Cancelled**
  - **No-show**
- Completion record becomes **immutable** after submission.

---

## Flow 4: Trip Execution – Return Leg (Driver, if applicable)

**Actor:** Driver  
**Purpose:** Document return trip when round-trip applies

Steps:
1. Return leg is triggered:
   - At scheduled time OR
   - Via will-call notification
2. Driver completes a separate trip leg.
3. Completion is recorded independently.

Notes:
- The return leg may occur hours later.
- The return leg may be canceled (e.g., patient admitted).
- Each leg is treated as its own record for accuracy and dispute protection.

Output:
- Return trip leg is:
  - **Completed**
  - **Cancelled**
  - **No-show**

---

## Flow 5: Exceptions and Changes (Dispatcher + Driver)

**Actors:** Dispatcher and driver  
**Purpose:** Maintain accurate records during real-world changes

Common events:
- Patient cancels before pickup
- No-show at pickup
- Return leg cancelled due to admission
- Driver reassigned
- Time or address changes

System behavior:
- Status is updated rather than deleting records
- Cancellation reason may be recorded
- Timeline of events remains visible (audit-friendly)

Output:
- Trip record reflects real-world outcome with traceable status changes.

---

## Flow 6: Billing Preparation (Billing Function)

**Actor:** Billing staff  
**Purpose:** Prepare accurate invoices efficiently

Steps:
1. Billing filters **completed trip legs** within billing period.
2. System groups data:
   - By tenant (facility or individual)
   - By patient
3. Billing reviews completion data for accuracy.
4. System generates **billing-ready output** (structured summary).
5. Billing marks workflow status:
   - **Ready → Billed → Paid**

Output:
- Billing packets are prepared.
- Data is ready for invoicing and record retention.

---

## Assisted Billing Workflow (Key Enhancement)

Instead of manual reconstruction:

1. **Validation Phase**
   - Confirm trip accuracy (completed, mileage correct, no duplicates)

2. **System Translation Phase**
   - System generates invoice-ready data (human-readable format)

3. **Entry Phase**
   - User copies or exports data into QuickBooks

This reduces:
- Cognitive load
- Manual sorting
- Billing errors
- Time spent reconstructing trip history

---

## Key Design Principles Reflected in the Flow

- Familiar paper workflow is preserved.
- Information is added incrementally across roles.
- Accountability is role-specific.
- Round trips are modeled as separate legs.
- Explicit **status transitions** replace implicit assumptions.
- Completion records are **immutable** for dispute protection.
- Billing is **system-assisted**, not manually reconstructed.
- The system supports real-world exceptions (cancellations, delays).

---

## Design Decisions Deferred

- Final database schema
- QuickBooks integration method (manual vs API)
- Advanced billing automation
- Contact lifecycle edge cases
