## bakers-agent

> An AI-powered marketing automation platform for **My Baking Creations**, a family-owned Bay Area bakery (founded 2012, Daly City CA).

# Bakers Agent — Project Instructions

## What This Is
An AI-powered marketing automation platform for **My Baking Creations**, a family-owned Bay Area bakery (founded 2012, Daly City CA).
Two repos, one business:
- **This repo (`bakers-agent`):** ~41 Cloud Functions (GCP), Pub/Sub pipeline, React dashboard at bakers-agent.web.app
- **Website repo (`gugosf114/gugosf114.github.io`):** Static site at mybakingcreations.com (GitHub Pages), ~800 commits

## The Website (mybakingcreations.com)
- **Repo:** github.com/gugosf114/gugosf114.github.io
- **Stack:** Static HTML/CSS/JS, GitHub Pages, custom domain
- **Pages:** Home, About, Gallery (cakes/cookies/cake pops/cupcakes + sub-galleries: sculpted, realistic, wedding, hand-piped, printed), Corporate, Blog, Party Rentals, Contact, Order Form, Buy Now, city landing pages
- **SEO:** JSON-LD structured data (Bakery, FAQPage, LocalBusiness, AggregateRating), Open Graph, Twitter Cards, CSP headers
- **Analytics:** GA4 (G-KB96GDJ011)
- **Features:** Hero carousel, Instagram video feed, corporate logo grid, live activity feed widget, AI chatbot, consultation/party rentals widget, site search with typewriter effect, nav particles canvas
- **Fonts:** Fredoka One (headings) + Nunito (body)
- **Theme color:** #EC268F (pink)
- **Corporate clients:** Google, Meta, Microsoft, Salesforce, DocuSign, OpenAI, Instagram, PayPal, Gap, Alaska Airlines, Warriors, Twitch, Cooley, Stripe, Levi's, Kaiser
- **corporate.html:** Title/H1 leads with cakes, not cookies (updated 2026-03-05)
- **robots.txt:** Wix migration slugs (/services-5, /contact-3, /about-1, /gallery-7) are Disallowed to stop crawl budget starvation
- **Social:** Instagram, Facebook, Pinterest, Yelp
- **Contact:** (415) 568-8060, info@mybakingcreations.com, 1096 Wildwood Ave, Daly City, CA 94015
- **Footer:** Credits Anthropic Claude for AI features

## Architecture
- **Cloud Functions (GCP):** ~41 functions, Gen2, Python 3.11+ (deploy uses 3.12)
- **Pub/Sub pipeline:** State machine with fan-out, DLQ (`bakers-agent-dlq`), memory/learning loop
- **Shared library:** `shared/bakers_shared/` — 27 modules, synced to each function at deploy via `tools/sync_shared.py`
- **Dashboard UI:** React 18 + Tailwind + shadcn/ui + Recharts + Firebase, hosted on Firebase Hosting
- **Database:** Firestore (database: `bakers-agent-db`)
- **Smart Router:** `content-router-v1` uses Gemini REST API (NOT the SDK) to decide platforms and adapt captions
- **Service accounts:** `bakers-brain-sa` (decision-making), `bakers-poster-sa` (posting)
- **Autonomy system:** 3 levels — AUTONOMOUS, SUPERVISED, RECOMMEND — per track via `get_track_autonomy()`

## Key Shared Modules (`shared/bakers_shared/`)
- **Core clients:** `firestore.py` (singleton via `client()`, uses `FIRESTORE_DB`), `pubsubutil.py` (`get_publisher()`), `secrets.py` (`access_secret_text()` with `lru_cache`), `pubsub.py` (`Publisher` + `decode_pubsub()`)
- **LLM:** `llm.py` — REST API only, exports `GeminiConfig`, `generate_text()`, `generate_json()`, `generate_markdown_from_brief()`
- **Content:** `sentinel.py` (validation: word count, banned words, price detection), `poster_base.py` (multichannel posting pattern), `firestore_store.py` (delegates to `firestore.client()`)
- **External APIs:** `github.py`, `gbp.py`, `klaviyo.py`, `yelp_profiles.py`, `mcp_client.py`
- **Utilities:** `logging.py` (JSON for Cloud Logging), `rate_limit.py`, `hashing.py`, `ids.py`, `textutil.py`, `timeutil.py`, `events.py`, `http.py`, `config.py`, `env.py`, `autonomy.py`

