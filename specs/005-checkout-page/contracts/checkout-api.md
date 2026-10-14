# Checkout API Contract

Cart, Catalog, and Order are independently deployable services with separate MySQL data ownership. The Order service coordinates through their versioned contracts and owns order persistence.

## Cart service: `GET /api/v1/cart`

Resolves cart from the opaque cart-session cookie and returns item identifiers, current catalog-resolved display values, quantities, and total quantity. Cookie contents are never treated as cart data.

## Cart service: `PATCH /api/v1/cart/items/{productId}` and `DELETE /api/v1/cart/items/{productId}`

PATCH request: `{ "quantity": 1 }`. Quantity must be a positive bounded integer. Delete removes the item. Both return the updated server cart summary.

## Order service: `POST /api/v1/orders`

Headers: `Idempotency-Key: {unique-request-id}`.

Request:

```json
{
  "customer": {
    "name": "string",
    "mobile": "string",
    "deliveryAddress": "string"
  }
}
```

The service resolves the cart session, obtains current price/availability through Cart/Catalog contracts, validates customer fields, recomputes totals, and persists order/item snapshots. The request key and order record are persisted before cart consumption. Cart consumption uses an idempotent operation identifier derived from the order submission so a retry can resume coordination after a partial cross-service failure without creating a second order.

Success returns `{ "orderId": "string", "status": "In Progress", "total": 0 }`.

If price or availability changed, return a review-required response with changed lines and recalculated total; no order is created until the shopper explicitly confirms the revised cart. Repeated requests with the same idempotency key must not create duplicate orders.

## Error semantics

- Validation errors identify affected fields.
- Empty cart is rejected.
- Unavailable products identify affected items and retain the cart.
- Service/transaction failures return explicit retry guidance; no success response is returned.
- A retry with the same idempotency key returns the original successful order if one was already committed.
- If order persistence succeeded but cart consumption is still being retried, the service returns the same order result and resumes idempotent cart consumption; it must not create a duplicate order.

Payment processing is not part of this contract.
