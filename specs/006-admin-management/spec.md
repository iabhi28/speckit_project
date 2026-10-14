# Feature Specification: Admin Management

**Feature Branch**: `Not created (no before_specify hook is configured)`

**Created**: 2026-10-14

**Status**: Draft

**Input**: User description: "Admin-only page for category, subcategory, and product CRUD; registered-user and order lists; order status updates from In Progress to Delivered; no admin self-registration."

## Clarifications

### Session 2026-10-14

- Q: Should the provisioned administrator be required to replace the initial password at first sign-in? → A: No; keep the provisioned password until changed manually.
- Q: What should happen when an administrator tries to delete a product that appears in a submitted order? → A: Archive or hide it from the storefront but retain it for order history.
- Q: Which customer details should appear in the registered-user list? → A: Mobile number only; show delivery details only within each order.
- Q: When an administrator marks an order Delivered, should the system notify the customer automatically? → A: No; update the status in the order record only.
- Q: How should administrators locate customers and orders in the management lists? → A: Search and filter the lists, with results shown in pages.
- Q: Where should admin-managed catalog data, customer accounts, orders, and cart state be stored? → A: Store durable records in their respective database tables; store cart state in the database and identify the cart from the browser cookie.
- Q: What should the browser cookie contain for a cart that is also stored in the database? → A: Only a cart/session identifier; the database is authoritative.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Sign in as an administrator (Priority: P1)

As an administrator, I want to sign in to a protected management area so I can manage the store without exposing administrative capabilities to shoppers.

**Why this priority**: Authentication and access control protect every administrative operation and the customer and order information shown in the area.

**Independent Test**: Attempt to open the management area signed out, as a customer, and as an authenticated administrator; verify only the administrator is granted access.

**Acceptance Scenarios**:

1. **Given** a visitor is not authenticated as an administrator, **When** they open an administrative page, **Then** access is denied and they are directed to administrator sign-in.
2. **Given** a customer is signed in through customer authentication, **When** they attempt to access an administrative page, **Then** they remain unauthorized.
3. **Given** a valid provisioned administrator username and password are submitted, **When** they are verified, **Then** the administrator can access the management area.
4. **Given** an invalid administrator credential is submitted, **When** it is verified, **Then** access is denied with a clear message that does not reveal which credential was incorrect.
5. **Given** an administrator has signed in, **When** the administrator signs out, **Then** protected pages are no longer accessible using that session.
6. **Given** an administrator account must be provisioned, **When** its credentials are configured, **Then** the credentials are not exposed in application source, public pages, or committed project files, administrator self-registration is unavailable, and the administrator is not forced to change the password at first sign-in.

### User Story 2 - Manage catalog categories and products (Priority: P1)

As an administrator, I want to create, view, update, and remove categories, subcategories, and products so I can keep the storefront catalog accurate.

**Why this priority**: Catalog maintenance keeps products discoverable and purchasable with correct information.

**Independent Test**: Create a category, add a subcategory and product, edit their details, and remove an item that is not referenced by other catalog records.

**Acceptance Scenarios**:

1. **Given** the administrator is signed in, **When** they open catalog management, **Then** they can view categories, subcategories, and products.
2. **Given** valid required details are provided, **When** the administrator creates a category, subcategory, or product, **Then** it is saved and appears in the catalog list.
3. **Given** a catalog record exists, **When** the administrator edits valid details, **Then** the updated details are shown in management and in the shopper-facing catalog where applicable.
4. **Given** a catalog record is eligible for removal, **When** the administrator confirms deletion, **Then** it is removed from the catalog and shopper-facing views.
5. **Given** a category or subcategory is still referenced by dependent catalog records, **When** the administrator attempts to delete it, **Then** deletion is prevented and the blocking dependencies are explained.
6. **Given** an administrator submits missing or invalid catalog data, **When** the data is checked, **Then** the record is not saved and clear field-specific correction guidance is shown.
7. **Given** a product appears in a submitted order, **When** an administrator removes it from the active catalog, **Then** it is archived from storefront and future purchase options while remaining available in historical order details.

### User Story 3 - Review registered users and orders (Priority: P1)

As an administrator, I want to review registered users and submitted orders so I can support store operations.

**Why this priority**: The user and order lists provide the operational visibility needed to fulfill purchases.

