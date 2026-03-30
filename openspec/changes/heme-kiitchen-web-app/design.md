## Context

Hémé Kiitchen is a small premium food brand (Jain-friendly dips) currently operating entirely through offline channels. This is a greenfield build with no existing system to migrate from. The business is run by a small team managing orders from a phone, so the admin interface must be fast and usable on mobile. The customer website needs to convey premium brand quality and Jain-friendly credentials clearly.

Stakeholders: business owner (primary admin user), end customers (online buyers).

## Goals / Non-Goals

**Goals:**
- Single backend API that unifies online and offline orders
- Customer website: brand homepage, product listing, platter builder, checkout
- Admin panel optimised for phone use: create offline orders, manage all orders, update payment, track inventory
- JSON file storage that can be swapped for a database without API contract changes
- Static asset management for product images and logo that requires no code change to update

**Non-Goals:**
- Automated payment gateway integration (manual UPI / COD only)
- Multi-user admin with role-based access (single owner for now)
- Real-time notifications or live order tracking
- Native mobile app
- Complex analytics or reporting

## Decisions

### 1. Tech Stack: Next.js (App Router) + Node/Express API

**Decision**: Next.js for the customer website and admin panel; a separate lightweight Express API for the backend.

**Rationale**: Next.js gives server-side rendering for good SEO on the customer site, and the App Router supports mobile-responsive layouts cleanly. A separate Express API keeps the data layer decoupled from the UI so the frontend can evolve independently. Chosen over a monorepo full-stack framework (like Remix or SvelteKit) because Express is simpler to reason about for JSON file storage and future DB swap.

**Alternatives considered**: Pure React SPA + Express — rejected because SSR matters for the product listing pages (brand discoverability). Next.js API routes only — rejected because a standalone API makes it easier to replace storage later without touching the frontend.

### 2. Storage: JSON Files with Repository Pattern

**Decision**: Store orders, products, and inventory as JSON files on disk, accessed through a repository interface (e.g., `OrderRepository`, `ProductRepository`).

**Rationale**: Zero infrastructure needed to get started. The repository pattern means swapping to SQLite, Postgres, or a hosted DB later requires only replacing the repository implementation, not changing API controllers or business logic.

**Alternatives considered**: SQLite from day one — overhead not justified for the initial launch volume. A hosted DB (Firestore, PlanetScale) — adds cost and complexity before product-market fit is confirmed.

### 3. Admin Panel: Same Next.js App, Separate Route Group

**Decision**: Admin panel lives at `/admin/*` within the same Next.js application, protected by a simple session secret (env var), not a separate app.

**Rationale**: Reduces deployment complexity (one app, one deploy). Mobile-first CSS is applied per route group. Single secret is sufficient for a sole-owner business.

**Alternatives considered**: Separate admin app — unnecessary operational overhead for one user.

### 4. Static Assets: `/public/images/` Directory Convention

**Decision**: Product images and logo live in `/public/images/products/<slug>.jpg` and `/public/images/logo.*`. Image paths are stored as relative strings in the product JSON. Updating an image = replacing the file; no code change needed.

**Rationale**: Simple, understandable for a non-technical owner. No CDN needed at this scale.

### 5. Product Catalog: JSON File, Admin-Editable via Panel

**Decision**: Products defined in `data/products.json`. New products are added through an admin form (no manual JSON editing required by the owner).

**Rationale**: Keeps product management accessible without technical knowledge.

## Risks / Trade-offs

- **JSON file concurrency** → Mitigation: Use write locks (or atomic rename) for order writes; acceptable at low order volume.
- **No payment automation** → Mitigation: Manual UPI reference captured at checkout; admin marks paid manually. Document the expected flow clearly.
- **Single admin secret** → Mitigation: Env var, not hardcoded. Rotate by redeploying. Acceptable for sole-owner use.
- **Image hosting on same server** → Mitigation: Fine at low traffic; CDN can be added later by pointing image paths to an object store URL.

## Migration Plan

Greenfield — no migration needed. Deployment steps:
1. Deploy Next.js app + Express API on a VPS or platform (e.g., Railway, Render, or Vercel + Railway).
2. Seed `data/products.json` with initial product list and images.
3. Set admin secret env var.
4. DNS cutover to new domain.

Rollback: Re-point DNS; JSON data files can be downloaded before any change.

## Open Questions

- Preferred deployment platform? (Affects environment variable and static asset strategy)
- Should the FSSAI certificate PDF be downloadable directly from the site, or just display the licence number?
- Are there any dip categories / groupings needed on the product listing, or is a flat list sufficient for launch?
