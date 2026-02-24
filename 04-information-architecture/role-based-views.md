# Role-Based Views

This document defines what each stakeholder role should see and do in the system. The same underlying data is presented differently depending on the user’s task (dispatching, driving, billing, or submitting requests).

The system supports a multi-tenant model. Every TripRequest is associated with exactly one tenant. A tenant may represent:

- A Healthcare Facility
- An Individual / Private-Pay Rider

TripRequests must never exist without a tenant association.

These views are defined before database and architecture decisions to ensure the system design supports real workflows while enforcing tenant isolation.

---

## View 1: Dispatcher / Owner View (Scheduling + Oversight)

**Primary purpose:** Maintain visibility, schedule trips, assign drivers, and manage exceptions with minimal cognitive load.

### Key screens / sections

- **Incoming Requests Queue**
  - New requests waiting to be scheduled
  - Quick indicators:
    - Service date
    - Pickup time
    - Trip type (one-way / round trip)
    - Wheelchair needs
    - Tenant (Facility name or Individual account)
    - Requester type (Facility or Individual)

- **Daily Schedule View**
  - All trip legs for a selected day
  - Sorted by time
  - Driver assignment visible
  - Status indicators:
    - scheduled
    - completed
    - cancelled
    - no-show

- **Weekly / Monthly Overview**
  - High-level visibility of workload trends
  - Clear grouping by tenant
  - Helps anticipate billing volume and staffing needs

### Actions allowed

- Create or edit trip requests
- Confirm requester type (Healthcare Facility or Individual / Private Pay)
- Confirm one-way vs round trip
- Split round trip into two legs (outbound + return)
- Assign drivers to each leg
- Update times (pickup, appointment, back time, will-call changes)
- Mark or record exceptions (cancelled, no-show, rescheduled)
- Link trip requests to existing tenant records (facility or individual)

### Information required in this view

- Tenant information (facility name or individual account)
- Contact information used for this request
- Patient name and mobility needs
- Trip date/time details
- Pickup/drop-off addresses
- Escort requirements
- Driver assignment and leg status

Dispatcher view has full visibility across all tenants.

---

## View 2: Driver View (Execution + Documentation)

**Primary purpose:** Provide a clear ordered ride list and make completion documentation fast and defensible.

Drivers do not manage tenant grouping or billing workflows. They only see assigned trips.

### Key screens / sections

- **Today’s Assigned Trip Legs**
  - Ordered list of trip legs assigned to the driver
  - Each entry shows:
    - Time
    - Pickup
    - Drop-off
    - Patient
    - Mobility needs
    - Tenant name (for context only)

- **Trip Leg Detail View**
  - Shows the original intake information relevant to the trip
  - Includes notes (escort, wheelchair size, special instructions)
  - Displays whether trip originated from facility or individual

- **Completion Logging Panel**
  - Odometer start
  - Start time
  - Odometer end
  - End time
  - Total mileage
  - Driver confirmation/signature

### Actions allowed

- View assigned trip legs for the day
- Record completion data per leg
- Mark trip leg status:
  - completed
  - cancelled
  - no-show
- Add brief notes when exceptions occur (optional)

Drivers cannot:
- Edit intake data
- Modify tenant association
- Change billing status

### Information required in this view

- Addresses and schedule expectations
- Mobility requirements
- Escort requirements (if applicable)
- Clear separation of outbound vs return leg for round trips

---

## View 3: Billing View (Grouping + Tracking)

**Primary purpose:** Reduce manual sorting and make it easy to prepare invoices accurately.

Billing grouping is performed by tenant, which may represent either:

- A Healthcare Facility
- An Individual / Private-Pay Rider

### Key screens / sections

- **Billing Period Selector**
  - Choose date range (week/month/custom)

- **Tenant Grouping View**
  - Completed trip legs grouped by tenant
  - Sub-grouped by patient
  - Totals visible:
    - Number of legs
    - Mileage totals (if applicable)
    - Billing status

- **Billing Workflow Status**
  - Status column or checkbox per leg:
    - ready to bill
    - billed
    - paid
  - Allows stopping and restarting billing without losing place

- **Export / Summary Output**
  - Exportable report (PDF/CSV)
  - Suitable for billing entry into QuickBooks

### Actions allowed

- Filter for completed trip legs
- Group by tenant and patient
- Verify completion documentation exists
- Mark billing progress status
- Export billing summaries

Billing users cannot:
- Modify driver-recorded completion data
- Change tenant association
- Delete records

---

## View 4: Facility Portal View (Authenticated Tenant Users)

Facility portal access applies to Healthcare Facility tenants only.

Two roles exist:

- FacilityAdmin
- FacilityUser

### FacilityAdmin

Can:
- View all requests associated with their facility
- Cancel trips prior to execution
- Submit dispute or clarification requests
- Manage facility portal users

Cannot:
- Edit completed trips
- Modify billing flags
- Delete records
- Merge patient records

### FacilityUser

Can:
- Submit new trip requests
- View:
  - Requests they personally submitted
  - Requests where they are the designated contact
- Cancel trips prior to execution
- Submit dispute or clarification requests

Cannot:
- Edit completed trips
- Modify billing flags
- Delete records

Facility portal users are strictly scoped to their facility_id.

---

## View 5: Individual / Private-Pay Rider View (Optional Portal Access)

Individual riders may:

- Submit ride requests without creating an account (guest intake)
- Receive confirmation via SMS or email
- Optionally create an account later to review ride history

If registered:

- They function as a tenant-scoped user
- They can view only trips associated with their individual tenant account
- They can cancel future trips
- They cannot edit completed trips
- They cannot modify billing flags

---

## Shared System Concepts Across Views

### Tenant Model

- Every TripRequest must be associated with exactly one tenant.
- A tenant may represent:
  - A Healthcare Facility
  - An Individual / Private-Pay Rider
- NULL tenant references are not permitted.
- Tenant isolation is enforced across all authenticated views.

### Trip vs Trip Leg

- A TripRequest may produce:
  - One leg (one-way)
  - Two legs (round trip)
- Each TripLeg is documented independently (times, odometer, signature).
- Return legs may be cancelled without cancelling the outbound leg.

### Status Vocabulary (Consistent Across Views)

- scheduled
- completed
- cancelled
- no-show
- ready to bill
- billed
- paid

---

## Output: Design Implications

Defining role-based and tenant-scoped views implies the system must support:

- Role-based access control (RBAC)
- Tenant isolation (facility or individual)
- Separate data capture moments across time:
  - request → schedule → complete → bill
- Queryable structure that supports grouping by tenant, patient, and date range
- Fast driver interaction with minimal steps and strong documentation integrity
- Strict prevention of cross-tenant data exposure
