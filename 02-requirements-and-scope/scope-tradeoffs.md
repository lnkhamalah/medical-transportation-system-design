# Scope and Tradeoffs

This project began as a design-and-planning capstone. Scope boundaries are explicitly defined to prevent overreach and keep the work feasible within a 10-week course, while also supporting a phased transition into a real-world implementation.

---

## In Scope (Course Deliverables)

- Stakeholder analysis and workflow documentation
- Requirements definition (functional and nonfunctional)
- Human-centered design principles and usability considerations
- Conceptual system architecture (high-level)
- Data model design (entities, relationships, and form-to-data mapping)
- Security and ethics considerations (privacy, access roles, retention)
- Multi-tenant scoping model (facility and individual)
- Implementation roadmap with risks and tradeoffs
- Conceptual billing workflow design (grouping, status tracking, and assisted billing model)

---

## Extended Scope (Post-Course Implementation Phase)

These items are not required for the course but are part of the planned real-world system:

- Frontend implementation (web-based interface)
- Backend services (API, business logic, validation)
- Deployment to cloud infrastructure (AWS)
- Assisted billing workflow implementation (review-first model)
- Billing translation layer (system-generated invoice-ready outputs)
- Basic reporting and operational dashboards

---

## Out of Scope (Not Completed During Course)

- Full production-grade system implementation
- Fully automated accounting system integration (e.g., direct QuickBooks API integration)
- Mobile app development
- GPS or automated location tracking (may be explored as future enhancement only)
- HIPAA compliance certification work (awareness will be documented, but not formal compliance)
- Advanced security hardening (e.g., penetration testing, enterprise monitoring, full compliance controls)

---

## Key Tradeoffs

- Prioritize a simple, familiar form experience over advanced automation.
- Prioritize data accuracy, auditability, and billing traceability over complex analytics.
- Prioritize assisted billing (human review + system-generated outputs) over full automation to reduce financial risk.
- Model individuals as tenant accounts rather than allowing unscoped or NULL facility records.
- Enforce immutable completion records rather than allowing flexible edits that could introduce billing ambiguity.
- Document a buildable and extensible system plan rather than promising a fully deployed production system within course constraints.

---

## Rationale for Tradeoffs

This design intentionally avoids premature automation in critical financial workflows.

While full automation (e.g., direct accounting system integration) could reduce manual effort, it introduces higher risk if implemented incorrectly. Instead, the system adopts an assisted billing model that:

- Reduces manual reconstruction effort  
- Preserves human validation  
- Minimizes risk of billing errors  
- Creates a clear path toward future automation  

Similarly, the system prioritizes structured, auditable data over advanced features such as real-time tracking or analytics dashboards. This ensures that foundational operational integrity is established before additional complexity is introduced.

The result is a system that is:

- Safe to operate  
- Incrementally extensible  
- Aligned with real business constraints  
- Realistic for a first implementation by a new developer  

---

## Scope Control Strategy

To prevent scope creep, the project is structured in phases:

1. **Design Phase (Course Scope)**  
   Define system structure, workflows, and constraints.

2. **Implementation Phase (Initial Build)**  
   Build core intake, scheduling, and assisted billing functionality.

3. **Enhancement Phase (Post-MVP)**  
   Introduce automation, integrations, and performance improvements.

This phased approach ensures that the system delivers value early while maintaining a clear path for future growth without requiring redesign.
