# Architectural Principles

This architecture is driven by operational constraints rather than technology preference.

## 1. Layered Enforcement of Business Rules

User interfaces must not directly access persistence layers. All validation, status transitions, merge operations, and billing state changes occur in a controlled application layer.

This prevents bypassing of audit logic and preserves data integrity.

## 2. Relational Integrity First

The system models structured relationships:

- Facility → TripRequest
- Patient → TripRequest
- TripRequest → TripLeg (1..2)
- Driver → TripLeg
- PatientMergeLog → Patient

Because billing, grouping, and audit queries depend on relational consistency, the architecture prioritizes ACID-compliant storage and foreign key enforcement.

## 3. Audit Preservation Over Mutability

Historical records must not be overwritten.

Snapshot fields (e.g., contact information used at intake) are preserved even if upstream records change later.

Merge and unmerge operations are logged and reversible.

## 4. Network Isolation of Persistence Layer

The database must never be publicly accessible.

All access must occur through controlled application compute inside a restricted network boundary.

## 5. Minimize Operational Overhead

The business does not have a dedicated IT team.

The architecture favors managed services over infrastructure management.

## 6. Separate Structured and Unstructured Storage

Relational data remains in the database.
Documents and exports are stored in object storage.

This prevents database bloat and supports scalable retention.

## 7. Explicit Role Segmentation

Dispatcher, driver, and billing roles have different permissions and data visibility requirements.

Authorization must be enforced at the token and application layer, not solely in the UI.

## 8. Write-Only Public Intake

Public intake endpoints must never expose internal system data.

Guest users may submit transportation requests without authentication, but they must not:

- query existing patients
- retrieve trip history
- access facility records
- receive internal identifiers

Public intake is strictly write-only and returns receipt-level confirmation only. All data exposure requires authenticated and authorized access.

## 9. Asynchronous Notification Decoupling

Notification delivery must not block transactional persistence.

Trip creation and status changes are committed before notification processing occurs. Notification events are published to a message queue and processed separately to prevent downstream SMS or email failures from interrupting core operations.

This design ensures operational continuity during temporary notification service disruptions.

## 10. Controlled Cancellation Governance

Cancellations are not destructive actions and must follow explicit governance rules.

- Facility roles may request cancellation of future `TripLeg` records.
- Same-day cancellation requests are denied through the portal and require direct phone communication.
- Upon a cancellation request, the dispatcher receives a notification and must finalize the cancellation.
- Once finalized, confirmation notifications are sent to:
  - the facility contact
  - the dispatcher

This prevents operational blind spots and ensures drivers are not unexpectedly impacted by late changes.

## 11. Immutable Operational Records

Completion data entered by drivers must be treated as immutable once finalized.

Facility roles may not edit or overwrite:

- completion timestamps
- odometer readings
- driver-entered notes
- billing flags

Cancellations and disputes are recorded as state transitions or separate records rather than destructive edits.

This preserves billing defensibility and audit integrity.

## 12. Defense-in-Depth Hardening

Security controls operate at multiple layers:

- edge filtering via AWS WAF
- API throttling and request size limits
- CAPTCHA for public intake abuse mitigation
- server-side role validation using token claims
- tenant scoping enforced before database query execution
- network isolation of the persistence layer

No single control is relied upon in isolation. The architecture assumes that any individual layer could fail and compensates accordingly.
