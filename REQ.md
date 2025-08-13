Prompt for AI Coding Agent
Objective:
Develop a super professional, high-performance Progressive Web App (PWA) for mobile-first e-commerce that works seamlessly on Android and iOS, complete with a fully featured Content Management System (CMS) similar to WooCommerce but entirely headless and serverless, powered by Netlify Functions and a cloud database.
Technical Requirements
1. Frontend (PWA)
Framework: React + Next.js (with Static Site Generation + Incremental Static Regeneration for performance).
Styling: Tailwind CSS + Headless UI (with custom theming and responsive components).
PWA Features:
Installable app on Android & iOS with custom splash screen & icons.
Offline support using Workbox + IndexedDB caching for products & cart.
Background sync for cart & orders when offline.
UI/UX Goals:
Mobile-first design with native-like animations (Framer Motion).
Accessible design (WCAG 2.1 AA).
Dark/light mode toggle.
Core Pages:
Home with hero banner, category navigation, product highlights.
Category listing with infinite scroll & filters.
Product detail with image carousel, zoom, reviews, related products.
Cart with real-time updates.
Checkout with guest & logged-in flows.
Order confirmation & tracking.
User profile, order history, wishlists.
Admin login (for CMS control panel link).
Search & Recommendations:
Search-as-you-type with serverless search endpoint (Algolia or custom).
AI-driven product recommendations (optional).
2. Backend (Headless CMS & APIs)
Platform: Netlify Functions (Node.js 18+ runtime).
Database:
Primary: FaunaDB, Supabase, or MongoDB Atlas (serverless-friendly).
Product search indexing: Algolia or Elasticlunr.js.
Features:
Product management (CRUD with categories, tags, inventory).
Order management (create, update, track status).
User authentication (JWT or Auth0/Clerk).
Payment integration (Stripe for payments, Razorpay for India).
Shipping integration (Shiprocket API or custom shipping rules).
Coupon & discount system.
Inventory sync with stock alerts.
Webhooks for payment confirmation & order status updates.
Multi-language & multi-currency support.
API Security:
Role-based access (admin, staff, customer).
API rate limiting & JWT validation middleware.
Secure environment variable handling (Netlify environment).
3. CMS Interface
Framework: Next.js + Tailwind Admin dashboard.
Features:
Product CRUD with image uploads (Cloudinary/S3).
Category & tag management.
Order tracking & status updates.
Customer database & analytics.
Discounts & promotional campaigns.
Stock management & low-stock alerts.
Content editor for homepage banners & featured sections.
Role-based user management.
UI/UX Goals for CMS:
Professional, clean UI with responsive layout.
Charts for sales, orders, traffic (Chart.js or Recharts).
Quick actions (add product, process order).
Searchable tables with pagination.
Bulk upload via CSV/Excel.
4. Additional Features
SEO Optimized:
Meta tags, OpenGraph, structured data for products.
Sitemap generation via Netlify build plugins.
Analytics:
Google Analytics 4 / Plausible / PostHog.
Notifications:
Email confirmations via SendGrid/Mailgun.
Push notifications via OneSignal.
Automation:
Daily backup of product & order data.
Serverless cron jobs for cleanup tasks (Netlify Scheduled Functions).
5. Deployment
Hosting: Netlify (frontend + functions).
CDN: Netlify Edge network for global delivery.
Image Optimization: Next.js Image + Cloudinary.
CI/CD: GitHub Actions or Netlify build hooks.
6. Deliverables
Fully functional mobile PWA with installable capabilities.
Complete headless CMS with admin dashboard.
Documentation:
Developer setup guide.
API reference.
CMS usage guide.
Automated Tests:
Jest + React Testing Library for frontend.
Supertest for API endpoints.
Development Priorities:
Performance first (Lighthouse score > 95 for PWA metrics).
Pixel-perfect UI with smooth animations.
Scalable backend that can handle thousands of products.
Security best practices for API and auth.
