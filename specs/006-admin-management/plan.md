# Implementation Plan: Admin Management

**Branch**: `006-admin-management` | **Date**: 2026-10-14 | **Spec**: [./spec.md](./spec.md)

**Input**: Feature specification from `/specs/006-admin-management/spec.md`

## Summary

This feature adds a protected admin-only management surface for catalog CRUD, registered-user review, order review, and order-status updates. The design uses the existing Next.js App Router, keeps durable state in MySQL, enforces admin-only access before serving protected pages or mutating data, and uses a cookie that contains only a cart/session identifier while treating MySQL as the authoritative cart and pricing source.

## Technical Context

**Language/Version**: TypeScript with Next.js 16, React 19, and Tailwind CSS 4

**Primary Dependencies**: `next`, `react`, `react-dom`, `mysql2`, `tailwindcss`, `eslint`, `eslint-config-next`

**Storage**: MySQL for durable admin, catalog, customer, order, and cart records; browser cookie limited to cart/session identifier only

**Testing**: ESLint + targeted unit/integration tests; exact runner selection to be finalized during implementation (NEEDS CLARIFICATION)

**Target Platform**: Web application optimized for phone, tablet, and desktop browser layouts

**Project Type**: web-application

**Performance Goals**: Support admin list search/filter/pagination and status updates with standard storefront response times for a small-to-medium catalogue and order volume (NEEDS CLARIFICATION)

**Constraints**: Admin-only access required; catalog delete rules enforced; no customer notification for Delivered updates; product references retained for order history; access must terminate on logout or session expiry

**Scale/Scope**: Single storefront with admin management surfaces for catalog, customer, and order workflows (NEEDS CLARIFICATION)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- Pass: The project uses the approved Next.js, React, and Tailwind versions in `package.json`.
- Pass: Durable application data remains in MySQL, with browser-only session state restricted to the cart identifier.
- Pass: The design separates shopper and admin authentication and keeps protected admin routes behind an explicit authorization boundary.
- Pass: The implementation uses explicit validation and dependency checks at API boundaries, matching the requirement to reject invalid inputs and explicit dependency blockers.
- Pass: The design keeps domain responsibilities narrow and service-oriented for auth, catalog, customer review, and order workflow operations.
- Pass: The admin UI will include required empty, loading, validation, and service-error states as dictated by the feature requirements.

## Project Structure

### Documentation (this feature)

```text
specs/006-admin-management/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)

```text
app/
├── admin/
│   ├── page.tsx
│   ├── categories/page.tsx
│   ├── products/page.tsx
│   ├── users/page.tsx
│   └── orders/page.tsx
├── api/
│   └── admin/
│       ├── auth/login/route.ts
│       ├── categories/route.ts
│       ├── categories/[id]/route.ts
│       ├── products/route.ts
│       ├── products/[id]/route.ts
│       ├── users/route.ts
│       ├── orders/route.ts
│       └── orders/[id]/status/route.ts
├── globals.css
├── layout.tsx
└── page.tsx

components/
├── admin/
│   ├── admin-shell.tsx
│   ├── catalog-table.tsx
│   ├── customer-list.tsx
│   ├── order-list.tsx
│   └── status-form.tsx
└── ui/

lib/
├── auth/
│   └── admin.ts
├── db/
│   └── mysql.ts
├── domain/
│   ├── catalog.ts
│   ├── customer.ts
│   ├── order.ts
│   └── cart.ts
└── validation/
    └── admin.ts

tests/
├── unit/
├── integration/
└── e2e/
```

**Structure Decision**: The feature will live within the existing Next.js App Router project, with admin pages under `app/admin` and admin API handlers under `app/api/admin`. Domain and validation logic will be isolated in `lib/` to keep auth, catalog, customer, and order rules explicitly separated from the UI layer while preserving the existing project structure.

## Complexity Tracking

No constitution violations require justification for this feature; all design constraints are directly aligned with the project constitution and the explicit feature requirements.
