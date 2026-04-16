# Problem Statement

## Business Context
A small, owner-operated medical transportation business relies on phone calls and paper forms to schedule rides and manage billing.

The business serves both healthcare facilities and individual/private-pay riders. Requests may originate from facility staff, repeat patients, or one-time callers.

## Current Workflow
Ride requests are taken by phone and recorded on paper forms. These forms are used for dispatching drivers and later referenced for billing.

Individual/private-pay riders and healthcare facilities are handled using the same paper process, but billing and grouping differ depending on the requester type.

Dispatching requires active decision-making, including scheduling, coordination, and exception handling. Billing, however, is a separate downstream activity that requires reconstructing completed rides from paper records and re-entering them into billing software.

## Core Problem
The most significant challenge is the manual sorting and assembling of completed rides by patient and facility for billing, followed by retyping the same data into billing software. This process is time-consuming and delays invoicing.

Because both facilities and individual riders generate repeat trips, the lack of structured tenant grouping increases administrative friction and slows billing cycles.

Additionally, the workflow requires users to perform two fundamentally different types of work using the same data:

- **Dispatching**, which requires real-time decision-making and contextual awareness  
- **Billing**, which requires repetitive reconstruction and transcription of already-known information  

This mismatch introduces unnecessary cognitive load, increases the likelihood of errors, and slows processing speed.

## Impact
Billing delays create cashflow stress and unnecessary operational burden for the business owner.

Manual tracking also increases the risk of documentation inconsistencies, which can affect facility trust and private-pay rider transparency.

The need to repeatedly reconstruct billing data from operational records leads to:

- Increased administrative time  
- Higher risk of missed or duplicated trips  
- User fatigue during repetitive tasks  
- Reduced operational efficiency as trip volume grows  

Over time, these inefficiencies compound, limiting the business’s ability to scale without increasing administrative workload.

---

## Problem Framing (Refined)

The problem is not simply that the workflow is paper-based.

The underlying issue is that:

- Operational data is captured in an unstructured format  
- Billing requires reconstruction instead of reuse  
- Cognitive effort is repeated unnecessarily across workflow stages  

As a result, the system lacks:

- Structured tenant-based grouping (facility vs individual)  
- A persistent, queryable record of completed trips  
- A mechanism to translate operational data into billing-ready outputs  

---

## Design Implication

A successful solution must:

- Capture structured data at the point of intake and dispatch  
- Preserve clear tenant association for every trip  
- Eliminate the need to reconstruct billing information  
- Separate decision-making tasks from repetitive execution tasks  
- Support a workflow where billing becomes a review and approval process rather than a manual transcription process  
