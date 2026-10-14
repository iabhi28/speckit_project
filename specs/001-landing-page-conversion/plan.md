# Implementation Plan: Landing Page Conversion

**Branch**: `001-landing-page-conversion` | **Date**: 2026-10-14 | **Spec**: [./spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-landing-page-conversion/spec.md`

## Summary

Build a mobile-first Next.js storefront landing page with sticky navigation, predictive catalog search, featured products, direct cart actions, category links, store assurances, support/newsletter footer, and a purchase-history panel shown only when the authenticated customer has a prior order. The Next.js application is the presentation tier; versioned Catalog, Cart, Order, Identity, and Newsletter service contracts keep durable data ownership and deployment boundaries explicit.

## Technical Context

**Language/Version**: TypeScript, Next.js 16, React 19, Tailwind CSS 4

**Primary Dependencies**: Existing `next`, `react`, `react-dom`; `mysql2` belongs to service implementations and is already listed in the project dependencies

**Storage**: MySQL in independently owned service stores; browser cookie contains only the Cart service session identifier

**Testing**: ESLint plus Vitest and V8 coverage in the implementation test setup; enforce the constitution's >90% statement, branch, function, and line coverage

**Target Platform**: Responsive web browsers, mobile-first with tablet and desktop support

**Project Type**: Next.js storefront web application integrated with independently deployable domain services

**Performance Goals**: Meet spec's 2.5-second mobile-content visibility at the 75th percentile; search suggestions within one second in at least 95% of representative paused-input runs; reflect cart additions within one second

**Constraints**: Authentication is supplied by Identity service; last-purchase data is private to the authenticated customer; ratings/reviews are excluded; Newsletter and cart failures must be explicit; catalog and order data remain authoritative in their owning services

**Scale/Scope**: One storefront landing experience, featured/category/search entry points, a cart badge/action, newsletter signup, and a conditional last-purchase module

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- Pass: Uses the declared Next.js, React, and Tailwind versions without a framework upgrade.
- Pass: Separates presentation from independently deployable Identity, Catalog, Cart, Order, and Newsletter service contracts; each service owns its persistent data and exposes a versioned interface.
- Pass: Uses MySQL-backed domain records; browser cookie is an opaque cart/session identifier only.
- Pass: Inputs for search and newsletter signup are validated at their service boundaries, and service failures are not rendered as success.
- Pass: Responsive accessible controls and loading, empty, and error states are part of the page design.
- Pass: The test approach requires >90% coverage for each constitution metric.

## Project Structure

```text
app/
└── page.tsx
components/
└── storefront/
    ├── header.tsx
    ├── search-suggestions.tsx
    ├── hero.tsx
    ├── featured-products.tsx
    ├── category-links.tsx
    ├── last-purchase.tsx
    └── footer.tsx
lib/
├── api/
│   ├── catalog-client.ts
│   ├── cart-client.ts
│   ├── order-client.ts
│   └── newsletter-client.ts
└── validation/
    └── newsletter.ts
tests/
├── unit/
├── integration/
└── e2e/
```

**Structure Decision**: Keep the existing Next.js App Router frontend. UI/API client modules call versioned domain-service contracts; they do not connect directly to another service's MySQL store. Each backend service is independently deployable and owns its tables.

## Complexity Tracking

No constitution violations. The dedicated Newsletter service is justified by persistent subscription ownership and the independently versioned integration contract.
