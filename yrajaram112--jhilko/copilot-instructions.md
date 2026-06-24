## jhilko

> > This file is the **single source of truth** for any developer or AI assistant working on this project.

# CLAUDE.md — Jhilko: Nepal Local Gift Delivery Platform
## Technical Architecture, Stack Decisions & Implementation Guide

> This file is the **single source of truth** for any developer or AI assistant working on this project.
> Read this fully before writing a single line of code.
>
> **v3.0 (June 2026) revision:** Stack upgraded to current versions (Next.js 16, NestJS 11, Prisma 7, Node 22 LTS, Tailwind v4, Zod 4). `next-pwa` replaced with Serwist. Google Maps replaced with Baato (Nepal-local, ~70% cheaper). Stripe section corrected — **Stripe is NOT available to Nepal-registered companies**; international payments are now Phase-2 via Khalti's international card processing / bank acquirer. Brand finalized as **Jhilko**.

---

## 1. PROJECT IDENTITY

**Name:** `Jhilko` (झिल्को — "spark/flash" in Nepali) — **final brand name**
**Domains:** jhilko.com.np (primary), jhilko.com (redirect)
**Tagline:** *"Send love anywhere in Nepal."*
**Core Mission:** Let anyone in Nepal (or abroad) send a meaningful gift — cake, flowers, sweets, cosmetics, hampers — to someone in any city or village of Nepal, fulfilled by a vetted local partner in that city. Not shipped from KTM. **Sourced and delivered locally.**

**Key Differentiation from Competitors:**
- KoselXpress / YourKoseli serve **diaspora (USA/UK → KTM)** — we serve **domestic Nepal to Nepal (Dharan → Dang, Butwal → Palpa)**
- Pathao/Foodmandu operate **within-city only** — we go **city-to-city**
- We use **local partners** in every city — no centralized warehouse, ultra-low delivery cost

---

## 2. TECH STACK DECISIONS

### 2.1 Frontend — Next.js 16 (App Router)
**Why Next.js:**
- Superior ecosystem for this use case (next/image for product photos, proxy/middleware for auth, ISR for catalog pages)
- Better SEO out of the box (critical for organic discovery)
- More hire-able in Nepal dev market
- App Router allows server components (fast product catalog rendering)
- Native API routes handle webhooks (payment callbacks) cleanly

```
Framework:       Next.js 16.x (App Router, Turbopack, TypeScript 5.x) + React 19.2.x
Styling:         Tailwind CSS v4 (CSS-first config via @theme, no tailwind.config.js)
Animations:      Motion v12+ (package name "motion", NOT "framer-motion")
UI Components:   shadcn/ui (CLI: npx shadcn@latest) + Aceternity UI
State:           Zustand v5 + TanStack Query v5
Forms:           React Hook Form v7 + Zod v4 (import from "zod", use z.email() etc. top-level validators)
Maps:            Baato Maps (Nepal-local, MapLibre GL JS renderer) — see §13
Icons:           Lucide React
Fonts:           Geist (primary) + Noto Sans Devanagari (Nepali) via next/font
PWA:             Serwist (@serwist/next) — next-pwa is UNMAINTAINED, do not use
Image CDN:       Cloudinary (next/image custom loader)
Lint/Format:     ESLint 9 (flat config) + Prettier
```

**Version gotchas Sonnet/codegen must respect (Next.js 16):**
- `cookies()`, `headers()`, `params`, `searchParams` are **async** — always `await` them
- `middleware.ts` is renamed to `proxy.ts` in Next 16
- Use Turbopack (default) — do not add webpack config
- Server Actions for mutations; Route Handlers only for webhooks/external callbacks
- Node.js >= 20.9 required; we standardize on Node 22 LTS

**Design System — Apple Vision / Glassmorphism:**
```css
/* Core CSS Variables — put in globals.css */
:root {
  /* Brand Colors */
  --brand-primary:    #FF4D6D;   /* Deep rose — emotion, gifting */
  --brand-secondary:  #FFB830;   /* Golden amber — Nepali festivals */
  --brand-accent:     #7B2FBE;   /* Royal purple — premium trust */
  
  /* Surfaces */
  --surface-glass:    rgba(255,255,255,0.08);
  --surface-glass-border: rgba(255,255,255,0.15);
  --surface-card:     rgba(255,255,255,0.72);
  --backdrop-blur:    20px;
  
  /* Text */
  --text-primary:     #0A0A0B;
  --text-secondary:   #6B7280;
  --text-muted:       #9CA3AF;
  
  /* Gradients */
  --gradient-hero:    linear-gradient(135deg, #FF4D6D 0%, #FFB830 50%, #7B2FBE 100%);
  --gradient-card:    linear-gradient(145deg, rgba(255,77,109,0.1) 0%, rgba(123,47,190,0.05) 100%);
  
  /* Spacing tokens */
  --radius-sm:        8px;
  --radius-md:        16px;
  --radius-lg:        24px;
  --radius-xl:        32px;
  --radius-pill:      9999px;
}
```

**Motion Principles:**
- Spring physics everywhere (no `ease-in-out` flat curves)
- Page transitions: slide + fade (300ms spring)
- Cards: `whileHover={{ y: -4, scale: 1.02 }}` with shadow
- Buttons: `whileTap={{ scale: 0.97 }}`
- Modals: scale from 0.94 + fade in (apple-style)
- Skeleton loaders: pulse shimmer animation
- Delivery tracker: animated progress ring + bouncing location pin

---

### 2.2 Backend — NestJS + PostgreSQL + Redis

**Why NestJS:**
- TypeScript-native (matches frontend, single language stack)
- Excellent WebSocket support via `@nestjs/websockets` + Socket.IO
- Built-in support for BullMQ (job queues for delivery status, SMS)
- Modular architecture scales well as features grow
- Nepal developers familiar with it (common in KTM tech scene)

```
Runtime:         Node.js 22 LTS
Framework:       NestJS 11.x (stable — do NOT use v12 alphas until GA)
Database:        PostgreSQL 17 (Neon)
ORM:             Prisma 7 (TypeScript runtime; prisma-client generator, no binary engines)
Cache/PubSub:    Redis 7 (Upstash managed)
Job Queue:       BullMQ v5 + Redis
Real-time:       Socket.IO v4 (NestJS WebSocket gateway)
Auth:            JWT (access + refresh tokens) + argon2 (preferred over bcrypt in 2026)
File Storage:    Cloudinary (images, partner KYC docs)
Email:           Resend (transactional email)
SMS:             Sparrow SMS (Nepal) — no Twilio needed for MVP (all recipients are in Nepal)
Push:            Firebase FCM (web push for PWA; native later)
Validation:      Zod v4 via nestjs-zod (shared schemas with frontend) — preferred over class-validator
API Docs:        Swagger (auto-generated via @nestjs/swagger)
Tests:           Vitest + Supertest
```

**Version gotchas for codegen (NestJS 11 / Prisma 7):**
- Prisma 7: client output path is required in `schema.prisma` generator block; import client from the generated path, not `@prisma/client` legacy default
- Prisma 7 has no Rust engine binaries — nothing special needed on Railway, just `prisma migrate deploy`
- NestJS 11 uses Express v5 under the hood — wildcard routes are `/*splat`, not `*`
- Use `@nestjs/throttler` v6+ with Redis storage for distributed rate limiting

---

### 2.3 Infrastructure & Deployment

```
Frontend:        Vercel (Next.js — zero config deploy)
                 NOTE: Vercel Hobby tier prohibits commercial use. Dev/staging on Hobby is fine;
                 at launch either pay Vercel Pro ($20/mo) OR deploy Next.js on Railway alongside
                 the API (~$5/mo total). Decide at launch based on traffic — Railway is the
                 cheaper default for a Nepal-revenue business.
Backend:         Railway.app (NestJS — supports WebSockets, persistent processes, ~$5/mo)
Database:        Neon.tech (serverless Postgres 17 — free tier, ap-southeast region for latency)
Redis:           Upstash (serverless Redis — pay per request, $0 to start)
Media CDN:       Cloudinary (free tier 25GB)
DNS/CDN:         Cloudflare (free, fast globally, DDoS protection)
Monitoring:      Sentry (free tier: 5K errors/mo — enough for MVP) + Railway logs (skip Axiom for MVP)
CI/CD:           GitHub Actions
Monorepo:        pnpm workspaces + Turborepo (jhilko-web, jhilko-api, packages/shared)

Realistic monthly cost at launch: $5–25 total (Railway + everything else on free tiers).
```

**Why NOT AWS/GCP for MVP:**
- Cost at zero-scale is near-zero on above stack
- Railway = $5/month to start (vs AWS ~$50+ minimum viable)
- Vercel has Nepal CDN edge nodes (fast load times)
- Focus on product, not DevOps

**Environment Variables Pattern:**
```
.env.local          (local dev — never committed)
.env.production     (production secrets — in Vercel/Railway env panel)
```

---

## 3. DATABASE SCHEMA (PostgreSQL via Prisma)

### 3.1 Core Models

