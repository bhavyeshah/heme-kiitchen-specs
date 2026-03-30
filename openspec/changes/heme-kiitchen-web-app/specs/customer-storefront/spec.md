## ADDED Requirements

### Requirement: Homepage displays brand identity
The system SHALL render a homepage that communicates Hémé Kiitchen's brand identity: premium, 100% vegetarian, Jain-friendly (No Onion No Garlic), no preservatives, no artificial flavours. The page SHALL display the brand logo, a hero section with a short brand tagline, and navigation to the product listing and platter builder.

#### Scenario: Visitor lands on homepage
- **WHEN** a visitor navigates to the root URL
- **THEN** the page displays the Hémé Kiitchen logo, a hero section, the brand tagline, and links to Products and Build a Platter

#### Scenario: Homepage is responsive on mobile
- **WHEN** a visitor views the homepage on a screen narrower than 480px
- **THEN** all sections stack vertically and remain fully readable without horizontal scrolling

### Requirement: Product listing displays all active dips
The system SHALL display all active products from the product catalog on a dedicated products page. Each product card SHALL show the product image, name, description, and price.

#### Scenario: Products page loads active products
- **WHEN** a visitor navigates to /products
- **THEN** all products with status "active" are displayed as cards with image, name, description, and price

#### Scenario: Product image is shown per dip
- **WHEN** a product card is rendered
- **THEN** the image is loaded from the product's configured image path and displayed without broken image placeholders

#### Scenario: No active products exist
- **WHEN** no products have status "active"
- **THEN** the page displays a "Coming soon" message instead of an empty grid

### Requirement: Information page surfaces FSSAI details
The system SHALL include an information or about page that displays the FSSAI licence number and offers access to the registration certificate.

#### Scenario: FSSAI licence number is displayed
- **WHEN** a visitor navigates to the about/info page
- **THEN** the FSSAI licence number is visible as text on the page

#### Scenario: FSSAI certificate is accessible
- **WHEN** a visitor clicks the FSSAI certificate link
- **THEN** the registration certificate PDF opens or downloads in the browser
