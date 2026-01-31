# Validation and Matching Rules

This document defines cross-field logic and matching behaviors that support accurate scheduling and billing preparation while protecting sensitive information.

## 1. Facility-facing intake behavior (no PHI disclosure)
Facility submissions do not use autosuggest for patient names or mobility information. All patient fields are entered as free text to prevent exposure of Protected Health Information (PHI) to unauthenticated users.

## 2. Patient matching workflow (internal users only)
When a TripRequest is submitted, the system performs a similarity check against existing Patient records and flags the request for review if potential matches are found.

Only authorized internal users (owner/dispatcher role) can confirm matches, create a new Patient record, or merge duplicates.

### Allowed actions
- **Link to existing patient:** Attach the request to a known Patient record.
- **Create new patient:** Create a new Patient when no match exists.
- **Merge duplicate patients:** Combine duplicate Patient records into one active record.
- **Undo merge:** Reverse a merge if it was performed incorrectly.

## 3. Merge and unmerge rules (audit-safe)
Patient merges must be reversible and must not rewrite historical trip records.

### Merge requirements
- Merges are recorded in a merge log with who/when/why.
- A merged record must point to the active (canonical) patient record.

### Unmerge requirements
- Unmerge restores the duplicate record as active.
- Unmerge is recorded in the merge log.

## 4. Data integrity rules
- TripRequest must always be linked to exactly one Patient record (active or merged-to).
- TripRequest must preserve original submitted patient name as a snapshot for audit purposes, even if patient records are later corrected.