```prisma
// schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ─── USERS ─────────────────────────────────────────────────────────────────

model User {
  id            String    @id @default(cuid())
  phone         String?   @unique
  email         String?   @unique
  name          String?
  avatarUrl     String?
  isGuest       Boolean   @default(false)
  preferredLang String    @default("ne") // "ne" | "en"
  
  orders        Order[]
  addresses     Address[]
  reviews       Review[]
  
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  @@index([phone])
  @@index([email])
}

// ─── DELIVERY PARTNERS ──────────────────────────────────────────────────────

model Partner {
  id                String         @id @default(cuid())
  name              String
  phone             String         @unique
  email             String?        @unique
  profilePhoto      String?
  
  // KYC Fields
  citizenshipFront  String?        // Cloudinary URL
  citizenshipBack   String?        // Cloudinary URL
  selfieWithId      String?        // Cloudinary URL
  citizenshipNumber String?
  dateOfBirth       DateTime?
  permanentAddress  String?
  
  // KYC Status
  kycStatus         KYCStatus      @default(PENDING)
  kycVerifiedAt     DateTime?
  kycVerifiedBy     String?        // Admin user ID
  kycRejectionNote  String?
  
  // Legal
  agreementSignedAt DateTime?
  agreementVersion  String?
  
  // Banking
  bankName          String?
  bankAccountNumber String?
  bankBranch        String?
  
  // Assignment
  cityId            String
  city              City           @relation(fields: [cityId], references: [id])
  zones             Zone[]
  
  // Performance
  totalDeliveries   Int            @default(0)
  successRate       Float          @default(0)
  avgRating         Float          @default(0)
  isActive          Boolean        @default(false) // Only after KYC approval
  isOnline          Boolean        @default(false) // Currently available
  lastSeenAt        DateTime?
  
  orders            Order[]
  reviews           Review[]
  payouts           Payout[]
  
  emergencyContactName  String?
  emergencyContactPhone String?
  
  createdAt         DateTime       @default(now())
  updatedAt         DateTime       @updatedAt
  
  @@index([cityId])
  @@index([kycStatus])
}

enum KYCStatus {
  PENDING
  UNDER_REVIEW
  APPROVED
  REJECTED
  SUSPENDED
}

// ─── GEOGRAPHY ──────────────────────────────────────────────────────────────

model Province {
  id        String   @id @default(cuid())
  name      String
  nameNe    String   // Nepali name
  districts District[]
}

model District {
  id          String   @id @default(cuid())
  name        String
  nameNe      String
  provinceId  String
  province    Province @relation(fields: [provinceId], references: [id])
  cities      City[]
}

model City {
  id            String    @id @default(cuid())
  name          String
  nameNe        String
  slug          String    @unique  // e.g. "dharan", "butwal"
  districtId    String
  district      District  @relation(fields: [districtId], references: [id])
  
  latitude      Float?
  longitude     Float?
  
  isActive      Boolean   @default(false) // Has at least 1 approved partner
  coverageRadius Float?   // km radius we cover
  
  partners      Partner[]
  zones         Zone[]
  orders        Order[]
  
  @@index([slug])
  @@index([isActive])
}

model Zone {
  id          String    @id @default(cuid())
  name        String    // e.g. "Dharan Bazaar", "Biratnagar Ward 5"
  cityId      String
  city        City      @relation(fields: [cityId], references: [id])
  partners    Partner[]
  baseDeliveryFee Int   @default(100) // NPR
}

// ─── PRODUCTS ───────────────────────────────────────────────────────────────

model Category {
  id          String    @id @default(cuid())
  name        String
  nameNe      String
  slug        String    @unique
  icon        String?   // emoji or icon name
  imageUrl    String?
  sortOrder   Int       @default(0)
  isActive    Boolean   @default(true)
  products    Product[]
}

model Product {
  id            String    @id @default(cuid())
  name          String
  nameNe        String?
  slug          String    @unique
  description   String?
  descriptionNe String?
  
  categoryId    String
  category      Category  @relation(fields: [categoryId], references: [id])
  
  images        ProductImage[]
  variants      ProductVariant[]
  
  // Availability per city — product may not be available everywhere
  cityAvailability ProductCityAvailability[]
  
  tags          String[]  // ["birthday", "anniversary", "flower"]
  occasions     String[]  // ["bday", "valentine", "dashain"]
  
  isFeatured    Boolean   @default(false)
  isActive      Boolean   @default(true)
  
  orderItems    OrderItem[]
  reviews       Review[]
  
  newProductRequestId String?   // If added from partner/user request
  
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  @@index([categoryId])
  @@index([slug])
}

model ProductVariant {
  id            String    @id @default(cuid())
  productId     String
  product       Product   @relation(fields: [productId], references: [id])
  name          String    // "500g", "1kg", "Red Roses 12pc"
  basePrice     Int       // NPR — price platform charges customer
  partnerCost   Int?      // NPR — estimated cost to partner (sourcing)
  stock         String    @default("AVAILABLE") // AVAILABLE | LIMITED | UNAVAILABLE
  orderItems    OrderItem[]
}

model ProductImage {
  id        String   @id @default(cuid())
  productId String
  product   Product  @relation(fields: [productId], references: [id])
  url       String
  alt       String?
  isPrimary Boolean  @default(false)
  sortOrder Int      @default(0)
}

model ProductCityAvailability {
  id        String   @id @default(cuid())
  productId String
  product   Product  @relation(fields: [productId], references: [id])
  cityId    String
  city      City     @relation(fields: [cityId], references: [id])
  isAvailable Boolean @default(true)
  
  @@unique([productId, cityId])
}

// ─── ORDERS ─────────────────────────────────────────────────────────────────

model Order {
  id              String        @id @default(cuid())
  orderNumber     String        @unique  // Human readable: KH-20250601-0042
  
  // Parties
  userId          String?
  user            User?         @relation(fields: [userId], references: [id])
  guestEmail      String?
  guestPhone      String?
  
  partnerId       String?
  partner         Partner?      @relation(fields: [partnerId], references: [id])
  
  cityId          String
  city            City          @relation(fields: [cityId], references: [id])
  
  // Items
  items           OrderItem[]
  
  // Recipient Info
  recipientName   String
  recipientPhone  String
  recipientAddress String?
  recipientLat    Float?        // GPS coordinates if provided
  recipientLng    Float?
  recipientLandmark String?
  deliveryNote    String?       // "It's a surprise, don't tell!"
  
  // Sender Info
  senderName      String
  senderPhone     String
  senderMessage   String?       // Personal message on the card
  
  // Is it a surprise?
  isSurprise      Boolean       @default(false)
  surpriseConfirmedAt DateTime? // When recipient confirmed address
  surpriseToken   String?       // Token sent to recipient via SMS
  
  // Delivery
  deliveryDate    DateTime?     // Requested delivery date
  deliverySlot    String?       // "morning" | "afternoon" | "evening"
  scheduledAt     DateTime?
  deliveredAt     DateTime?
  deliveryProofPhoto String?    // Partner uploads photo after delivery
  
  // Pricing
  subtotal        Int           // NPR (sum of items)
  deliveryFee     Int           // NPR
  discount        Int           @default(0) // NPR (from promo code)
  total           Int           // NPR
  
  // Promo
  promoCode       String?
  promoDiscount   Int           @default(0)
  
  // Payment
  paymentStatus   PaymentStatus @default(PENDING)
  paymentMethod   String?       // "esewa" | "khalti" | "stripe" | "cod" | "fonepay"
  paymentGatewayRef String?     // Gateway transaction ID
  paymentVerifiedAt DateTime?
  
  // Partner payout
  partnerPayout   Int?          // How much partner gets paid (NPR)
  partnerPaidAt   DateTime?
  
  // Status
  status          OrderStatus   @default(PLACED)
  statusHistory   OrderStatusEvent[]
  
  // International sender
  senderCurrency  String        @default("NPR")
  senderAmountPaid Float?       // What they paid in their currency
  exchangeRate    Float?
  
  createdAt       DateTime      @default(now())
  updatedAt       DateTime      @updatedAt
  
  @@index([orderNumber])
  @@index([userId])
  @@index([partnerId])
  @@index([cityId])
  @@index([status])
  @@index([createdAt])
}

model OrderItem {
  id          String         @id @default(cuid())
  orderId     String
  order       Order          @relation(fields: [orderId], references: [id])
  productId   String
  product     Product        @relation(fields: [productId], references: [id])
  variantId   String?
  variant     ProductVariant? @relation(fields: [variantId], references: [id])
  
  quantity    Int            @default(1)
  unitPrice   Int            // NPR at time of order
  totalPrice  Int
  
  customNote  String?        // e.g. "Write 'Happy Birthday Ram' on cake"
}

model OrderStatusEvent {
  id          String      @id @default(cuid())
  orderId     String
  order       Order       @relation(fields: [orderId], references: [id])
  status      OrderStatus
  note        String?
  actorType   String?     // "system" | "partner" | "admin" | "customer"
  actorId     String?
  metadata    Json?       // e.g. GPS coordinates at the time
  createdAt   DateTime    @default(now())
  
  @@index([orderId])
}

enum OrderStatus {
  PLACED              // Order placed, awaiting payment
  PAYMENT_PENDING     // Payment initiated but not confirmed
  PAYMENT_CONFIRMED   // Payment verified
  PARTNER_NOTIFIED    // Local partner notified
  PARTNER_ACCEPTED    // Partner accepted order
  ITEMS_SOURCED       // Partner sourced/prepared items
  OUT_FOR_DELIVERY    // Partner heading to deliver
  DELIVERY_ATTEMPTED  // Tried, recipient unavailable
  DELIVERED           // Successfully delivered
  DELIVERY_FAILED     // Could not deliver
  CANCELLED           // Cancelled by customer/admin
  REFUND_INITIATED    // Refund started
  REFUNDED            // Full/partial refund done
}

enum PaymentStatus {
  PENDING
  PROCESSING
  CONFIRMED
  FAILED
  REFUNDED
  PARTIALLY_REFUNDED
}

// ─── PAYMENTS ───────────────────────────────────────────────────────────────

model Payment {
  id              String        @id @default(cuid())
  orderId         String        @unique
  gateway         String        // "esewa" | "khalti" | "stripe" | "fonepay" | "cod"
  amount          Int           // In smallest unit (paisa for NPR, cents for USD)
  currency        String        @default("NPR")
  status          PaymentStatus @default(PENDING)
  gatewayRef      String?       // External transaction ID from gateway
  gatewayResponse Json?         // Full response stored for audit
  verifiedAt      DateTime?
  createdAt       DateTime      @default(now())
  updatedAt       DateTime      @updatedAt
}

model Payout {
  id          String   @id @default(cuid())
  partnerId   String
  partner     Partner  @relation(fields: [partnerId], references: [id])
  orderIds    String[] // Orders included in this payout
  amount      Int      // NPR
  method      String   // "bank_transfer" | "esewa" | "khalti"
  reference   String?
  status      String   @default("PENDING") // PENDING | SENT | CONFIRMED
  paidAt      DateTime?
  createdAt   DateTime @default(now())
}

// ─── ADDRESSES ──────────────────────────────────────────────────────────────

model Address {
  id          String   @id @default(cuid())
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  label       String?  // "Home", "Office"
  fullAddress String
  cityId      String
  landmark    String?
  latitude    Float?
  longitude   Float?
  isDefault   Boolean  @default(false)
  createdAt   DateTime @default(now())
}

// ─── REVIEWS ────────────────────────────────────────────────────────────────

model Review {
  id          String   @id @default(cuid())
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  orderId     String   @unique
  partnerId   String
  partner     Partner  @relation(fields: [partnerId], references: [id])
  productId   String?
  product     Product? @relation(fields: [productId], references: [id])
  
  rating      Int      // 1-5
  comment     String?
  photos      String[] // Customer can upload delivery photos
  isVisible   Boolean  @default(true)
  
  createdAt   DateTime @default(now())
}

// ─── PROMO CODES ─────────────────────────────────────────────────────────────

model PromoCode {
  id            String    @id @default(cuid())
  code          String    @unique
  description   String?
  discountType  String    // "PERCENT" | "FIXED"
  discountValue Int       // Percent or NPR amount
  minOrderValue Int?      // Minimum order total to apply
  maxDiscount   Int?      // Cap on discount
  usageLimit    Int?      // Total uses allowed
  usedCount     Int       @default(0)
  perUserLimit  Int?      // How many times per user
  validFrom     DateTime
  validUntil    DateTime
  isActive      Boolean   @default(true)
  applicableCities String[] // Empty = all cities
  createdAt     DateTime  @default(now())
}

// ─── PRODUCT REQUESTS ────────────────────────────────────────────────────────

model ProductRequest {
  id          String   @id @default(cuid())
  requestedBy String?  // User ID or "anonymous"
  cityId      String?
  productName String
  description String?
  reason      String?
  status      String   @default("PENDING") // PENDING | ADDED | REJECTED
  adminNote   String?
  createdAt   DateTime @default(now())
}

// ─── NOTIFICATIONS ───────────────────────────────────────────────────────────

model Notification {
  id          String   @id @default(cuid())
  userId      String?
  partnerId   String?
  type        String   // "order_placed" | "partner_accepted" | "delivered" etc
  title       String
  body        String
  data        Json?    // Deep link info, order ID, etc
  isRead      Boolean  @default(false)
  sentVia     String[] // ["push", "sms", "email"]
  createdAt   DateTime @default(now())
  
  @@index([userId])
  @@index([partnerId])
}

// ─── ADMIN ───────────────────────────────────────────────────────────────────

model AdminUser {
  id          String   @id @default(cuid())
  email       String   @unique
  name        String
  role        String   @default("MODERATOR") // "SUPER_ADMIN" | "ADMIN" | "MODERATOR" | "SUPPORT"
  passwordHash String
  totpSecret  String?  // 2FA
  lastLoginAt DateTime?
  isActive    Boolean  @default(true)
  createdAt   DateTime @default(now())
}

model AuditLog {
  id          String   @id @default(cuid())
  adminId     String
  action      String   // "APPROVED_KYC" | "SUSPENDED_PARTNER" | "UPDATED_PRODUCT"
  entityType  String   // "partner" | "order" | "product"
  entityId    String
  before      Json?
  after       Json?
  ipAddress   String?
  createdAt   DateTime @default(now())
  
  @@index([adminId])
  @@index([entityType, entityId])
}
```

---

## 4. FOLDER STRUCTURE

