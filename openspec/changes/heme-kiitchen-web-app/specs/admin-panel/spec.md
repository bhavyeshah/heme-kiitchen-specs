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

The phone number field SHALL be labelled "Customer WhatsApp Number" and SHALL accept only valid Indian mobile numbers (10 digits, starting with 6, 7, 8, or 9). The number SHALL be stored with the +91 country code prefix. This ensures WhatsApp status notifications reach the customer consistently for both online and offline orders.

#### Scenario: Admin creates offline order
- **WHEN** an admin fills in customer details, selects items from the product catalog, chooses payment method, and submits the form
- **THEN** a new order is created with source = "offline" and status = "pending"

#### Scenario: Admin submits offline order with invalid phone number
- **WHEN** an admin enters a phone number that is not a valid 10-digit Indian mobile number
- **THEN** the system shows a validation error and does not create the order

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

### Requirement: Brand logo is displayed on all admin panel pages
The system SHALL display the Hémé Kiitchen logo in the admin panel header across all /admin/* pages — including the login page, orders, products, branding, and any future admin sections. The logo reinforces brand identity and provides consistent visual anchoring for the admin interface.

#### Scenario: Logo appears on every admin page
- **WHEN** an authenticated admin navigates to any admin panel page
- **THEN** the Hémé Kiitchen logo is visible in the admin header

#### Scenario: Logo appears on admin login page
- **WHEN** an unauthenticated user lands on /admin/login
- **THEN** the Hémé Kiitchen logo is displayed above the login form

### Requirement: Admin panel is optimised for mobile use
The system SHALL render all admin panel pages with a mobile-first layout, with tap targets no smaller than 44×44px, readable font sizes, and no horizontal overflow on screens 375px wide.

#### Scenario: Admin views orders on phone
- **WHEN** an authenticated admin opens /admin/orders on a 375px wide screen
- **THEN** all order cards are fully readable, buttons are tappable, and no content is cut off

### Requirement: Admin order list indicates orders with special instructions
The system SHALL display a "Note" badge on any order card in the order list where `special_instructions` is non-empty. The badge SHALL be visible without opening the order, so the admin can identify at a glance which orders require special attention before preparation.

#### Scenario: Order with special instructions shows badge
- **WHEN** an admin views the order list and an order has non-empty special_instructions
- **THEN** a "Note" badge is displayed on that order's card

#### Scenario: Order without special instructions shows no badge
- **WHEN** an order has special_instructions = null or empty
- **THEN** no badge is shown on that order's card

### Requirement: Admin order detail view highlights special instructions prominently
The system SHALL display special instructions in a visually distinct callout box in the order detail view, positioned above the items and status sections, so it is the first thing the admin sees when opening an order. The callout SHALL only be rendered when special_instructions is non-empty.

The callout SHALL be styled to stand out from surrounding content (e.g., distinct background colour, border, or icon) and SHALL be labelled "Chef Note" or equivalent.

The admin SHALL be able to edit the special_instructions field from the order detail view on any non-terminal order, consistent with the existing edit requirement.

#### Scenario: Admin views order with special instructions
- **WHEN** an admin opens an order that has non-empty special_instructions
- **THEN** a highlighted callout labelled "Chef Note" is shown at the top of the detail view with the instruction text

#### Scenario: Admin views order without special instructions
- **WHEN** an admin opens an order with special_instructions = null
- **THEN** no chef note callout is rendered in the detail view

#### Scenario: Admin edits special instructions from detail view
- **WHEN** an admin taps to edit special instructions on a non-terminal order and saves
- **THEN** the updated value is persisted and the callout reflects the new text immediately

### Requirement: Admin can add special instructions when creating an offline order
The system SHALL include the special instructions field in the offline order creation form, identical in behaviour to the checkout flow — optional, max 500 characters, with a live character counter.

#### Scenario: Admin enters special instructions in offline order form
- **WHEN** an admin fills in special instructions and submits the offline order form
- **THEN** the order is created with the provided special_instructions value

#### Scenario: Admin leaves special instructions blank in offline order form
- **WHEN** an admin submits the offline order form without entering special instructions
- **THEN** the order is created with special_instructions = null

### Requirement: Admin can edit homepage brand content and social links
The system SHALL provide a "Branding" section in the admin panel where the admin can update the homepage tagline, brand description, highlights list, and Instagram handle. Changes SHALL take effect on the storefront immediately on next page load — no code change or redeployment required.

The tagline SHALL have a maximum of 150 characters. The brand description SHALL have a maximum of 500 characters. Highlights SHALL be a configurable list of short strings (max 80 characters each); the admin can add new highlights, edit existing ones, and remove any highlight. The Instagram handle is optional — if left blank, no Instagram link is shown on the storefront.

The logo is intentionally NOT editable via the admin panel — it is a registered brand asset managed via file replacement, representing a significant business decision.

#### Scenario: Admin updates tagline
- **WHEN** an admin edits the tagline and saves
- **THEN** site-content.json is updated and the homepage reflects the new tagline on next load

#### Scenario: Admin updates brand description
- **WHEN** an admin edits the brand description and saves
- **THEN** site-content.json is updated and the homepage reflects the new description on next load

#### Scenario: Admin adds a highlight
- **WHEN** an admin adds a new highlight and saves
- **THEN** the new highlight appears in the homepage highlights section on next load

#### Scenario: Admin removes a highlight
- **WHEN** an admin removes a highlight and saves
- **THEN** the highlight no longer appears on the homepage

#### Scenario: Admin submits tagline exceeding character limit
- **WHEN** an admin enters a tagline longer than 150 characters
- **THEN** the system shows a validation error and does not save

#### Scenario: Admin submits empty tagline
- **WHEN** an admin clears the tagline field and saves
- **THEN** the system shows a validation error and does not save

#### Scenario: Admin sets Instagram handle
- **WHEN** an admin enters an Instagram handle and saves
- **THEN** site-content.json is updated and the Instagram link appears in the storefront footer on next load

#### Scenario: Admin clears Instagram handle
- **WHEN** an admin removes the Instagram handle and saves
- **THEN** the Instagram link is no longer rendered in the storefront footer
