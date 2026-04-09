## gritful

> **THIS APP CONTAINS REAL USER DATA IN BOTH LOCAL AND PRODUCTION DATABASES**

# Gritful - Claude Notes & Pinned Issues

## ⚠️ CRITICAL: Data Safety - READ THIS FIRST ⚠️

### 🚨 ABSOLUTE RULE: NEVER RUN `supabase db reset` 🚨

**THIS APP CONTAINS REAL USER DATA IN BOTH LOCAL AND PRODUCTION DATABASES**

⛔️ **STOP! DO NOT RUN `supabase db reset` - EVER!** ⛔️

This command will **PERMANENTLY DELETE ALL USER DATA**:
- All user accounts and profiles
- All challenges and participant data
- All daily entries and progress tracking
- All points, streaks, and achievements
- ALL EMAIL HISTORY AND MESSAGES

**There is NO undo. Data cannot be recovered.**

### ✅ SAFE Database Migration Process

**ALWAYS use these safe methods:**

1. **For New Migrations** (PREFERRED METHOD - Use Supabase Dashboard):

   **We apply migrations via Supabase Dashboard, NOT via CLI.**

   Why? CLI migration tracking gets out of sync with dashboard changes, causing conflicts.

   **Step-by-step process:**
   1. Create migration file in `supabase/migrations/<timestamp>_name.sql`
   2. Review the SQL carefully
   3. Go to Supabase Dashboard → SQL Editor
   4. Copy the migration file contents
   5. Paste and execute in SQL Editor
   6. Verify changes were applied successfully
   7. Commit the migration file to git

   **DO NOT use `supabase db push`** - it causes "policy already exists" errors

2. **Before ANY database change:**
   - Review the migration SQL file carefully
   - Test on a separate test database first if possible
   - Ensure RLS policies are correct
   - Double-check foreign key constraints
   - Make sure column additions use `IF NOT EXISTS` or `ADD COLUMN IF NOT EXISTS`

3. **If CLI is needed (rare cases):**
   ```bash
   # First sync remote changes
   supabase db pull

   # Then push (only if necessary)
   supabase db push
   ```

4. **Emergency Backup (if needed):**
   ```bash
   # Backup before major changes
   supabase db dump -f backup_$(date +%Y%m%d_%H%M%S).sql
   ```

### 🎯 Key Principle

**User data is irreplaceable and the most valuable asset of this application.**
- Every user's challenge progress represents days/weeks of their effort
- Losing this data destroys user trust permanently
- Database changes must be additive (ADD columns, not DROP)
- ALWAYS use migrations, NEVER manual schema changes

**If in doubt, ask the user before running ANY database command.**

## Production Domain

**CRITICAL: The production domain is `www.gritful.app`**
- Production URL: `https://www.gritful.app`
- Email templates and links must use `www.gritful.app`
- Supabase secrets: `APP_URL=https://www.gritful.app`
- Supabase Site URL: `https://www.gritful.app`
- NEVER use `gritful.com` or bare `gritful.app` - always use the `www` subdomain
- The bare domain `gritful.app` (without www) does not work for OAuth redirects

## Code Style Guidelines

**NO EMOJIS**: Do not use emojis anywhere in the codebase, file content, UI/UX, or commit messages unless explicitly necessary. Keep all text professional and emoji-free.

## RLS (Row Level Security) - IMPORTANT FOR NEW FEATURES

**Every new feature that queries the database must consider RLS policies.**

### Common RLS Issues

1. **Private data not accessible**: If a query returns empty or errors, check if RLS is blocking access
2. **Unauthenticated users blocked**: RLS policies with `TO authenticated` block anonymous users
3. **Cross-user data access**: Users can only see data they have explicit access to

### Checklist for New Features

Before implementing any feature that reads/writes data:

- [ ] **Identify who needs access**: authenticated users, anonymous users, specific roles (creator, participant)?
- [ ] **Check existing RLS policies**: Does the table have policies that cover this use case?
- [ ] **Test as different user types**: Test as logged-out user, logged-in non-participant, participant, creator
- [ ] **Use service role client when needed**: For features like invite link previews that need to bypass RLS, use `createServiceRoleClient()` (use sparingly!)

