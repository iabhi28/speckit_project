# Implementation Plan: Product Detail Page

**Branch**: `004-product-detail-page` | **Date**: 2026-10-14 | **Spec**: [./spec.md](./spec.md)

**Input**: Feature specification from `/specs/004-product-detail-page/spec.md`

## Summary

Add a responsive product detail route that presents Catalog-owned product data, a usable image fallback, availability-aware Add to Cart action, and available related products. The page consumes versioned Catalog and Cart service contracts; related products and availability are derived from Catalog, while the Cart service owns persisted cart contents and its opaque browser identifier.

## Technical Context

**Language/Version**: TypeScript, Next.js 16, React 19, Tailwind CSS 4

**Primary Dependencies**: Existing Next.js/React stack; Catalog and Cart services use MySQL-backed records and parameterized queries

**Storage**: Catalog service owns product/category records; Cart service owns carts and cart items; browser cookie stores only the cart/session identifier

**Testing**: ESLint plus Vitest and V8 coverage; enforce >90% statement, branch, function, and line coverage

**Target Platform**: Responsive web browsers, with two-column desktop presentation and a narrow-screen stacked layout

**Project Type**: Next.js storefront feature integrated with independently deployable Catalog and Cart microservices

**Performance Goals**: Product details and related recommendations must load with explicit loading/error states; successful add-to-cart updates shared quantity and confirmation without leaving the page

**Constraints**: Unavailable products cannot be added; related results exclude the current or unavailable product; no ratings/reviews in MVP; failure must not appear as successful cart mutation

**Scale/Scope**: Product image and information, unavailable/not-found states, Add to Cart, and optional related-products section

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- Pass: Uses the approved Next.js/React/Tailwind versions.
- Pass: Catalog and Cart are independently deployable services with explicit versioned contracts and separate MySQL data ownership.
- Pass: Product availability is checked by the Catalog service; Cart mutation cannot trust client-provided product price.
- Pass: User inputs are validated at service boundaries and failures remain explicit.
- Pass: Responsive accessible actions and >90% coverage in each required test metric are planned.

## Project Structure

```text
app/
└── products/
    └── [productId]/
        └── page.tsx
components/
└── products/
    ├── product-details.tsx
    ├── product-image.tsx
    ├── add-to-cart-button.tsx
    └── related-products.tsx
lib/
└── api/
    ├── catalog-client.ts
    └── cart-client.ts
tests/
├── unit/
├── integration/
└── e2e/
```

**Structure Decision**: The dynamic Next.js product route composes Catalog and Cart API clients. Service data access remains private to each independently deployable service; shared cart feedback is surfaced through the storefront's shared cart state.

## Complexity Tracking

No constitution violations. Distinct Catalog/Cart contracts preserve data ownership while the detail page composes the two user-visible capabilities.
