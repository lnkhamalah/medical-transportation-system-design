# Cognitive Load Considerations

This system must support non-technical users in a fast-paced environment. The design should reduce memory demands, prevent mistakes, and support task completion even when users are interrupted.

The system serves both healthcare facilities and individual/private-pay riders. Intake must clearly distinguish between these requester types to prevent downstream correction work, billing confusion, or tenant misclassification.

## Cognitive Load Risks in the Current Workflow
- Users must remember details during phone calls and write them accurately.
- Paper forms can be misplaced or difficult to search later.
- Billing requires mentally sorting rides by patient and facility before re-entering data.
- Requests from individuals and facilities are not structurally separated, increasing grouping and reconciliation effort.

## Cognitive Load Reduction Strategies

- **Clear requester type selection**
  - At the beginning of intake, the user must select:
    - Healthcare Facility
    - Individual / Private Pay
  - This prevents ambiguous tenant assignment and reduces rework.

- **Chunk information into sections**
  - Keep the same clear section breaks used in the paper form.

- **Use structured inputs**
  - Date pickers for dates.
  - Time pickers for time.
  - Dropdowns for controlled options (wheelchair size, payment method).
  - Controlled tenant selection (facility lookup or individual designation).

- **Conditional fields**
  - Only show wheelchair size if wheelchair is selected.
  - Only require attendant name if escort is “Yes.”
  - Only require facility lookup if requester type = Healthcare Facility.

- **Save progress and allow interruption**
  - Support marking rides as “ready to bill” vs “billed” so billing work can pause and resume.

- **Prevent common errors**
  - Validate required fields before submission.
  - Basic logic checks (e.g., pickup time should not occur after end time for a completed leg).
  - Prevent creation of unscoped trip requests (every request must belong to exactly one tenant).

## Design Constraints Derived from Cognitive Load

The human-centered considerations in this project are not abstract preferences; they function as hard constraints on system design.

- Users cannot be expected to remember information across screens, so the system must display relevant ride details at the moment of action.
- Drivers must be able to confirm and document a trip without navigating away from the assigned ride view.
- Repeated data entry must be minimized by reusing patient and tenant records rather than retyping.
- Required fields and validation rules must prevent incomplete or inconsistent submissions, as errors propagate directly into billing disputes.
- Every TripRequest must be associated with exactly one tenant (facility or individual); shared or NULL tenant states are not permitted.

These constraints directly influence database structure, form validation rules, tenant scoping, and role-based views in later design phases.

## Why This Matters
Reducing cognitive load increases accuracy, reduces rework, and supports trust with facilities and individual riders by maintaining consistent documentation.
