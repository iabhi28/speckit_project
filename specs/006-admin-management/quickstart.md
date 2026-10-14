# Quickstart: Admin Management Validation

This guide describes the validation scenarios that prove the admin-management feature works end-to-end without diving into implementation code.

## Prerequisites

- Node.js and npm available for the project.
- MySQL database configured for the application.
- Environment variables for database access and admin session configuration.
- A provisioned administrator account with credentials stored outside the repository.

## Setup

1. Install project dependencies:
   ```bash
   npm install
   ```
2. Configure MySQL connection settings and session secret in the runtime environment.
3. Start the app:
   ```bash
   npm run dev
   ```

## Validation scenarios

### 1. Admin access control

1. Open the admin route while unauthenticated.
2. Sign in as a customer account and attempt access.
3. Sign in as the provisioned administrator.
4. Confirm that only the administrator is granted access and that logout revokes it.

Expected outcome: unauthorized users are denied and the admin session is invalidated on sign-out.

### 2. Catalog CRUD

1. Create a category.
2. Add a subcategory and a product under it.
3. Edit the product details.
4. Attempt to delete a category or subcategory that still has dependent records.
5. Remove a product that is referenced by a submitted order and confirm it is hidden from active storefront catalog while retaining historical details.

Expected outcome: valid operations succeed; invalid deletions fail with explicit dependency guidance.

### 3. Customer and order review

1. Open the registered-user list and search by mobile number.
2. Open the order list and search by order identifier or mobile number.
3. Filter the orders by status and date, then navigate across pages.
4. Confirm the customer list shows only mobile numbers and the order details include delivery address and item totals.

Expected outcome: search, filter, and pagination produce only matching records.

### 4. Order status update

1. Select an `In Progress` order.
2. Submit a status change to `Delivered`.
3. Attempt an unsupported transition.
4. Simulate a service failure during the update and confirm the prior status remains visible.

Expected outcome: valid transitions succeed, unsupported transitions are rejected, and failures show explicit retry messaging.

## Reference artifacts

- Data model: [data-model.md](./data-model.md)
- API contract: [contracts/admin-management-api.md](./contracts/admin-management-api.md)

## Notes

- The browser cookie must contain only a cart/session identifier.
- Cart pricing and order totals must be derived from authoritative catalog data, not from cookie values.
- No automatic customer message is sent when an order is marked delivered.
