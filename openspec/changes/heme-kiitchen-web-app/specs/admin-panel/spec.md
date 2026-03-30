## ADDED Requirements

### Requirement: Admin panel is protected by authentication
The system SHALL require authentication before any admin panel page or action is accessible. Authentication SHALL use a single admin password stored as an environment variable.

#### Scenario: Unauthenticated access is redirected
- **WHEN** an unauthenticated user navigates to any /admin/* route
- **THEN** they are redirected to the admin login page

#### Scenario: Correct password grants access
- **WHEN** an admin enters the correct password on the login page
- **THEN** a session is created and the admin is redirected to the admin dashboard

#### Scenario: Incorrect password is rejected
- **WHEN** an admin enters an incorrect password
- **THEN** an error message is shown and no session is created

### Requirement: Admin can view all orders
The system SHALL display a list of all orders (online and offline) in the admin panel, showing order ID, customer name, items summary, total, source, status, and payment status. The list SHALL be paginated or virtualized for performance.

#### Scenario: Admin views order list
- **WHEN** an authenticated admin navigates to /admin/orders
- **THEN** all orders are displayed sorted by created_at descending with key fields visible

#### Scenario: Admin filters orders by status
- **WHEN** an admin applies a status filter on the order list
- **THEN** only orders matching the selected status are displayed

#### Scenario: Admin opens order detail
- **WHEN** an admin taps an order row
- **THEN** a detail view shows all order fields including full customer address, items, and payment info

### Requirement: Admin can create offline orders
The system SHALL provide a form in the admin panel to create orders manually for customers who order via WhatsApp or phone.

#### Scenario: Admin creates offline order
- **WHEN** an admin fills in customer details, selects items from the product catalog, chooses payment method, and submits the form
- **THEN** a new order is created with source = "offline" and status = "pending"

#### Scenario: Offline order requires at least one item
- **WHEN** an admin submits the offline order form with no items selected
- **THEN** the system displays a validation error and does not create the order

### Requirement: Admin can accept or decline pending orders
The system SHALL display prominent Accept and Decline actions on any order with status "pending". These are the primary actions that start or reject the fulfilment lifecycle.

#### Scenario: Admin accepts a pending order
- **WHEN** an admin taps "Accept" on a pending order
- **THEN** the order status changes to "confirmed" and the order moves out of the pending queue

#### Scenario: Admin declines a pending order
- **WHEN** an admin taps "Decline" on a pending order
- **THEN** the order status changes to "declined" and the order is marked as a terminal rejected state

#### Scenario: Accept and Decline are not shown on non-pending orders
- **WHEN** an admin views an order that is not in "pending" status
- **THEN** the Accept and Decline actions are not shown

### Requirement: Admin can update order and payment status
The system SHALL allow the admin to change a confirmed order's status and payment status from the order detail view.

#### Scenario: Admin updates order status
- **WHEN** an admin selects a new status from the order detail view and saves
- **THEN** the order status is updated and the list view reflects the change

#### Scenario: Admin marks order as paid
- **WHEN** an admin taps "Mark as Paid" on an order after receiving the customer's UPI payment screenshot on WhatsApp
- **THEN** payment_status changes to "paid" immediately without a full page reload

<!-- Inventory management is deferred. /admin/inventory will be added when inventory tracking is plugged in. -->

### Requirement: Admin panel is optimised for mobile use
The system SHALL render all admin panel pages with a mobile-first layout, with tap targets no smaller than 44×44px, readable font sizes, and no horizontal overflow on screens 375px wide.

#### Scenario: Admin views orders on phone
- **WHEN** an authenticated admin opens /admin/orders on a 375px wide screen
- **THEN** all order cards are fully readable, buttons are tappable, and no content is cut off
