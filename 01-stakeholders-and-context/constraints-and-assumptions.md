# Constraints and Assumptions

This document captures real-world constraints and assumptions that shape the system design. These constraints help prevent unrealistic scope and ensure the design remains usable for the business.

## Operational Constraints
- The business is small (2–3 drivers) and owner-operated, so administrative time is limited.
- Rides are booked primarily through phone calls from patients and facilities.
- Billing is currently delayed due to manual sorting and re-entry into QuickBooks.
- Round trips are two separate legs and may not occur back-to-back; the return leg may be cancelled (e.g., patient is admitted).

## User Constraints (Human-Centered)
- Users are non-technical and require minimal training.
- Literacy levels may be moderate; at least one user may have a language barrier.
- The system must feel like a digital version of the existing paper form (familiarity over novelty).
- Drivers must be able to document trips quickly without disrupting their workflow.

## Technical / Project Constraints
- This capstone is design-and-planning focused; production implementation is out of scope during the course.
- The system must support secure handling of personal and limited medical-related scheduling information.
- The design should assume encryption in transit and at rest, and role-based access controls.

## Assumptions
- Facility and patient records will repeat over time; the system should reduce retyping by reusing stored records.
- QuickBooks billing will continue to be used initially; outputs must support easier billing preparation even if direct integration is not implemented.
- The business will require long-term record retention for billing and dispute resolution purposes.
