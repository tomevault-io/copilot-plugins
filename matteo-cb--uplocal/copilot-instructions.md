## uplocal

> Uplocal is a browser-based AI image upscaler that runs entirely client-side using WebAssembly. Zero server cost for processing. Users upload an image, it gets upscaled via Real-ESRGAN compiled to WASM inside a Web Worker, and they download the result. The core value proposition is absolute privacy (no image ever leaves the browser) and speed. Monetization via Stripe: free tier allows one upscale per day at 2x, paid unlocks 4x and 8x, batch processing, custom output formats, and unlimited usage.

# CLAUDE.md — Uplocal (uplocal.app)

## Project Identity

Uplocal is a browser-based AI image upscaler that runs entirely client-side using WebAssembly. Zero server cost for processing. Users upload an image, it gets upscaled via Real-ESRGAN compiled to WASM inside a Web Worker, and they download the result. The core value proposition is absolute privacy (no image ever leaves the browser) and speed. Monetization via Stripe: free tier allows one upscale per day at 2x, paid unlocks 4x and 8x, batch processing, custom output formats, and unlimited usage.

The name "Uplocal" communicates both the action (upscale) and the method (local processing). This is the primary marketing angle: your images stay on your device, always.

The site must look like it was designed by a high-end Tokyo design studio. Think Morisawa type foundry meets Vercel engineering. Every page must feel intentional, typographically rich, and quietly confident. Zero AI-generated aesthetic.

## Tech Stack

- Next.js 15 (App Router, Server Components by default)
- TypeScript strict mode everywhere
- Tailwind CSS 4
- next-intl for 10 languages
- Prisma with MongoDB (user accounts, usage tracking, subscription status)
- NextAuth.js v5 (Google, GitHub, Email magic link providers)
- Stripe Checkout + Webhooks (subscriptions and one-time credit packs)
- Real-ESRGAN NCNN compiled to WebAssembly (runs in dedicated Web Worker)
- ONNX Runtime Web as secondary/fallback upscaling engine
- next-sitemap for XML sitemap generation
- Metadata API for all SEO meta tags (no next-seo package needed)

## Absolute Rules

- NEVER use emojis anywhere in the codebase, UI, or content
- NEVER use CSS gradients of any kind (linear, radial, conic)
- NEVER use dashes in user-facing UI text (use colons, commas, or periods instead)
- NEVER use badge/pill components with rounded-full
- NEVER use Lucide React or any icon library with generic line icons
- NEVER use Inter, Roboto, Arial, or system font stacks as design fonts
- NEVER use purple as a primary or accent color
- NEVER use drop shadows on cards as the primary visual treatment
- NEVER put placeholder "Lorem ipsum" in shipped pages
- ALL images must have explicit width, height, and alt text
- ALL pages must have unique, keyword-rich titles and descriptions per locale
- ALL interactive elements must have visible focus states
- ALL forms must have proper labels and error handling

## Design System

### Philosophy

"Japanese editorial minimalism." Clean, asymmetric layouts. Bold typographic hierarchy. Sharp geometry. Intentional whitespace used as a design element. Restrained color palette with one warm accent. The design should feel like opening a well-made architecture magazine, not a SaaS landing page.

### Typography

Display font: "Instrument Serif" from Google Fonts. Elegant, editorial, unexpected for a tech product. Used for all headings h1 through h3.

Body font: "Satoshi" from Fontshare CDN. Geometric sans-serif, modern, excellent readability at all sizes. Used for body text, UI labels, navigation, buttons.

Monospace: "JetBrains Mono" from Google Fonts. Used sparingly for technical specs, file sizes, resolution numbers.

Font loading: Use `next/font/google` for Instrument Serif and JetBrains Mono. Use `next/font/local` for Satoshi (self-hosted woff2). Display swap for headings, optional for body to prevent layout shift.

### Color Palette

```css
:root {
  --color-ink: #0A0A0A;
  --color-paper: #F6F4F0;
  --color-accent: #C2410C;
  --color-accent-hover: #9A3412;
  --color-accent-light: #FFF7ED;
  --color-muted: #78716C;
  --color-border: #D6D3D1;
  --color-surface: #FAFAF9;
  --color-error: #B91C1C;
  --color-success: #15803D;
}

[data-theme="dark"] {
  --color-ink: #FAFAF9;
  --color-paper: #0C0A09;
  --color-accent: #EA580C;
  --color-accent-hover: #F97316;
  --color-accent-light: #1C1917;
  --color-muted: #A8A29E;
  --color-border: #292524;
  --color-surface: #1C1917;
  --color-error: #EF4444;
  --color-success: #22C55E;
}
```

### Spacing Scale

Use Tailwind default spacing. Sections use py-24 or py-32 for generous vertical rhythm. Max content width is max-w-6xl. The upscaler tool itself uses max-w-4xl centered.

### Component Patterns

Buttons: Rectangular with sharp corners (rounded-none or rounded-sm max). Primary button uses bg-accent with text-white. Secondary uses border with border-ink. Hover states use background color shift, never opacity change. All buttons have min-height of 48px for touch targets.

Cards: Use border with border-color-border. Background is color-surface. No shadow. On hover, border shifts to color-ink. Transition duration 200ms.

Inputs: Full width, border-bottom only (no full border box), with a thick 2px bottom border. On focus, border color shifts to accent. Label sits above input, font-size 0.75rem uppercase tracking-widest in color-muted.

Section dividers: Use a thin 1px line in color-border. Never decorative. Sometimes use a large typographic element (oversized number or word) as a section marker instead.

### Layout Principles

