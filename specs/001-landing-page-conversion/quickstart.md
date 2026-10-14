# Quickstart: Landing Page Validation

## Prerequisites

- Node.js/npm and the project dependencies installed.
- Running versioned Catalog, Cart, Identity, Order, and Newsletter services with test data.
- Test configuration for service URLs and a test authenticated customer; no credentials are committed.

## Run

```bash
npm run dev
```

In a second terminal, run the focused tests once the feature test files are added:

```bash
npm test -- --run landing-page
npm run lint
```

Coverage validation must include statement, branch, function, and line metrics above 90%, consistent with the project constitution.

## End-to-end checks

1. Open the landing page at mobile and desktop viewport sizes; verify sticky navigation, hero CTA, visible categories, benefit messages, and support footer.
2. Type a matching and a non-matching product query; verify suggestions/no-results feedback and that stale responses do not replace newer input.
3. Add a featured product; verify Cart service success and the returned total quantity in the badge. Simulate an error and confirm the badge is not falsely advanced.
4. Submit valid and invalid newsletter email addresses; confirm explicit success, validation, and retryable service-error states.
5. Compare signed-out, signed-in-without-order, and signed-in-with-order states; only the authenticated prior-purchase module should vary.

API shape and data ownership are documented in [contracts/storefront-api.md](./contracts/storefront-api.md) and [data-model.md](./data-model.md).