### 4.1 Frontend (Next.js)
```
jhilko-web/
├── app/
│   ├── (auth)/
│   │   ├── login/page.tsx           # Phone OTP login
│   │   └── verify/page.tsx          # OTP verification
│   ├── (marketing)/
│   │   ├── page.tsx                 # Landing page / home
│   │   ├── about/page.tsx
│   │   ├── become-partner/page.tsx  # Partner recruitment
│   │   └── cities/page.tsx          # Covered cities
│   ├── (shop)/
│   │   ├── layout.tsx               # Shop layout with nav
│   │   ├── products/
│   │   │   ├── page.tsx             # Product catalog / search
│   │   │   └── [slug]/page.tsx      # Product detail
│   │   ├── categories/
│   │   │   └── [slug]/page.tsx      # Category view
│   │   ├── cart/page.tsx            # Cart
│   │   └── checkout/
│   │       ├── page.tsx             # Checkout form
│   │       └── success/page.tsx     # Order success
│   ├── orders/
│   │   ├── page.tsx                 # Order history
│   │   └── [id]/page.tsx            # Order tracking
│   ├── partner/
│   │   ├── onboard/page.tsx         # Partner registration + KYC
│   │   ├── dashboard/page.tsx       # Partner dashboard
│   │   ├── orders/page.tsx          # Partner order management
│   │   └── earnings/page.tsx        # Partner earnings
│   ├── surprise/
│   │   └── [token]/page.tsx         # Recipient's "confirm address" page
│   ├── admin/
│   │   ├── layout.tsx               # Admin shell
│   │   ├── dashboard/page.tsx
│   │   ├── orders/page.tsx
│   │   ├── partners/
│   │   │   ├── page.tsx
│   │   │   └── [id]/kyc/page.tsx
│   │   ├── products/page.tsx
│   │   ├── cities/page.tsx
│   │   └── analytics/page.tsx
│   ├── api/
│   │   ├── auth/[...nextauth]/route.ts
│   │   ├── webhooks/
│   │   │   ├── esewa/route.ts       # eSewa payment callback
│   │   │   ├── khalti/route.ts      # Khalti payment callback
│   │   │   └── stripe/route.ts      # Stripe webhook
│   │   └── upload/route.ts          # Cloudinary signed upload
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── ui/                          # shadcn/ui components
│   ├── brand/
│   │   ├── Logo.tsx
│   │   └── HeroSection.tsx
│   ├── products/
│   │   ├── ProductCard.tsx          # Animated product card
│   │   ├── ProductGrid.tsx
│   │   ├── ProductSearch.tsx
│   │   └── ProductCatalog.tsx
│   ├── cart/
│   │   ├── CartDrawer.tsx
│   │   └── CartItem.tsx
│   ├── checkout/
│   │   ├── RecipientForm.tsx
│   │   ├── SurpriseToggle.tsx
│   │   ├── LocationPicker.tsx
│   │   ├── PaymentSelector.tsx
│   │   └── OrderSummary.tsx
│   ├── tracking/
│   │   ├── OrderTracker.tsx         # Main tracking component
│   │   ├── StatusTimeline.tsx       # Animated status timeline
│   │   └── LiveMap.tsx              # Partner location map
│   ├── partner/
│   │   ├── KYCForm.tsx
│   │   ├── DocumentUpload.tsx
│   │   └── PartnerDashboard.tsx
│   ├── notifications/
│   │   └── NotificationBell.tsx
│   └── layout/
│       ├── Navbar.tsx
│       ├── BottomNav.tsx            # Mobile bottom nav (critical!)
│       └── Footer.tsx
├── hooks/
│   ├── useAuth.ts
│   ├── useCart.ts
│   ├── useSocket.ts                 # WebSocket for real-time tracking
│   ├── useGeolocation.ts
│   └── useOrderTracking.ts
├── lib/
│   ├── api.ts                       # API client (axios instance)
│   ├── socket.ts                    # Socket.IO client config
│   ├── payments/
│   │   ├── esewa.ts
│   │   ├── khalti.ts
│   │   └── stripe.ts
│   └── utils.ts
├── stores/
│   ├── cartStore.ts
│   ├── authStore.ts
│   └── uiStore.ts
└── types/
    └── index.ts                     # Shared TypeScript types
```

### 4.2 Backend (NestJS)
```
jhilko-api/
├── src/
│   ├── main.ts
│   ├── app.module.ts
│   ├── auth/
│   │   ├── auth.module.ts
│   │   ├── auth.service.ts
│   │   ├── auth.controller.ts
│   │   ├── strategies/
│   │   │   ├── jwt.strategy.ts
│   │   │   └── otp.strategy.ts
│   │   └── guards/
│   ├── users/
│   ├── partners/
│   │   ├── partners.module.ts
│   │   ├── partners.service.ts
│   │   ├── partners.controller.ts
│   │   └── kyc/
│   │       ├── kyc.service.ts       # KYC verification logic
│   │       └── kyc.controller.ts
│   ├── orders/
│   │   ├── orders.module.ts
│   │   ├── orders.service.ts
│   │   ├── orders.controller.ts
│   │   └── order-status.service.ts  # Status machine
│   ├── products/
│   ├── cities/
│   ├── payments/
│   │   ├── payments.module.ts
│   │   ├── payments.service.ts
│   │   ├── gateways/
│   │   │   ├── esewa.service.ts
│   │   │   ├── khalti.service.ts
│   │   │   ├── stripe.service.ts
│   │   │   └── fonepay.service.ts
│   │   └── escrow.service.ts       # Hold funds until delivery
│   ├── notifications/
│   │   ├── sms.service.ts          # Sparrow SMS + Twilio
│   │   ├── email.service.ts        # Resend
│   │   ├── push.service.ts         # Firebase FCM
│   │   └── viber.service.ts        # Viber messages (Nepal)
│   ├── tracking/
│   │   ├── tracking.module.ts
│   │   └── tracking.gateway.ts     # WebSocket gateway
│   ├── admin/
│   ├── upload/
│   │   └── cloudinary.service.ts
│   ├── queues/
│   │   ├── order.processor.ts      # BullMQ job processors
│   │   └── notification.processor.ts
│   ├── analytics/
│   └── prisma/
│       ├── prisma.module.ts
│       └── prisma.service.ts
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
└── test/
```

---

## 5. AUTHENTICATION STRATEGY

### 5.1 User Auth (Phone OTP Primary)
```
1. User enters phone number
2. Backend sends 6-digit OTP via Sparrow SMS (Nepal) 
3. OTP valid for 5 minutes, stored in Redis: otp:{phone} = {code, attempts}
4. On verify: issue JWT access token (15min) + refresh token (30 days)
5. Refresh token stored in Redis + httpOnly cookie
6. Guest checkout: user provides email OR phone → auto-create guest account
   - Guest can view order status without full account
   - On next visit with same phone → merge guest → real account
```

### 5.2 Partner Auth (Full Auth Required)
```
1. Partner registers with phone + email
2. Full KYC document upload required
3. Admin reviews KYC (1-3 business days)
4. On approval: partner gets login credentials + onboarding email
5. Partner uses email + password (strong, bcrypt)
6. 2FA optional (TOTP via Google Authenticator)
```

### 5.3 Admin Auth
```
1. Email + password (very strong requirement, bcrypt hash)
2. TOTP 2FA mandatory
3. IP allowlist (optional but recommended)
4. Session: JWT with short expiry (4 hours) + audit log every action
```

---

## 6. PAYMENT INTEGRATION

### 6.1 Nepal Domestic Payments
```javascript
// eSewa Integration (most users)
// Test: merchant_id = EPAYTEST, secret = 8gBm/:&EnhH.1/q
const initiateESewa = async (order: Order) => {
  const hash = generateHMAC(`total_amount=${order.total},transaction_uuid=${order.id},product_code=${ESEWA_MERCHANT}`)
  return {
    url: "https://rc-epay.esewa.com.np/api/epay/main/v2/form",  // Test
    // "https://epay.esewa.com.np/api/epay/main/v2/form"         // Production
    params: {
      amount: order.subtotal,
      tax_amount: 0,
      total_amount: order.total,
      transaction_uuid: order.id,
      product_code: process.env.ESEWA_MERCHANT_ID,
      product_service_charge: 0,
      product_delivery_charge: order.deliveryFee,
      success_url: `${APP_URL}/checkout/success`,
      failure_url: `${APP_URL}/checkout/failed`,
      signed_field_names: "total_amount,transaction_uuid,product_code",
      signature: hash
    }
  }
}

// Khalti Integration (developer-friendly, growing)
// Test: Key = test_secret_key_f59e8b7d18b4499ca40f68195a846e9b
const initiateKhalti = async (order: Order) => {
  const response = await axios.post(
    "https://a.khalti.com/api/v2/epayment/initiate/",  // Test
    // "https://khalti.com/api/v2/epayment/initiate/"  // Production
    {
      return_url: `${APP_URL}/checkout/success`,
      website_url: APP_URL,
      amount: order.total * 100,  // In paisa!
      purchase_order_id: order.id,
      purchase_order_name: `Jhilko Order #${order.orderNumber}`,
      customer_info: { name, email, phone }
    },
    { headers: { Authorization: `Key ${process.env.KHALTI_SECRET_KEY}` } }
  )
  return response.data.payment_url
}
```

### 6.2 International Payments — REALITY CHECK (June 2026)

> ⚠️ **Stripe is NOT available to Nepal-registered companies.** Nepal is not on Stripe's
> supported-countries list, and direct Stripe merchant accounts cannot be opened from Nepal.
> NRB has announced policy intent to legalize international gateways, but nothing is live yet.
> Any code that assumes a direct Stripe account is wrong. Do not build it.

**Phased strategy for diaspora payments:**

```
PHASE 1 (MVP — launch): Domestic only.
  eSewa + Khalti + COD. Diaspora users can still order if they have an
  eSewa/Khalti wallet (many do via family) — don't block them, just don't
  promise foreign-card support yet.

PHASE 2 (after traction): International cards WITHOUT a foreign entity:
  Option A — Khalti international card processing (Khalti has a Stripe-powered
             inbound product; funds settle in NPR to the Khalti merchant wallet).
             Confirm current merchant API terms with Khalti BD before building.
  Option B — Card acquiring via a Nepali bank (Himalayan Bank, NIC Asia offer
             Visa/Mastercard e-commerce acquiring, ~2.5–3.5%). Slower paperwork,
             fully NRB-compliant.

PHASE 3 (at scale, if diaspora volume justifies it): Foreign entity
  (Singapore/Delaware via Stripe Atlas) + real Stripe account + NRB-compliant
  repatriation. Requires a CA + lawyer. Only do this with proven demand.
```

**Implementation note:** keep a `PaymentGateway` interface (`initiate()`, `verify()`,
`refund()`) so eSewa/Khalti/COD/bank-acquirer/Stripe are swappable adapters. The Stripe
adapter stays unwritten until Phase 3.

### 6.3 Escrow Model (Critical for Trust)
```
Flow:
1. Customer pays → money held in platform account (NOT released to partner yet)
2. Partner delivers → uploads photo proof + recipient OTP confirmation
3. System waits 2 hours (dispute window)
4. Auto-release payment to partner after 2 hours if no dispute
5. If dispute raised → admin reviews → manual release/refund

Never pay partner before delivery is confirmed.
```

### 6.4 Currencies Supported (phased)
| Phase | Currency | Gateway | Commission |
|-------|----------|---------|------------|
| 1 (MVP) | NPR | eSewa, Khalti, Fonepay, COD | 1.5–2.5% (COD: 0%) |
| 2 | USD/GBP/AUD/EUR/AED (card) | Khalti international OR Nepali bank acquirer | ~2.9–3.5% + FX spread |
| 3 (scale) | Full multi-currency | Stripe via foreign entity | 2.9% + fixed |

Display prices in the sender's currency (daily exchange-rate feed, §27) but **always settle
and store in NPR integer paisa**. Add a 4–5% buffer on displayed FX conversions in Phase 2
to cover gateway FX spread.

---

## 7. PARTNER KYC SYSTEM

### 7.1 KYC Document Requirements
```
Required (all mandatory before approval):
1. Citizenship Certificate — front photo
2. Citizenship Certificate — back photo  
3. Selfie holding citizenship beside face (liveness check)
4. Bank account details (for payout)
5. Phone number verified via OTP
6. Emergency contact (name + phone)

Strongly recommended:
7. Ward-level reference (local ward office stamp preferred)