Use CSS Grid for page-level layout. Asymmetric two-column layouts where the left column is narrower (content label/heading) and the right column is wider (main content). This creates the editorial magazine feel.

Homepage hero: No centered text block. Use a split layout with the heading on the left (large, Instrument Serif, 4xl to 6xl) and a live demo preview on the right. The demo shows a before/after slider with a sample image.

Navigation: Fixed top bar, transparent on scroll at top, solid color-paper background after scrolling 50px. Logo on left (wordmark "Uplocal" in Instrument Serif, not an icon), language switcher and auth buttons on right. Mobile uses a full-screen overlay menu, not a hamburger dropdown.

Footer: Three-column grid. Column 1 is the wordmark with a one-sentence description. Column 2 is navigation links grouped by category. Column 3 is legal links and language switcher duplicate. Below is a full-width line then copyright.

### Animation

Use CSS transitions only (no Framer Motion, no GSAP). Page elements fade in on load with a staggered delay using CSS animation-delay. The before/after image slider has a smooth drag interaction. Processing progress uses a minimal horizontal bar animation, not a spinner. Hover states transition in 200ms ease-out.

### Dark Mode

Respect system preference via `prefers-color-scheme`. Also provide a manual toggle stored in localStorage and a cookie (for SSR). The toggle is a simple text button reading "Light" or "Dark" in the navigation, not a sun/moon icon.

## Project Structure

```
uplocal/
  CLAUDE.md
  next.config.ts
  tailwind.config.ts
  prisma/
    schema.prisma
  public/
    models/
      realesrgan-x2.bin
      realesrgan-x4.bin
    wasm/
      realesrgan.wasm
      realesrgan.js
    fonts/
      Satoshi-Variable.woff2
      Satoshi-VariableItalic.woff2
    og/
      og-default.png
      og-en.png
      og-fr.png
      og-de.png
      og-es.png
      og-pt.png
      og-ja.png
      og-ko.png
      og-zh.png
      og-ar.png
      og-hi.png
  src/
    app/
      [locale]/
        layout.tsx
        page.tsx
        upscale/
          page.tsx
        pricing/
          page.tsx
        about/
          page.tsx
        blog/
          page.tsx
          [slug]/
            page.tsx
        auth/
          signin/
            page.tsx
          signup/
            page.tsx
        dashboard/
          page.tsx
          settings/
            page.tsx
          history/
            page.tsx
        legal/
          privacy/
            page.tsx
          terms/
            page.tsx
        faq/
          page.tsx
      api/
        auth/
          [...nextauth]/
            route.ts
        stripe/
          checkout/
            route.ts
          webhook/
            route.ts
          portal/
            route.ts
        usage/
          route.ts
          check/
            route.ts
    components/
      layout/
        Header.tsx
        Footer.tsx
        Navigation.tsx
        MobileMenu.tsx
        LanguageSwitcher.tsx
        ThemeToggle.tsx
      upscaler/
        UpscalerCanvas.tsx
        DropZone.tsx
        BeforeAfterSlider.tsx
        ProcessingBar.tsx
        OutputPreview.tsx
        DownloadButton.tsx
        ScaleSelector.tsx
        FormatSelector.tsx
        BatchUploader.tsx
      pricing/
        PricingTable.tsx
        PlanCard.tsx
        FeatureComparison.tsx
      dashboard/
        UsageChart.tsx
        HistoryGrid.tsx
        AccountSettings.tsx
      shared/
        Button.tsx
        Input.tsx
        SectionLabel.tsx
        PageTransition.tsx
        SEOHead.tsx
        StructuredData.tsx
        CookieConsent.tsx
    lib/
      prisma.ts
      auth.ts
      stripe.ts
      usage.ts
      wasm/
        upscaler-worker.ts
        upscaler-engine.ts
        wasm-loader.ts
      seo/
        metadata.ts
        structured-data.ts
        hreflang.ts
      i18n/
        request.ts
        routing.ts
    hooks/
      useUpscaler.ts
      useUsage.ts
      useTheme.ts
    types/
      index.ts
      upscaler.ts
      stripe.ts
    messages/
      en.json
      fr.json
      de.json
      es.json
      pt.json
      ja.json
      ko.json
      zh.json
      ar.json
      hi.json
  middleware.ts
```

## Internationalization (next-intl)

### Supported Locales

1. en (English) — default
2. fr (French)
3. de (German)
4. es (Spanish)
5. pt (Portuguese)
6. ja (Japanese)
7. ko (Korean)
8. zh (Simplified Chinese)
9. ar (Arabic)
10. hi (Hindi)

### i18n Architecture

Use next-intl with the App Router integration. The middleware handles locale detection from Accept-Language header and redirects. All routes are prefixed with locale: `/en/upscale`, `/fr/upscale`, etc. The default locale (en) also gets the prefix for URL consistency.

```typescript
// src/lib/i18n/routing.ts
import { defineRouting } from "next-intl/routing";

export const routing = defineRouting({
  locales: ["en", "fr", "de", "es", "pt", "ja", "ko", "zh", "ar", "hi"],
  defaultLocale: "en",
  localePrefix: "always",
  pathnames: {
    "/": "/",
    "/upscale": {
      en: "/upscale",
      fr: "/ameliorer",
      de: "/hochskalieren",
      es: "/mejorar",
      pt: "/melhorar",
      ja: "/upscale",
      ko: "/upscale",
      zh: "/upscale",
      ar: "/upscale",
      hi: "/upscale"
    },
    "/pricing": {
      en: "/pricing",
      fr: "/tarifs",
      de: "/preise",
      es: "/precios",
      pt: "/precos",
      ja: "/pricing",
      ko: "/pricing",
      zh: "/pricing",
      ar: "/pricing",
      hi: "/pricing"
    },
    "/about": {
      en: "/about",
      fr: "/a-propos",
      de: "/ueber-uns",
      es: "/sobre",
      pt: "/sobre",
      ja: "/about",
      ko: "/about",
      zh: "/about",
      ar: "/about",
      hi: "/about"
    },
    "/blog": "/blog",
    "/faq": "/faq",
    "/dashboard": "/dashboard",
    "/legal/privacy": "/legal/privacy",
    "/legal/terms": "/legal/terms"
  }
});
```

