# Research: Admin Management

## Decision

Use a Next.js App Router admin workspace with protected server-rendered/admin API routes backed by normalized MySQL tables. Admin authentication will be separate from shopper authentication, and the admin UI will read customer and order records from MySQL while using only a cart/session identifier in the browser cookie.

## Rationale

- The feature is a web application in the existing Next.js codebase, so App Router pages and route handlers fit the project rules without introducing new architecture.
- MySQL is the required durable store and must hold catalog, customer, order, and cart records. This matches the spec's `FR-018`, `FR-019`, and `FR-020` constraints.
- The spec explicitly requires an admin-only management area, separate shopper sign-in, and protected routes. A server-side auth boundary ensures unauthorized access is denied before data is returned.
- The catalog and order rules are all business-level concerns (archive on delete, dependency checks, page search/filter, status-only transitions) best handled in a domain/service layer with validation at the HTTP boundary.
- The clarifications already resolve the required behavior: no forced password reset, archived product retention for historical orders, mobile-number-only customer list, no automatic notification on delivery, search/filter/pagination, and a cookie that carries only a cart identifier.

## Alternatives considered

1. Browser-only admin UI with client-side data stores
   - Rejected because the requirement mandates server-side access protection, durable persistence in MySQL, and validation at request boundaries.

2. Treating the cart cookie as authoritative data
   - Rejected because the spec explicitly requires the cookie to contain only a cart identifier and MySQL to remain authoritative for both cart state and pricing.

3. Allowing full product deletion even when referenced by submitted orders
   - Rejected because the requirement requires archival/hiding from storefront while keeping historical record integrity.

4. Automatic customer notifications on Delivered updates
   - Rejected because the clarifications explicitly require the status change to update the order record only.

## Research notes

- Admin sign-in: Use a provisioned administrator account that is not self-registrable and is not required to change the password on first sign-in.
- Catalog dependency handling: Block deletion when categories or subcategories are still assigned to live catalog records; products may be archived instead of deleted when referenced by submitted orders.
- Customer data exposure: The registered-user list shows mobile number only; delivery details remain available only within the order record.
- Order management: filter by status and date, search by order identifier or mobile number, navigate paginated results, and accept only valid status transitions from In Progress to Delivered.
- UX requirements: The admin area must support keyboard operation, visible focus, label association, and clear empty/loading/error states.

## Open implementation decisions

The remaining design decisions are execution details rather than unresolved product requirements:

- Keep admin and shopper auth separately implemented but in a shared Next.js runtime.
- Use SQL parameterized queries and explicit validation to satisfy MySQL safety and input-boundary requirements.
- Implement a thin service layer around catalog, user, and order operations to keep validation and rule checks centralized.
