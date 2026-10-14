# Quickstart: Checkout Validation

## Prerequisites

- Node.js/npm and project dependencies.
- Cart, Catalog, and Order services running with isolated MySQL test data.
- Cart with multiple available products and a test product whose price/availability can be changed.

## Run

```bash
npm run dev
npm test -- --run checkout
npm run lint
```

Coverage must exceed 90% for statements, branches, functions, and lines.

## End-to-end checks

1. Open checkout with multiple cart products/quantities; reconcile line totals and overall total against current Catalog prices.
2. Update quantity, remove one item, and remove the last item; verify summary, total, empty state, and submit availability.
3. Submit valid details; verify persisted order, order-item snapshots, confirmation, and consumed cart.
4. Submit invalid/oversized name, invalid Indian mobile, blank/oversized address, and empty cart; verify field-level feedback and no order.
5. Change a product price or availability between display and submit; verify review-required response and no automatic order until confirmed.
6. Retry a submission after simulated network loss using the same idempotency key; verify only one order exists and cart is not lost on failure.
7. Confirm phone and desktop layouts, two-column desktop arrangement, keyboard controls, labels, focus, and accessible summary.

See [checkout API contract](./contracts/checkout-api.md) and [data model](./data-model.md).
