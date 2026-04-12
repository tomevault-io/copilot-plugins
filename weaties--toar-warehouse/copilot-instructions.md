## toar-warehouse

> Build a cross-platform data capture and management application for **Team Okanagan Animal Rescue**, a small volunteer-run animal rescue organization. The app is used by 2–5 volunteers in the field and admin staff at a desk to capture client and pet information, track rescue status, manage contacts, and export data.

# CLAUDE.md — Okanagan Animal Rescue Data Management App

## Project Overview

Build a cross-platform data capture and management application for **Team Okanagan Animal Rescue**, a small volunteer-run animal rescue organization. The app is used by 2–5 volunteers in the field and admin staff at a desk to capture client and pet information, track rescue status, manage contacts, and export data.

---

## Tech Stack

### Frontend
- **React Native** with **Expo** — single codebase targeting iOS, Android, and web browser (desktop + mobile)
- Use **Expo Router** for file-based navigation
- Use **React Native Paper** or **NativeWind** (Tailwind for React Native) for UI components
- Offline-first: all data entry should work without internet; sync when connection is restored

### Backend & Database
- **Supabase** (PostgreSQL) — free tier, handles auth, real-time sync, storage, and REST/GraphQL API
- **Supabase Storage** for pet photo uploads
- **Supabase Auth** for volunteer login (email/password is fine; no social login needed)

### Sync & Export
- Offline queue using **WatermelonDB** or **MMKV + custom sync layer** syncing to Supabase when online
- CSV export via `react-native-csv` or a simple in-app export screen
- Google Sheets sync: optional stretch goal — use Google Sheets API v4 with a service account to push exported data

### Other Libraries
- `expo-image-picker` — field photo capture
- `expo-camera` — optional direct camera access
- `date-fns` — date formatting and calculations
- `react-hook-form` + `zod` — form state management and validation

---

## Data Models

### `owners` table
```sql
id               UUID PRIMARY KEY DEFAULT gen_random_uuid()
full_name        TEXT                 -- optional
phone_primary    TEXT                 -- optional
email            TEXT                 -- optional
phone_secondary  TEXT                 -- optional
street_address   TEXT                 -- optional
city             TEXT                 -- optional
province         TEXT DEFAULT 'BC'    -- optional
postal_code      TEXT                 -- optional
contact_type     TEXT[]               -- optional: ['adopter', 'foster', 'donor']
notes            TEXT                 -- optional
created_at       TIMESTAMPTZ DEFAULT now()
updated_at       TIMESTAMPTZ DEFAULT now()
```

### `pets` table
```sql
id               UUID PRIMARY KEY DEFAULT gen_random_uuid()
owner_id         UUID REFERENCES owners(id) ON DELETE SET NULL  -- optional
name             TEXT NOT NULL         -- REQUIRED
species          TEXT NOT NULL         -- REQUIRED: 'dog', 'cat', 'other'
species_other    TEXT                  -- optional: if species = 'other'
breed            TEXT                  -- optional
age_years        INTEGER               -- optional
age_months       INTEGER               -- optional
sex              TEXT                  -- optional: 'male', 'female', 'unknown'
colour_markings  TEXT                  -- optional
weight_lbs       NUMERIC(5,1)          -- optional: weight in pounds
microchip_number TEXT                  -- optional
spayed_neutered  BOOLEAN               -- optional (null = unknown)
intake_type      TEXT                  -- optional: 'surrender', 'stray', 'found', 'transfer'
current_status   TEXT NOT NULL DEFAULT 'intake'
  -- values: 'intake', 'vet_check', 'available', 'foster', 'adopted', 'deceased'
medical_notes    TEXT                  -- optional
behavioural_notes TEXT                 -- optional
photo_urls       TEXT[]                -- optional: Supabase Storage URLs
created_at       TIMESTAMPTZ DEFAULT now()
updated_at       TIMESTAMPTZ DEFAULT now()
```

### `vaccinations` table
```sql
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
pet_id      UUID REFERENCES pets(id) ON DELETE CASCADE
type        TEXT NOT NULL -- 'rabies', 'distemper', 'bordetella', 'other'
date_given  DATE
notes       TEXT
```

### `status_log` table
```sql
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
pet_id      UUID REFERENCES pets(id) ON DELETE CASCADE
status      TEXT NOT NULL
changed_at  TIMESTAMPTZ DEFAULT now()
changed_by  UUID REFERENCES auth.users(id)
notes       TEXT
```

