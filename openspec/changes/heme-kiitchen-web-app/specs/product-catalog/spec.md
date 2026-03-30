## ADDED Requirements

### Requirement: Products are defined in a data file
The system SHALL store all product definitions in a JSON data file. Each product SHALL have: id, name, description, price, category, image_path, status (active/inactive), and is_addon flag.

#### Scenario: Active products are returned by API
- **WHEN** GET /api/products is called with no filters
- **THEN** only products with status = "active" are returned

#### Scenario: All products including inactive are returned with admin filter
- **WHEN** GET /api/products?include_inactive=true is called by an authenticated admin
- **THEN** all products including inactive ones are returned

### Requirement: Product images are managed via file replacement
The system SHALL load product images from a predictable path under /public/images/products/. Updating a product image SHALL require only replacing the image file at the stored path — no code or data changes needed.

#### Scenario: Product image renders from configured path
- **WHEN** a product card is rendered on the storefront
- **THEN** the image is served from the path stored in the product's image_path field

#### Scenario: New image replaces old without code change
- **WHEN** an image file at a product's image_path is replaced on disk
- **THEN** the updated image is served on the next page load without any application code changes

### Requirement: Admin can add new products
The system SHALL provide a form in the admin panel to add a new product, including uploading an image.

#### Scenario: Admin adds a new dip
- **WHEN** an admin fills in product details and uploads an image via the admin add-product form
- **THEN** a new product entry is saved to the data file and the product appears on the storefront immediately

#### Scenario: Product name is required
- **WHEN** an admin submits the add-product form without a product name
- **THEN** the system shows a validation error and does not save the product

### Requirement: Admin can deactivate products
The system SHALL allow an admin to set a product's status to "inactive" so it no longer appears on the storefront without deleting the record.

#### Scenario: Admin deactivates a product
- **WHEN** an admin sets a product's status to "inactive"
- **THEN** the product no longer appears on the customer storefront or platter builder

#### Scenario: Inactive product remains in admin list
- **WHEN** an admin views the products list in the admin panel
- **THEN** inactive products are shown with a visual indicator (e.g., greyed out or a badge)