### RTL Support

Arabic (ar) requires RTL layout. Use the `dir` attribute on the `<html>` tag, conditionally set in the root layout based on locale. Tailwind CSS logical properties (ps, pe, ms, me) must be used instead of pl, pr, ml, mr throughout the entire codebase so that RTL works automatically.

### Message Files Structure

Each locale JSON file follows this structure:

```json
{
  "metadata": {
    "title": "Uplocal — AI Image Upscaler, 100% Private, Runs in Your Browser",
    "description": "Upscale your images up to 8x using AI directly in your browser. No upload, no server, total privacy. Your files never leave your device.",
    "keywords": "image upscaler, AI upscale, enhance image, increase resolution, private upscaler, browser upscaler"
  },
  "nav": {
    "upscale": "Upscale",
    "pricing": "Pricing",
    "about": "About",
    "blog": "Blog",
    "signin": "Sign in",
    "signup": "Get started",
    "dashboard": "Dashboard",
    "themeLight": "Light",
    "themeDark": "Dark"
  },
  "hero": {
    "title": "Upscale any image with AI. Right in your browser.",
    "subtitle": "Your images never leave your device. Uplocal runs entirely in your browser using WebAssembly. No server, no upload, no compromise on privacy.",
    "cta": "Upscale an image now",
    "secondaryCta": "See pricing"
  },
  "features": {
    "sectionLabel": "Why Uplocal",
    "privacy": {
      "title": "100% Private",
      "description": "Your images are processed locally in your browser. Nothing is uploaded. Nothing is stored. Nobody sees your files."
    },
    "quality": {
      "title": "AI Powered Quality",
      "description": "Real-ESRGAN neural network running via WebAssembly delivers professional-grade upscaling with artifact removal and detail enhancement."
    },
    "speed": {
      "title": "Instant Results",
      "description": "No waiting in server queues. Processing starts immediately on your hardware. Most images upscale in under 10 seconds."
    },
    "formats": {
      "title": "All Formats Supported",
      "description": "Upload PNG, JPEG, WebP, or BMP. Download in your preferred format with adjustable quality settings."
    }
  },
  "upscaler": {
    "dropzone": "Drop your image here or click to browse",
    "dropzoneFormats": "PNG, JPEG, WebP, BMP up to 10MB",
    "scaleLabel": "Scale",
    "formatLabel": "Output format",
    "qualityLabel": "Quality",
    "processing": "Processing your image",
    "download": "Download upscaled image",
    "compare": "Compare before and after",
    "newImage": "Upscale another image",
    "freeLimit": "You have used your free upscale for today",
    "upgradePrompt": "Upgrade to unlock unlimited upscaling at 4x and 8x",
    "batchTitle": "Batch Processing",
    "batchDescription": "Drag multiple images to process them all at once"
  },
  "pricing": {
    "sectionLabel": "Pricing",
    "title": "Simple, transparent pricing",
    "free": {
      "name": "Free",
      "price": "0",
      "period": "forever",
      "features": [
        "1 upscale per day",
        "2x maximum scale",
        "PNG and JPEG output",
        "Standard processing"
      ],
      "cta": "Get started free"
    },
    "pro": {
      "name": "Pro",
      "priceMonthly": "9",
      "priceYearly": "79",
      "period": "month",
      "periodYearly": "year",
      "features": [
        "Unlimited upscales",
        "Up to 4x scale",
        "All output formats",
        "Batch processing up to 10 images",
        "Priority model loading"
      ],
      "cta": "Start Pro"
    },
    "studio": {
      "name": "Studio",
      "priceMonthly": "24",
      "priceYearly": "199",
      "period": "month",
      "periodYearly": "year",
      "features": [
        "Everything in Pro",
        "Up to 8x scale",
        "Batch processing up to 50 images",
        "Custom output dimensions",
        "API access (coming soon)",
        "Commercial license"
      ],
      "cta": "Start Studio"
    },
    "toggle": {
      "monthly": "Monthly",
      "yearly": "Yearly",
      "save": "Save 27%"
    }
  },
  "faq": {
    "sectionLabel": "Questions",
    "items": [
      {
        "question": "How does Uplocal upscale images without a server?",
        "answer": "Uplocal uses Real-ESRGAN, an AI upscaling model, compiled to WebAssembly. This means the AI model runs directly in your browser, on your own device. Your images never leave your computer."
      },
      {
        "question": "Is there a file size limit?",
        "answer": "Free users can upscale images up to 5MB. Pro and Studio users can process images up to 10MB. For best results with very large images, we recommend starting with images under 4000x4000 pixels."
      },
      {
        "question": "What image formats are supported?",
        "answer": "Uplocal accepts PNG, JPEG, WebP, and BMP as input. Output can be saved as PNG, JPEG, or WebP with adjustable quality settings."
      },
      {
        "question": "How long does upscaling take?",
        "answer": "Processing time depends on the image size and your device. A typical 1080p image upscaled to 2x takes 5 to 15 seconds on a modern laptop. 4x and 8x scales take proportionally longer."
      },
      {
        "question": "Can I cancel my subscription?",
        "answer": "Yes, you can cancel anytime from your dashboard. You will retain access to your plan features until the end of your current billing period."
      },
      {
        "question": "Does it work on mobile?",
        "answer": "Uplocal works on mobile browsers that support WebAssembly, including Chrome and Safari on iOS and Android. Performance will vary based on your device capabilities."
      }
    ]
  },
  "footer": {
    "description": "AI image upscaling, 100% in your browser. Your files never leave your device.",
    "product": "Product",
    "company": "Company",
    "legal": "Legal",
    "copyright": "Uplocal. All rights reserved."
  },
  "auth": {
    "signinTitle": "Welcome back",
    "signupTitle": "Create your account",
    "emailLabel": "Email address",
    "passwordLabel": "Password",
    "magicLink": "Send magic link",
    "orContinueWith": "Or continue with",
    "google": "Google",
    "github": "GitHub",
    "noAccount": "No account yet?",
    "hasAccount": "Already have an account?",
    "signupLink": "Create one",
    "signinLink": "Sign in"
  },
  "dashboard": {
    "title": "Dashboard",
    "usage": "Usage this month",
    "history": "Recent upscales",
    "plan": "Current plan",
    "upgrade": "Upgrade plan",
    "manageBilling": "Manage billing",
    "settings": "Settings",
    "noHistory": "No upscales yet. Try your first one."
  }
}
```

