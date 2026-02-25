# High-Level Architecture

## Architectural Style

The system uses a layered, serverless architecture deployed within a VPC-isolated cloud environment.

Logical Flow:

Client → CDN → Static Frontend → API Gateway → Lambda → RDS Proxy → Relational Database

Public Ingress Protection Flow:

Client → CloudFront → AWS WAF → API Gateway (Rate Limited) → Lambda

Public intake endpoints additionally enforce:

- CAPTCHA validation before request submission
- Per-IP rate limiting
- Payload size restrictions
- Bot and reputation filtering via AWS WAF

Asynchronous Notification Flow:

Lambda → SQS → Notification Lambda → SNS / SES

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

API Gateway enforces:

- Request throttling and rate limits
- Payload size limits
- JWT token validation (for authenticated routes)
- Integration with AWS WAF for abuse and bot protection

All public endpoints apply strict input validation rules before invoking application logic.

Public intake endpoints also enforce CAPTCHA verification to reduce automated abuse.

Rate-based rules within AWS WAF automatically block excessive or suspicious request patterns before Lambda invocation.

This ensures that automated spam or denial-of-service attempts cannot trigger large-scale database writes or notification floods.

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
- Cancellation request workflow enforcement
- Same-day cancellation restriction logic
- Asynchronous event publishing for notification dispatch
- Public intake submissions are initially marked with an internal "unverified" status until reviewed or confirmed by dispatch

This layer centralizes all business rules and prevents direct database access from clients.

All cancellation requests initiated by facility users are treated as request-based state transitions. 

Same-day cancellation attempts are automatically denied and instruct the facility to contact dispatch by phone. This prevents operational disruption when dispatch staff may not be actively monitoring the portal.

Similarly, public intake submissions do not automatically assign drivers or trigger billing workflows. Dispatcher confirmation is required before operational scheduling proceeds.

This prevents automated or malicious submissions from entering active workflow queues.

Approved cancellations trigger asynchronous notification events to both the dispatcher and the listed contact.


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

Application compute connects to the database through Amazon RDS Proxy to manage connection pooling and protect the database from Lambda concurrency spikes.

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

The system uses an asynchronous notification pattern to prevent user-facing delays and to improve resiliency.

Upon submission or state transition of a TripRequest:

1. The primary Lambda publishes a notification event to Amazon SQS.
2. A dedicated Notification Lambda consumes the queue.
3. The Notification Lambda sends:
   - SMS via Amazon SNS
   - Email via Amazon SES

Notification content is intentionally segmented:

- Requesters receive structured confirmation summaries containing service date, time window, and confirmation reference.
- Dispatcher receives a full intake snapshot copy for operational redundancy.
- A canonical PDF snapshot of each intake submission is stored in encrypted S3 for immutable archival.

Sensitive internal metadata or billing-related flags are not transmitted in external emails.

This design ensures:

- Trip creation does not fail if SMS or email services are temporarily unavailable
- Notification retries can occur independently
- Operational workflows are not blocked by third-party delivery delays

### Cancellation Notifications

When a cancellation is approved:

- A notification is sent to the facility contact
- A notification is sent to the dispatcher

Same-day cancellation attempts are denied automatically and instruct the facility to call dispatch directly.

---

## Logging and Monitoring

Logging is enabled at:

- API request layer
- Application execution layer
- Database audit layer

Logs capture metadata and action history but avoid storing PHI in log payloads.

This supports dispute resolution and operational debugging.

Monitoring also includes detection of abnormal request spikes, rate-limit triggers, and repeated failed CAPTCHA validations.

These signals can be used to temporarily tighten rate limits or block suspicious IP ranges via AWS WAF without interrupting legitimate operational traffic.

