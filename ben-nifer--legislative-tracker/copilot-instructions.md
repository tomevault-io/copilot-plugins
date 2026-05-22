## legislative-tracker

> > This file provides context to Claude Code CLI (running in VS Code integrated terminal) about our project. Keep it updated as the project evolves.

# CLAUDE.md - Legislative Tracker

> This file provides context to Claude Code CLI (running in VS Code integrated terminal) about our project. Keep it updated as the project evolves.

## Project Overview

**Legislative Tracker** is a Letterboxd-style civic engagement platform for New York City Council legislation. Users can browse, track, review, and engage with NYC legislation in a social, accessible way.

**GitHub Repository:** `legislative-tracker`
**Development Environment:** Claude Code CLI in VS Code integrated terminal

### Core Value Proposition
"Make NYC legislation accessible and engaging for everyday New Yorkers"

### Target Users
- NYC residents who want to stay informed
- Advocacy groups tracking specific issues
- Journalists covering city politics
- Anyone curious about how NYC laws are made

---

## MVP Features (Version 1)

### Home Page
- **Personal tracked legislation overview** (click to expand to full watchlist)
- **Trending legislation section** (click to expand to full trending page)
- **"New from friends/followed accounts"** feed (activity from followed users and council members)

### Search
- **Legislation search:**
  - Search by keyword
  - Filter by: Status, Topic, Sponsor, Date range, Trending
  - Filter by Resolution/Introduction type
  - ❌ Location filter (post-MVP)
- **Profile search:**
  - Search users by name

### Watchlist Page
- Accessible via tab or from home page
- **Widgetized hearing schedule** (expandable) for followed legislation
- **Notification toggle** per item (in-app only for MVP)

### Trending Page
- Full trending legislation list
- Filters: Status, Topic, Sponsor, Date range, Resolution/Introduction

### Legislation Display (Three Levels)

**Level 1 - Card/Overview (in lists):**
- ID#, status, summary
- Engagement counts: # support, # opposed, # neutral, # saves, # comments
- Save/bookmark button

**Level 2 - Expanded Dropdown:**
- Everything from Level 1, plus:
- Committee name, date introduced, last action (with date)
- "View Details" button

**Level 3 - Full Detail Page:**
- Complete summary (Claude API generated or official)
- Sponsor information with link to profile
- Latest updates / next scheduled hearing
- **Engagement section:**
  - Support / Oppose / Neutral counts (separate from "Watching")
  - Comments with filtering:
    - Latest
    - Most engaged (overall) - based on upvote + downvote count
    - Most engaged by stance (support/oppose/neutral commenters)
    - People you follow
  - Upvote/downvote on comments (Reddit-style, affects ranking)
- Full bill text (embedded or linked, check API availability)

### User Profile

**Public Profile Page:**
- Profile picture
- Bio
- External links (social media style - platform + URL)
- Interest tags (custom + predefined, separate from legislation topics)
- Public bookmarks
- Stance summary (Supporting/Opposing/Neutral counts)
- Following count (users + council members)
- Followers count
- Comment history (filterable by most engaged)

**Profile Settings:**
- Edit profile picture, bio, links
- Manage interest tags (select predefined or create custom)
- ❌ Private profile toggle (premium, post-MVP)

### Notifications (In-App Only for MVP)
- Updates to bills you follow
- Hearing alerts
- Bill amendments
- Engagement on your comments (replies, upvotes)

### Features NOT in MVP
- ❌ Location-based filtering
- ❌ Private profiles (premium feature)
- ❌ Interest groups
- ❌ User-created lists (Letterboxd-style)
- ❌ Email notifications
- ❌ Push notifications

---

## Tech Stack

| Layer | Technology | Notes |
|-------|------------|-------|
| **Framework** | Next.js 14 (App Router) | Using `src/` directory, Server Components by default |
| **Language** | TypeScript | Strict mode enabled |
| **Styling** | Tailwind CSS + shadcn/ui | Dark theme, custom color palette |
| **Database** | Supabase (PostgreSQL) | Hosted, with Row Level Security |
| **Auth** | Supabase Auth | Email magic links + Google OAuth |
| **AI** | Anthropic Claude API | For legislation summaries |
| **Moderation** | OpenAI Moderation API | Free, for content safety |
| **Hosting** | Vercel | Auto-deploy from main branch |

