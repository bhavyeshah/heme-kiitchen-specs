## ADDED Requirements

### Requirement: Customer selects a delivery type at checkout
The system SHALL require the customer to choose between "Pickup" and "Home Delivery" before placing an order. This selection determines whether a delivery address is required and whether delivery charges apply.

#### Scenario: Customer selects Home Delivery
- **WHEN** a customer selects "Home Delivery"
- **THEN** a delivery address field appears and is required before the order can be submitted

#### Scenario: Customer selects Pickup
- **WHEN** a customer selects "Pickup"
- **THEN** the delivery address field is hidden and the business pickup address is shown to the customer

#### Scenario: No delivery type selected on submit
- **WHEN** a customer submits the checkout form without selecting a delivery type
- **THEN** the system shows an inline error and does not place the order

### Requirement: Delivery charge notice is shown at checkout
The system SHALL display a delivery charge notice on the checkout page based on the order total and selected delivery type. The charge is paid directly to the third-party delivery partner — the system does not collect or calculate the amount.

#### Scenario: Home delivery order above free-delivery threshold
- **WHEN** a customer selects "Home Delivery" and the order total is above 1500 INR
- **THEN** the checkout displays "Free delivery" with no additional notice

#### Scenario: Home delivery order at or below free-delivery threshold
- **WHEN** a customer selects "Home Delivery" and the order total is 1500 INR or less
- **THEN** the checkout displays a notice: "Delivery charges apply — payable directly to the delivery partner on arrival"

#### Scenario: Pickup order shows no delivery charge
- **WHEN** a customer selects "Pickup"
- **THEN** no delivery charge notice is shown

#### Scenario: Delivery charge notice updates when order total changes
- **WHEN** a customer adds or removes items and the total crosses the 1500 INR threshold
- **THEN** the delivery charge notice updates immediately without a page reload

### Requirement: Customer provides delivery details at checkout
The system SHALL collect the customer's name and phone number for all orders. Delivery address SHALL only be collected when delivery type is "Home Delivery".

#### Scenario: Customer fills details for home delivery
- **WHEN** a customer selects "Home Delivery" and is on the checkout page
- **THEN** the form shows fields for full name, phone number, and delivery address (all required)

#### Scenario: Customer fills details for pickup
- **WHEN** a customer selects "Pickup" and is on the checkout page
- **THEN** the form shows fields for full name and phone number only (address not shown)

#### Scenario: Customer submits with missing required fields
- **WHEN** a customer submits the checkout form without filling all required fields
- **THEN** the system highlights the missing fields with inline error messages and does not place the order

### Requirement: Payment method is determined by delivery type
The system SHALL restrict available payment methods based on the selected delivery type. Cash on Delivery (COD) is only available for pickup orders. Home delivery orders require UPI payment.

#### Scenario: Pickup order shows both payment options
- **WHEN** a customer selects "Pickup"
- **THEN** both "Cash on Delivery" and "UPI" are available as payment options

#### Scenario: Home delivery order shows UPI only
- **WHEN** a customer selects "Home Delivery"
- **THEN** only "UPI" is available as a payment option and COD is not shown

#### Scenario: Switching from pickup to home delivery removes COD
- **WHEN** a customer changes delivery type from "Pickup" to "Home Delivery" after having selected COD
- **THEN** COD is removed, UPI is auto-selected, and the UPI payment instructions are shown

#### Scenario: Customer selects COD for pickup
- **WHEN** a customer selects "Cash on Delivery" on a pickup order
- **THEN** no UPI details are required and the order can be placed with payment_method = "COD"

#### Scenario: Customer selects UPI
- **WHEN** a customer selects "UPI"
- **THEN** the system displays the business UPI ID and instructs the customer to complete the transfer and share the payment screenshot with the admin via WhatsApp before the order is confirmed

#### Scenario: Customer submits UPI order without transaction reference
- **WHEN** a customer selects UPI but does not enter a transaction reference and submits
- **THEN** the system displays an inline error requiring the transaction reference field

### Requirement: Order is created on submission
The system SHALL create an order record when the customer submits a valid checkout form.

#### Scenario: Successful order placement
- **WHEN** a customer submits a complete and valid checkout form
- **THEN** the system creates an order with status "pending", records all items, delivery type, payment method, and customer details, and displays a confirmation message with an order reference number

#### Scenario: Order confirmation shown to customer for pickup
- **WHEN** an order is successfully created with delivery_type = "pickup"
- **THEN** the customer sees a confirmation page with the order reference number, summary of items, total price, and a note: "We will confirm your order via WhatsApp"

#### Scenario: Order confirmation shown to customer for home delivery with UPI
- **WHEN** an order is successfully created with delivery_type = "home_delivery"
- **THEN** the customer sees a confirmation page with the order reference number, summary of items, total price, delivery charge notice, and an instruction to share the UPI payment screenshot with the admin on WhatsApp at the business number before the order can be confirmed

#### Scenario: Duplicate submission prevention
- **WHEN** a customer submits the checkout form and the request is still processing
- **THEN** the submit button is disabled to prevent duplicate orders