---

## App Structure / Screens

```
app/
├── (auth)/
│   └── login.tsx               # Email/password login
├── (tabs)/
│   ├── dashboard.tsx           # Summary counts by status, recent activity
│   ├── pets/
│   │   ├── index.tsx           # Pet list with search + filter
│   │   ├── [id].tsx            # Pet detail view
│   │   ├── new.tsx             # New pet intake form
│   │   └── [id]/edit.tsx       # Edit pet record
│   ├── owners/
│   │   ├── index.tsx           # Owner/client list
│   │   ├── [id].tsx            # Owner detail + their pets
│   │   ├── new.tsx             # New owner form
│   │   └── [id]/edit.tsx       # Edit owner
│   └── admin/
│       ├── export.tsx          # CSV export screen
│       └── settings.tsx        # App settings, Google Sheets config
└── _layout.tsx
```

---

## Key Features & Requirements

### 1. Offline-First Data Entry
- All forms must be usable without internet connectivity
- Queue writes locally; sync to Supabase when back online
- Show a visible sync status indicator (synced / pending / error)

### 2. Pet Intake Form
Fields (in order):
1. Owner selection (search existing or create new inline) — optional
2. Pet name — **required**
3. Species (dog / cat / other) — **required**
4. Breed — optional
5. Age (years + months) — optional
6. Sex (male / female / unknown) — optional
7. Weight in lbs — optional
8. Colour/markings — optional
9. Microchip number — optional
10. Spayed/neutered (yes / no / unknown) — optional
11. Intake type (surrender / stray / found / transfer) — optional
12. Initial status (defaults to "Intake")
13. Vaccinations (rabies, distemper, bordetella — each with date picker) — optional
14. Medical notes — optional
15. Behavioural notes — optional
16. Photos (up to 5, from camera or gallery) — optional

### 3. Owner/Client Form
Owner information is entirely optional — a pet record can be saved with no owner attached. If a person chooses to provide contact info, collect whatever they're comfortable sharing:

1. Full name — optional
2. Primary phone — optional
3. Email — optional
4. Secondary phone — optional
5. Street address, city, province (default BC), postal code — all optional
6. Contact type (multi-select: adopter, foster, donor) — optional
7. Notes — optional

The intake flow should offer a clear **"Skip — no owner info"** path so volunteers don't feel pressure to collect contact details.