---

## Project Structure

```
legislative-tracker/
├── CLAUDE.md                     # This file - context for Claude Code
├── src/
│   ├── app/                      # Next.js App Router
│   │   ├── (auth)/               # Auth-required routes
│   │   │   ├── watchlist/        # User's watchlist page
│   │   │   ├── profile/          # User's own profile settings
│   │   │   ├── notifications/    # In-app notifications
│   │   │   └── settings/         # App settings
│   │   ├── (public)/             # Public routes
│   │   │   ├── legislation/
│   │   │   │   ├── page.tsx      # Browse/search all legislation
│   │   │   │   └── [slug]/       # Detail view (Level 3)
│   │   │   ├── trending/         # Trending page
│   │   │   ├── council-members/
│   │   │   │   ├── page.tsx      # Browse council members
│   │   │   │   └── [slug]/       # Council member profile
│   │   │   ├── users/
│   │   │   │   └── [username]/   # Public user profile
│   │   │   └── search/           # Search results page
│   │   ├── api/                  # API routes
│   │   │   ├── cron/             # Vercel cron jobs
│   │   │   │   ├── sync-legislation/
│   │   │   │   └── refresh-stats/
│   │   │   └── ...
│   │   ├── actions/              # Server Actions
│   │   │   ├── engagement.ts     # Stances, bookmarks
│   │   │   ├── comments.ts       # Comments, votes
│   │   │   ├── social.ts         # Following
│   │   │   └── profile.ts        # Profile updates
│   │   ├── auth/callback/        # Auth callback handler
│   │   ├── layout.tsx
│   │   ├── page.tsx              # Homepage
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/                   # shadcn/ui components
│   │   ├── layout/               # Header, Footer, Nav, SearchBar
│   │   ├── home/                 # Homepage sections
│   │   │   ├── TrackedOverview.tsx
│   │   │   ├── TrendingSection.tsx
│   │   │   └── FriendActivity.tsx
│   │   ├── legislation/          # Legislation components
│   │   │   ├── LegislationCard.tsx      # Level 1: Overview card
│   │   │   ├── LegislationExpanded.tsx  # Level 2: Dropdown details
│   │   │   ├── LegislationDetail.tsx    # Level 3: Full page
│   │   │   ├── LegislationFilters.tsx   # Filter controls
│   │   │   ├── StanceButtons.tsx        # Support/Oppose/Neutral/Watching
│   │   │   ├── BookmarkButton.tsx
│   │   │   └── HearingSchedule.tsx      # Widgetized schedule
│   │   ├── comments/             # Comment system
│   │   │   ├── CommentThread.tsx
│   │   │   ├── CommentItem.tsx
│   │   │   ├── CommentInput.tsx
│   │   │   ├── CommentFilters.tsx       # Latest, Most Engaged, etc.
│   │   │   └── VoteButtons.tsx          # Upvote/downvote (Reddit-style)
│   │   ├── search/               # Search components
│   │   │   ├── SearchBar.tsx
│   │   │   ├── SearchFilters.tsx
│   │   │   └── SearchResults.tsx
│   │   ├── profile/              # Profile components
│   │   │   ├── ProfileHeader.tsx
│   │   │   ├── ProfileStats.tsx         # Following, followers, stances
│   │   │   ├── ProfileLinks.tsx         # Social links
│   │   │   ├── InterestTags.tsx         # Custom + predefined tags
│   │   │   ├── CommentHistory.tsx
│   │   │   └── FollowButton.tsx
│   │   ├── council/              # Council member components
│   │   │   ├── CouncilMemberCard.tsx
│   │   │   └── CouncilMemberProfile.tsx
│   │   └── notifications/
│   │       ├── NotificationBell.tsx
│   │       ├── NotificationList.tsx
│   │       └── NotificationItem.tsx
│   ├── lib/
│   │   ├── supabase/
│   │   │   ├── client.ts         # Browser client
│   │   │   ├── server.ts         # Server client
│   │   │   └── middleware.ts     # Auth middleware helpers
│   │   ├── legistar/             # NYC Council API integration
│   │   │   ├── client.ts
│   │   │   ├── sync.ts
│   │   │   └── types.ts
│   │   ├── ai/
│   │   │   └── summarize.ts      # Claude API integration
│   │   ├── moderation/
│   │   │   └── check.ts          # OpenAI moderation
│   │   └── utils/
│   │       ├── dates.ts
│   │       └── formatting.ts
│   ├── hooks/
│   │   ├── useUser.ts
│   │   ├── useNotifications.ts
│   │   └── useSearch.ts
│   └── types/
│       └── database.ts           # Supabase generated types
├── public/
├── .env.local
├── middleware.ts                 # Next.js middleware
├── vercel.json
└── package.json
```

