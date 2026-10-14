# Implementation Plan: Checkout Page

**Branch**: `005-checkout-page` | **Date**: 2026-10-14 | **Spec**: [./spec.md](./spec.md)

**Input**: Feature specification from `/specs/005-checkout-page/spec.md`

## Summary

Build a responsive checkout that displays database-backed cart items, current prices, quantities, and authoritative totals; allows quantity edits/removal; validates name, India-only mobile number, and delivery address; and submits one order through the versioned Order service contract. The Order service coordinates with Catalog and Cart services, snapshots order details, and clears the cart only after a successful order commit. Payment processing is outside this feature.

## Technical Context

**Language/Version**: TypeScript, Next.js 16, React 19, Tailwind CSS 4

**Primary Dependencies**: Existing Next.js/React stack; `mysql2` in the owning service implementations; no payment provider dependency

**Storage**: Cart service owns carts/items; Catalog owns product availability/pricing; Order service owns orders/items and fulfillment snapshots, all in MySQL

**Testing**: ESLint plus Vitest and V8 coverage; enforce >90% statement, branch, function, and line coverage

**Target Platform**: Responsive web browsers, with stacked mobile and two-column larger-screen layout

**Project Type**: Next.js checkout feature integrated with independently deployable Cart, Catalog, and Order microservices

**Performance Goals**: Displayed totals must equal displayed line totals; one in-flight submission creates at most one order; successful order confirmation follows a persisted order result

**Constraints**: Cart cookie contains only an opaque identifier; server recomputes prices and totals; unavailable/changed items must be surfaced for review; no payment processing, shipping charge, tax, or discount behavior is defined

**Scale/Scope**: Cart review/edit, three required customer fields, validation, order placement, explicit recovery, and confirmation

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- Pass: Uses the existing approved Next.js/React/Tailwind versions.
- Pass: Cart, Catalog, and Order are separate data-owning services with versioned contracts; order submission does not directly query another service's private tables.
- Pass: MySQL is authoritative for carts, product prices, and orders; browser storage is not trusted for cart contents or totals.
- Pass: Boundary validation, explicit service failures, and idempotent order submission are included.
- Pass: Accessible responsive form/list behavior and >90% coverage for every required metric are planned.

## Project Structure

```text
app/
└── checkout/
    └── page.tsx
components/
└── checkout/
    ├── customer-details-form.tsx
    ├── order-summary.tsx
    ├── cart-item-editor.tsx
    └── order-confirmation.tsx
lib/
├── api/
│   ├── cart-client.ts
│   ├── catalog-client.ts
│   └── order-client.ts
└── validation/
    ├── indian-mobile.ts
    └── checkout-details.ts
tests/
├── unit/
├── integration/
└── e2e/
```

**Structure Decision**: Checkout is a Next.js presentation flow. The Order service owns order records and exposes an idempotent versioned submission contract; it obtains authoritative availability/pricing and consumes the cart through their service contracts. Each backend service retains independent MySQL ownership.

## Complexity Tracking

No constitution violations. Cross-service order creation is required to combine authoritative Catalog and Cart state while preserving single Order ownership and preventing duplicate submissions.
