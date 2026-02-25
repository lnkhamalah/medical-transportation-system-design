# Network Topology

## Overview

The platform is deployed inside a Virtual Private Cloud (VPC) to enforce network-level isolation of the persistence layer and restrict lateral movement between components.

The design separates:

- Public-facing components
- Application compute
- Private data storage

This separation reduces attack surface and ensures that the relational database is never directly accessible from the public internet.

---

## VPC Structure

The system uses a single VPC with the following segmentation:

- Public-facing managed entry services
- Private Application Subnet
- Private Database Subnet

The VPC spans multiple Availability Zones to support high availability for database and compute resources.

Database components are deployed in a multi-AZ configuration to prevent single-zone failure from impacting availability.
---

## Public Edge Layer

### Components

- Amazon CloudFront
- AWS WAF (attached to CloudFront / API Gateway)
- Amazon API Gateway

### Purpose

Handles inbound HTTPS traffic.

AWS WAF provides request filtering, rate-based protections, and bot mitigation before traffic reaches API Gateway.

No persistence components reside here.

No direct database access is permitted.

---

## Application Subnet (Private)

### Components

- Primary AWS Lambda functions (business logic)
- Notification Lambda
- Amazon SQS (notification queue)
- Amazon RDS Proxy
- VPC Interface Endpoints (SNS, SES, SQS)

### Purpose

Primary Lambda functions execute business logic and are the only components allowed to communicate with the database through Amazon RDS Proxy.

Notification Lambda consumes events from Amazon SQS and dispatches outbound notifications via SNS and SES using VPC Interface Endpoints.

No component in the application subnet exposes a public endpoint.

These functions are not publicly accessible directly.  
They are invoked only through API Gateway.

### Security Controls

- No public IP assignment
- Security groups allowing:
  - Lambda → RDS Proxy
  - RDS Proxy → RDS
  - Lambda → SQS
  - Lambda → VPC Endpoints (SNS, SES, SQS)
- No inbound internet access
- No inbound internet access

---

## Database Subnet (Private)

### Component

- Amazon RDS (PostgreSQL, Multi-AZ)

### Isolation Strategy

The database:

- Resides in a private subnet
- Has no public endpoint
- Is not accessible from the internet
- Accepts connections only from the RDS Proxy security group
  

This enforces a strict one-way trust boundary:

Client → API → Lambda → RDS Proxy → Database

Direct Client → Database communication is impossible by design.

---

## Trust Zones

There are three trust zones:

1. Public Internet  
2. Application Layer (controlled compute)  
3. Data Layer (restricted persistence)  

Movement between zones requires authenticated and authorized transitions.

All service-to-service communication within the VPC occurs over private networking. No database, queue, or notification service is directly accessible from the public internet.

---

## Why This Topology Matters

This topology enforces:

- Database isolation
- Least privilege access
- Segmented trust zones
- Defense-in-depth security

Even if frontend code were compromised, an attacker would still need:

- Valid authentication (for protected endpoints)
- Valid role authorization
- Access to API Gateway
- Lambda execution context
- Correct security group permissions

Network isolation reinforces application-level security rather than relying on it alone.
