# Stakeholders

This project supports a small, owner-operated non-emergency medical transportation business. The system is intended to replace paper intake and reduce billing delays by capturing structured ride data that can be grouped, tracked, and verified.

## Primary Stakeholders (Direct Users)

- **Business Owner / Dispatcher**
  - Schedules rides, assigns drivers, and manages daily operations.
  - Needs a clear day-by-day view of rides, driver availability, and ride status (scheduled/completed/cancelled).
  - Responsible for ensuring ride details are accurate before billing.
  - Reviews and links trip requests submitted by both facilities and individual riders.

- **Drivers**
  - Use ride details to complete trips and document ride completion.
  - Must record proof of service per trip leg (times, odometer start/end, total mileage, signature).
  - Needs the system to be simple, familiar, and usable with minimal training.

- **Billing Function (Owner or Billing Support)**
  - Assembles completed rides by tenant (facility or individual) and by patient for invoicing.
  - Needs rides grouped correctly with a clear “ready to bill / billed” status to allow stopping and resuming work without losing progress.
  - Needs accurate completion records to protect trust and prevent disputes.

## Secondary Stakeholders (External / Indirect)

- **Healthcare Facilities**
  - Request transportation services and receive invoices.
  - May have multiple users (admin, scheduling staff, billing staff).
  - Rely on timely, accurate billing supported by documentation that rides occurred.

- **Individual / Private-Pay Riders**
  - May schedule rides directly without a facility.
  - May require visibility into their own ride history for payment tracking.
  - Are impacted by correct scheduling, reliability, and accurate trip records.

- **Patients**
  - Receive transportation services and may have repeat rides.
  - Are impacted by correct scheduling, reliability, and accurate trip records.

## Stakeholder Notes
- The business serves repeat patients (“frequent flyers”), so patient and facility/individual tenant data must be reusable across trips.
- Round trips are always treated as **two trip legs**, which may occur at different times and may be cancelled independently.
- Individual/private-pay riders are modeled as tenant accounts to prevent shared or unscoped trip records.