Optional (for high-value categories):
8. Criminal background check reference (self-declaration)
```

### 7.2 KYC Verification Flow
```
Partner submits documents via web form
        ↓
Auto-checks run:
  - Image quality check (not blurry, full document visible)
  - Phone OTP verified
  - Duplicate citizenship number check (DB)
        ↓
Status: UNDER_REVIEW (admin notified via Slack/email)
        ↓
Admin reviews (target: within 48 hours)
  - Visual document verification
  - Call partner to confirm identity (important for Nepal)
  - Cross-check citizenship number if possible
        ↓
APPROVED → Welcome SMS + onboarding call
REJECTED → Specific rejection reason sent
```

### 7.3 Partner Agreement (Digital Signing)
```
The partner agreement must cover (draft with a Nepali lawyer):
1. Non-refundable deposit by partner (optional, Rs 1000-2000 commitment fee) 
2. Code of conduct (no tampering with items, professional behavior)
3. Liability for failed deliveries
4. Photo proof requirement
5. Payment terms (payout weekly/biweekly)
6. Platform's right to suspend/terminate
7. Non-solicitation of customers directly
8. Data privacy (customer data cannot be shared)
9. Governing law: Nepal
10. Dispute resolution: Kathmandu District Court

Use DocuSign-style digital signature or build simple "I Agree + timestamp + IP" into the form.
```

### 7.4 Partner Performance Scoring
```
Score = (Delivery Success Rate × 0.5) + (Avg Rating × 0.3) + (Response Time Score × 0.2)

Actions on score:
< 3.0:  Warning email + 30-day improvement period
< 2.0:  Auto-suspension + admin review
> 4.5:  "Top Partner" badge + higher commission (%)
```

---

## 8. REAL-TIME DELIVERY TRACKING

### 8.1 WebSocket Architecture
```
Customer opens order tracking page
        ↓
Frontend subscribes to Socket.IO room: order:{orderId}
        ↓
Partner app (PWA) sends GPS location every 15-30 seconds:
  socket.emit('location_update', { lat, lng, orderId, timestamp })
        ↓
Backend WebSocket Gateway:
  - Receives location update from partner
  - Stores in Redis: order:{orderId}:location = { lat, lng, timestamp }
  - Broadcasts to room: order:{orderId}
        ↓
Customer sees live pin movement on map
```

### 8.2 Order Status Machine
```typescript
// State transitions (backend enforces these — no skipping)
const VALID_TRANSITIONS = {
  PLACED:             ['PAYMENT_PENDING', 'PAYMENT_CONFIRMED', 'CANCELLED'],
  PAYMENT_PENDING:    ['PAYMENT_CONFIRMED', 'PAYMENT_FAILED'],
  PAYMENT_CONFIRMED:  ['PARTNER_NOTIFIED', 'CANCELLED'],
  PARTNER_NOTIFIED:   ['PARTNER_ACCEPTED', 'CANCELLED'],
  PARTNER_ACCEPTED:   ['ITEMS_SOURCED', 'CANCELLED'],
  ITEMS_SOURCED:      ['OUT_FOR_DELIVERY'],
  OUT_FOR_DELIVERY:   ['DELIVERED', 'DELIVERY_ATTEMPTED', 'DELIVERY_FAILED'],
  DELIVERY_ATTEMPTED: ['OUT_FOR_DELIVERY', 'DELIVERY_FAILED'],
  DELIVERED:          ['REFUND_INITIATED'],  // Only if dispute
  DELIVERY_FAILED:    ['REFUND_INITIATED'],
  REFUND_INITIATED:   ['REFUNDED'],
}
```

### 8.3 Notification Triggers
```
ORDER_PLACED        → Sender: "Order confirmed! #KH-XXXX"
                    → Partner: "New order in your city!"
PARTNER_ACCEPTED    → Sender: "Your partner accepted. Delivery soon."
OUT_FOR_DELIVERY    → Sender: SMS + push "Delivery is on the way!"
                    → Recipient: SMS "Someone sent you a gift! Delivery in X mins"
DELIVERED           → Sender: "Delivered! 🎉 See photo proof"
                    → Auto-release partner payment after 2 hours
DELIVERY_FAILED     → Sender: "Delivery failed. We'll retry or refund."
```

---

## 9. SURPRISE DELIVERY FLOW

### 9.1 The "Surprise" Problem (Critical UX)
When sender selects "It's a surprise! 🎁":

```
1. Sender checks "Surprise mode" in checkout
2. Sender enters recipient phone number
3. System generates unique surprise token (UUID)
4. SMS sent to recipient (from masked number):
   "Someone special sent you a surprise gift! 🎁
    To receive it, confirm your delivery address:
    [https://jhilko.com/surprise/TOKEN]
    Reply STOP to decline."

5. Recipient opens link → enters delivery address OR shares GPS
6. Sender gets notification: "Your recipient confirmed their address! ✅"
7. Order proceeds to partner

FALLBACK if no response in 6 hours:
→ Sender notified → Asked to provide address directly
→ OR: Partner calls recipient number (with masking if possible)

LOCATION FALLBACK if no address:
→ Partner uses nearest famous landmark as delivery guide
→ Partner calls recipient: "I have a delivery for you near [landmark]"
```

---

## 10. PRODUCT CATALOG SYSTEM

### 10.1 Category Structure
```
├── 🎂 Cakes & Baked Goods
│   ├── Birthday Cakes
│   ├── Custom/Photo Cakes
│   ├── Cup Cakes
│   └── Pastries & Sweets
├── 💐 Flowers & Bouquets  
│   ├── Rose Bouquets
│   ├── Mixed Flowers
│   └── Indoor Plants
├── 🍫 Chocolates & Sweets
│   ├── Chocolates (Cadbury, Ferrero, etc.)
│   ├── Mithai & Ladoo (local sweets)
│   └── Dry Fruits Hampers
├── 🧴 Cosmetics & Beauty
│   ├── Skincare Sets
│   └── Makeup Kits
├── 🎁 Gift Hampers
│   ├── Birthday Combos
│   ├── Festival Hampers (Dashain, Tihar)
│   └── New Baby Gifts
├── 💌 Personalized Items
│   ├── Custom Mugs
│   ├── Photo Frames
│   └── Cushions
├── 🍎 Fruits & Healthy
├── 📚 Books
├── 👗 Clothing & Accessories
└── 🎉 Party Supplies
```

### 10.2 New Product Request System
```
Partners can request new products they can source locally:
  "I can source X in my city, please add it"

Users can request:
  "I want to send Y but it's not listed"

Admin reviews → adds to catalog with city availability mapping
```

### 10.3 Delivery Fee Calculation
```javascript
const calculateDeliveryFee = (city: City, zone: Zone, items: CartItem[]) => {
  const baseZoneFee = zone.baseDeliveryFee  // NPR 80-150
  const bulkMultiplier = items.length > 3 ? 1.2 : 1.0  // 20% extra for many items
  const urgencyMultiplier = isUrgent ? 1.5 : 1.0        // 50% for express
  
  return Math.ceil(baseZoneFee * bulkMultiplier * urgencyMultiplier)
}

// Partner commission from delivery fee
// Platform takes: 30% of delivery fee
// Partner gets:   70% of delivery fee + their product sourcing margin

// Product pricing:
// Partner buys: NPR 500 (wholesale local price)
// Platform sells: NPR 650 (30% markup)  
// Partner gets: NPR 550 (10% profit on item)
// Platform keeps: NPR 100 (margin)
```

---

## 11. ADMIN PANEL SPECIFICATION

### 11.1 Dashboard KPIs (Real-time)
```
- Orders today / this week / this month
- Revenue today / MTD / YTD
- Active partners (online now)
- Pending KYC reviews
- Failed deliveries (requires action)
- Cities covered / total partners
- Top selling products
- Geographic heatmap of orders
```

### 11.2 Admin Modules
```
Orders:
  - Full order list with filters (status, city, date, payment)
  - Manual status updates (for edge cases)
  - Assign/reassign partner
  - Refund initiation
  - View payment proof
  - Customer support notes

Partners:
  - KYC review queue (see all UNDER_REVIEW)
  - Approve/Reject with note
  - View full partner profile + history
  - Suspend partner
  - Manual payout trigger
  - Performance dashboard per partner

Products:
  - Add/edit/deactivate products
  - Manage city availability (which cities stock which items)
  - Price management
  - Featured product management
  - Product request review

Cities & Coverage:
  - Activate/deactivate cities
  - View partners per city
  - Set delivery fees per zone
  - Coverage radius management

Finance:
  - Revenue breakdown
  - Partner payout management
  - Payment gateway reconciliation
  - Export to CSV/Excel

Notifications:
  - Broadcast push/SMS to all users or segment
  - Festival/promotional campaigns
  - System-wide announcements

Settings:
  - Delivery fee configuration
  - Commission rates
  - Promo code creation
  - Supported currencies + exchange rates
  - App version management
```

---

## 12. NOTIFICATIONS IMPLEMENTATION

### 12.1 SMS (Sparrow SMS — Nepal)
```javascript
// Sparrow SMS API (most popular SMS gateway in Nepal)
const sendSMS = async (to: string, message: string) => {
  await axios.post('https://api.sparrowsms.com/v2/sms/', {
    token: process.env.SPARROW_SMS_TOKEN,
    from: 'Jhilko',  // Sender ID (6-11 chars, pre-registered)
    to,
    text: message
  })
}

// OTP template (Nepali): "Tapainko OTP: XXXXXX. 5 minute samma valid cha."
// Delivery alert: "Tapainko gift OUT FOR DELIVERY cha! Order #XX herna: jhilko.com/orders/XX"
```

### 12.2 Viber (Very popular in Nepal)
```
Viber Business Messages API for Nepal market
- 70%+ of Nepali smartphone users have Viber
- Free for business messages to opted-in users
- Rich messages with images possible
- Use for delivery confirmations + surprise notifications
```

### 12.3 Push Notifications (Firebase FCM)
```javascript
// For installed PWA (Web Push) + future native app
await firebaseAdmin.messaging().send({
  token: userFCMToken,
  notification: {
    title: "Your delivery is on the way! 🛵",
    body: `${partnerName} is heading to ${recipientName}`
  },
  data: {
    orderId,
    action: 'VIEW_TRACKING'
  },
  webpush: {
    notification: {
      icon: '/icon-192.png',
      badge: '/badge-72.png',
      vibrate: [200, 100, 200]
    }
  }
})
```

### 12.4 Email (Resend)
```javascript
// Order confirmation email with beautiful HTML template
// Include: order summary, tracking link, partner info
// Personalized message card preview
// Festival-specific templates (Dashain, Tihar, Valentine's)
```

---

## 13. MAPS & LOCATION

### 13.1 Baato Maps Integration (Nepal-local — replaces Google Maps)

> **Why Baato over Google Maps:** Baato (baato.io) is Nepali-built, OSM-based, ~70% cheaper
> than Google (free up to ~200,000 map loads/month), and has BETTER local data — gallis,
> landmarks, toles that Google Maps misses entirely. Landmark-based addressing matters in
> Nepal where street names barely exist. Galli Maps is the backup vendor if Baato terms change.
> Keep a thin `MapsProvider` abstraction so swapping vendors is a one-file change.

```javascript
// Frontend: MapLibre GL JS (open-source renderer) + Baato vector tiles
// Baato REST APIs: Search, Reverse Geocode, Places, Directions
// npm: maplibre-gl + @baatomaps/baato-js-client

// Used for:
// 1. Recipient address input (Baato Search API — Nepali + English queries)
// 2. GPS location sharing (Geolocation API → Baato Reverse Geocode)
// 3. Order tracking (MapLibre map + live partner pin via Socket.IO)
// 4. Partner navigation (deep link — Google Maps app is still fine for NAVIGATION,
//    it's the embedded JS API billing we're avoiding)
// 5. City coverage area display

const LocationPicker = () => {
  // Option 1: Search by place/landmark (Baato Search API)
  // Option 2: "Use my current location" button
  // Option 3: Tap on map to drop pin
  // Option 4: Enter landmark manually (free text — critical for rural Nepal)
  // Option 5: Choose from saved addresses
}

// Deep link for partner navigation (free — opens the native app, no API billing)
const getNavLink = (lat: number, lng: number) =>
  `https://www.google.com/maps/dir/?api=1&destination=${lat},${lng}&travelmode=driving`
```

### 13.2 GPS-to-City Mapping
```javascript
// Reverse geocode GPS coordinates → determine which city/zone
// This auto-selects the correct city when user drops pin
const getCityFromCoords = async (lat: number, lng: number) => {
  const res = await fetch(
    `https://api.baato.io/api/v1/reverse?key=${BAATO_KEY}&lat=${lat}&lon=${lng}`
  )
  const { data } = await res.json()
  // data[0].address contains municipality/district — match against our City table
  // Fallback: nearest active City within coverageRadius using simple haversine
}
```

---

## 14. MOBILE-FIRST UI PATTERNS

### 14.1 Bottom Navigation (Critical — 99% Mobile)
```tsx
// Bottom nav: always visible on mobile
// Items: Home | Explore | Cart | Orders | Profile
// Active item: animated indicator (sliding underline or filled circle)
// Haptic feedback on tap (navigator.vibrate([10]))
// Safe area padding for notched phones (env(safe-area-inset-bottom))
```

### 14.2 Apple Vision UI Patterns
```
Cards:
  - backdrop-filter: blur(20px) saturate(1.8)
  - background: rgba(255,255,255,0.72)
  - border: 1px solid rgba(255,255,255,0.3)
  - border-radius: 24px
  - box-shadow: 0 8px 32px rgba(0,0,0,0.08)

Modals:
  - Full-screen overlay on mobile (not small popup)
  - Drag-to-dismiss gesture (useGesture from @use-gesture/react)
  - Spring animation: scale(0.94) → scale(1) + opacity 0 → 1

Product Cards:
  - Large image (60% of card height)
  - Price badge (pill shape, brand color)
  - Quick-add button (animated +)
  - Parallax on scroll (subtle, ~10% depth)

Hero Sections:
  - Full-viewport height
  - Video background or gradient mesh
  - Large typography (56px+ on mobile)
  - CTA button with shimmer animation
  - Scroll-triggered reveals (Intersection Observer)
```

### 14.3 Delivery Tracker UI
```tsx
// Full-screen animated order tracker
// Shows:
// 1. Animated progress bar (custom SVG path)  
// 2. Status icons with checkmarks (animate in sequence)
// 3. Partner photo + name card
// 4. Live map with bouncing delivery pin
// 5. Estimated time remaining (countdown)
// 6. "Call Partner" button (with partner phone masking)

// Status step animation: each step fades in + check mark draws itself
// The progress line "fills up" using SVG stroke-dashoffset animation
```

---

## 15. SEO & PERFORMANCE

```
Next.js App Router built-ins:
- Server Components for product catalog (fast TTFB, good for SEO)
- generateStaticParams for popular products + cities (fully static)
- Metadata API for OG tags per product/category
- next/image for all product photos (WebP, lazy load, blur placeholder)

Target Core Web Vitals:
- LCP < 2.5s (large product image)
- FID/INP < 100ms (cart interactions)  
- CLS < 0.1 (reserve space for images)

URLs:
  /products/birthday-cake-kathmandu      (city-specific product pages)
  /cities/dharan/birthday-gifts          (city + occasion pages)
  /send-gifts-to-nepal-from-usa          (diaspora landing pages)
```

---

## 16. ENVIRONMENT VARIABLES

```bash
# .env.local

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_API_URL=http://localhost:4000

# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/jhilko_dev

# Redis (Upstash)
UPSTASH_REDIS_URL=redis://...
UPSTASH_REDIS_TOKEN=...

# Auth
JWT_SECRET=...
NEXTAUTH_SECRET=...

# Payments — Nepal
ESEWA_MERCHANT_ID=EPAYTEST
ESEWA_SECRET_KEY=8gBm/:&EnhH.1/q

KHALTI_PUBLIC_KEY=test_public_key_...
KHALTI_SECRET_KEY=test_secret_key_...

FONEPAY_MERCHANT_CODE=...        # Phase 2 — skip for MVP
FONEPAY_SECRET_KEY=...           # Phase 2 — skip for MVP

# Payments — International (Phase 2/3 ONLY — leave unset for MVP, see §6.2)
# KHALTI_INTL_* or bank acquirer keys (Phase 2)
# STRIPE_* keys only exist after foreign entity (Phase 3)

# Storage
CLOUDINARY_CLOUD_NAME=...
CLOUDINARY_API_KEY=...
CLOUDINARY_API_SECRET=...

# SMS
SPARROW_SMS_TOKEN=...

# Email
RESEND_API_KEY=re_...

# Push
FIREBASE_SERVICE_ACCOUNT_JSON=...
NEXT_PUBLIC_FIREBASE_CONFIG=...

# Maps (Baato — baato.io dashboard)
NEXT_PUBLIC_BAATO_ACCESS_TOKEN=...

# Monitoring
SENTRY_DSN=...
NEXT_PUBLIC_SENTRY_DSN=...
```

---

## 17. CODING CONVENTIONS

```typescript
// ✅ Use server actions for mutations (Next.js App Router)
// ✅ Use React Query for data fetching + caching
// ✅ Always validate with Zod on both frontend and backend
// ✅ TypeScript strict mode — no `any`
// ✅ All currency values in INTEGER (paisa/paise, not float)
// ✅ All dates stored in UTC, displayed in Nepal time (UTC+5:45)
// ✅ All amounts in NPR unless explicitly noted
// ✅ Snake_case in DB, camelCase in code, kebab-case in URLs
// ✅ Phone numbers: normalize to +977XXXXXXXXXX format
// ✅ Always log security events (KYC changes, payment events, auth attempts)
// ❌ Never store raw payment credentials in DB
// ❌ Never expose partner phone numbers directly to customers (use masking)
// ❌ Never skip payment verification (always verify via gateway API, not just webhook)
```

---

## 18. TESTING STRATEGY

```
Unit tests:      Vitest (fast, ESM-compatible)
Integration:     Playwright (E2E, test payment flows with test cards)
API tests:       Supertest + Jest for NestJS
Coverage target: 60%+ on payment + order logic

Key test cases:
- Payment success + webhook verification
- Payment failure + order status rollback
- Partner KYC approval/rejection flow
- Surprise delivery token expiry
- Order cancellation + refund
- Admin panel RBAC (role access)
```

---

## 19. LAUNCH CHECKLIST

```
□ All payment gateways in test mode → test full flow
□ Sparrow SMS test messages working
□ KYC upload to Cloudinary tested
□ Partner mobile UI tested on real Android device (not simulator)
□ Surprise SMS link works on Viber + SMS
□ eSewa → Khalti → COD all tested end-to-end with real NPR (test NPR 10 orders)
□ Rate limiting on OTP endpoint (max 3 per hour per phone)
□ Admin 2FA working
□ Error monitoring (Sentry) catching real errors
□ Privacy policy + Terms accessible via footer
□ Contact form working (goes to admin email)
□ PWA install banner tested
□ Google Search Console submitted
□ Facebook Pixel installed (for retargeting ads)
□ TikTok Pixel installed (critical for viral marketing)
□ Partner onboarding flow tested end-to-end
□ Backup: manual order management via admin if app fails
```

---

*Last updated: June 2025 | v1.0 | Owner: Founder*

---

## 20. STANDARD API RESPONSE ENVELOPE

Every single endpoint returns the same shape. No exceptions. Frontend can write one interceptor and handle all responses uniformly.

```typescript
// shared/types/response.ts
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: { code: string; message: string; field?: string }
  meta?: { page?: number; limit?: number; total?: number; hasMore?: boolean }
}