All 10 locale files must be fully translated with native-quality translations. No machine-translation artifacts. Each locale must have unique SEO metadata with locally relevant keywords.

### SEO Keywords Per Locale

- en: "image upscaler", "AI image enhancer", "upscale image online", "increase image resolution", "enhance photo quality", "private image upscaler", "browser image upscaler"
- fr: "agrandir image", "ameliorer qualite image", "upscaler image IA", "augmenter resolution image", "ameliorer photo en ligne", "upscaler image sans upload"
- de: "Bild vergroessern", "Bildqualitaet verbessern", "KI Bild hochskalieren", "Bildaufloesung erhoehen", "Bild hochskalieren online"
- es: "mejorar calidad imagen", "aumentar resolucion imagen", "escalador de imagenes IA", "mejorar foto online", "mejorar imagen sin subir"
- pt: "melhorar qualidade imagem", "aumentar resolucao imagem", "upscaler de imagem IA", "melhorar foto online"
- ja: "画像 高画質化", "AI 画像 アップスケール", "画像 解像度 上げる", "写真 高画質", "画像拡大 ブラウザ"
- ko: "이미지 업스케일", "AI 이미지 화질 개선", "사진 해상도 높이기", "브라우저 이미지 업스케일러"
- zh: "AI图片放大", "图片增强", "提高图片分辨率", "图片无损放大", "浏览器图片放大"
- ar: "تحسين جودة الصورة", "تكبير الصورة بالذكاء الاصطناعي", "رفع دقة الصورة"
- hi: "इमेज अपस्केलर", "फोटो क्वालिटी बढ़ाएं", "AI इमेज एन्हांसर", "ब्राउज़र इमेज अपस्केलर"

## Database Schema (Prisma + MongoDB)

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

model User {
  id            String    @id @default(auto()) @map("_id") @db.ObjectId
  name          String?
  email         String    @unique
  emailVerified DateTime?
  image         String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  accounts      Account[]
  sessions      Session[]
  subscription  Subscription?
  usageRecords  UsageRecord[]
  preferences   UserPreferences?
}

model Account {
  id                String  @id @default(auto()) @map("_id") @db.ObjectId
  userId            String  @db.ObjectId
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(auto()) @map("_id") @db.ObjectId
  sessionToken String   @unique
  userId       String   @db.ObjectId
  expires      DateTime

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  id         String   @id @default(auto()) @map("_id") @db.ObjectId
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}

model Subscription {
  id                   String   @id @default(auto()) @map("_id") @db.ObjectId
  userId               String   @unique @db.ObjectId
  stripeCustomerId     String   @unique
  stripeSubscriptionId String?  @unique
  stripePriceId        String?
  plan                 Plan     @default(FREE)
  status               SubscriptionStatus @default(ACTIVE)
  currentPeriodStart   DateTime?
  currentPeriodEnd     DateTime?
  cancelAtPeriodEnd    Boolean  @default(false)
  createdAt            DateTime @default(now())
  updatedAt            DateTime @updatedAt

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model UsageRecord {
  id             String   @id @default(auto()) @map("_id") @db.ObjectId
  userId         String   @db.ObjectId
  action         String   // "upscale"
  scale          Int      // 2, 4, or 8
  inputWidth     Int
  inputHeight    Int
  outputWidth    Int
  outputHeight   Int
  inputFormat    String
  outputFormat   String
  processingTime Int      // milliseconds
  createdAt      DateTime @default(now())

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId, createdAt])
}

