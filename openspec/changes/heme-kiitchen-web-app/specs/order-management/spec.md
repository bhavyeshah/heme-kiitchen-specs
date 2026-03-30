## ADDED Requirements

### Requirement: Every order has a consistent data structure
The system SHALL represent all orders (online and offline) with a unified schema: order ID, source (online/offline), status, items (with quantities), total price, customer name, phone, delivery_type, delivery_address (conditional), delivery_charges_applicable, payment method, payment status, optional UPI reference, deleted (boolean, default false), deleted_at (timestamp, nullable), and created/updated timestamps.

The `delivery_address` field SHALL be required when `delivery_type` is "home_delivery" and SHALL be null when `delivery_type` is "pickup".

The `delivery_charges_applicable` field SHALL be derived as follows:
- `false` when `delivery_type` is "pickup"
- `false` when `delivery_type` is "home_delivery" and `total_price` > 1500
- `true` when `delivery_type` is "home_delivery" and `total_price` <= 1500

#### Scenario: Online order has source "online"
- **WHEN** an order is created through the customer checkout flow
- **THEN** the order record has source = "online"

#### Scenario: Offline order has source "offline"
- **WHEN** an order is created by an admin through the admin panel
- **THEN** the order record has source = "offline"

#### Scenario: Home delivery order above threshold has no delivery charge
- **WHEN** an order is created with delivery_type = "home_delivery" and total_price > 1500
- **THEN** delivery_charges_applicable = false

#### Scenario: Home delivery order at or below threshold has delivery charge
- **WHEN** an order is created with delivery_type = "home_delivery" and total_price <= 1500
- **THEN** delivery_charges_applicable = true

#### Scenario: Pickup order never has delivery charge
- **WHEN** an order is created with delivery_type = "pickup"
- **THEN** delivery_charges_applicable = false and delivery_address = null

### Requirement: Order status tracks fulfilment lifecycle
The system SHALL support the following statuses: pending, confirmed, preparing, dispatched, completed, cancelled, declined.

- `pending` — order placed, awaiting admin review. No fulfilment work begins at this stage.
- `confirmed` — admin has accepted the order. Fulfilment lifecycle begins here.
- `preparing` — order is being prepared.
- `dispatched` — applicable to home delivery orders only. Skipped for pickup.
- `completed` — order delivered or picked up. Terminal.
- `cancelled` — order was accepted but subsequently stopped. Terminal.
- `declined` — admin rejected the order at the pending stage before accepting. Terminal. Distinct from cancelled.

#### Scenario: New order starts as pending
- **WHEN** any order is created
- **THEN** its status is set to "pending"

#### Scenario: Admin accepts a pending order
- **WHEN** an admin accepts an order with status "pending"
- **THEN** the order status changes to "confirmed" and updated_at is refreshed

#### Scenario: Admin declines a pending order
- **WHEN** an admin declines an order with status "pending"
- **THEN** the order status changes to "declined" and updated_at is refreshed

#### Scenario: Only pending orders can be declined
- **WHEN** an admin attempts to decline an order that is not in "pending" status
- **THEN** the system rejects the update with a validation error

#### Scenario: Admin updates order status through fulfilment
- **WHEN** an admin changes an order's status to a valid next value after confirmation
- **THEN** the order record is updated and updated_at is refreshed

#### Scenario: Admin sets invalid status
- **WHEN** an admin attempts to set a status value not in the allowed list
- **THEN** the system rejects the update and returns a validation error

#### Scenario: Pickup order completed without dispatched step
- **WHEN** an admin marks a pickup order as "completed"
- **THEN** the system accepts the transition even if the order was never set to "dispatched"

#### Scenario: Admin cancels a confirmed or in-progress order
- **WHEN** an admin sets an order's status to "cancelled" and the current status is "confirmed", "preparing", or "dispatched"
- **THEN** the order status is updated to "cancelled" and updated_at is refreshed

