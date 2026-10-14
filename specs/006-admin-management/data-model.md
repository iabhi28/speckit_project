# Data Model: Admin Management

This feature relies on durable MySQL records for admin authentication, catalog content, customer accounts, orders, and cart state. The model below captures the required entities and core validation rules.

## Entity: AdministratorAccount

| Field | Type | Required | Notes |
|---|---|---:|---|
| id | bigint / UUID | Yes | Primary key for the provisioned admin record. |
| username | varchar(64) | Yes | Unique administrator login identifier; not publicly exposed. |
| password_hash | varchar(255) | Yes | One-way hash stored in the database; no plaintext credential in source control. |
| created_at | datetime | Yes | Provisioning timestamp. |
| updated_at | datetime | Yes | Updated on password or account metadata changes. |

Validation rules:
- Username must be present and unique.
- Password must be stored as a hash; the feature does not allow self-registration or forced first-login password reset.
- Access to protected admin routes is granted only after credential verification.

## Entity: Category

| Field | Type | Required | Notes |
|---|---|---:|---|
| id | bigint | Yes | Primary key. |
| name | varchar(255) | Yes | User-facing category name. |
| slug | varchar(255) | Yes | Stable catalog identifier; unique when active. |
| parent_id | bigint nullable | No | Optional parent category if supported. |
| is_active | boolean | Yes | False indicates archived or hidden state. |
| created_at | datetime | Yes | Record creation time. |
| updated_at | datetime | Yes | Last modification time. |

Validation rules:
- Name and slug are required.
- Category deletion is blocked if dependent catalog records remain assigned to it.
- Deletion should rely on dependency checks rather than a blind delete.

## Entity: Subcategory

| Field | Type | Required | Notes |
|---|---|---:|---|
| id | bigint | Yes | Primary key. |
| category_id | bigint | Yes | Parent category. |
| name | varchar(255) | Yes | User-facing label. |
| slug | varchar(255) | Yes | Unique within the parent category. |
| is_active | boolean | Yes | Active or archived status. |
| created_at | datetime | Yes | Record creation time. |
| updated_at | datetime | Yes | Last modification time. |

Validation rules:
- Subcategory belongs to an existing category.
- Subcategory delete is blocked while dependent products remain assigned.

## Entity: Product

| Field | Type | Required | Notes |
|---|---|---:|---|
| id | bigint | Yes | Primary key. |
| category_id | bigint | Yes | Active category link. |
| subcategory_id | bigint nullable | Yes/conditional | Required when the catalog uses subcategory grouping. |
| name | varchar(255) | Yes | Product name. |
| description | text | No | Extended product details. |
| price | decimal(10,2) | Yes | Store price; authoritative source for order total calculations. |
| sku | varchar(100) | No | Optional item identifier. |
| is_active | boolean | Yes | Active storefront visibility flag. |
| archived_at | datetime nullable | No | Set when a product is hidden from active storefront because of order history constraints. |
| created_at | datetime | Yes | Record creation time. |
| updated_at | datetime | Yes | Last modification time. |

Validation rules:
- Required catalog fields must be present before create/update.
- Product removal: if referenced by a submitted order, set `is_active = false` and keep record for historical order details.
- Price used for order totals must come from authoritative catalog data, not browser-provided cookie values.

## Entity: CustomerAccount

| Field | Type | Required | Notes |
|---|---|---:|---|
| id | bigint | Yes | Primary key. |
| name | varchar(255) | Yes | Customer name. |
| mobile_number | varchar(32) | Yes | Primary lookup key for the customer admin list. |
| created_at | datetime | Yes | Record creation time. |
| updated_at | datetime | Yes | Last change. |

Validation rules:
- Mobile number is the search key in the admin list.
- Delivery addresses are not shown in the registered-user list; they remain in order data only.

## Entity: Order

| Field | Type | Required | Notes |
|---|---|---:|---|
| id | bigint | Yes | Order identifier. |
| customer_id | bigint | Yes | Linked customer account. |
| customer_name | varchar(255) | Yes | Snapshot of the customer name at order time. |
| mobile_number | varchar(32) | Yes | Snapshot used for status and fulfillment lookup. |
| delivery_address | text | Yes | Fulfillment address retained for order records. |
| status | enum('In Progress','Delivered') | Yes | Supported statuses for this feature. |
| total_amount | decimal(10,2) | Yes | Computed from authoritative catalog pricing. |
| created_at | datetime | Yes | Order creation time. |
| updated_at | datetime | Yes | Last status or metadata change. |

Validation rules:
- Orders start in `In Progress` and may transition to `Delivered` only.
- Unsupported transitions are rejected and the prior status remains.
- No automatic notification is sent on status update.
- Delivery details remain part of the order record rather than the customer list.

## Entity: OrderItem

| Field | Type | Required | Notes |
|---|---|---:|---|
| id | bigint | Yes | Primary key. |
| order_id | bigint | Yes | Parent order. |
| product_id | bigint | Yes | Product reference for historical order details. |
| product_name | varchar(255) | Yes | Snapshot of product name at order time. |
| quantity | int | Yes | Ordered quantity. |
| unit_price | decimal(10,2) | Yes | Price snapshot used for the order total. |
| total_price | decimal(10,2) | Yes | `quantity * unit_price`. |

Validation rules:
- Order item snapshots remain immutable after order submission.
- Historical order details must still reference archived products without exposing them in the active storefront.

## Entity: Cart

| Field | Type | Required | Notes |
|---|---|---:|---|
| id | bigint | Yes | Persistent cart identifier. |
| session_id | varchar(255) | Yes | Browser cookie identifier or equivalent session key. |
| customer_id | bigint nullable | No | Optional customer association if the user is logged in. |
| created_at | datetime | Yes | Record creation time. |
| updated_at | datetime | Yes | Last modification time. |

Validation rules:
- The browser cookie contains only the cart/session identifier.
- Cart contents are stored in the database and are not treated as authoritative cookie data.
- Cart data is separate from the order record and must not be trusted for pricing.

## Entity: CartItem

| Field | Type | Required | Notes |
|---|---|---:|---|
| id | bigint | Yes | Primary key. |
| cart_id | bigint | Yes | Parent cart. |
| product_id | bigint | Yes | Active product reference. |
| quantity | int | Yes | Quantity in cart. |
| price_snapshot | decimal(10,2) | Yes | Current catalog-derived price snapshot at save time. |
| created_at | datetime | Yes | Record creation time. |
| updated_at | datetime | Yes | Last modification time. |

Validation rules:
- Cart prices are recalculated from authoritative catalog data when an order is created.
- Browser-provided cart values are never trusted as the source of truth for totals.

## Relationships

- One administrator may provision one or more admin session records, but this feature supports a single administrator identity.
- One category has many subcategories.
- One category has many products.
- One subcategory has many products.
- One customer has many orders and may have a cart.
- One order has many order items.
- One cart has many cart items.

## State transitions

- Product: active -> archived/hidden (when it is removed from active catalog but referenced by a submitted order)
- Order: In Progress -> Delivered (allowed); any other transition is rejected.
- Category/Subcategory: active -> blocked delete (when dependencies exist); active -> archived/hidden when supported in the domain model.
