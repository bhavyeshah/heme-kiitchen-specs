## ADDED Requirements

### Requirement: Customer can select dips for a platter
The system SHALL provide an interactive platter builder page where a customer can select one or more dips from the active product catalog to include in their platter.

#### Scenario: Dips are listed for selection
- **WHEN** a customer opens the platter builder
- **THEN** all active dips are displayed with their name, image, and price, each selectable via a toggle or checkbox

#### Scenario: Customer selects multiple dips
- **WHEN** a customer selects more than one dip
- **THEN** all selected dips are highlighted and included in the running platter summary

#### Scenario: Customer deselects a dip
- **WHEN** a customer deselects a previously selected dip
- **THEN** that dip is removed from the platter summary and its price is subtracted from the total

### Requirement: Customer can add add-ons to a platter
The system SHALL allow customers to add accompaniments (falafel, lavash, or other configured add-ons) to their platter.

#### Scenario: Add-ons are listed separately from dips
- **WHEN** a customer views the platter builder
- **THEN** add-ons are shown in a distinct section from dips, each with name and price

#### Scenario: Customer adds an add-on
- **WHEN** a customer selects an add-on
- **THEN** the add-on is included in the platter summary with its price added to the total

### Requirement: Platter builder shows a live order summary
The system SHALL display a live summary panel showing all selected dips and add-ons with individual prices and a running total.

#### Scenario: Summary updates on selection change
- **WHEN** a customer adds or removes a dip or add-on
- **THEN** the summary panel updates immediately to reflect the current selection and total price

#### Scenario: Proceed to checkout from platter builder
- **WHEN** a customer has at least one item selected and clicks "Proceed to Checkout"
- **THEN** the system navigates to the checkout page with the platter selection pre-populated

#### Scenario: Proceed to checkout with empty platter
- **WHEN** a customer clicks "Proceed to Checkout" with no items selected
- **THEN** the system displays an inline error indicating at least one item must be selected