// ✅ Success
{ success: true, data: { order: { id: "...", status: "DELIVERED" } } }

// ✅ Paginated success
{ success: true, data: [...products], meta: { page: 1, limit: 20, total: 143, hasMore: true } }

// ✅ Error
{ success: false, error: { code: "ORDER_NOT_FOUND", message: "Order not found" } }

// ✅ Validation error (field-level)
{ success: false, error: { code: "VALIDATION_ERROR", message: "Invalid phone", field: "phone" } }
```

### 20.1 Error Code Registry
```typescript
// shared/constants/error-codes.ts
export const ERROR_CODES = {
  // Auth
  AUTH_INVALID_OTP:           'Invalid or expired OTP',
  AUTH_TOO_MANY_ATTEMPTS:     'Too many OTP attempts. Try again in 1 hour',
  AUTH_TOKEN_EXPIRED:         'Session expired. Please log in again',
  AUTH_UNAUTHORIZED:          'You do not have permission to do this',
  AUTH_PARTNER_KYC_PENDING:   'Your KYC is under review. We will notify you once approved',
  AUTH_PARTNER_SUSPENDED:     'Your partner account has been suspended',

  // Orders
  ORDER_NOT_FOUND:            'Order not found',
  ORDER_CITY_NOT_COVERED:     'We do not deliver to this city yet',
  ORDER_INVALID_TRANSITION:   'This status change is not allowed',
  ORDER_PAYMENT_REQUIRED:     'Complete payment before proceeding',
  ORDER_ALREADY_CANCELLED:    'This order has already been cancelled',
  ORDER_CANNOT_CANCEL:        'Order cannot be cancelled at this stage',

  // Products
  PRODUCT_NOT_FOUND:          'Product not found',
  PRODUCT_UNAVAILABLE:        'This product is not available in the selected city',

  // Partners
  PARTNER_NOT_FOUND:          'Partner not found',
  PARTNER_UNAVAILABLE:        'No partners available in this city right now. Try again soon.',
  PARTNER_DUPLICATE:          'A partner with this phone number already exists',

  // Payments
  PAYMENT_FAILED:             'Payment could not be processed. Please try again.',
  PAYMENT_ALREADY_COMPLETED:  'This order has already been paid for',
  PAYMENT_WEBHOOK_INVALID:    'Invalid payment callback signature',
  PAYMENT_IDEMPOTENCY_REQUIRED: 'Idempotency key is required for payment requests',

  // Validation
  VALIDATION_ERROR:           'Please check your input',
  PHONE_INVALID:              'Enter a valid Nepal phone number (98XXXXXXXX)',

  // Generic
  NOT_FOUND:                  'Resource not found',
  SERVER_ERROR:               'Something went wrong. Our team has been notified.',
  RATE_LIMITED:               'Too many requests. Please slow down.',
  DUPLICATE_ENTRY:            'This record already exists',
} as const
```

### 20.2 Global Exception Filter (NestJS)
```typescript
// common/filters/global-exception.filter.ts
import { ExceptionFilter, Catch, ArgumentsHost, HttpException, HttpStatus } from '@nestjs/common'
import { Prisma } from '@prisma/client'
import * as Sentry from '@sentry/node'

@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx   = host.switchToHttp()
    const res   = ctx.getResponse()
    const req   = ctx.getRequest()

    let status  = HttpStatus.INTERNAL_SERVER_ERROR
    let code    = 'SERVER_ERROR'
    let message = 'Something went wrong. Our team has been notified.'
    let field: string | undefined

    if (exception instanceof HttpException) {
      status = exception.getStatus()
      const body = exception.getResponse() as any
      code    = body.code    ?? code
      message = Array.isArray(body.message) ? body.message[0] : (body.message ?? message)
      field   = body.field
    }

    else if (exception instanceof Prisma.PrismaClientKnownRequestError) {
      if (exception.code === 'P2002') { status = 409; code = 'DUPLICATE_ENTRY'; message = 'This record already exists' }
      if (exception.code === 'P2025') { status = 404; code = 'NOT_FOUND';       message = 'Record not found' }
    }

    if (status >= 500) Sentry.captureException(exception, { extra: { url: req.url, body: req.body } })

    res.status(status).json({
      success: false,
      error: { code, message, ...(field ? { field } : {}) },
      timestamp: new Date().toISOString(),
      path: req.url,
    })
  }
}
```

### 20.3 Response Interceptor (wraps all success responses)
```typescript
// common/interceptors/response.interceptor.ts
@Injectable()
export class ResponseInterceptor<T> implements NestInterceptor<T, ApiResponse<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<ApiResponse<T>> {
    return next.handle().pipe(
      map(data => ({ success: true, ...data }))  // data should already be { data: ..., meta?: ... }
    )
  }
}