#### Scenario: Admin cannot cancel a pending order
- **WHEN** an admin attempts to set status to "cancelled" on a "pending" order
- **THEN** the system rejects the update with a validation error — pending orders must be declined, not cancelled

#### Scenario: Admin cannot transition out of a terminal status
- **WHEN** an admin attempts to change the status of an order with status "completed", "cancelled", or "declined"
- **THEN** the system rejects the update with a validation error

### Requirement: Admin can edit order details
The system SHALL allow an admin to update any field of an order — including items, delivery type, delivery address, customer name, phone, and payment method — to correct erroneous data. Edits SHALL be accepted on orders with any non-terminal status. Editing a terminal order (completed, cancelled, or declined) SHALL be rejected.

When items or delivery type are edited, `total_price` and `delivery_charges_applicable` SHALL be recalculated automatically.

#### Scenario: Admin edits customer details
- **WHEN** an admin updates the customer name, phone, or delivery address on a non-terminal order
- **THEN** the order record is updated and updated_at is refreshed

#### Scenario: Admin changes order items
- **WHEN** an admin modifies the items (add, remove, or change quantity) on a non-terminal order
- **THEN** the order record is updated, total_price is recalculated, and delivery_charges_applicable is re-derived from the new total

#### Scenario: Admin changes delivery type
- **WHEN** an admin changes delivery_type from "pickup" to "home_delivery" or vice versa on a non-terminal order
- **THEN** the order record is updated, delivery_charges_applicable is re-derived, and delivery_address validation applies to the new delivery type

#### Scenario: Admin cannot edit a terminal order
- **WHEN** an admin attempts to edit an order with status "completed", "cancelled", or "declined"
- **THEN** the system rejects the update with a validation error

### Requirement: Admin can soft-delete an erroneous order
The system SHALL allow an admin to remove an order that was created by mistake. Removal SHALL be a soft delete: the order record is retained with `deleted = true` and `deleted_at` set to the current timestamp. Soft-deleted orders SHALL be excluded from all normal order list views and counts.

#### Scenario: Admin removes an erroneous order
- **WHEN** an admin deletes an order
- **THEN** the order's deleted flag is set to true, deleted_at is set to the current timestamp, and the order no longer appears in normal order lists

#### Scenario: Deleted orders are excluded from default list
- **WHEN** GET /api/orders is called without a deleted filter
- **THEN** orders with deleted = true are not included in the response

#### Scenario: Admin can view deleted orders
- **WHEN** GET /api/orders is called with deleted=true filter
- **THEN** only soft-deleted orders are returned

#### Scenario: Deleted order is not editable
- **WHEN** an admin attempts to edit or update the status of a deleted order
- **THEN** the system rejects the request with a validation error

### Requirement: Payment status is tracked independently from order status
The system SHALL track payment status separately (unpaid, paid, refunded) so it can be updated without changing the fulfilment status.

#### Scenario: Order defaults to unpaid
- **WHEN** any order is created
- **THEN** payment_status is set to "unpaid"

#### Scenario: Admin marks order as paid
- **WHEN** an admin updates payment_status to "paid"
- **THEN** the order record reflects paid status and the updated_at timestamp is refreshed

### Requirement: Orders can be listed and filtered
The system SHALL provide an API endpoint to list all orders, with optional filters for source (online/offline), status, payment_status, and delivery_type.

#### Scenario: List all orders
- **WHEN** a request is made to GET /api/orders with no filters
- **THEN** all orders are returned sorted by created_at descending

#### Scenario: Filter by source
- **WHEN** a request includes source=offline filter
- **THEN** only offline orders are returned

#### Scenario: Filter by status
- **WHEN** a request includes status=pending filter
- **THEN** only orders with status "pending" are returned

#### Scenario: Filter by delivery type
- **WHEN** a request includes delivery_type=pickup filter
- **THEN** only pickup orders are returned
