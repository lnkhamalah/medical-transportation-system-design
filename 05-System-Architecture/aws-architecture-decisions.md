# AWS Architecture Decisions (Design-Level)

This document explains the AWS services selected for the system and why each choice supports business realism, security boundaries, and long-term maintainability.

The design goal is a believable architecture that a small organization can operate with minimal infrastructure management.

---

## Architecture Goals

- Support public guest intake without revealing internal data  
- Support facility portal access with strict tenant scoping  
- Support internal operational access with role-based controls  
- Preserve data integrity for billing and disputes  
- Use a relational database foundation  
- Keep operating overhead low  

---

## Service Selections and Rationale

### Frontend

- **Amazon S3** (static hosting)  
- **Amazon CloudFront** (CDN + TLS)  
- **Amazon Route 53** (DNS)  

**Reasoning:**

- Low cost  
- Low maintenance  
- Secure edge delivery  
- Separation of UI from backend  

### Private Service Connectivity

- **VPC Interface Endpoints (SNS, SES, SQS)**

**Reasoning:**

- Allows private communication between Lambda and AWS services
- Eliminates need for public internet routing or NAT Gateway
- Reduces attack surface
- Keeps service-to-service communication within AWS private networking
  
---

### API Entry

- **Amazon API Gateway**

**Reasoning:**

- Single entry point  
- Token validation  
- Throttling, request rate limits, and payload size limits  
- Integration with AWS WAF for bot mitigation and abuse protection   
- Clear separation of public vs authenticated routes  

---

### Application Logic

- **AWS Lambda**

**Reasoning:**

- Serverless scaling  
- Centralized enforcement of business rules  
- Lambda enforces role-based authorization and tenant scoping using claims extracted from the validated Cognito JWT  
- Facility scoping is derived from token claims and never from client-supplied request data  
- No infrastructure management burden  
- Cost aligned with usage patterns
- Supports asynchronous event publishing to Amazon SQS for resilient notification processing  
- Prevents user-facing operations from being blocked by external notification delivery failures
  

---

### Database

- **Amazon RDS (PostgreSQL, Multi-AZ)**
- **Amazon RDS Proxy**

**Reasoning:**

- Relational data model requires joins  
- Strong ACID guarantees  
- Billing grouping and audit queries depend on relational consistency  
- Managed backups and patching  

RDS is deployed in a private subnet inside a VPC.  
It has no public endpoint and accepts connections only from the RDS Proxy security group.

Application compute connects to the database through Amazon RDS Proxy to:
- Manage connection pooling for Lambda concurrency
- Protect the database from connection exhaustion
- Centralize credential handling  
Direct client-to-database access is not permitted.

**DynamoDB was not selected because:**

- Data model is relational  
- Billing workflows require grouping across entities  
- Referential integrity is critical  

---

### Identity

- **Amazon Cognito (single User Pool)**

**Reasoning:**

- Managed authentication  
- Supports role groups  
- Supports custom tenant attributes  
- Integrates with API Gateway authorizers  

---

### Storage

- **Amazon S3 (private buckets)**

**Reasoning:**

- Stores unstructured documents  
- Separates structured relational data from exports  
- Supports lifecycle retention policies  

---

### Notifications

- **Amazon SQS** (notification queue)
- **AWS Lambda (Notification Processor)**
- **Amazon SNS** (SMS receipts)
- **Amazon SES** (email confirmations)

**Reasoning:**

- Decouples notification delivery from primary business logic
- Prevents trip submission failures due to SMS/email outages
- Allows retry logic and independent scaling of notification processing
- Ensures operational alerts (e.g., cancellations) reach both dispatcher and facility contact 

---

### Logging

- **Amazon CloudWatch**  
- **AWS CloudTrail**

**Reasoning:**

- Operational debugging  
- Audit visibility  
- Low administrative overhead  

---

## Trust Boundaries

The system enforces three trust zones:

1. **Public Internet (untrusted)**  
2. **Application Layer (controlled compute)**  
3. **Data Layer (restricted persistence)**  

**Rules:**

- Public guest intake is write-only and cannot query existing data  
- Authenticated facility portal access is tenant-scoped using `facility_id` claims from Cognito tokens  
- Internal users operate with broader permissions but are still enforced through server-side authorization logic  
- All authorization checks occur inside Lambda before database queries are executed  

---

## Summary

This AWS design is intentionally minimal but realistic:

- Static frontend with secure backend APIs  
- Serverless business logic for low operational overhead  
- Relational database foundation for integrity and billing workflows  
- Cognito-based identity with strict server-side authorization  
- Clear separation between public intake, facility portal, and internal operations
- Asynchronous notification processing for operational resilience  
- RDS Proxy protection against connection spikes  
- WAF and API throttling for edge-layer defense  
- Multi-AZ database deployment for availability  
- Private VPC endpoints for secure service communication  