### When to Use Service Role Client

Use `createServiceRoleClient()` only when:
- Showing preview data to unauthenticated users (e.g., invite link landing page)
- Admin operations that need to bypass RLS
- Background jobs/cron tasks

**Never expose service role operations to user-controlled input without validation!**

### Schema Cache Issue (Foreign Key Joins)

PostgREST may not detect foreign key relationships. If you get errors like:
```
"Could not find a relationship between 'X' and 'Y' in the schema cache"
```

**Workaround**: Fetch related data in separate queries and join in application code:
```typescript
// Instead of:
.select('*, profile:profiles!user_id_fkey(username)')

// Do this:
const { data } = await supabase.from('table').select('*');
const { data: profiles } = await supabase.from('profiles').select('*').in('id', userIds);
// Join in code using Map
```

## Pinned Issues

### 1. Supabase Schema Cache Issue - Foreign Key Relationship
**Status**: Workaround implemented, needs investigation

**Problem**:
Supabase PostgREST is not detecting the foreign key relationship between `challenge_participants.user_id` and `profiles.id`, even though the foreign key exists in the migration file.

**Current Workaround**:
- Fetching participants and profiles in separate queries
- Joining data in application code using Map
- Location: `/app/challenges/[id]/progress/page.tsx` (lines 67-91)

**Proper Fix Needed**:
1. Verify foreign key exists in production database by running:
```sql
SELECT
    tc.constraint_name,
    tc.table_name,
    kcu.column_name,
    ccu.table_name AS foreign_table_name,
    ccu.column_name AS foreign_column_name
FROM information_schema.table_constraints AS tc
JOIN information_schema.key_column_usage AS kcu
  ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage AS ccu
  ON ccu.constraint_name = tc.constraint_name
WHERE tc.table_name = 'challenge_participants'
  AND tc.constraint_type = 'FOREIGN KEY';
```

2. If foreign key is missing, create it:
```sql
ALTER TABLE challenge_participants
ADD CONSTRAINT challenge_participants_user_id_fkey
FOREIGN KEY (user_id)
REFERENCES profiles(id)
ON DELETE CASCADE;
```

3. Refresh Supabase schema cache (done automatically after schema changes, or restart PostgREST)

**Expected Behavior**:
Should be able to use:
```javascript
.select(`
  id,
  user_id,
  profiles (
    username,
    avatar_url
  )
`)
```

---

### 2. Investigate `participant_details` View Usage
**Status**: Needs investigation

**Background**:
A database view `public.participant_details` exists that joins `challenge_participants` with `profiles`. It was flagged by Supabase security advisor for using SECURITY DEFINER (fixed in migration `20260106000001_fix_security_issues.sql` to use SECURITY INVOKER).

**View Definition**:
```sql
SELECT cp.id, cp.challenge_id, cp.user_id, cp.status, cp.current_streak, cp.longest_streak, p.username, p.avatar_url
FROM challenge_participants cp
LEFT JOIN profiles p ON p.id = cp.user_id;
```

