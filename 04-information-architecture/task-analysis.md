# Task Analysis

This document breaks the user flows into practical tasks, decisions, and failure points. It focuses on what users are trying to accomplish, what information they need at the moment of action, and what can go wrong. This task analysis will later inform validation rules, role-based access, and database structure.

---

## Task 1: Capture a New Trip Request (Facility → Dispatcher)

**Primary actor:** Facility staff (or dispatcher entering on their behalf)  
**User goal:** Submit a complete and accurate ride request quickly.

### Inputs required (from the form)
- Facility name, contact name, phone, ext (optional), fax
- Order date, service date, pickup time, appointment time, back time / will call
- One-way vs round trip
- Mobility needs (wheelchair size or ambulatory)
- Pickup and drop-off addresses
- Escort yes/no and escort name if yes
- Patient name, gender
- Payment method (private/check/cash)
- Direct billing authorization signature/date (if applicable)

### Decisions
- Is this **one-way** or **round trip**?
- Is the ride **will call** (time unknown until later)?
- Does the patient require **wheelchair** (and which size) or is the patient ambulatory?
- Is an **escort** present?
- Is this a **direct billing** request requiring authorized signature/date?

### Common failure points
- Missing required times (pickup/appointment) or conflicting time info (e.g., will call + fixed back time both selected)
- Incomplete contact info (phone missing or formatted incorrectly; ext confusion)
- Address missing key details (suite, entrance, facility wing)
- Escort selected “Yes” but name missing
- Payment method unclear or multiple selected incorrectly
- Direct billing selected but signature/date missing

### What the system must support
- Required field prompts and clear validation messages
- Optional fields clearly marked (e.g., ext)
- Conditional fields (escort name only required if escort = yes)
- Clear handling of “will call” vs scheduled times

---

## Task 2: Review and Schedule Trips (Dispatcher)

**Primary actor:** Dispatcher / owner  
**User goal:** Create a workable day plan and assign drivers while maintaining accuracy.

### Inputs needed at scheduling time
- Service date and planned pickup time(s)
- Trip type (one-way vs round trip)
- Mobility requirements (affects vehicle/driver planning)
- Pickup/drop-off locations (for routing)
-Escort present (affects number of seats/spaces needed in vehicle)

### Decisions
- Can the business accept this ride given current capacity?
- If round trip, should the return leg be:
  - scheduled with a specific back time, or
  - marked as will call?
- Which driver is assigned to each leg?
- Are there conflicts with existing rides (time overlap, location constraints)?

### Common failure points
- Overbooking a driver due to missing schedule visibility
- Treating a round trip as one event instead of two legs
- Missing or unclear return leg details
- Accepting rides without enough time buffer for travel

### What the system must support
- Calendar/day view that shows all legs by time
- Driver availability and assignment tracking
- Two-leg representation for round trips (separate outbound and return legs)
- Quick edit/update when times shift

---

## Task 3: Execute Trip Leg and Document Completion (Driver)

**Primary actor:** Driver  
**User goal:** Complete the leg and record defensible proof of service.

### Inputs needed at execution time
- Pickup address and drop-off address
- Planned time expectations (pickup time / appointment time)
- Patient and escort notes
- Mobility requirements (wheelchair size, ambulatory)

### Actions (per leg)
- Record odometer start and start time
- Complete transport
- Record odometer end and end time
- Confirm total mileage
- Sign/confirm completion

### Decisions
- Did the trip occur as scheduled?
- Was the patient late/no-show/cancelled?
- If return leg exists, is it still happening or cancelled (e.g., admission)?

### Common failure points
- Driver forgets to record start/end details in the moment
- Wrong leg documented (outbound vs return)
- Missing signature/confirmation
- Dispute risk: facility/patient questions lateness or non-occurrence

### What the system must support
- Fast “driver view” that is optimized for completion logging
- Clear separation of outbound vs return leg
- Ability to mark status: completed / cancelled / no-show
- Simple validation (e.g., odometer end should be ≥ start)

---

## Task 4: Handle Exceptions and Changes (Dispatcher + Driver)

**Primary actors:** Dispatcher and driver  
**User goal:** Maintain accurate records when real-world changes occur.

### Common exceptions
- Patient cancels
- Return leg cancelled due to hospital admission
- Driver delayed or reassigned
- Pickup time changes or will-call triggered late
- Address corrections

### What the system must support
- Updating a trip leg without losing original context
- Recording cancellation reason and timestamp (lightweight, not bureaucratic)
- Keeping a traceable history for billing disputes (at minimum, status + notes)

---

## Task 5: Prepare Billing (Billing Function)

**Primary actor:** Billing staff / owner  
**User goal:** Produce accurate invoices quickly with minimal sorting and retyping.

### Inputs needed for billing
- Completed trip legs within billing period
- Facility associated with each leg
- Patient associated with each leg
- Mileage totals and completion confirmation
- Billing method (private/check/cash/direct billing indicator)

### Actions
- Filter for completed legs in a date range (billing period)
- Group by facility → then by patient
- Confirm each leg has required completion data
- Mark billing workflow status (ready to bill → billed)

### Common failure points
- Legs missing completion data but still counted
- Duplicate trips or missing trips when sorting manually
- Losing place during billing and rechecking work repeatedly

### What the system must support
- “Billing view” that groups automatically
- Status checkboxes or workflow states
- Exportable summaries (PDF/CSV) suitable for QuickBooks entry

---

## Output: Design Implications

This task analysis implies that the system must support:
- Incremental data capture across roles over time
- Separate trip legs for round trips
- Strong validation for required/conditional fields
- Role-based views (dispatcher vs driver vs billing)
- Workflow statuses that support interruptions and resumption