model UserPreferences {
  id              String  @id @default(auto()) @map("_id") @db.ObjectId
  userId          String  @unique @db.ObjectId
  theme           String  @default("system") // "light", "dark", "system"
  defaultScale    Int     @default(2)
  defaultFormat   String  @default("png")
  defaultQuality  Int     @default(90)
  locale          String  @default("en")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model BlogPost {
  id          String   @id @default(auto()) @map("_id") @db.ObjectId
  slug        String   @unique
  locale      String
  title       String
  excerpt     String
  content     String
  author      String
  tags        String[]
  publishedAt DateTime
  updatedAt   DateTime @updatedAt
  isPublished Boolean  @default(false)

  @@unique([slug, locale])
  @@index([locale, isPublished, publishedAt])
}

enum Plan {
  FREE
  PRO
  STUDIO
}

enum SubscriptionStatus {
  ACTIVE
  PAST_DUE
  CANCELED
  UNPAID
}
```

## Authentication (NextAuth.js v5)

### Providers

1. Google OAuth
2. GitHub OAuth
3. Email Magic Link (using Resend as email provider, free tier is sufficient)

### Auth Configuration

```typescript
// src/lib/auth.ts
import NextAuth from "next-auth";
import Google from "next-auth/providers/google";
import GitHub from "next-auth/providers/github";
import Resend from "next-auth/providers/resend";
import { PrismaAdapter } from "@auth/prisma-adapter";
import { prisma } from "./prisma";

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
    GitHub({
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    }),
    Resend({
      from: "Uplocal <auth@uplocal.app>",
    }),
  ],
  pages: {
    signIn: "/auth/signin",
    newUser: "/dashboard",
  },
  callbacks: {
    async session({ session, user }) {
      session.user.id = user.id;
      return session;
    },
  },
});
```

### Auth Middleware

The middleware in `middleware.ts` handles both locale routing and auth protection. Protected routes: `/dashboard`, `/dashboard/settings`, `/dashboard/history`. All other routes are public. The middleware checks the session cookie and redirects to `/auth/signin` with a callbackUrl if unauthenticated.

### Session Strategy

Use database sessions (default with PrismaAdapter). This is more secure than JWT for a payment-enabled application. Session data includes user ID, email, name, image, and current plan.

## Stripe Integration

### Products and Prices

Create three products in Stripe Dashboard:

1. **Free** — No Stripe product needed. This is the default plan.
2. **Pro** — Monthly ($9/mo) and Yearly ($79/yr) price variants.
3. **Studio** — Monthly ($24/mo) and Yearly ($199/yr) price variants.

Store the Stripe Price IDs in environment variables:

```env
STRIPE_PRO_MONTHLY_PRICE_ID=price_xxx
STRIPE_PRO_YEARLY_PRICE_ID=price_xxx
STRIPE_STUDIO_MONTHLY_PRICE_ID=price_xxx
STRIPE_STUDIO_YEARLY_PRICE_ID=price_xxx
```

### Checkout Flow

1. User clicks a plan CTA on the pricing page.
2. Frontend calls `POST /api/stripe/checkout` with the selected priceId.
3. Server creates a Stripe Checkout Session with the user's email prefilled.
4. User is redirected to Stripe Checkout.
5. On success, Stripe redirects to `/dashboard?session_id={CHECKOUT_SESSION_ID}`.
6. The Stripe webhook (`checkout.session.completed`) creates or updates the Subscription document in MongoDB.

### Webhook Events to Handle

```typescript
// POST /api/stripe/webhook
// Events:
// checkout.session.completed → Create/update Subscription
// customer.subscription.updated → Update plan, status, period dates
// customer.subscription.deleted → Set status to CANCELED
// invoice.payment_failed → Set status to PAST_DUE
// invoice.payment_succeeded → Set status to ACTIVE, update period
```

### Customer Portal

Provide a "Manage billing" button in the dashboard that calls `POST /api/stripe/portal`. This creates a Stripe Customer Portal session and redirects. Users can update payment methods, view invoices, and cancel subscriptions.

### Usage Enforcement

Usage is checked server-side via `GET /api/usage/check`. The endpoint returns:

```typescript
{
  canUpscale: boolean;
  remainingToday: number; // -1 for unlimited
  maxScale: number; // 2, 4, or 8 based on plan
  maxFileSize: number; // bytes
  batchLimit: number; // max images per batch
  plan: "FREE" | "PRO" | "STUDIO";
}
```

The upscaler UI calls this endpoint before processing. After successful upscale, the frontend calls `POST /api/usage` to log the record.

Free tier logic: Count UsageRecords where userId matches AND createdAt is today (UTC). If count >= 1, deny.

## WebAssembly Upscaling Engine

### Architecture

The upscaling runs entirely in the browser via a Web Worker to avoid blocking the main thread.

```
Main Thread                    Web Worker
    |                              |
    |-- postMessage(imageData) --> |
    |                              |-- Load WASM module
    |                              |-- Initialize model
    |                              |-- Process image tiles
    |                              |-- Reassemble output
    |<-- postMessage(result) ------|
```

### Web Worker Implementation

```typescript
// src/lib/wasm/upscaler-worker.ts

// The worker loads the Real-ESRGAN WASM module on first use.
// Model weights are loaded from /public/models/.
// Images larger than 512x512 are processed in overlapping tiles
// to manage memory usage (WASM has limited heap).
// Tiles are reassembled with blended overlap regions.

// Message types:
// { type: "init", scale: 2 | 4 | 8 } — Load model for given scale
// { type: "process", imageData: ImageData, options: ProcessOptions }
// { type: "progress", percent: number } — Sent during processing
// { type: "result", imageData: ImageData, stats: ProcessStats }
// { type: "error", message: string }
```

### Process Options

```typescript
interface ProcessOptions {
  scale: 2 | 4 | 8;
  outputFormat: "png" | "jpeg" | "webp";
  quality: number; // 1-100, only for jpeg and webp
  tileSize: number; // default 512
  tileOverlap: number; // default 32
}