// In controllers, return:
return { data: order }
return { data: products, meta: { page, limit, total, hasMore } }
// Interceptor adds success: true automatically
```

---

## 21. RATE LIMITING (Per Endpoint)

```typescript
// Install: npm install @nestjs/throttler

// app.module.ts
ThrottlerModule.forRootAsync({
  imports: [RedisModule],
  inject: [REDIS_CLIENT],
  useFactory: (redis) => ({
    throttlers: [{ name: 'default', ttl: 60_000, limit: 60 }],
    storage: new ThrottlerStorageRedisService(redis),  // Distributed — works across multiple servers
  }),
}),

// Endpoint-specific limits
@Throttle({ default: { ttl: 3_600_000, limit: 3 } })   // OTP: 3 per hour
@Post('auth/send-otp') sendOtp() {}

@Throttle({ default: { ttl: 3_600_000, limit: 10 } })  // Login attempts: 10 per hour
@Post('auth/verify-otp') verifyOtp() {}

@Throttle({ default: { ttl: 900_000, limit: 20 } })    // Create order: 20 per 15 min
@Post('orders') createOrder() {}

@Throttle({ default: { ttl: 60_000, limit: 120 } })    // GPS ping: 2 per second
@Post('tracking/ping') gpsPing() {}

@SkipThrottle()                                          // Webhooks: no rate limit
@Post('webhooks/esewa') esewaWebhook() {}
```

---

## 22. SECURITY HARDENING (main.ts bootstrap)

```typescript
// jhilko-api/src/main.ts
import helmet from 'helmet'
import * as morgan from 'morgan'
import * as cookieParser from 'cookie-parser'
import * as compression from 'compression'

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    rawBody: true,  // CRITICAL: needed for raw-body webhook signature verification (eSewa/Khalti/future gateways)
    logger: ['error', 'warn', 'log'],
  })

  // 1. Helmet — all security headers
  app.use(helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc:  ["'self'"],
        scriptSrc:   ["'self'"],
        imgSrc:      ["'self'", 'data:', 'https://res.cloudinary.com'],
        connectSrc:  ["'self'", 'https://api.khalti.com', 'https://epay.esewa.com.np', 'https://api.stripe.com'],
        frameAncestors: ["'none'"],
      },
    },
    hsts: { maxAge: 31_536_000, includeSubDomains: true, preload: true },
    noSniff: true,
    xssFilter: true,
  }))

  // 2. CORS — strict whitelist only
  app.enableCors({
    origin: (origin, cb) => {
      const allowed = [
        'https://jhilko.com.np',
        'https://www.jhilko.com.np',
        'https://admin.jhilko.com.np',
        ...(process.env.NODE_ENV !== 'production' ? ['http://localhost:3000', 'http://localhost:3001'] : []),
      ]
      if (!origin || allowed.includes(origin)) cb(null, true)
      else cb(new Error(`CORS blocked: ${origin}`))
    },
    credentials: true,
    methods: ['GET', 'POST', 'PATCH', 'DELETE', 'OPTIONS'],
    allowedHeaders: ['Authorization', 'Content-Type', 'x-idempotency-key', 'stripe-signature'],
    maxAge: 86_400,
  })

  // 3. Global validation — strip unknown fields, auto-transform
  app.useGlobalPipes(new ValidationPipe({
    whitelist:              true,
    forbidNonWhitelisted:   true,
    transform:              true,
    transformOptions:       { enableImplicitConversion: true },
    disableErrorMessages:   process.env.NODE_ENV === 'production',
    stopAtFirstError:       true,
  }))

  // 4. Global filters + interceptors
  app.useGlobalFilters(new GlobalExceptionFilter())
  app.useGlobalInterceptors(new ResponseInterceptor())

  // 5. Cookie parser (for refresh token httpOnly cookie)
  app.use(cookieParser(process.env.COOKIE_SECRET))

  // 6. Compression
  app.use(compression())

  // 7. Request logging
  app.use(morgan(process.env.NODE_ENV === 'production' ? 'combined' : 'dev'))

  // 8. Body size limit
  app.use(express.json({ limit: '5mb' }))
  app.use(express.urlencoded({ limit: '5mb', extended: true }))

  // 9. API versioning
  app.setGlobalPrefix('api/v1')

  // 10. Swagger (dev + staging only — never production)
  if (process.env.NODE_ENV !== 'production') {
    const config = new DocumentBuilder()
      .setTitle('Jhilko API')
      .setVersion('1.0')
      .addBearerAuth()
      .build()
    SwaggerModule.setup('docs', app, SwaggerModule.createDocument(app, config))
  }

  // 11. Graceful shutdown
  app.enableShutdownHooks()

  // 12. Health check
  app.use('/health', (_, res) => res.json({ status: 'ok', timestamp: new Date() }))

  await app.listen(process.env.PORT ?? 4000)
}
```

---

## 23. PAYMENT WEBHOOK SIGNATURE VERIFICATION

```typescript
// payments/webhooks/esewa.webhook.ts
// NEVER process payment without verifying the signature first

@Post('webhooks/esewa')
@HttpCode(200)
@SkipThrottle()
async esewaCallback(@Req() req: Request, @Body() body: any) {
  // eSewa sends base64-encoded JSON in body.data
  let decoded: any
  try {
    decoded = JSON.parse(Buffer.from(body.data, 'base64').toString('utf-8'))
  } catch {
    throw new BadRequestException({ code: 'PAYMENT_WEBHOOK_INVALID', message: 'Cannot parse eSewa callback' })
  }

  // Rebuild the signed fields string (same as you sent originally)
  const signedFields = decoded.signed_field_names.split(',')
  const message = signedFields.map((f: string) => `${f}=${decoded[f]}`).join(',')

  const expectedHash = createHmac('sha256', process.env.ESEWA_SECRET_KEY!)
    .update(message)
    .digest('base64')

  if (expectedHash !== decoded.signature) {
    // Log attempted spoofing
    this.logger.warn(`eSewa signature mismatch. IP: ${req.ip}. Body: ${JSON.stringify(decoded)}`)
    throw new ForbiddenException({ code: 'PAYMENT_WEBHOOK_INVALID' })
  }

  if (decoded.status !== 'COMPLETE') return { received: true }  // Ignore non-success

  await this.paymentsService.confirmPayment({ gateway: 'esewa', ref: decoded.transaction_code, orderId: decoded.transaction_uuid })
}

// payments/khalti-return.controller.ts
// ⚠️ IMPORTANT: Khalti KPG-2 (ePayment) does NOT send a signed webhook.
// It redirects the user to your return_url with ?pidx=...&status=...
// The query params are UNTRUSTED. The ONLY source of truth is the Lookup API.
// NEVER mark an order paid without a successful lookup.
@Get('payments/khalti/return')
@SkipThrottle()
async khaltiReturn(@Query('pidx') pidx: string, @Query('purchase_order_id') orderId: string) {
  // Server-to-server verification — the only trusted check
  const res = await fetch('https://khalti.com/api/v2/epayment/lookup/', { // test: a.khalti.com
    method: 'POST',
    headers: { Authorization: `Key ${process.env.KHALTI_SECRET_KEY!}`, 'Content-Type': 'application/json' },
    body: JSON.stringify({ pidx }),
  })
  const lookup = await res.json()

  if (lookup.status !== 'Completed') {
    this.logger.warn(`Khalti lookup not completed: ${pidx} → ${lookup.status}`)
    return { received: true } // Pending/Expired/User canceled — leave order PAYMENT_PENDING
  }

  // Also verify amount matches the order total (paisa) — prevents tampering
  await this.paymentsService.confirmPayment({ gateway: 'khalti', ref: lookup.transaction_id, orderId, expectedAmountPaisa: lookup.total_amount })
}

// payments/webhooks/stripe.webhook.ts
// ⚠️ PHASE 3 ONLY — requires a foreign entity (see §6.2). Reference implementation,
// do NOT build or register this route until a real Stripe account exists.
@Post('webhooks/stripe')
@HttpCode(200)
@SkipThrottle()
async stripeCallback(@Req() req: Request, @Headers('stripe-signature') sig: string) {
  // app.create() MUST have rawBody: true for this to work
  let event: Stripe.Event
  try {
    event = this.stripe.webhooks.constructEvent(req.rawBody, sig, process.env.STRIPE_WEBHOOK_SECRET!)
  } catch (err) {
    this.logger.warn(`Stripe signature failed: ${err.message}`)
    throw new ForbiddenException({ code: 'PAYMENT_WEBHOOK_INVALID' })
  }

  if (event.type === 'payment_intent.succeeded') {
    const pi = event.data.object as Stripe.PaymentIntent
    await this.paymentsService.confirmPayment({ gateway: 'stripe', ref: pi.id, orderId: pi.metadata.orderId })
  }
}

// Shared payment confirmation — runs after any gateway confirms
async confirmPayment(dto: { gateway: string; ref: string; orderId: string }) {
  // Idempotency check — process each payment exactly once
  const alreadyProcessed = await this.redis.get(`payment:processed:${dto.ref}`)
  if (alreadyProcessed) return   // Duplicate webhook — ignore

  await this.redis.set(`payment:processed:${dto.ref}`, '1', 'EX', 86_400)  // 24hr

  await this.prisma.$transaction(async (tx) => {
    await tx.payment.update({ where: { orderId: dto.orderId }, data: { status: 'CONFIRMED', gatewayRef: dto.ref, verifiedAt: new Date() } })
    await tx.order.update({ where: { id: dto.orderId }, data: { status: 'PAYMENT_CONFIRMED', paymentStatus: 'CONFIRMED' } })
    await tx.orderStatusEvent.create({ data: { orderId: dto.orderId, status: 'PAYMENT_CONFIRMED', actorType: 'system', note: `Confirmed via ${dto.gateway}` } })
  })

  // Trigger: notify partner of new order
  await this.notificationsQueue.add('order_paid', { orderId: dto.orderId })
}
```

---

## 24. IDEMPOTENCY KEYS (Prevent Double Charges)

```typescript
// common/middleware/idempotency.middleware.ts
@Injectable()
export class IdempotencyMiddleware implements NestMiddleware {
  constructor(@InjectRedis() private redis: Redis) {}

  async use(req: Request, res: Response, next: NextFunction) {
    if (!['POST', 'PATCH'].includes(req.method)) return next()
    if (!req.path.includes('/payments')) return next()  // Only for payment routes

    const key = req.headers['x-idempotency-key'] as string
    if (!key) throw new BadRequestException({ code: 'PAYMENT_IDEMPOTENCY_REQUIRED' })
    if (key.length < 16 || key.length > 64) throw new BadRequestException({ code: 'VALIDATION_ERROR', message: 'Idempotency key must be 16-64 chars' })

    const cached = await this.redis.get(`idempotency:${key}`)
    if (cached) {
      // Return exact same response as original request
      const original = JSON.parse(cached)
      return res.status(original.statusCode).json(original.body)
    }

    // Intercept the response to cache it
    const originalJson = res.json.bind(res)
    res.json = (body: any) => {
      if (res.statusCode < 500) {  // Only cache success responses
        this.redis.set(`idempotency:${key}`, JSON.stringify({ statusCode: res.statusCode, body }), 'EX', 86_400)
      }
      return originalJson(body)
    }

    next()
  }
}

// Frontend — generate key once per checkout, persist in sessionStorage
const getIdempotencyKey = (): string => {
  const existing = sessionStorage.getItem('payment_key')
  if (existing) return existing
  const key = `${Date.now()}-${crypto.randomUUID()}`
  sessionStorage.setItem('payment_key', key)
  return key
}

