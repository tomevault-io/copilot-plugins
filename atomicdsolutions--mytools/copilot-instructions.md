## mytools

> This is an Angular web application. NOT React Native. NOT Flutter. NOT Expo.

# Travel App — Claude Code Project Instructions

## Project Overview
This is an Angular web application. NOT React Native. NOT Flutter. NOT Expo.
Backend: Supabase (auth, database, storage, realtime, edge functions).
Workflow Automation: n8n owns all scheduled jobs, email parsing,
notification dispatch, and external API integrations.
Language: TypeScript strict mode throughout.
Core success drivers: Security and User Experience.

## Tech Stack
- Frontend: Angular (latest stable, standalone components, Angular 17+)
- UI: Angular Material + TailwindCSS
- State: NgRx (global) + Angular signals (local component state)
- Routing: Angular Router with lazy-loaded feature routes
- Forms: Angular Reactive Forms only
- HTTP: Angular HttpClient + auth interceptor + Supabase JS SDK
- Backend: Supabase (PostgreSQL, Auth, Realtime, Storage, Edge Functions)
- Workflow Automation: n8n
- AI: Claude/OpenAI via Supabase Edge Functions ONLY — never from Angular
- Maps: Google Maps JavaScript API + Google Places API
- Payments: Stripe via Supabase Edge Functions ONLY — never from Angular
- Notifications: Web Push via Angular PWA + n8n dispatch
- Error Monitoring: Sentry

## Stack Boundaries — What Goes Where
| Concern | Owner |
|---------|-------|
| UI, routing, forms, user interaction | Angular |
| Database, auth, file storage, realtime | Supabase |
| AI API calls | Supabase Edge Functions |
| Stripe payment processing | Supabase Edge Functions |
| Signed media URL generation | Supabase Edge Functions |
| Email parsing & auto-import | n8n |
| Push notification dispatch | n8n |
| Onboarding email sequences | n8n |
| Budget threshold alerts | n8n (Supabase trigger → n8n webhook) |
| Flight status polling | n8n (scheduled) |
| Testimonial reminders | n8n (scheduled) |
| Bucket list & partner matching | n8n (scheduled + webhook) |
| Wellness nudge scheduling | n8n (scheduled) |

Never call AI or Stripe from Angular.
Never put scheduling or notification dispatch in Angular or Edge Functions.

## Project Structure
```
src/
  app/
    core/           # Auth service, guards, interceptors, Supabase client, models
    shared/         # Shared components, pipes, directives
    features/       # Lazy-loaded feature modules
      auth/         # Login, signup, onboarding quiz
      dashboard/    # Home dashboard
      trips/        # Trip creation, itinerary
      group/        # Group chat, location sharing
      discovery/    # Nightlife discovery
      flights/      # Flight & travel tracking
      expenses/     # Expense tracking
      wellness/     # Wellness nudges, checklists
      media/        # Gallery, testimonials
      ai-assistant/ # AI trip chat & knowledge base
      subscription/ # Premium tier
      profile/      # User profile & settings
    app.routes.ts
    app.config.ts
  environments/
supabase/
  functions/        # Edge Functions
  migrations/       # SQL with RLS
n8n/
  workflows/        # Exported n8n workflow JSON (version controlled)
  README.md
```

## Angular Rules (Non-Negotiable)
- Standalone components only (Angular 17+)
- Lazy load every feature route
- Angular Reactive Forms only — never template-driven
- NgRx for global state: auth, subscription status, active trip
- Angular signals for local component state
- TypeScript strict mode — no `any` types ever
- Unsubscribe all Realtime subscriptions on destroy (takeUntilDestroyed)
- AuthGuard on every private route
- AuthInterceptor injects bearer token on all Supabase API calls

## Security Rules (Non-Negotiable)
- NEVER store JWT in localStorage or sessionStorage
- RLS enabled on every Supabase table — no exceptions
- All AI calls via Edge Functions only — never from Angular
- All Stripe calls via Edge Functions only — never from Angular
- All n8n webhooks validated with x-webhook-secret header
- Never commit API keys — use environment.ts for client-safe vars only

## Database Schema

### Core Tables
```sql
users (id, email, display_name, avatar_url, avatar_type, subscription_tier, created_at)
user_devices (id, user_id, push_token, platform, updated_at)
travel_profiles (user_id, frequency, travel_type, travel_style[], frustrations, bucket_list[], priorities[])
trips (id, name, destination, start_date, end_date, organizer_id, budget, budget_currency, created_at)
trip_members (trip_id, user_id, role ENUM['organizer','co_organizer','member'], joined_at)
flight_segments (id, trip_id, user_id, airline, flight_number, origin, destination, departure_time, arrival_time, status, gate, terminal, confirmation_ref)
hotel_segments (id, trip_id, user_id, hotel_name, address, check_in, check_out, confirmation_number, phone)
group_messages (id, trip_id, sender_id, content, type ENUM['text','image'], sent_at, is_archived)
direct_messages (id, trip_id, sender_id, recipient_id, content, sent_at)
expenses (id, trip_id, user_id, amount, currency, home_currency_amount, category, type ENUM['personal','business'], receipt_url, vendor, notes, created_at)
trip_activities (id, trip_id, organizer_id, title, description, cost, currency, deadline, is_optional)
activity_optins (activity_id, user_id, payment_status, opted_at)
saved_spots (id, user_id, place_id, place_name, city, category, vibe_note, created_at)
community_favorites (place_id, city, favorite_count, last_updated)
bucket_list (id, user_id, destination, event_type, created_at)
wellness_preferences (user_id, notifications_enabled, frequency ENUM['daily','occasional','off'])
onboarding_responses (session_token, user_id NULLABLE, responses JSONB, created_at)
partner_trips (id, partner_id, destination, event_type, start_date, end_date, cost, description, created_at)
```