**Independent Test**: Open each list as an administrator and verify registered customer records and submitted order details are visible and understandable.

**Acceptance Scenarios**:

1. **Given** the administrator is signed in, **When** they open the registered-user list, **Then** each registered customer is identified by their mobile number, without exposing delivery addresses in that list.
2. **Given** the administrator is signed in, **When** they open the order list, **Then** submitted orders show an order identifier, customer name, mobile number and delivery address for fulfillment, ordered items and quantities, total, and current status.
3. **Given** there are no registered users or submitted orders, **When** the administrator opens the corresponding list, **Then** a clear empty state is shown.
4. **Given** user or order data cannot be loaded, **When** the failure occurs, **Then** an explicit error and retry path are shown instead of an empty-list state.
5. **Given** the administrator is reviewing customers or orders, **When** they search by customer mobile number or order identifier/mobile number, filter orders by status or date, or navigate result pages, **Then** only matching records are shown.

### User Story 4 - Update order status (Priority: P1)

As an administrator, I want to mark an in-progress order as delivered so the order list reflects fulfillment progress.

**Why this priority**: Accurate fulfillment status lets store staff track completion and communicate reliable order information.

**Independent Test**: Select an in-progress order, mark it delivered, and verify the new status is shown in the order list and order details.

**Acceptance Scenarios**:

1. **Given** an order is In Progress, **When** the administrator changes its status to Delivered, **Then** the updated status is saved and shown consistently.
2. **Given** the administrator attempts to change an order to an unsupported status, **When** the change is submitted, **Then** it is rejected and the current status remains unchanged.
3. **Given** an order-status update fails, **When** the failure is reported, **Then** the prior status remains visible and an explicit retry message is shown.
4. **Given** an order is already Delivered, **When** the administrator views it, **Then** its status is clearly shown and it is not presented as an In Progress order.
5. **Given** an order status changes to Delivered, **When** the update succeeds, **Then** the order record reflects the status change without sending an automatic customer notification.

### Edge Cases

- Repeated submission of a create or delete action MUST NOT create duplicate records or misleading success messages.
- Category or subcategory names that conflict with an existing record at the same level MUST be rejected with clear guidance.
- Product price, required text, category assignment, and availability values MUST be validated before a product is saved.
- A product referenced by a submitted order MUST be archived rather than permanently deleted; it MUST remain available in the historical order details and MUST not be offered for new purchases.
- Removing a category with subcategories or products, or a subcategory with products, MUST be blocked until dependent records are reassigned or removed.
- A stale edit submitted after another administrator has changed a record MUST be detected or safely resolved without silently overwriting newer information.
- A direct link to an administrative page MUST enforce administrator access even if the visitor bypasses the visible sign-in screen.
- Expired administrator sessions MUST not allow further reads or changes to protected information.
- Customer and order details MUST only be visible to authenticated administrators.
- An order status update MUST not silently apply to a different order if the selected order has changed or no longer exists.
- Deleting or updating a record MUST produce clear success or failure feedback.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The management area MUST require administrator authentication and MUST deny access to signed-out visitors and customer accounts without the administrator role.
- **FR-002**: The system MUST provide a provisioned administrator username and password without a public administrator registration flow.
- **FR-003**: Administrator credentials MUST be handled as protected configuration and MUST NOT be hard-coded into source code, exposed to shoppers, or committed to the repository.
- **FR-004**: The management area MUST allow an administrator to create, view, update, and delete categories, subcategories, and products.
- **FR-005**: The system MUST validate required catalog fields, allowed values, and field limits before creating or updating catalog records and MUST report invalid input explicitly.
- **FR-006**: The system MUST prevent deleting a category or subcategory while it has dependent catalog records and MUST explain which dependency must be addressed.
- **FR-007**: The management area MUST provide a registered-user list showing the registered customer's mobile number and MUST NOT show delivery addresses in that list.
- **FR-008**: The management area MUST provide an order list showing an order identifier, customer name, mobile number, delivery address, items, quantities, total, and status.
- **FR-009**: The administrator MUST be able to change an order status from In Progress to Delivered.
- **FR-010**: The system MUST reject unsupported order-status changes and MUST retain the existing status when a change fails.
- **FR-011**: The system MUST provide clear loading, empty, success, validation-error, and service-error states for administrative lists and changes.
- **FR-012**: The system MUST end access to protected management data when the administrator signs out or the administrator session expires.
- **FR-013**: The management area MUST be usable on phone and larger screens, with labeled keyboard-operable controls and visible keyboard focus.
- **FR-014**: The system MUST NOT require an administrator to change the provisioned password at first sign-in; any manual password change is performed through secure credential provisioning.
- **FR-015**: When an administrator removes a product referenced by a submitted order, the system MUST archive it from the active storefront catalog while retaining its details for historical order records.
- **FR-016**: Changing an order status MUST update the order record without automatically notifying the customer.
- **FR-017**: The registered-user list MUST be searchable by customer mobile number; the order list MUST be searchable by order identifier or customer mobile number and filterable by order status or date. Both lists MUST present results in navigable pages.
- **FR-018**: Categories, subcategories, products, customer accounts, orders, and cart contents MUST be stored as durable records in their respective MySQL tables.
- **FR-019**: The browser cookie used to identify a cart MUST contain only a cart/session identifier; cart contents, prices, customer details, and order data MUST NOT be treated as cookie data or as the authoritative source of truth.
- **FR-020**: Cart prices used for order totals MUST be determined from authoritative catalog data, not trusted from browser-provided cookie values.

