# Research: Landing Page Conversion

## Decision: Keep the landing page in the existing Next.js App Router frontend and integrate through versioned domain APIs.

**Rationale**: Next.js, React, and Tailwind are the approved existing presentation stack. The constitution requires independently deployable services with explicit contracts; the UI must not access domain-owned MySQL tables directly.

**Alternatives considered**: Implementing catalog/cart/newsletter data access inside page components was rejected because it couples presentation to persistence and bypasses service boundaries.

## Decision: Use Catalog for product/search/category data, Cart for cart mutations/count, Order for last-purchase data, Identity for the authenticated principal, and Newsletter for signups.

**Rationale**: Each source owns its domain records. The last-purchase request is made only with an authenticated customer context, and the Newsletter service owns subscription data.

**Alternatives considered**: Duplicating product data or order history in a landing-page store was rejected because it creates competing sources of truth.

## Decision: Use the opaque cart/session identifier cookie only to address Cart service state.

**Rationale**: The existing feature specs and shared storefront requirements require MySQL to remain authoritative for cart contents and prices.

**Alternatives considered**: Storing quantities/prices in cookie payloads was rejected as untrusted and inconsistent with the cart persistence requirement.

## Decision: Debounce predictive search at the UI boundary, cancel stale requests, and expose explicit no-result/error states.

**Rationale**: This satisfies the measurable suggestion latency while preventing older, slower responses from overwriting newer input.

**Alternatives considered**: Fetching on every keystroke without cancellation was rejected due to unnecessary service load and stale-result risk.

## Decision: Keep newsletter signup behind a versioned service contract and validate email both in the browser and at the service boundary.

**Rationale**: Client feedback is immediate, while server-side validation protects the persistence boundary and service errors can remain explicit.

**Alternatives considered**: Client-only validation was rejected because it is bypassable.
