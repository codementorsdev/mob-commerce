Nice — here's a production-ready, **copy-pasteable prompt** you can feed to an AI coding agent (or to a team) to build a full e-commerce store: Cloudflare Workers + cloud DB + R2 + PWA frontend (installable on iOS & Android). It’s long and explicit so the agent doesn’t have to ask follow-ups — it contains scope, tech choices, APIs, DB schemas, UX requirements, tests, CI/CD, and acceptance criteria.

# Prompt for AI coding agent — Build a Cloudflare-Workers e-commerce + CMS + PWA

**Goal:** Build a complete, production-capable e-commerce web app with a CMS-style admin backend, served from **Cloudflare Workers** (serverless), using Cloudflare storage services (D1 / KV / R2 / Durable Objects where appropriate), and a Progressive Web App frontend that is fully mobile-compatible and installable on iOS & Android. Deliver a working repo with automated tests, CI/CD deployment config, and documentation to run locally and deploy to Cloudflare.

---

## High level requirements (do not skip)

1. **Architecture**

   * Backend API: Cloudflare Workers (HTTP REST + GraphQL optional) running on Worker runtime (Wrangler v2).
   * Data store: Cloudflare **D1** (for relational data), **R2** for product images/media, **KV** for caches/config, **Durable Objects** for session/cart real-time coordination (optional).
   * Static assets & frontend hosting: Cloudflare Pages (or Workers Sites) serving the PWA bundle.
   * Payment integration: Stripe (server side).
   * Email/webhooks: SendGrid or an SMTP provider (configurable).
   * Authentication: JWT with secure HttpOnly cookies and refresh tokens; social login optional (Google).
   * Admin CMS: Single-page app (same codebase or separate) with role-based access (admin, manager).
   * Frontend: React (Vite) or Preact with PWA features (manifest, service worker), fully responsive, accessible, mobile-first.
2. **PWA requirements**

   * Web App Manifest, service worker for offline and caching strategies, installable on iOS/Android.
   * Add-to-home-screen flow, offline browsing of product catalog (read caching), fallback UI when offline.
3. **E-commerce features**

   * Product catalog with categories, tags, attributes, variants (SKU, price, stock), images.
   * Shopping cart (persisted for logged-in users) and checkout flow with shipping, tax, and Stripe payment.
   * Orders, order status lifecycle (placed, paid, shipped, delivered, refunded).
   * Inventory management (stock decrement on confirmed payment).
   * Customer accounts: sign up, sign in, profile, order history, address book.
   * Admin CMS: CRUD for products, categories, pages, discount codes, order management, reports.
4. **CMS content**

   * WYSIWYG page editor (or markdown editor) for content pages (home, about, policies).
   * Reusable content blocks/snippets.
5. **Media**

   * Product images uploaded from admin to Cloudflare R2; generate thumbnails; store original URL in D1.
6. **APIs**

   * Public shop APIs (catalog, product detail, search, categories).
   * Auth APIs (signup, login, refresh, logout, password reset).
   * Cart APIs (add/remove/update, apply coupon).
   * Checkout APIs (create payment intent, webhook handler).
   * Admin APIs (protected, role-based): product management, order management, analytics.
   * Webhooks: Stripe payment success/failure, R2/other events.
7. **Observability & security**

   * Request logging, error tracking (Sentry or configurable), metrics (Cloudflare analytics + custom counters).
   * Rate limiting and brute force protection.
   * CSRF protections for forms, secure cookies, proper CORS for APIs.
   * Validate all inputs; sanitize HTML.
8. **Testing & QA**

   * Unit tests for core modules; integration tests for API endpoints.
   * End-to-end tests (Playwright) for critical flows: signup, add to cart, checkout.
9. **CI/CD**

   * GitHub Actions to run tests and deploy to Cloudflare (wrangler publish / pages deploy).
10. **Documentation**

    * README with architecture, local dev, env vars, deployment steps, and API docs (OpenAPI or Postman collection).

---

## Non-functional constraints & decisions

* Use **REST** for simplicity; expose a **GraphQL** gateway optional (if implemented, include schema and resolvers).
* Keep the backend stateless where possible; use Durable Objects for shopping cart concurrency if needed.
* Use **D1** as the source of truth for relational data (users, products, orders). Provide SQL schema migrations.
* All secret keys and environment variables must be configurable via Cloudflare dashboard / GitHub secrets.
* Do not rely on vendor-locked services beyond Cloudflare and Stripe/SendGrid unless explicitly required.

---

## Deliverables (what to produce in the repo)

