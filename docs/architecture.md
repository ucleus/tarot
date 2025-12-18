# Ucleus Content Helper Architecture

This document summarizes the planned architecture, features, and operational practices for the Ucleus Content Helper SaaS platform.

## Overview
- **Goal:** Deliver AI-powered content optimization (keyword research, readability improvement, outline generation) for marketers.
- **Stack:** React SPA frontend, PHP REST API backend, MySQL database; optional Redis/Memcached for caching and OTP/session storage.
- **Hosting:** Decoupled frontend (static assets + CDN) and stateless backend behind a load balancer; HTTPS enforced end-to-end.

## Core Modules
- **Public Marketing Site:** Landing, features, pricing, blog/resources, and legal pages built as public React routes.
- **Authenticated App:** Dashboard, project/workspace management, AI tools, content drafts, AI history, account settings, subscription & billing views.
- **Admin Panel:** User management, AI log review/moderation, analytics, plan adjustments, and system settings (RBAC-gated).

## Frontend
- React SPA with client-side routing; mobile-first responsive design and accessibility (semantic HTML, proper contrast, keyboard support).
- Uses fetch/Axios for API calls; loading/error states on AI requests; consistent component library/design system.
- Possible features: inline AI assistance inside draft editor, usage banners when quota is near limits, Stripe Checkout/Portal entry points.

## Backend
- PHP service (framework-friendly) exposing JSON REST APIs under `/api/v1`.
- Middleware for authentication (JWT), role checks, and rate/usage enforcement per subscription plan.
- Service layers for OpenAI integration (prompt construction, moderation, error handling) and Stripe billing/webhook processing.
- Input validation/sanitization on every request; structured success/error responses.

## Data Model (relational, MySQL)
- **Users:** id, contact info, role, plan_id, verification flags, timestamps.
- **Plans:** name, pricing, limits (max queries/month), optional metadata, Stripe price IDs.
- **Subscriptions:** user linkage to plan + Stripe subscription IDs, status, billing dates.
- **Projects:** user-scoped containers for drafts; soft delete/archival recommended.
- **ContentDrafts:** title, body, outline/readability metadata, timestamps.
- **AI_Logs:** tool type (keyword/readability/structure), inputs/outputs, tokens/usage, project context, created_at.
- **OTP_Codes:** hashed codes, channel, expiry, attempt counts (if not stored solely in cache).
- **BlogPosts:** title, slug, content, author, publish metadata (if marketing blog is in-app).
- **AdminActions/Payments:** optional audit/payment reference tables.

## Key API Surface (representative)
- **Auth:** `POST /register`, `POST /verify-otp`, `POST /login` (OTP trigger), `POST /resend-otp`, `POST /logout`, `GET/PUT /profile`, `GET /usage`.
- **Projects/Drafts:** CRUD for `/projects`, `/projects/:id/drafts`, `/drafts/:id`.
- **AI Tools:** `POST /ai/keyword-research`, `/ai/improve-readability`, `/ai/generate-outline` with quota enforcement and AI log writes.
- **History:** `GET /ai/history` (user), `GET /admin/ai-logs` (admin filters/search, delete if needed).
- **Billing:** `POST /stripe/create-checkout-session`, `POST /stripe/webhook`, Stripe Customer Portal links from frontend.
- **Admin:** `/admin/users`, `/admin/stats`, plan/ban/impersonation actions (role-guarded).

## Security & Compliance
- HTTPS everywhere; HSTS; strong JWT signing keys with short expirations; optional refresh tokens.
- OTP codes expire quickly with request/attempt throttling; generic responses to prevent account enumeration.
- Parameterized queries/ORM; output escaping; content moderation via OpenAI as needed.
- RBAC on admin routes and per-user ownership checks on data access.
- CSRF mitigated via token-based auth and strict CORS; if cookies used, add CSRF tokens and SameSite/HttpOnly flags.
- Secrets via environment variables/secret manager; encrypted storage/backups; Stripe webhook signature validation.

## Performance & Scalability
- Stateless backend enables horizontal scaling; DB tuned with indexes (email, user_id, created_at) and InnoDB engine.
- CDN for static assets; API compression (Gzip/Brotli); caching for repeat AI responses/reference data.
- Consider background jobs for long AI tasks; rate limiting/backoff for OpenAI; logging/monitoring for usage and anomalies.

## Deployment & Operations
- CI/CD builds React assets and deploys backend (container-friendly). Separate environments with env-specific configs.
- Managed MySQL with backups/point-in-time recovery; optional read replicas as demand grows.
- Health checks and rolling deploys for zero/minimal downtime; centralized logging and metrics for uptime/alerting.

## Subscription Strategy
- Free tier plus two premium tiers (e.g., Pro, Enterprise) with defined quotas/feature flags stored in Plans table.
- Quota enforcement in backend middleware/services; user-facing usage indicators and upgrade prompts.
- Stripe Checkout for purchase, webhooks to sync subscription state, optional billing portal for self-serve management.

## UX Guidelines
- Clear navigation (hamburger on mobile), accessible forms/buttons, readable typography hierarchy.
- Visible feedback for loading/errors/success; announce alerts for screen readers.
- Auto-save drafts and AI outputs; version/history retrieval; export options (copy, docx/Markdown) planned.