---

## Database Schema

### Core Tables

**legislatures** - Support for multiple legislative bodies (NYC now, Congress later)
- `id`, `slug`, `name`, `jurisdiction`, `level`, `api_base_url`

**legislation** - Bills, resolutions, etc.
- `id`, `legislature_id`, `file_number`, `slug`, `title`, `status`
- `type` ('resolution' | 'introduction')
- `intro_date`, `last_action_date`
- `ai_summary`, `official_summary`
- `committee_id`, `legistar_url`, `full_text_url`

**legislators** - Council members
- `id`, `legislature_id`, `full_name`, `slug`, `district`, `borough`, `party`
- `email`, `photo_url`, `is_active`

**sponsorships** - Many-to-many: legislation ↔ legislators
- `legislation_id`, `legislator_id`, `is_primary`

**legislation_history** - Timeline of actions
- `id`, `legislation_id`, `action_date`, `action_text`, `passed_flag`

**events** - Hearings and meetings
- `id`, `legislation_id`, `event_date`, `event_type`, `location`, `video_url`

**topics** - Categories for legislation (Housing, Transportation, etc.)
- `id`, `legislature_id`, `name`, `slug`

**legislation_topics** - Many-to-many: legislation ↔ topics
- `legislation_id`, `topic_id`

### User Tables

**user_profiles** - Extends Supabase auth.users
- `id` (matches auth.users.id)
- `username` (unique), `display_name`, `avatar_url`, `bio`
- `links` (JSONB array: `[{platform: string, url: string}]`)

**interest_tags** - All interest tags (predefined + custom)
- `id`, `name`, `slug`
- `is_predefined` (boolean)
- `created_by_user_id` (null if predefined)

**user_interest_tags** - User's selected interests
- `user_id`, `tag_id`

**user_stances** - Support/Oppose/Neutral/Watching
- `user_id`, `legislation_id`
- `stance` ('support' | 'oppose' | 'neutral' | 'watching')
- `created_at`, `updated_at`

**bookmarks** - Saved legislation (visible on public profile)
- `user_id`, `legislation_id`, `created_at`

### Comment System

**comments** - Threaded discussions on legislation
- `id`, `user_id`, `legislation_id`
- `parent_comment_id` (for threading, null if top-level)
- `body` (text content)
- `stance_context` ('support' | 'oppose' | 'neutral' | null) - user's stance when commenting
- `is_hidden`, `is_flagged` (for moderation)
- `created_at`, `updated_at`

**comment_votes** - Upvotes/downvotes (Reddit-style)
- `user_id`, `comment_id`
- `vote` (1 for upvote, -1 for downvote)
- Unique constraint on (user_id, comment_id)

### Social Tables

**user_follows** - User-to-user following
- `follower_id`, `following_id`
- `created_at`
- Unique constraint on (follower_id, following_id)

**legislator_follows** - User following council members
- `user_id`, `legislator_id`
- `created_at`

**legislation_follows** - User following specific legislation (watchlist)
- `user_id`, `legislation_id`
- `notify_updates` (boolean)
- `notify_hearings` (boolean)
- `notify_amendments` (boolean)
- `created_at`

### Notification Tables

**notifications** - In-app notifications
- `id`, `user_id`
- `type` (enum - see below)
- `title`, `body`, `url`
- `read` (boolean, default false)
- `created_at`
- Polymorphic references:
  - `legislation_id` (optional)
  - `comment_id` (optional)
  - `actor_user_id` (optional - who triggered the notification)

**Notification types:**
- `legislation_update` - Bill you follow was updated
- `hearing_alert` - Upcoming hearing for followed bill
- `bill_amendment` - Amendment to followed bill
- `comment_reply` - Someone replied to your comment
- `comment_upvote` - Your comment was upvoted
- `new_follower` - Someone followed you

