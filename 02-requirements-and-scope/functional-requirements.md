# Functional Requirements

These requirements describe what the system must do to replace the paper-based workflow for a small non-emergency medical transportation business. Requirements are written in plain language and organized by user role.

## Dispatcher / Owner (Scheduling + Oversight)
- The system must allow a dispatcher to create a new ride request using a digital version of the existing paper form.
- The system must support reuse of existing Facility, Contact, and Patient information for repeat riders to reduce retyping.
- The system must display a daily schedule view showing rides by date and time.
- The system must allow assigning a driver to each trip leg.
- The system must show ride status (scheduled, completed, cancelled, no-show).

## Drivers (Daily Work + Documentation)
- The system must provide drivers an ordered list of assigned trip legs for a given day.
- The system must show the completed intake information for each ride (to confirm details and protect against disputes).
- The system must allow drivers to record trip completion data per leg:
  - start time, end time
  - odometer start, odometer end, total mileage
  - driver signature / confirmation

## Billing (Grouping + Tracking)
- The system must support grouping completed rides by facility and by patient for a selected billing period.
- The system must include a billing status indicator (e.g., ready to bill, billed) so billing work can be paused and resumed.
- The system must support exporting or generating a billing-friendly output (e.g., PDF summary or report) suitable for manual entry into QuickBooks.
- System shall support generating patient-month billing packets grouped by facility and billing month.
- System shall support marking packets as sent/resubmitted/closed in QuickBooks.
- System must support bulk status updates by facility + month to minimize manual work.

## Trip Structure and Special Cases
- The system must treat a round trip as two independent trip legs.
- The system must allow the return leg to be cancelled independently (e.g., patient admitted).
- The system must retain a record of cancellations and completion status for audit and billing clarity.
