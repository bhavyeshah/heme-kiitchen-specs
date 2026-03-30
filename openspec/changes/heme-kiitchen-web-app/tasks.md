## 1. Project Setup

- [ ] 1.1 Initialise Next.js (App Router) project with TypeScript
- [ ] 1.2 Initialise Express API project with TypeScript
- [ ] 1.3 Set up monorepo structure (apps/web, apps/api, shared types package)
- [ ] 1.4 Configure environment variables (.env.example) for admin secret, API base URL, UPI ID
- [ ] 1.5 Add `/public/images/products/` and `/public/images/logo.*` placeholder structure
- [ ] 1.6 Create `data/` directory with empty seed files: `products.json`, `orders.json`, `inventory.json`

## 2. Product Catalog — Data Layer

- [ ] 2.1 Define `Product` TypeScript type (id, name, description, price, category, image_path, status, is_addon)
- [ ] 2.2 Implement `ProductRepository` with JSON file read/write and atomic save
- [ ] 2.3 Add GET /api/products endpoint (returns active products; supports include_inactive for admin)
- [ ] 2.4 Add POST /api/products endpoint (admin-only, creates new product)
- [ ] 2.5 Add PATCH /api/products/:id endpoint (admin-only, update status / details)
- [ ] 2.6 Seed `products.json` with initial dip catalogue and add-ons (falafel, lavash)

## 3. Inventory — Data Layer

- [ ] 3.1 Define `InventoryEntry` TypeScript type (product_id, quantity, low_stock_threshold)
- [ ] 3.2 Implement `InventoryRepository` with JSON file read/write
- [ ] 3.3 Add GET /api/inventory endpoint (admin-only)
- [ ] 3.4 Add PATCH /api/inventory/:product_id endpoint (admin-only, set quantity)
- [ ] 3.5 Seed initial inventory entries alongside product seed

## 4. Order Management — Data Layer

- [ ] 4.1 Define `Order` TypeScript type (all fields per order-management spec)
- [ ] 4.2 Implement `OrderRepository` with JSON file read/write, atomic append, and write lock
- [ ] 4.3 Add POST /api/orders endpoint (creates order, validates items exist and are active)
- [ ] 4.4 Add GET /api/orders endpoint with filters: source, status, payment_status
- [ ] 4.5 Add GET /api/orders/:id endpoint
- [ ] 4.6 Add PATCH /api/orders/:id endpoint (update status and/or payment_status, admin-only)

## 5. Admin Authentication — API & Middleware

- [ ] 5.1 Implement session-based auth middleware in Express (reads ADMIN_SECRET from env)
- [ ] 5.2 Add POST /api/auth/login endpoint
- [ ] 5.3 Add POST /api/auth/logout endpoint
- [ ] 5.4 Protect all admin-only API routes with auth middleware

## 6. Customer Storefront — UI

- [ ] 6.1 Build shared layout (header with logo + nav links, footer with FSSAI licence number)
- [ ] 6.2 Build Homepage: hero section with brand tagline, highlights (Jain-friendly, no preservatives), CTA buttons
- [ ] 6.3 Build Products page: grid of product cards (image, name, description, price, out-of-stock state)
- [ ] 6.4 Build About/Info page: brand story, FSSAI licence number, downloadable FSSAI certificate link
- [ ] 6.5 Ensure all storefront pages pass mobile responsiveness check (375px viewport, no horizontal overflow)

## 7. Platter Builder — UI

- [ ] 7.1 Build platter builder page layout (dips section + add-ons section + summary panel)
- [ ] 7.2 Implement dip selection toggle with live summary updates
- [ ] 7.3 Implement add-on selection with live summary updates
- [ ] 7.4 Implement "Proceed to Checkout" button with empty-selection validation
- [ ] 7.5 Pass selected items to checkout page via URL params or session storage

## 8. Checkout — UI

- [ ] 8.1 Build checkout page with order summary (items from platter builder)
- [ ] 8.2 Build customer details form (name, phone, delivery address — all required)
- [ ] 8.3 Implement payment method selector: COD vs UPI (UPI shows business UPI ID + transaction ref field)
- [ ] 8.4 Implement form validation with inline error messages
- [ ] 8.5 Wire form submission to POST /api/orders; disable submit button during request
- [ ] 8.6 Build order confirmation page (order ref number, summary, WhatsApp follow-up note)

## 9. Admin Panel — Orders

- [ ] 9.1 Build admin login page (password form, error state)
- [ ] 9.2 Implement client-side session guard (redirect to login if unauthenticated)
- [ ] 9.3 Build admin order list page (all orders, sortable, filterable by status/source/payment)
- [ ] 9.4 Build order detail view (all fields, status update selector, "Mark as Paid" action)
- [ ] 9.5 Build create-offline-order form (customer details + item picker from product catalog)
- [ ] 9.6 Verify all admin pages pass mobile layout check (375px viewport, 44px tap targets)

## 10. Admin Panel — Products & Inventory

- [ ] 10.1 Build admin products list page (all products including inactive, with status badge)
- [ ] 10.2 Build add-product form (name, description, price, category, image upload, is_addon toggle)
- [ ] 10.3 Build product edit/deactivate actions from product list
- [ ] 10.4 Build admin inventory page (product list with stock quantities, low-stock highlight)
- [ ] 10.5 Implement inline stock quantity edit with save action

## 11. Integration & Polish

- [ ] 11.1 Wire storefront product images to actual dip photos (add files to /public/images/products/)
- [ ] 11.2 Add the brand logo to /public/images/ and update layout header
- [ ] 11.3 Add FSSAI certificate PDF to /public/ and link from about page
- [ ] 11.4 Test full online order flow end-to-end (platter builder → checkout → confirmation → admin view)
- [ ] 11.5 Test offline order creation flow in admin panel
- [ ] 11.6 Test payment status update flow in admin panel
- [ ] 11.7 Verify out-of-stock products are blocked in platter builder
- [ ] 11.8 Write a short README with local dev setup instructions and asset update guide
