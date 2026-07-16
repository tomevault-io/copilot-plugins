## kauvex

> Instructions building apps with MCP


# KAUVEX COMMERCE CLOUD (KCC)
# Version: 2.0

## Platform
- Name: KAUVEX — "Everything. Everywhere. Delivered."
- Framework: Next.js 14 App Router + TypeScript
- Styling: Tailwind CSS + shadcn/ui
- ORM: Prisma
- Database: PostgreSQL (Supabase)
- Auth: Supabase Auth
- Storage: Supabase Storage
- Hosting: Vercel
- Colors: Navy #0A1628 | Orange #FF6B00
- Font: Inter

## Architecture
- Multi-storefront: path | subdomain | custom domain
- Multi-vendor: unlimited vendors with plan tiers
- Multi-warehouse: FBK + merchant fulfilled
- Centralized: one DB, one admin, one vendor login

## Key Directories
- /prisma/schema.prisma — Full database schema (~375 models)
- /lib/affiliates/ — Affiliate & Influencer Network (tracking, commission, payouts, fraud, onelink, promotions, b2b)
- /app/partners/ — Partner portal (associate/influencer registration, dashboard, links, tools, analytics)
- /app/influencer/ — Influencer storefront builder, product picker, promo manager
- /app/admin/affiliates/ — Admin full control panel (partners, commissions, payouts, fraud, promotions, b2b)
- /prisma/seeds/roles.ts — RBAC seed script
- /lib/permissions.ts — RBAC permission system
- /lib/storefront-context.tsx — Storefront context provider
- /lib/storefront-resolver.ts — Server-side storefront resolution
- /lib/buybox.ts — Buy box engine with weighted scoring
- /lib/search-engine.ts — Client search utilities
- /lib/security.ts — Rate limiting, 2FA, audit logging, validation
- /lib/ai/ — AI feature modules (descriptions, SEO, recommendations)
- /lib/shipping/ — Carrier integrations (dhl, fedex, aramex, local, gig, kwik, dhl-express-international, fedex-international, aramex-international, freight-forwarder)
- /lib/packaging-engine.ts — Packaging material selection, tier logic, compliance, order packaging records
- /lib/logistics-warehouse.ts — Warehouse staff portal (pick tasks, pack tasks, inbound receiving, inventory)
- /lib/logistics/ — Logistics engine (dispatch.ts, shipping-engine.ts, partner-tiers.ts, delivery-tiers.ts, fbk-debt.ts, terminology.ts)
- /lib/fuel/ — Fuel management engine (surcharge.ts, data-service.ts)
- /components/logistics/ShipmentTimeline.tsx — Unified cross-tier tracking display
- /app/express/ — Kauvex Express public courier (landing, book, track, business)
- /app/logistics/ — Partner portal (register, login, dashboard)
- /app/admin/logistics/ — Full admin control panel (rates, payouts, packaging, map, insurance, gaps, fbk, express)
- /app/admin/shipping/ — Admin shipping management (zones, surge-pricing, restrictions, hs-codes, business-accounts)
- /app/vendor/shipping/profiles/ — Profile builder for vendor shipping rules
- /app/vendor/shipping/dropoff/ — Drop-off manifest system
- /app/admin/packaging/ — Packaging materials master registry
- /app/admin/logistics/jobs/ — Admin logistics jobs management
- /app/admin/logistics/warehouses/ — Admin warehouse locations
- /app/admin/cj-dropshipping/packaging/ — CJ Dropshipping packaging config
- /app/express/carbon/ — Carbon footprint tracker (Phase 23)
- /app/express/corporate/ — Corporate & B2B services (Phase 23)
- /app/express/rates/calendar/ — Smart rate calendar (Phase 23)
- /app/express/delivery-confidence/ — Delivery confidence score (Phase 23)
- /app/logistics/why-kauvex/ — Competitor comparison page (Phase 23)
- /app/vendor/logistics/ — Vendor logistics section (shipments, pickups, manifests, performance, packaging guide)
- /app/vendor/fbk/packaging/ — FBK packaging tier configuration
- /app/supplier/logistics/ — Supplier packaging + delivery management
- /app/logistics/fleet/ — Logistics partner fleet management
- /app/warehouse/ — Warehouse staff portal (dashboard, inbound, outbound, inventory, packaging-stock, reports)
- /app/orders/[id]/tracking/ — Customer order tracking with live GPS
- /app/track/[trackingNumber]/ — Public Express tracking page
- /app/api/v1/express/ — Express API routes (waybills, pricing, tracking)
- /app/api/v1/logistics/ — Logistics API routes (tracking, partners, jobs, payouts)
- /app/api/v1/shipping/insurance/ — Insurance reserve API
- /app/api/v1/shipping/packaging/ — Packaging elements API
- /app/api/v1/shipping/customs/ — Customs document generation API
- /lib/cart-recovery.ts — Abandoned cart recovery engine
- /lib/bundles.ts — Product bundle management
- /lib/catalog-mode.ts — Catalog mode for B2B storefronts
- /lib/vendor-metrics.ts — Vendor health scoring
- /lib/api-helpers.ts — REST API response utilities
- /lib/validators/ — Zod validation schemas
- /components/home/ — Homepage section components (8 sections)
- /components/search/ — Voice search + barcode scanner
- /app/admin/ — Admin panel routes (60+ pages: commerce, sales, marketing, marketplace, operations, system)
- /app/admin/analytics/ — Analytics dashboards (realtime, search, BI)
- /app/vendor/ — Vendor panel routes
- /app/vendor/store-builder/ — Store builder with plan-gated features
- /app/vendor/fbk/ — FBK enrollment and management
- /app/vendor/advertising/ — Ad campaign manager
- /app/api/v1/ — REST API v1 (17 route groups)
- /app/vendor/inventory/ — Full inventory management (FBK + merchant)
- /app/vendor/inventory/replenishment-alerts/ — Reorder threshold alerts
- /app/vendor/products/add/ — Catalog matching entry point
- /app/vendor/products/bulk-upload/ — CSV bulk product upload
- /app/vendor/products/approval-request/ — Gated category approval requests
- /app/vendor/products/[id]/edit/ — Tabbed listing editor
- /app/vendor/products/[id]/offer/ — Multi-storefront offer management
- /app/vendor/orders/reports/ — Order reports & exports
- /app/vendor/advertising/campaigns/new/ — Campaign creation wizard
- /app/vendor/advertising/campaigns/[id]/ — Campaign performance detail
- /app/vendor/settings/permissions/ — Granular user permission grid
- /app/vendor/settings/permissions/history/ — Permission change audit log
- /app/vendor/settings/api-access/ — API key & third-party app management
- /app/vendor/university/ — Kauvex Seller University
- /app/vendor/b2b/ — B2B Central (wholesale, quotes, volume tiers)
- /app/vendor/brand-registry/ — Brand Registry enrollment & counterfeit reporting
- /app/vendor/a-plus-content/ — A+ Content module-based page builder
- /app/vendor/account-health/ — Account health dashboard & notifications
- /app/vendor/channels/ — Multi-Channel Integration Hub (eBay/Etsy sync)
- /app/vendor/reports/ — Reports Repository & custom report builder
- /app/admin/catalog/restricted-categories/ — Category/brand gating management
- /app/admin/catalog/approval-requests/ — Vendor approval queue
- /app/admin/university/ — University lesson content management
- /app/admin/brand-registry/ — Brand application review & approval
- /components/ui/brand-tokens.ts — All brand color, font, shadow constants
- /components/ui/Button.tsx — Branded button component all variants
- /styles/globals.css — CSS custom properties for brand colors
- /app/admin/brand/assets/ — Brand asset portal (admin)
- /app/admin/brand/protection/ — Brand violation reporting
- /app/partners/dashboard/brand-assets/ — Partner brand asset downloads
- /lib/email/templates.ts — Email master + transactional templates
- /lib/notifications/sms-templates.ts — SMS notification templates
- /lib/notifications/push-templates.ts — Push notification templates
- /lib/documents/templates.ts — Document templates (labels, waybills, invoices, packing lists, FBK statements)
- /lib/brand/seo.ts — Brand SEO helpers
- /lib/brand/compliance.ts — Brand compliance checker
- /components/notifications/in-app-notification.tsx — In-app notification centre
- /components/admin/brand-asset-portal.tsx — Admin brand asset management UI
- /components/admin/brand-violation-report.tsx — Brand violation report form
- /components/partners/brand-assets-page.tsx — Partner brand asset download page
- /lib/pay/ — Kauvex Pay engine (wallet.ts, bnpl.ts, cashback.ts, credit-score.ts, float.ts)
- /app/account/wallet/ — Customer wallet dashboard (top-up, withdraw, history, security)
- /app/account/pay-later/ — BNPL agreements (active, completed, overdue)
- /app/account/pay-later/[id]/ — BNPL agreement detail + payment schedule
- /app/vendor/wallet/ — Vendor wallet (earnings, withdrawal, transactions)
- /app/admin/pay-later/ — Admin BNPL dashboard (overview, agreements, config, risk)
- /app/admin/wallets/ — Admin wallet oversight (all wallets, balances, freeze/unfreeze)
- /app/api/v1/pay/wallet/ — Wallet API routes (topup, withdraw, virtual-account, pin)
- /app/api/v1/pay/bnpl/ — BNPL API routes (eligibility, agreements, repay)
- /app/api/v1/pay/cashback/ — Cashback API route
- /app/api/v1/pay/float/ — Float income tracking API
- /app/api/v1/cron/bnpl-charge/ — BNPL auto-charge cron job
- /app/api/v1/cron/cashback-process/ — Cashback processing cron job
- /app/api/v1/cron/float-track/ — Float tracking cron job
- /supabase/migrations/00018_kcc_phase18_kauvex_pay.sql — KV Pay database migration
- /supabase/migrations/00023_kcc_phase23_logistics_upgrade.sql — Phase 23 logistics upgrade (14 new tables, 15 carrier seeds)
- /supabase/migrations/00024_kcc_phase24_manufacturers.sql — Phase 24 manufacturer portal (kv_mfg_* tables)
- /supabase/migrations/00026_kcc_phase25_security.sql — Phase 25 security tables (kv_sec_*: blocked_requests, identity_verifications, fraud_scores, blacklist, file_scans, backups, credential_audit, otp_rate_limits)
- /lib/manufacturers/ — Manufacturer portal engine (registration.ts, verification.ts, categories.ts, inquiries.ts, production.ts, escrow.ts, disputes.ts, samples.ts, hubs.ts)
- /app/manufacturers/ — Manufacturer portal (landing, register, search, dashboard, quotes, request-quote, landed-cost, [slug] profile)
- /app/manufacturers/dashboard/ — Dashboard pages (overview, storefront, inquiries, quotes, orders, samples, production, escrow, reviews, analytics, settings)
- /app/admin/manufacturers/ — Admin manufacturer management (directory, hubs, disputes)
- /app/api/v1/manufacturers/ — Manufacturer API routes (registration, slug, stats, inquiries, quotes, orders, production, samples, escrow, rfq, disputes, ai-quote-draft, reviews)
- /app/api/v1/admin/manufacturers/ — Admin manufacturer API (list, update, hubs, disputes)
- /lib/security/ — Security engine (firewall.ts, fraud-rules.ts, file-scan.ts, otp-rate-limit.ts, backups.ts, credentials.ts, identity-verification.ts)
- /app/admin/security/ — Admin security dashboards (firewall, fraud, identity-review, backups, credentials)
- /app/api/v1/admin/security/ — Admin security API routes (firewall, fraud, identity, backups, credentials)
- /app/api/cron/independent-backup/ — Daily backup cron job
- /app/api/v1/cron/independent-backup/ — Backup API endpoint
- /scripts/setup-demo-accounts.js — Demo account seeding script
- /app/api/setup/demo-accounts/ — Demo accounts setup API route

