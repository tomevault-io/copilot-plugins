## zachsruns

> This is a basketball pickup game ("runs") organizer web app. A "run" is basketball slang for a scheduled pickup game session. The app allows users to RSVP for runs and admins to manage the community.

# Zach's Runs - AI Assistant Rules

## Project Overview
This is a basketball pickup game ("runs") organizer web app. A "run" is basketball slang for a scheduled pickup game session. The app allows users to RSVP for runs and admins to manage the community.

## Tech Stack
- **Frontend**: Next.js 16 with TypeScript, Tailwind CSS, App Router
- **Backend**: Flask (Python) with SQLAlchemy ORM
- **Database**: SQLite (local) / Neon Postgres (production via Vercel)
- **Deployment**: Vercel (frontend + serverless Python backend)
- **Email**: Resend API with QStash queue (Upstash)

## Email System Architecture

### How It Works
- **Local Development**: In-memory queue with background daemon thread (1 email/sec rate limit)
- **Production**: QStash (Upstash) handles scheduling and delivery to worker endpoint
- **Fire-and-Forget**: API returns immediately; emails processed async

### Email Flow
```
Local:      API → Queue Email → Return Success → Background Thread (1/sec)
Production: API → Enqueue to QStash → Return Success → QStash → /api/email-worker
```

### Email Types Implemented
1. Welcome Email (on signup - sent even if not verified)
2. Account Verified Email (when admin verifies user)
3. Admin New User Notification (to all admins on signup)
4. Run Created Email (to verified users)
5. Run Reminder Email (manual admin trigger)
6. Run Completed Email (to attendees)
7. Run Cancelled Email (when run deleted)
8. Run Modified Email (when run updated, shows what changed)
9. Announcement Email (to verified users)

### Key Files
- `backend/utils/email.py` - Email utilities, queue system, send functions
- `backend/routes/email_worker.py` - QStash worker endpoint (`/api/email-worker`)
- `backend/templates/emails/` - Jinja2 HTML email templates

### Environment Variables for Email
```bash
RESEND_API_KEY=xxx                    # Resend API key
EMAIL_FROM_ADDRESS=notifications@send.zachsruns.com
EMAIL_FROM_NAME="Zach's Organized Runs"
FRONTEND_URL=https://zachsruns.vercel.app
QSTASH_TOKEN=xxx                      # Production only
QSTASH_CURRENT_SIGNING_KEY=xxx        # Production only
QSTASH_NEXT_SIGNING_KEY=xxx           # Production only
LOCAL_USE_QSTASH=true                 # Optional: test QStash locally (requires ngrok)
QSTASH_CALLBACK_URL=xxx               # Your ngrok URL for local QStash testing
```

### Testing QStash Locally with ngrok

The local threading system works for most cases, but to test the actual QStash queue implementation (signature verification, scheduling, retries), use ngrok:

**Terminal 1 - Start ngrok:**
```bash
ngrok http 5001
# Copy the HTTPS URL (e.g., https://abc123.ngrok-free.app)
# ⚠️ FREE NGROK: URL changes every restart! Update .env each time.
```

**Terminal 2 - Backend with QStash:**
```bash
cd backend
source venv/bin/activate
# Edit .env with the NEW ngrok URL from Terminal 1:
#   LOCAL_USE_QSTASH=true
#   QSTASH_CALLBACK_URL=https://abc123.ngrok-free.app  ← paste new URL here
python app.py
# ⚠️ Must restart backend if ngrok URL changes
```

**Terminal 3 - Frontend:**
```bash
cd frontend
npm run dev
```

**How it works:**
1. Backend enqueues emails to QStash (external service)
2. QStash calls back to your ngrok URL → local backend `/api/email-worker`
3. Worker verifies signature and sends email via Resend
4. Email links point to `localhost:3000` (via `FRONTEND_URL`)

**When to use:** Testing QStash-specific behavior (signature verification, scheduled delays, retry logic). For normal development, the default threading system is simpler.

### Email Conventions
- Only verified users receive emails (exception: welcome email on signup)
- Templates extend `base.html` with basketball theme styling
- All run emails include "View Runs →" link to main site
- Local dev filters emails to real addresses only (`zmann`, `rantesting22`)

