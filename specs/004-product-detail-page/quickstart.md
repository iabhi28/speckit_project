# Quickstart: Product Detail Validation

## Prerequisites

- Node.js/npm and project dependencies.
- Catalog and Cart services with available, unavailable, image-missing, related, and no-related test products.

## Run

```bash
npm run dev
npm test -- --run product-detail
npm run lint
```

Coverage must exceed 90% for statements, branches, functions, and lines.

## End-to-end checks

1. Open an available product at desktop and mobile sizes; check image, title, description, price, category, availability, and responsive layout.
2. Add the product once, then again; verify confirmation, server-returned cart quantity, and no navigation.
3. Simulate Cart failure and a product becoming unavailable after page load; confirm no false cart success.
4. Open unknown product ID and simulate Catalog failure; verify distinct not-found and retry/error states.
5. Check recommendations exclude current/unavailable products, open their detail pages, and disappear when none are available.
6. Disable the main image and confirm product details/action remain accessible with fallback alt text.

Contract and entity definitions: [product detail contract](./contracts/product-detail-api.md), [data model](./data-model.md).