// On payment success or failure — clear the key so next order gets a fresh one
sessionStorage.removeItem('payment_key')
```

---

## 25. REDIS CACHING STRATEGY

```typescript
// All TTLs in one place — never hardcode seconds in service files
export const CACHE_TTL = {
  PRODUCTS_BY_CITY:   600,     // 10 min  — product catalog changes infrequently
  PRODUCT_DETAIL:     600,     // 10 min
  CATEGORIES:         1_800,   // 30 min
  CITIES_ACTIVE:      3_600,   // 1 hour  — city list rarely changes
  PARTNER_ONLINE:     120,     // 2 min   — partner is/isn't available
  ORDER_STATUS:       60,      // 1 min   — frequently changing
  OTP:                300,     // 5 min   — OTP expiry
  SESSION:            2_592_000, // 30 days — refresh token
  IDEMPOTENCY:        86_400,  // 24 hr
  GPS_LAST_PING:      120,     // 2 min   — last known location
  EXCHANGE_RATES:     21_600,  // 6 hr    — daily-ish updates
  SEARCH_RESULTS:     300,     // 5 min   — cache popular searches
} as const

// Cache-aside pattern (standard for all reads)
async getCachedOrFetch<T>(key: string, ttl: number, fetcher: () => Promise<T>): Promise<T> {
  const cached = await this.redis.get(key)
  if (cached) return JSON.parse(cached)

  const fresh = await fetcher()
  await this.redis.set(key, JSON.stringify(fresh), 'EX', ttl)
  return fresh
}

// Cache invalidation — call these after mutations
async invalidateProductCache(cityId: string, productId: string) {
  await Promise.all([
    this.redis.del(`products:city:${cityId}`),
    this.redis.del(`product:${productId}`),
  ])
}
```

---

## 26. KYC DOCUMENT SECURITY

```typescript
// partners/kyc/kyc-storage.service.ts

// 1. Upload to Cloudinary with AUTHENTICATED type (not public)
async uploadKYCDocument(file: Buffer, mimeType: string, partnerId: string, docType: KYCDocType): Promise<string> {
  return new Promise((resolve, reject) => {
    const stream = cloudinary.uploader.upload_stream(
      {
        folder:       `kyc/${partnerId}`,
        public_id:    `${docType}_${Date.now()}`,
        resource_type: 'image',
        type:         'authenticated',           // NEVER 'upload' — requires signed URL to view
        overwrite:    false,
        tags:         [`partner:${partnerId}`, `doc:${docType}`],
        context:      `partner_id=${partnerId}|doc_type=${docType}|uploaded_at=${Date.now()}`,
        transformation: [
          { quality: 'auto:good' },
          { fetch_format: 'auto' },
          { width: 1200, crop: 'limit' },       // Limit resolution — original not needed
        ],
      },
      (err, result) => err ? reject(err) : resolve(result!.public_id)  // Store public_id, not URL
    )
    stream.end(file)
  })
}

// 2. Generate signed URL — expires in 1 hour, for admin review only
getKYCViewURL(publicId: string): string {
  return cloudinary.url(publicId, {
    type:        'authenticated',
    sign_url:    true,
    expires_at:  Math.floor(Date.now() / 1000) + 3_600,
    resource_type: 'image',
    secure:      true,
  })
}

// 3. Log EVERY access to KYC documents
async viewDocument(adminId: string, partnerId: string, docType: string, ip: string): Promise<string> {
  await this.prisma.auditLog.create({
    data: { adminId, action: 'VIEW_KYC_DOCUMENT', entityType: 'partner', entityId: partnerId, after: { docType, viewedAt: new Date().toISOString() } as any, ipAddress: ip }
  })
  const partner = await this.prisma.partner.findUniqueOrThrow({ where: { id: partnerId } })
  const publicId = partner[`${docType}PublicId` as keyof typeof partner] as string
  return this.getKYCViewURL(publicId)
}

// 4. Hash citizenship number — store hash only, never raw number
hashCitizenshipNumber(raw: string): string {
  // Normalize: trim, uppercase, remove hyphens
  const normalized = raw.trim().toUpperCase().replace(/-/g, '')
  return createHash('sha256').update(normalized + process.env.KYC_HASH_SALT).digest('hex')
}

// 5. Detect duplicate citizenship on submission
async checkDuplicateCitizenship(citizenshipNumber: string): Promise<boolean> {
  const hash = this.hashCitizenshipNumber(citizenshipNumber)
  const existing = await this.prisma.partner.findFirst({ where: { citizenshipHash: hash } })
  return !!existing
}

// 6. Auto-delete documents when partner is inactive for 2 years
@Cron('0 2 1 * *')  // 2am on 1st of every month
async cleanupInactiveKYCDocs() {
  const cutoff = new Date(Date.now() - 2 * 365 * 24 * 60 * 60 * 1000)  // 2 years ago
  const inactive = await this.prisma.partner.findMany({
    where: { isActive: false, updatedAt: { lt: cutoff } },
    select: { id: true, citizenshipFrontPublicId: true, citizenshipBackPublicId: true, selfiePublicId: true }
  })

  for (const partner of inactive) {
    const toDelete = [partner.citizenshipFrontPublicId, partner.citizenshipBackPublicId, partner.selfiePublicId].filter(Boolean)
    await Promise.all(toDelete.map(id => cloudinary.uploader.destroy(id!, { type: 'authenticated' })))
    await this.prisma.partner.update({ where: { id: partner.id }, data: { citizenshipFrontPublicId: null, citizenshipBackPublicId: null, selfiePublicId: null } })
    await this.prisma.auditLog.create({ data: { adminId: 'system', action: 'DELETED_KYC_DOCS_INACTIVE', entityType: 'partner', entityId: partner.id } })
  }
}
```

---

## 27. CURRENCY EXCHANGE RATE FEED

```typescript
// settings/exchange-rate.service.ts

@Injectable()
export class ExchangeRateService {
  private readonly SUPPORTED = ['USD', 'GBP', 'AUD', 'AED', 'SAR', 'QAR', 'EUR', 'INR', 'MYR', 'KRW', 'JPY']

  @Cron('0 6 * * *', { timeZone: 'Asia/Kathmandu' })  // 6 AM NPT daily
  async refreshRates() {
    try {
      // Free tier: 1500 requests/month — daily cron uses 30/month
      const res  = await fetch(`https://api.exchangerate-api.com/v4/latest/NPR`)
      const data = await res.json()

      const rates: Record<string, number> = { NPR: 1 }
      for (const currency of this.SUPPORTED) {
        if (data.rates[currency]) {
          rates[currency] = Math.round((1 / data.rates[currency]) * 100) / 100  // NPR per 1 unit
        }
      }
      rates.updatedAt = Date.now() as any

      await this.redis.set('exchange:rates', JSON.stringify(rates), 'EX', CACHE_TTL.EXCHANGE_RATES)
      await this.prisma.setting.upsert({ where: { key: 'exchange_rates' }, update: { value: JSON.stringify(rates) }, create: { key: 'exchange_rates', value: JSON.stringify(rates) } })
    } catch (err) {
      // On failure: keep using cached rates (they are stored in DB as fallback)
      this.logger.error('Exchange rate refresh failed', err)
      Sentry.captureException(err)
    }
  }

  async getRates(): Promise<Record<string, number>> {
    const cached = await this.redis.get('exchange:rates')
    if (cached) return JSON.parse(cached)

    // Redis miss — load from DB (fallback for cold starts)
    const setting = await this.prisma.setting.findUnique({ where: { key: 'exchange_rates' } })
    return setting ? JSON.parse(setting.value) : { NPR: 1 }
  }

  convertToNPR(amount: number, fromCurrency: string, rates: Record<string, number>): number {
    if (fromCurrency === 'NPR') return amount
    const rate = rates[fromCurrency]
    if (!rate) throw new BadRequestException({ code: 'VALIDATION_ERROR', message: `Unsupported currency: ${fromCurrency}` })
    return Math.ceil(amount * rate)  // Always round up — platform's benefit
  }
}
```

---

## 28. GITHUB ACTIONS CI/CD PIPELINE

```yaml
# .github/workflows/ci-cd.yml
name: CI / CD

on:
  push:
    branches: [main, staging]
  pull_request:
    branches: [main]

jobs:
  # ─── TEST ────────────────────────────────────────────────────────────────
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env: { POSTGRES_PASSWORD: test, POSTGRES_DB: jhilko_test }
        options: --health-cmd "pg_isready" --health-interval 5s --health-retries 5
        ports: ['5432:5432']
      redis:
        image: redis:7-alpine
        options: --health-cmd "redis-cli ping" --health-interval 5s --health-retries 5
        ports: ['6379:6379']

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }

      - name: Install API deps
        run: npm ci --prefix jhilko-api

      - name: Run Prisma migrations
        run: npx prisma migrate deploy
        working-directory: jhilko-api
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/jhilko_test

      - name: Run API tests
        run: npm test -- --passWithNoTests --forceExit
        working-directory: jhilko-api
        env:
          NODE_ENV: test
          DATABASE_URL: postgresql://postgres:test@localhost:5432/jhilko_test
          REDIS_URL: redis://localhost:6379
          JWT_SECRET: ci_test_secret_32_chars_minimum_!
          ESEWA_SECRET_KEY: test_key
          KHALTI_SECRET_KEY: test_key

      - name: Install Web deps
        run: npm ci --prefix jhilko-web

      - name: TypeScript check
        run: npm run type-check --prefix jhilko-web

      - name: Lint
        run: npm run lint --prefix jhilko-web

      - name: Build web
        run: npm run build --prefix jhilko-web
        env:
          NEXT_PUBLIC_API_URL: https://api-staging.jhilko.com.np
          NEXT_PUBLIC_BAATO_ACCESS_TOKEN: ${{ secrets.NEXT_PUBLIC_BAATO_ACCESS_TOKEN }}

  # ─── DEPLOY BACKEND (staging on push to staging branch) ──────────────────
  deploy-api-staging:
    needs: test
    if: github.ref == 'refs/heads/staging'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy API to Railway (staging)
        run: npx @railway/cli deploy --service jhilko-api-staging
        env: { RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }} }

  # ─── DEPLOY FRONTEND (staging) ───────────────────────────────────────────
  deploy-web-staging:
    needs: test
    if: github.ref == 'refs/heads/staging'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          alias-domains: staging.jhilko.com.np

  # ─── DEPLOY PRODUCTION (main branch only) ────────────────────────────────
  deploy-production:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production    # Requires manual approval in GitHub
    steps:
      - uses: actions/checkout@v4

      - name: Run Prisma migrations on production DB
        run: npx prisma migrate deploy
        working-directory: jhilko-api
        env: { DATABASE_URL: ${{ secrets.PRODUCTION_DATABASE_URL }} }

      - name: Deploy API to Railway (production)
        run: npx @railway/cli deploy --service jhilko-api
        env: { RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }} }

      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

---

## 29. SPRINT-BY-SPRINT BUILD ORDER