interface ProcessStats {
  inputWidth: number;
  inputHeight: number;
  outputWidth: number;
  outputHeight: number;
  processingTimeMs: number;
  tilesProcessed: number;
}
```

### WASM Model Loading

Models are loaded lazily. On first upscale, the worker fetches the WASM binary and model weights. These are cached in the browser's Cache API so subsequent loads are instant. Show a "Loading AI model" progress bar on first use.

```typescript
// src/lib/wasm/wasm-loader.ts
export async function loadWasmModule(): Promise<WasmModule> {
  const cache = await caches.open("uplocal-models-v1");
  
  // Check cache first
  let wasmResponse = await cache.match("/wasm/realesrgan.wasm");
  if (!wasmResponse) {
    wasmResponse = await fetch("/wasm/realesrgan.wasm");
    await cache.put("/wasm/realesrgan.wasm", wasmResponse.clone());
  }
  
  const wasmBytes = await wasmResponse.arrayBuffer();
  const module = await WebAssembly.compile(wasmBytes);
  return module;
}
```

### Fallback Strategy

If WebAssembly is not supported (extremely rare in 2026), show a clear message: "Your browser does not support WebAssembly. Please use a modern browser like Chrome, Firefox, Safari, or Edge." Do not attempt server-side fallback; the entire point is zero server cost.

If ONNX Runtime Web is available and the WASM module fails to load (corrupted download, etc.), attempt to use ONNX Runtime Web with a smaller model as fallback.

### Memory Management

WebAssembly has a fixed memory limit. For large images:
- Tile the input image into 512x512 chunks with 32px overlap
- Process tiles sequentially (not in parallel) to avoid memory exhaustion
- Free each tile's memory after it's been placed into the output canvas
- Use OffscreenCanvas in the worker for zero-copy image handling

### Hook: useUpscaler

```typescript
// src/hooks/useUpscaler.ts
export function useUpscaler() {
  // States: idle, loading-model, processing, complete, error
  // Returns:
  // - state: UpscalerState
  // - progress: number (0-100)
  // - result: { imageUrl: string, stats: ProcessStats } | null
  // - error: string | null
  // - upscale: (file: File, options: ProcessOptions) => Promise<void>
  // - reset: () => void
  // - downloadResult: (filename?: string) => void
}
```

## SEO Strategy

### Technical SEO

Every page must generate complete metadata via the Next.js Metadata API:

```typescript
// src/lib/seo/metadata.ts
export function generatePageMetadata(
  locale: string,
  page: string,
  t: (key: string) => string
): Metadata {
  return {
    title: t(`${page}.metadata.title`),
    description: t(`${page}.metadata.description`),
    keywords: t(`${page}.metadata.keywords`),
    alternates: {
      canonical: `https://uplocal.app/${locale}/${page}`,
      languages: Object.fromEntries(
        LOCALES.map((l) => [l, `https://uplocal.app/${l}/${page}`])
      ),
    },
    openGraph: {
      title: t(`${page}.metadata.title`),
      description: t(`${page}.metadata.description`),
      url: `https://uplocal.app/${locale}/${page}`,
      siteName: "Uplocal",
      images: [
        {
          url: `https://uplocal.app/og/og-${locale}.png`,
          width: 1200,
          height: 630,
          alt: t(`${page}.metadata.title`),
        },
      ],
      locale: locale,
      type: "website",
    },
    twitter: {
      card: "summary_large_image",
      title: t(`${page}.metadata.title`),
      description: t(`${page}.metadata.description`),
      images: [`https://uplocal.app/og/og-${locale}.png`],
    },
    robots: {
      index: true,
      follow: true,
      googleBot: {
        index: true,
        follow: true,
        "max-video-preview": -1,
        "max-image-preview": "large",
        "max-snippet": -1,
      },
    },
  };
}
```

### Hreflang Tags

Every page must include hreflang tags for all 10 locales plus x-default pointing to /en/. These are generated in the root layout's metadata.

### Structured Data (JSON-LD)

Every page must include relevant structured data:

Homepage:
```json
{
  "@context": "https://schema.org",
  "@type": "WebApplication",
  "name": "Uplocal",
  "url": "https://uplocal.app",
  "description": "AI image upscaler that runs 100% in your browser. No upload, total privacy.",
  "applicationCategory": "MultimediaApplication",
  "operatingSystem": "Any",
  "offers": [
    {
      "@type": "Offer",
      "name": "Free",
      "price": "0",
      "priceCurrency": "USD"
    },
    {
      "@type": "Offer",
      "name": "Pro",
      "price": "9",
      "priceCurrency": "USD",
      "billingIncrement": "P1M"
    },
    {
      "@type": "Offer",
      "name": "Studio",
      "price": "24",
      "priceCurrency": "USD",
      "billingIncrement": "P1M"
    }
  ],
  "featureList": [
    "AI image upscaling up to 8x",
    "100% browser-based processing",
    "No file upload required",
    "Supports PNG, JPEG, WebP, BMP",
    "Batch processing",
    "Available in 10 languages"
  ]
}
```

Pricing page: Use `Product` with `Offer` for each plan.

FAQ page: Use `FAQPage` schema with all questions and answers.

Blog posts: Use `Article` schema with author, datePublished, dateModified.

### Sitemap

Use next-sitemap to generate:
- `/sitemap.xml` — index pointing to locale sitemaps
- `/sitemap-en.xml`, `/sitemap-fr.xml`, etc. — one per locale
- Each sitemap includes all pages for that locale with lastmod dates

### Robots.txt

```
User-agent: *
Allow: /
Sitemap: https://uplocal.app/sitemap.xml