## Pub/Sub Topics
- **SEO:** `seo-ingest-topic`, `seo-opportunities-topic`, `seo-brief-topic`, `seo-pr-topic`, `seo-feedback-topic`
- **Content delivery:** `content-router-topic` → fan-out to `content-post-{gbp,facebook,instagram,pinterest,apple,yelp}-topic`
- **Agent Dun:** `dun-scheduler-topic`, `dun-digest-topic`, `dun-curiosity-topic`, `dun-pope-topic`, `dun-seo-bridge-topic`
- **Analytics/Ads:** `analytics-snapshot-trigger`, `analytics-alerts-topic`, `ads-daily-scheduler-trigger`, `ads-jobs-topic`
- **Reviews:** `gbp-reviews-topic`, `gbp-review-responder-topic`
- **Other:** `ai-visibility-topic`, `brand-consistency-topic`, `commerce-feeds-topic`, `directory-feeds-topic`, `llms-txt-topic`, `attribution-topic`, `segment-rebuild-topic`, `gsc-jobs-topic`, `local-seo-feeder-topic`
- **DLQ:** `bakers-agent-dlq`

## Firestore Collections
- `dun_config/`, `dun_jobs/` — Agent Dun config and unified job model
- `seo_jobs/`, `seo_sources/` — SEO pipeline
- `ai_visibility/`, `pope_visibility_log/` — AI search monitoring + Pope agent
- `curiosity_log/`, `memory/` — Learning loop (confidence-weighted insights)
- `analytics_snapshots/` — GA4 daily snapshots
- `brand_consistency/` — NAP audit results
- `google_ads/` — Ads campaign data
- `gbp_reviews/` — Google Business Profile reviews
- `captions/` — Social media captions (read-only for UI)
- `airlock/` — Human-in-the-loop approval queue
- `dun_ui_triggers/` — UI-initiated agent triggers

## Environment Variables
- **Core:** `GCP_PROJECT`, `FIRESTORE_DB`, `SITE_URL`, `SITE_NAME`
- **Feature toggles:** `DISABLE_INGEST`, `DISABLE_POSTING`, `DISABLE_PR_CREATION`
- **Gemini:** `GEMINI_MODEL` (default: `gemini-2.5-flash`), `GEMINI_TEMPERATURE`, `GEMINI_API_KEY`
- **Secrets from Secret Manager:** `GSC_SECRET_NAME`, `GEMINI_SECRET_NAME`, `OPENAI_SECRET_NAME`, `PERPLEXITY_SECRET_NAME`
- **Validation:** `config.py` rejects placeholders (`CHANGEME`, `REPLACE_ME`, `YOUR_`, `INSERT_`, `TODO`)

## Tools (`tools/`)
- `sync_shared.py` — Copy shared lib into each function dir before deploy
- `deploy.sh` — Main deployment orchestrator with registry of 46+ functions
- `check_ai_jobs.py` — Debug AI visibility job types
- `seed_mcp_registry.py`, `seed_pope_memory.py`, `seed_ai_visibility_queries.py` — Seeding scripts
- `gbp_oauth_flow.py`, `gbp_exchange_code.py` — Google Business Profile OAuth helpers

## Deploy
- **Cloud Functions:** `gcloud functions deploy <name> --gen2 --runtime python311 --source ./<function-dir> --env-vars-file ./<function-dir>/.env.yaml`
- **Full deploy:** `tools/deploy.sh` (handles shared sync + all functions)
- **Shared lib sync:** `python tools/sync_shared.py`
- **Dashboard UI:** `cd ui && npm run build` then `npx firebase-tools deploy --only hosting`
- **Firestore rules:** `npx firebase-tools deploy --only firestore:rules`
- **Website:** Push to `gugosf114/gugosf114.github.io` main branch (GitHub Pages auto-deploys)

