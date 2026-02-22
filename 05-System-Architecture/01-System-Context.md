# System Context

## Purpose

This platform supports scheduling, documentation, and billing preparation for a non-emergency medical transportation operation operating in healthcare-adjacent environments.

The system replaces a paper-based intake and trip documentation workflow with a structured, role-based, secure digital platform.

## Core Capabilities

- Trip request intake and validation
- Automatic generation of trip legs (one-way or round trip)
- Driver-side execution and documentation logging
- Billing grouping by facility, patient, and period
- Patient duplicate detection and reversible merges
- Audit-safe status tracking

## Operating Environment

The system serves a small-to-midsize transportation business with:

- Multiple drivers
- A dispatcher/owner
- A billing function
- Repeated facilities and patients
- Healthcare-adjacent documentation sensitivity

The architecture must assume:

- Limited internal IT support
- Moderate but steady operational use
- Long-term record retention
- Legal defensibility of trip documentation

## System Boundaries

The system does NOT:

- Integrate directly with accounting platforms (exports only)
- Provide GPS tracking
- Provide real-time route optimization
- Act as a claims processor
- Store medical records beyond necessary transport documentation

The system DOES:

- Store transport-related personal identifiers
- Store billing-relevant documentation
- Enforce role-based access control
- Preserve historical data without destructive edits
