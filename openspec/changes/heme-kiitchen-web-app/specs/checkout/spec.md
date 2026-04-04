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

The phone number field SHALL be labelled with a helper text: "Enter your WhatsApp number — order updates will be sent here." The field SHALL accept only valid Indian mobile numbers (10 digits, starting with 6, 7, 8, or 9). The number SHALL be stored with the +91 country code prefix.

#### Scenario: Customer fills details for home delivery
- **WHEN** a customer selects "Home Delivery" and is on the checkout page
- **THEN** the form shows fields for full name, phone number, and delivery address (all required)

#### Scenario: Customer fills details for pickup
- **WHEN** a customer selects "Pickup" and is on the checkout page
- **THEN** the form shows fields for full name and phone number only (address not shown)

#### Scenario: Phone number field shows WhatsApp helper text
- **WHEN** a customer views the checkout form
- **THEN** the phone number field displays helper text: "Enter your WhatsApp number — order updates will be sent here"

#### Scenario: Customer submits with missing required fields
- **WHEN** a customer submits the checkout form without filling all required fields
- **THEN** the system highlights the missing fields with inline error messages and does not place the order

#### Scenario: Customer submits invalid phone number
- **WHEN** a customer enters a phone number that is not a valid 10-digit Indian mobile number
- **THEN** the system shows an inline error and does not place the order

#### Scenario: Phone number stored with country code
- **WHEN** a valid 10-digit phone number is submitted
- **THEN** the order record stores it as +91XXXXXXXXXX

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
- **THEN** the system displays a note: "After placing your order, contact us on WhatsApp with your Order ID to receive payment details and complete your payment." No transaction reference field is shown.

### Requirement: Order is created on submission
The system SHALL create an order record when the customer submits a valid checkout form.

#### Scenario: Successful order placement
- **WHEN** a customer submits a complete and valid checkout form
- **THEN** the system creates an order with status "pending", records all items, delivery type, payment method, and customer details, and displays a confirmation message with an order reference number

#### Scenario: Order confirmation shown to customer for pickup
- **WHEN** an order is successfully created with delivery_type = "pickup"
- **THEN** the customer sees a confirmation page with: the order reference number (prominently displayed), summary of items, total price, a note "We will confirm your order via WhatsApp", and an instruction: "To make any changes to your order, please contact us on WhatsApp with your Order ID"

#### Scenario: Order confirmation shown to customer for home delivery with UPI
- **WHEN** an order is successfully created with delivery_type = "home_delivery"
- **THEN** the customer sees a confirmation page with: the order reference number (prominently displayed), summary of items, total price, delivery charge notice, and an instruction: "To complete your payment and confirm your order, contact us on WhatsApp at [business number] with your Order ID. You can also request any changes at this stage — payment will only be collected once your order is finalised."

#### Scenario: Duplicate submission prevention
- **WHEN** a customer submits the checkout form and the request is still processing
- **THEN** the submit button is disabled to prevent duplicate orders

### Requirement: Customer can add special instructions for the chef at checkout
The system SHALL provide an optional textarea on the checkout page for the customer to enter any special instructions for the chef (e.g., dietary notes, packaging requests). The field SHALL be labelled "Any special instructions for the chef? (optional)" and SHALL display a live character counter ("X / 500"). The field SHALL accept a maximum of 500 characters. Submission with instructions exceeding 500 characters SHALL be blocked with an inline error.

When provided, the value SHALL be passed to the order creation API as `special_instructions`.

#### Scenario: Customer enters special instructions
- **WHEN** a customer types in the special instructions textarea
- **THEN** the character counter updates live and the value is included in the order on submission

#### Scenario: Customer leaves special instructions blank
- **WHEN** a customer submits without entering special instructions
- **THEN** the order is created with special_instructions = null and no error is shown

#### Scenario: Customer exceeds character limit
- **WHEN** a customer types more than 500 characters in the special instructions field
- **THEN** an inline error is shown and the form cannot be submitted until the text is within the limit

### Requirement: Customers cannot modify or cancel orders through the website
The system SHALL NOT provide any customer-facing interface to edit, cancel, or track an order after it has been placed. The customer's only post-order action is to contact the admin via WhatsApp with their order ID.

All order modifications are made exclusively by the admin through the admin panel. All order updates — status changes and any modifications — are communicated to the customer via WhatsApp. The order reference number on the confirmation page is the customer's identifier for any WhatsApp communication with the admin.

#### Scenario: No order management UI exists for customers
- **WHEN** a customer has placed an order
- **THEN** there is no page, button, or link on the website to view order status, modify items, or cancel the order

#### Scenario: Customer wants to make a change to their order
- **WHEN** a customer wants to modify or cancel their order
- **THEN** they must contact the admin via WhatsApp with their order ID; the admin makes the change in the admin panel
