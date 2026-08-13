## madhuban

> These instructions apply to the entire repository rooted at this directory.

# Madhuban Garden Resort Website

## Scope

These instructions apply to the entire repository rooted at this directory.

## Project Overview

- Build a full-stack resort website for **Madhuban Garden Resort** in **Agar Malwa District, Madhya Pradesh, India**.
- Domain: `madhubangarden.com`
- This is a client project. Build clean, production-grade code. No shortcuts.

## Current Build Phase

- **Phase 1: Frontend with dummy data only**
- Build all pages with static or dummy data first.
- Do not connect the real database yet.
- Do not integrate real payments yet.
- Goal: get the full UI approved by the client before backend work begins.

## UI Design Reference

- All screen designs, layouts, and visual references are inside the `/UI design/` folder at the project root.
- Before building any page or component, open and read the relevant file in `/UI design/` first.
- Match the design as closely as possible.
- Do not invent layouts that are not in the design files.

## Required Tech Stack

Do not deviate from this stack:

| Layer         | Tool                       |
| ------------- | -------------------------- |
| Framework     | Next.js 14 (App Router)    |
| Styling       | Tailwind CSS               |
| UI Components | shadcn/ui                  |
| Animations    | Framer Motion              |
| Icons         | Lucide React               |
| Database      | Supabase (PostgreSQL)      |
| ORM           | Prisma                     |
| Validation    | Zod                        |
| Auth          | Supabase Auth              |
| Payments      | Razorpay                   |
| Email         | Resend                     |
| Caching       | Upstash Redis              |
| iCal Sync     | node-ical + ical-generator |
| Hosting       | Vercel                     |

## Design System

### Theme

- Style: Crayons Light Green
- Feel: light, fresh, natural, premium, peaceful, lush, resort-like
- Do not make the site feel corporate or generic.
- Use an elegant serif for headings and a clean sans-serif for body text.
- Do not use Inter, Roboto, or Arial.

### Colors

- Primary: `#4CAF50`
- Background: `#f5f9f0`
- Accent: `#2e7d32`
- Text: `#1a1a1a`
- White: `#ffffff`

### Design Rules

- Mobile-first and fully responsive
- Smooth Framer Motion page transitions
- Use Next.js `Image` for optimized images
- No purple gradients
- No generic AI aesthetics
- Layouts should feel light, airy, and nature-inspired

## Tagline

Use this exact tagline across the site, including hero section, meta description, and OG tags:

> "The most peaceful & lush green premises in Agar Malwa District."

## Pages To Build

| Route           | Page                        | Priority |
| --------------- | --------------------------- | -------- |
| `/`             | Homepage                    | High     |
| `/rooms`        | All Rooms (19)              | High     |
| `/rooms/[slug]` | Individual Room             | High     |
| `/wedding`      | Wedding Venue               | High     |
| `/contact`      | Contact + Query Form        | High     |
| `/banquet`      | Banquet Hall                | Medium   |
| `/restaurant`   | Indoor + Outdoor Restaurant | Medium   |
| `/pool`         | Swimming Pool               | Medium   |
| `/events`       | Small Events + Parties      | Medium   |
| `/gallery`      | Image Gallery               | Medium   |
| `/attractions`  | Nearby Attractions          | Low      |
| `/admin`        | Admin Dashboard (protected) | Low      |

## Homepage Sections

Build homepage sections in this order:

1. Hero: full-screen, tagline, Book Now CTA, background resort image
2. Quick Highlights: Rooms, Banquet, Pool, Restaurant icon cards
3. Wedding Feature: big bold section with "Make your wedding unforgettable"
4. Rooms Preview: 3 featured rooms with Book Now
5. Core Services: Hotel, Banquet, Restaurant, Pool, Events
6. Amenities Strip: Parking, WiFi, Laundry, In-Room Dining, Breakfast
7. Instagram Feed: Behold.so embed
8. Google Reviews: Google Places API source later
9. Nearby Attractions: Maa Baglabukhi Temple, Mahakaleshwar Temple
10. Footer: links, contact, WhatsApp button, social links

## Core Services

- Hotel Rooms: 19 rooms, likely increasing later
- Banquet Hall: 1 hall, likely increasing later
- Indoor Restaurant
- Outdoor Restaurant
- Swimming Pool: 1

## Addons And Events

- Birthday Parties
- Small Parties
- Catering
- Event Management
- Decoration
- Corporate Meets & Conferences

