# Data Model: Product Detail Page

Catalog and Cart services own their persistent records. The detail page composes responses and stores no duplicate product or cart records.

## Product (Catalog service)

- `id`, `name`, `description`, `image_url`, `price`, `category_id`, `is_available`, active state.
- Category relationship: product belongs to a category; related items default to other active, available products in that category.
- Validation: price is nonnegative; archived/unavailable items cannot be added to a cart.

## RelatedProduct (derived Catalog response)

- Product fields needed for card rendering: `id`, `name`, `image_url`, `price`, `href`.
- Excludes the current product and any unavailable/archived product.
- Omitted entirely when there are no matching records.

## Cart and CartItem (Cart service)

- Cart: `id`, opaque `session_id`, optional customer identity, timestamps.
- CartItem: `id`, `cart_id`, `product_id`, positive `quantity`, timestamps.
- Cookie contains only opaque cart/session ID. Cart service verifies Product availability and authoritative price on mutation.

## Page state

- Product route identifier resolves to detail, not-found, or retryable service-error state.
- Add action transitions from idle -> pending -> success with server-returned cart count, or pending -> explicit error with prior cart count retained.
- Unavailable products display status and have no enabled add action.