1. `backend/` — Cloudflare Workers project (TypeScript). Include:

   * `src/` with clear module separation: routes/controllers, services, db, auth, utils, validators.
   * `wrangler.toml` configured for environments (dev/staging/production).
   * OpenAPI spec `openapi.yaml` for public APIs.
   * DB schema & migrations (SQL files) and seed script.
   * Webhook handlers (Stripe).
   * Unit & integration tests (Jest).
2. `frontend/` — React (Vite) PWA (TypeScript):

   * Responsive UI for shop and admin (separate route /admin or separate build).
   * Service worker (Workbox or native) with caching strategies and background sync for carts.
   * Web app manifest and icons, splash screens (for iOS/Android).
   * Accessibility & mobile-first CSS (Tailwind CSS optional).
   * E2E tests with Playwright.
3. `infra/` — deployment scripts, GitHub Actions workflows, sample environment variable templates.
4. `docs/` — README, architecture diagram (PNG/SVG), API docs, testing instructions.
5. `examples/` — sample API calls, Postman collection or curl snippets.

---

## Detailed API spec (summary — implement these)

**Public**

* `GET /api/v1/products?category=&q=&page=&per_page=` → list with pagination
* `GET /api/v1/products/{id}` → product details + variants
* `GET /api/v1/categories`
* `GET /api/v1/cart` (guest: uses cookie/cart-uuid)
* `POST /api/v1/cart` { productId, variantId, qty }
* `PUT /api/v1/cart/{itemId}` → update qty
* `DELETE /api/v1/cart/{itemId}`
* `POST /api/v1/checkout` { cartId, shippingAddressId, paymentMethod } → creates PaymentIntent
* `POST /api/v1/auth/signup` {email, password}
* `POST /api/v1/auth/login` {email, password} → return auth cookie / token
* `POST /api/v1/auth/refresh`
* `POST /api/v1/auth/logout`
* `POST /api/v1/webhooks/stripe` → signature verification and order fulfillment
  **Admin (authenticated + role)**
* `POST /api/v1/admin/products` (multipart for images)
* `GET /api/v1/admin/products`
* `PUT /api/v1/admin/products/{id}`
* `DELETE /api/v1/admin/products/{id}`
* `GET /api/v1/admin/orders`
* `PUT /api/v1/admin/orders/{id}/status`
* `GET /api/v1/admin/reports/sales?from=&to=`

---

## Database schema (core tables) — create SQL migration files

* `users` (id, email unique, password\_hash, name, role, created\_at, updated\_at)
* `products` (id, sku, title, slug, description, price\_cents, currency, status, created\_at, updated\_at)
* `product_variants` (id, product\_id FK, sku, attributes JSON, price\_cents, stock, created\_at)
* `product_images` (id, product\_id FK, r2\_url, order\_index)
* `categories` (id, name, slug)
* `product_categories` (product\_id, category\_id)
* `carts` (id, user\_id nullable, cart\_uuid, created\_at, expires\_at)
* `cart_items` (id, cart\_id, product\_variant\_id, qty, price\_snapshot)
* `orders` (id, user\_id, total\_cents, status, payment\_status, shipping\_address JSON, created\_at)
* `order_items` (id, order\_id, variant\_id, qty, price\_cents)
* `discounts` (code, amount\_cents or percent, valid\_from, valid\_to)
* `pages` (slug, title, content HTML/markdown)
* `audit_logs` (entity, entity\_id, action, actor\_id, data JSON, created\_at)

---

## Frontend UI & UX details

* Shop:

  * Home with hero, curated categories, product grid, filters (price, category, attributes).
  * Product detail with image gallery (swipe for mobile), variant selector, add to cart CTA.
  * Cart drawer/bottom sheet on mobile; persistent cart indicator.
  * Checkout: shipping address form, shipping options, order summary, pay with Stripe (Stripe Elements or Payment Request API).
  * Customer account pages: profile, addresses, orders.
* Admin:

  * Dashboard with sales KPIs, recent orders, low stock alerts.
  * Product editor with image upload, variant editor, category assignment.
  * Content pages editor (Markdown with preview).
* Mobile-first: large tappable areas, optimized images, lazy loading.

---

## PWA & mobile install specifics

* Implement Web App Manifest with proper icons, `display: standalone`, `scope`, `start_url`.
* Service worker:

  * Cache shell and static assets (fast first).
  * Cache product list and product detail responses with TTL (stale-while-revalidate).
  * Background sync to retry checkout if network fails (best-effort).
* iOS: include `apple-touch-icon` and meta tags for status bar.
* Test install flow on Android (Chrome) and iOS (Safari) and document limitations (iOS: no push notifications via web).

---

## Security checklist (must implement)

