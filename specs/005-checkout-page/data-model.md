# Data Model: Checkout Page

Cart, Catalog, and Order services own their MySQL data independently. Checkout obtains all information through versioned APIs and never writes directly to their private tables.

## Cart and CartItem (Cart service)

- Cart: `id`, opaque `session_id`, optional customer identity, timestamps.
- CartItem: `id`, `cart_id`, `product_id`, positive `quantity`, timestamps.
- The cookie contains only the opaque session identifier. Server services resolve content from MySQL.

## Product (Catalog service)

- `id`, `name`, current `price`, `is_available`, active state.
- Catalog is authoritative for price and purchase availability at order submission.

## Order (Order service)

- `id`, `customer_name`, normalized `mobile_number`, `delivery_address`, `status`, `total_amount`, `created_at`.
- Initial status is `In Progress` per the store order flow.
- Name/mobile/address are validated at Order service boundary; mobile follows India-only normalization.

## OrderItem (Order service)

- `id`, `order_id`, `product_id`, `product_name_snapshot`, `quantity`, `unit_price_snapshot`, `line_total`.
- Snapshots are immutable after order creation and retain historical details if a Catalog product is later archived.
- `total_amount` equals the sum of persisted line totals. No tax, shipping, or discount is included in this feature.

## OrderSubmission (request state)

- `idempotency_key`, request fingerprint, resulting `order_id`, current coordination state, and completion outcome.
- Reuse with the same payload returns the existing result; reuse with a different payload is rejected.
- Coordination state records durable order creation and idempotent cart consumption separately so retrying after a partial cross-service failure can resume without creating another order.

## State transitions

- Checkout view: loading -> populated/empty/error.
- Submission: ready -> pending -> confirmed only after durable Order success; pending -> review-required when authoritative price/availability differs; pending -> retryable error while cart/details are preserved.
- Successful order creation consumes the submitted cart exactly once.
