## socioverse-skill

> Full Instagram content engine powered by the Socioverse API. Runs competitor intelligence, AI ideation, content production (carousels + captions), scheduling, DM automation, lead management, and weekly reporting - all end-to-end. Use when the user wants to run the content pipeline, generate ideas, schedule posts, manage automations, handle leads, send DMs, analyze competitors, manage the content bank, use IdeaPad, generate reports, create carousels, write captions, or do anything with Socioverse programmatically. Trigger on phrases like "socioverse", "schedule post", "run pipeline", "generate ideas", "push to ideapad", "content bank", "automate DMs", "lead magnet", "competitor analysis", "weekly report", "instagram insights", "create carousel", "write caption".


# Socioverse - Industry-Ready Content Engine

Full-lifecycle Instagram content automation: intelligence → ideation → production → scheduling → engagement → reporting.

**Base URL:** `https://api.socioverse.io`
**Auth:** `Authorization: Bearer $SOCIOVERSE_API_KEY`
**Rate limits:** 60 req/min, 1000 req/day
**Key format:** `sk_live_*` (get from dashboard → Settings → API Keys)

---

## Quick Start

```bash
# Verify setup
/socioverse status

# Full weekly content pipeline
/socioverse pipeline

# Just generate ideas
/socioverse ideate --topic "founder burnout"

# Check what user approved in IdeaPad
/socioverse review

# Produce approved ideas into content
/socioverse produce

# Schedule everything
/socioverse schedule
```

---

## Modes

| Mode | Command | Description |
|------|---------|-------------|
| Status | `/socioverse status` | Health check, usage stats, connected accounts |
| Intel | `/socioverse intel` | Competitor scrape + analysis + intelligence brief |
| Ideate | `/socioverse ideate` | Generate ideas → push to IdeaPad |
| Review | `/socioverse review` | Pull IdeaPad, show approved/rejected/pending, iterate |
| Produce | `/socioverse produce` | Accepted ideas → scripts/carousels → Content Bank |
| Schedule | `/socioverse schedule` | Content Bank → scheduled Instagram posts |
| Pipeline | `/socioverse pipeline` | Full orchestration (intel→ideate→pause→produce→schedule) |
| Automate | `/socioverse automate` | DM automations, sequences, lead magnets |
| Leads | `/socioverse leads` | Lead CRUD, messaging, profile lookup |
| Report | `/socioverse report` | Weekly reports, insights, analytics dashboard |

---

## Environment Setup

The skill reads `SOCIOVERSE_API_KEY` from environment. For public users:

```bash
# In your project .env or shell
export SOCIOVERSE_API_KEY=sk_live_your_key_here
```

All API calls use:
```bash
curl -s -X {METHOD} "https://api.socioverse.io{ENDPOINT}" \
  -H "Authorization: Bearer $SOCIOVERSE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{BODY}'
```

---

## Mode 1: Status

**Command:** `/socioverse status`

Performs a full account health check.

### Steps:
1. `GET /v1/health` - verify key validity
2. `GET /v1/usage` - 30-day usage stats
3. `GET /v1/keys/verify` - active scopes
4. `GET /v1/instagram/accounts` - connected accounts
5. `GET /v1/competitors` - tracked competitors count
6. `GET /v1/automations` - active automations count
7. `GET /v1/leads?limit=1` - total leads (from pagination meta)

### Output:
Present a dashboard table:
```
┌─────────────────────────────────────────┐
│ SOCIOVERSE STATUS                       │
├─────────────────────────────────────────┤
│ API Key:      sk_live_ab12...  ✓ Active │
│ Scopes:       21/21 enabled            │
│ Plan:         Growth                    │
│ Rate:         58/60 remaining          │
│ Daily:        847/1000 remaining       │
├─────────────────────────────────────────┤
│ IG Accounts:  2 connected              │
│ Competitors:  5 tracked                │
│ Automations:  3 active                 │
│ Leads:        127 total                │
│ Posts queued:  12 scheduled            │
└─────────────────────────────────────────┘
```

---

## Mode 2: Intel

**Command:** `/socioverse intel [--analyze-count N] [--competitors ID1,ID2]`

Builds an intelligence brief from competitor data and knowledge base.

### Parameters:
| Flag | Default | Description |
|------|---------|-------------|
| `--analyze-count` | 6 | Number of top posts to deep-analyze |
| `--competitors` | all | Specific competitor IDs (comma-sep) |
| `--skip-scrape` | false | Use existing scraped posts only |

### Steps:

#### Step 1: Pull Knowledge Context
```
GET /v1/knowledge
```
Read all knowledge documents to understand the user's brand voice, audience, and positioning. This context informs all downstream generation.

#### Step 2: Scrape Competitors (async)
```
POST /v1/competitors/scrape
{ "batch_size": 12, "post_types": ["reel", "carousel", "image"] }
```
If `--competitors` specified, add `"competitor_ids": [...]`.

Response includes `pipeline_run_id`. The scrape runs asynchronously - the response returns BEFORE the posts are queryable. Do not try to read scraped-posts immediately.

#### Step 3: Wait for the scrape run to finish, then read posts

Poll the pipeline run until terminal:
```
GET /v1/content-studio/pipeline-runs/{pipeline_run_id}
```
Response includes:
- `status`: `running | completed | failed | cancelled`
- `is_terminal`: `true` when status is completed/failed/cancelled
- `items_processed`, `items_failed`: scrape progress
- `posts_attached`: count of scraped_posts created by this run

Poll every 3-5s until `is_terminal: true`. Typical scrape runs complete in 30-90s depending on batch size.

Then fetch the actual posts:
```
GET /v1/content-studio/scraped-posts?pipeline_run_id={pipeline_run_id}&limit=50
```
Or all unprocessed posts across runs:
```
GET /v1/content-studio/scraped-posts?status=unprocessed&limit=50
```

Sort by engagement (response is already ordered by `created_at desc`; sort client-side by `views_count + likes_count*3 + comments_count*5` for engagement-weighted ranking).

**scraped_posts response schema:**
```json
{
  "id": "uuid",
  "competitor_id": "uuid",
  "ig_username": "handle",
  "ig_post_id": "DYrrccWCqNR",
  "post_url": "https://www.instagram.com/reel/DYrrccWCqNR/",
  "shortcode": "DYrrccWCqNR",
  "post_type": "reel | image | carousel",
  "caption": "...",
  "views_count": 50000,
  "likes_count": 3200,
  "comments_count": 124,
  "thumbnail_url": "https://...",
  "permanent_thumbnail_url": "https://cdn.socioverse.io/...",
  "video_url": "https://...",
  "image_urls": ["...", "..."] ,
  "carousel_count": 5,
  "is_ad": false,
  "is_paid_partnership": false,
  "is_affiliate": false,
  "outlier_score": 2.4,
  "status": "unprocessed | analyzing | analyzed | failed",
  "posted_at": "2026-03-07T20:00:00Z",
  "created_at": "2026-03-07T22:00:00Z",
  "transcript": "Verbatim spoken + on-screen text",
  "transcript_status": "pending | available | unavailable | failed",
  "transcribed_at": "..."
}
```
**Field-naming gotcha:** counts are `views_count`, `likes_count`, `comments_count` (with the `s`) - NOT `view_count` / `like_count` / `comment_count`. Instagram does NOT expose share or save counts to third-party scrapers, so there is no `shares_count` or `saves_count`. The `shortcode` field is derived server-side from `post_url`.

