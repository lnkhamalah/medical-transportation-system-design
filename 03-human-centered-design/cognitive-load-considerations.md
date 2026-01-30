# Cognitive Load Considerations

This system must support non-technical users in a fast-paced environment. The design should reduce memory demands, prevent mistakes, and support task completion even when users are interrupted.

## Cognitive Load Risks in the Current Workflow
- Users must remember details during phone calls and write them accurately.
- Paper forms can be misplaced or difficult to search later.
- Billing requires mentally sorting rides by patient and facility before re-entering data.

## Cognitive Load Reduction Strategies
- **Chunk information into sections**
  - Keep the same clear section breaks used in the paper form.

- **Use structured inputs**
  - Date pickers for dates, time pickers for time, dropdowns for controlled options (wheelchair size, payment method).

- **Conditional fields**
  - Only show wheelchair size if wheelchair is selected.
  - Only require attendant name if escort is “Yes.”

- **Save progress and allow interruption**
  - Support marking rides as “ready to bill” vs “billed” so billing work can pause and resume.

- **Prevent common errors**
  - Validate required fields before submission.
  - Basic logic checks (e.g., pickup time should not occur after end time for a completed leg).

## Why This Matters
Reducing cognitive load increases accuracy, reduces rework, and supports trust with facilities by maintaining consistent documentation.
