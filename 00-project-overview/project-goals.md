# Project Goals

The goals of this project are to:

- Design a system that replaces paper-based intake with structured digital data
- Preserve familiar workflows for non-technical users
- Improve organization of ride and billing data
- Reduce manual re-entry and billing delays
- Support both healthcare facilities and individual/private-pay riders using a unified tenant model
- Produce a validated system architecture and database design

---

## Extended System Goals (Refined)

- Transform billing from a manual reconstruction process into a structured review and approval workflow
- Capture operational data once and reuse it across scheduling, execution, and billing
- Separate decision-making tasks (dispatching) from repetitive tasks (billing entry)
- Reduce cognitive load by minimizing redundant thinking and data translation
- Provide a system that supports incremental data capture across roles (dispatcher → driver → billing)
- Ensure every trip is fully traceable from request through completion and billing state

---

## Operational Efficiency Goals

- Reduce billing preparation time by organizing trips automatically by tenant and patient
- Enable bulk billing workflows (tenant + billing period grouping)
- Support a “review-first” billing process where data is validated before entry into accounting systems
- Minimize clerical errors caused by repeated manual transcription
- Allow billing work to be paused and resumed without losing progress

---

## Data Integrity and Structure Goals

- Ensure every TripRequest is associated with exactly one tenant (no NULL or ambiguous ownership)
- Preserve immutable completion records for dispute protection
- Track trip lifecycle states (scheduled → completed → billed → paid)
- Maintain structured relationships between trips, patients, and tenants
- Support future automation (e.g., accounting integration) without redesigning core data structures

---

## User Experience Goals

- Maintain the mental model of the existing paper form
- Reduce friction for non-technical users through clear structure and validation
- Support interruption-friendly workflows (especially for dispatchers)
- Provide role-specific views optimized for:
  - Dispatching
  - Driving
  - Billing
- Ensure the system feels intuitive rather than “new” or disruptive

---

## Strategic and Long-Term Goals

- Create a scalable system that grows with the business without requiring redesign
- Establish a foundation for future automation (e.g., billing integration, reporting)
- Improve cashflow timing by accelerating billing readiness
- Reduce operational risk through structured data and auditability
- Produce a portfolio-quality system design demonstrating real-world problem solving

---

## Design Philosophy

This project is not only about digitization.

It is designed to:

- Eliminate redundant cognitive work
- Convert operational data into reusable system assets
- Shift billing from reconstruction to verification
- Create a system where effort is distributed naturally across the workflow instead of concentrated at the end

The ultimate goal is to make the system feel effortless in use while significantly improving accuracy, speed, and scalability behind the scenes.
