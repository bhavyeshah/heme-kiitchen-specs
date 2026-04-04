## Context

Hémé Kiitchen is a small premium food brand (Jain-friendly dips) currently operating entirely through offline channels. This is a greenfield build with no existing system to migrate from. The business is run by a small team managing orders from a phone, so the admin interface must be fast and usable on mobile. The customer website needs to convey premium brand quality and Jain-friendly credentials clearly.

Stakeholders: business owner (primary admin user), end customers (online buyers).

## Goals / Non-Goals

**Goals:**
- Single backend API that unifies online and offline orders
- Customer website: brand homepage, product listing, platter builder, checkout
- Admin panel optimised for phone use: create offline orders, manage all orders, update payment, track inventory
- JSON file storage that can be swapped for a database without API contract changes
- Product image storage via Cloudinary (free tier) — no persistent disk required on the server

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

**Rationale**: Zero infrastructure needed to get started. The repository pattern means swapping to SQLite, Postgres, or a hosted DB later requires only replacing the repository implementation — API controllers, business logic, and the frontend remain untouched.

**Alternatives considered**: SQLite from day one — overhead not justified for the initial launch volume. A hosted DB (Firestore, PlanetScale) — adds cost and complexity before product-market fit is confirmed.

**Scale path**: Replace `OrderRepository` and `ProductRepository` implementations with a DB-backed version (e.g. PostgreSQL via Prisma). No changes to API routes or business logic. Recommended DB when scaling: PostgreSQL on AWS RDS or Supabase.

### 3. Admin Panel: Same Next.js App, Separate Route Group

**Decision**: Admin panel lives at `/admin/*` within the same Next.js application, protected by a simple session secret (env var), not a separate app.

**Rationale**: Reduces deployment complexity (one app, one deploy). Mobile-first CSS is applied per route group. Single secret is sufficient for a sole-owner business.

**Alternatives considered**: Separate admin app — unnecessary operational overhead for one user.

### 4. Image Storage: Cloudinary (free tier)

**Decision**: Product images are uploaded to Cloudinary via the admin panel. The API receives the image file, forwards it to Cloudinary, which compresses and stores it, and returns a URL. That URL is stored as `image_path` in the product record. When a product photo is replaced, the old Cloudinary asset is deleted. The logo is managed as a static file in `/public/images/` (one-time setup, no upload flow needed).

**Rationale**: Cloudinary's free tier (25GB storage, 25GB bandwidth/month) is sufficient for a dip catalogue. It handles compression automatically, serves images over a CDN, and removes the need for persistent disk on the server — which in turn enables free-tier hosting on Render. Google Drive was considered but rejected: it is not a CDN, has rate limits on direct image serving, and causes CORS issues on websites.

**Alternatives considered**: Local `/public/images/` directory — requires persistent disk, which is not available on Render's free tier. Google Drive — unreliable for serving images on a website (rate limits, no CDN, CORS issues).

### 5. Hosting: Render (free tier)

**Decision**: Deploy both the Next.js app and the Express API on Render's free tier as two separate web services. Render is chosen over other platforms because it offers always-on-capable free web services (cold starts acceptable at low traffic), easy env var management via dashboard, and no persistent disk requirement once images are on Cloudinary.

**Rationale**: Free, no credit card required for free tier, straightforward dashboard for env var updates (including `ADMIN_SECRET`). Cold starts (~30s after inactivity) are acceptable given that online traffic is expected to be low initially. The production domain is **www.hemekiitchen.com** — Render supports custom domains on the free tier. The demo and production share the same deployment; WhatsApp notifications are inactive for the demo and enabled post-demo once Meta approval is complete.

**Alternatives considered**: Railway — $5/month credit, not permanently free. Fly.io — free but CLI-heavy setup. Vercel — serverless, no persistent filesystem, cannot run Express as-is.

### 6. WhatsApp Notifications: Meta WhatsApp Cloud API

**Decision**: Use the Meta WhatsApp Cloud API (official) to send automated status notifications to customers. Notifications are triggered server-side when an order transitions to a high-value status (confirmed, dispatched, completed, declined, cancelled). Messages use pre-approved Meta message templates. The API is called after the order record is updated — failures are logged but do not block status updates.

**Rationale**: Free up to 1000 conversations/month — sufficient for low online traffic. Official API with no ToS risk. Notification logic is the same for both online and offline orders since both capture the customer's phone number in +91 format. Template approval is a one-time process per message format.

**Alternatives considered**: Third-party BSPs (Interakt, AiSensy) — easier setup but paid beyond free tier limits. Unofficial libraries (baileys) — against WhatsApp ToS, risk of number ban.

