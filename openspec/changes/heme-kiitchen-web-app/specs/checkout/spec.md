## ADDED Requirements

### Requirement: Customer provides delivery details at checkout
The system SHALL collect the customer's name, phone number, and delivery address before placing an order.

#### Scenario: Customer fills delivery details
- **WHEN** a customer is on the checkout page
- **THEN** the form shows fields for full name, phone number, and delivery address (all required)

#### Scenario: Customer submits with missing required fields
- **WHEN** a customer submits the checkout form without filling all required fields
- **THEN** the system highlights the missing fields with inline error messages and does not place the order

### Requirement: Customer selects a payment method
The system SHALL offer two payment methods: Cash on Delivery (COD) and manual UPI. The customer MUST select one before placing the order.

#### Scenario: Customer selects COD
- **WHEN** a customer selects "Cash on Delivery"
- **THEN** no UPI details are required and the order can be placed with payment_method = "COD"

#### Scenario: Customer selects UPI
- **WHEN** a customer selects "UPI"
- **THEN** the system displays the business UPI ID and instructs the customer to complete the transfer and note their transaction reference

#### Scenario: Customer submits UPI order without transaction reference
- **WHEN** a customer selects UPI but does not enter a transaction reference and submits
- **THEN** the system displays an inline error requiring the transaction reference field

### Requirement: Order is created on submission
The system SHALL create an order record when the customer submits a valid checkout form.

#### Scenario: Successful order placement
- **WHEN** a customer submits a complete and valid checkout form
- **THEN** the system creates an order with status "pending", records all items, payment method, and customer details, and displays a confirmation message with an order reference number

#### Scenario: Order confirmation shown to customer
- **WHEN** an order is successfully created
- **THEN** the customer sees a confirmation page with the order reference number, summary of items, total price, and a note on next steps (e.g., "We will confirm your order via WhatsApp")

#### Scenario: Duplicate submission prevention
- **WHEN** a customer submits the checkout form and the request is still processing
- **THEN** the submit button is disabled to prevent duplicate orders