### Analytics & Trending

**engagement_events** - Raw engagement data
- `id`, `legislation_id`, `user_id`
- `event_type` ('view' | 'stance' | 'comment' | 'bookmark')
- `created_at`

**legislation_stats** - Materialized view for performance
- `legislation_id`
- `support_count`, `oppose_count`, `neutral_count`, `watching_count`
- `comment_count`, `bookmark_count`
- `trending_score`
- `engagement_24h`, `engagement_7d`

---

## Environment Variables

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# NYC Council API
LEGISTAR_API_TOKEN=

# AI
ANTHROPIC_API_KEY=
OPENAI_API_KEY=

# Cron
CRON_SECRET=
```

---

## Key Patterns

### Stance System (4 states)

```typescript
type Stance = 'support' | 'oppose' | 'neutral' | 'watching';

// Neutral = "I've reviewed this and have no strong opinion"
// Watching = "I'm tracking this but haven't taken a stance yet"
```

### Data Fetching (Server Components)

```typescript
import { createServerClient } from '@/lib/supabase/server';

export default async function Page() {
  const supabase = createServerClient();
  const { data } = await supabase
    .from('legislation')
    .select(`
      *,
      sponsors:sponsorships(legislator:legislators(*)),
      stats:legislation_stats(*)
    `)
    .order('intro_date', { ascending: false });
  
  return <LegislationList items={data} />;
}
```

### Server Actions

```typescript
// app/actions/engagement.ts
'use server'

import { createServerClient } from '@/lib/supabase/server';
import { revalidatePath } from 'next/cache';

export async function setStance(
  legislationId: string, 
  stance: 'support' | 'oppose' | 'neutral' | 'watching'
) {
  const supabase = createServerClient();
  const { data: { user } } = await supabase.auth.getUser();
  
  if (!user) throw new Error('Not authenticated');
  
  await supabase.from('user_stances').upsert({
    user_id: user.id,
    legislation_id: legislationId,
    stance,
  }, { onConflict: 'user_id,legislation_id' });
  
  revalidatePath('/legislation');
}
```

### Comment Voting (Reddit-style)

```typescript
// app/actions/comments.ts
'use server'

export async function voteComment(commentId: string, vote: 1 | -1) {
  const supabase = createServerClient();
  const { data: { user } } = await supabase.auth.getUser();
  
  if (!user) throw new Error('Not authenticated');
  
  // Upsert allows changing vote direction
  await supabase.from('comment_votes').upsert({
    user_id: user.id,
    comment_id: commentId,
    vote,
  }, { onConflict: 'user_id,comment_id' });
  
  revalidatePath('/legislation');
}
```

### Comment Filtering (Most Engaged)

```sql
-- Most engaged = highest absolute vote count (upvotes + downvotes)
SELECT 
  c.*,
  COALESCE(SUM(cv.vote), 0) as vote_score,
  COUNT(cv.id) as engagement_count  -- Total votes for "most engaged"
FROM comments c
LEFT JOIN comment_votes cv ON c.id = cv.comment_id
WHERE c.legislation_id = $1
GROUP BY c.id
ORDER BY engagement_count DESC;  -- Most engaged first
```

### Following System

```typescript
// app/actions/social.ts
'use server'

export async function followUser(targetUserId: string) {
  const supabase = createServerClient();
  const { data: { user } } = await supabase.auth.getUser();
  
  await supabase.from('user_follows').insert({
    follower_id: user.id,
    following_id: targetUserId,
  });
  
  // Create notification for target user
  await supabase.from('notifications').insert({
    user_id: targetUserId,
    type: 'new_follower',
    actor_user_id: user.id,
    title: 'New follower',
    body: 'Someone started following you',
  });
}

export async function followLegislator(legislatorId: string) {
  // Similar pattern for council members
}

export async function followLegislation(
  legislationId: string, 
  notifySettings: { updates: boolean; hearings: boolean; amendments: boolean }
) {
  // Add to watchlist with notification preferences
}
```

### Interest Tags (Custom + Predefined)

```typescript
// Get all available tags (predefined + user's custom)
const { data: tags } = await supabase
  .from('interest_tags')
  .select('*')
  .or(`is_predefined.eq.true,created_by_user_id.eq.${userId}`);

