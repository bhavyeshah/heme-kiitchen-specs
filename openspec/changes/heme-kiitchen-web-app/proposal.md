## Why

Hémé Kiitchen is a premium Jain-friendly dip brand that currently takes orders offline (WhatsApp, calls) with no unified system to manage them. Building a web presence with an integrated order management system will enable online sales, reduce manual coordination, and give the business a single place to track all orders regardless of origin.

## What Changes

- Introduce a customer-facing website with brand identity, product listings, a platter builder, and online checkout (COD + manual UPI)
- Introduce a mobile-first admin panel for creating offline orders, viewing all orders, updating payment status, and tracking inventory
- Introduce a backend API layer with JSON-based storage (replaceable with a database later) to unify online and offline order flows
- Support easy image and product updates without code changes
- Surface FSSAI registration number and certificate in the site's information section

## Capabilities

### New Capabilities

- `customer-storefront`: Homepage with brand identity, product listing with individual dip images and descriptions, and informational pages (about, FSSAI details)
- `platter-builder`: Interactive builder allowing customers to select dips and add-ons (falafel, lavash) to compose a custom platter order
- `checkout`: Order placement flow supporting Cash on Delivery and manual UPI payment; collects customer details and generates an order
- `order-management`: Unified order record covering both online (from checkout) and offline (admin-created) orders; supports status and payment updates
- `admin-panel`: Mobile-first admin interface to create offline orders, view the full order list, update payment status, and access inventory
- `product-catalog`: Data-driven product definitions (name, description, image path, price, category) making it easy to add products or swap images without touching code
- `inventory-tracking`: Simple stock-level tracking per product, viewable and editable from the admin panel

### Modified Capabilities

<!-- None — this is a greenfield system -->

## Impact

- New frontend application (customer website + admin panel)
- New backend API service with JSON file storage
- Static assets (product images, logo, FSSAI PDF) managed under a versioned assets directory
- No existing systems affected; greenfield build
