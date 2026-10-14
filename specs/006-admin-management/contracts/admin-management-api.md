# Admin Management API Contract

This document captures the required external interfaces for the admin management feature in the Next.js application. All routes are authenticated for administrator access and validate request payloads before mutating data.

## Authentication

### `POST /api/admin/auth/login`

Request body:

```json
{
  "username": "string",
  "password": "string"
}
```

Behavior:
- Verifies the provisioned administrator credential.
- Rejects invalid credentials without revealing which field was wrong.
- Creates a secure admin session on success.
- Denies access to protected pages when the admin session is missing or expired.

Success response:

```json
{
  "success": true,
  "redirect": "/admin"
}
```

Error response:

```json
{
  "success": false,
  "message": "Invalid administrator credentials."
}
```

## Catalog routes

### `GET /api/admin/categories`

Returns paginated or filtered category data for admin management.

### `POST /api/admin/categories`

Request:

```json
{
  "name": "Electronics",
  "slug": "electronics"
}
```

Validation:
- `name` and `slug` required.
- Duplicate or invalid values rejected.

### `PATCH /api/admin/categories/:id`

Request may contain editable fields such as name or slug.

### `DELETE /api/admin/categories/:id`

Behavior:
- Rejects deletion when dependent catalog records remain assigned to the category.
- Returns a clear dependency message.

### `GET /api/admin/subcategories`

Returns subcategory entries for the selected category context.

### `POST /api/admin/subcategories`

Request:

```json
{
  "category_id": 12,
  "name": "Audio",
  "slug": "audio"
}
```

### `PATCH /api/admin/subcategories/:id`

Updates valid fields only.

### `DELETE /api/admin/subcategories/:id`

Rejects deletion when dependent products remain assigned.

### `GET /api/admin/products`

Returns product records with search and filter metadata.

### `POST /api/admin/products`

Request:

```json
{
  "category_id": 12,
  "subcategory_id": 31,
  "name": "Wireless Headphones",
  "description": "Noise-cancelling headphones",
  "price": 149.99,
  "sku": "WH-100"
}
```

Validation:
- Category and product fields required and valid.
- Reject invalid or missing required values with field-level guidance.

### `PATCH /api/admin/products/:id`

Allows update of editable catalog fields.

### `DELETE /api/admin/products/:id`

Behavior:
- If the product is referenced in a submitted order, archive or hide it from storefront while keeping it in historical order details.
- Otherwise, remove it from the active catalog.

## Registered user and order routes

### `GET /api/admin/customers`

Returns a paginated customer list searchable by mobile number.

Response excerpt:

```json
{
  "items": [
    { "id": 1, "mobile_number": "+923331234567" }
  ],
  "page": 1,
  "page_size": 25,
  "total": 1
}
```

### `GET /api/admin/orders`

Returns paginated order records searchable by order identifier or customer mobile number and filterable by status or date.

Response excerpt:

```json
{
  "items": [
    {
      "id": 4001,
      "customer_name": "Aisha Khan",
      "mobile_number": "+923331234567",
      "delivery_address": "12 Lane 7, Karachi",
      "total_amount": 2456.50,
      "status": "In Progress"
    }
  ],
  "page": 1,
  "page_size": 25,
  "total": 12
}
```

### `PATCH /api/admin/orders/:id/status`

Request:

```json
{
  "status": "Delivered"
}
```

Validation:
- Allows only `In Progress` -> `Delivered`.
- Rejects unsupported transitions.
- Leaves the prior status visible if the operation fails.
- Updates the order record without sending an automatic notification.

## Response and error requirements

All admin routes MUST:
- Require administrator authentication.
- Return explicit service errors when data loading or mutation fails.
- Show loading, empty, validation-error, and service-error states in the UI.
- Keep user-facing feedback readable and non-revealing for failed credential checks.
- Use database-backed records as the single source of truth for catalog, customer, and order data.
