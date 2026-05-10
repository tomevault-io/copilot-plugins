## co-study-scheduler

> A multi-tenant real-time scheduling tool where anyone can create a "room," get a shareable link, and let others book 2-hour co-study sessions. All times auto-convert to the viewer's local timezone. The host shares the link with their study group; members open it, see available slots in their own timezone, and book.

# Co-Study Scheduler

## Project Overview
A multi-tenant real-time scheduling tool where anyone can create a "room," get a shareable link, and let others book 2-hour co-study sessions. All times auto-convert to the viewer's local timezone. The host shares the link with their study group; members open it, see available slots in their own timezone, and book.

## Tech Stack
- **Frontend:** Vite + React + Tailwind CSS
- **Database/Backend:** Supabase (PostgreSQL + Realtime + Edge Functions + RLS)
- **Hosting:** Vercel (static build deployment)
- **Email:** Resend (transactional emails via Supabase Edge Functions)
- **Routing:** react-router-dom v6+
- **Timezone handling:** date-fns + date-fns-tz (NOT moment.js)
- **Language:** JavaScript (not TypeScript for v1 — keep it simple)

## Pages & Routes

### 1. Landing Page (`/`)
- Hero section explaining what the tool does
- "Create a Room" CTA button
- Brief "How it works" section (3 steps: create room → share link → study together)

### 2. Create Room (`/create`)
- Form fields:
  - Host name (required)
  - Room title (required, e.g., "AI/ML Co-Study with Deo")
  - Description (optional)
  - Slug (auto-generated from title, editable, validated for uniqueness)
  - Zoom/meeting link (required — private, never shown on public board, delivered only via email/confirmation)
  - Host timezone (auto-detected, editable dropdown)
  - Morning/afternoon window: start hour, end hour (defaults: 10 AM – 3 PM)
  - Evening window: start hour, end hour (defaults: 7 PM – 11 PM)
  - Slot duration: fixed at 120 minutes for v1
  - Slot interval: 30 minutes (rolling start times)
  - Admin PIN (required, 4-6 digits, for accessing admin view)
- On submit: create room in Supabase, redirect to `/r/:slug`

