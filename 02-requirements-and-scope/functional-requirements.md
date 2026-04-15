# Functional Requirements

These requirements describe what the system must do to replace the paper-based workflow for a small non-emergency medical transportation business. Requirements are written in plain language and organized by user role.

---

## Dispatcher / Owner (Scheduling + Oversight)

- The system must allow a dispatcher to create a new ride request using a digital version of the existing paper form.
- The system must support reuse of existing Facility, Contact, and Patient information for repeat riders to reduce retyping.
- The system must require selection of requester type: Healthcare Facility or Individual / Private Pay.
- If Healthcare Facility is selected, the system must require selection or creation of a Facility record.
- If Individual / Private Pay is selected, the system must automatically create or reuse an Individual-type tenant record.
- The system must ensure that every TripRequest is linked to a tenant record (facility or individual); NULL tenant references are not permitted.
- The system must display a daily schedule view showing rides by date and time.
- The system must allow assigning a driver to each trip leg.
- The system must show ride status (scheduled, completed, cancelled, no-show).
- The system must allow dispatchers to view and update trip status in a controlled manner (no silent overwrites of completion data).
- The system must maintain a complete history of status changes for audit purposes.

---

## Drivers (Daily Work + Documentation)

- The system must provide drivers an ordered list of assigned trip legs for a given day.
- The system must show the completed intake information for each ride (to confirm details and protect against disputes).
- The system must allow drivers to record trip completion data per leg:
  - start time, end time  
  - odometer start, odometer end, total mileage  
  - driver signature / confirmation  
- The system must allow drivers to mark trip legs as completed, cancelled, or no-show.
- The system must prevent modification of submitted completion data after confirmation (immutability enforcement).
- The system must timestamp all completion actions.

---

## Billing (Grouping + Tracking)

- The system must support grouping completed rides by tenant (facility or individual) and by patient for a selected billing period.
- The system must include a billing status indicator so billing work can be paused and resumed.

- The system must support a structured billing workflow with the following states:
  - not_ready  
  - pending_review  
  - approved  
  - exported  
  - paid  

- The system must automatically determine billing eligibility based on trip status and completion data.
- The system must generate billing-ready outputs derived from structured trip and trip leg data (no manual reconstruction required).
- The system must allow users to review and edit billing outputs before final use (assisted billing model).
- The system must preserve original trip data alongside any edited billing output for auditability.

- The system must support exporting or generating a billing-friendly output (e.g., formatted text, PDF summary, or report) suitable for manual entry into QuickBooks.

- The system shall support generating patient-month billing packets grouped by tenant and billing month.
- The system shall support marking packets as sent/resubmitted/closed in QuickBooks.
- The system must support bulk status updates by tenant + month to minimize manual work.

---

## Trip Structure and Special Cases

- The system must treat a round trip as two independent trip legs.
- The system must allow the return leg to be cancelled independently (e.g., patient admitted).
- The system must retain a record of cancellations and completion status for audit and billing clarity.
- The system must ensure that cancelled or no-show trips are excluded from billing eligibility unless explicitly overridden by authorized roles.
- The system must maintain historical records for all trip outcomes (completed, cancelled, no-show) without deletion.

---

## System Behavior and Data Integrity (Cross-Cutting Requirements)

- The system must enforce tenant isolation at all times; users may only access data associated with their authorized tenant.
- The system must enforce server-side validation for all inputs (client-side validation alone is insufficient).
- The system must prevent direct database access from the client.
- The system must ensure all critical operations (trip creation, completion, billing transitions) are atomic and consistent.
- The system must log key system events (status changes, billing transitions, cancellations) for traceability.
- The system must ensure that billing data is always derived from structured operational data, not manually entered summaries.

---

## Future-Ready Capabilities (Non-Blocking Requirements)

- The system should support future integration with external accounting systems (e.g., QuickBooks).
- The system should support automated generation of invoice data through API-based integrations.
- The system should allow extension of billing workflows without redesigning the core data model.
