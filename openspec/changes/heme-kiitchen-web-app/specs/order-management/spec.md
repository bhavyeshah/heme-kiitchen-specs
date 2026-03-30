## ADDED Requirements

### Requirement: Every order has a consistent data structure
The system SHALL represent all orders (online and offline) with a unified schema: order ID, source (online/offline), status, items (with quantities), total price, customer name, phone, delivery address, payment method, payment status, optional UPI reference, and created/updated timestamps.

#### Scenario: Online order has source "online"
- **WHEN** an order is created through the customer checkout flow
- **THEN** the order record has source = "online"

#### Scenario: Offline order has source "offline"
- **WHEN** an order is created by an admin through the admin panel
- **THEN** the order record has source = "offline"

### Requirement: Order status tracks fulfilment lifecycle
The system SHALL support the following order statuses: pending, confirmed, preparing, dispatched, delivered, cancelled.

#### Scenario: New order starts as pending
- **WHEN** any order is created
- **THEN** its status is set to "pending"

#### Scenario: Admin updates order status
- **WHEN** an admin changes an order's status to a valid next status
- **THEN** the order record is updated and the updated_at timestamp is refreshed

#### Scenario: Admin sets invalid status transition
- **WHEN** an admin attempts to set a status value not in the allowed list
- **THEN** the system rejects the update and returns a validation error

### Requirement: Payment status is tracked independently from order status
The system SHALL track payment status separately (unpaid, paid, refunded) so it can be updated without changing the fulfilment status.

#### Scenario: Order defaults to unpaid
- **WHEN** any order is created
- **THEN** payment_status is set to "unpaid"

#### Scenario: Admin marks order as paid
- **WHEN** an admin updates payment_status to "paid"
- **THEN** the order record reflects paid status and the updated_at timestamp is refreshed

### Requirement: Orders can be listed and filtered
The system SHALL provide an API endpoint to list all orders, with optional filters for source (online/offline), status, and payment_status.

#### Scenario: List all orders
- **WHEN** a request is made to GET /api/orders with no filters
- **THEN** all orders are returned sorted by created_at descending

#### Scenario: Filter by source
- **WHEN** a request includes source=offline filter
- **THEN** only offline orders are returned

#### Scenario: Filter by status
- **WHEN** a request includes status=pending filter
- **THEN** only orders with status "pending" are returned
