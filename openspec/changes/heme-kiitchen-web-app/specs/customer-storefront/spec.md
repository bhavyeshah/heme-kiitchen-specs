## ADDED Requirements

### Requirement: Brand logo is displayed on every page
The system SHALL display the Hémé Kiitchen logo in the shared header across all customer-facing pages — homepage, products, cart, checkout, order confirmation, and about/info. The logo SHALL be served from `/public/images/logo.*` and SHALL link back to the homepage. The header is part of the shared layout and does not need to be individually specified per page.

#### Scenario: Logo appears on every storefront page
- **WHEN** a visitor navigates to any page on the storefront
- **THEN** the Hémé Kiitchen logo is visible in the header

#### Scenario: Logo links to homepage
- **WHEN** a visitor clicks the logo from any page
- **THEN** they are taken to the homepage

### Requirement: Instagram handle is displayed in the footer on every page
The system SHALL display a link to the brand's Instagram profile in the shared footer across all customer-facing pages. The Instagram handle SHALL be stored in `data/site-content.json` as `instagram_handle` and SHALL be configurable via the admin branding panel without any code change or redeployment.

The footer SHALL only render the Instagram link when `instagram_handle` is non-empty. At launch, the field is blank and no link is shown. Once the admin sets the handle, the link appears on all pages immediately on next load.

The link SHALL open `https://www.instagram.com/[handle]` and SHALL be labelled with the handle (e.g. `@hemekiitchen`).

#### Scenario: Instagram handle is set and appears in footer
- **WHEN** an admin has saved a non-empty instagram_handle via the branding panel
- **THEN** the footer on every storefront page displays the Instagram link as `@[handle]` linking to the Instagram profile

#### Scenario: Instagram handle is blank at launch
- **WHEN** instagram_handle is null or empty in site-content.json
- **THEN** no Instagram link is rendered in the footer

#### Scenario: Admin updates Instagram handle and it reflects in footer
- **WHEN** an admin saves a new instagram_handle via the branding panel
- **THEN** the footer reflects the updated handle on next page load across all pages

### Requirement: Homepage displays brand identity
The system SHALL render a homepage that communicates Hémé Kiitchen's brand identity. The page SHALL display the brand logo, a hero section with the brand tagline, the brand description, brand highlights, and navigation to the product listing and platter builder.

All text content — tagline, brand description, and highlights — SHALL be driven from `data/site-content.json`, editable by the admin via the admin panel without any code changes. The logo is the only static asset managed via file replacement (rarely changes; a business-level decision).

#### Scenario: Visitor lands on homepage
- **WHEN** a visitor navigates to the root URL
- **THEN** the page displays the brand logo, the current tagline, the current brand description, the current list of highlights, and an "Order Now" button linking to the products page

#### Scenario: Admin updates tagline and it reflects on homepage
- **WHEN** an admin updates the tagline via the admin panel
- **THEN** the homepage shows the updated tagline on next load without any code change

#### Scenario: Admin updates brand description and it reflects on homepage
- **WHEN** an admin updates the brand description via the admin panel
- **THEN** the homepage shows the updated description on next load

#### Scenario: Admin updates highlights and they reflect on homepage
- **WHEN** an admin adds, edits, or removes a highlight via the admin panel
- **THEN** the homepage reflects the updated highlights list on next load

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
The system SHALL include an information or about page that displays the FSSAI licence number and renders the registration certificate inline for viewing. The certificate SHALL be viewable but SHALL NOT offer a download option.

The certificate SHALL be rendered using PDF.js (`pdfjs-dist` npm package, self-hosted) so that behaviour is consistent across all modern browsers (Chrome, Firefox, Safari, Edge) without relying on each browser's native PDF viewer, which varies in behaviour and always exposes a download toolbar. No download button or link to the raw PDF file SHALL be present on the page.

#### Scenario: FSSAI licence number is displayed
- **WHEN** a visitor navigates to the about/info page
- **THEN** the FSSAI licence number is visible as text on the page

#### Scenario: FSSAI certificate is rendered inline for viewing
- **WHEN** a visitor navigates to the about/info page
- **THEN** the FSSAI certificate is rendered inline on the page via PDF.js with no download button visible

#### Scenario: Certificate renders consistently across browsers
- **WHEN** a visitor views the about/info page on Chrome, Firefox, Safari, or Edge
- **THEN** the certificate renders correctly in all browsers without relying on browser-native PDF plugins

### Requirement: Customer can add products to a cart and review before checkout
The system SHALL allow customers to add products from the product listing to a cart, adjust quantities, and review a full order summary on a dedicated cart page before proceeding to checkout. The cart is stored in the browser (no server-side cart required).

A cart icon in the navigation SHALL display the current item count. The cart page SHALL show each selected product with its name, image, unit price, quantity controls, and line total, along with the overall order total. From the cart page, the customer proceeds to checkout.

#### Scenario: Customer adds a product to the cart
- **WHEN** a customer clicks "Add to Cart" on a product card
- **THEN** the product is added to the cart, the cart icon count increments, and a confirmation is shown

#### Scenario: Customer adjusts quantity in the cart
- **WHEN** a customer increases or decreases the quantity of a product in the cart
- **THEN** the line total and order total update immediately

#### Scenario: Customer removes a product from the cart
- **WHEN** a customer removes a product from the cart
- **THEN** the item is removed and totals update; if the cart is now empty a message prompts the customer to browse products

#### Scenario: Customer proceeds to checkout from cart
- **WHEN** a customer clicks "Proceed to Checkout" from the cart page
- **THEN** the selected items and quantities are carried into the checkout flow

#### Scenario: Customer tries to checkout with empty cart
- **WHEN** a customer navigates to checkout with no items in the cart
- **THEN** they are redirected to the products page