**Setup required**: Facebook Business account verification + WhatsApp Cloud API access + message template submission and approval (takes a few days). Once approved, templates are reusable indefinitely.

### 7. Product Catalog: JSON File, Admin-Editable via Panel

**Decision**: Products defined in `data/products.json`. New products are added through an admin form (no manual JSON editing required by the owner).

**Rationale**: Keeps product management accessible without technical knowledge.

## Risks / Trade-offs

- **JSON file concurrency** → Mitigation: Use write locks (or atomic rename) for order writes; acceptable at low order volume.
- **No payment automation** → Mitigation: Manual UPI reference captured at checkout; admin marks paid manually. Document the expected flow clearly.
- **Single admin secret** → Mitigation: Env var, not hardcoded. Rotate by updating the env var in the hosting platform dashboard and restarting the app. No in-app password change flow — acceptable for sole-owner use where the admin has hosting platform access.
- **Render cold starts** → Acceptable at low initial online traffic. If response times become a problem, upgrade to a paid Render instance or migrate to Railway.
- **Cloudinary free tier limits** → 25GB storage and 25GB bandwidth/month is well in excess of what a small dip catalogue requires. Monitor usage if traffic grows significantly.

## Migration Plan

Greenfield — no migration needed. Deployment steps:
1. Create a free Cloudinary account. Note the Cloud Name, API Key, and API Secret from the Cloudinary dashboard.
2. Set up WhatsApp Cloud API: create a Facebook Business account, register a WhatsApp Business phone number, submit message templates for Meta approval (allow a few days). Note the Phone Number ID and Access Token.
3. Deploy the Express API on Render (free tier) as a web service. Set the following env vars in the Render dashboard: `ADMIN_SECRET`, `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET`, `WHATSAPP_PHONE_NUMBER_ID`, `WHATSAPP_ACCESS_TOKEN`.
4. Deploy the Next.js app on Render (free tier) as a second web service. Set `NEXT_PUBLIC_API_URL` to the Express API's Render URL.
5. Log in to the admin panel and add initial products with photos via /admin/products — no manual JSON editing required.
6. Point www.hemekiitchen.com DNS to the Render deployment (add a custom domain in the Render dashboard for the Next.js service).

To change the admin password later: update `ADMIN_SECRET` in the Render dashboard and redeploy the API service. No code change required.

Rollback: Re-point DNS; JSON data files (`orders.json`, `products.json`) can be downloaded from the Render shell before any change.

## Scaling Paths

This section documents how each architectural decision can evolve when the application needs to scale, without requiring a rewrite.

### Storage → Database
Replace the JSON file repositories with a database-backed implementation. The repository interface is the only change surface — no API or frontend changes needed.
- **Recommended**: PostgreSQL via Prisma ORM
- **Hosted options**: AWS RDS (PostgreSQL), Supabase (managed PostgreSQL, generous free tier), PlanetScale (MySQL)
- **Trigger**: When concurrent writes cause data corruption or JSON files grow unwieldy (typically >10k orders)

### Image Storage → S3 or Cloudinary paid
Cloudinary free tier (25GB) covers early growth. When it's no longer sufficient:
- **AWS S3**: Store images in an S3 bucket; update `image_path` to store S3 URLs. Requires updating the upload logic in the API only — no frontend changes.
- **Cloudinary paid**: Simply upgrade the plan; no code changes needed.

### Hosting → AWS or another platform
The Express API and Next.js app are standard Node.js services — portable to any platform.
- **AWS path**: EC2 or ECS (Fargate) for the API; Vercel or Amplify for Next.js; RDS for the database; S3 + CloudFront for images. This is the full production-grade setup.
- **Mid-tier path**: Railway or Fly.io for both services — more control than Render, still managed.
- **Trigger**: When Render free tier cold starts become unacceptable for real traffic, or when uptime SLAs are required.

### Authentication → Multi-user or OAuth
The single `ADMIN_SECRET` approach works for a sole owner. If the business grows:
- Add user records to the database with hashed passwords and role-based access
- Or integrate Google OAuth (passport.js) for admin login — the session middleware is the only change surface

### WhatsApp → Higher volume
WhatsApp Cloud API free tier covers 1000 conversations/month. Beyond that:
- Upgrade to a paid Meta plan, or
- Move to a BSP (Business Solution Provider) like Interakt or AiSensy for higher volume with template management dashboards

## Open Questions

- Should the FSSAI certificate PDF be downloadable directly from the site, or just display the licence number?
- Are there any dip categories / groupings needed on the product listing, or is a flat list sufficient for launch?