**Ranking:** prefer `outlier_score` (the over-performance signal vs. the creator's own baseline, observed range ~0.10-5.42) over raw `views_count`. View counts can lag or read 0 on freshly scraped reels depending on the upstream scraper payload; `outlier_score`, `likes_count`, and `comments_count` are reliable. Sort top posts by `outlier_score` desc.

The transcript appears once the reel has been analyzed via `POST /v1/videos/analyze`. Use the transcript verbatim when building intelligence briefs - it's the actual hook delivery and word-for-word framing the competitor used, which is more replicable than a paraphrased summary. For image/carousel posts, transcript is always null (status stays `pending`).

#### Step 4: Analyze Top Posts
For each of the top `--analyze-count` posts:
```
POST /v1/videos/analyze
{ "scraped_post_id": "{id}" }
```
This produces **structural "Emotional DNA"** of the post (not a transcript). The verbatim transcript - the literal words spoken / shown - lives separately on `scraped_posts.transcript` and is written as a side-effect of the same call.

So after analyze you have two complementary artifacts per reel:
- `scraped_posts.transcript` - what was said, word for word
- `analyzed_reels.analysis` - why it works (hook formula, psychological mechanism, structural blueprint, etc.)

**How transcription is triggered (important):** transcription is NOT automatic on scrape and there is NO separate `/transcribe` endpoint. It happens *only* as a side-effect of `POST /v1/videos/analyze` on a reel. Freshly scraped reels stay `transcript_status:"pending"` with empty `transcript` until you analyze them. Flow: `scrape` → poll `pipeline-runs` → `videos/analyze` each reel → re-read `scraped-posts`, now `transcript_status:"available"`. If analyze fails (e.g. transient Gemini overload), the post is left analyzable again - just re-call `videos/analyze`; the gateway now surfaces the real underlying error (e.g. `Gemini API error: 503`) instead of a generic message.

#### Step 5: Generate Intelligence Brief
```
GET /v1/content-studio/analyzed-posts?limit=20
```

**analyzed-posts response schema:**
```json
{
  "id": "uuid",
  "scraped_post_id": "uuid",
  "raw_idea": "Abstract reusable template (e.g. 'Public failure reframed as proof of expertise')",
  "analysis": {
    "emotional_core": "Relatable struggle - validation",
    "psychological_mechanism": "Loss aversion via missed-opportunity framing",
    "metaphor_meaning": "none",
    "structural_blueprint": "Hook → contradiction → reveal → CTA",
    "hook_formula": "Bold counterclaim + proof tease",
    "transformation_arc": "Viewer starts believing X, ends believing Y",
    "emotional_journey": "inadequacy → recognition → hope → motivation",
    "replication_guide": "Apply hook structure to your niche",
    "non_negotiable_elements": "Vulnerability + concrete proof",
    "visual_elements": "Direct-to-camera, natural light",
    "audio_elements": "Conversational pacing, no music",
    "tone": "..."
  },
  "metadata": { "topic": "Marketing", "tags": ["hooks"], "audience": "Solo founders", "multimodal_analysis": true, "post_type": "reel" },
  "status": "unprocessed | refining | refined | failed",
  "source_type": "reel | image | carousel",
  "scheduled_post_id": null,
  "created_at": "...",
  "processed_at": null
}
```
**Note:** `analysis` is returned as a JSON object directly (not a stringified blob). For image/carousel posts, sub-keys shift slightly: `visual_composition`, `color_psychology`, `slide_narrative_arc`, `swipe_triggers`, `information_architecture` may appear instead of `audio_elements`. Status meanings: `unprocessed` = analysis ready but no idea generated yet; `refined` = idea was generated from this analysis.

Synthesize into an intelligence brief:

```markdown
# Intelligence Brief - {date}

## Top Performing Hooks
1. [hook] - why it works
2. [hook] - why it works
3. [hook] - why it works

## Format Patterns
- Reels: [what's working]
- Carousels: [what's working]
- Static: [what's working]

## Emotional Triggers
- [trigger 1]: engagement level
- [trigger 2]: engagement level

## Content Gaps (opportunities)
- [gap 1]
- [gap 2]

## Recommended Angles for This Week
- [angle 1]
- [angle 2]
- [angle 3]
```

#### Step 6: Store Brief
Save intelligence brief to IdeaPad for reference:
```
POST /v1/ideapad/notes/create
{ "title": "Intel Brief - {date}", "body": "{brief}", "tags": ["intel", "weekly"], "color": "blue" }
```

### Output:
Display the intelligence brief and confirm it's saved to IdeaPad.

---

## Mode 3: Ideate

**Command:** `/socioverse ideate [--topic TOPIC] [--count N] [--from-intel] [--platform PLATFORM] [--goal GOAL]`

Generates content ideas and pushes them to IdeaPad for user review.

### Parameters:
| Flag | Default | Description |
|------|---------|-------------|
| `--topic` | from intel/knowledge | Specific topic to ideate around |
| `--count` | 10 | Number of ideas to generate |
| `--from-intel` | true | Use latest intelligence brief as context |
| `--platform` | instagram | Target platform |
| `--goal` | general | awareness, engagement, conversion, general |
| `--use-api-gen` | false | Use Socioverse's built-in idea generator |

### Steps:

#### Step 1: Gather Context

**From Knowledge Base:**
```
GET /v1/knowledge
```
Extract brand pillars, voice guidelines, audience profile, and positioning.

**From Intelligence (if --from-intel):**
```
GET /v1/ideapad/notes?tag=intel&limit=1
```
Pull latest intelligence brief for trend context.

**From Instagram Insights:**
```
GET /v1/instagram/insights
```
Pull recent performance data - what formats/times performed best.

**From Content Studio (avoid repeats):**
```
GET /v1/content-studio/ideas?status=approved&limit=20
```
Check recent approved ideas to avoid duplication.

#### Step 2: Generate Ideas

**Option A - AI-native generation (default):**
Use the gathered context to generate ideas following this framework:

| # | Format | Hook (first line) | Angle/Message | Pillar | Expected Impact |
|---|--------|-------------------|---------------|--------|-----------------|
| 1 | Reel | [hook] | [angle] | [pillar] | High/Med/Low |

**Content Mix:**
- 4 educational (teach something actionable)
- 3 personal/story (founder journey, behind-the-scenes)
- 2 engagement-bait (polls, questions, hot takes)
- 1 promotional (product, offer, CTA)

**Hook Rules:**
- Pattern interrupt OR bold claim OR curiosity gap
- Never generic ("5 tips for..." is banned)
- First line must earn the second line
- Reference specific numbers, names, or situations

**Option B - Socioverse API generation (--use-api-gen):**
```
POST /v1/ideas/generate
{ "user_topic": "{topic}", "batch_size": {count} }
```
Or from analyzed posts:
```
POST /v1/ideas/generate
{ "analyzed_reel_ids": [...], "batch_size": {count} }
```

#### Step 3: Push to IdeaPad

For EACH generated idea:
```
POST /v1/ideapad/notes/create
{
  "title": "{hook}",
  "body": "**Format:** {format}\n**Angle:** {angle}\n**Pillar:** {pillar}\n**Impact:** {impact}\n\n---\n\n{expanded concept if available}",
  "tags": ["{format}", "{pillar}", "batch-{date}"],
  "color": "{color based on format: reel=purple, carousel=green, static=yellow}",
  "source": "skill"
}
```

#### Step 4: AI-Expand Top Ideas (optional)

For the top 3 flagged ideas, expand them using Socioverse AI:
```
POST /v1/ideapad/expand
{ "note_id": "{id}" }
```
This adds detailed concept breakdown, suggested visuals, and script outline.

### Output:
Display the idea table with all 10 ideas. Confirm pushed to IdeaPad. Tell user:
> "{count} ideas pushed to your IdeaPad. Open Socioverse → IdeaPad to approve or reject. Then run `/socioverse review` to pull approved ideas and produce content."

---

## Mode 4: Review

**Command:** `/socioverse review [--status STATUS] [--iterate]`

Pulls the current state of IdeaPad and shows what's been approved/rejected.

### Parameters:
| Flag | Default | Description |
|------|---------|-------------|
| `--status` | all | Filter: approved, rejected, draft, expanded |
| `--iterate` | false | Iterate on approved ideas (expand, refine) |
| `--batch` | latest | Which batch to review (date or "all") |

### Steps:

#### Step 1: Pull IdeaPad State
```
GET /v1/ideapad/notes?limit=100
```
Group by status and batch tag.

#### Step 2: Display Review Dashboard

```
┌─────────────────────────────────────────────────┐
│ IDEAPAD REVIEW - Batch 2026-05-17               │
├─────────────────────────────────────────────────┤
│ ✅ Approved (4)                                  │
│   1. "I automated my entire..." [Reel]          │
│   2. "Stop posting at 9am..." [Carousel]        │
│   3. "My client fired me..." [Reel]             │
│   4. "The $0 tool that..." [Reel]               │
├─────────────────────────────────────────────────┤
│ ❌ Rejected (3)                                  │
│   5. "AI will replace..." [Reel]                │
│   6. "Here's my morning..." [Static]            │
│   7. "Unpopular opinion..." [Carousel]          │
├─────────────────────────────────────────────────┤
│ ⏳ Draft/Expanded (3)                            │
│   8. "I spent 47 hours..." [Reel]               │
│   9. "Your competitors..." [Carousel]           │
│  10. "The real reason..." [Reel]                │
└─────────────────────────────────────────────────┘
```

#### Step 3: Iterate on Accepted (if --iterate)

For each approved idea, use AI chat to expand:
```
POST /v1/ideapad/chat
{
  "note_id": "{id}",
  "messages": [
    { "role": "user", "content": "Expand this into a full content brief with: 1) Opening hook delivery (first 3 seconds), 2) Core message structure, 3) CTA strategy, 4) Visual/format notes" }
  ]
}
```

Update the note with the expanded brief:
```
PATCH /v1/ideapad/notes/{id}
{ "body": "{original + expanded brief}" }
```

### Output:
Display review dashboard. If `--iterate`, show expanded briefs. Prompt:
> "Ready to produce? Run `/socioverse produce` to generate scripts and push to Content Bank."

---

## Mode 5: Produce

**Command:** `/socioverse produce [--ideas ID1,ID2] [--format FORMAT] [--push-to-bank]`

Takes approved ideas and produces full content - scripts, carousels (HTML→PNG), captions (short+long) - then pushes to Content Bank.

### Parameters:
| Flag | Default | Description |
|------|---------|-------------|
| `--ideas` | all approved | Specific idea IDs to produce |
| `--format` | auto | Force format: reel, carousel, static |
| `--push-to-bank` | true | Auto-push to Content Bank |
| `--generate-caption` | true | Auto-generate captions via /write-caption logic |
| `--collection` | auto | Content Bank collection to push to |
| `--carousel-style` | socioverse-violet | Style preset for carousels |

### Steps:

#### Step 1: Load Accepted Ideas
```
GET /v1/ideapad/notes?status=approved
```
Or if `--ideas` specified, fetch each:
```
GET /v1/ideapad/notes/{id}
```
Response includes `ai_expanded` field with full concept breakdown.

#### Step 2: Load Context for Production
```
GET /v1/knowledge
```
Pull brand voice, style guidelines, and audience context. This informs caption tone, carousel copy density, and script hooks.

#### Step 3: Create Collection for This Batch
```
POST /v1/content-bank/collections/create
{ "name": "Week of {date}", "description": "Produced batch from {date} pipeline run", "color": "#7c3aed" }
```

#### Step 4: Produce Each Idea

**For Reels:**

Push idea to Content Studio pipeline:
```
POST /v1/ideapad/push-to-pipeline
{ "note_id": "{id}" }
```
Response: `{ "idea_id": "uuid" }`

Then generate script:
```
POST /v1/scripts/generate
{ "idea_id": "{content_studio_idea_id}", "user_custom_idea": "{expanded brief from ai_expanded}" }
```
Response: `{ "script_id": "uuid", "revision_number": 1, "provider": "gemini" }`

Script output includes: hook, scene breakdown, voiceover text, text overlays, CTA, and timing.

---

**For Carousels (integrated /social-media-carousel workflow):**

Build a full carousel using these production rules:

**Structure (7-slide framework):**
1. Cover slide: Hook from the idea (pattern interrupt or bold claim)
2. Slide 2-5: Content slides (1 key point per slide)
3. Slide 6: Summary/recap
4. Slide 7: CTA slide

**Design Rules:**
- Dimensions: 1080x1350px (4:5 ratio for Instagram/LinkedIn)
- MANDATORY: Alternate light/dark slides throughout
- Max 30-40 words per content slide
- Gradient palette rotation between slides
- Brand logo on every slide (bottom-right, semi-transparent)
- Font pairing: Bold sans-serif heading + regular body

**Style Presets:**
| Preset | Colors | Use for |
|--------|--------|---------|
| socioverse-violet | Purple gradients, white text | Default/product |
| gradient | Multi-color shifting gradients | Educational |
| pink-flow | Pink/coral, soft shadows | Story/personal |
| dark-mode | Near-black, neon accents | Technical |
| clean-white | White bg, bold black text | Minimalist |

**Production Steps:**
1. Generate slide content from idea's `hookline`, `bullets`, and `details`
2. Create HTML files per slide using CSS variables for theming
3. Screenshot each HTML to PNG (1080x1350) using Chrome DevTools or Playwright
4. Upload PNGs to get public media URLs

**Slide HTML template structure:**
```html
<div class="slide" style="width:1080px;height:1350px;...">
  <div class="content">
    <h1>{heading}</h1>
    <p>{body}</p>
  </div>
  <img class="logo" src="{brand_logo}" />
  <span class="slide-number">{n}/{total}</span>
</div>
```

---

**For Static Posts:**

Generate visual direction + caption-focused content.

---

#### Step 5: Generate Captions (integrated /write-caption workflow)

For EACH produced piece, generate two caption variants:

**Caption Generation Rules (voice-native):**
- Authentic, first person, no corporate speak
- Never start with "I'm excited to share" or "So..."
- Failures are content - vulnerability > polish
- CTA must feel natural, not forced

**Variant 1 - SHORT (50-100 words):**
- Punchy opener (hook from the content)
- 2-3 lines of value
- Soft CTA or question
- 3-5 relevant hashtags

**Variant 2 - LONG (150-250 words):**
- Hook line (earns the scroll-stop)
- Story/context (why this matters)
- Core value delivery (the meat)
- CTA with clear next step
- Line breaks for readability
- 10-15 hashtags

**For carousel captions specifically:**
- Reference "swipe" or slide count
- Tease what's on the last slide
- Ask which slide resonated most

**Using Socioverse API for caption generation:**
If the content is already scheduled:
```
POST /v1/captions/generate
{ "scheduled_post_id": "{id}" }
```
Response: `{ "caption": "...", "hashtags": ["#contentcreator", "#reels"] }`

Or generate manually using knowledge base context and write-caption rules above.

#### Step 6: Push to Content Bank

For each produced content piece:
```
POST /v1/content-bank/create
{
  "media_type": "{reel|carousel|image}",
  "media_url": "{url - rendered video/first carousel slide}",
  "caption_draft": "{long caption variant}",
  "tags": ["{pillar}", "{format}", "batch-{date}"],
  "collection_id": "{collection_id from step 3}",
  "thumbnail_url": "{thumbnail/cover slide URL}",
  "notes": "SHORT CAPTION:\n{short variant}\n\n---\nFULL SCRIPT/BRIEF:\n{script or slide content}"
}
```

For carousels with multiple slides, the `content_bank_item_media` children are auto-created when multiple URLs are provided.

#### Step 7: Update IdeaPad Status

Mark produced ideas:
```
PATCH /v1/ideapad/notes/{id}
{ "status": "produced", "tags": [...existing, "produced"] }
```

### Output:
Display production summary table:
```
┌─────────────────────────────────────────────────────────────────┐
│ PRODUCTION COMPLETE                                              │
├─────────────────────────────────────────────────────────────────┤
│ # │ Idea                      │ Format   │ Slides │ Status      │
│ 1 │ "I automated my..."       │ Reel     │ -      │ ✓ In Bank   │
│ 2 │ "Stop posting at 9am..."  │ Carousel │ 7      │ ✓ In Bank   │
│ 3 │ "My client fired me..."   │ Reel     │ -      │ ✓ In Bank   │
│ 4 │ "The $0 tool that..."     │ Carousel │ 8      │ ✓ In Bank   │
├─────────────────────────────────────────────────────────────────┤
│ Content Bank: 4 items added to "Week of 2026-05-19"            │
│ Captions: 8 variants generated (4 short + 4 long)              │
│ Carousel PNGs: 15 slides rendered                               │
└─────────────────────────────────────────────────────────────────┘
```

Prompt:
> "Content ready in your Content Bank. Run `/socioverse schedule` to distribute across the week."

---

## Mode 6: Schedule

**Command:** `/socioverse schedule [--week DATE] [--times SLOTS] [--accounts ACCT] [--from-bank COLLECTION]`

Pulls content from Content Bank and schedules across the week.

### Parameters:
| Flag | Default | Description |
|------|---------|-------------|
| `--week` | next Monday | Target week (YYYY-MM-DD) |
| `--times` | 9:00,12:00,18:00 IST | Posting time slots |
| `--accounts` | all connected | Specific IG account IDs |
| `--from-bank` | latest collection | Collection ID or "unscheduled" |
| `--priority` | reels-first | Ordering: reels-first, carousels-first, mixed |

### Steps:

#### Step 1: Get Connected Accounts
```
GET /v1/instagram/accounts
```

#### Step 2: Pull Content from Bank
```
GET /v1/content-bank?collection_id={id}&status=unused
```
Or get all unscheduled:
```
GET /v1/content-bank?status=unused&limit=100
```

#### Step 3: Get Existing Schedule (avoid conflicts)
```
GET /v1/posts/scheduled
```
Map existing scheduled slots to avoid double-booking.

#### Step 4: Build Schedule Grid

Default posting times (IST, converted to UTC):
- Morning: 09:00 IST (03:30 UTC)
- Midday: 12:00 IST (06:30 UTC)
- Evening: 18:00 IST (12:30 UTC)

Distribution rules:
- Reels get prime slots (morning + evening)
- Carousels get midday slots
- Max 2 posts per day per account
- Interleave accounts if multiple connected
- Weekends get 1 post/day max

#### Step 5: Schedule Each Post
For each content item matched to a slot:
```
POST /v1/posts/schedule
{
  "caption": "{caption from content bank item}",
  "scheduled_at": "{ISO 8601 UTC timestamp}",
  "instagram_account_id": "{account_id}",
  "media_url": "{media_url from content bank}",
  "media_type": "{reel|image|carousel|story}",
  "thumbnail_url": "{thumbnail if reel}"
}
```
`media_type` accepts: `reel`, `image`, `carousel`, or `story`. For `story`, omit/leave caption empty — stories do not display feed captions.

##### Posting Stories (v1)
Stories use a dedicated endpoint that pulls from the same scheduled_posts → create-instagram-container → publish-instagram-reel pipeline.

```
POST /v1/stories/post
{
  "instagram_account_id": "{account_id}",
  "image_url": "{publicly reachable JPEG/PNG, ideally 1080x1920 9:16}",
  "scheduled_at": "{ISO 8601 UTC, OR omit to publish ASAP}",
  "content_bank_item_id": "{optional, links the story back to its bank source}"
}
```
- Stories expire 24h after publish (Meta).
- v1 limitations (Meta API): no stickers, link sticker, polls, music, mention sticker, or carousel stories. Only static image. Add stickers/links manually if needed.
- Quota: 100 published API posts per IG account per 24h, shared across feed posts + reels + stories.

```
GET /v1/stories
```
Returns the workspace's last 30 days of scheduled / published stories.

#### Step 6: Update Content Bank Status
```
PATCH /v1/content-bank/{id}
{ "status": "scheduled" }
```

### Output:
Display the schedule grid:
```
┌──────────────────────────────────────────────────────────┐
│ SCHEDULE - Week of 2026-05-19                            │
├──────────────────────────────────────────────────────────┤
│ Mon 09:00 IST │ @personal   │ Reel: "I automated..."  ✓ │
│ Mon 18:00 IST │ @socioverse │ Reel: "The $0 tool..."  ✓ │
│ Tue 09:00 IST │ @personal   │ Reel: "My client..."    ✓ │
│ Tue 12:00 IST │ @socioverse │ Carousel: "Stop post.." ✓ │
│ Wed 09:00 IST │ @personal   │ Reel: "47 hours..."     ✓ │
│ ...                                                      │
├──────────────────────────────────────────────────────────┤
│ Total: 9 posts scheduled across 2 accounts               │
└──────────────────────────────────────────────────────────┘
```

---

## Mode 7: Pipeline (Full Orchestration)

**Command:** `/socioverse pipeline [--week DATE] [--analyze-count N] [--idea-count N] [--auto]`

Runs the complete content lifecycle end-to-end with strategic pause points.

### Parameters:
| Flag | Default | Description |
|------|---------|-------------|
| `--week` | next Monday | Target week |
| `--analyze-count` | 6 | Posts to deep-analyze |
| `--idea-count` | 10 | Ideas to generate |
| `--auto` | false | Skip pauses (fully autonomous) |
| `--resume` | - | Resume from: review, produce, schedule |

### Full Pipeline Flow:

```
┌─────────────────────────────────────────────────────────────┐
│                    SOCIOVERSE PIPELINE                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │ KNOWLEDGE│───▶│  INTEL   │───▶│  IDEATE  │              │
│  │   BASE   │    │ Scrape + │    │ Generate │              │
│  │          │    │ Analyze  │    │ + Push   │              │
│  └──────────┘    └──────────┘    └──────────┘              │
│                                        │                     │
│                                        ▼                     │
│                                  ┌──────────┐               │
│                                  │ IDEAPAD  │               │
│                                  │ User     │               │
│                                  │ Reviews  │  ◀── PAUSE 1  │
│                                  └──────────┘               │
│                                        │                     │
│                                        ▼                     │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │ SCHEDULE │◀───│ CONTENT  │◀───│ PRODUCE  │              │
│  │ Distribute│   │   BANK   │    │ Scripts  │              │
│  │ + Post   │    │  Store   │    │ + Media  │              │
│  └──────────┘    └──────────┘    └──────────┘              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Execution:

**Phase 1 - Intelligence (auto):**
Runs `/socioverse intel --analyze-count {N}` internally.

**Phase 2 - Ideation (auto → PAUSE):**
Runs `/socioverse ideate --from-intel --count {N}` internally.

**⏸ PAUSE 1** (unless `--auto`):
> "Ideas pushed to IdeaPad. Review and accept/reject in Socioverse UI. Then: `/socioverse pipeline --resume review`"

**Phase 3 - Review + Iterate (auto):**
Runs `/socioverse review --iterate` internally. Expands approved ideas.

**Phase 4 - Production (auto):**
Runs `/socioverse produce --push-to-bank` internally.

**Phase 5 - Scheduling (auto):**
Runs `/socioverse schedule --week {date} --from-bank {collection}` internally.

### Auto Mode (--auto):
Skips pause. Uses AI judgment to auto-approve top-scoring ideas (top 60% by expected impact). Produces and schedules without human review. Good for established accounts with consistent content pillars.

---

## Mode 8: Automate

**Command:** `/socioverse automate [ACTION] [--type TYPE]`

Manage DM automations, sequences, and lead magnets.

### Actions:

#### `list` - List all automations
```
GET /v1/automations
GET /v1/sequences
```
Display active automations with trigger type, message preview, and stats.

#### `create` - Create new automation
```
/socioverse automate create --type keyword --keyword "free,guide" --message "Hey {first_name}! Here's your free guide" --lead-magnet ID
```

Steps:
1. `GET /v1/instagram/accounts` - get account ID
2. Build automation config based on type:

**Keyword on Post:**
```
POST /v1/automations/create
{
  "name": "{auto-generated from keyword}",
  "trigger_type": "keyword_on_post",
  "instagram_account_id": "{id}",
  "trigger_params": {
    "keywords": ["free", "guide"],
    "follow_gate_enabled": true
  },
  "message_text": "Hey {first_name}! Here's your free guide 👇",
  "follow_gate_message": "Follow us first to get access!",
  "follow_gate_button_text": "I followed!",
  "lead_magnet_id": "{id}",
  "is_active": true
}
```

**Story Mention:**
```
POST /v1/automations/create
{
  "name": "Story Mention Thank You",
  "trigger_type": "story_mention",
  "instagram_account_id": "{id}",
  "message_text": "Thanks for the mention {first_name}! 🙏",
  "is_active": true
}
```

**Story Reply (any text):**
```
POST /v1/automations/create
{
  "name": "Story Reply Engagement",
  "trigger_type": "story_reply",
  "instagram_account_id": "{id}",
  "trigger_params": {},
  "message_text": "Thanks for replying to my story 🙌",
  "is_active": true
}
```
Triggers when anyone replies to your story with text (24h messaging window applies, like Meta's policy).

**Story Reply (keyword-filtered):**
```
POST /v1/automations/create
{
  "name": "Send the guide on story reply",
  "trigger_type": "story_reply",
  "instagram_account_id": "{id}",
  "trigger_params": {
    "keywords": ["guide", "send", "yes"]
  },
  "message_text": "Here's your guide 👉 {{link}}",
  "is_active": true
}
```
The reply text is matched case-insensitively against the keyword list. Leave `keywords` off to match every reply.

**Story Reply (specific story only, e.g. one with a CTA):**
```
POST /v1/automations/create
{
  "name": "Reply to launch story = early access",
  "trigger_type": "story_reply",
  "instagram_account_id": "{id}",
  "trigger_params": {
    "story_id": "<ig_story_media_id>",
    "keywords": ["yes"]
  },
  "message_text": "Welcome to early access ✨"
}
```

Implementation notes (for debugging story-reply automations):
- The Meta webhook field is `messages` (NOT a separate story webhook). Socioverse detects `message.reply_to.story` for replies and `message.attachments[0].type === 'story_mention'` for mentions.
- All story DMs respect the standard 24-hour messaging window, the 200 DMs/hour cap, and the 1-DM-per-user-per-24h limit on story-triggered automations.
- Story reactions / GIF replies do NOT trigger a webhook (Meta limitation). Only text replies work.
- Private accounts mentioning you in a story will NOT trigger a webhook unless you follow them back.

#### `update` - Update automation
```
PATCH /v1/automations/{id}
{ ...fields to update }
```

#### `delete` - Delete automation
```
DELETE /v1/automations/{id}
```

#### `sequence` - Manage DM sequences

**List sequences:**
```
GET /v1/sequences
```
Response includes: `steps_count`, `enrolled_count`, `completed_count`, `trigger_type`, `status`.

**Get sequence with steps:**
```
GET /v1/sequences/{id}
```
Response includes `dm_sequence_steps` array with `step_order`, `message_text`, `delay_minutes`.

**Create a sequence:**
```
POST /v1/sequences/create
{
  "name": "Onboarding Drip",
  "description": "3-day welcome sequence for new leads",
  "trigger_type": "manual",
  "goal_type": "email_capture",
  "is_active": true
}
```
Fields:
- `trigger_type`: "automation" | "manual" (default) | "scheduled"
- `goal_type`: "email_capture" | "booking" | "sale" | "engagement" (default)

**Update a sequence:**
```
PATCH /v1/sequences/{id}
{ "name": "Updated Name", "is_active": false, "goal_type": "booking" }
```

**Delete a sequence:**
```
DELETE /v1/sequences/{id}
```

**Start/pause/resume sequence for a lead:**
```
POST /v1/sequences/start
{ "sequence_id": "{id}", "lead_id": "{lead_id}", "action": "start" }
```
Actions: `"start"` | `"pause"` | `"resume"`
Response: `{ "conversation_id": "conv_abc123", "action": "start", "next_step_at": "2026-03-08T13:00:00Z" }`

#### `lead-magnet` - Generate lead magnets

```
POST /v1/lead-magnets/generate
{ "source_type": "custom", "custom_topic": "5 Instagram automation hacks for agencies" }
```

Then generate PDF:
```
POST /v1/lead-magnets/generate-pdf
{ "lead_magnet_id": "{id}" }
```

List existing:
```
GET /v1/lead-magnets
```

### Templates:

Manage message templates for reuse:
```
GET    /v1/templates
POST   /v1/templates/create   { "name": "Welcome DM", "body": "Hey {first_name}!..." }
PATCH  /v1/templates/{id}     { "body": "updated text" }
DELETE /v1/templates/{id}
```

---

## Mode 9: Leads

**Command:** `/socioverse leads [ACTION]`

Full lead management - CRM-lite functionality.

### Actions:

#### `list` - List all leads
```
GET /v1/leads?limit=50
```
Display with: username, stage, source, score, tags.

#### `get` - Get lead details
```
GET /v1/leads/{id}
```

#### `add` - Add new lead
```
POST /v1/leads/create
{
  "ig_username": "targetuser",
  "source_type": "manual|automation|comment|dm",
  "stage": "new|warm|hot|customer|lost",
  "tags": ["agency-owner", "interested-in-automation"],
  "full_name": "John Doe",
  "email": "john@example.com",
  "score": 75,
  "marketing_opt_in": true
}
```

#### `update` - Update lead
```
PATCH /v1/leads/{id}
{ "stage": "hot", "score": 90, "tags": [...] }
```

#### `delete` - Delete lead
```
DELETE /v1/leads/{id}
```

#### `lookup` - Profile lookup
```
POST /v1/instagram/profile/lookup
{ "username": "targetuser" }
```
Returns public profile data: bio, follower count, post count, category.

#### `message` - Send DM to lead
```
POST /v1/messages/send
{
  "lead_id": "{id}",
  "message_body": "Hey {first_name}, loved your recent post about {topic}!",
  "button_text": "Check this out",
  "button_url": "https://..."
}
```
Note: Max 1000 chars body, 20 chars button text. Supports variables: `{username}`, `{first_name}`, `{name}`, `{email}`.

#### `messages` - Message history
```
GET /v1/messages?limit=50
```

#### `sequence` - Start a lead on a sequence
```
POST /v1/sequences/start
{ "sequence_id": "{id}", "lead_id": "{lead_id}", "action": "start" }
```

---

## Mode 10: Report

**Command:** `/socioverse report [--generate] [--send EMAIL] [--insights] [--week-offset N]`

Weekly reporting and Instagram analytics.

### Parameters:
| Flag | Default | Description |
|------|---------|-------------|
| `--generate` | false | Generate a fresh report |
| `--send` | - | Email address to deliver report to |
| `--insights` | false | Show full Instagram insights |
| `--week-offset` | 1 | 0=this week, 1=last week, 2=two weeks ago |
| `--account` | first connected | Instagram account ID |

### Actions:

#### Default - View latest report
```
GET /v1/reports?limit=1
GET /v1/reports/{id}
```
Response includes:
```json
{
  "id": "uuid",
  "ig_username": "yourhandle",
  "week_start": "2026-05-04",
  "week_end": "2026-05-10",
  "pdf_url": "https://cdn.socioverse.io/reports/uuid.html",
  "ai_summary": "Reach grew 14% this week driven by 3 reels...",
  "recipient_email": null,
  "email_sent_at": null
}
```
Display AI summary, link to PDF, key metrics.

#### `--generate` - Generate new report
```
GET /v1/instagram/accounts   # get instagram_account_id
POST /v1/reports/generate
{
  "instagram_account_id": "{id}",
  "week_offset": 1,
  "recipient_email": "optional@email.com",
  "send_email": false
}
```
**Note:** Takes 15-60 seconds. Set HTTP timeout to 90s+.
Response: `{ "report_id": "uuid", "pdf_url": "https://cdn.socioverse.io/reports/uuid.html", "email_sent_at": null }`

#### `--send` - Email report to client
```
POST /v1/reports/{id}/send
{ "email": "client@agency.com" }
```
Response: `{ "sent": true, "email": "client@agency.com", "email_sent_at": "2026-05-11T09:15:00Z" }`

Useful for agencies automating client delivery: generate report, then email via this endpoint.

#### `--insights` - Full Instagram insights
```
GET /v1/instagram/insights
```
**Insights window limitation (Meta Graph API):** `media_list` only includes the most recent ~12 posts with per-post `insights_data` (reach, likes, comments, saves, shares). Instagram does NOT retain per-post insights for older media via the Graph API, so for an account with 96 posts only the most recent ~12 will have `insights_data`. Use `daily_metrics` + `reach_time_series` for account-level history; per-post insights for older posts cannot be retrieved.
Response includes:
```json
{
  "daily_metrics": { "reach": 12500, "accounts_engaged": 890, "likes": 1240, "comments": 156, "shares": 89, "saves": 234 },
  "reach_time_series": [{ "date": "2026-03-01", "value": 4200 }],
  "demographics": {
    "cities": [{ "name": "Mumbai", "value": 1200 }],
    "countries": [{ "name": "IN", "value": 8500 }]
  },
  "media_list": [{ "id": "media_123", "media_type": "REELS", "insights_data": { "reach": 5600, "likes": 340 } }],
  "account_info": { "ig_user_id": "12345", "username": "mybrand", "followers_count": 45200 }
}
```
Display:
- Follower count + growth
- Reach and impressions (daily)
- Top performing content by reach
- Audience demographics (cities, countries)
- Engagement rate per media type

#### `--list` - All historical reports
```
GET /v1/reports?limit=20
```

---

## Webhooks (Event-Driven Automation)

For advanced users who want to trigger external workflows:

### Setup webhook:
```
POST /v1/webhooks/create
{
  "url": "https://your-server.com/webhook",
  "events": ["lead.created", "message.sent", "automation.triggered", "sequence.completed", "report.generated"]
}
```

### Available events:
| Event | Fires when |
|-------|-----------|
| `lead.created` | New lead added (manual or auto) |
| `lead.updated` | Lead info changed |
| `message.sent` | DM delivered successfully |
| `message.failed` | DM delivery failed |
| `automation.triggered` | Automation fired for a user |
| `sequence.completed` | All sequence steps done |
| `report.generated` | Weekly report ready |
| `report.sent` | Report emailed |
| `lead_magnet.generated` | PDF lead magnet ready |

### Verify webhooks:
All payloads signed with HMAC-SHA256. Check `X-Socioverse-Signature` header against your webhook secret (`whsec_*`).

### Manage:
```
GET    /v1/webhooks
PATCH  /v1/webhooks/{id}   { "events": [...], "is_active": true/false }
DELETE /v1/webhooks/{id}
```

---

## Bio Link Management

Quick access to bio link profile:

```
GET  /v1/biolink/profile
POST /v1/biolink/profile/update
{ "display_name": "Your Name", "bio": "Updated bio text", "theme": "dark" }
```

---

## Content Studio (Direct Access)

For power users who want granular control over the content pipeline:

### Browse scraped posts:
```
GET /v1/content-studio/scraped-posts?post_type=reel&status=unprocessed&competitor_id={id}&pipeline_run_id={run_id}&transcript_status=available
GET /v1/content-studio/scraped-posts/{id}
```
Supported filters: `post_type` (reel/image/carousel), `status` (unprocessed/analyzing/analyzed/failed), `competitor_id`, `pipeline_run_id` (matches the scrape_batch_id from the scrape run), `transcript_status` (pending/available/unavailable/failed). Full response schema is in Mode 2 Step 3 above.

### Poll async pipeline runs:
```
GET /v1/content-studio/pipeline-runs?pipeline_type=scrape&status=running
GET /v1/content-studio/pipeline-runs/{id}
```
`POST /v1/competitors/scrape` and `POST /v1/content-studio/run-pipeline` are async and return a `pipeline_run_id`. Poll the `:id` endpoint every 3-5s until `is_terminal: true`, then read the results from scraped-posts / analyzed-posts.

**Reel transcript fields** (populated by `/v1/videos/analyze` for reel-type posts):
```json
{
  "id": "uuid",
  "post_type": "reel",
  "status": "analyzed",
  "transcript": "Most creators think more posting equals more growth. But here's what nobody tells you...",
  "transcript_status": "available",
  "transcribed_at": "2026-03-08T10:00:00Z"
}
```
`transcript_status` values:
- `pending` - never analyzed yet, or non-reel post (default)
- `available` - transcript text present in `transcript`
- `unavailable` - reel had no spoken words or on-screen text (silent / music-only)
- `failed` - Gemini multimodal upload failed and analysis fell back to text-only; re-run `/v1/videos/analyze` after a CDN refresh to retry

Transcripts are NOT extracted for `image` or `carousel` posts - those will always have `transcript_status: "pending"` and `transcript: null`.

### Browse analyzed posts:
```
GET /v1/content-studio/analyzed-posts
GET /v1/content-studio/analyzed-posts/{id}
```

### Manage ideas:
```
GET   /v1/content-studio/ideas?status=pending&post_type=reel
PATCH /v1/content-studio/ideas/{id}   { "status": "approved" }
PATCH /v1/content-studio/ideas/{id}   { "status": "rejected" }
```

**Idea response fields (used in produce mode):**
```json
{
  "id": "uuid",
  "idea": "5 tips for engagement",
  "hookline": "Most creators miss this...",
  "details": "Detailed breakdown of the content angle...",
  "bullets": ["Tip 1", "Tip 2", "Tip 3"],
  "post_type": "reel",
  "visual_direction": "Dark moody aesthetic with text overlays",
  "slide_outline": ["Cover: hook", "Slide 2: problem", "Slide 3: solution"],
  "status": "pending"
}
```
The `hookline`, `bullets`, and `slide_outline` fields directly feed carousel production. `visual_direction` informs style selection.

### Run pipeline with fine control:
```
POST /v1/content-studio/run-pipeline
{
  "post_types": ["reel", "carousel"],
  "max_reels_per_competitor": 5,
  "max_reels_to_analyze": 8,
  "max_ideas_to_generate": 12,
  "start_from_step": "analyze",
  "target_ids": ["specific-post-id-1", "specific-post-id-2"]
}
```

---

## Composable Workflows

These are pre-built workflow recipes combining multiple modes:

### 1. Weekly Content Engine (recommended)
```
/socioverse pipeline --week 2026-05-19 --idea-count 12
```
Full cycle: intel → ideate → review → produce → schedule.

### 2. Competitor React
```
/socioverse intel --analyze-count 3
/socioverse ideate --from-intel --count 5 --topic "react to competitor trend"
```
Quick competitive response content.

### 3. Lead Magnet Funnel
```
/socioverse automate create --type keyword --keyword "free" --lead-magnet {id}
/socioverse leads lookup targetuser
/socioverse leads message {id} "Here's your exclusive access..."
```
End-to-end funnel from trigger to DM.

### 4. Content Refresh
```
/socioverse report --insights
/socioverse ideate --goal conversion --count 5
/socioverse produce --format carousel
/socioverse schedule --priority carousels-first
```
Data-driven content refresh based on what's working.

### 5. Engagement Autopilot
```
/socioverse automate create --type story_mention --message "Thanks for sharing! 🙏"
/socioverse automate create --type story_reply --message "Appreciate you! What did you think?"
/socioverse automate sequence create --name "New Follower Drip" --steps [...]
```
Set-and-forget engagement automation.

---

## API Reference (Complete)

### All Endpoints

| Method | Endpoint | Scope |
|--------|----------|-------|
| GET | `/v1/health` | - |
| GET | `/v1/usage` | - |
| GET | `/v1/keys/verify` | - |
| POST | `/v1/scripts/generate` | scripts |
| POST | `/v1/ideas/generate` | ideas |
| POST | `/v1/captions/generate` | captions |
| POST | `/v1/lead-magnets/generate` | leads |
| POST | `/v1/lead-magnets/generate-pdf` | leads |
| GET | `/v1/lead-magnets` | leads |
| GET | `/v1/lead-magnets/:id` | leads |
| DELETE | `/v1/lead-magnets/:id` | leads |
| POST | `/v1/videos/analyze` | videos |
| GET | `/v1/posts/scheduled` | scheduling |
| POST | `/v1/posts/schedule` | scheduling |
| POST | `/v1/posts/publish` | publishing |
| GET | `/v1/content-bank` | content_bank |
| GET | `/v1/content-bank/:id` | content_bank |
| POST | `/v1/content-bank/create` | content_bank |
| PATCH | `/v1/content-bank/:id` | content_bank |
| DELETE | `/v1/content-bank/:id` | content_bank |
| GET | `/v1/content-bank/collections` | content_bank |
| POST | `/v1/content-bank/collections/create` | content_bank |
| PATCH | `/v1/content-bank/collections/:id` | content_bank |
| DELETE | `/v1/content-bank/collections/:id` | content_bank |
| GET | `/v1/ideapad/notes` | ideapad |
| GET | `/v1/ideapad/notes/:id` | ideapad |
| POST | `/v1/ideapad/notes/create` | ideapad |
| PATCH | `/v1/ideapad/notes/:id` | ideapad |
| DELETE | `/v1/ideapad/notes/:id` | ideapad |
| POST | `/v1/ideapad/expand` | ideapad |
| POST | `/v1/ideapad/chat` | ideapad |
| POST | `/v1/ideapad/push-to-pipeline` | ideapad |
| GET | `/v1/content-studio/scraped-posts` | content_studio |
| GET | `/v1/content-studio/scraped-posts/:id` | content_studio |
| GET | `/v1/content-studio/analyzed-posts` | content_studio |
| GET | `/v1/content-studio/analyzed-posts/:id` | content_studio |
| GET | `/v1/content-studio/ideas` | content_studio |
| PATCH | `/v1/content-studio/ideas/:id` | content_studio |
| GET | `/v1/content-studio/pipeline-runs` | content_studio |
| GET | `/v1/content-studio/pipeline-runs/:id` | content_studio |
| POST | `/v1/content-studio/run-pipeline` | content_studio |
| GET | `/v1/automations` | automations |
| POST | `/v1/automations/create` | automations |
| PATCH | `/v1/automations/:id` | automations |
| DELETE | `/v1/automations/:id` | automations |
| GET | `/v1/sequences` | sequences |
| GET | `/v1/sequences/:id` | sequences |
| POST | `/v1/sequences/create` | sequences |
| POST | `/v1/sequences/start` | sequences |
| PATCH | `/v1/sequences/:id` | sequences |
| DELETE | `/v1/sequences/:id` | sequences |
| GET | `/v1/competitors` | competitors |
| POST | `/v1/competitors/scrape` | competitors |
| GET | `/v1/instagram/accounts` | instagram |
| GET | `/v1/instagram/insights` | instagram |
| POST | `/v1/instagram/profile/lookup` | instagram |
| GET | `/v1/leads` | lead_bank |
| GET | `/v1/leads/:id` | lead_bank |
| POST | `/v1/leads/create` | lead_bank |
| PATCH | `/v1/leads/:id` | lead_bank |
| DELETE | `/v1/leads/:id` | lead_bank |
| GET | `/v1/messages` | messages |
| POST | `/v1/messages/send` | messages |
| GET | `/v1/biolink/profile` | biolink |
| POST | `/v1/biolink/profile/update` | biolink |
| GET | `/v1/knowledge` | knowledge |
| POST | `/v1/knowledge/upload` | knowledge |
| DELETE | `/v1/knowledge/:id` | knowledge |
| GET | `/v1/reports` | reports |
| GET | `/v1/reports/:id` | reports |
| POST | `/v1/reports/:id/send` | reports |
| POST | `/v1/reports/generate` | reports |
| GET | `/v1/templates` | templates |
| POST | `/v1/templates/create` | templates |
| PATCH | `/v1/templates/:id` | templates |
| DELETE | `/v1/templates/:id` | templates |
| GET | `/v1/webhooks` | webhooks |
| POST | `/v1/webhooks/create` | webhooks |
| PATCH | `/v1/webhooks/:id` | webhooks |
| DELETE | `/v1/webhooks/:id` | webhooks |

### Pagination
All list endpoints: `?limit=50&offset=0` (max 100). Response includes `meta.pagination.has_more`.

### Error Codes
| Code | HTTP | Action |
|------|------|--------|
| INVALID_API_KEY | 401 | Check key in .env |
| EXPIRED_API_KEY | 401 | Generate new key from dashboard |
| INSUFFICIENT_SCOPE | 403 | Regenerate key with required scopes |
| PLAN_NO_API_ACCESS | 403 | Upgrade plan to one with API access |
| SUBSCRIPTION_INACTIVE | 403 | Renew subscription |
| TRIAL_EXPIRED | 403 | Subscribe to a paid plan |
| PLAN_LIMIT_REACHED | 403 | Upgrade to increase limits (e.g. max automations) |
| RATE_LIMIT_EXCEEDED | 429 | Wait 60s, retry |
| DAILY_LIMIT_EXCEEDED | 429 | Wait until next day |
| QUOTA_EXCEEDED | 429 | Wait for monthly reset or upgrade |
| NOT_FOUND | 404 | Verify resource ID exists |
| METHOD_NOT_ALLOWED | 405 | Wrong HTTP method for this endpoint |
| INVALID_REQUEST | 400 | Check required fields |
| BODY_TOO_LARGE | 413 | Request body exceeds 1MB limit |
| AI_GENERATION_FAILED | 500 | Retry once, then report |
| INTERNAL_ERROR | 500 | Unexpected server error - report to support |
| REPORT_NOT_READY | 400 | Report PDF is still generating - wait and retry |

### Common Field Formats
- All IDs: UUID format
- Dates: ISO 8601 UTC (`2026-05-19T03:30:00Z`)
- Media URLs: Must be publicly accessible HTTPS URLs
- Message body: Max 1000 chars, supports `{username}`, `{first_name}`, `{name}`, `{email}`
- Button text: Max 20 chars
- Tags: Array of strings
- Colors: String (blue, green, purple, yellow, red, orange)

---

## For Public Users

### Setup:
1. Sign up at https://www.socioverse.io
2. Go to Dashboard → Settings → API Keys
3. Create a key with all scopes (or restrict as needed)
4. Set `SOCIOVERSE_API_KEY=sk_live_...` in your `.env`
5. Run `/socioverse status` to verify

### Minimum viable workflow:
```bash
/socioverse status          # verify connection
/socioverse intel           # gather intelligence
/socioverse ideate          # generate & push ideas
# ... review in Socioverse UI ...
/socioverse review          # check what's approved
/socioverse produce         # create content
/socioverse schedule        # schedule for the week
```

### Tips:
- Run `/socioverse pipeline` for the full automated experience
- Use `--auto` flag for fully autonomous mode (no pauses)
- Ideas appear in your IdeaPad within seconds of generation
- Content Bank items are immediately visible in dashboard
- All scheduled posts appear in your calendar view
- Webhooks let you trigger n8n/Zapier/Make workflows on events
- Postman collection available from the API docs page in dashboard

---

## Testing All Endpoints

**Command:** `/socioverse test [--scope SCOPE] [--destructive]`

Run this to verify all API endpoints work with your key. Non-destructive by default (only GETs + safe POSTs).

### Quick Smoke Test (read-only):
```bash
# Test all read endpoints in sequence
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/health
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/usage
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/keys/verify
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/instagram/accounts
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/competitors
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/automations
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/sequences
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/leads
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/messages
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/content-bank
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/content-bank/collections
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/ideapad/notes
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/content-studio/scraped-posts
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/content-studio/analyzed-posts
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/content-studio/ideas
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/knowledge
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/lead-magnets
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/biolink/profile
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/reports
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/templates
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/webhooks
curl -s -H "Authorization: Bearer $SOCIOVERSE_API_KEY" https://api.socioverse.io/v1/instagram/insights
```

### Full Integration Test (creates + cleans up):

This test chain verifies the complete CRUD lifecycle:

```bash
# 1. System health
GET /v1/health                          # Expect: success=true, key_valid=true

# 2. Create a lead
POST /v1/leads/create                   # { ig_username: "test_api_lead", source_type: "api", tags: ["test"] }
# Save returned lead_id

# 3. Get the lead
GET /v1/leads/{lead_id}                 # Expect: ig_username="test_api_lead"

# 4. Update the lead
PATCH /v1/leads/{lead_id}              # { stage: "engaged", score: 50 }

# 5. Create an IdeaPad note
POST /v1/ideapad/notes/create          # { title: "Test Idea", body: "API test", tags: ["test"] }
# Save returned note_id

# 6. Expand the note
POST /v1/ideapad/expand                # { note_id: "{note_id}" }
# Expect: expanded text returned

# 7. Chat with AI about the note
POST /v1/ideapad/chat                  # { messages: [{ role: "user", content: "expand this" }], note_id: "{note_id}" }
# Expect: reply text

# 8. Create a content bank collection
POST /v1/content-bank/collections/create  # { name: "API Test", color: "#7c3aed" }
# Save returned collection_id

# 9. Create a content bank item
POST /v1/content-bank/create           # { media_type: "image", media_url: "https://placehold.co/1080x1350", collection_id: "{collection_id}" }
# Save returned item_id

# 10. Update content bank item
PATCH /v1/content-bank/{item_id}       # { caption_draft: "Test caption", tags: ["api-test"] }

# 11. Create a message template
POST /v1/templates/create              # { name: "API Test Template", body: "Hi {{name}}!", variables: ["name"] }
# Save returned template_id

# 12. Generate ideas (if analyzed posts exist)
POST /v1/ideas/generate                # { user_topic: "test automation", batch_size: 1 }

# 13. Profile lookup
POST /v1/instagram/profile/lookup      # { username: "instagram" }
# Expect: followers_count, biography, etc.

# --- CLEANUP ---
# 14. Delete test resources
DELETE /v1/leads/{lead_id}
DELETE /v1/ideapad/notes/{note_id}
DELETE /v1/content-bank/{item_id}
DELETE /v1/content-bank/collections/{collection_id}
DELETE /v1/templates/{template_id}
```

### Test Output Format:
```
┌───────────────────────────────────────────────────────┐
│ API ENDPOINT TEST RESULTS                              │
├───────────────────────────────────────────────────────┤
│ ✓ GET  /v1/health                    200  12ms        │
│ ✓ GET  /v1/usage                     200  45ms        │
│ ✓ GET  /v1/keys/verify               200  23ms        │
│ ✓ GET  /v1/instagram/accounts        200  67ms        │
│ ✓ GET  /v1/competitors               200  34ms        │
│ ✓ GET  /v1/automations               200  89ms        │
│ ✓ GET  /v1/sequences                 200  43ms        │
│ ✓ GET  /v1/leads                     200  56ms        │
│ ✓ POST /v1/leads/create              200  123ms       │
│ ✓ GET  /v1/leads/:id                 200  34ms        │
│ ✓ PATCH /v1/leads/:id                200  45ms        │
│ ✓ DELETE /v1/leads/:id               200  23ms        │
│ ✓ POST /v1/ideapad/notes/create      200  67ms        │
│ ✓ POST /v1/ideapad/expand            200  2340ms      │
│ ✓ POST /v1/ideapad/chat              200  1890ms      │
│ ... (all endpoints)                                    │
├───────────────────────────────────────────────────────┤
│ PASSED: 21/21  |  FAILED: 0  |  SKIPPED: 0           │
│ Total time: 8.4s  |  Avg response: 234ms             │
│ Rate limit remaining: 39/60                           │
└───────────────────────────────────────────────────────┘
```

### Scope-Specific Testing:
```bash
/socioverse test --scope content_bank    # Only test content bank endpoints
/socioverse test --scope automations     # Only test automation endpoints
/socioverse test --scope ideapad         # Only test IdeaPad endpoints
```

---

## Publishing This Skill

### For skill authors who want to distribute /socioverse:

**Step 1 - Validate:**
```bash
python "d:/Anshul/Personal Assistant/.agents/skills/skill-creator/scripts/quick_validate.py" \
  "d:/Anshul/Personal Assistant/Socials/.claude/skills/socioverse/"
```

**Step 2 - Package:**
```bash
python "d:/Anshul/Personal Assistant/.agents/skills/skill-creator/scripts/package_skill.py" \
  "d:/Anshul/Personal Assistant/Socials/.claude/skills/socioverse/" \
  "./dist/"
```
Creates `socioverse.skill` (ZIP) in `./dist/`.

**Step 3 - Push to GitHub:**
```bash
# Create public repo
gh repo create AnshulRastogi20/socioverse-skill --public
# Push skill files
git add . && git commit -m "v1.0.0 - Industry-ready Socioverse skill"
git push origin main
```

**Step 4 - Register on skills.sh:**
Once on GitHub, skills.sh crawls and indexes automatically. Users install with:
```bash
npx skills add AnshulRastogi20/socioverse-skill
```

**Step 5 - Submit to ClawHub (optional):**
Upload the `.skill` package to ClawHub marketplace for broader distribution.

### Requirements for public release:
- [x] Valid YAML frontmatter with name, version, license, description
- [x] `compatibility` field present
- [x] `metadata.openclaw.requires.env` lists required env vars
- [x] `metadata.tags` for discoverability
- [x] `metadata.triggers` for activation phrases
- [x] Self-contained (no dependency on private skills)
- [x] Works with any Socioverse account (key-based auth)
- [ ] Add LICENSE.txt to skill directory
- [ ] Run validation script (zero errors)
- [ ] Test with a fresh API key on a new account

---
> Source: [AnshulRastogi20/socioverse-skill](https://github.com/AnshulRastogi20/socioverse-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
