# Nonfunctional Requirements

These requirements describe how the system must behave in order to be usable, reliable, secure, and appropriate for healthcare-adjacent operational and billing workflows.

---

## Usability and Accessibility

- The system must preserve the paper form’s ordering and familiar wording.
- The system must be usable with minimal training for non-technical users.
- The system must support users with moderate literacy and potential language barriers through clear labels and simple inputs.
- The system must clearly distinguish between Healthcare Facility and Individual / Private Pay requesters to reduce confusion during intake.
- The system must minimize cognitive load by reducing the need for users to interpret or reconstruct information during scheduling and billing workflows.
- The system should support clear visual indicators for trip status and billing status to guide user actions.

---

## Reliability and Data Integrity

- The system must prevent incomplete submissions of required fields.
- The system must maintain consistent records for repeat patients and tenants (facility or individual).
- The system must ensure every TripRequest is linked to exactly one tenant record.
- The system must support accurate documentation of trip completion to protect against disputes.
- The system must enforce immutability of completed trip data after submission.
- The system must ensure that all billing outputs are derived from structured operational data, not manual summaries.
- The system must maintain consistency between trip records and billing states at all times.
- The system must support Dispatcher/Owner correction of inaccuracies through controlled workflows (no silent overwrites).

---

## Security and Privacy

- The system must support encryption in transit and encryption at rest.
- The system must enforce role-based access (dispatcher, driver, billing, facility users) so users only see what they need.
- The system must enforce tenant scoping to prevent cross-tenant data exposure.
- The system must ensure that sensitive operational and billing data is only accessible through authenticated and authorized requests.
- The system must log and preserve changes to key records related to billing, trip completion, and status transitions.
- The system must prevent unauthorized modification of completed or billed records.

---

## Maintainability and Scalability

- The system should support growth from 2–3 drivers to a larger operation without redesign.
- The system design should support future enhancements (e.g., automation, deeper reporting, external integrations) without requiring replacement of core data structures.
- The system should support modular extension of billing workflows and integration layers (e.g., accounting systems such as QuickBooks).
- The system should allow updates to business rules (e.g., billing eligibility, status transitions) without requiring full system redesign.

---

## Data Retention and Auditability

- The system must support long-term retention of ride records and billing documentation for legal and business needs.
- The system must preserve historical records of trip status, completion data, and billing transitions.
- The system must ensure that deleted records are either prevented or recoverable through audit logs (soft delete or equivalent strategy).
- The system must support traceability of actions (who changed what, and when) for operational and billing events.

---

## Performance and Responsiveness

- The system should provide responsive interaction times for core workflows (ride creation, driver updates, billing review).
- The system should support concurrent usage by multiple roles (dispatcher, drivers, billing) without data inconsistency.
- The system should ensure that delays in non-critical processes (e.g., notifications) do not block core operations such as trip creation or completion.

---

## Billing Workflow Integrity (Cross-Cutting Requirement)

- The system must support a billing workflow that reduces end-of-period administrative burden by preparing billing data continuously as trips are completed.
- The system must ensure that billing outputs are consistent, repeatable, and derived from verified trip data.
- The system must support human review and approval prior to final billing export (assisted billing model).
- The system must ensure that any edits to billing outputs are tracked and auditable.
