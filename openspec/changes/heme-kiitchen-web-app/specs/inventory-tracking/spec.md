## ADDED Requirements

### Requirement: Each product has a stock level
The system SHALL maintain a stock quantity for each product. Stock is stored as a whole number (units available). Products with stock = 0 SHALL be shown as "out of stock" to customers.

#### Scenario: Product shown as out of stock
- **WHEN** a product's stock level is 0
- **THEN** the product card on the storefront displays an "Out of Stock" label and cannot be added to a platter

#### Scenario: Product with stock > 0 is orderable
- **WHEN** a product's stock level is greater than 0
- **THEN** it can be selected in the platter builder

### Requirement: Admin can view current stock levels
The system SHALL display current stock levels for all products in the admin panel inventory view.

#### Scenario: Admin views inventory
- **WHEN** an authenticated admin navigates to /admin/inventory
- **THEN** all products are listed with their current stock quantity

#### Scenario: Low stock is highlighted
- **WHEN** a product's stock level is at or below a configured low-stock threshold (default: 5)
- **THEN** the product row is visually highlighted in the inventory list

### Requirement: Admin can manually update stock levels
The system SHALL allow the admin to set the stock quantity for any product directly from the inventory view.

#### Scenario: Admin updates stock quantity
- **WHEN** an admin enters a new quantity for a product and saves
- **THEN** the stock level is updated and the inventory view reflects the new value

#### Scenario: Negative stock is rejected
- **WHEN** an admin attempts to set a stock quantity below 0
- **THEN** the system rejects the input with a validation error
