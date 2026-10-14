# Product Detail API Contract

Catalog and Cart services are independently deployable, own separate MySQL records, and expose versioned APIs. The frontend composes these APIs but does not access service tables directly.

## Catalog service: `GET /api/v1/catalog/products/{productId}`

Returns `{ "id": "string", "name": "string", "description": "string", "imageUrl": "string|null", "price": 0, "category": { "id": "string", "name": "string" }, "available": true }`.

- Unknown ID returns 404.
- Inactive/unavailable item is explicitly represented and cannot be purchased.
- A service failure is distinguishable from 404.

## Catalog service: `GET /api/v1/catalog/products/{productId}/related`

Returns eligible related available items, excluding the requested ID, in the same category by default. Empty result returns `{ "items": [] }`.

## Cart service: `POST /api/v1/cart/items`

Request: `{ "productId": "string", "quantity": 1 }`.

- Requires/creates cart using the opaque cart-session cookie.
- Revalidates product availability and obtains authoritative price through Catalog's public contract.
- Adds quantity to existing item where applicable.
- Success returns updated cart count/summary. Failure leaves client state unchanged and is explicit.

## Error semantics

Use 404 for unknown product, validation errors for malformed IDs/quantity, conflict/unavailable response for unavailable items, and retryable service errors for dependency failures. The UI must preserve a useful recovery route in every failure state.