**Investigation Needed**:
1. This view is NOT used anywhere in the codebase (we fetch participants and profiles separately)
2. It may have been created as a workaround for the schema cache issue (Pinned Issue #1)
3. Determine if we should:
   - Start using this view instead of separate queries (cleaner code)
   - Or drop it entirely (less database objects to maintain)

**Decision**: TBD - either adopt the view or remove it

---

## Recent Features Implemented

### Challenge Template System (2025-01-13)
- [DONE] Created 8 pre-configured challenge templates
- [DONE] Template selection screen with category filtering
- [DONE] Template preview modal showing metrics and bonuses
- [DONE] Pre-fill functionality - templates populate entire create form
- [DONE] Users can fully customize templates before creating

### Key Files:
- `/lib/templates/challengeTemplates.ts` - Template definitions
- `/components/challenges/create/TemplateSelectionScreen.tsx` - Template gallery UI
- `/components/challenges/create/TemplateCard.tsx` - Individual template cards
- `/lib/stores/challengeStore.ts` - Template pre-fill logic

### Public Profile System (2025-01-13)
- [DONE] Public profile page at `/profile/[username]`
- [DONE] Shows aggregate stats from all challenges (total points, active challenges, longest streak)
- [DONE] Lists public challenges with full details
- [DONE] Counts private challenges without revealing details (privacy-safe)
- [DONE] Clickable usernames throughout app (leaderboard, participants, challenge creators)
- [DONE] Social links display (Twitter, GitHub, Instagram, website)
- [DONE] Fixed RLS policy to allow viewing participants of public challenges

### Key Files:
- `/app/profile/[username]/page.tsx` - Public profile page
- `/components/progress/ParticipantsLeaderboard.tsx` - Clickable usernames on leaderboard
- `/components/progress/ParticipantProgressCard.tsx` - Clickable usernames in progress cards
- `/supabase/migrations/20250113000001_fix_public_profile_rls.sql` - RLS policy fix

### UX Improvements (2025-01-13)
- [DONE] Fixed form values not displaying when navigating between dates in /entries
- [DONE] Fixed number input UX - empty fields instead of showing "0"
- [DONE] Made challenge titles clickable on /today page with green color and arrow icon
- [DONE] Fixed step indicator dark mode colors (changed blue to green, proper contrast)

### Points-Based Scoring System
- [DONE] Configurable points per metric
- [DONE] Threshold-based scoring (binary, scaled, tiered modes)
- [DONE] Minimum/Maximum threshold selection
- [DONE] Bonus points for streaks and perfect days
- [DONE] Edit challenge settings page
- [DONE] Recalculate historical points
- [DONE] Points display on progress page
- [DONE] Leaderboard with points

### Key Files:
- `/lib/utils/scoring.ts` - Scoring calculation logic
- `/components/challenges/EditChallengeForm.tsx` - Settings editor
- `/app/actions/updateChallenge.ts` - Save settings action
- `/app/actions/recalculatePoints.ts` - Recalculate all points
- `/app/challenges/[id]/progress/page.tsx` - Progress & leaderboard

---

## Pending Work

### App Redesign - Match Landing Page Style
- [ ] **Add Dark/Light Theme Toggle System**
  - Implement theme provider (next-themes or custom context)
  - Add theme toggle component to navigation
  - Persist theme preference in localStorage

- [ ] **Update Color Scheme**
  - Primary background: `#0A0A0B`
  - Secondary background: `#0F1419`
  - Accent color: `#00FF88`
  - Card background: `#151515`
  - Card borders: `#2A2A2A`
  - Primary text: `#FFFFFF`
  - Secondary text: `#9CA3AF`

- [ ] **Apply Dark Gradient Backgrounds**
  - Use `linear-gradient(to bottom, #0A0A0B 0%, #0F1419 100%)` throughout app
  - Add subtle radial green glow (`rgba(0, 255, 136, 0.15)`) where appropriate

- [ ] **Update Button Styles**
  - Primary: Green accent (`#00FF88`) with dark text (`#0A0A0B`)
  - Secondary: White (`#FFFFFF`) with dark text
  - Add hover scale animations (`hover:scale-105`)

- [ ] **Update Icons and Components**
  - Ensure all icons from lucide-react match landing page style
  - Update icon colors to match theme

- [ ] **Update Card Components**
  - Background: `#151515`
  - Borders: `#2A2A2A` (1px)
  - Border radius: rounded-2xl
  - Apply consistent padding

- [ ] **Apply Inter Font Family**
  - Set `fontFamily: 'Inter, system-ui, -apple-system, sans-serif'` globally
  - Update all custom styled components

- [ ] **Update Navigation Bar**
  - Match landing page dark aesthetic
  - Use new color scheme
  - Add theme toggle button

- [ ] **Test Theme Switching**
  - Test all pages with dark theme
  - Test all pages with light theme (if implemented)
  - Ensure no style conflicts or broken components

### Email Notification System (Using Resend + Supabase)
**Strategy**: Conservative defaults - only challenge updates enabled initially
**Status**: MVP Complete - Challenge updates working! ✅

#### Phase 1: Core Infrastructure ✅ COMPLETE
- [x] **Database Schema for Email System**
  - Created `email_queue` table for reliable processing
  - Created `email_unsubscribe_tokens` table
  - Updated `user_preferences` with email settings (all OFF except challenge_updates)
  - Migrations: `/supabase/migrations/20250115000001_email_system_setup.sql`

- [x] **Resend Email Service Setup**
  - Signed up for Resend free tier (3,000 emails/month)
  - Added gritful.com domain to Resend with DNS verification
  - Configured DNS records (SPF, DKIM, DMARC)
  - Stored RESEND_API_KEY in Supabase secrets

- [x] **Email Processor Edge Function**
  - Created `/supabase/functions/email-processor/index.ts`
  - Implemented queue processing with retry logic and exponential backoff
  - Added email templates (HTML + text versions) for all 4 types
  - Deployed with rate limit protection (1-second delays)

#### Phase 2: Challenge Update Emails (MVP) ✅ COMPLETE
- [x] **Challenge Update UI**
  - Added "Email Participants" button to challenge detail page (creator only)
  - Created `/components/challenges/SendUpdateModal.tsx`
  - Shows participant count who will receive email
  - Component: `/components/challenges/EmailParticipantsButton.tsx`

- [x] **Server Action for Updates**
  - Created `/app/actions/sendChallengeUpdate.ts`
  - Verifies sender is challenge creator
  - Queues emails for opted-in participants only
  - Uses secure RPC function to retrieve emails
  - Triggers email processor instantly after queuing

- [x] **Unsubscribe System**
  - Created `/app/unsubscribe/[token]/page.tsx`
  - Generates secure unsubscribe tokens
  - Adds List-Unsubscribe headers to emails
  - Token-based one-click unsubscribe

#### Phase 3: User Preferences ✅ COMPLETE
- [x] **Email Settings in Profile**
  - Updated `/components/profile/ProfileSettingsSection.tsx`
  - Added email notification toggles (all OFF by default except challenge_updates)
  - Shows clear opt-in messaging
  - Conservative defaults implemented

#### Remaining Work & Improvements
- [ ] **Set up automated cron job for email processing**
  - GitHub Actions workflow to trigger processor every 5 minutes
  - Automatic retry for failed/pending emails
  - Monitor queue health

- [ ] **Remove 1-second delay after implementing better solution**
  - Current: Sequential sending with 1s delays (temporary rate limit workaround)
  - Future: Use Resend Batch API or upgrade plan
  - Location: `/supabase/functions/email-processor/index.ts`

- [ ] **Verify and test unsubscribe functionality end-to-end**
  - Test unsubscribe flow from email link
  - Verify preferences update correctly
  - Test re-subscription

- [ ] **Implement selective participant emailing**
  - Add participant checkbox selection to SendUpdateModal
  - Modify sendChallengeUpdate to accept optional recipientUserIds[]
  - Add individual "Email" button per participant
  - Foundation for messaging system Phase 1

#### Known Issues & Tech Debt
- **Rate Limiting**: Using 1-second delays between emails as temporary workaround
- **Email Processor**: Needs automated triggering (currently manual/instant)
- **URL Configuration**: APP_URL set in Supabase secrets, fallback to gritful.com
- **Custom Action URLs**: Partially implemented, needs full integration for different email types

#### Phase 4: Future Email Types (Not Active by Default)
- [ ] **Daily Reminders** (Default: OFF)
  - Implement with pg_cron scheduling
  - Timezone-aware delivery
  - Users must explicitly opt-in

- [ ] **Weekly Summaries** (Default: OFF)
  - Monday morning delivery
  - Include stats, streaks, points
  - Users must explicitly opt-in

- [ ] **Join Request Notifications** (Default: OFF)
  - Database trigger on join_requests table
  - Instant delivery to challenge creator
  - Users must explicitly opt-in

### Unified Messaging System Architecture
**Vision**: Comprehensive messaging platform integrating email, in-app notifications, and direct messaging

#### Phase 1: Selective Participant Emailing ✅ COMPLETE
- [x] **Enhanced SendUpdateModal UI**
  - Participant checkbox list with Select All/Clear toggles
  - Real-time search/filter functionality (appears for 5+ participants)
  - Smart buttons that adapt to filtered results ("Select Filtered" vs "Select All")
  - Selection count display showing X of Y participants selected
  - Improved scrolling with 300px max height
  - Component: `/components/challenges/SendUpdateModal.tsx`

- [x] **Updated Backend**
  - Modified sendChallengeUpdate to accept optional recipientUserIds[] parameter
  - Filters email queue based on selection with validation
  - Maintains backward compatibility (no recipients = all participants)
  - Server-side validation ensures recipients are active participants
  - Location: `/app/actions/sendChallengeUpdate.ts`

- [x] **Individual Email Actions**
  - EmailParticipantButton component for single participant emails
  - Integrated into ParticipantDetailModal (creator-only, not visible to self)
  - Pre-selects specific participant in SendUpdateModal
  - Reuses email infrastructure
  - Locations: `/components/challenges/EmailParticipantButton.tsx`, `/components/progress/ParticipantDetailModal.tsx`

- [x] **Technical Decisions Made**
  - Modal pattern appropriate for infrequent email actions
  - Client-side search/filter for better UX (no server round-trips)
  - Workaround for Supabase schema cache issue (fetch participants and profiles separately, merge in code)
  - Email queue integration with existing system
  - Validation at multiple layers (client, server, RPC function)

#### Phase 2: Challenge Update Center/Message Board (NEXT - 3-4 days)

**Research-Based Design Decisions:**

**Database Schema** (Timestamp-based approach, NOT junction tables):
```sql
-- Main messages table
CREATE TABLE challenge_messages (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  challenge_id UUID REFERENCES challenges(id) ON DELETE CASCADE,
  sender_id UUID REFERENCES profiles(id),
  subject VARCHAR(200),
  message TEXT NOT NULL,
  sent_via_email BOOLEAN DEFAULT false,
  recipient_count INT DEFAULT 0,
  is_pinned BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW(),
  edited_at TIMESTAMP,
  parent_message_id UUID REFERENCES challenge_messages(id), -- for threading (future)

  INDEX idx_challenge_messages_challenge (challenge_id, created_at DESC),
  INDEX idx_challenge_messages_pinned (challenge_id, is_pinned, created_at DESC)
);

-- Lightweight read tracking (timestamp-based, scales better than junction tables)
ALTER TABLE challenge_participants ADD COLUMN
  last_message_read_at TIMESTAMP DEFAULT NOW();

-- Alternative for explicit tracking of important messages (optional)
-- Only use for pinned/important messages, not all messages
CREATE TABLE message_read_receipts (
  message_id UUID REFERENCES challenge_messages(id),
  user_id UUID REFERENCES profiles(id),
  read_at TIMESTAMP DEFAULT NOW(),
  PRIMARY KEY (message_id, user_id)
);
```

**Why Timestamp-Based vs Junction Tables:**
- ✅ Scales: No millions of read-receipt rows
- ✅ Fast queries: `WHERE created_at > last_read_at` is instant
- ✅ Auto-cleanup: Older messages auto-marked as read
- ✅ Flexible: Can still track important messages explicitly
- ❌ Junction table per message: phpBB tried this, abandoned it (doesn't scale)

**UI Pattern** (Feed/Timeline, NOT modal):
- [ ] New "Updates" tab in challenge navigation (part of tab redesign)
- [ ] Reverse-chronological feed with pagination (20 messages per page)
- [ ] Sticky compose box at top (creators/admins only)
- [ ] Message cards show:
  - Sender avatar and name (link to profile)
  - Timestamp (relative: "2 hours ago", hover shows full date)
  - Subject line (if present, bold if unread)
  - Message content (plain text + markdown support)
  - Pin indicator for important messages
  - "Sent via email to X participants" badge if applicable
- [ ] Unread indicator badge on tab
- [ ] "Mark all as read" button
- [ ] Infinite scroll or pagination controls

**Integration with Email System:**
- [ ] When sending email via SendUpdateModal, also create message record
- [ ] Set sent_via_email=true and recipient_count
- [ ] Dual visibility: email recipients get email, all participants see in-app
- [ ] Unread count includes messages created after last_message_read_at

**Performance Optimizations:**
- [ ] Pagination: 20 messages per page
- [ ] Database indexes on challenge_id and created_at
- [ ] Cache unread counts (update on message create/read)
- [ ] Lazy load message content (summary first, expand for full)
- [ ] Virtual scrolling for very long feeds (optional)

**Implementation Steps:**
1. Create migration for challenge_messages table and read tracking
2. Create /app/challenges/[id]/updates page (within tab structure)
3. Build MessageFeed component with pagination
4. Implement PostMessageForm (reuse email modal logic where possible)
5. Add unread count badge to Updates tab
6. Update participant read timestamp on tab view
7. Modify sendChallengeUpdate to create message records

#### Phase 3: Direct User Messaging (Future - 1 week)
- [ ] **Database Schema**
  - `conversations` table
  - `direct_messages` table
  - Conversation participants tracking

- [ ] **Messaging UI**
  - Inbox/outbox interface
  - Real-time message delivery
  - Typing indicators
  - Read receipts
  - Block/mute options

#### Phase 4: Unified Notification Preferences (Future)
- [ ] **Preference Center**
  - Granular controls per message type
  - Delivery channels (in-app, email, push)
  - Frequency options (instant, batch, daily)
  - Quiet hours configuration

#### Key Implementation Notes:
- **Progressive Enhancement**: Each phase adds value without breaking previous features
- **No Throwaway Work**: Email system becomes one delivery channel in larger system
- **Conservative approach**: Start with only challenge updates enabled
- **Free tier friendly**: Stay under 3,000 emails/month initially
- **Spam prevention**: Proper DNS setup, warming strategy, unsubscribe links
- **User trust**: Let users opt-in to additional emails vs forcing them

### Reminder System (Web Push Notifications - Future)
- [ ] **Implement Daily Reminder Notifications**
  - Set up notification service (Web Push API or external service)
  - Create backend endpoints for scheduling reminders
  - Handle notification permissions

- [ ] **Add Reminder Settings Page**
  - Time selection for daily reminders
  - Notification preferences (email, push, SMS)
  - Per-challenge reminder customization
  - Snooze options

### UX Improvements
- [ ] **Participant Section for Each Challenge** - Add dedicated participants tab/section to challenge detail page
  - Show all participants with their stats
  - Make it easy to see who's in the challenge
  - Include participant search/filter
  - Show participant activity/progress
  - Location: `/app/challenges/[id]/page.tsx`

- [ ] **Interactive Daily Metrics Cards** - Make metrics clickable on challenge detail page for quick logging
  - Need to handle challenges with many metrics (pagination or scrolling)
  - Show recent values/activity for each metric
  - Consider modal or inline quick-log functionality
  - Location: `/app/challenges/[id]/page.tsx`

### Task System Enhancements (Future)
- [ ] **Rename internal variables from "metric" to "task"** - After Phase 1 (one-time tasks), refactor codebase to use "task" terminology internally (not just UI strings). This is a larger refactor affecting types, database queries, and variable names.
- [ ] **Phase 2: Weekly/Monthly Tasks** - Add `weekly` and `monthly` frequency options with period-based completion tracking
- [ ] **Change default frequency to weekly** - Once Phase 2 (weekly/monthly) is implemented, change the default frequency from "daily" to "weekly"
- [ ] **Milestones** - Big checkpoint goals at specific challenge dates (e.g., "Complete 100 pushups by Week 4")
- [ ] **Subtasks** - Break complex tasks into smaller steps with sub-checkboxes
- [ ] **Task Categories/Groups** - Organize tasks into sections (Morning, Evening, Fitness, etc.)
- [ ] **Task Dependencies** - "Task B unlocks after Task A is complete"
- [ ] **Participant-specific Tasks** - Assign certain tasks to specific people (for team challenges)

### Challenge Template System
**Status**: Phase 1 (MVP) Complete

#### Phase 1: Basic Templates (DONE)
- [x] **Template Data Structure**
  - Created 8 default templates (Fitness, Healthy Habits, Productivity, Morning Routine, Study, Reading, Mindfulness, No-Zero Days)
  - Location: `/lib/templates/challengeTemplates.ts`

- [x] **Template Selection UI**
  - Template selection screen (Step 0) with "Start from Scratch" vs "Use Template"
  - Template gallery with category filtering (All, Fitness, Productivity, Learning, Wellness)
  - Template cards with preview functionality
  - Location: `/components/challenges/create/TemplateSelectionScreen.tsx`

- [x] **Pre-fill Functionality**
  - Templates pre-populate entire create form (name, metrics, scoring, bonuses)
  - Users can edit everything before creating
  - Seamless transition to 4-step wizard
  - Location: `/lib/stores/challengeStore.ts` (loadFromTemplate action)

#### Phase 2: User-Created Templates (Future)
- [ ] **Allow users to publish their challenges as templates**
  - Add "Publish as Template" option in challenge settings page
  - Template visibility options: Private (only you), Public (anyone), Link only
  - Category selection for templates
  - Template description field

- [ ] **Database Schema for User Templates**
  ```sql
  CREATE TABLE challenge_templates (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    creator_id UUID REFERENCES profiles(id),
    source_challenge_id UUID REFERENCES challenges(id),
    name VARCHAR NOT NULL,
    description TEXT,
    category VARCHAR NOT NULL,
    metrics JSONB NOT NULL,
    settings JSONB NOT NULL,
    visibility VARCHAR DEFAULT 'private',
    times_used INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW()
  );
  ```

- [ ] **Template Management Features**
  - My Templates page - view/edit/delete user's published templates
  - Usage statistics (how many times template was used)
  - Update template from source challenge

#### Phase 3: Community Templates (Future)
- [ ] **Enhanced Discovery**
  - Search templates by name/description
  - Sort by popularity (times used)
  - Filter by creator
  - Featured templates section

- [ ] **Quality & Engagement**
  - Template ratings/reviews
  - Report inappropriate templates
  - Admin approval/moderation system (optional)

### Database Migrations
- [ ] Run `supabase/temp_add_threshold_type.sql` to add threshold_type to existing metrics
- [ ] Investigate and fix foreign key schema cache issue

---

## Architecture Notes

### RLS (Row Level Security) Pattern
Uses `user_challenge_access` table to break circular dependencies:
- Triggers maintain access automatically when users join/create challenges
- All RLS policies check this table instead of nested queries
- Location: `/supabase/migrations/20250127000002_row_level_security.sql`

### Scoring System
Three modes:
1. **Binary**: Full points if threshold met, zero otherwise
2. **Scaled**: Proportional points based on progress toward threshold
3. **Tiered**: Different point values for different achievement levels

Supports both minimum (at least X) and maximum (at most X) thresholds.

---

## Development Notes

### Common Commands
```bash
# Run dev server
npm run dev

# Run migrations locally
supabase db reset

# Generate types
supabase gen types typescript --local > lib/supabase/database.types.ts

# Push to production
git add . && git commit && git push
```

### Environment Variables Needed
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY` (for admin operations)
- `RESEND_API_KEY` (for email service)

---

## Tech Stack
- **Framework**: Next.js 15 (App Router)
- **Database**: Supabase (PostgreSQL)
- **Auth**: Supabase Auth
- **Styling**: Tailwind CSS + shadcn/ui
- **Language**: TypeScript

---

*Last updated: 2025-01-13*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Miandari)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Miandari)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