# Block auth and API routes
Disallow: /api/
Disallow: /*/auth/
Disallow: /*/dashboard/
```

### Performance Targets

- Lighthouse Performance: 95+
- Lighthouse Accessibility: 100
- Lighthouse Best Practices: 100
- Lighthouse SEO: 100
- First Contentful Paint: under 1.2s
- Largest Contentful Paint: under 2.5s
- Cumulative Layout Shift: under 0.1

Achieve this via:
- Server Components by default (no client JS for static content)
- Dynamic imports for the upscaler (loaded only on /upscale page)
- next/image for all images with priority on above-fold images
- Font preloading for Instrument Serif only
- Minimal client-side JavaScript on homepage
- No third-party scripts except Stripe.js (loaded only on pricing/checkout)

### Blog Strategy for SEO

The blog exists primarily for SEO. Each post targets a specific long-tail keyword. All posts are available in all 10 locales. Suggested initial posts:

1. "How to upscale an image without losing quality" (targets primary keyword cluster)
2. "Browser vs server image processing: why local is better" (targets privacy-conscious users)
3. "What is Real-ESRGAN and how does it work" (targets technical audience)
4. "Best image formats for upscaling: PNG vs JPEG vs WebP" (targets format-related queries)
5. "How to upscale old photos for printing" (targets photo restoration audience)
6. "AI image upscaling explained: a complete guide" (targets educational queries)
7. "Upscaling images for social media: the complete resolution guide" (targets social media managers)
8. "How WebAssembly makes AI possible in your browser" (targets developer audience)

Each blog post must be 1500+ words, include internal links to the upscaler tool, and have a CTA to try Uplocal.

## Environment Variables

```env
# Database
DATABASE_URL=mongodb+srv://...

# Auth
AUTH_SECRET=generated-secret
GOOGLE_CLIENT_ID=xxx
GOOGLE_CLIENT_SECRET=xxx
GITHUB_CLIENT_ID=xxx
GITHUB_CLIENT_SECRET=xxx
RESEND_API_KEY=re_xxx

# Stripe
STRIPE_SECRET_KEY=sk_xxx
STRIPE_PUBLISHABLE_KEY=pk_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
STRIPE_PRO_MONTHLY_PRICE_ID=price_xxx
STRIPE_PRO_YEARLY_PRICE_ID=price_xxx
STRIPE_STUDIO_MONTHLY_PRICE_ID=price_xxx
STRIPE_STUDIO_YEARLY_PRICE_ID=price_xxx

