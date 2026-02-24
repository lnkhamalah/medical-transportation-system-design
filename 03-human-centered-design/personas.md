# User Personas

The following personas represent the primary users of the proposed medical transportation system. They are based on observed workflows and real operational constraints rather than hypothetical or idealized users. These personas are used to guide design decisions and to constrain system complexity.

---

## Persona 1: Owner / Dispatcher

**Role**  
Owner-operator who manages scheduling, dispatching, and billing oversight.

**Goals**
- Schedule rides efficiently with minimal back-and-forth
- Maintain visibility into daily, weekly, and monthly trips
- Ensure billing information is complete and accurate
- Reduce administrative stress and rework
- Clearly distinguish facility vs individual rides for grouping

**Constraints**
- Non-technical background
- High cognitive load during peak scheduling hours
- Relies on familiarity and repetition rather than training
- Often multitasking while answering phones

**Risks if System Fails**
- Incomplete or incorrect trip records
- Delayed billing and cashflow disruption
- Increased reliance on memory or paper backups
- Loss of trust in the system

---

## Persona 2: Driver

**Role**  
Completes assigned trips and documents mileage and timing for billing and accountability.

**Goals**
- Clearly understand assigned rides and locations
- Complete documentation quickly and accurately
- Protect themselves against disputes or accusations of missed or late trips

**Constraints**
- Limited time between trips
- May have varying comfort with digital tools
- Often working under time pressure and environmental stress
- Needs documentation to be defensible

**Risks if System Fails**
- Inability to prove trip completion
- Incorrect mileage or time records
- Increased disputes with patients or facilities
- Frustration leading to workarounds outside the system

---

## Persona 3: Billing / Administrative Function

**Role**  
Assembles completed rides and prepares invoices for facilities and individual/private-pay riders.

**Goals**
- Group rides accurately by tenant and patient
- Quickly verify which trips are billable
- Minimize re-entry of data into accounting systems

**Constraints**
- Dependent on data quality from earlier steps
- Limited authority to correct missing or inaccurate data
- Often working after the fact with no direct user contact

**Risks if System Fails**
- Billing delays or rejections
- Increased manual reconciliation
- Financial loss or strained relationships

---

## Persona 4: Facility Portal User (Admin or User)

**Role**  
Facility staff member who submits and reviews ride requests.

**Goals**
- Submit requests quickly
- View ride history for reconciliation
- Cancel or contest trips before execution

**Constraints**
- May share responsibility with other staff
- Admin roles may change over time
- Does not control dispatch decisions

**Risks if System Fails**
- Inability to reconcile invoices
- Duplicate or conflicting submissions
- Confusion about trip status

---

## Persona 5: Individual / Private-Pay Rider

**Role**  
Schedules transportation without a facility affiliation.

**Goals**
- Submit ride requests easily
- Receive confirmation that request was received
- View ride history for payment tracking if desired

**Constraints**
- May not want to create an account
- May have limited technical comfort
- May rely on SMS confirmation

**Risks if System Fails**
- Confusion about whether ride was received
- No visibility into ride history
- Billing misunderstandings
