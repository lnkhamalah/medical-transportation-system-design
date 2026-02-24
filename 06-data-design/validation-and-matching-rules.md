# Validation and Matching Rules

This document defines cross-field validation logic, identity matching behavior,
and tenant isolation rules that support accurate scheduling, billing integrity,
and secure multi-tenant operation.

The system handles transportation requests that may include PII and healthcare-adjacent data.
Validation rules must preserve billing defensibility and prevent cross-tenant exposure.

---

## 1. Tenant Isolation Rules (Critical)

Every TripRequest must be associated with exactly one Facility record.

- `TripRequest.facility_id` must never be NULL.
- `Facility.facility_type` must be either:
  - `healthcare`
  - `individual`

All facility-facing queries must include tenant scoping:
