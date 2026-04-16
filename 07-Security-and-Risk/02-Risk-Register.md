# Risk Register

## Purpose

This document identifies material risks associated with the design, planning, and eventual implementation of the transportation management system.

Each risk includes:

- Category
- Description
- Probability assessment
- Impact assessment
- Mitigation strategy
- Residual risk evaluation

The objective is not to eliminate all risk, but to reduce preventable risk and ensure informed decision-making.

This version reflects updated system design decisions, including:
- Separation of dispatch cognition from billing execution
- Translation layer for invoice-ready data
- Structured approval and edit workflows
- Increased audit visibility for post-entry corrections

---

# Risk Register Table

| ID | Risk | Category | Probability | Impact | Mitigation Strategy | Residual Risk |
|----|------|----------|------------|--------|--------------------|---------------|
| R1 | Unauthorized data access or breach | Security | Low | High | Role-based access control, tenant isolation, encryption in transit and at rest, private database subnet, no public DB endpoint, audit logging | Low |
| R2 | Cross-tenant data exposure | Security / Legal | Low | High | Enforced `facility_id` scoping from validated JWT claims, no client-supplied tenant identifiers trusted, server-side query enforcement | Very Low |
| R3 | Revenue loss due to documentation disputes | Operational / Financial | Medium | High | Immutable completion records, structured cancellation workflow, audit logging, independent leg modeling | Low |
| R4 | Billing delays persist despite system implementation | Financial | Medium | Medium | Structured billing statuses, group-by tenant-month logic, reusable patient/facility records, bulk billing updates, and pre-translated billing outputs | Low |
| R5 | Infrastructure outage affecting availability | Technical | Low | Medium | Multi-AZ RDS deployment, serverless compute scaling, decoupled SQS notifications, managed services | Low |
| R6 | Cost overrun during production implementation | Triple Constraint (Cost) | Medium | Medium | Phased implementation roadmap, managed services to reduce infrastructure overhead, defined scope boundaries | Medium |
| R7 | Scope creep during development | Triple Constraint (Scope) | Medium | Medium | Explicit scope definition, deferred feature list documented, design-first approach before implementation | Medium |
| R8 | Delayed adoption or resistance from users | Organizational | Medium | Medium | Human-centered form design, preserve paper workflow structure, minimal training burden, role simplicity, reduced billing cognitive load through translation workflow | Low–Medium |
| R9 | Overengineering relative to business size | Strategic | Low | Medium | Managed services, serverless model, avoidance of unnecessary microservices, minimal operational overhead, optional automation layers (not required for operation) | Low |
| R10 | Regulatory exposure due to improper operational policy | Legal / Governance | Low | High | Architecture supports secure handling; workforce training and policy enforcement assumed as business governance responsibilities | Medium (organizational control dependent) |
| R11 | Billing translation errors (dispatch → invoice layer mismatch) | Operational / Financial | Medium | Medium | Introduce reviewable translation layer, approval workflow, and editable flagged corrections before final billing export | Low |
| R12 | Silent data corruption through post-completion edits | Operational / Integrity | Low | High | Immutable completion records + flagged edit system with required visibility and optional justification notes | Very Low |
| R13 | Cognitive overload during dispatch leading to downstream billing errors | Human Factors | Medium | Medium | Separation of dispatch task (thinking) and billing task (copy/paste execution), structured data capture at source | Low |
| R14 | Over-reliance on automation reducing user understanding | Organizational / Cognitive | Medium | Low | Maintain visibility of translated outputs, allow review/edit workflows, preserve human-in-the-loop validation before billing finalization | Low |

---

# Risk Analysis Narrative

## 1. Security and Data Exposure Risks

The highest-impact risks relate to unauthorized access and cross-tenant exposure.

Mitigation is embedded directly into architecture:

- Server-side role enforcement
- Token-based tenant scoping
- Private database deployment
- Encryption in transit and at rest
- Immutable operational records

These are structural controls rather than procedural recommendations.

### What This Means for the Business

