# Implementation Plan: Product List Filters

**Branch**: `003-product-list-filters` | **Date**: 2026-10-14 | **Spec**: [./spec.md](./spec.md)

**Input**: Feature specification from `/specs/003-product-list-filters/spec.md`

## Summary

Create a responsive product listing that combines category/subcategory, price, and availability filters with supported sorting. Filter and sort state is encoded in URL query parameters so refreshes, share links, and browser back/forward restore the view. Product search/filter results are served by the versioned Catalog service contract; the web client never queries the Catalog service's MySQL store directly.

## Technical Context

**Language/Version**: TypeScript, Next.js 16, React 19, Tailwind CSS 4

**Primary Dependencies**: Existing Next.js/React stack; Catalog service uses MySQL and parameterized queries

**Storage**: Catalog service-owned MySQL tables for categories, subcategories, and products; filter/sort selection remains in URL state

**Testing**: ESLint plus Vitest and V8 coverage; enforce >90% statement, branch, function, and line coverage

**Target Platform**: Responsive web browsers with phone-first filter access and larger-screen two-area layout

**Project Type**: Next.js storefront feature integrated with an independently deployable Catalog microservice

**Performance Goals**: Results and counts must correspond to all active filters; paging/incremental loading must preserve consistent server-side filtering and sorting

**Constraints**: AND semantics across filter types; at most one category and one subcategory; dependent subcategory clears when its category is removed; stale URL values are ignored while valid selections remain

**Scale/Scope**: Single storefront catalog with category, subcategory, price range, availability, relevance/price/newest sorting, removable chips, and URL-restorable state

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- Pass: Uses the existing approved Next.js, React, and Tailwind stack.
- Pass: Catalog data is MySQL-backed and owned by the independently deployable Catalog service.
- Pass: Catalog requests use a versioned contract, validated query parameters, and parameterized service-side queries.
- Pass: URL state is user navigation state, not a substitute for durable product records.
- Pass: Mobile filter access, keyboard operation, visible focus, loading/error/empty states, and >90% coverage per required metric are included.

## Project Structure

```text
app/
└── products/
    └── page.tsx
components/
└── products/
    ├── filter-panel.tsx
    ├── active-filter-chips.tsx
    ├── sort-control.tsx
    ├── product-grid.tsx
    └── filter-drawer.tsx
lib/
├── api/
│   └── catalog-client.ts
└── products/
    ├── filter-state.ts
    └── query-parameters.ts
tests/
├── unit/
├── integration/
└── e2e/
```

**Structure Decision**: Product results and filter metadata come from the Catalog service via a versioned API client. The URL is the canonical source for selected filters and sorting in the browser; Catalog remains authoritative for product availability, pricing, and category membership.

## Complexity Tracking

No constitution violations. Query-state parsing and serialization are required by the explicit share/refresh/back-forward behavior.