## Project Structure
```
ZachsRuns/
├── frontend/           # Next.js app
│   ├── app/           # App Router pages
│   ├── components/    # React components
│   ├── context/       # React contexts (AuthContext)
│   ├── lib/           # API client, auth utilities
│   └── types/         # TypeScript interfaces
├── backend/           # Flask API
│   ├── routes/        # Blueprint route handlers
│   ├── models.py      # SQLAlchemy models
│   ├── database.py    # DB initialization
│   ├── middleware.py  # Auth middleware
│   └── utils/         # Email utilities
└── frontend/api/      # Vercel serverless entry point
```

## Code Style & Conventions

### Frontend (TypeScript/React)
- Use `'use client'` directive for components with hooks/interactivity
- Import types from `@/types` (path alias configured)
- Use `@/lib/api` functions for all API calls (never raw fetch)
- Use `@/context/AuthContext` for auth state via `useAuth()` hook
- Tailwind classes for styling - use custom colors from tailwind.config.js
- Component files are PascalCase (e.g., `RunCard.tsx`)
- Page files use Next.js App Router convention (`page.tsx`)

### Custom Tailwind Colors
```js
'basketball-orange': '#FF6B35'
'basketball-black': '#1A1A1A'
'wood-light': '#D4A574'
'wood-medium': '#8B6F47'
'wood-dark': '#5C4A37'
```

### Backend (Python/Flask)
- Use Flask Blueprints for route organization
- Models in `models.py` with `to_dict()` methods for serialization
- Use `@require_auth` decorator from middleware.py for protected routes
- Use `@require_admin` decorator for admin-only routes
- Access current user via `request.current_user` in protected routes
- Always return JSON responses with appropriate status codes
- Log errors with `logger.error()`, don't expose internal errors to users

### API Patterns
- All endpoints prefixed with `/api/`
- Auth endpoints: `/api/auth/signup`, `/api/auth/login`, `/api/auth/me`
- Resource endpoints follow REST: `/api/runs`, `/api/runs/:id`
- Return `{ error: string }` for errors with appropriate HTTP status
- Return `{ message: string, resource: object }` for success

### Database Models
- User: username, email, badge, is_verified, is_admin, runs_attended_count, no_shows_count
- Run: title, date, start_time, end_time, location_id, is_completed, is_historical
- RunParticipant: links users to runs with status (confirmed/interested/out)
- Location: name, address, description, image_url
- Announcement: message, is_active

### Badge System
Badges are manually assigned by admin:
- `regular` - Consistent regular at runs
- `plus_one` - Guest referred by a member (has `referred_by` field)
- `null` - No badge assigned

## Domain Terminology
- **Run**: A scheduled basketball pickup game session
- **RSVP statuses**: confirmed, interested, out
- **Completed run**: Past run with attendance marked
- **Historical run**: Imported data from before the app existed
- **Verified user**: Approved by admin to RSVP for runs

## Common Patterns

### Frontend API Call Pattern
```typescript
const [loading, setLoading] = useState(false);
try {
  setLoading(true);
  const data = await runsApi.getAll();
  // handle success
} catch (error: any) {
  console.error('Error:', error);
  alert(error?.message || 'Something went wrong');
} finally {
  setLoading(false);
}
```

### Backend Route Pattern
```python
@blueprint.route('/resource', methods=['POST'])
@require_auth
def create_resource():
    data = request.get_json()
    if not data or not data.get('required_field'):
        return jsonify({'error': 'Required field missing'}), 400
    
    try:
        # create resource
        db.session.commit()
        return jsonify({'message': 'Created', 'resource': resource.to_dict()}), 201
    except Exception as e:
        db.session.rollback()
        logger.error(f"Error: {str(e)}")
        return jsonify({'error': 'Failed to create'}), 500
```

## Important Notes
- User base is small (<40 users) - keep solutions simple
- Local dev uses SQLite; production uses Postgres (be database-agnostic)
- JWT tokens stored in localStorage, sent as Bearer token
- CORS configured for localhost:3000 and production domains
- Email sending is async via QStash in production, threading locally

## Don't
- Don't add complex caching or optimization - app is low traffic
- Don't change database schema without considering both SQLite and Postgres
- Don't expose password_hash in User.to_dict()
- Don't allow unverified users to RSVP (frontend + backend enforce this)
- Don't use raw SQL - use SQLAlchemy ORM

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZachManno) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
