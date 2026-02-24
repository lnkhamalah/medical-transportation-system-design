# Paper-Form to Digital-Form Design Principles

This system is intended to replace a paper-based ride intake form with a digital version that preserves the existing workflow. The primary design goal is familiarity: the digital form should feel like the paper form, not like a new system users must “learn.”

The digital system must also support structured tenant isolation while maintaining the visual simplicity of the paper process.

## Design Principles

- **Preserve the paper form order**
  - Keep the field sequence and section structure consistent with the existing paper form.
  - Users should be able to predict what comes next without searching.

- **Familiar language and labels**
  - Use the same wording as the paper form (e.g., “Facility Name,” “Trip Information,” “Official Use Only”).
  - Avoid technical terms that users do not already use in daily work.

- **Minimal structural additions**
  - Introduce a clear Requester Type selection (Healthcare Facility or Individual / Private Pay).
  - Automatically associate each request with exactly one tenant account.
  - Prevent unscoped or shared “none” tenant records.

- **Minimize training requirements**
  - The form should be usable without a class or manual.
  - Provide subtle guidance (examples, placeholders) rather than long instructions.

- **Reduce retyping**
  - Reuse known data (repeat patients, facilities, and contacts) when possible.
  - Individual/private-pay riders may also be reused as tenant records.
  - The system should prevent repetitive work while keeping user control.

- **Support real workflow timing**
  - Round trips are two separate legs and may occur hours apart.
  - A return leg may be cancelled (e.g., patient admitted), and the system must handle this cleanly.

## Outcome of These Principles

If these principles are followed, the digital form will:
- Reduce cognitive burden
- Protect documentation quality
- Support billing and scheduling without disrupting the current business workflow
- Enforce clean tenant isolation for both facilities and individual riders
