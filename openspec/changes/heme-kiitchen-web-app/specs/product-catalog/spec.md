## ADDED Requirements

### Requirement: Products are defined in a data file
The system SHALL store all product definitions in a JSON data file. Each product SHALL have: id, name, description, price, image_path, and status (active/inactive).

The catalogue is intentionally flat — there is no category or classification field. All products (dips, falafel, lavash, platters) are equal entries. Product grouping and classification is deferred to a future release, at which point the admin will be able to define categories and assign products to them.

#### Scenario: Active products are returned by API
- **WHEN** GET /api/products is called with no filters
- **THEN** only products with status = "active" are returned

#### Scenario: All products including inactive are returned with admin filter
- **WHEN** GET /api/products?include_inactive=true is called by an authenticated admin
- **THEN** all products including inactive ones are returned

### Requirement: Product images are uploaded to Cloudinary via the admin panel
The system SHALL allow admins to upload product images directly through the admin panel — no server file access required. The API SHALL forward the uploaded file to Cloudinary, which compresses and stores it, and return a CDN URL. That URL SHALL be stored as `image_path` in the product record and used to render images on the storefront.

When a product's image is replaced, the old Cloudinary asset SHALL be deleted to avoid accumulating unused storage.

#### Scenario: Product image renders from Cloudinary URL
- **WHEN** a product card is rendered on the storefront
- **THEN** the image is served from the Cloudinary CDN URL stored in the product's image_path field

#### Scenario: Uploaded image is compressed by Cloudinary
- **WHEN** an admin uploads a product image
- **THEN** Cloudinary compresses and optimises the image automatically; the admin does not need to pre-process the photo

#### Scenario: Old Cloudinary asset is deleted when photo is replaced
- **WHEN** an admin uploads a new photo for an existing product
- **THEN** the previous Cloudinary asset is deleted and image_path is updated to the new Cloudinary URL

#### Scenario: Upload rejected if file is not an image
- **WHEN** an admin uploads a non-image file (e.g., PDF, video)
- **THEN** the system rejects the upload with a validation error and does not save the product

### Requirement: Admin can add new products
The system SHALL provide a form in the admin panel to add a new product. A photo upload is mandatory — a product cannot be saved without an image, ensuring the catalogue always displays correctly on the storefront.

#### Scenario: Admin adds a new dip
- **WHEN** an admin fills in product details, uploads a photo, and submits the add-product form
- **THEN** the image is compressed and stored, a new product entry is saved to the data file, and the product appears on the storefront immediately

#### Scenario: Product name is required
- **WHEN** an admin submits the add-product form without a product name
- **THEN** the system shows a validation error and does not save the product

#### Scenario: Photo is required on product creation
- **WHEN** an admin submits the add-product form without uploading a photo
- **THEN** the system shows a validation error and does not save the product

### Requirement: Admin can edit existing product details
The system SHALL allow an admin to update a product's name, description, price, and photo from the admin panel. The edit form SHALL display the current photo so the admin can see what is already set before deciding to replace it. Photo replacement is optional during an edit — the existing photo is kept if no new file is uploaded. Edits SHALL take effect immediately — the storefront and platter builder reflect the updated values on the next page load. Product records are NEVER hard-deleted; removal from the catalogue is handled exclusively via deactivation.

#### Scenario: Admin updates a product price
- **WHEN** an admin edits a product's price and saves
- **THEN** the product record is updated and the new price is shown on the storefront and platter builder on next load

#### Scenario: Admin updates a product name or description
- **WHEN** an admin edits a product's name or description and saves
- **THEN** the product record is updated and the changes are visible to customers on next load

#### Scenario: Admin submits edit with empty name
- **WHEN** an admin clears the product name and saves
- **THEN** the system shows a validation error and does not save the change

#### Scenario: Admin submits edit with invalid price
- **WHEN** an admin enters a non-positive or non-numeric price and saves
- **THEN** the system shows a validation error and does not save the change

#### Scenario: Admin updates product photo
- **WHEN** an admin uploads a new photo on the edit form and saves
- **THEN** the new image is compressed, stored, image_path is updated, and the old image file is deleted from disk

#### Scenario: Admin edits product without changing photo
- **WHEN** an admin updates name, description, or price without selecting a new photo
- **THEN** the existing image is retained unchanged

### Requirement: Admin can deactivate and reactivate products
The system SHALL allow an admin to set a product's status to "inactive" so it no longer appears on the storefront, and to reactivate it when it becomes available again. Product records are never deleted — deactivation is always reversible. This supports seasonal products that cycle on and off the menu.

#### Scenario: Admin deactivates a product
- **WHEN** an admin sets a product's status to "inactive"
- **THEN** the product no longer appears on the customer storefront or platter builder

#### Scenario: Admin reactivates a product
- **WHEN** an admin sets an inactive product's status back to "active"
- **THEN** the product reappears on the customer storefront and platter builder immediately

#### Scenario: Inactive product remains in admin list
- **WHEN** an admin views the products list in the admin panel
- **THEN** inactive products are shown with a visual indicator (e.g., greyed out or a badge)
