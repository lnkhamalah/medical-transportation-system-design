# Success Metrics

These metrics define what “success” means for the business and stakeholders. Metrics are stated in plain language and tied to workflow outcomes rather than technical features.

## Owner / Dispatcher Success
- Can view rides by day/week/month and quickly understand what is scheduled and who is assigned.
- Can confirm ride status without relying on paper.
- Spends less time reconstructing information for billing.
- Can clearly distinguish and group rides by tenant (facility or individual).

### Extended Dispatcher Success
- Can trust that all required billing data is captured during normal workflow (not reconstructed later)
- Can manage exceptions (cancellations, no-shows, reschedules) without losing historical clarity
- Can rely on the system to enforce structure (no missing tenant, no incomplete critical data)

---

## Driver Success
- Can see an ordered ride list for the day.
- Can document each trip leg with required proof (time, mileage, signature) quickly and consistently.
- Experiences minimal friction and minimal new training burden.

### Extended Driver Success
- Can complete documentation in real time without needing to remember details later
- Clearly understands trip status options (completed, cancelled, no-show)
- Is not responsible for billing or data interpretation beyond completion recording

---

## Billing Success
- Can group rides by tenant (facility or individual) and patient automatically for a billing period.
- Can track billing progress using statuses (e.g., ready to bill, billed) so work can be paused and resumed.
- Reduces retyping and manual sorting compared to the current paper workflow.

### Extended Billing Success (Critical System Goal)
- No longer reconstructs billing manually from paper records
- Reviews system-prepared billing data instead of interpreting raw trip details
- Can move through billing in a consistent, repeatable workflow
- Can clearly see which trips are:
  - ready for billing
  - already billed
  - needing review or correction
- Can export or view billing-ready outputs formatted for easy entry into accounting systems (e.g., QuickBooks)

---

## Business-Level Success
- Billing turnaround time decreases compared to the current week+ delay.
- Cashflow improves due to faster, more organized billing cycles.
- Documentation quality improves, supporting trust and dispute protection.
- Individual/private-pay riders can review their ride history without manual record reconstruction.

### Extended Business-Level Success
- Administrative workload shifts from end-of-month reconstruction to distributed daily workflow
- Billing becomes a verification process rather than a thinking-intensive task
- Reduced risk of missed invoices or duplicate billing
- Increased confidence in operational data as a reliable business asset
- Improved scalability without proportional increase in administrative labor

---

## System-Level Success Indicators (New)

These indicators measure whether the system design is working as intended:

- 100% of TripRequests are associated with a valid tenant (no NULL or ambiguous ownership)
- 100% of completed trip legs include required completion data
- Billing workflow states are consistently used (ready → billed → paid)
- Cancellation and exception events are recorded rather than overwritten
- Data is reused across workflow stages instead of re-entered

---

## Cognitive Load Reduction Indicators (New)

Because a core goal of this system is reducing mental effort:

- Dispatcher does not need to mentally regroup rides for billing
- Billing user does not need to reinterpret trip data
- Driver does not need to remember trip details after completion
- System presents information at the moment it is needed (not requiring recall)

---

## Out of Scope for Course Success
- Direct QuickBooks integration and full production deployment are not required for course success.
- A validated architecture, data model draft, and documented workflows are the primary outcomes for this capstone.
