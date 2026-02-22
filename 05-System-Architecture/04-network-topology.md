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

- Public Subnet
- Private Subnet (Application)
- Private Subnet (Database)

Each subnet is associated with distinct routing rules and security groups.

---

## Public Subnet

### Components

- CloudFront (edge layer, managed)
- API Gateway (managed entry point)

### Purpose

The public subnet handles inbound HTTPS traffic from users.

No persistence components reside in this subnet.

No database access is allowed from this subnet.

---

## Application Subnet (Private)

### Components

- AWS Lambda functions (attached to VPC)

### Purpose

Lambda functions execute business logic and are the only components allowed to communicate with the database.

These functions are not publicly accessible directly.
They are invoked only through API Gateway.

### Security Controls

- Security group allowing outbound access only to:
  - Database security group
- No inbound internet access
- No public IP assignment

---

## Database Subnet (Private)

### Component

- Amazon RDS (PostgreSQL)

### Isolation Strategy

The database:

- Resides in a private subnet
- Has no public endpoint
- Is not accessible from the internet
- Only accepts connections from the Lambda security group

This enforces a strict one-way trust boundary:

Client → API → Lambda → Database

Direct Client → Database communication is impossible by design.

---

## Routing

- Public subnet routes to Internet Gateway
- Private subnets route through NAT Gateway (if outbound internet required)
- Database subnet does not require direct internet access

---

## Security Groups

### API Layer

- Allows HTTPS (443) inbound
- Forwards authenticated requests to Lambda

### Lambda Security Group

- No public inbound rules
- Outbound allowed only to:
  - RDS security group on database port (e.g., 5432)

### RDS Security Group

- Inbound allowed only from:
  - Lambda security group
- No public inbound traffic permitted

---

## Trust Boundaries

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

Even if frontend code were compromised, the attacker would still need:

- Valid authentication
- Valid role authorization
- Access to API Gateway
- Lambda execution context
- Correct security group permissions

Network isolation reinforces application-level security rather than relying on it alone.