Brand Quick Reference:
  Primary color: #0A1628 (navy) — kauvex-navy
  Accent color: #FF6B00 (orange) — kauvex-orange
  Font: Inter (400/500/600/700/800/900)
  Mono font: JetBrains Mono (tracking numbers)
  Primary CTA button: orange background, white text
  Secondary button: navy background, white text
  Card radius: 12px (rounded-xl)
  Button radius: 8px (rounded-lg)
  Dark mode: DISABLED (not supported at launch)

Sub-Brand Colors:
  Express: orange-forward (#FF6B00 primary)
  Logistics: navy-forward (#0A1628 primary)
  FBK: navy + green (#059669 accent)
  Pay: navy + gold (#D97706 accent)
  Live: orange + red (#DC2626 accent)
  Partners: navy + purple (#7C3AED accent)
  Originals: navy + gold (premium treatment)

Voice Rules:
  Always: direct, warm, active voice, short sentences
  Never: all caps in body text, technical jargon
    to customers, excessive apology, vague errors

## Default Storefronts
1. kauvex.com — Global USD (DEFAULT)
2. kauvex.com/uk — UK GBP
3. kauvex.com/ca — Canada CAD
4. kauvex.com/au — Australia AUD
5. kauvex.com/ng — Nigeria NGN

## Database Migrations
- 00001_kcc_core.sql — Core schema (users, products, orders, etc.)
- 00002_kcc_phase1_new_tables.sql — Phase 1 tables
- 00003_kcc_phase2_homepage.sql — Homepage sections
- 00004_kcc_phase3_search_ai.sql — Search + AI features
- 00005_kcc_phase4_logistics.sql — Logistics tables
- 00006_kcc_phase5_platform.sql — Platform features
- 00007_kcc_phase5b.sql — Phase 5b additions
- 00008_kcc_v4.sql — V4 migration
- 00017_kcc_brand_system.sql — Phase 17 brand system tables (kv_brand_assets, kv_brand_asset_downloads, kv_brand_violations)
- 00009_kcc_v2_enterprise.sql — V2 Enterprise+ (ERP, Procurement, Suppliers, RFQ, B2B, BNPL, Vendor Financing, Affiliates, Social Commerce, Live Shopping, Auctions, Subscriptions, Digital Products, Email Marketing, AI Assistant, Chat, Multi-Language, Franchise, Reputation, Authenticity, Tax, Accounting, Insurance, Credit, Forecasting, Fraud Detection) — ~60 tables, 60+ indexes

Key V2 Enterprise+ tables: erp_accounts, journal_entries, cost_centers, budgets, procurement_suppliers, supplier_products, purchase_orders, po_items, rfqs, rfq_responses, b2b_companies, b2b_users, b2b_price_tiers, b2b_quotes, b2b_invoices, bnpl_plans, bnpl_credit_scores, bnpl_contracts, bnpl_payments, vendor_financing_applications, vendor_financing_repayments, affiliate_groups, affiliate_commissions, affiliate_payouts, social_creators, social_content, social_content_products, live_streams, live_stream_products, auctions, auction_bids, auction_watchlists, subscription_plans, customer_subscriptions, subscription_orders, digital_products, license_keys, email_templates, email_campaigns, email_campaign_logs, email_lists, email_subscribers, crm_tickets, crm_messages, crm_pipelines, crm_deals, crm_tasks, ai_conversations, demand_forecasts, fraud_checks, conversations, conversation_participants, messages, languages, translation_keys, translations, pos_terminals, pos_sessions, franchise_agents, franchise_mini_stores, product_geo_visibility, vendor_reputation_scores, product_authenticity_codes, accounting_invoices, accounting_invoice_items, general_ledger, insurance_policies, insurance_claims, credit_applications, credit_lines

## CS-Cart Addon Equivalents (Native Builds)
- Live Search: PostgreSQL full-text search + autocomplete
- Abandoned Cart Recovery: 3-stage email sequence
- Product Bundles: Discounted multi-product bundles
- Catalog Mode: View-only storefronts for B2B
- Back-in-Stock Notifications: Email alerts on restock
- Product Comparison: Side-by-side (max 4)
- Gift Certificates: Digital gift codes
- Call Requests: Customer callback system
- Vendor Payouts: Batch payout processing
- Google Merchant Feed: XML export
- Digital Downloads: Expiring download links
- Seller Performance: Account health scoring (ODR, cancellation, late shipment)
- Reward Points: Loyalty program with tiers
- Product Video: Gallery video support
- Age Verification: Modal for restricted products
- Vendor Staff Management: Role-based staff access
- API Keys: Full REST API with key auth
- Webhooks: Event-driven integrations

## Build Status
- [x] Phase 0 (Pre-flight): Complete
- [x] Phase 1 (MVP Core): Complete
- [x] Phase 2 (Homepage+): Complete
- [x] Phase 3 (Search+AI): Complete
- [x] Phase 4 (Logistics): Complete
- [x] Phase 5 (Platform): Complete
- [x] V2 Enterprise+ (Sections 86-116): Complete
- [x] V3 Local Supplier Portal (Part 11): Complete
- [x] V3 Sourcing Module (Part 12): Complete
- [x] V3 POD System (Part 18): Complete
- [x] V3 Unique Features (Part 19): Complete
- [x] V3 Vendor Dropshipping (Part 29): Complete
- [x] V3 Art Marketplace (Part 30): Complete
- [x] Navigation integration (header, footer, homepage, dashboards): Complete
- [x] Admin pages for POD, Art Marketplace, Group Buy: Complete
- [x] Phase 11 (Seller Central Full): Complete
- [x] Phase 14 (Complete Shipping & Logistics): Complete
- [x] Phase 15 (Affiliate & Influencer Network): Complete
- [x] Phase 16 (Packaging + Logistics Dashboards): Complete
- [x] Phase 17 (Complete Brand System): Complete
- [x] Phase 18 (Kauvex Pay Wallet + BNPL): Complete
- [x] Phase 19 (Global Logistics Network GL1-GL15): Complete
- [x] Phase 20 (KSP + DB Migrations): Complete
- [x] Phase 21 (Fuel Intelligence System): Complete
- [x] Phase 22 (Domain Provisioning System): Complete
- [x] Phase 23 (Shipping & Logistics Platform Upgrade): Complete
- [x] Phase 24 (Global Manufacturer Portal): Complete
- [x] Phase 25 (Security & External Services): Complete

## Recent Enhancements (August 2026)
- **V3 Database**: 40+ new Prisma models (local suppliers, sourcing, POD, dropshipping, art/NFT, group buy, price alerts, live commerce, mentorship, carbon offsets, competition intel, Kauvex Originals, subscription boxes)
- **Local Supplier Portal** (`/supplier/`): Registration, login, dashboard, products, orders, earnings, coverage management
- **Product Sourcing Module**: Admin sourcing dashboard, product research pipeline, customer product requests (`/request-product/`), AI sourcing agent foundation, landed cost calculator
- **Print on Demand (POD) System** (`/vendor/pod/`): Design studio with Fabric.js canvas (text/image/AI tools), POD products management, orders, analytics dashboard
- **POD Design Marketplace** (`/pod-marketplace/`): Browse & license designs from creators, apply to POD products
- **Vendor Dropshipping Marketplace** (`/vendor/dropshipping/`): Multi-source product import (CJ, AliExpress, eBay, Etsy), per-vendor OAuth for eBay/Etsy, shared catalog integration
- **Live Commerce** (`/live/`): Active live streams grid, upcoming streams, one-tap purchase
- **Group Buy** (`/group-buy/`): Social shopping deals, invite friends, unlock lower prices
- **Concierge Shopping Assistant** (`/concierge/`): AI-powered personal shopping assistant chat
- **Digital Art Marketplace** (`/art-marketplace/`): Buy/sell digital art & illustrations, commercial licenses, instant download
- **NFT Marketplace** (`/nft-marketplace/`): Buy, sell & collect NFTs on Ethereum & Polygon with wallet support
- **Kauvex Originals tracker**: Admin product sourcing pipeline for private label
- **Vendor Mentorship marketplace**: Peer-to-peer paid mentorship sessions
- **Carbon footprint tracker**: Per-order CO2 estimates with tree planting offsets
- **NFT Marketplace**: Full blockchain-ready NFT support with Prisma models, API routes, lib functions, and admin/frontend pages
- **Price history & deal alerts**: 90-day price charts, target price notifications
- **Supabase Cron Jobs**: `supabase/migrations/00010_kcc_v3_cron_jobs.sql` — 5 automated functions for supplier escalation, price alerts, group buy expiry, price history recording, daily cleanup
- **Seller Central (Phase 11)**: `supabase/migrations/00011_kcc_seller_central.sql` — 9 new tables for restricted categories, approval requests, university lessons/progress, business customers, B2B volume tiers, brand registry, authorized sellers, counterfeit reports, A+ content, and multi-channel product sync

- **Phase 11 (Seller Central Full Replication)**: Amazon-style vendor dashboard with enhanced widgets, catalog matching with gated categories, multi-storefront offer management, bulk CSV upload, tabbed listing editor (8 tabs), full inventory management with FBK tools, orders with returns/claims/RMA, full Campaign Manager with 6-step wizard, granular user permissions matrix, Kauvex Seller University, B2B Central (quotes/volume tiers), Reports Repository (custom reports builder), Brand Registry (enrollment/counterfeit reporting), A+ Content module builder, Account Health dashboard with deactivation warnings, and Multi-Channel Integration Hub (eBay/Etsy product sync)
- **Fuel Management System**: Full fuel price tracking, surcharge rules, cost analysis, route impact calculator, fuel stations map, partner profitability dashboard, price alerts, fuel history — integrated across Express, Logistics partner, and Admin panels
- **Phase 24 (Global Manufacturer Portal)**: Cross-border B2B manufacturing marketplace with 12 Prisma models (kv_mfg_*), 13 API routes, 14 frontend pages, 10 lib modules. Features: 7-step manufacturer registration, 4-tier verification system (unverified/document/factory/gold), production tracker with 8-stage pipeline, milestone-based escrow extending Kauvex Pay, AI-assisted quote drafting, RFQ broadcast-to-multiple, quote comparison, sample ordering, landed cost calculator, manufacturing hub directory (31 hubs across 13 countries), admin management panel with dispute resolution. 19 manufacturing categories, 15 manufacturing hubs seeded. Admin sidebar integration, footer links added.
- **Domain Provisioning System** (`/admin/domains`, `/vendor/settings/domain`): Multi-storefront domain routing (15 country TLDs + vendor subdomains + custom domains + white label), Vercel/Cloudflare API provisioning, SSL monitoring cron, middleware-based domain detection, subdomain availability checker
- **Phase 25 (Security & External Services)**: Defense-in-depth security layer on top of Supabase RLS + Vercel native. 8 kv_sec_* database tables, 7 security lib modules (firewall, fraud detection, file scanning, OTP rate limiting, backups, credential rotation, identity verification), 5 admin API routes, 5 admin dashboards (Firewall & WAF, Fraud Detection, Identity Review, Backups, Credentials), Sentry error monitoring integration, independent daily backup cron job. External services: VirusTotal + Sightengine for file scanning, Smile Identity + Onfido for KYC, Cloudflare R2 for backups. All API keys server-side only via Vercel env vars.

## Phase 25 Security & External Services Knowledge Base

**Key Directories:**
  `/lib/security/` — Security engine (firewall.ts, fraud-rules.ts, file-scan.ts, otp-rate-limit.ts, backups.ts, credentials.ts, identity-verification.ts)
  `/app/admin/security/` — Admin security dashboards (firewall, fraud, identity-review, backups, credentials)
  `/app/api/v1/admin/security/` — Admin security API routes (firewall, fraud, identity, backups, credentials)
  `/app/api/cron/independent-backup/` — Daily backup cron job
  `sentry.client.config.ts` — Sentry client-side initialization
  `sentry.server.config.ts` — Sentry server-side initialization
  `sentry.edge.config.ts` — Sentry edge runtime initialization

**Security Modules:**
  `firewall.ts` — Attack pattern detection, IP blocking, WAF logging
  `fraud-rules.ts` — Risk scoring engine with configurable signals
  `file-scan.ts` — VirusTotal (malware) + Sightengine (content moderation)
  `otp-rate-limit.ts` — 3 attempts per 15min, 30min lockout
  `backups.ts` — Cloudflare R2 backup lifecycle management
  `credentials.ts` — API key rotation tracking with auto-reminders
  `identity-verification.ts` — Smile Identity (Africa) + Onfido (Global) KYC

**Admin Security Pages:**
  /admin/security/firewall — WAF dashboard (blocked requests, attack patterns)
  /admin/security/fraud — Fraud detection (blacklist, risk scores, order review)
  /admin/security/identity-review — KYC review queue (pending/approved/rejected)
  /admin/security/backups — Backup dashboard (status, history, manual trigger)
  /admin/security/credentials — Credential rotation tracker (API keys, passwords)

**API Endpoints:**
  GET /api/v1/admin/security/firewall — Blocked requests, IP blocks, attack stats
  POST /api/v1/admin/security/firewall — Block/unblock IP, log manual blocks
  GET /api/v1/admin/security/fraud — Fraud scores, blacklist, order risk data
  POST /api/v1/admin/security/fraud — Add/remove blacklist, update risk scores
  GET /api/v1/admin/security/identity — Identity verifications (KYC queue)
  POST /api/v1/admin/security/identity — Approve/reject KYC, update status
  GET /api/v1/admin/security/backups — Backup list, status, history
  POST /api/v1/admin/security/backups — Trigger manual backup
  GET /api/v1/admin/security/credentials — Credential audit log, rotation status
  POST /api/v1/admin/security/credentials — Log credential rotation
  POST /api/cron/independent-backup — Daily backup cron endpoint

**Database Tables (kv_sec_ prefix):**
  kv_sec_blocked_requests — WAF blocked request log
  kv_sec_identity_verifications — KYC verification attempts
  kv_sec_fraud_scores — Order fraud risk scores
  kv_sec_blacklist — IP/email/phone/device blacklist
  kv_sec_file_scans — File upload scan results
  kv_sec_backups — Backup history
  kv_sec_credential_audit — API key rotation audit log
  kv_sec_otp_rate_limits — OTP abuse prevention

**Security Headers (already in next.config.mjs):**
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  X-XSS-Protection: 1; mode=block
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=()
  Strict-Transport-Security: max-age=63072000; includeSubDomains; preload

**Demo Accounts:**
  manufacturer@kauvex.com / Manufacturer1! — role: vendor, manufacturer profile (Shenzhen Precision Electronics)
  wholesale@kauvex.com / Wholesale1! — role: customer
  Both login pages have demo credential buttons (auto-fill on click)
  Setup API: POST /api/setup/demo-accounts (Bearer: demo-accounts-secret-key-2026)
  Standalone script: scripts/setup-demo-accounts.js

## Phase 21 Fuel Intelligence System Knowledge Base

**Key Directories:**
  `/lib/fuel/surcharge.ts` — Fuel surcharge calculation engine
  `/lib/fuel/data-service.ts` — Fuel price data service (external API integration)
  `/app/express/fuel/` — Express fuel dashboard, route impact, cost planner, history, alerts
  `/app/express/fuel-stations/` — Express fuel stations map
  `/app/express/fuel-tracker/` — Express fuel tracker
  `/app/logistics/fuel/` — Logistics partner fuel dashboard & profitability
  `/app/admin/fuel/` — Admin fuel management (dashboard, prices, surcharge rules, cost analysis)
  `/app/admin/logistics/fuel/` — Admin logistics fuel stations management

**Express Fuel Pages:**
  /express/fuel — Fuel Dashboard (overview, stats)
  /express/fuel-stations — Nearby fuel stations map
  /express/fuel/route-impact — Fuel cost impact per route
  /express/fuel/cost-planner — Route cost planning tool
  /express/fuel/history — Historical fuel spend
  /express/fuel/alerts — Fuel price change alerts
  /express/fuel-tracker — Full fuel tracker

**Logistics Partner Fuel Pages:**
  /logistics/fuel — Fuel & Profitability tab (partner dashboard)
  /logistics/fuel/profitability — Detailed profitability breakdown

**Admin Fuel Pages:**
  /admin/fuel — Fuel Dashboard (system-wide overview)
  /admin/fuel/prices — Fuel price management per country
  /admin/fuel/surcharge-rules — Surcharge rule configuration
  /admin/fuel/cost-analysis — Cost analysis & reporting
  /admin/logistics/fuel — Fuel stations management

**API Endpoints:**
  GET /api/v1/fuel/prices — Get fuel prices
  GET /api/v1/fuel/prices/[countryCode] — Get prices for country
  GET /api/v1/fuel/surcharge — Calculate surcharge
  POST /api/v1/fuel/surcharge/rules — Manage surcharge rules
  GET /api/v1/fuel/dashboard — Admin fuel dashboard
  GET /api/v1/fuel/cost-planner — Cost planner data
  GET /api/v1/fuel/history — Fuel history
  GET /api/v1/fuel/alerts — Price alerts
  GET /api/v1/fuel/data-sources — External data source status
  GET /api/v1/fuel/profitability — Partner profitability
  GET /api/v1/fuel/partner-profile — Partner fuel profile
  POST /api/v1/cron/fuel-fetch — Cron: fetch latest fuel prices

## Phase 22 Domain Provisioning System Knowledge Base

**Key Directories:**
  `/lib/domains/country-domains.ts` — 15 Kauvex country TLD config, provision, remove
  `/lib/domains/provisioning.ts` — Vercel + Cloudflare API service
  `/lib/domains/vendor-subdomain.ts` — Vendor subdomain provisioning + availability check
  `/lib/domains/vendor-custom-domain.ts` — Custom domain provisioning + DNS instructions
  `/lib/domains/remove-domain.ts` — Domain removal service
  `/lib/domains/whitelabel-domain.ts` — White label domain provisioning
  `/lib/middleware/helpers.ts` — Middleware helper functions (getStorefrontByPath, getVendorBySubdomain, getVendorByCustomDomain)
  `/src/middleware.ts` — Master domain routing middleware
  `/app/admin/domains/` — Admin domain management dashboard
  `/app/vendor/settings/domain/` — Vendor domain settings page

**Kauvex Country Domains (15):**
  kauvex.com (US/Global — USD), kauvex.co.uk (UK — GBP), kauvex.ca (Canada — CAD),
  kauvex.com.au (Australia — AUD), kauvex.ng (Nigeria — NGN), kauvex.in (India — INR),
  kauvex.ae (UAE — AED), kauvex.de (Germany — EUR/DE), kauvex.fr (France — EUR/FR),
  kauvex.com.gh (Ghana — GHS), kauvex.co.ke (Kenya — KES), kauvex.co.za (South Africa — ZAR),
  kauvex.sa (Saudi Arabia — SAR), kauvex.com.br (Brazil — BRL), kauvex.jp (Japan — JPY)

**Domain Types:**
  `core` — kauvex.com + subdomains (admin, seller, logistics, etc.)
  `kauvex_country` — Country-specific TLDs (kauvex.co.uk, kauvex.ca, etc.)
  `vendor_subdomain` — {shop}.kauvex.com
  `vendor_custom` — merchants own domain (e.g., mystore.com)
  `whitelabel` — Enterprise white-label domains

**Middleware Routing Logic:**
  1. Kauvex country TLDs → set storefront headers (currency, country, language)
  2. Root domain + path → /ng, /uk, /ca, /au, etc. storefront routing
  3. Core subdomains → rewrite to admin/vendor/logistics/etc.
  4. Vendor subdomains → rewrite to (stores)/{vendor-slug}/
  5. Custom domains → rewrite to (stores)/{hostname}/
  6. Protected subdomains require auth cookie

**API Endpoints:**
  GET /api/v1/domains/check-availability?subdomain= — Check subdomain availability
  POST /api/v1/domains/provision — Provision subdomain or custom domain
  GET /api/v1/domains/status?vendor_id= — Get domain status
  DELETE /api/v1/domains/remove?vendor_id=&domain= — Remove domain
  GET /api/v1/domains/country-domains — List all 15 country TLDs with status
  POST /api/v1/domains/country-domains — Provision all pending country domains
  POST /api/v1/domains/whitelabel — Provision white label domain
  GET /api/cron/ssl-check — SSL certificate status check (cron)

**Database Tables (kv_dom_ prefix):**
  kv_dom_domains — All domains (core, country, vendor, custom, whitelabel)
  kv_dom_subdomain_checks — Subdomain availability check history
  kv_dom_ssl_checks — SSL certificate status tracking
  kv_dom_dns_events — DNS provisioning event log

**Critical Rules:**
  Country domains are KEPT ACTIVE — never removed on vendor cancellation
  Vendor subdomains use wildcard CNAME — no per-vendor DNS needed
  Custom domains require vendor to add CNAME at their registrar
  SSL is auto-provisioned by Vercel for verified domains
  Middleware sets x-storefront-* headers for downstream use by storefront context

## Navigation & UI Links
- **Footer**: All V3 features linked under "Explore" section
- **Mega Menu**: "Explore" category added with links to Live, Group Buy, POD, Art, Concierge, Request Product
- **Homepage**: "Explore Kauvex" feature card section showing all V3 features
- **Admin Sidebar**: POD, Art Marketplace, Group Buy, Sourcing under "Sourcing & Products" in Marketplace section
- **Admin Sidebar**: Logistics section: Global Overview, Countries, Carriers, IOSS, DDP, Compliance
- **Admin Sidebar**: Fuel Management section: Fuel Dashboard, Fuel Prices, Surcharge Rules, Cost Analysis
- **Admin Sidebar**: System > Developers: Domains (new)
- **Vendor Sidebar**: Settings > Domain (new)
- **Express Sidebar**: Fuel section: Fuel Dashboard, Fuel Stations, Route Impact, Cost Planner, Fuel History, Price Alerts, Fuel Tracker
- **Logistics Partner Sidebar**: Fuel & Profitability tab added to partner dashboard
- **Vendor Sidebar**: POD section (Dashboard, Design Studio, Products, Orders, Design Marketplace) and Dropshipping section under Products
- **Admin Pages**: `/admin/pod`, `/admin/art-marketplace`, `/admin/group-buy` created with management tables
- **Admin Pages**: `/admin/logistics/global`, `/admin/logistics/countries`, `/admin/logistics/countries/[code]`, `/admin/logistics/carriers`, `/admin/logistics/ioss`, `/admin/logistics/ddp`, `/admin/logistics/compliance`
- **Footer**: Manufacturer Portal, Find Manufacturers added to "Sell" column
- **Admin Sidebar**: Marketplace > Partner Network: Manufacturers, Manufacturer Hubs, Manufacturer Disputes (Phase 24)

## Phase 15 Affiliate & Influencer Network Knowledge Base

**Key Directories:**
  `/lib/affiliates/` — Core engine (commission.ts, payouts.ts, fraud.ts, onelink.ts, tracking.ts, promotions.ts, b2b.ts, storefront.ts)
  `/app/partners/` — Partner portal (register, login, dashboard, tools, analytics, settings)
  `/app/influencer/` — Influencer storefront builder & product picker
  `/app/admin/affiliates/` — Admin panel (partners, commissions, payouts, fraud, promotions, b2b, settings)
  `/components/partners/` — Shared UI (PartnerLayout, StorefrontPage, ProductCard, StatCard, ToolCard, PromoCard, empty-state, loading-skeleton, payout-chart)

**Commission Models:**
  Percentage: fixed % of sale amount
  Flat Fee: fixed $ per conversion
  Tiered: rate increases with volume (e.g., 5% base → 8% at 50 sales/mo)
  Performance: rate varies by conversion rate bands
  Bounty: one-time fixed reward per action (signup, sale, etc.)

**Partner Types:**
  Associate: link-based promotion, commission on sales
  Influencer: personal storefront + product picks + promotions
  B2B Referral: commission on business account signups + purchases

**Fraud Detection Triggers:**
  Rapid clicks (>50/min), same IP multiple clicks, VPN/proxy detection, cookie stuffing, bot user-agent patterns, conversion rate outliers (>50%), self-referral (same IP as purchaser), multiple accounts same device

**Storefront Builder Modes:**
  Quick: auto-generate from product picks, social links, bio
  Custom: drag-and-drop sections with full control

**OneLink Routing:**
  Detects visitor country via IP → redirects to best regional storefront
  Fallback: default global storefront

**Key API Routes:**
  POST /api/v1/affiliates/clicks — Record affiliate click
  POST /api/v1/affiliates/convert — Record conversion (order completion)
  GET /api/v1/affiliates/commissions — List partner commissions
  POST /api/v1/affiliates/payouts/request — Request payout
  GET /api/v1/affiliates/payouts — List payouts
  POST /api/v1/affiliates/onelink/resolve — Resolve OneLink redirect
  GET /api/v1/affiliates/links — Get partner's tracking links
  POST /api/v1/affiliates/links — Create tracking link
  GET /api/v1/affiliates/banners — Available banners
  GET /api/v1/affiliates/analytics — Partner analytics data
  POST /api/v1/affiliates/promotions — Create promotion/bounty
  GET /api/v1/affiliates/storefront — Get influencer storefront
  GET api/v1/affiliates/products — Trackable products catalog

  POST /api/v1/admin/affiliates/partners — Admin: create/update partners
  POST /api/v1/admin/affiliates/commissions/approve — Admin: approve commission
  POST /api/v1/admin/affiliates/payouts/process — Admin: process payout batch
  GET /api/v1/admin/affiliates/promotions — Admin: list all promotions
  GET /api/v1/admin/affiliates/b2b — Admin: B2B partner management
  GET /api/v1/admin/affiliates/fraud/flags — Admin: fraud flag queue
  POST /api/v1/admin/affiliates/fraud/resolve — Admin: resolve fraud flag
  GET /api/v1/admin/affiliates/analytics — Admin: system-wide analytics

**Cron Jobs (Phase 15):**
  `kv_aff_calculate_commissions()` — Daily batch commission calculation
  `kv_aff_process_payouts()` — Weekly payout processing
  `kv_aff_expire_promotions()` — End date cleanup
  `kv_aff_fraud_scan()` — Daily fraud pattern scan
  `kv_aff_cleanup_stale_clicks()` — Remove clicks older than 90 days
  `kv_aff_calculate_tiered_rates()` — Monthly tier rate recalculation

## Database Migration Instructions
1. Apply SQL migration: `cd supabase && supabase migration up`
2. Generate Prisma client: `npx prisma generate`
3. Seed roles: POST to `/api/setup/seed-roles` with Bearer token matching SEED_SECRET env var
4. V2 Enterprise+ (migration 00009) must be applied manually via Supabase Dashboard SQL Editor — copy contents of `supabase/migrations/00009_kcc_v2_enterprise.sql` and paste into project `stbgamqenraauqpgtbkv`

## Phase 14 Shipping & Logistics Knowledge Base

**Terminology Reference:**
  Marketplace orders → Shipping Label
  Kauvex Express courier → Waybill
  Intercity road freight → Consignment Note
  International air freight → Air Waybill (AWB)
  International sea freight → Bill of Lading (BOL)
  Multi-item shipments → Packing List
  International commercial sales → Commercial Invoice
  Below $300 customs → CN22
  Above $300 customs → CN23

**Carriers available:**
  Domestic: dhl, fedex, aramex, local, gig, kwik
  International: dhl-international, fedex-international, aramex-international
  Aggregator: freight-forwarder (routes not covered by single carrier)

**Tier routing (determineTier):**
  TIER_1_LOCAL → independent partners (riders/drivers) + GIG/Kwik
  TIER_2_DOMESTIC_FREIGHT → freight partners + domestic carrier fallback
  TIER_3_INTERNATIONAL → carrier APIs ONLY (never independent partners)

**FBK Fee Triggers:**
  Inbound handling → on warehouse receipt confirmation
  Storage fee → monthly per unsold unit (from vendor wallet)
  Pick & pack → deducted from sale earnings
  Sales commission → deducted from sale earnings
  Removal fee → on vendor stock removal request
  Long-term surcharge → after 180 days unsold
  Debt interest → after 30 days outstanding (2% monthly)

**Key API Endpoints:**
  POST /api/v1/express/waybills — Create express waybill
  POST /api/v1/express/pricing — Get instant pricing quote
  GET /api/v1/express/tracking?waybillNumber= — Track express shipment
  GET /api/v1/logistics/tracking?shipmentId= — Get tracking timeline
  GET /api/v1/logistics/partners — List/filter logistics partners
  GET/POST/PATCH /api/v1/logistics/jobs — Manage delivery jobs
  GET/POST/PATCH /api/v1/logistics/payouts — Manage partner payouts
  GET/POST/PATCH /api/v1/shipping/insurance — Insurance reserve management
  GET/POST /api/v1/shipping/packaging — Packaging elements & add-ons
  POST /api/v1/shipping/customs — Generate customs documents
  POST /api/v1/shipping/rates — Get shipping rates
  POST /api/v1/shipping/labels — Create shipping label
  POST /api/v1/shipping/auto-route — Auto-route shipment

**Asset Ownership Milestones:**
  Launch: zero vehicles — asset light, pure technology
  500+ orders/day: 2-3 branded vans per major city (warehouse only)
  2,000+ orders/day: own trucks on Lagos-Abuja + Lagos-PHC routes
  3+ countries: lease airline cargo space
  Dominant Nigerian market: lease first cargo aircraft
  Pan-African: port terminal + vessel strategy

## Phase 18 Kauvex Pay Knowledge Base

**Key Directories:**
  `/lib/pay/` — Core engine (wallet.ts, bnpl.ts, cashback.ts, credit-score.ts, float.ts)
  `/app/account/wallet/` — Customer wallet dashboard
  `/app/account/pay-later/` — BNPL agreements + detail pages
  `/app/vendor/wallet/` — Vendor wallet (earnings, withdrawal, transactions)
  `/app/admin/pay-later/` — Admin BNPL dashboard (overview, agreements, config, risk)
  `/app/admin/wallets/` — Admin wallet oversight

**Wallet System (KP1):**
  Every account gets a wallet on registration (auto-created via DB trigger)
  Top-up methods: Card (Paystack), Bank Transfer (virtual account), USSD
  Spending: one-click checkout, split payment (wallet + card)
  Withdrawal: below ₦50K instant, above ₦50K manual review (24h)
  Security: 4-digit PIN, daily spend limits, fraud flagging
  Cashback: configurable per category/storefront, 30-day pending period

**BNPL System (KP2):**
  Customer pays 25% upfront, receives item IMMEDIATELY
  Remaining 75% in 3 installments over 9 weeks (21 days apart)
  Vendor receives FULL earnings on Day 1 (Kauvex pays vendor)
  Kauvex holds the credit risk
  Auto-charge runs daily at 9AM via cron
  Late fee: ₦500 after 7-day grace period
  BNPL suspended: cannot START new BNPL, existing agreements remain active

**BNPL Eligibility:**
  Account age: 3+ months
  Order history: 2+ completed orders
  No outstanding Kauvex debt
  External credit check (Carbon/FairMoney/Lenco) for orders ≥₦50,000
  Limits: ₦20K new → ₦50K after 2 → ₦100K after 5 → ₦200K after 10

**Float Income:**
  All wallet balances earn bank interest (tracked daily)
  Treasury reporting via admin dashboard

**BNPL Critical Rules:**
  Customer receives item IMMEDIATELY after 25% first payment
  Vendor receives FULL earnings on Day 1
  Kauvex holds the credit risk
  Auto-charges run daily at 9AM via cron
  Late fee: only after 7-day grace period
  BNPL suspended: cannot START new BNPL, existing agreements remain active
  Never cancel or reverse a shipped order due to missed BNPL payment

**Key API Endpoints:**
  GET/POST /api/v1/pay/wallet/topup — Wallet info + top-up
  POST /api/v1/pay/wallet/withdraw — Withdraw to bank
  GET /api/v1/pay/wallet/virtual-account — Get dedicated account
  POST /api/v1/pay/wallet/pin — Set/verify PIN
  GET/POST /api/v1/pay/bnpl/eligibility — Check eligibility
  GET/POST /api/v1/pay/bnpl/agreements — List/create agreements
  GET /api/v1/pay/bnpl/agreements/[id] — Agreement detail
  POST /api/v1/pay/bnpl/agreements/[id]/repay — Early repayment
  GET /api/v1/pay/cashback — Customer cashback history
  GET /api/v1/pay/float — Admin float tracking
  POST /api/v1/cron/bnpl-charge — Daily auto-charge cron
  POST /api/v1/cron/cashback-process — Daily cashback processing
  POST /api/v1/cron/float-track — Daily float tracking

## Phase 19 Global Logistics Network Knowledge Base

**Key Directories:**
  `/lib/logistics/global-config.ts` — Country config CRUD, rate cards, shipping fee calc
  `/lib/logistics/carrier-selector.ts` — Carrier selection by country/tier/weight/budget
  `/lib/logistics/carrier-integrations.ts` — Static registry of 35 carriers across 15 countries
  `/lib/logistics/what3words.ts` — What3Words integration
  `/lib/logistics/cod.ts` — COD collection/remittance management
  `/lib/logistics/customs.ts` — Duties estimation, commercial invoice, CN22/CN23, packing list
  `/lib/logistics/packaging-options.ts` — 8 packaging types with smart suggestion engine
  `/app/admin/logistics/global/page.tsx` — Admin world map overview
  `/app/admin/logistics/countries/page.tsx` — Country list with search
  `/app/admin/logistics/countries/[code]/page.tsx` — Per-country panel (4 tabs)
  `/app/admin/logistics/carriers/page.tsx` — Carrier API integrations list
  `/app/admin/logistics/ioss/page.tsx` — IOSS EU VAT calculator
  `/app/admin/logistics/ddp/page.tsx` — DDP compliance per country
  `/app/admin/logistics/compliance/page.tsx` — Compliance dashboard
  `/app/express/book/packaging/page.tsx` — Express packaging visual selector
  `/app/logistics/register/country/page.tsx` — Country-specific partner registration

**Launch Countries (15):** NG, GB, US, AE, IN, AU, DE, CA, GH, KE, ZA, SA, BR, JP, FR

**Carrier Integrations (35):**
  DHL: dhl, dhl-international, dhl-express-international
  FedEx: fedex, fedex-international
  Aramex: aramex, aramex-international
  GIG: gig-logistics (Nigeria)
  Kwik: kwik-delivery (Nigeria)
  And 26 more across 15 countries

**Packaging Types:** express-poly, express-box-s/m/l, express-fragile, express-cold, express-crate

**API Endpoints:**
  GET /api/v1/logistics/countries — List all countries with configs
  GET /api/v1/logistics/rate-cards?country=XX — Get rate cards for country
  GET /api/v1/logistics/packaging?country=XX — Get packaging fees
  POST /api/v1/logistics/cod — Manage COD collections
  POST /api/v1/logistics/w3w — Resolve What3Words
  POST /api/v1/logistics/customs — Estimate duties/CN22/CN23
  GET /api/v1/logistics/exchange-rate?from=XX&to=YY — Get exchange rate

**Database Tables (kv_glx_ prefix):**
  kv_glx_countries — Country configs (15 seeded)
  kv_glx_carriers — Carrier definitions (23 seeded)
  kv_glx_carrier_integrations — Live API integration status
  kv_glx_rate_cards — Rate cards (8 seeded)
  kv_glx_packaging_options — Packaging fee schedule (21 seeded)
  kv_glx_duty_estimates — Customs duty estimates
  kv_glx_cod_collections — COD tracking
  kv_glx_exchange_rates — Currency conversion
  kv_glx_customs_documents — CN22/CN23/commercial invoices

**Key Rules:**
  Logistics = direct point to point (NO intermediate warehouse unless FBK, Tier 2 hub, or Tier 3 customs)
  Packaging = EXPRESS senders ONLY — marketplace customers NEVER choose packaging
  Tier 3 international = carrier APIs NEVER independent partners
  Every country has own carriers, rates, partners, currency, tax, customs
  IOSS = EU €150 threshold for VAT collection at checkout
  DDP = required for DE, FR, SA, IN

## Fuel Management System

**Key Directories:**
  `/lib/fuel/surcharge.ts` — Fuel surcharge calculation engine
  `/lib/fuel/data-service.ts` — Fuel price data service (external API integration)
  `/app/express/fuel/` — Express fuel dashboard, route impact, cost planner, history, alerts
  `/app/express/fuel-stations/` — Express fuel stations map
  `/app/express/fuel-tracker/` — Express fuel tracker
  `/app/logistics/fuel/` — Logistics partner fuel dashboard & profitability
  `/app/admin/fuel/` — Admin fuel management (dashboard, prices, surcharge rules, cost analysis)
  `/app/admin/logistics/fuel/` — Admin logistics fuel stations management

**Express Fuel Pages:**
  /express/fuel — Fuel Dashboard (overview, stats)
  /express/fuel-stations — Nearby fuel stations map
  /express/fuel/route-impact — Fuel cost impact per route
  /express/fuel/cost-planner — Route cost planning tool
  /express/fuel/history — Historical fuel spend
  /express/fuel/alerts — Fuel price change alerts
  /express/fuel-tracker — Full fuel tracker

**Logistics Partner Fuel Pages:**
  /logistics/fuel — Fuel & Profitability tab (partner dashboard)
  /logistics/fuel/profitability — Detailed profitability breakdown

**Admin Fuel Pages:**
  /admin/fuel — Fuel Dashboard (system-wide overview)
  /admin/fuel/prices — Fuel price management per country
  /admin/fuel/surcharge-rules — Surcharge rule configuration
  /admin/fuel/cost-analysis — Cost analysis & reporting
  /admin/logistics/fuel — Fuel stations management

**API Endpoints:**
  GET /api/v1/fuel/prices — Get fuel prices
  GET /api/v1/fuel/prices/[countryCode] — Get prices for country
  GET /api/v1/fuel/surcharge — Calculate surcharge
  POST /api/v1/fuel/surcharge/rules — Manage surcharge rules
  GET /api/v1/fuel/dashboard — Admin fuel dashboard
  GET /api/v1/fuel/cost-planner — Cost planner data
  GET /api/v1/fuel/history — Fuel history
  GET /api/v1/fuel/alerts — Price alerts
  GET /api/v1/fuel/data-sources — External data source status
  GET /api/v1/fuel/profitability — Partner profitability
  GET /api/v1/fuel/partner-profile — Partner fuel profile
  POST /api/v1/cron/fuel-fetch — Cron: fetch latest fuel prices

---
> Source: [shuddi1962/kauvex](https://github.com/shuddi1962/kauvex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
