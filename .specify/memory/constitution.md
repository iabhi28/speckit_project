<!--
Sync Impact Report
Version change: 1.0.0 -> 1.1.0
Modified principles:
- Added VI. Microservices and Service Boundaries
- Added VII. Documentation and Input Validation
Added sections: None
Removed sections: None
Follow-up TODOs: None
-->

# speckit_project Constitution

## Core Principles

### I. SOLID and Clean Code
All production code MUST follow SOLID principles where applicable. Each module MUST have a
clear, cohesive responsibility, expose deliberate interfaces, and avoid coupling that makes
independent change or testing difficult. Names and control flow MUST make intent clear. Code
MUST avoid duplication and unnecessary abstractions; deviations from SOLID MUST be justified
by the actual requirements rather than convention.

### II. Test Quality Above 90%
Automated tests MUST cover behavior and regression risks, not merely execute code. The project
MUST maintain more than 90% coverage for statements, branches, functions, and lines. Coverage
MUST be measured by the project test tooling and enforced as a quality gate; exclusions MUST
be narrow, explicit, and justified. Changes MUST include or update tests for their behavior.

### III. Simple, Responsive User Experience
User journeys MUST be understandable without unnecessary steps or choices. Interfaces MUST
work at supported mobile, tablet, and desktop viewport sizes, with no avoidable horizontal
overflow or inaccessible controls. Interactive elements MUST provide clear labels, visible
keyboard focus, and usable touch targets. Visual consistency MUST support, not obscure,
content and task completion.

### IV. Minimal Dependencies
The project MUST prefer existing platform capabilities and installed libraries over adding
dependencies. A new dependency MUST have a documented need that cannot reasonably be met by
the current stack, and MUST be reviewed for maintenance, compatibility, and security impact.
Development-only needs MUST NOT become runtime dependencies.

### V. Approved Platform and MySQL
The application MUST use Next.js, React, and Tailwind CSS at the versions declared in the
project's `package.json`. Framework upgrades MUST be deliberate dependency changes and MUST
pass the project's validation gates. MySQL MUST be the database for durable application data;
schema changes MUST be versioned and repeatable, and database access MUST use parameterized
queries or an equivalent safe data-access mechanism.

### VI. Microservices and Service Boundaries
The application MUST be designed and implemented as microservices with clearly defined,
cohesive service responsibilities. Each service MUST expose explicit, versioned contracts
for communication and MUST NOT depend on another service's internal implementation details.
Services MUST be independently testable and deployable. Persistent data ownership and
cross-service access MUST be explicit; a service MUST NOT bypass another service's contract
to access its private data.

### VII. Documentation and Input Validation
Production code MUST be documented sufficiently for another engineer to understand and safely
use and maintain it. Public interfaces, service contracts, configuration, and non-obvious
business rules MUST document purpose, inputs, outputs, errors, and relevant constraints.
Documentation MUST be updated with behavior or contract changes. All untrusted inputs MUST
be validated at the boundary where they enter a service, including type, format, size, and
allowed values as applicable. Invalid input MUST be rejected explicitly and MUST NOT be
silently coerced into success-shaped results.

## Technology and Data Constraints

`package.json` is the source of truth for the approved Next.js, React, and Tailwind CSS
versions. Persistent application records MUST be stored in MySQL. Browser storage MAY be used
for client-only convenience state, but MUST NOT be treated as the durable database or a
substitute for MySQL-backed records. Database credentials MUST be provided through runtime
configuration and MUST NOT be committed to the repository.

## Development and Quality Workflow

Implementation plans and changes MUST identify the requirements they satisfy. Changes MUST
be reviewed for principle compliance and validated with the smallest relevant tests, lint,
type-check, and build commands available in the project. A change MUST NOT be considered
complete when its relevant checks fail or when coverage falls to 90% or below in any required
metric. Any exception MUST be documented with its reason and scope and approved during review.

## Governance

This constitution takes precedence over conflicting project practices. Amendments MUST be
proposed as explicit changes to this document, reviewed for impact on existing work, and
updated with an amendment date and version. Changes to principles or constraints MUST include
a corresponding version increment and a concise impact report.

Versioning follows semantic versioning: MAJOR for incompatible governance changes or removal
of a principle; MINOR for a new principle or materially expanded obligation; PATCH for
clarifications and non-semantic corrections. Contributors and reviewers MUST check applicable
requirements against this constitution during planning, implementation, and review. A
non-compliant change MUST be revised or receive a documented, review-approved exception before
it is accepted.

**Version**: 1.1.0 | **Ratified**: 2026-10-14 | **Last Amended**: 2026-10-14