# App
NEXT_PUBLIC_APP_URL=https://uplocal.app
NEXT_PUBLIC_APP_NAME=Uplocal
```

## Deployment

Deploy on Vercel. Configure:
- Custom domain: uplocal.app
- Environment variables from above
- Build command: `npx prisma generate && next build`
- Node.js version: 20.x
- Edge middleware for locale routing

### Vercel Configuration Notes

- The WASM files in `/public/wasm/` must be served with correct MIME type `application/wasm`
- Model files in `/public/models/` are large (10-50MB). Configure `Cache-Control: public, max-age=31536000, immutable` for these paths in `next.config.ts` headers
- Enable Vercel Analytics for real user monitoring
- Enable Vercel Speed Insights for Core Web Vitals tracking

## Page-by-Page Specifications

### Homepage (`/[locale]`)

The homepage is the primary landing page and SEO entry point.

Layout:
1. Hero section: Split layout. Left: "Upscale any image with AI. Right in your browser." in Instrument Serif, 5xl on desktop, 3xl on mobile. Below: subtitle in Satoshi, color-muted. Two buttons: primary CTA "Upscale an image now" and secondary "See pricing". Right side: An animated before/after comparison of a sample image being upscaled. The slider auto-animates slowly on load to demonstrate the effect.

2. Trust bar: A thin section below the hero with three proof points in a row: "100% browser based", "No image upload", "10M+ images upscaled". Use oversized monospace numbers for the stats.

3. How it works: Three-step section with numbered steps (01, 02, 03 in large Instrument Serif). Step 1: Drop your image. Step 2: Choose your scale. Step 3: Download instantly. Each step has a small illustration or screenshot.

4. Features grid: Asymmetric two-column layout. Left column: section label "Why Uplocal" in small caps. Right column: four feature blocks stacked vertically with title in Instrument Serif and description in Satoshi.

5. Live demo: An embedded mini version of the upscaler. A pre-loaded sample image with a "Try it now" button that activates the upscaler inline. This section demonstrates the tool without requiring navigation.

6. Pricing preview: Show the three plans in a horizontal row. Highlight Pro as "Most popular". Link to full pricing page.

7. FAQ section: Collapsible accordion with the top 4 questions. "See all questions" link to full FAQ page.

8. Final CTA: Full-width section with "Start upscaling for free" headline and primary CTA button.

### Upscale Tool Page (`/[locale]/upscale`)

This is the core product page. Must be lightning fast to load.

Layout:
1. Page title: "Upscale your image" in Instrument Serif, centered.
2. Dropzone: Large bordered area (dashed border, 2px, color-border) centered. On hover, border shifts to accent. Accepts drag-and-drop and click-to-browse. Shows accepted formats and size limit.
3. After image is loaded:
   - Image preview on the left (original)
   - Controls panel on the right:
     - Scale selector: Three buttons (2x, 4x, 8x) in a horizontal row. Disabled scales are grayed out with a lock icon (custom SVG, not Lucide) and "Pro" or "Studio" label.
     - Output format: Dropdown (PNG, JPEG, WebP)
     - Quality slider: Only visible for JPEG and WebP. Range 1-100, default 90.
     - "Upscale" primary button
4. During processing:
   - The upscale button transforms into a progress bar
   - Percentage and estimated time remaining shown
   - "Processing your image" text
5. After completion:
   - Before/after slider showing original vs upscaled
   - Stats: original resolution, new resolution, processing time, file size
   - "Download" primary button
   - "Upscale another image" secondary button
   - "Compare" toggle to switch between side-by-side and overlay views

### Pricing Page (`/[locale]/pricing`)

Layout:
1. Page title: "Simple, transparent pricing" in Instrument Serif.
2. Monthly/Yearly toggle with "Save 27%" label next to yearly.
3. Three plan cards in a horizontal row (stack vertically on mobile):
   - Free: Subtle, no background emphasis
   - Pro: Emphasized with accent border on top (4px solid accent)
   - Studio: Subtle, same as Free
4. Feature comparison table below the cards. Full grid showing every feature with checkmarks (custom SVG, not Lucide) for included features.
5. FAQ section specific to pricing: "Can I switch plans?", "Do you offer refunds?", etc.

### Dashboard (`/[locale]/dashboard`)

Protected route. Layout:
1. Welcome message with user name
2. Current plan card with upgrade CTA if on Free
3. Usage stats: upscales this month, average processing time
4. Simple bar chart showing daily usage (last 30 days)
5. Recent history: Grid of thumbnail previews of recent upscales with date, scale used, and processing time
6. Quick actions: "Upscale new image", "Manage billing", "Settings"

### Blog (`/[locale]/blog`)

Layout:
1. Page title: "Blog" in Instrument Serif
2. Grid of post cards: Two columns on desktop. Each card shows title, excerpt, date, reading time, and tags. No images on cards (keep it typographic).
3. Pagination at bottom

### Blog Post (`/[locale]/blog/[slug]`)

Layout:
1. Full-width article layout, max-w-3xl centered
2. Title in Instrument Serif, 4xl
3. Metadata line: author, date, reading time
4. Article body in Satoshi, 1.125rem line-height 1.75
5. In-article CTA: After the third paragraph, insert a subtle inline CTA to try the upscaler
6. Related posts at bottom

### Auth Pages

Minimal, centered card layout. No sidebar, no decorative elements. Just the form with social login buttons above and magic link form below. The Uplocal wordmark sits above the form.

### Legal Pages (Privacy, Terms)

Standard prose layout, max-w-3xl. Table of contents on the left as sticky sidebar on desktop. All sections use semantic heading hierarchy for accessibility.

## Testing Strategy

### Unit Tests

Use Vitest for all utility functions, hooks, and server-side logic. Coverage targets:
- Usage enforcement logic: 100%
- Stripe webhook handlers: 100%
- i18n routing: 100%
- WASM loader: 90%+

### E2E Tests

Use Playwright for critical user flows:
1. Homepage loads in all 10 locales
2. Free user can upload, upscale, and download
3. Free user is blocked after daily limit
4. Stripe checkout flow completes
5. Dashboard shows usage data
6. Language switcher changes locale and preserves path
7. Dark mode toggle works and persists

### Performance Tests

Run Lighthouse CI in the CI pipeline. Fail the build if:
- Performance score drops below 90
- SEO score drops below 95
- Accessibility score drops below 95

## Analytics

Use Vercel Analytics (built-in, no third-party script). Track:
- Page views per locale
- Upscale conversions (landing to completed upscale)
- Plan conversion rate (free to paid)
- Most popular scales and formats
- Average processing time by device type

Do NOT use Google Analytics or any tracking that requires cookie consent for basic analytics. Vercel Analytics is privacy-friendly and does not use cookies.

For the cookie consent banner: Only needed if Stripe sets cookies (it does for fraud prevention). Show a minimal, non-intrusive banner at the bottom. Text: "This site uses essential cookies for payments and authentication." with an "OK" button. No cookie preferences modal needed since we only use essential cookies.

## Content Guidelines

All user-facing text must:
- Be written in clear, confident language
- Avoid marketing fluff and superlatives
- Use active voice
- Be translated by native speakers (not machine translated)
- Never use dashes in body text (use commas, colons, or separate sentences)
- Never use exclamation marks except in the hero CTA
- Use title case for headings, sentence case for buttons and labels
- Use numbers for statistics and data, words for small quantities in prose

## Accessibility

- All images have descriptive alt text
- All interactive elements are keyboard accessible
- Focus order follows visual order
- Color contrast ratio meets WCAG AA minimum (4.5:1 for text, 3:1 for large text)
- Form inputs have associated labels (not just placeholders)
- Error messages are announced to screen readers via aria-live
- The upscaler progress is communicated via aria-valuenow on the progress bar
- Skip navigation link at the top of every page
- Reduced motion: Respect `prefers-reduced-motion` and disable all animations

## Error Handling

### Client-Side

- WASM load failure: Show clear message with browser requirements
- File too large: Show max size and current file size
- Unsupported format: Show list of accepted formats
- Processing failure: Show "Something went wrong. Please try again." with a retry button
- Network offline: The upscaler works offline after first model load. Show "You are offline. Upscaling still works." message.

### Server-Side

- Stripe webhook signature mismatch: Log error, return 400
- Database connection failure: Return 503 with retry-after header
- Auth session expired: Redirect to sign-in with callback URL
- Rate limiting: Return 429 with retry-after header (100 requests per minute per user)

## Security

- All API routes validate session before processing
- Stripe webhooks verify signature with STRIPE_WEBHOOK_SECRET
- CSRF protection via NextAuth.js built-in
- Content Security Policy headers configured in next.config.ts
- No user-uploaded content is stored on the server (everything is client-side)
- Rate limiting on auth endpoints (10 requests per minute per IP)
- Input validation on all API endpoints using Zod schemas

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Matteo-CB) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
