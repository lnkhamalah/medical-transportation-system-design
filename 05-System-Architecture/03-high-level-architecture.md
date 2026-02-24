# High-Level Architecture

## Architectural Style

The system uses a layered, serverless architecture deployed within a VPC-isolated cloud environment.

Logical Flow:

Client → CDN → Static Frontend → API Gateway → Lambda → Relational Database

Supporting components include:

- Identity provider
- Object storage
- Notification services
- Logging and monitoring services

---

## Frontend Layer

### Responsibilities

- Render dispatcher, driver, billing, facility portal, and individual views
- Collect and validate basic client-side inputs
- Submit structured requests to backend APIs
- Display role-filtered and tenant-filtered data

### Implementation

- Static frontend hosted in Amazon S3
- Delivered via Amazon CloudFront (CDN + TLS)
- HTTPS enforced

This design eliminates the need for managed web servers and reduces attack surface.

---

## API Layer

### Responsibilities

- Expose REST endpoints
- Validate authentication tokens (when required)
- Route requests to application logic
- Log request metadata

All external traffic enters through this controlled gateway.

Public intake endpoints are write-only.

Authenticated endpoints require valid tokens.

---

## Application Layer (Serverless Compute)

### Responsibilities

- Trip request creation and validation
- Requester type enforcement (Facility vs Individual)
- Automatic tenant association
- Trip leg generation
- Merge and unmerge logic (internal only)
- Billing grouping logic
- Status transitions
- Enforcement of role permissions
- Enforcement of tenant isolation

This layer centralizes all business rules and prevents direct database access from clients.

The serverless model was selected due to event-driven workload characteristics and low steady-state compute demand.

---

## Database Layer (Relational)

### Responsibilities

- Persist structured transport and billing data
- Enforce referential integrity
- Support grouped billing queries
- Maintain transactional safety
- Enforce tenant association on all TripRequests

The database resides in a private subnet and is not publicly accessible.

PostgreSQL is preferred due to strong relational support and maturity.

---

## Object Storage

### Responsibilities

- Store PDF exports
- Store billing summaries
- Store signature images (future capability)
- Store form snapshot archives

Object storage is separated from structured relational data.

All buckets block public access by default.

---

## Identity and Access Management

Role-based access control is enforced using token-based authentication via Amazon Cognito.

### Single User Pool Design

- One Cognito User Pool
- Roles stored as Cognito Groups
- Tenant scoping stored as custom attributes (e.g., tenant_id, contact_id)

User categories:

- Dispatcher
- Driver
- Billing
- FacilityAdmin
- FacilityUser

Public intake does not authenticate.

All authorization decisions are enforced server-side in Lambda.

---

## Notifications

Upon submission of a TripRequest:

- A full copy is sent to the dispatcher/billing inbox (trusted internal path)
- A confirmation receipt is sent to the requester

SMS messages contain receipt-level content only (no sensitive detail).  
Email may contain operational details but avoids clinical context.

Notification events are logged.

---

## Logging and Monitoring

Logging is enabled at:

- API request layer
- Application execution layer
- Database audit layer

Logs capture metadata and action history but avoid storing PHI in log payloads.

This supports dispute resolution and operational debugging.