// Add custom tag
export async function createCustomTag(name: string) {
  const supabase = createServerClient();
  const { data: { user } } = await supabase.auth.getUser();
  
  const { data: tag } = await supabase
    .from('interest_tags')
    .insert({
      name,
      slug: slugify(name),
      is_predefined: false,
      created_by_user_id: user.id,
    })
    .select()
    .single();
  
  // Auto-add to user's interests
  await supabase.from('user_interest_tags').insert({
    user_id: user.id,
    tag_id: tag.id,
  });
}
```

---

## External APIs

### NYC Council Legistar API

- Base URL: `https://webapi.legistar.com/v1/nyc`
- Auth: Token as query param `?token=YOUR_TOKEN`
- Rate limit: 1000 results per query, paginate with `$top` and `$skip`
- Key endpoints:
  - `/matters` - All legislation
  - `/matters/{id}/sponsors` - Sponsors
  - `/matters/{id}/histories` - Action history
  - `/matters/{id}/texts` - Bill text (check availability)
  - `/events` - Hearings and meetings
  - `/persons` - Council members

### Anthropic Claude API

- Model: `claude-sonnet-4-20250514`
- Used for: Plain-language summaries
- Cost: ~$0.003 per summary

### OpenAI Moderation API

- Endpoint: `POST https://api.openai.com/v1/moderations`
- Cost: FREE
- Used for: Comment moderation

---

## Design System

### Colors (Dark Theme)

```css
--background: slate-950
--card: slate-800/80 with backdrop-blur
--primary: indigo-500
--secondary: purple-500
--accent: pink-500
```

### Stance Colors

| Stance | Color | Use |
|--------|-------|-----|
| Support | emerald-500 | Buttons, badges, counts |
| Oppose | red-500 | Buttons, badges, counts |
| Neutral | amber-500 | Buttons, badges, counts |
| Watching | blue-500 | Buttons, badges |

### Status Colors

| Status | Background | Text |
|--------|------------|------|
| Committee | amber-500/20 | amber-300 |
| Hearing Scheduled | blue-500/20 | blue-300 |
| Enacted | emerald-500/20 | emerald-300 |
| Vetoed | red-500/20 | red-300 |

---

## Working with Claude Code CLI

### Setup

1. Open VS Code with `legislative-tracker` folder
2. Open integrated terminal (Ctrl + `)
3. Run `claude` to start Claude Code CLI
4. This CLAUDE.md file provides context automatically

### Effective Prompts

**Reference specific files:**
```
Look at src/components/legislation/LegislationCard.tsx and help me 
add the Level 2 expanded dropdown when the card is clicked.
```

**Use the schema:**
```
Based on the database schema in CLAUDE.md, create a Supabase query 
to fetch comments sorted by "most engaged" (highest vote count).
```

**Build on existing patterns:**
```
Look at how StanceButtons.tsx works and create a similar VoteButtons 
component for upvoting/downvoting comments.
```

**Debug issues:**
```
I'm getting this error when running npm run dev: [paste error]
The relevant file is src/app/actions/comments.ts
```

---

## Testing Checklist (MVP)

### Core Flows
- [ ] Browse legislation without login
- [ ] Search legislation by keyword
- [ ] Filter by status, topic, sponsor, date
- [ ] View legislation detail (all 3 levels)
- [ ] Sign up / Sign in
- [ ] Set stance (support/oppose/neutral/watching)
- [ ] Bookmark legislation
- [ ] Add to watchlist with notification settings

### Social Features
- [ ] View user profile
- [ ] Follow user
- [ ] Follow council member
- [ ] See "new from followed" on homepage

### Comments
- [ ] Post comment
- [ ] Reply to comment
- [ ] Upvote/downvote comment
- [ ] Filter comments (latest, most engaged, by stance)

### Profile
- [ ] Edit profile (bio, links, avatar)
- [ ] Add/remove interest tags
- [ ] View own comment history

### Notifications
- [ ] See notification bell with count
- [ ] View notification list
- [ ] Mark as read
- [ ] Receive notifications for followed bills

---

*Last updated: February 2025*

---
> Source: [Ben-nifer/legislative-tracker](https://github.com/Ben-nifer/legislative-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
