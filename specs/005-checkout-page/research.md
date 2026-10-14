# Research: Checkout Page

## Decision: Treat the Cart service as the authoritative cart source and the Order service as the sole owner of submitted orders.

**Rationale**: A browser cookie is only a cart/session identifier; cart item contents and prices must be read from MySQL-backed services. Orders need a durable owner and an atomic submission boundary.

**Alternatives considered**: Submitting client-rendered item/price totals as authoritative order data was rejected because browser input is untrusted.

## Decision: Order service revalidates catalog availability and current price, calculates totals from Catalog data, and snapshots product/customer details into the order.

**Rationale**: Product prices or availability can change after checkout is displayed; order snapshots preserve what was actually accepted for historical review.

**Alternatives considered**: Trusting cart price snapshots alone was rejected because they may be stale or manipulated.

## Decision: Use an idempotency key for order placement and atomically persist order plus order items before consuming/clearing cart state.

**Rationale**: Repeated submissions or a lost confirmation response must not create duplicate orders or lose the cart before durable success.

**Alternatives considered**: Disabling the button only in the browser was rejected because concurrent requests and retries can bypass it.

## Decision: Return price/availability changes as a review-required response, not an automatic order.

**Rationale**: The shopper must see revised totals and explicitly review before continuing.

**Alternatives considered**: Silently ordering at changed prices was rejected because it violates the requirement to review material changes.

## Decision: Checkout is guest-capable and collects name, India-only mobile number, and address; payment, shipping, tax, and discounts remain out of scope.

**Rationale**: Matches the approved requirements and avoids implying payment or fees that are not defined.

**Alternatives considered**: Requiring sign-in or adding payment collection was rejected because neither is requested.
