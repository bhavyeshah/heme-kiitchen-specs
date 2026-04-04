<!-- DEFERRED: Order availability / blackout management is out of scope for the initial release.
     For launch, the admin manages unavailability manually by communicating with customers
     directly (e.g., via WhatsApp status or Instagram).

     FUTURE DIRECTION (captured for when this is scoped):
     - Admin defines blackout windows (specific dates, days of the week, or time ranges)
       during which home delivery orders are not accepted.
     - Blackout windows are shown as a ticker/notice on the checkout screen.
     - Unavailable delivery slots are visually greyed out or disabled on the delivery
       date selector so customers cannot accidentally place unserviceable orders.
     - Pickup orders may or may not be affected — to be decided at scoping time.

     OPEN QUESTIONS FOR FUTURE SCOPING:
     - Is the blackout specific to home delivery only, or also affects pickup?
     - Does the admin set a full-day blackout, or specific time windows within a day?
     - Is this a recurring schedule (e.g. "no delivery on Sundays") or one-off date blocks?
     - Does a live blackout prevent order placement entirely, or just delivery date selection?
     - What is the ticker message — fixed ("Orders not accepted on selected dates") or
       admin-configurable per blackout window?
     - Should existing confirmed orders within a blackout window be flagged or auto-notified?
     - Is there a "next available delivery date" suggestion shown to the customer? -->


## DEFERRED Requirements

### Requirement: Admin can define delivery blackout windows
The system SHALL allow the admin to mark specific dates, date ranges, or recurring day patterns
as unavailable for home delivery orders. Blackout windows SHALL be configurable entirely from
the admin panel without any code change or redeployment.

#### Scenario: Admin marks a date as unavailable for delivery
- **WHEN** an admin adds a blackout entry for a specific date
- **THEN** home delivery orders cannot be placed for that date; the date is visually disabled
  on the customer checkout delivery date selector

#### Scenario: Admin marks a recurring day as unavailable
- **WHEN** an admin sets a recurring blackout (e.g. "no delivery on Sundays")
- **THEN** all future occurrences of that day are disabled on the delivery date selector

#### Scenario: Admin removes a blackout window
- **WHEN** an admin deletes a blackout entry
- **THEN** that date/pattern is immediately available again for delivery orders

### Requirement: Checkout shows a ticker when delivery blackouts are active
The system SHALL display a notice or ticker on the checkout screen whenever there are upcoming
blackout windows that affect delivery availability. The notice SHALL be visible before the
customer selects a delivery date, so they are aware of constraints upfront.

#### Scenario: Customer views checkout with an upcoming blackout
- **WHEN** there is at least one active or upcoming blackout window configured
- **THEN** a ticker or notice is shown on the checkout screen informing the customer that
  delivery is unavailable on certain dates

#### Scenario: No blackout windows are active
- **WHEN** no blackout windows are configured
- **THEN** no ticker is shown and all dates on the delivery date selector are available

### Requirement: Unavailable delivery dates are greyed out on the date selector
The system SHALL render the delivery date picker with blackout dates visually disabled
(greyed out, unselectable) so customers cannot place orders for dates the business cannot service.

#### Scenario: Customer attempts to select a blacked-out date
- **WHEN** a customer taps a greyed-out date on the delivery date selector
- **THEN** no selection is made; the date remains visually disabled with no error shown
  (the visual treatment alone is sufficient)

#### Scenario: Customer selects an available date
- **WHEN** a customer selects a date that is not blacked out
- **THEN** the date is selected normally and checkout proceeds
