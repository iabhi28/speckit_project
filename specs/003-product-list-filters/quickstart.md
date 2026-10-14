# Quickstart: Product List Filters Validation

## Prerequisites

- Node.js/npm and project dependencies.
- Catalog service with representative categories, subcategories, in-stock/out-of-stock products, prices, and dates.

## Run

```bash
npm run dev
npm test -- --run product-list
npm run lint
```

Coverage must be greater than 90% for statements, branches, functions, and lines.

## End-to-end checks

1. Open the product list at phone and desktop sizes; confirm a mobile filter drawer and desktop two-area layout.
2. Apply a category, valid subcategory, price range, and availability filter; verify every result satisfies all active filters and count matches the service response.
3. Change each sort mode; ensure active filters persist and ordering is correct.
4. Remove a chip and clear all; verify only relevant URL values change and category removal also removes subcategory.
5. Refresh, share the URL, and use browser back/forward; verify selections and results restore.
6. Test invalid/stale query values, zero results, empty subcategories, and Catalog service failure/retry.

See [Catalog listing contract](./contracts/catalog-list-api.md) and [data model](./data-model.md).
