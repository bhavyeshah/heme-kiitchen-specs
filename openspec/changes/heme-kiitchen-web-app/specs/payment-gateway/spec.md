<!-- DEFERRED: Online payment gateway integration is out of scope for the initial release.
     Current payment flow is fully offline — COD (cash at pickup) and manual UPI via WhatsApp.
     Gateway integration will be introduced as a separate change when the business is ready to
     accept pre-payments online.

     GATEWAY CHOICE: Razorpay is the preferred candidate (supports UPI, cards, netbanking,
     wallets; widely used in India; good developer APIs). Other gateways (PayU, CCAvenue,
     Cashfree) are also viable and will be evaluated at scoping time.

     REFUND POLICY: Refund rules for declined, cancelled, or modified orders will be defined
     by the business at scoping time and are intentionally left open here.

     CURRENT DESIGN COMPATIBILITY NOTES (read before scoping):
     - The Order type already has `payment_method` (open string) and `payment_status` fields —
       adding "razorpay" as a valid payment_method value requires no schema change.
     - A nullable `gateway_reference` field will need to be added to the Order type when this
       is scoped. Shape: { provider, gateway_order_id, payment_id, signature } — null for all
       COD and manual UPI orders.
     - The order-first flow (Option B) is the intended integration pattern: our order is created
       first (payment_status = "awaiting_payment"), then the gateway payment is initiated, then
       a webhook from the gateway confirms payment and transitions the order automatically.
       This mirrors the current offline flow and keeps the Order ID visible to the customer
       before payment completes.
     - The pending payment expiry logic (see below) should be implemented as a background
       job / scheduled task — it is the same abstraction as any future cron-based feature. -->


## DEFERRED Requirements

### Requirement: Online payment is accepted via a payment gateway
The system SHALL support an "online" payment method alongside COD and manual UPI. When a customer selects online payment at checkout, the system SHALL initiate a payment session with the configured gateway (e.g. Razorpay) and present the gateway's payment widget to the customer. Orders are created in the system before payment is collected (order-first flow).

The order SHALL be created with `payment_status = "awaiting_payment"` and SHALL only transition to `payment_status = "paid"` upon receiving a verified webhook confirmation from the gateway. The gateway transaction reference SHALL be stored on the order for reconciliation.

#### Scenario: Customer completes online payment
- **WHEN** a customer selects online payment, submits the checkout form, and completes payment in the gateway widget
- **THEN** the gateway sends a webhook to the API; the API verifies the signature; payment_status is updated to "paid"; a WhatsApp confirmation is sent to the customer

#### Scenario: Customer abandons payment after order is created
- **WHEN** a customer creates an order with online payment but does not complete payment
- **THEN** the order remains in payment_status = "awaiting_payment" until the expiry window passes

#### Scenario: Gateway payment fails
- **WHEN** a customer's payment attempt fails (insufficient funds, bank decline)
- **THEN** the order remains in "awaiting_payment" state; the customer may retry within the expiry window

### Requirement: Pending online payment orders expire if not completed
To prevent accumulation of abandoned orders and to keep the admin order queue clean, orders with online payment that remain unpaid SHALL be automatically cancelled after 30 minutes. The customer SHALL receive a WhatsApp reminder at the 15-minute mark prompting them to complete payment.

#### Scenario: Customer receives payment reminder at 15 minutes
- **WHEN** an order with payment_status = "awaiting_payment" has been open for 15 minutes
- **THEN** the system sends a WhatsApp notification to the customer's phone with a reminder to complete payment to confirm the order

#### Scenario: Unpaid order is auto-cancelled at 30 minutes
- **WHEN** an order with payment_status = "awaiting_payment" has been open for 30 minutes without payment confirmation
- **THEN** the order status is set to "cancelled" and payment_status is set to "expired"; the customer receives a WhatsApp notification that the order has been cancelled due to payment timeout; no refund action is triggered (no payment was collected)

#### Scenario: Order completed before expiry is not cancelled
- **WHEN** a customer completes payment before the 30-minute window closes
- **THEN** the expiry job skips this order; no cancellation is triggered

### Requirement: Admin can view online payment status per order
The system SHALL display the gateway payment reference and payment status clearly in the order detail view so the admin can reconcile payments without leaving the admin panel.

#### Scenario: Admin views a paid online order
- **WHEN** an admin opens an order with payment_method = "online" and payment_status = "paid"
- **THEN** the gateway transaction reference (payment ID) is visible in the order detail view

### Requirement: Refunds for cancelled or declined online orders
<!-- OPEN: Refund policy and rules to be defined by the business at scoping time. -->
<!-- Questions to resolve:
  - Full refund always, or case-by-case?
  - Who initiates: automatic on cancellation/decline, or admin-triggered?
  - Partial refunds needed for modified orders where customer overpaid?
  - What is the SLA for refund processing communicated to the customer?
  - Gateway refund API calls: synchronous or async (most gateways are async)?
  - Refund status tracking: new field on Order, or separate Refund entity?
-->

The system SHALL support initiating refunds through the gateway API for cancelled or declined online orders. Refund rules, triggers, and customer communication will be specified when this feature is formally scoped.