## Testing
- Tests in `tests/` — use `importlib.util.spec_from_file_location()` to import individual function `main.py` files with unique module names
- Shared fixtures in `conftest.py`: `gcp_mocks` (pre-built mock dict), `load_function(fn_dir, module_name, extra_modules={})`
- GCP deps must be mocked via `mock.patch.dict(sys.modules, ...)` BEFORE importing
- Must mock: `google.cloud.firestore`, `functions_framework`, `bakers_shared.firestore`, `bakers_shared.pubsubutil`, `bakers_shared.secrets`
- Run: `python -m pytest tests/ -v`

## Critical Lessons (Do Not Repeat These Mistakes)
- **Gemini SDK poisons gRPC:** `google-generativeai` SDK's `genai.configure(api_key=...)` corrupts global gRPC auth, breaking Firestore. Always use REST API (`requests.post` to `generativelanguage.googleapis.com`) instead.
- **Firestore `== None`:** Only matches explicit null, NOT missing fields. Always filter client-side.
- **`contextlib.suppress(BaseException)`:** Catches KeyboardInterrupt/SystemExit. Never use it; use `Exception`.
- **Always log in except blocks:** Never `except: pass`, especially in DLQ functions.
- **Secret Manager trailing newlines:** Always `.strip()` API keys from Secret Manager.
- **CloudEvents format (Gen2):** Data is at `event.data["message"]["data"]`, not `event["data"]`.
- **Apple Maps:** JS SPA, can't be HTTP-scraped.
- **MagicMock auto-attrs:** Use `mock.MagicMock(spec=[])` to prevent auto-attribute creation (critical for CloudEvent tests where `get_json` auto-attr causes wrong code path).
- **Patch targets:** Patch on the *module* object (`seo_pr.decode_pubsub`), not on `sys.modules["bakers_shared.pubsub"]` — `from X import Y` binds locally.
- **ui-trigger-v1 result truncation:** Set to 8000 chars (was 1000). Do not shrink it — agent results like brand_consistency analysis JSON exceed 1000 easily.

## Dashboard UI Stack
- React 18, TypeScript, Vite, Tailwind CSS 3
- shadcn/ui (Radix primitives), Lucide icons, Recharts
- Firebase Auth (anonymous), Firestore real-time subscriptions
- Dark theme with warm amber/gold palette (primary: HSL 38 92% 50%)
- Fonts: DM Sans (body), JetBrains Mono (mono)
- Olympic Loader animation for loading states (runner, swimmer, cyclist)
- `AgentResultPanel` — subscribes to source Firestore collections per agent (brand_consistency, ai_visibility, seo_jobs, pope_visibility_log); renders score ring, per-platform bars, issues list, appearances inline below tile grid
- `AgentTiles` — selectedAgent state drives accent border + AgentResultPanel display

## Key Design Docs
- `AGENT_DUN_CONTEXT.md` — Unified job model, track registry, Phase 0 scope
- `docs/THE_POPE.md` — Meta-agent orchestrator that deploys sub-agents
- `docs/CURIOSITY_LOOP_DESIGN.md` — Self-questioning insight generation
- `README_MULTICHANNEL.md` — Multi-channel content delivery schema

## Verified Working (as of 2026-03-21)

### Functions confirmed against real APIs
| Function | Status | Notes |
|----------|--------|-------|
| `brand-consistency-v1` | **LIVE** | Checks website + 2 GBP locations via real OAuth. Yelp needs Fusion API key. |
| `ai-visibility-v1` | **LIVE** | ChatGPT (Responses API + web search), Gemini (Google Search grounding), Claude (Anthropic API + web search). Perplexity removed (key expired). |
| `bakers-agent-v1` | **LIVE** | GCS Eventarc trigger. Gemini 2.5 Flash generates captions as plain text (not JSON schema — structured output unreliable on Vertex AI). Sentinel validates in Python. |
| `content-router-v1` | **LIVE** | Smart routing via Gemini. Fans out to GBP + Facebook. |
| `poster-gbp-v1` | **LIVE** | GBP localPosts API v4 works. Bucket `bakers-agent-input-bakers-agent` is public (required for GBP media fetch). |
| `poster-facebook-v1` | **LIVE** | Graph API v21.0. Token refreshed 2026-03-21. Page ID: `117991029605684`. Listens on `content-post-facebook-topic`. |