## Amenities

- Indoor Parking
- Free WiFi
- Laundry Service
- In-Room Dining
- Complimentary Breakfast

## Major Selling Point

- Wedding Venue is the number one revenue driver.
- The `/wedding` page must be the most polished and detailed page on the site.
- Feature the wedding offering prominently on the homepage.
- Use emotional language appropriately.
- Show venue photos, capacity, and packages in the final implementation.

## Booking System

Planned booking flow:

1. User selects room plus check-in and check-out dates
2. System checks availability via `/api/availability` and Supabase later
3. User fills guest details: name, phone, email
4. User chooses payment method:
   - Pay Now: Razorpay checkout, webhook confirms, booking saved
   - Pay at Reception: booking saved with `pending` status
5. Show confirmation on screen and send email via Resend

### Payment Requirements

- Gateway: Razorpay
- Support UPI, Cards, Net Banking, Wallets
- Webhook route: `/api/payments/webhook`

## iCal Sync Requirements

- Do not use Djubo or any paid channel manager.
- Build a custom iCal-based sync system.
- Our system must expose an iCal feed at `madhubangarden.com/api/ical/export`
- Booking.com and MMT will pull this feed from their extranets
- Our system will pull iCal feeds from Booking.com and MMT every 30 minutes
- Sync will run via Vercel Cron Job at `/api/cron/sync-ical`
- Use `node-ical` to parse incoming feeds
- Use `ical-generator` to generate outgoing feeds
- Cache availability in Upstash Redis

## Planned Database Schema

```sql
rooms (
  id, slug, name, type, description,
  price_per_night, capacity, images[],
  amenities[], is_active
)

bookings (
  id, room_id, guest_name, guest_phone, guest_email,
  check_in, check_out, guests_count,
  payment_method, payment_status, razorpay_order_id,
  source (website/booking_com/mmt/manual),
  status (confirmed/pending/cancelled),
  created_at
)

inquiries (
  id, name, phone, email,
  event_type (wedding/birthday/corporate/other),
  event_date, guests_count, message,
  status (new/contacted/closed),
  created_at
)

blocked_dates (
  id, room_id, date, source (ical/manual)
)

gallery (
  id, image_url, category (rooms/wedding/events/pool/restaurant),
  caption, sort_order
)

reviews (
  id, guest_name, rating, review_text,
  source (google/manual), created_at
)
```

## Admin Dashboard

Protect `/admin` using Supabase Auth with email plus OTP login.

Sections to build later:

- Bookings: view all, filter by date/status/source, update status
- Availability: calendar view, manually block/unblock dates
- Inquiries: view all wedding and event inquiries, mark as contacted
- Gallery: upload, delete, reorder images
- Rooms: edit room details, pricing, images
- iCal Sync: show last sync time, manual trigger button, sync logs

## API Routes

```txt
GET  /api/availability?room_id=&check_in=&check_out=
POST /api/bookings
GET  /api/bookings
POST /api/payments/order
POST /api/payments/webhook
POST /api/inquiry
GET  /api/ical/export
POST /api/cron/sync-ical
```

## Environment Variables

```env
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
DATABASE_URL=
RAZORPAY_KEY_ID=
RAZORPAY_KEY_SECRET=
RAZORPAY_WEBHOOK_SECRET=
RESEND_API_KEY=
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=
BOOKINGCOM_ICAL_URL=
MMT_ICAL_URL=
NEXT_PUBLIC_GOOGLE_PLACES_API_KEY=
NEXT_PUBLIC_BEHOLD_FEED_ID=
CRON_SECRET=
```

## What Not To Do

- Do not use WordPress or any CMS
- Do not use the Pages Router
- Use the App Router only
- Do not use plain CSS or CSS modules
- Use Tailwind CSS only
- Do not use paid plugins or services beyond the approved stack
- Do not use generic fonts like Inter, Roboto, or Arial
- Do not skip mobile responsiveness
- Do not hardcode credentials
- Use `.env.local` for secrets
- Do not build backend before frontend is approved
- Do not invent UI
- Always check `/UI design/` before implementing

## Project Info

- Client: Madhuban Garden Resort, Agar Malwa District, MP
- Agency: Xternal Media
- Domain: `madhubangarden.com`

---
> Source: [xternalmedia/Madhuban](https://github.com/xternalmedia/Madhuban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