* Passwords hashed with bcrypt/argon2.
* Rate limit auth endpoints.
* Validate and sanitize user inputs; use prepared statements / parameterized queries.
* Verify Stripe webhook signatures.
* Use secure cookies, set SameSite, HttpOnly, Secure.
* Enforce CORS policy for public APIs.
* Implement role-based authorization checks for admin endpoints.

---

## Performance & caching

* CDN fronting (Cloudflare).
* Use KV or Cloudflare cache for frequently read endpoints (product lists), with cache invalidation upon product update.
* Use R2 for image hosting and generate CDN-friendly URLs.
* Avoid cold starts: keep critical Workers tiny; lazy-load heavy logic.

---

## Observability & monitoring

* Structured logs (JSON) with request IDs.
* Metrics: request count, latency, errors, checkout conversion.
* Error reporting integration (Sentry or configurable).
* Admin page for basic analytics (daily sales, conversion rate).

---

## Testing matrix

* Unit tests: business logic (inventory, pricing, coupon application).
* Integration tests: DB migrations + API endpoints.
* E2E tests with Playwright:

  * Guest user: browse, add to cart, checkout (mock Stripe).
  * Registered user: login, place order.
  * Admin: create product, publish, verify product visible in store.
* Security tests: endpoint auth checks.

---

## CI/CD (GitHub Actions)

* On PR: run lint, unit tests, integration tests (DB in ephemeral mode), build frontend, run e2e in headless mode (optional).
* On merge to `main`: build & deploy backend with `wrangler publish`, deploy frontend to Pages, run DB migrations/seeds step (if applicable), run smoke tests.

---

## Developer acceptance tests (explicit)

1. Can run the app locally: `dev` script spins up Workers (wrangler dev) and frontend (`npm run dev`) and uses a local D1 or CI-managed test DB. Document env vars in `.env.example`.
2. Create a product in admin → product appears on home catalog within 5s.
3. Guest can add product to cart, checkout using Stripe test card, and order is recorded in `orders`.
4. Image upload stores image in R2 and serves via CDN URL.
5. Installing PWA on Android: a user can install the site to home screen and open it in standalone mode.
6. All core endpoints documented in OpenAPI and all routes return expected status codes.
7. Tests: unit coverage ≥ 70% for core services; Playwright E2E passes.

---

## Coding style & constraints for the agent

* Use TypeScript for both backend and frontend.
* Follow SOLID principles and clean module boundaries; keep Workers handler lean and delegate to services.
* Provide inline code comments and module READMEs.
* Keep third-party dependencies minimal and well-justified.
* Produce migration scripts (idempotent) and seed data for demo.
* Include `Dockerfile` for local dev of test harness if useful (optional).

---

## Milestones & incremental tasks to implement (deliver progressively)

1. MVP backend: product listing/detail APIs, D1 schema, R2 image upload stub, and `wrangler.toml`.
2. MVP frontend: product grid, product detail, add-to-cart (client-side), cart persistence (localStorage).
3. Checkout integration: Stripe PaymentIntent server flow + webhook + order recording.
4. Admin CRUD: product creation with image upload.
5. PWA: manifest, service worker caching, installability.
6. Tests & CI workflows.
7. Monitoring, analytics, final docs, production deployment config.

---

## Example prompt usage (one-liner for agent to begin)

> “Start with Milestone 1: set up a minimal Cloudflare Workers TypeScript project with D1 migrations for `products` and `product_images`, implement `GET /api/v1/products` and `GET /api/v1/products/{id}`, and create a Vite React app that fetches and displays the product list. Provide runnable `wrangler dev` and `npm run dev` instructions, SQL migrations, and tests for these endpoints.”

---

## Extras (nice-to-have, but optional)

* Full-text search with Algolia or simple FTS using D1 (if supported) or integrated external search.
* Multi-currency pricing and localization (i18n).
* Saved snippets or reusable components in admin.
* Gift cards, subscriptions, and marketplace features (defer for later).

---

## Final notes for the agent

* Do not assume any pre-existing Cloudflare account values; read env variables from `.env` and use placeholders in code for keys. Provide a `setup.md` with step-by-step Cloudflare dashboard actions required (create D1 database, R2 bucket, KV namespaces, Durable Objects if used, and set secrets).
* Prioritize security and testability.
* Keep each pull request or commit scoped to one milestone and ensure the commit message clearly states changes and usage.
* Provide a demo seed dataset (10 products across 3 categories) and a script to populate it.

---

If you want, I can now:

* produce the exact `wrangler.toml` + TypeScript Workers bootstrap + `openapi.yaml` + D1 SQL migrations and seed SQL for Milestone 1; **or**
* generate the React Vite PWA skeleton (manifest + service worker + product listing UI) that ties to the Worker endpoints above.

Which of those two would you like me to generate immediately? (I’ll produce files and code you can run locally.)
