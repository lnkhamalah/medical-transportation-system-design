# Nonfunctional Requirements

These requirements describe how the system must behave in order to be usable, reliable, and appropriate for healthcare-adjacent work.

## Usability and Accessibility
- The system must preserve the paper form’s ordering and familiar wording.
- The system must be usable with minimal training for non-technical users.
- The system must support users with moderate literacy and potential language barriers through clear labels and simple inputs.

## Reliability and Data Integrity
- The system must prevent incomplete submissions of required fields.
- The system must maintain consistent records for repeat patients, facilities, and billing periods.
- The system must support accurate documentation of trip completion to protect against disputes.
- The system must support Dispatcher/Owner correction of inaccuracies
  
## Security and Privacy
- The system must support encryption in transit and encryption at rest.
- The system must enforce role-based access (dispatcher, driver, billing) so users only see what they need.
- The system must log or preserve changes to key records related to billing and completion confirmation.

## Maintainability and Scalability
- The system should support growth from 2–3 drivers to a larger operation without redesign.
- The system design should support future enhancements (e.g., automation, deeper reporting) without requiring replacement of core data structures.

## Data Retention
- The system must support long-term retention of ride records and billing documentation for legal and business needs.
