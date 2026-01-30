# Role-Based Views

This document defines what each stakeholder role should see and do in the system. The same underlying data is presented differently depending on the user’s task (dispatching, driving, billing). These views are defined before database and architecture decisions to ensure the system design supports real workflows.

---

## View 1: Dispatcher / Owner View (Scheduling + Oversight)

**Primary purpose:** Maintain visibility, schedule trips, and assign drivers with minimal cognitive load.

### Key screens / sections
- **Incoming Requests Queue**
  - New requests waiting to be scheduled
  - Quick indicators: date, pickup time, trip type, wheelchair needs, facility

- **Daily Schedule View**
  - All trip legs for a selected day
  - Sorted by time, with driver assignment visible
  - Status indicators: scheduled / completed / cancelled / no-show

- **Weekly / Monthly Overview**
  - High-level visibility of workload trends
  - Helps anticipate billing volume and staffing needs

### Actions allowed
- Create or edit trip requests
- Confirm one-way vs round trip
- Split round trip into two legs (outbound + return)
- Assign drivers to each leg
- Update times (pickup, appointment, back time, will-call changes)
- Mark or record exceptions (cancelled, no-show, rescheduled)

### Information required in this view
- Facility + contact info
- Patient name and mobility needs
- Trip date/time details
- Pickup/drop-off addresses
- Escort requirements
- Driver assignment and leg status

---

## View 2: Driver View (Execution + Documentation)

**Primary purpose:** Provide a clear ordered ride list and make completion documentation fast and defensible.

### Key screens / sections
- **Today’s Assigned Trip Legs**
  - Ordered list of trip legs assigned to the driver
  - Each entry shows: time, pickup, drop-off, patient, mobility needs
  - Status indicator per leg

- **Trip Leg Detail View**
  - Shows the original intake information relevant to the trip
  - Includes any notes (escort, wheelchair size, special instructions)

- **Completion Logging Panel**
  - Odometer start, start time
  - Odometer end, end time
  - Total mileage
  - Driver confirmation/signature

### Actions allowed
- View assigned trip legs for the day
- Record completion data per leg
- Mark trip leg status: completed / cancelled / no-show
- Add brief notes when exceptions occur (optional)
- Edit trips to reconcile patient information

### Information required in this view
- Addresses and schedule expectations
- Mobility requirements
- Escort requirements and name (if applicable)
- Clear separation of outbound vs return leg for round trips

---

## View 3: Billing View (Grouping + Tracking)

**Primary purpose:** Reduce manual sorting and make it easy to prepare invoices accurately.

### Key screens / sections
- **Billing Period Selector**
  - Choose date range (week/month/custom)

- **Facility Grouping View**
  - Completed trip legs grouped by facility
  - Sub-grouped by patient
  - Totals visible (number of legs, mileage totals if needed)

- **Billing Workflow Status**
  - Status column or checkbox per leg:
    - ready to bill
    - billed
  - Allows stopping and restarting billing without losing place

- **Export / Summary Output**
  - Exportable report (PDF/CSV) suitable for billing entry into QuickBooks

### Actions allowed
- Filter for completed trip legs
- Group by facility and patient
- Verify completion documentation exists
- Mark billing progress status
- Export billing summaries

### Information required in this view
- Facility name and billing-relevant identifiers
- Patient name
- Service date
- Completion confirmation (driver signature/confirmation)
- Mileage totals and trip leg status
- Payment method / direct billing indicators

---

## Shared System Concepts Across Views

### Trip vs Trip Leg
- A **Trip Request** may produce one leg (one-way) or two legs (round trip).
- Each **Trip Leg** is documented independently (times, odometer, signature).
- Return legs may be cancelled without cancelling the outbound leg.

### Status Vocabulary (Consistent Across Views)
- scheduled
- completed
- cancelled
- no-show
- ready to bill
- billed

---

## Output: Design Implications

Defining role-based views implies the system must support:
- Role-based access control (each user sees only appropriate functions/data)
- Separate data capture moments across time (request → schedule → complete → bill)
- Queryable structure that supports grouping (facility, patient, date range)
- Fast driver interaction with minimal steps and strong documentation integrity