### Credentials Status
| Secret | Status | Notes |
|--------|--------|-------|
| `gbp-oauth-secret` | **Working** | Refresh token valid. Both location IDs confirmed. |
| `fb-page-access-token` | **Working** | Permanent page token (version 7, 2026-04-01). Never expires. App creds: `fb-app-id`, `fb-app-secret`. |
| `gemini-api-key` | **Working** | Used for caption gen, smart routing, AI visibility. |
| `openai-api-key` | **Working** | Responses API with `web_search_preview` tool. |
| `anthropic-api-key` | **Working** | Claude with `web_search_20250305` tool. |
| `perplexity-api-key` | **Expired** | 401 error. Replaced by Claude in AI visibility. |
| `pinterest-access-token` | **Pending** | Standard API upgrade submitted 2026-04-01. OAuth flow built (`pinterest-oauth-v1`). App creds: `pinterest-app-id`, `pinterest-app-secret`. |
| `gsc-oauth-secret` | **Working** | Verified 2026-04-01. Token refresh OK, GSC API returns real query data for mybakingcreations.com. |
| `github-pat` | **Working** | Verified 2026-04-01. Push + admin access to gugosf114/gugosf114.github.io confirmed. |

### Critical Fixes Applied This Session
- **bakers-agent-v1**: Gemini structured output (`responseMimeType: "application/json"`) does NOT work reliably on Vertex AI with `gemini-2.5-flash`. Switched to plain text generation + Python-side validation (word count, price detection via regex). Do NOT revert to JSON schema mode.
- **ai-visibility-v1**: OpenAI Chat Completions API without tools returns training-data-only results (MBC not found). Must use Responses API with `web_search_preview` to match what real ChatGPT users see. Same for Gemini — must include `tools: [{"google_search": {}}]` in request body.
- **poster-facebook-v1**: Media array contains plain URL strings, not dicts. Adapter must handle `isinstance(first, str)` before calling `.get("url")`.
- **poster-facebook-v1**: Was deployed on wrong Pub/Sub topic (`poster-facebook-topic`). Must be `content-post-facebook-topic` to match router's `CHANNEL_TOPICS_JSON`.
- **AgentResultPanel.tsx**: `onSnapshot` subscription must await `authReady` before connecting — otherwise Firestore security rules reject the query and `loading` state hangs forever.
- **GCS bucket**: `bakers-agent-input-bakers-agent` must be publicly readable (`roles/storage.objectViewer` for `allUsers`) — GBP API fetches images via HTTP URL.

### Dashboard UI
- **Theme**: Lavrentiy industrial — dark instrument panels, nixie tube readouts, orange/copper accents
- **Sidebar**: Dark rack, resizable (drag right edge), quick stats (live from Firestore), platform LEDs, upload button
- **Boot sequence**: "AGENT DUN ONLINE" power-on animation on page load
- **Guide**: Interactive operations manual via sidebar button (11 sections)
- **Fonts**: Outfit (body), JetBrains Mono (mono), Nixie One (readouts)

## Known Bugs (Not Yet Fixed)
- **AgentResultPanel "Running..." label:** Driven by `dun_ui_triggers` collection. Old trigger docs stuck in PENDING status cause permanent "Running..." display even after agent completes. Need cleanup or TTL.

## Resolved Bugs
- **Pipeline Health 100% bug:** Resolved — old pipeline health tile replaced by 4-tile layout in dashboard rewrite (commit `9278049`).
- **Sidebar stats POSTS counter:** Resolved — sidebar removed in dashboard rewrite (commit `9278049`).
- **Reverted commit eb8d834 (partial):** Two critical fixes re-applied 2026-04-01: seo-brief-v1 null guard on `job_id`, seo-pr-writer-v1 catch-all `Exception` around execute modes. Remaining items from that commit (seo-feedback GSC creds escalation, brand-consistency logging) were addressed separately in commits `9f6d88f` and `fbdf8a1`.

## Coding Style
- Python: Type hints, f-strings, structured logging via `bakers_shared.logging`
- TypeScript: Strict mode, functional components, hooks
- No over-engineering — keep it simple, direct, working
- Prefer editing existing files over creating new ones

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gugosf114) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