```
SPRINT 1 — WEEK 1-2: Foundation
□ Monorepo init: /jhilko-api (NestJS) + /jhilko-web (Next.js) + shared /types
□ Prisma schema complete migration applied to Neon DB
□ Redis connected (Upstash)
□ NestJS: all modules scaffolded (empty, but importable)
□ GitHub Actions CI running (test job only)
□ Vercel + Railway preview deployments from staging branch
□ .env.local documented, all keys mapped
□ Health check endpoint: GET /health
GOAL: Both apps deploy without errors. Nothing works yet.

SPRINT 2 — WEEK 3-4: Auth + Catalog
□ NestJS: Phone OTP send (Sparrow SMS)
□ NestJS: OTP verify → issue JWT + refresh token (httpOnly cookie)
□ NestJS: Guest checkout — phone/email → temp account
□ NestJS: Cities list API (with active filter)
□ NestJS: Products by city API (Redis cached)
□ NestJS: Product detail API
□ Next.js: Landing page (static — no data yet)
□ Next.js: City selector → product catalog page
□ Next.js: Product detail + variants
□ Next.js: Auth flow (OTP login modal)
GOAL: User can browse products in a city. Cannot order yet.

SPRINT 3 — WEEK 5-6: Orders + eSewa Payment
□ NestJS: Create order API (recipient info, items, city, message)
□ NestJS: Order state machine (enforced transitions)
□ NestJS: eSewa payment initiation
□ NestJS: eSewa webhook + HMAC signature verify
□ NestJS: Idempotency middleware on payment routes
□ NestJS: Order confirmation SMS (sender + recipient)
□ NestJS: Surprise mode — generate token, SMS to recipient
□ Next.js: Full checkout flow (recipient form → location → payment)
□ Next.js: Surprise mode toggle + surprise token confirm page
□ Next.js: Payment redirect → success/fail screens
□ Next.js: Order status page (no live tracking yet)
GOAL: First real order can be placed and paid. Test with NPR 1.

SPRINT 4 — WEEK 7-8: Partner KYC + Dashboard
□ NestJS: Partner registration API
□ NestJS: Cloudinary signed upload for KYC docs (authenticated type)
□ NestJS: KYC status workflow + notifications on status change
□ NestJS: Partner auth (email + password + bcrypt)
□ NestJS: Partner order list API (orders in their city)
□ NestJS: Partner accept/decline order
□ NestJS: Partner status update (ITEMS_SOURCED, OUT_FOR_DELIVERY)
□ NestJS: Delivery photo upload + recipient OTP confirm
□ NestJS: Escrow release trigger after OTP confirm
□ Next.js: Partner onboarding + KYC form
□ Next.js: Partner dashboard (order cards, accept/decline)
□ Admin panel (basic): KYC review queue — view docs, approve/reject
GOAL: First end-to-end order: customer orders → partner delivers → paid.

SPRINT 5 — WEEK 9-10: Live Tracking + Notifications
□ NestJS: WebSocket gateway (Socket.IO)
□ NestJS: GPS ping endpoint (partner sends lat/lng)
□ NestJS: Broadcast location to order room
□ NestJS: BullMQ notification queue
□ NestJS: All SMS triggers (partner notified, out for delivery, delivered)
□ NestJS: Firebase FCM push notifications
□ NestJS: Viber notifications (optional — if Viber API access obtained)
□ Next.js: Order tracking screen — live map + status timeline
□ Next.js: Animated delivery ring + partner card
□ Next.js: PWA manifest + service worker (for push notifications)
GOAL: Customer can watch partner move to their door in real time.

SPRINT 6 — WEEK 11-12: Admin + International + Security Hardening
□ Full admin panel: orders, partners (all tabs), products, analytics dashboard
□ Khalti payment integration
□ International cards (Phase 2 path: Khalti international / bank acquirer — NOT direct Stripe; only if diaspora demand is proven)
□ Exchange rate feed (daily cron)
□ Rate limiting on all endpoints (Throttler + Redis)
□ Helmet.js + strict CORS
□ Sentry error monitoring (frontend + backend)
□ Privacy policy + Terms of Service pages
□ Contact form → admin email
□ Promo codes system
□ Load test with k6: 100 concurrent users, all critical paths
□ Fix every bug found in load test
□ SOFT LAUNCH — 3 partner cities, 10 partners, invite-only first week
□ HARD LAUNCH — week 2 open to public + influencer campaign

BEYOND SPRINT 6 (after revenue):
→ Native mobile app (React Native)
→ Corporate gifting portal
→ Subscription gifting (recurring monthly)
→ Airbyte + dbt + BigQuery pipeline
→ Demand forecasting model
→ Partner auto-assignment algorithm
```

---

## 30. FRONTEND ANIMATION PATTERNS (Complete)

```tsx
// PATTERN 1: Page wrapper — every page
export const PageTransition = ({ children }: { children: ReactNode }) => (
  <motion.div
    initial={{ opacity: 0, y: 10 }}
    animate={{ opacity: 1, y: 0 }}
    exit={{ opacity: 0, y: -6 }}
    transition={{ type: 'spring', damping: 28, stiffness: 220 }}
  >
    {children}
  </motion.div>
)

// PATTERN 2: Staggered grid (product catalog, city list)
const container = { hidden: {}, visible: { transition: { staggerChildren: 0.055 } } }
const item = {
  hidden:  { opacity: 0, y: 22, scale: 0.96 },
  visible: { opacity: 1, y: 0,  scale: 1, transition: { type: 'spring', damping: 22, stiffness: 260 } },
}
export const StaggerGrid = ({ children }: { children: ReactNode }) => (
  <motion.div variants={container} initial="hidden" animate="visible" className="grid grid-cols-2 gap-3">
    {children}  {/* Each child must be wrapped in <motion.div variants={item}> */}
  </motion.div>
)

// PATTERN 3: Bottom sheet with drag-to-dismiss
export const BottomSheet = ({ open, onClose, children }: BottomSheetProps) => (
  <AnimatePresence>
    {open && <>
      <motion.div
        className="fixed inset-0 z-40 bg-black/40 backdrop-blur-[2px]"
        initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }}
        onClick={onClose}
      />
      <motion.div
        className="fixed bottom-0 inset-x-0 z-50 rounded-t-[28px] bg-white overflow-auto"
        style={{ maxHeight: '92svh', paddingBottom: 'env(safe-area-inset-bottom)' }}
        initial={{ y: '100%' }}
        animate={{ y: 0 }}
        exit={{ y: '100%' }}
        transition={{ type: 'spring', damping: 32, stiffness: 340 }}
        drag="y" dragConstraints={{ top: 0 }} dragElastic={0.2}
        onDragEnd={(_, i) => { if (i.offset.y > 80 || i.velocity.y > 500) onClose() }}
      >
        <div className="w-9 h-1 rounded-full bg-gray-200 mx-auto mt-3 mb-1 flex-shrink-0" />
        {children}
      </motion.div>
    </>}
  </AnimatePresence>
)

// PATTERN 4: Delivery progress ring
export const DeliveryRing = ({ progress, status }: { progress: number; status: string }) => {
  const r = 44; const circ = 2 * Math.PI * r
  const color = status === 'DELIVERED' ? '#1D9E75' : status === 'DELIVERY_FAILED' ? '#E24B4A' : '#FF4D6D'
  return (
    <div className="relative w-28 h-28 flex items-center justify-center">
      <svg viewBox="0 0 100 100" className="absolute inset-0 w-full h-full -rotate-90">
        <circle cx="50" cy="50" r={r} fill="none" stroke="#f3f4f6" strokeWidth="7"/>
        <motion.circle
          cx="50" cy="50" r={r} fill="none" stroke={color} strokeWidth="7" strokeLinecap="round"
          strokeDasharray={circ}
          initial={{ strokeDashoffset: circ }}
          animate={{ strokeDashoffset: circ - (progress / 100) * circ }}
          transition={{ duration: 1.2, ease: [0.22, 1, 0.36, 1] }}
        />
      </svg>
      <span className="text-2xl">
        {status === 'DELIVERED' ? '✅' : status === 'OUT_FOR_DELIVERY' ? '🛵' : '🎁'}
      </span>
    </div>
  )
}

// PATTERN 5: Shimmer skeleton
export const SkeletonProductCard = () => (
  <div className="rounded-2xl overflow-hidden bg-gray-50 border border-gray-100">
    <div className="h-44 bg-gradient-to-r from-gray-100 via-gray-200 to-gray-100 bg-[length:200%] animate-[shimmer_1.4s_ease_infinite]" />
    <div className="p-3 space-y-2">
      <div className="h-3.5 bg-gray-200 rounded-full animate-pulse" />
      <div className="h-3 bg-gray-100 rounded-full w-2/3 animate-pulse" />
      <div className="h-5 bg-gray-100 rounded-full w-1/3 mt-1 animate-pulse" />
    </div>
  </div>
)
// globals.css: @keyframes shimmer { to { background-position: -200% center } }

// PATTERN 6: Haptic feedback wrapper (mobile only)
export const HapticButton = ({ onClick, children, ...props }: ButtonHTMLAttributes<HTMLButtonElement>) => {
  const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    if ('vibrate' in navigator) navigator.vibrate(8)  // 8ms — subtle
    onClick?.(e)
  }
  return (
    <motion.button whileTap={{ scale: 0.96 }} transition={{ type: 'spring', damping: 20 }} onClick={handleClick} {...props}>
      {children}
    </motion.button>
  )
}

// PATTERN 7: Number counter animation (for dashboard stats)
export const AnimatedNumber = ({ value }: { value: number }) => {
  const count = useMotionValue(0)
  const rounded = useTransform(count, Math.round)
  useEffect(() => { animate(count, value, { duration: 1.5, ease: 'easeOut' }) }, [value])
  return <motion.span>{rounded}</motion.span>
}
```

---

## 31. PWA MANIFEST + SERVICE WORKER

```json
// public/manifest.json
{
  "name": "Jhilko — Send Gifts Anywhere in Nepal",
  "short_name": "Jhilko",
  "description": "Send birthday cakes, flowers, and gifts to anyone in Nepal. Delivered locally, same day.",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#FAFAFA",
  "theme_color": "#FF4D6D",
  "orientation": "portrait",
  "categories": ["shopping", "lifestyle"],
  "icons": [
    { "src": "/icons/icon-72.png",  "sizes": "72x72",   "type": "image/png", "purpose": "any" },
    { "src": "/icons/icon-96.png",  "sizes": "96x96",   "type": "image/png", "purpose": "any" },
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png", "purpose": "any maskable" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png", "purpose": "any maskable" }
  ],
  "shortcuts": [
    { "name": "Send a Gift", "url": "/products", "icons": [{ "src": "/icons/shortcut-gift.png", "sizes": "96x96" }] },
    { "name": "Track Order", "url": "/orders",   "icons": [{ "src": "/icons/shortcut-track.png","sizes": "96x96" }] }
  ]
}
```

```typescript
// PWA via Serwist (@serwist/next + serwist) — next-pwa is unmaintained, do NOT use it.
// npm i @serwist/next && npm i -D serwist

// next.config.ts
import withSerwistInit from "@serwist/next";

const withSerwist = withSerwistInit({
  swSrc: "app/sw.ts",
  swDest: "public/sw.js",
  disable: process.env.NODE_ENV === "development",
});
export default withSerwist(nextConfig);

// app/sw.ts
import { defaultCache } from "@serwist/next/worker";
import { Serwist, StaleWhileRevalidate, NetworkFirst, NetworkOnly, ExpirationPlugin } from "serwist";

const serwist = new Serwist({
  precacheEntries: self.__SW_MANIFEST,
  skipWaiting: true,
  clientsClaim: true,
  navigationPreload: true,
  runtimeCaching: [
    {
      matcher: ({ url }) => url.hostname === "res.cloudinary.com",
      handler: new StaleWhileRevalidate({
        cacheName: "cloudinary-images",
        plugins: [new ExpirationPlugin({ maxEntries: 200, maxAgeSeconds: 7 * 24 * 60 * 60 })],
      }),
    },
    { matcher: ({ url }) => url.pathname.startsWith("/api/v1/products"), handler: new NetworkFirst({ cacheName: "api-products", networkTimeoutSeconds: 3 }) },
    { matcher: ({ url }) => url.pathname.startsWith("/api/v1/orders"), handler: new NetworkOnly() }, // Never cache order mutations
    ...defaultCache,
  ],
});
serwist.addEventListeners();
```

---

*CLAUDE.md — Jhilko edition.*
*Last updated: June 2026 | v3.0 — stack verified against live versions: Next.js 16.2.x, React 19.2.x, NestJS 11.1.x, Prisma 7.x, Node 22 LTS, Tailwind v4, Zod 4, Motion 12, Serwist, Baato Maps. Stripe-in-Nepal myth corrected.*

---
> Source: [Yrajaram112/jhilko](https://github.com/Yrajaram112/jhilko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
