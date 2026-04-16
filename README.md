# medical-transportation-system-design

Design and planning of a cloud-based, human-centered system to replace a paper-based workflow for a small medical transportation business.

This repository documents the design and planning of a cloud-based system intended to replace a paper-based workflow for a small, owner-operated medical transportation business.

The project focuses on system architecture, database design, human-centered form design, and project management planning. The goal is to improve scheduling visibility, billing preparation, and data organization while preserving familiar workflows for non-technical users.

This version of the project also introduces structured workflow improvements, including:
- Separation of dispatch decision-making from billing execution
- A translation layer that converts operational data into invoice-ready outputs
- Approval and audit workflows to preserve data integrity while reducing cognitive load

---

## Project Status

Design and planning phase (no production system implemented)

---

## Key Focus Areas

- Human-centered and inclusive system design  
- AWS-based system architecture  
- Data organization to support accurate billing  
- Cognitive load reduction in repetitive administrative workflows  
- Translation of operational data into billing-ready formats  
- Scope-controlled project management and planning  

---

## Background

The current workflow relies on phone calls, paper forms, and manual re-entry of data into billing software. The most significant challenge is assembling completed rides by patient and facility for billing, which causes delays and unnecessary operational strain.

In the current state:
- Dispatching requires active decision-making and contextual awareness  
- Billing requires repetitive reconstruction of information  
- Cognitive fatigue leads to slower processing and higher error risk  

The system is designed not only to digitize this workflow, but to restructure it.

Instead of requiring users to reconstruct billing information at the end of the month, the system captures structured data during dispatch and converts it into billing-ready outputs.

This allows billing to function as a:
- Review  
- Approval  
- Copy/paste or export task  

rather than a manual reconstruction process.

---

## System Design Philosophy

This project is built on three core principles:

### 1. Preserve Familiar Workflows
- Maintain the structure and language of the paper form
- Minimize training requirements
- Support non-technical users

### 2. Enforce Structural Integrity
- Every trip is associated with a tenant (facility or individual)
- Completion data is immutable
- Billing states are structured and auditable

### 3. Reduce Cognitive Load
- Separate high-effort decision-making (dispatch) from repetitive tasks (billing)
- Introduce a translation layer to eliminate redundant thinking
- Support approval and edit workflows with full audit visibility

---

## What This System Solves

This system directly addresses:

- Billing delays caused by manual grouping and re-entry  
- Documentation inconsistency and dispute risk  
- Lack of structured, queryable operational data  
- Cognitive fatigue from repetitive administrative tasks  

It transforms the workflow from:

**Manual reconstruction → Structured data reuse → Verified output**

---

## Product Development Lifecycle Alignment

This project follows a structured Product Development Lifecycle (PDLC) approach:

- Discovery & Ideation: problem framing and stakeholder analysis  
- Requirements & Planning: functional and nonfunctional requirements  
- Design: human-centered design, system architecture, and data modeling  
- Development, Testing, Deployment: planned for post-graduation implementation  
- Maintenance & Iteration: documented through reflection and future roadmap  

---

## Future Implementation Direction

Planned future phases include:

- Frontend development (React-based UI)  
- Backend implementation (API + database integration)  
- AWS deployment (serverless architecture)  
- Billing workflow enhancements (translation → approval → export)  
- Optional integration with accounting systems (e.g., QuickBooks)  

---

## Repository Structure

This repository is organized into:

- Stakeholder and requirements documentation  
- Human-centered design artifacts (personas, flows, task analysis)  
- System architecture and AWS design decisions  
- Data model and form-to-structure mapping  
- Security, risk, and performance planning  

Each section reflects a stage in the system design process and contributes to a complete, implementation-ready blueprint.

---

## Summary

This project is not simply a website redesign.

It is a structured system design that:

- Improves operational visibility  
- Reduces billing time and errors  
- Preserves data integrity and auditability  
- Reduces cognitive load for non-technical users  
- Creates a scalable foundation for future automation  

The result is a system that transforms manual workflows into structured operational infrastructure, enabling efficiency, accuracy, and long-term growth.
