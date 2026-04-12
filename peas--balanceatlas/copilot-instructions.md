## balanceatlas

> Community-driven directory for the global handstand / hand balancing community.

# Balance Atlas

Community-driven directory for the global handstand / hand balancing community.
**Site**: balanceatlas.com | **Repo**: github.com/balanceatlas/balanceatlas
**Language**: ALL in English — code, content, commit messages, branch names, PR descriptions, GitHub issue templates, CLAUDE.md, and memory files. No exceptions.

## Stack
- Astro 5.18 + Tailwind CSS v4 + Motion + Lenis
- Content: Markdown + YAML frontmatter (Astro Content Collections)
- Hosted on GitHub Pages
- Typography: Sora (headings) + DM Sans (body)
- Design: Photography-forward, coral/orange + navy + teal palette
- Plugin: `frontend-design@claude-code-plugins` — follow its anti-AI-slop rules

## Content Conventions
- Coaches: one `.md` per coach in `src/content/coaches/`, slug = filename
- Retreats: one `.md` per retreat in `src/content/retreats/`, filename pattern: `YYYY-location-coach.md`
- Photos: `public/images/retreats/<event-slug>/` (numbered 01.jpg, 02.jpg...)
- Videos: `public/videos/retreats/<event-slug>/` (numbered 01.mp4, 02.mp4...)
- Coach photos: `public/images/coaches/<coach-slug>.jpg`
- Venue photos: `public/images/retreats/<descriptive-name>.jpg`

## Media Hosting (Cloudflare R2)
- **Bucket**: `media-handstand` on Cloudflare R2
- **Public URL**: `https://media.balanceatlas.com` (custom domain on R2)
- **Credentials**: `.env` file (see `.env.example`), never committed
- `public/images/` and `public/videos/` are **gitignored** — media lives only on R2
- **`MEDIA_URL` pattern** (`src/config.ts`): prod builds use R2 public URL, dev uses `''` (empty, serves from local `public/`)
- Content `.md` files keep relative paths (e.g. `/images/retreats/...`) — templates add `MEDIA_URL` prefix at render time

### Scripts
- `scripts/upload_r2.py` — uploads `public/images/` + `public/videos/` to R2 (skips existing, `--force` to re-upload, `--dry-run`)
- `scripts/download_r2.py` — downloads all media from R2 to `public/` (for fresh clones, `--force` to re-download)
- `scripts/download_gphotos.py` — reusable Google Photos album downloader
- All require: `pip install boto3 python-dotenv`

### Workflow: Adding new media
1. Download locally: `python scripts/download_gphotos.py <album-url> public/images/retreats/<slug>/`
2. **Optimize before uploading** (MANDATORY):
   - Images: `cwebp -q 82 -m 6 input.jpg -o output.webp` (replace original with .webp)
   - Videos: `ffmpeg -i input.mp4 -c:v libx264 -preset slow -crf 23 -vf scale=-2:720 -c:a aac -b:a 128k -movflags +faststart output.mp4`
   - Short clips (≤30s): CRF 23. Longer videos: CRF 26.
3. Upload to R2: `python scripts/upload_r2.py`
4. Update content `.md` with relative paths — use `.webp` extension for images (no MEDIA_URL prefix in .md files)

### Media Optimization Rules — CRITICAL
- **ALWAYS optimize media before uploading to R2.** Never upload raw camera/phone files.
- Images: WebP format, quality 82, `cwebp -q 82 -m 6`
- Videos: H.264 720p, CRF 23-26, AAC 128k, `faststart` flag for streaming
- This is non-negotiable — raw video from phones can be 50 Mbps, web needs 2-4 Mbps

### Image Handling
- **Format**: WebP (.webp) — all images converted from JPEG/PNG
- **Manifests**: each event photo dir has a `manifest.json` with filenames, dimensions, sizes, and content hashes (md5) for dedup
- **Max items per event**: ~100 photos + videos combined (Paulo's preference)
- **Cache-Control**: `public, max-age=31536000, immutable` set on all R2 objects (upload script applies automatically)
- **TODO**: implement dedup check using manifest hashes when adding photos from new albums

## Sourcing Images for Retreats/Coaches
When a retreat or coach doesn't have photos, search in this order:
1. **Event registration page** — look for og:image meta tag or hero images
2. **Venue website** — look for location/facility photos
3. **Coach's personal website** — about page or portfolio
4. **Coach's Instagram** — profile picture (use WebFetch on instagram.com/username)
5. **YouTube thumbnails** — for video-based sources, use `https://img.youtube.com/vi/VIDEO_ID/maxresdefault.jpg`
Always verify downloaded images are real (>5KB, not HTML redirects).

## Photo Selection Guidelines
- **Hero images**: prefer ACTION shots (handstands, movement, training) over scenery-only. Scenery is OK as fallback, but people doing handstands is always better.
- **Gallery balance**: mix action shots (handstands, training), social moments (group photos, meals), stretching/warmups, and scenery. Don't front-load stretching-only photos — lead with handstands.
- **Ordering**: when selecting gallery photos from a large album, don't just pick the first N photos sequentially. Scan the full album and curate a balanced mix.

## Design Reference
- Alun.com.br v2 home page patterns: ScrollReveal, full-bleed photos, dramatic typography, staggered reveals
- Details in `memory/alun-v2-design.md`

## Key Patterns
- Retreat detail pages: **showcase** layout (past + gallery) vs **simple** layout (upcoming)
- Bento grid for photo galleries
- Horizontal carousel with snap scrolling
- Scroll-triggered animations via Intersection Observer (`data-animate`)
- Video autoplay: `autoplay muted loop playsinline` for short clips (≤20s)

## Tone
- Community-driven, NOT salesy
- We're documenting and sharing, not pushing or selling
- Warm, inclusive language — "hand balancers" not "students/clients"

## Content Writing Rules — CRITICAL
- **NEVER write descriptions using LLM-generated text.** All descriptions of retreats, coaches, venues, etc. must be quotes or excerpts from real sources (official websites, event pages, social media bios).
- Always include a visible source link (small, in a corner or footer of the section) so the reader knows where the text came from.
- If no source text is available for a coach/retreat, leave the description empty or use a minimal factual stub (name, location, dates) — do NOT invent prose.

## Link Quality Rules — CRITICAL
- **ALWAYS verify links exist before using them.** Do not guess Instagram handles or URLs — verify with a real fetch/search.
- **Retreats**: the Source link should point to the MOST SPECIFIC page possible — the retreat/event page itself, not just the coach's Instagram. Priority: event registration page > event info page > venue event listing > coach website events page > coach Instagram. Only use Instagram as last resort.
- **Coaches**: the Source link should point to their official about/bio page when available, not a generic homepage.
- **Instagram handles**: verify the handle is correct (not a 404). When unsure, search for the person and confirm the exact handle.
- **Never assume** a handle follows a pattern (e.g., `firstname.lastname.topic`). Always verify.

## FAQ Page
- The site has a `/faq` page explaining the community, how to contribute, etc.
- Must include a question: "I'd like to remove a photo or my retreat/intensive, how do I do that?" → Answer: "Just send an email to paulo.silveira@gmail.com"
- Contact email for removals: paulo.silveira@gmail.com

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/peas)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
