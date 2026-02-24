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

---

## Public Edge Layer

### Components

- Amazon CloudFront
- Amazon API Gateway

### Purpose

Handles inbound HTTPS traffic.

No persistence components reside here.

No direct database access is permitted.

---

## Application Subnet (Private)

### Components

- AWS Lambda functions attached to VPC

### Purpose

Lambda functions execute business logic and are the only components allowed to communicate with the database.

These functions are not publicly accessible directly.  
They are invoked only through API Gateway.

### Security Controls

- No public IP assignment
- Security group allowing outbound access only to the RDS security group
- No inbound internet access

---

## Database Subnet (Private)

### Component

- Amazon RDS (PostgreSQL)

### Isolation Strategy

The database:

- Resides in a private subnet
- Has no public endpoint
- Is not accessible from the internet
- Accepts connections only from the Lambda security group

This enforces a strict one-way trust boundary:

Client → API → Lambda → Database

Direct Client → Database communication is impossible by design.

---

## Trust Zones

There are three trust zones:

1. Public Internet  
2. Application Layer (controlled compute)  
3. Data Layer (restricted persistence)  

Movement between zones requires authenticated and authorized transitions.

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
