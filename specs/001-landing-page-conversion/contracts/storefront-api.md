# Storefront API Contract

All routes are versioned under `/api/v1`. The Next.js frontend may call a same-origin BFF only as a transport adapter; domain state and writes remain owned by the named independently deployable service. Each service validates inputs and returns explicit errors.

## Catalog service

- `GET /api/v1/catalog/search?q={query}&limit={n}` returns product suggestions; validates query size and limits.
- `GET /api/v1/catalog/featured` returns active featured products.
- `GET /api/v1/catalog/categories` returns active visual category links.
- Suggestion response: `{ "items": [{ "id": "string", "name": "string", "imageUrl": "string|null", "price": "number", "href": "string" }] }`.

## Cart service

- `GET /api/v1/cart` resolves the opaque cart session identifier and returns current items and total quantity.
- `POST /api/v1/cart/items` accepts `{ "productId": "string", "quantity": 1 }`; server confirms availability and returns updated cart summary.
- Invalid/unavailable products or persistence failure return a non-success response; the client must not increment its badge optimistically as final state.

## Order service

- `GET /api/v1/orders/me/last-purchase` requires an authenticated customer and returns `{ "productId": "string", "name": "string", "imageUrl": "string|null", "purchasedAt": "ISO-8601", "status": "string", "available": true }`, or an empty result when no prior purchase exists.
- The route returns no other customer's purchase data. Unauthenticated callers receive unauthorized.

## Newsletter service

- `POST /api/v1/newsletter/subscriptions` accepts `{ "email": "string" }`.
- Success confirms subscription; invalid email returns field validation details; service failure is retryable and does not claim success.

## Error semantics

Services use appropriate 4xx responses for invalid/unauthorized requests and 5xx responses for service failures with a safe user-facing message and correlation identifier. The UI separately represents loading, empty, success, and error states.