### Accessibility Requirements

- Administrative forms MUST provide visible, programmatically associated labels and clear field-level validation.
- Catalog actions, list navigation, and order-status controls MUST be keyboard-operable with visible focus.
- Success, empty, and error states MUST be perceivable without relying on color alone.

### Key Entities *(include if feature involves data)*

- **Administrator Account**: A provisioned account allowed to access store-management operations, separate from shopper accounts.
- **Category**: A top-level catalog grouping that may contain subcategories and products.
- **Subcategory**: A catalog grouping associated with a parent category and containing products.
- **Product**: A catalog item with validated details, category assignment, and storefront availability or archive state.
- **Customer Account**: A registered shopper record identified by a mobile number.
- **Order**: A submitted purchase with customer name, mobile number, delivery address, items, total, and fulfillment status information.
- **Order Status**: A supported fulfillment state, including In Progress and Delivered.
- **Cart**: A shopper's active collection of cart items, persisted in the database and identified in the browser by a cart/session identifier.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In access-control tests, 100% of unauthenticated and customer-only attempts to open protected administrative pages are denied.
- **SC-002**: At least 90% of usability-test participants acting as administrators can create or edit a product and save valid changes within two minutes without assistance.
- **SC-003**: In acceptance tests, each category, subcategory, and product create, read, update, and eligible delete operation produces the expected record state.
- **SC-004**: In acceptance tests, invalid catalog data and deletion of catalog records with active dependencies never produce a successful change.
- **SC-005**: In acceptance tests, every supported In Progress-to-Delivered update is reflected in the order list, and unsupported transitions do not change the order.
- **SC-006**: At least 90% of usability-test participants acting as administrators can locate a specific customer or order in the relevant list within one minute.

## Assumptions

- Administrator sign-in is separate from shopper mobile OTP authentication and uses a provisioned administrator account; no public administrator registration is provided.
- The administrator may keep the provisioned password until it is changed through secure credential provisioning; an enforced first-sign-in password change or an in-app password-change flow is not part of this feature.
- Credential values from the request are intentionally not reproduced. Because a credential was disclosed in conversation, it MUST be replaced before use; administrator credentials MUST be provisioned and stored outside committed project files.
- This feature supports one administrator identity unless a later requirement expands administration to multiple roles or accounts.
- Orders begin in the In Progress state; this feature supports the requested transition to Delivered and does not define cancellation, refunds, or other order states.
- Order-status changes do not send automatic customer notifications in this feature.
- Registered-user details are read-only in this feature; the registered-user list shows mobile numbers only, while delivery details are limited to the associated order.
- A category or subcategory cannot be deleted while dependent catalog records remain assigned to it.
- Products referenced by submitted orders are archived from the active catalog and retained for historical order details.
- Product fields follow the existing storefront catalog requirements; this feature does not define new product attributes.
- MySQL is the durable source of truth for catalog, customer, order, and cart records; the cart cookie carries only an identifier used to find the persisted cart.
