# High-Level Architecture

## Architectural Style

The system uses a layered, serverless architecture deployed within a VPC-isolated cloud environment.

Logical Flow:

Client → CDN → Static Frontend → API Layer → Business Logic → Relational Database

Supporting components include:

- Identity provider
- Object storage
- Logging and monitoring services

---

## Frontend Layer

### Responsibilities

- Render dispatcher, driver, and billing views
- Collect and validate basic client-side inputs
- Submit structured requests to backend APIs
- Display role-filtered data

### Implementation

- Static frontend hosted in object storage
- Content delivered via CDN
- HTTPS enforced

This design eliminates the need for managed web servers and reduces attack surface.

---

## API Layer

### Responsibilities

- Expose REST endpoints
- Validate authentication tokens
- Route requests to application logic
- Log request activity

All external traffic enters through this controlled gateway.

---

## Application Layer (Serverless Compute)

### Responsibilities

- Trip request creation and validation
- Trip leg generation
- Merge and unmerge logic
- Billing grouping logic
- Status transitions
- Enforcement of role permissions

This layer centralizes all business rules and prevents direct database access from clients.

The serverless model was selected due to event-driven workload characteristics and low steady-state compute demand.

---

## Database Layer (Relational)

### Responsibilities

- Persist structured transport and billing data
- Enforce referential integrity
- Support grouped billing queries
- Maintain transactional safety

The database resides in a private subnet and is not publicly accessible.

---

## Object Storage

### Responsibilities

- Store PDF exports
- Store billing summaries
- Store signature images (future capability)
- Support lifecycle retention policies

Object storage is separated from structured relational data.

---

## Identity and Access Management

Role-based access control is enforced using token-based authentication.

Roles include:

- Dispatcher
- Driver
- Billing

Authorization decisions are evaluated before business logic execution.

---

## Logging and Monitoring

Logging is enabled at:

- API request layer
- Application execution layer
- Database audit layer

This supports dispute resolution and operational debugging.
