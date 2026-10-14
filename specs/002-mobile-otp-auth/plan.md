# Implementation Plan: Mobile OTP Authentication

**Branch**: `002-mobile-otp-auth` | **Date**: 2026-10-14 | **Spec**: [./spec.md](./spec.md)

**Input**: Feature specification from `/specs/002-mobile-otp-auth/spec.md`

## Summary

Implement a unified customer sign-in/registration flow using normalized Indian mobile numbers and SMS one-time codes. An independently deployable Identity service owns account, challenge, attempt-limit, and session state in MySQL, integrates with an SMS provider through a replaceable delivery adapter, and exposes versioned authentication contracts to the Next.js UI.

## Technical Context

**Language/Version**: TypeScript, Next.js 16, React 19, Tailwind CSS 4

**Primary Dependencies**: Existing Next.js/React stack; `mysql2` for the Identity service's MySQL access; SMS provider SDK remains a service deployment choice behind an adapter

**Storage**: Identity service-owned MySQL records for customer accounts, OTP challenges, and sessions; secure HTTP-only cookies carry session identifiers

**Testing**: ESLint plus Vitest and V8 coverage; enforce >90% statement, branch, function, and line coverage

**Target Platform**: Responsive web browsers on mobile, tablet, and desktop

**Project Type**: Next.js customer authentication UI integrated with an independently deployable Identity microservice

**Performance Goals**: At least 95% of valid-code acceptance tests reach authenticated state within 10 seconds; invalid attempts never create sessions

**Constraints**: India-only mobile numbers; code lifetime five minutes; resend cooldown 60 seconds; five incorrect attempts cause a 15-minute block; per-number and per-source request rate limits; no account-existence disclosure before verification

**Scale/Scope**: Single customer identity domain with unified registration/sign-in, SMS delivery, session creation, and return-to-destination routing

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- Pass: The page uses the declared Next.js, React, and Tailwind versions.
- Pass: Identity state and customer records are owned by a separately deployable Identity service and stored in MySQL; no other service reads its private tables.
- Pass: Identity endpoints are versioned, validate all input, and provide explicit errors.
- Pass: Passwordless authentication only establishes a session after one-time code verification.
- Pass: SMS integration is isolated behind an adapter so the domain contract is not coupled to a vendor.
- Pass: Accessible/responsive controls and test coverage above 90% for every required metric are included in the design.

## Project Structure

```text
app/
└── auth/
    └── page.tsx
components/
└── auth/
    ├── mobile-entry-form.tsx
    ├── otp-verification-form.tsx
    └── resend-countdown.tsx
lib/
├── api/
│   └── identity-client.ts
└── validation/
    └── indian-mobile.ts
tests/
├── unit/
├── integration/
└── e2e/
```

**Structure Decision**: The Next.js app owns only authentication presentation and a typed Identity API client. The Identity service owns account/challenge/session tables and rate limiting; the SMS provider is reached only by that service. This preserves independently deployable service boundaries and avoids exposing provider credentials to the browser.

## Complexity Tracking

No constitution violations. The service and delivery adapter are required to isolate private identity data and provider credentials.