Security risk is reduced through architecture, not policy alone.

The system is designed so that accidental misconfiguration or casual misuse does not result in exposure. This protects reputation, billing integrity, and long-term trust.

Organizations that rely on UI-level restrictions often discover weaknesses only after exposure occurs. This design prevents that class of failure.

---

## 2. Financial and Billing Risks

The most common financial risks in small transportation operations are:

- Billing reconstruction delays
- Documentation disputes
- Unclear status tracking
- Retroactive edits
- Cognitive fatigue during repetitive billing tasks

The system mitigates these through:

- Immutable completion records
- Structured billing states
- Tenant grouping
- Audit-friendly workflows
- Introduction of a translation layer that converts operational data into invoice-ready format

Residual risk primarily depends on user adoption and process discipline.

### What This Means for the Business

Revenue protection is strengthened by eliminating ambiguity.

The introduction of a translation layer reduces the cognitive burden of billing by converting structured dispatch data into copy-ready invoice inputs.

This reduces fatigue, speeds up billing cycles, and improves consistency.

Over time, this reduces operational stress and increases financial predictability.

---

## 3. Cognitive and Workflow Risks (New)

A key design insight is that dispatching and billing require different types of thinking:

- Dispatching requires active decision-making and contextual awareness
- Billing is repetitive and prone to fatigue-based errors

Without separation, this leads to:

- Slower billing
- Increased mistakes
- Reduced user engagement

### Mitigation

- Separate dispatch cognition from billing execution
- Introduce translation outputs that require minimal transformation
- Allow approve/edit/flag workflows before final billing

### What This Means for the Business

Billing becomes a low-cognitive-load task.

This allows less experienced workers to perform billing accurately, while preserving high-skill decision-making during dispatch.

It increases operational scalability without increasing staffing complexity.

---

## 4. Triple Constraint Risks (Scope, Cost, Time)

### Scope Creep

New feature requests (e.g., automation, QuickBooks integration, AI-assisted workflows) could expand implementation complexity.

Mitigation:
- Clear documentation of in-scope vs. deferred features
- Translation layer positioned as optional enhancement, not required dependency

### Cost Overrun

Cloud services introduce ongoing operational costs.

Mitigation:
- Serverless model scales with usage
- Managed services reduce labor overhead
- No dedicated infrastructure team required

### Schedule Risk

Implementation timeline may extend beyond academic planning phase.

Mitigation:
- Phased implementation roadmap
- Prioritize core intake + billing grouping first before automation layers

### What This Means for the Business

The design is intentionally layered:

- Core system delivers immediate value
- Translation and automation layers enhance efficiency over time

This prevents overcommitment while preserving long-term potential.

---

## 5. Organizational and Adoption Risk

Even well-designed systems fail if users resist them.

Mitigation strategies include:

- Preserving the structure of the existing paper form
- Minimal role complexity
- Clear cancellation rules
- Simple daily driver views
- Reduced cognitive burden during billing through translation outputs

Residual risk remains dependent on change management and leadership support.

### What This Means for the Business

The system does not force dramatic workflow reinvention.

Instead, it removes friction from the most repetitive and error-prone tasks.

This increases the likelihood of adoption and sustained use.

---

# Overall Risk Posture

This project intentionally shifts risk from:

- Informal manual processes  
to  
- Structured architectural enforcement  

And further evolves toward:

- Reduced cognitive dependency in repetitive workflows  
- Increased visibility and auditability in post-processing  

The highest-impact risks (data exposure, billing integrity, tenant mixing) are mitigated at the system level.

Remaining risks are primarily:

- Organizational adoption
- Scope discipline
- Implementation cost control

These are governance matters rather than architectural weaknesses.

---

# Conclusion

The overall residual risk profile of the proposed system is low-to-moderate and manageable.

The design reduces preventable security, billing, and operational risks while maintaining scalability and cost awareness.

By embedding controls into architecture and separating cognitive workflows, the system improves both accuracy and efficiency.

Risk is not eliminated.

It is structured, identified, and controlled.
