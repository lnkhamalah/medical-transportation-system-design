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
