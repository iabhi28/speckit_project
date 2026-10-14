# Catalog Listing API Contract

Catalog service exposes versioned listing/search endpoints and owns all product/category persistence in MySQL. The Next.js frontend accesses these contracts only; it must not connect to Catalog's tables.

## `GET /api/v1/catalog/products`

Query parameters:

- `category`, `subcategory`
- `minPrice`, `maxPrice`
- `availability`
- `sort`: `relevance`, `price-asc`, `price-desc`, `newest`
- `page` and `pageSize` or a documented cursor pair

Response:

```json
{
  "items": [{ "id": "string", "name": "string", "price": 0, "available": true, "href": "/products/example" }],
  "total": 0,
  "page": 1,
  "pageSize": 24,
  "facets": { "categories": [], "subcategories": [] }
}
```

Semantics:
- Distinct filter dimensions combine with AND.
- Category/subcategory relationship is validated; incompatible subcategory is rejected or omitted consistently.
- Invalid values are rejected with field details; unavailable/stale values can be omitted with normalized filters in the response.
- `total` describes the full matching set before pagination; sorting is stable across pages.

## `GET /api/v1/catalog/categories`

Returns active categories and subcategories used to populate filter controls.

## Errors

Malformed query values receive explicit 400 validation errors; upstream/data failures receive a retryable service error. Empty match sets return success with `items: []` and `total: 0`, not a service error.