### 3. Public Room View (`/r/:slug`)
- Room header: title, host name, description, session format reminder
- Timezone indicator: auto-detected local TZ with option to change
- Week navigator: prev/next week arrows, current week range display
- Slot grid:
  - Columns = Mon–Fri (with dates)
  - Rows = available 2-hour slots at 30-min intervals within configured windows
  - Morning/afternoon and evening sections visually separated
  - Cell states: Open (green), Booked (orange/amber, shows booker's name), Blocked/overlap (gray), Past (dimmed)
- Click open slot → booking form appears
- Booking form (3 fields):
  - Name (required)
  - Email (optional — helper text: "Optional — to receive your Zoom link by email")
  - Private note to host (optional — helper text: "Only the host sees this")
- After booking → confirmation screen:
  - Shows date/time in booker's timezone
  - If email provided: "You're booked! You'll receive a Zoom meeting link at [email] before the session."
  - If no email: "You're booked! Deo will share your Zoom meeting link via direct message in the AI/ML Career Launchpad group."
  - Session format reminder: 5–10 min hello & goals → ~100 min focused study → 10–15 min wrap-up & chat
- Real-time updates: when someone else books while you're viewing, the board updates live via Supabase Realtime

### 4. Admin View (`/r/:slug/admin`)
- Protected by admin PIN (simple form, stored in room record)
- Shows everything on public board PLUS:
  - Booker's email (if provided)
  - Private note (if provided)
  - Cancel booking button per slot
  - Blackout: ability to mark specific dates as unavailable
- Summary section: total bookings this week, upcoming next session

## Database Schema

All timestamps stored in UTC. Host timezone stored as IANA string (e.g., `America/Chicago`).

```sql
-- Rooms table
create table rooms (
  id uuid default gen_random_uuid() primary key,
  slug text unique not null,
  host_name text not null,
  title text not null,
  description text,
  meeting_link text not null,
  host_timezone text default 'America/Chicago',
  morning_start int default 10,
  morning_end int default 15,
  evening_start int default 19,
  evening_end int default 23,
  slot_duration int default 120,
  slot_interval int default 30,
  admin_pin text not null,
  is_active boolean default true,
  created_at timestamptz default now()
);

-- Bookings table
create table bookings (
  id uuid default gen_random_uuid() primary key,
  room_id uuid references rooms(id) on delete cascade,
  name text not null,
  email text,
  note text,
  booking_date date not null,
  slot_start_utc timestamptz not null,
  slot_end_utc timestamptz not null,
  booker_timezone text,
  status text default 'confirmed' check (status in ('confirmed', 'cancelled')),
  created_at timestamptz default now()
);

-- Indexes
create index idx_bookings_room_date on bookings(room_id, booking_date);
create index idx_rooms_slug on rooms(slug);

-- Row Level Security
alter table rooms enable row level security;
alter table bookings enable row level security;

-- Public read access for rooms
create policy "Rooms are publicly readable"
  on rooms for select using (is_active = true);

-- Public read access for confirmed bookings
create policy "Confirmed bookings are publicly readable"
  on bookings for select using (status = 'confirmed');

-- Public insert for rooms (anyone can create)
create policy "Anyone can create a room"
  on rooms for insert with check (true);

-- Public insert for bookings (anyone can book)
create policy "Anyone can book a slot"
  on bookings for insert with check (true);

-- Public update for bookings (for cancellation — validated in app layer)
create policy "Bookings can be updated"
  on bookings for update using (true);
```

## Project Structure

```
co-study-scheduler/
├── src/
│   ├── components/
│   │   ├── WeekNavigator.jsx
│   │   ├── SlotGrid.jsx
│   │   ├── SlotCell.jsx
│   │   ├── BookingForm.jsx
│   │   ├── BookingSummary.jsx
│   │   ├── TimezoneIndicator.jsx
│   │   ├── ConfirmationScreen.jsx
│   │   ├── AdminPinForm.jsx
│   │   └── Layout.jsx
│   ├── pages/
│   │   ├── Home.jsx
│   │   ├── CreateRoom.jsx
│   │   ├── Room.jsx
│   │   └── Admin.jsx
│   ├── hooks/
│   │   ├── useBookings.js
│   │   ├── useRoom.js
│   │   └── useTimezone.js
│   ├── lib/
│   │   ├── supabase.js
│   │   ├── slots.js
│   │   └── timezone.js
│   ├── App.jsx
│   └── main.jsx
├── supabase/
│   └── migrations/
│       └── 001_create_tables.sql
│   └── functions/
│       └── notify-booking/
│           └── index.ts
├── docs/
│   └── ARCHITECTURE.md
├── public/
├── .env.example
├── .eslintrc.cjs
├── .prettierrc
├── tailwind.config.js
├── vite.config.js
├── package.json
├── CLAUDE.md
├── build_journal.md
└── README.md
```

## Key Technical Decisions

### Timezone Handling
- Store all booking times as UTC `timestamptz` in Supabase
- Store the room host's timezone as an IANA timezone string (e.g., `America/Chicago`)
- On the frontend, detect the viewer's timezone using `Intl.DateTimeFormat().resolvedOptions().timeZone`
- Use `date-fns-tz` to convert between UTC ↔ host timezone ↔ viewer timezone
- Slot definitions (10 AM, 10:30 AM, etc.) are always relative to the host's timezone
- When rendering, convert host-TZ slot times → UTC → viewer's local TZ for display
- When storing a booking, convert the selected slot from host-TZ → UTC for storage

### Slot Generation Logic
- Slots are 2-hour fixed duration with 30-minute rolling start times
- Morning/afternoon window example (10 AM – 3 PM host TZ): generates slots at 10:00, 10:30, 11:00, 11:30, 12:00, 12:30, 1:00
- Evening window example (7 PM – 11 PM host TZ): generates slots at 7:00, 7:30, 8:00, 8:30, 9:00
- A slot's last valid start time = window_end - slot_duration (e.g., 3 PM - 2 hrs = 1 PM last start)

### Overlap Detection
- Two slots overlap if: slotA.start < slotB.end AND slotB.start < slotA.end
- Before inserting a booking, query existing confirmed bookings for that room + date
- Check the requested slot against all existing bookings for overlaps
- Reject with clear error message if overlap detected

### Real-time Updates
- Subscribe to Supabase Realtime on the `bookings` table filtered by `room_id`
- On INSERT or UPDATE events, refetch the current week's bookings
- This ensures all viewers see bookings appear/cancel live without page refresh

### Notifications (Supabase Edge Functions + Resend)
- On booking insert, a Supabase Edge Function triggers
- Sends email to host: "New booking: [name] booked [slot] on [date]" + email + note if provided
- If booker provided email: sends confirmation email with date/time in their TZ + Zoom link
- If booker did not provide email: no email sent, host reaches out via group DM
- Use Resend free tier (100 emails/day)

## Coding Conventions
- Use functional components with hooks (no class components)
- Use `const` for components and functions (arrow functions)
- Destructure props
- Custom hooks prefixed with `use`
- File names: PascalCase for components, camelCase for utilities/hooks
- Keep components focused — extract when a component exceeds ~150 lines
- No inline styles — use Tailwind utility classes exclusively
- Use semantic HTML elements where appropriate
- Error boundaries around data-fetching components
- Console errors/warnings should be zero in production

## Environment Variables
```
VITE_SUPABASE_URL=your_supabase_project_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
```

## What NOT to Build in v1
- User authentication / login system
- Chat or messaging between users
- Calendar sync (Google Calendar, iCal)
- Recurring availability patterns
- Arbitrary custom-duration sessions
- Social profiles
- Payment system
- Native mobile app
- Dark mode (unless trivial to add via Tailwind)
- Analytics dashboard
- Push notifications (email only for v1)

## Development Workflow
1. Branch from `main` for each feature
2. Test locally with `npm run dev`
3. Commit with clear messages (e.g., "feat: add slot grid with overlap detection")
4. Push to GitHub → Vercel auto-deploys preview
5. Merge to `main` → production deploy

## Build Phases
1. **Scaffolding:** Vite + React + Tailwind + react-router + Supabase client setup
2. **Database:** Run migration, configure RLS policies
3. **Room creation:** Create room form + Supabase insert + redirect
4. **Slot grid:** Week navigator + slot generation + timezone conversion + display
5. **Booking flow:** Click slot → form → overlap check → insert → confirmation screen
6. **Real-time:** Supabase Realtime subscription for live updates
7. **Admin view:** PIN-protected admin page with cancel + email/note visibility
8. **Notifications:** Edge Function + Resend for host + booker emails
9. **Polish:** Mobile responsiveness, error handling, loading states, edge cases
10. **Deploy:** GitHub → Vercel, environment variables, production URL

---
> Source: [deepakdeo/co-study-scheduler](https://github.com/deepakdeo/co-study-scheduler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
