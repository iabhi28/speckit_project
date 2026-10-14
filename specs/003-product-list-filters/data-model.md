# Data Model: Product List Filters

The Catalog service owns persistent product/category data. The listing feature's selected filters are navigational URL state, not a durable database entity.

## Catalog entities

- Category: `id`, `name`, `slug`, `is_active`.
- Subcategory: `id`, `category_id`, `name`, `slug`, `is_active`.
- Product: `id`, `category_id`, optional `subcategory_id`, `name`, `price`, `is_available`, `created_at`, active state.
- Relationships: one Category has many Subcategories and Products; one Subcategory belongs to one Category and has many Products.
- Validation: prices are nonnegative; subcategory must belong to the selected category; inactive categories/products are excluded from shopper results; database queries are parameterized.

## FilterState (URL state)

- `category`: optional valid category ID/slug.
- `subcategory`: optional valid subcategory ID/slug whose parent matches selected category.
- `minPrice`, `maxPrice`: optional finite nonnegative amounts; minimum cannot exceed maximum.
- `availability`: supported availability choice only.
- `sort`: `relevance`, `price-asc`, `price-desc`, or `newest`; invalid/missing values resolve to the default sort.
- `page` or cursor: optional pagination state as supported by the Catalog response.

## ProductResultSet (derived response)

- Matching product page, total count, normalized active filter state, and sort order.
- All filters combine with AND semantics. Page contents, count, and ordering are computed over the complete match set.

## State rules

- Removing category also clears dependent subcategory.
- Removing subcategory leaves category unchanged.
- Clear all removes all filter selections and restores the default sort.
- An invalid/stale URL value is ignored while other valid selections remain.