### 4. Status Tracking
- Status changes must be logged in `status_log` with timestamp, user, and optional note
- Pet detail screen shows full status history timeline
- Valid transitions shown as buttons/dropdown (don't just allow free-form text)

### 5. Search & Filtering
Pet list should filter by:
- Pet name (text search)
- Species (dog / cat / other)
- Current status
- Intake type

Owner list should filter by:
- Name (text search)
- Contact type (adopter / foster / donor)

### 6. Photo Capture
- Volunteers can add photos from camera or gallery in the field
- Photos upload to Supabase Storage; URLs stored in `pets.photo_urls`
- If offline, photos queued and uploaded on next sync
- Display as a scrollable photo strip on the pet detail screen

### 7. CSV Export
- Export **Contacts list**: owner name, email, phone, address, contact type
- Export **Pets list**: pet name, species, breed, status, owner name, intake type, intake date
- Files saved to device or shared via share sheet
- Accessible from the Admin tab

### 10. Paper Form Photo Capture & AI Pre-fill

This is a key workflow feature. Volunteers can photograph a client's paper intake form and have Claude automatically extract the data and pre-fill the intake form for review before saving.

**Flow:**
1. On the new pet intake screen, volunteer taps **"Scan Paper Form"**
2. Camera opens — volunteer photographs the paper form (one or more pages)
3. Image is sent to the **Claude API** (`claude-sonnet-4-6`) with a prompt instructing it to extract all recognizable fields and return structured JSON
4. The returned JSON pre-populates the intake form fields (owner info + pet demographics)
5. Volunteer reviews every field, makes corrections, and taps **"Save Record"**
6. Nothing is written to the database until the volunteer explicitly confirms

**Claude API Integration:**
- Use `@anthropic-ai/sdk` in a lightweight backend function (Supabase Edge Function) so the API key is never exposed on the client
- Send the image as base64 with `media_type: image/jpeg`
- Prompt should instruct Claude to return only a JSON object with keys matching the data model (owner and pet fields), using `null` for any field it cannot confidently read
- The app should show a **loading/processing state** while the API call is in flight
- If the API call fails or returns unparseable data, fall back gracefully to a blank manual form with a toast notification

**Example extraction prompt (in the Edge Function):**
```
You are helping digitize a paper animal rescue intake form. 
Extract all readable information from this form image and return ONLY a valid JSON object with the following fields. Use null for any field that is blank, illegible, or not present on the form.

{
  "owner": {
    "full_name": null,
    "phone_primary": null,
    "phone_secondary": null,
    "email": null,
    "street_address": null,
    "city": null,
    "province": null,
    "postal_code": null,
    "contact_type": [],
    "notes": null
  },
  "pet": {
    "name": null,
    "species": null,
    "breed": null,
    "age_years": null,
    "age_months": null,
    "sex": null,
    "weight_lbs": null,
    "colour_markings": null,
    "microchip_number": null,
    "spayed_neutered": null,
    "intake_type": null,
    "medical_notes": null,
    "behavioural_notes": null,
    "vaccinations": []
  }
}

Return only the JSON object, no explanation or markdown.
```

**UI details:**
- Pre-filled fields should be visually highlighted (e.g. light blue background) so the volunteer knows which fields were auto-populated vs blank
- Volunteer must actively tab through or scroll past every section before the Save button becomes active (to encourage review)
- The original form photo should be stored alongside the record in Supabase Storage for reference/audit purposes
- Configure a Google Service Account in app settings
- On-demand "Push to Sheets" button in Admin tab
- Pushes the same two datasets (contacts + pets) to a configured Google Sheet
- Replace sheet contents (not append) to avoid duplicates

### 9. Authentication
- Simple email/password login via Supabase Auth
- No registration screen — admin creates volunteer accounts in Supabase dashboard
- Session persists across app restarts
- All routes protected; redirect to login if not authenticated

---

## UI/UX Guidelines

- Design for **field use on mobile first** — large tap targets, minimal typing where possible
- Use dropdowns/pickers instead of free text wherever data is categorical
- Forms should auto-save drafts locally so data isn't lost if the app is backgrounded
- Show loading/saving states clearly
- Use a calm, friendly colour palette appropriate for an animal rescue org (greens/teals/warm neutrals)
- The app name to display is **"OAR Rescue"** (Okanagan Animal Rescue)
- Include the rescue's paw-print style branding if possible (use a simple SVG paw icon as placeholder)

---

## Development Priorities (Build Order)

Build in this order to deliver value incrementally:

1. **Supabase project setup** — schema, RLS policies, storage bucket, auth
2. **Auth flow** — login screen, session management
3. **Owner CRUD** — list, detail, create, edit
4. **Pet CRUD** — list, detail, create (intake form), edit
5. **Status logging** — status change UI + history timeline
6. **Photo capture & upload**
7. **Paper form scan & AI pre-fill** — Supabase Edge Function calling Claude API, review UI
8. **Offline sync layer**
9. **Search & filtering**
10. **CSV export**
11. **Google Sheets sync** (stretch)

---

## Environment Variables

Create a `.env` file (and `.env.example` for the repo):

```
EXPO_PUBLIC_SUPABASE_URL=
EXPO_PUBLIC_SUPABASE_ANON_KEY=
ANTHROPIC_API_KEY=             # for paper form OCR via Claude API (set in Supabase Edge Function secrets, NOT in the app)
GOOGLE_SERVICE_ACCOUNT_JSON=   # for Sheets sync (stretch)
GOOGLE_SHEET_ID=               # target spreadsheet ID
```

> ⚠️ The `ANTHROPIC_API_KEY` must only ever exist in the Supabase Edge Function environment (set via `supabase secrets set`). It must never be bundled into the Expo app or committed to the repository.

---

## Notes for Claude Code

- Always use TypeScript with strict mode enabled
- Define shared types in `types/index.ts` (Owner, Pet, Vaccination, StatusLog, etc.)
- Use Supabase's generated types (`supabase gen types typescript`) for DB types
- Keep business logic in `lib/` (e.g., `lib/sync.ts`, `lib/export.ts`)
- Keep all Supabase calls in `lib/supabase/` service files, not inline in components
- Write RLS (Row Level Security) policies in Supabase so all authenticated users can read/write all records (small team, no role-based restrictions needed yet)
- Add a `README.md` with setup instructions (Supabase project creation, env vars, running the app)
- Do not hardcode any credentials anywhere

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/weaties)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/weaties)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
