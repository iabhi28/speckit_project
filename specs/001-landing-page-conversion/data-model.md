# Data Model: Landing Page Conversion

The page reads domain records through owning service contracts. The frontend does not own or persist duplicate copies of these records.

## Product and Category (Catalog service)

- Product: `id`, `name`, `description`, `image_url`, `price`, `category_id`, `is_available`, `created_at`.
- Category: `id`, `name`, `slug`, optional image/visual asset, active state.
- Collection: derived Catalog grouping for Categories, New Arrivals, Deals, or featured products.
- Validation: visible purchasable products require a valid identifier, display name, nonnegative price, and availability; unavailable products are not offered for cart addition.

## Cart and CartItem (Cart service)

- Cart: `id`, opaque `session_id`, optional `customer_id`, timestamps.
- CartItem: `id`, `cart_id`, `product_id`, positive `quantity`, timestamps.
- The browser cookie contains only the opaque session identifier. Quantity shown by the badge is derived from Cart service response; price is not trusted from the cookie.

## CustomerAccount and OrderItem (Identity/Order services)

- Authenticated customer identity is supplied by Identity; the landing page does not register or authenticate shoppers.
- Purchase summary: product reference/name/image, purchase date, order status, and order identifier from the customer's most recent eligible order.
- Validation: request the summary only for the authenticated customer; show no section when signed out or when no prior purchase exists. An unavailable product is explicitly identified and not offered as purchasable.

## NewsletterSignup (Newsletter service)

- `id`, normalized `email`, `created_at`, signup status.
- Email must be syntactically valid and unique according to Newsletter service policy; failed persistence must return an error rather than a success result.

## Relationships

- Category has many products.
- Cart has many cart items, each referencing a Catalog product.
- Customer has many orders; an order contains one or more order items.
- Newsletter signup is independent of cart and customer account state.

## State transitions

- Cart mutation: unchanged -> confirmed updated cart only after Cart service success.
- Newsletter signup: pending -> subscribed on service success; pending -> retryable error on failure.
- Last-purchase module: hidden for signed-out/no-history; visible for signed-in customer with history.