### AI Trip Assistant Tables
```sql
trip_knowledge_chunks (id, trip_id, source_file, template_type, content TEXT, embedding vector(1536), created_at)
trip_ai_messages (id, trip_id, user_id, role ENUM['user','assistant','organizer'], content, was_escalated BOOLEAN, was_flagged BOOLEAN, created_at)
trip_routing_rules (id, trip_id, organizer_id, keywords TEXT[], categories TEXT[], created_at)
trip_escalations (id, trip_id, message_id, member_id, organizer_id, status ENUM['pending','responded','resolved'], created_at)
```

### Media Sharing Tables
```sql
trip_media (id, trip_id, uploader_id, file_path, file_name, file_type ENUM['photo','video'], file_size_bytes, mime_type, trip_day, like_count, comment_count, created_at)
trip_media_likes (media_id, user_id, created_at)
trip_media_comments (id, media_id, user_id, content, created_at)
trip_media_tags (id, media_id, tagged_by, tagged_user, created_at)
trip_testimonials (id, trip_id, member_id, type ENUM['text','voice','video','photo'], content, file_path, is_approved, is_hidden, created_at)
trip_gallery_settings (trip_id PK, testimonials_enabled, testimonials_enabled_at, enabled_by)
```

### n8n Workflow Support Tables
```sql
notifications_log (id, user_id, type, payload JSONB, sent_at)
budget_alerts_sent (trip_id, user_id, threshold_percent, sent_at)
flight_status_log (flight_id, old_status, new_status, logged_at)
onboarding_emails_sent (user_id, sequence_step, sent_at)
partner_notifications_sent (user_id, partner_trip_id, sent_at)
wellness_nudges_sent (user_id, trip_id, sent_at, message)
```

### RLS Rules (Every Table)
- Users can only read/write their own data
- Trip data visible only to trip members (check trip_members table)
- Expense data accessible only to the owning user
- Group messages accessible only to trip members
- AI knowledge chunks: members read, organizer write only
- Media: members read, own uploader delete, organizer delete any
- n8n uses service role key — do NOT create other RLS bypasses

## n8n Workflows (n8n owns these — do not reimplement elsewhere)
1. Email parsing & travel confirmation auto-import (IMAP → AI → Supabase insert)
2. Web push notification dispatch (Supabase webhook → n8n → Expo Push API)
3. Onboarding email sequences (welcome, feature highlights, upgrade prompt)
4. Budget threshold alerts (Supabase DB trigger → n8n → push + email)
5. Flight status polling (n8n cron → AviationStack → Supabase → push)
6. Testimonial collection reminders (webhook → n8n sequence)
7. Partner trip & bucket list matching (n8n cron + webhook → push + email)
8. Wellness nudge scheduling (n8n cron → AI generate → push, max 1/day)

All workflow JSON files version-controlled in /n8n/workflows/.
Supabase DB triggers POST to n8n webhooks with x-webhook-secret header.

## Supabase Edge Functions
ai-trip-chat, ai-embed-document, ai-suggest-questions, ai-route-question,
ai-vibe-discovery, ai-checklist-generator, ai-group-summary (called by n8n),
ai-email-parser (called by n8n), stripe-create-subscription,
stripe-verify-receipt, media-upload, media-signed-url, media-download-zip

## Environment Variables
```
# environment.ts (Angular — client-safe only)
supabaseUrl=
supabaseAnonKey=
googleMapsApiKey=
sentryDsn=

# Supabase Edge Function secrets (server only — never in Angular)
SUPABASE_SERVICE_ROLE_KEY=
ANTHROPIC_API_KEY=
OPENAI_API_KEY=
STRIPE_SECRET_KEY=
AVIATION_API_KEY=

# n8n credentials (stored in n8n only — never in code)
N8N_WEBHOOK_SECRET=
SENDGRID_API_KEY=
IMAP_EMAIL=
IMAP_PASSWORD=
```

## Phase 1 MVP Scope (Build These First)
1. Angular scaffold + Material + TailwindCSS + NgRx
2. Supabase setup (all tables + RLS + pgvector extension)
3. Onboarding quiz (8 questions, animated, anonymous session token)
4. Authentication (email, Google, Apple, phone OTP, biometric)
5. Trip creation + invites + shared itinerary
6. Real-time group chat (Supabase Realtime)
7. Manual flight + hotel entry
8. Google Places nightlife browse + filter
9. Manual expense entry + budget tracker
10. AI-generated departure checklists (Edge Function)
11. n8n wellness nudge workflow setup
12. n8n onboarding email sequence workflow setup
13. Stripe subscription (free vs premium) via Edge Function
14. Web push notification registration → user_devices table
15. Privacy controls (location opt-in, data deletion)

## Key Constraints (Never Violate)
- NEVER call AI or Stripe from Angular
- NEVER store JWT in localStorage or sessionStorage
- NEVER put scheduling or notification dispatch anywhere except n8n
- NEVER put email sequences anywhere except n8n
- NEVER make location data persistent beyond trip end + 24 hours
- NEVER add partner integrations in Phase 1
- ALWAYS enforce RLS on every Supabase table
- ALWAYS validate n8n webhook requests with x-webhook-secret
- ALWAYS lazy load feature routes
- ALWAYS use standalone Angular components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atomicdsolutions)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/atomicdsolutions)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
