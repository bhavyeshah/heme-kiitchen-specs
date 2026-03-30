## 1. Project Setup

- [ ] 1.1 Initialise Next.js (App Router) project with TypeScript
- [ ] 1.2 Initialise Express API project with TypeScript
- [ ] 1.3 Set up monorepo structure (apps/web, apps/api, shared types package)
- [ ] 1.4 Configure environment variables (.env.example) for admin secret, API base URL, UPI ID, business WhatsApp number
- [ ] 1.5 Add `/public/images/products/` and `/public/images/logo.*` placeholder structure
- [ ] 1.6 Create `data/` directory with empty seed files: `products.json`, `orders.json`

## 2. Product Catalog — Data Layer

- [ ] 2.1 Define `Product` TypeScript type (id, name, description, price, category, image_path, status, is_addon)
- [ ] 2.2 Implement `ProductRepository` with JSON file read/write and atomic save
- [ ] 2.3 Add GET /api/products endpoint (returns active products; supports include_inactive for admin)
- [ ] 2.4 Add POST /api/products endpoint (admin-only, creates new product)
- [ ] 2.5 Add PATCH /api/products/:id endpoint (admin-only, update status / details)
- [ ] 2.6 Seed `products.json` with initial dip catalogue and add-ons (falafel, lavash)

<!-- Section 3 (Inventory Data Layer) removed — inventory is deferred, managed manually for now -->

## 4. Order Management — Data Layer

- [ ] 4.1 Define `Order` TypeScript type — fields: id, source, status, items, total_price, customer name, phone, delivery_type, delivery_address (nullable), delivery_charges_applicable, payment_method, payment_status, upi_reference (nullable), deleted, deleted_at (nullable), created_at, updated_at
- [ ] 4.2 Implement `OrderRepository` with JSON file read/write, atomic append, and write lock
- [ ] 4.3 Add POST /api/orders endpoint (creates order, validates items exist and are active, enforces payment method per delivery type)
- [ ] 4.4 Add GET /api/orders endpoint with filters: source, status, payment_status, delivery_type, deleted
- [ ] 4.5 Add GET /api/orders/:id endpoint
- [ ] 4.6 Add PATCH /api/orders/:id endpoint — handles: accept (pending → confirmed), decline (pending → declined), status update, payment_status update, full field edit; enforce terminal state rules
- [ ] 4.7 Add DELETE /api/orders/:id endpoint (soft delete — sets deleted=true, deleted_at=now)

## 5. Admin Authentication — API & Middleware

- [ ] 5.1 Implement session-based auth middleware in Express (reads ADMIN_SECRET from env)
- [ ] 5.2 Add POST /api/auth/login endpoint
- [ ] 5.3 Add POST /api/auth/logout endpoint
- [ ] 5.4 Protect all admin-only API routes with auth middleware

## 6. Customer Storefront — UI

- [ ] 6.1 Build shared layout (header with logo + nav links, footer with FSSAI licence number)
- [ ] 6.2 Build Homepage: hero section with brand tagline, highlights (Jain-friendly, no preservatives), CTA buttons
- [ ] 6.3 Build Products page: grid of product cards (image, name, description, price)
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
- [ ] 8.2 Build delivery type selector (Pickup / Home Delivery); show conditional address field for home delivery; show pickup location for pickup; show live delivery charge notice (free above ₹1500, charge notice at or below)
- [ ] 8.3 Build payment method selector — COD + UPI shown for pickup; UPI only for home delivery; switching delivery type to home delivery auto-selects UPI and hides COD; UPI shows business UPI ID + transaction reference field
- [ ] 8.4 Build customer details form (name + phone always required; address required for home delivery only)
- [ ] 8.5 Implement form validation with inline error messages
- [ ] 8.6 Wire form submission to POST /api/orders; disable submit button during request
- [ ] 8.7 Build order confirmation page — pickup: order ref + summary + "We will confirm via WhatsApp" note; home delivery: order ref + summary + delivery charge notice + WhatsApp payment screenshot instruction with business number

## 9. Admin Panel — Orders

- [ ] 9.1 Build admin login page (password form, error state)
- [ ] 9.2 Implement client-side session guard (redirect to login if unauthenticated)
- [ ] 9.3 Build admin order list page (all orders, filterable by status / source / payment_status / delivery_type; excludes deleted by default)
- [ ] 9.4 Build order detail view — show all fields; Accept + Decline actions for pending orders only; status update selector for confirmed+ orders; Cancel action for confirmed/preparing/dispatched; "Mark as Paid" action; Edit order fields (items, customer details, delivery type, address, payment method) for non-terminal orders
- [ ] 9.5 Build create-offline-order form (customer details, delivery type selector, item picker from product catalog, payment method conditional on delivery type)
- [ ] 9.6 Verify all admin pages pass mobile layout check (375px viewport, 44px tap targets)

## 10. Admin Panel — Products & Inventory

- [ ] 10.1 Build admin products list page (all products including inactive, with status badge)
- [ ] 10.2 Build add-product form (name, description, price, category, image upload, is_addon toggle)
- [ ] 10.3 Build product edit/deactivate actions from product list
<!-- 10.4–10.5 (Admin inventory page) removed — deferred until inventory tracking is plugged in -->

## 11. Integration & Polish

- [ ] 11.1 Wire storefront product images to actual dip photos (add files to /public/images/products/)
- [ ] 11.2 Add the brand logo to /public/images/ and update layout header
- [ ] 11.3 Add FSSAI certificate PDF to /public/ and link from about page
- [ ] 11.4 Test full online order flow — pickup with COD (platter builder → checkout → confirmation → admin accept → complete)
- [ ] 11.5 Test full online order flow — home delivery with UPI (checkout → WhatsApp payment instruction → admin receives screenshot → mark paid → accept → dispatch → complete)
- [ ] 11.6 Test admin decline flow on a pending order
- [ ] 11.7 Test offline order creation flow in admin panel (pickup + home delivery)
- [ ] 11.8 Test order edit and soft-delete from admin panel
- [ ] 11.9 Write a short README with local dev setup instructions and asset update guide
