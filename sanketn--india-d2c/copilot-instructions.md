## india-d2c

> Find launchable D2C product ideas for the Indian market, tuned to the user's taste, capital ceiling, distribution skills, and category preferences. Use when the user asks for D2C product ideas in India, wants to brainstorm consumer brand opportunities in India, wants to validate an Indian D2C idea, or wants to change their preferences (categories, capital, distribution channels, hard excludes) mid-conversation. Requires a one-time setup (~3 minutes) the first time it runs.


# D2C Idea Finder

Runs a multi-stage agentic pipeline that surfaces 5 launchable Indian D2C product ideas, ranked and tagged (passed/flagged) against the user's taste profile. Five collectors pull signals: Indian marketplaces (Amazon.in, Flipkart, Nykaa, Myntra, Lenskart, FirstCry, and ~12 other specialist sites including quick commerce), per-category exploration queries (Reddit subs like r/IndianTech, r/AsianBeauty, r/IndianFood — plus YouTube reviews and Quora for categories where those communities are strong), 4 rotating US D2C trade publications (Modern Retail, Retail Brew, The Fascination, Exploding Topics), cross-platform Reddit + Amazon US for arbitrage signal, and Google Trends for US-vs-India interest gaps. Instagram is used only weakly in competitor enrichment (not in collection).

## How this skill works

The skill has three entry points:

1. **First-time setup** — captures the user's taste profile (categories, distribution channels, hard excludes, capital ceiling, Exa + Anthropic API keys). Run only once per user.
2. **Find ideas** — runs the full pipeline and returns 5 ranked ideas as markdown (some may be flagged with reasons like "over budget" or "competition too high" — flagged ideas still show, just below the passing ones). Takes 12-18 minutes (signal enrichment and idea generation are the dominant stages; competitor enrichment runs Exa searches in parallel with a concurrency cap).
3. **Update profile** — when the user wants to change capital ceiling, categories, excludes, or distribution channels mid-conversation, applies the change to their profile.

The skill folder is at `~/.claude/skills/india-d2c/`. User data (profile, API key, ratings, state DB) lives in `<skill-folder>/user_data/` and is gitignored.

## When to invoke

**Invoke this skill when the user:**
- Asks "find me d2c ideas in India," "what should I build in Indian D2C," "any consumer brand opportunities in India," "d2c product ideas for India" or similar
- Says "skip pet forever," "bump my capital to 15L," "I'm great at paid ads" (these are profile changes — see Step 3b)
- Asks to "set up" or "configure" the India D2C idea finder

**Do NOT invoke for** general taste comments like "I like #3" or "the wellness angle is interesting" — just acknowledge those conversationally and ask if they want it translated into a profile change.

**Do not invoke for:**
- General market research that isn't about launching a D2C brand
- SaaS or B2B idea generation (different skill)
- Country-specific D2C outside India (this skill is India-focused)

## Step 1 — Check for first-time setup (MANDATORY)

**You MUST run this Bash check before doing ANYTHING else when the skill is invoked.** Do not skip it. Do not assume setup is needed without verifying. Do not tell the user "you don't have settings yet" without first running this command and getting `SETUP_NEEDED`:

```bash
test -f ~/.claude/skills/india-d2c/user_data/profile.yaml && echo "SETUP_DONE" || echo "SETUP_NEEDED"
```

**Read the output literally:**
- Output is `SETUP_DONE` → profile exists, skip to Step 3 (Find ideas) OR Step 3b (Update profile / show settings) depending on what the user asked for. Do NOT trigger onboarding.
- Output is `SETUP_NEEDED` → profile.yaml is missing, walk the user through Step 2 (onboarding).

If you skip this check and improvise an answer based on assumption, you'll incorrectly tell returning users that their settings are gone — which they aren't. This is a confidence-failure pattern we've seen and need to prevent.

### Slash-command note

The skill is invoked when the user's intent matches the skill description (e.g. "find me d2c ideas", "show me my d2c settings"). There is **no `/india-d2c` slash command** for users to type. Do not tell users to "run `/india-d2c`" — that command doesn't exist. If they want to trigger setup explicitly, the right phrasing is *"set up the india-d2c skill"* or *"configure my d2c idea finder"*.

## Step 2 — Setup conversation (first run only)

Tell the user: *"This is your first time running the India D2C Idea Finder. Quick 3-minute setup to tune it to your taste, then we'll find ideas."*

**IMPORTANT — Do NOT use the AskUserQuestion tool for any onboarding question.** Several of these questions have more than 4 options, which AskUserQuestion does not support, and it will throw a visible error. Always present questions as numbered chat text and ask the user to reply with numbers (e.g. "1, 3, 5") or names (e.g. "beauty, food, fitness"). Map their reply back to canonical keys yourself.

Then ask these questions, **one at a time, in order**. Wait for each answer before moving on.

### Q1. Categories of interest

> Which D2C categories are you interested in exploring? Pick as many as you like — recommend 2-5 for good idea diversity. I'll only surface ideas in these.

Present these 16 options:
- **Beauty** — Makeup, skincare, haircare, fragrance
- **Personal Care** — Bath/body, oral, hygiene, shaving, deodorant
- **Food & CPG** — Snacks, beverages, condiments (non-functional)
- **Home** — Cleaning, laundry, kitchen tools, decor, candles
- **Pet** — Food, accessories, grooming
- **Wellness** — Sleep, stress, recovery, topicals, ingestible wellness
- **Sexual Wellness** — Intimacy products, hormonal, performance (premium)
- **Innerwear & Loungewear** — Innerwear, loungewear, socks, tees
- **Fashion & Apparel** — Women's, men's, kids' fashion-forward
- **Footwear** — Sneakers, sandals, formal, athleisure
- **Jewellery & Watches** — Fashion jewellery, fine, smartwatches
- **Eyewear** — Sunglasses, prescription frames
- **Accessories** — Bags, travel gear, small leather, tech accessories
- **Consumer Electronics** — Audio, wearables, smart home, tech gadgets
- **Baby & Kids** — Clothing, feeding, toys, care
- **Fitness & Nutrition** — Gear, supplements, protein, functional foods, gummies

After the user answers, validate:

- **If they pick 0 categories**, error — they have to pick at least 1.
- **If they pick exactly 1 category**, warn before proceeding:
  > "Heads up — with 1 category, all 5 ideas will come from that single space. You'll still get 5 distinct ideas, but they'll all explore different angles within [picked category] rather than spreading across different consumer categories. The system works best with 2-5 categories so it can triangulate across angles. Want to add another category or two, or proceed with just [picked category]?"
  Wait for confirmation. If they want to add, take additions and continue.
- **If 2 or more**, continue normally.

### Q2. Distribution channels

> Which marketing/distribution channels are you comfortable executing on? Pick any number — most D2C launches use 2-4 channels.

Present the 10 options numbered 1-10, ordered by how commonly Indian D2C founders use them:

1. **Paid digital ads** — Meta, Google, YouTube ads
2. **Organic social content** — Instagram, YouTube, Reels
3. **Influencer marketing** — Instagram/YouTube creators
4. **Marketplace PPC** — Amazon, Flipkart, Nykaa sponsored placements
5. **Email + WhatsApp + SMS** — Klaviyo, WhatsApp Business, SMS
6. **SEO / content marketing** — Blog, organic search, Reddit/Quora answers
7. **Referral / virality** — Refer-a-friend, viral mechanics, word of mouth
8. **Quick commerce** — Getting listed on Blinkit, Zepto, Instamart, BBNow
9. **Affiliates / coupon sites** — Performance partners and cashback networks
10. **PR / press** — Earned media, press placements

After the user answers, validate:

- **If they pick 0 channels**, warn before saving:
  > "With 0 channels, every idea will score low on distribution fit (the system has no channels to score against). Pick at least 1 — even your weakest available channel helps. Most founders default to 'Organic social content' and 'Paid digital ads'. Want to add some?"
  Wait for them to add at least one before continuing.
- **If they pick exactly 1 channel**, soft warning before saving:
  > "With just 1 channel, every generated idea will get scored against the same single capability — so expect homogeneous output (same channel called out as the fit in all 5 ideas). Most launches use 2-4 channels. Want to add another, or proceed with just [picked channel]?"
  Wait for confirmation. If they want to add, take additions and continue.
- **If 2 or more**, continue normally.

### Q3. Hard excludes

> Anything that should be ruled out entirely? Reply with the numbers of any that apply (e.g. "1, 3, 5") or names. You can also add free-text excludes if you have others not on this list — just type them out.

Present these 5 options numbered 1-5:

1. **Cold chain / perishables** — products needing refrigerated logistics
2. **Complex formulation** — novel actives, R&D-heavy, requires specialist chemistry
3. **Tight regulation** — FSSAI nutraceutical cat 8, AYUSH, baby formula, medical devices
4. **Items over 10kg** — shipping cost eats into margin, returns and damage risk rise sharply
5. **Anything ingestible** — food, supplements, drinks, anything consumed

If they add free-text excludes (e.g. "no kids products" or "no pet category" — anything not in the 5 above), capture those strings in the JSON's `hard_excludes_custom` field. The 5 preset picks above go in `hard_excludes`.

### Q4. Capital ceiling

> What's the max capital you'd want to deploy to launch a first SKU? (Many bootstrap founders use ₹10-20 lakh; tighter is fine too, but under ₹10L makes most categories hard to launch.)

After the user answers, check the value before proceeding:

- **If they enter under ₹8 lakh**, warn them clearly before saving:
  > "Heads up — ₹X lakh is below the typical D2C launch floor in India. Most categories need ₹10-15L minimum to cover first-batch manufacturing, packaging, brand assets, and ~2 months of initial marketing. With ₹X, most ideas the system surfaces will likely get flagged as over-budget. You can still proceed, but consider whether you have access to additional capital or a co-founder, or whether you'd like to set a higher ceiling now.
  >
  > Proceed with ₹X, or change?"
  Wait for confirmation. If they want to change, ask for the new number.

- **If they enter ₹8-10 lakh**, lighter warning:
  > "₹X lakh is on the tight end. Some categories — formulation-heavy beauty, consumer electronics, fashion — will still flag as over-budget. Workable for simpler white-label launches in categories like candles, basic skincare, supplements, or accessories. Proceed?"
  Continue after acknowledgment.

- **If ₹10 lakh or more**, continue normally without commentary.

Once confirmed, use the value (whatever number they finally land on) for the JSON config.

### Writing the config

Once Q1-Q4 are answered, call:

```bash
cd ~/.claude/skills/india-d2c
python3 scripts/setup.py --config '<JSON>'
```

Where `<JSON>` is a single-line JSON object with these keys:

```json
{
  "categories": ["<from Q1>", "<...>"],
  "distribution_channels": ["<from Q2>", "<...>"],
  "hard_excludes": ["<from Q3>"],
  "hard_excludes_custom": ["<from Q3 free-text, can be empty>"],
  "capital_ceiling_lakh": <number from Q4 — DO NOT default to any specific value, use what the user said>
}
```

**IMPORTANT for capital_ceiling_lakh:** use the EXACT number the user provided in Q4. Do not substitute a default. If the user said "5 lakh" → use 5. If they said "12" → use 12. The user's specific answer is the only correct value here.

Note: channels are picked as a simple binary list — either available or not. There is no "superpower" or "comfortable" tier distinction. If the user later wants to add or drop a channel, see Step 3b.

API keys are NOT included in this JSON. Setup writes `.env` with empty
placeholders, and the user fills them in next (see "Adding API keys" below).

Use these canonical keys (lowercase, snake_case):

**Categories:** `beauty`, `personal_care`, `food_cpg`, `home`, `pet`, `wellness`, `sexual_wellness`, `innerwear_loungewear`, `fashion_apparel`, `footwear`, `jewellery_watches`, `eyewear`, `accessories`, `consumer_electronics`, `baby_kids`, `fitness_nutrition`

**Distribution channels:** `paid_digital_ads`, `organic_social`, `influencer`, `marketplace_ppc`, `direct_email_whatsapp_sms`, `seo_content`, `referral_virality`, `quick_commerce`, `affiliate_coupon`, `pr_press`

**Hard excludes:** `cold_chain`, `complex_formulation`, `tight_regulation`, `over_10kg`, `ingestible` (plus anything in `hard_excludes_custom`)

`setup.py` will:
1. Auto-install `uv` if missing, create a venv in the skill folder, install dependencies
2. Write `user_data/profile.yaml` and `user_data/.env` (with empty key placeholders). Exploration queries are now generated fresh per run from profile.yaml + a rotating theme — no cached query files.
3. Initialize an empty SQLite state DB at `user_data/state.db`
4. Print a confirmation summary

After setup completes, proceed to Step 2b (adding API keys), then Step 3.

## Step 2b — Adding API keys

After `setup.py` runs, the user needs to add two API keys. Tell the user:

> Setup is done. Two more things before I can run:
>
> **Anthropic API key** — powers signal enrichment, idea generation, evaluation, and competitor enrichment. **~$1-2 per run** (scales with signal volume; generates 5 ideas and surfaces all of them with passed/flagged tags). NOT covered by Claude Pro/Max subscription — bills separately from https://console.anthropic.com.
>
> **Exa API key** — powers all the web search in the skill. Free tier (1,000 requests/month) covers ~14-16 runs. Get one at https://exa.ai.
>
> Two options for adding them:
>
> 1. **Paste them here in chat** and I'll add them to the keys file for you. Faster.
> 2. **Open `~/.claude/skills/india-d2c/user_data/.env` in your text editor** and fill in the empty values. More private (keys never appear in this chat). Tell me when done.
>
> Which would you like to do?

### If the user pastes keys in chat

Call `set_key.py` with whichever keys they provide. Examples:

```bash
cd ~/.claude/skills/india-d2c
./venv/bin/python scripts/set_key.py --exa <exa-key> --anthropic <anthropic-key>
```

The script validates Anthropic keys start with `sk-ant-` and updates `.env` in place. Run it once with both keys, or twice if the user provides them separately.

### If the user edits `.env` manually

Just wait for them to say "done" or similar. Then verify both keys are present:

```bash
grep -E "^(EXA|ANTHROPIC)_API_KEY=.+" ~/.claude/skills/india-d2c/user_data/.env | wc -l
```

If the count is 2, both keys are filled. If not, ask the user to check the file. Do NOT print or read the key values aloud — only confirm presence/absence.

Once both keys are confirmed present, proceed to Step 3.

## Step 3 — Find ideas

**Before running, tell the user:**

> Running the pipeline now. This takes **12-18 minutes depending on how many signals get collected** (the slowest stages are signal enrichment and idea generation, which scale with signal volume). I'll stream progress as each stage or substep completes. Cadence varies — sometimes every 20-30 seconds during enrichment and competitor mapping, then 5-9 minutes of quiet during idea generation while Sonnet runs web searches and writes the full idea JSON. Quiet periods are normal, not stuck.

### Streaming progress — use Monitor, not ScheduleWakeup or manual polling

The pipeline runs 12-18 minutes. To give the user live updates, use the `Monitor` tool — it's push-based, waking you on each new stdout line from the background shell. **Do NOT use `ScheduleWakeup` (wrong tool — it's for /loop mode, intervals are too long). Do NOT rely on manual polling (you'll forget).**

`Monitor` is a deferred tool. Load it first:

```
ToolSearch with query "select:Monitor"
```

Then run the pipeline in background and immediately attach Monitor:

```bash
cd ~/.claude/skills/india-d2c
./venv/bin/python scripts/find_ideas.py
```
with `run_in_background: true`.

**Checkpoint behaviour:** if a previous run failed mid-pipeline, find_ideas.py automatically detects a recent checkpoint (≤24 hours old) and skips the slow collection + enrichment stages — picking up from idea generation. You'll see `Resuming from checkpoint: N enriched signals` early in the output. This is normal and saves ~10 minutes.

If the user explicitly asks to "force a fresh run", "ignore the checkpoint", or "use new signals", pass the `--fresh` flag:

```bash
./venv/bin/python scripts/find_ideas.py --fresh
```

This deletes any existing checkpoint and re-runs the full pipeline from scratch. After every successful run, the checkpoint is auto-deleted so the next intentional run is fresh.

Take the shell ID from that Bash response, then invoke `Monitor` on it. Each stdout line from `find_ideas.py` becomes a notification you'll receive automatically.

**What to relay to the user:**
ONLY relay lines that match one of these structured status patterns:

- Lines starting with `[N/6]` — stage transitions (most important)
- Lines starting with four spaces and a sub-step name — e.g. `    amazon_us: 7 signals`, `    enrichment: batch 3/7`
- Lines starting with `✓` — completion marks
- Lines starting with `Done in` or `DONE:` — final pipeline result
- Lines starting with `Read: /` — the path to the final output file
- Lines starting with `ERROR:` — hard failures

**What to silently skip (do NOT relay):**

- Any line that doesn't match the patterns above
- Logger output starting with timestamps (`2026-05-25 17:08:53 [india-d2c]...`)
- Any line about Google Trends rate-limiting / 429 errors (this is expected, ignore)
- Stack traces, warnings about tool retries, JSON parse retries
- ANY placeholder or filler text you might be tempted to emit when nothing new has happened — if no new structured status line has arrived, stay silent

**Critical: do NOT emit placeholder updates between stage transitions.** If the pipeline is mid-enrichment and you've already relayed `[2/6] Enriching 27 signals with Claude Sonnet...`, stay silent until the NEXT structured line arrives (e.g. `    enrichment: batch 4/7` or `[3/6] Generating ideas...`). Do not emit "still working" type messages, do not output truncated tokens, do not add commentary. Silence is correct between real updates.

Examples of lines you'll see and SHOULD relay:

- `[1/6] Collecting signals (Amazon US complaints, US D2C publications, India marketplaces, Google Trends, exploration)...`
- `    amazon_us: 7 signals`
- `    india_marketplaces: 23 signals`
- `[2/6] Enriching 27 signals with Claude Sonnet...`
- `[3/6] Generating ideas from 27 signals...`
- `[6/6] Writing ideas to user_data/latest_ideas.md...`
- `Done in 612.4s. 27 signals -> 5 ideas -> 4 passed filters.`
- `Read: /Users/.../latest_ideas.md`

Examples of lines you'll see and should NOT relay:

- `2026-05-25 17:06:18 [india-d2c] WARNING: [google_trends] error for 'shampoo bar': 429`
- `2026-05-25 17:08:53 [india-d2c] INFO: [enrichment] tool: web_search(['query'])`
- Any partial / truncated text

**As each notification arrives, relay the line to the user.** Don't wait until the pipeline finishes — surface progress live.

**Critical: relay EVERY substep line individually, even when multiple arrive in quick succession.** Each substep (lines starting with 4 spaces — e.g. `    Indian marketplaces: 23 signals`, `    enrichment: batch 3/7`) is a meaningful user-visible event. Do NOT batch or consolidate them. During the collection stage especially, the user needs to see per-source signal counts (Indian marketplaces, Adaptive discovery, US D2C publications, US Amazon + Reddit, Google Trends) as each completes — that's how they know which sources are productive.

When the shell exits (Monitor will signal this), read `user_data/latest_ideas.md` and present its FULL contents verbatim to the user.

### Verbatim relay — what this means concretely

**DO:**
- Read the file with the Read tool
- Output every line of the file content into the chat, with zero modifications
- Include EVERY idea card with EVERY section that appears in the markdown file: `Problem`, `Target consumer`, `Why now`, `Hero product`, `Unit economics`, `Sourcing`, `Go-to-market`, `Wedge / why it wins`, `Indian competitors`, `Reference brands (global)`, `Sub-scores`. Plus the diagnostic block at the end if any ideas are flagged.

**DO NOT:**
- Skip sections to make output shorter (this is the failure mode we've seen — Claude in chat drops `Indian competitors`, `Sourcing`, `Why now` etc. when relaying)
- Summarise or paraphrase any section ("the wedge here is X" instead of the actual wedge text)
- Truncate competitor lists, leaving "...and others" instead of the full list
- Reorder sections
- Add commentary like "this idea looks great" or "I'd skip this one" before/between/after ideas
- Drop sections marked as "—" or empty — render them as-is

**Self-check before sending your reply:**
1. Count the `## ` headings in the file you Read (one per idea). Your reply should have the same count of `## ` headings.
2. Count `**Indian competitors**` sections in the file. Your reply should have the same count.
3. Count `**Reference brands (global)**` sections in the file. Your reply should have the same count.
4. If any of these counts don't match, you've dropped content — go back and emit the missing sections before sending.

The file is typically ~300-500 lines for 5 ideas. Yes that's a long reply. Send it anyway. The user can scroll. Treating brevity as a virtue here costs the user real information they paid for (each idea is the output of ~$0.30 of LLM work + 12-18 min of pipeline time).

**The rendered file already ends with a "What's next?" section** (file path reminder, how to revisit past runs, how to change profile, how to re-run). When you output it verbatim, that section comes along naturally — you do NOT need to add anything extra after it. Don't append your own commentary or summary after the file content.

### Fallback if Monitor unavailable

If `ToolSearch` can't load `Monitor` (some Claude Code surfaces don't expose it), fall back to:
- Run `find_ideas.py` in background
- Every ~60 seconds, use the `Read` tool on `user_data/progress.txt` — it always contains the latest single status line (the Python script writes here at every stage)
- Also occasionally check `BashOutput` on the shell to detect crashes
- Stop polling when `progress.txt` shows `DONE:` OR the shell reports `status: completed`

**Do not use `ScheduleWakeup` in either path** — it's the wrong tool for this and won't fire fast enough.

## Step 3c — When the pipeline errors

If `find_ideas.py` emits an `ERROR:` line, crashes, or finishes with suspicious output (0 signals, 0 ideas, all ideas flagged), relay the error verbatim to the user, then match it to one of the known patterns below. **Do not invent fixes** — if the error doesn't match, say so and ask the user to file an issue.

### Known errors and the real fix

| Symptom | Real fix |
|---|---|
| `ANTHROPIC_API_KEY` missing / 401 from Anthropic | Re-run `./venv/bin/python scripts/set_key.py --anthropic <key>`. Tell the user Claude Pro/Max does NOT cover this — they need a key from https://console.anthropic.com. |
| `EXA_API_KEY` missing / 401 from Exa | Re-run `./venv/bin/python scripts/set_key.py --exa <key>`. Free tier at https://exa.ai. |
| Anthropic 429 (rate limit) | Wait 1-2 minutes and re-run. Checkpoint will skip already-done stages. No config knob fixes this. |
| Exa 429 / quota exhausted | Free tier is ~14-16 runs/month (1,000 requests). Wait until reset or upgrade Exa plan. No code fix. |
| `0 signals collected` | Almost always a network or API key issue. Verify keys with `grep -E "^(EXA\|ANTHROPIC)_API_KEY=.+" user_data/.env \| wc -l` (should be 2). If keys are present, just re-run. |
| `0 ideas generated` from N signals | Idea agent JSON parse failure or timeout. Re-run — checkpoint skips enrichment, so retry costs ~3 min not 15. |
| Pipeline crashes mid-run | Re-run normally. Checkpoint resumes from the last completed stage. If it crashes at the same stage twice, ask the user to file an issue with the stderr output. |
| All 5 ideas flagged (eval_status: rejected) | Not a bug — it's a profile/market mismatch. Look at the `eval_reasons` in `latest_ideas.md`. If every idea is "needs more capital," suggest `update_profile.py --capital <higher>`. If every idea is "no channel fit," ask which channels the user actually has. **Do NOT suggest editing scoring_weights or filters** — those are intentionally not user-facing. |
| Google Trends 429 spam in logs | Expected and ignorable. Pytrends rate-limits aggressively. Other collectors carry the run. Do not surface this to the user. |

### What NOT to suggest

Hard rules — if you catch yourself about to say any of these, stop:

- **Do not invent profile.yaml fields.** The only fields in profile.yaml are: `market.capital_ceiling_lakhs`, `categories`, `distribution`, `hard_excludes` (preset + custom). Internal pipeline config (scoring weights, filters, AOV floor, model IDs, generation knobs) lives in `scripts/utils/config.py` and `scripts/utils/models.py` — physically not in profile.yaml, so there's nothing to "edit" there.
- **Do not suggest editing scripts/** to fix a runtime error. That's a bug — file an issue.
- **Do not propose "increase timeout," "lower concurrency," "switch models"** as fixes. None of these are user-exposed knobs.
- **Do not hand-edit `user_data/` files** to "patch" a bad state. If state is corrupt, delete the offending file (`checkpoint.json`, `latest_ideas.json`) and re-run.

### When you genuinely don't recognize the error

Say so plainly: *"I'm seeing an error I don't have a fix mapping for: `<exact text>`. This looks like a skill bug rather than a config issue. Can you file it at https://github.com/sanketn/india-d2c/issues with the last ~20 lines of output? In the meantime, you can try `--fresh` to rule out checkpoint corruption."*

Do not guess. Do not propose speculative profile edits. Saying "I don't know" is correct.

## Step 3d — Surfacing past runs

Every run is auto-archived to `user_data/runs/<YYYY-MM-DD>_<HHMMSS>_ideas.md` (plus matching `.json`). The user can ask about them in chat.

### CRITICAL — only ONE place counts as "past runs"

**The only valid source for past india-d2c runs is `~/.claude/skills/india-d2c/user_data/runs/`.** Nothing else.

Do NOT look at, read, or quote from:
- `~/Documents/claude-projects-main/d2c-idea-finder/` (the user's separate personal cron-job project — different schema, different sheet, unrelated to this skill)
- Any Google Sheets, SQLite databases, or CSVs the user might have in other projects
- Any logs, archives, or run-history from anywhere else on disk

If the user has other D2C-related data on their machine, it is NOT "past runs of this skill." It belongs to a different system.

If `~/.claude/skills/india-d2c/user_data/runs/` is empty (no runs archived yet), say so honestly: *"No past runs archived yet. Run archives started landing after [date]; runs before that didn't get saved. Run `find_ideas.py` and the next run will be archived."*

### What columns the skill actually has

The archived files are markdown + JSON. The skill does NOT track:
- ❌ "Signals" count column
- ❌ "Generated vs Written" distinction (the skill always shows all 5 ideas; some are tagged flagged but none are dropped)
- ❌ Any "sheet" or "sheets" concept
- ❌ Any per-run database with rows

The skill DOES track per archived run:
- ✅ Timestamp (in filename)
- ✅ Pass/flagged count (parseable from the markdown header: e.g. "3 passed · 2 flagged")
- ✅ Run duration (in markdown header: e.g. "ran in 707.2s")
- ✅ Top idea title and category (from the JSON snapshot)

If you need to show a table, use ONLY those real columns. Do NOT invent columns from other systems' schemas.

### Handler patterns

| User says | Action |
|---|---|
| "Show me past runs" / "what runs do I have" | List `~/.claude/skills/india-d2c/user_data/runs/` directory contents. For each, parse the timestamp from the filename and (optionally) Read the JSON snapshot to get top idea title. Show as a simple list: `2026-05-26 09:10 — 5 ideas in consumer_electronics (3 passed, 2 flagged)`. Do NOT show invented columns. |
| "Show me last week's run" / "the run from [date]" | Find the matching file in `~/.claude/skills/india-d2c/user_data/runs/` and Read it, output verbatim |
| "Compare this run to my last one" | Read both `latest_ideas.md` and the most recent file in `~/.claude/skills/india-d2c/user_data/runs/` other than today's, then highlight differences in idea titles / categories / pass-flag counts |
| "Show me run #N" (counting from newest) | List `~/.claude/skills/india-d2c/user_data/runs/` sorted by filename descending, Read the Nth file |

If the user references a run that doesn't exist (no file matches), say so clearly: *"I don't see a run from [date] in the skill's archive. Your archived runs are: \<list\>."* Do NOT invent runs and do NOT fall back to looking in other folders.

`latest_ideas.md` (in `user_data/`, not in `runs/`) always points to the most recent run. The archive in `runs/` keeps every past run for as long as the user keeps the folder.

### Follow-up commands after the list is shown

After listing past runs, the user often wants to drill in. Handle these patterns explicitly.

**Numbering convention:** whenever you list runs, number them with **1 = most recent**. References like "#N", "the third one", "run 2" refer to that numbering — NOT some intrinsic run-ID, NOT chronological-ascending.

| User says (after list shown) | Action |
|---|---|
| "Show me #3" / "the third one" / "open #3" | Read the 3rd file in the list you displayed (1 = newest). Output the markdown verbatim. |
| "Show me yesterday's" / "last night's" | Find run(s) in `runs/` matching yesterday's date (`YYYY-MM-DD` prefix). If exactly one, Read it. If multiple, list them with times and ask which. If none, say "no run yesterday — the most recent run was [date]." |
| "Show me [date]" (e.g. "May 20", "2026-05-20") | Find run(s) matching that date. Single match → Read. Multiple → list with timestamps, ask which. Zero → "No run on [date]. Closest runs: [list 2-3 closest]." |
| "Show me from last week" | Find runs in the last 7 days. If 1, Read it. If multiple, list them; do not auto-pick. |
| "Compare this run to last week's" | If exactly one run in last 7 days (other than today), use that as comparison. If multiple, list them and ask "which one?". Comparison should be a short diff: idea titles + categories + pass/flag counts, not full re-output of both files. |
| "Compare run #2 and #4" | Read both, output a side-by-side diff of titles + categories + pass/flag counts. Don't paste both full files. |
| "What was the top idea in #3?" | Read the JSON snapshot for that run (`<timestamp>_ideas.json`), output just the title + tagline + category of the first idea. Don't paste the full markdown. |
| "Re-read the latest" / "show me the latest again" | Output `latest_ideas.md` verbatim (same as Step 3 verbatim relay). |

### What NOT to do for past-run queries

- **Do NOT do content search across runs** (e.g. "which run had earbuds ideas"). That would require reading every markdown file (slow, expensive). If the user asks, say honestly: *"I don't have a fast way to search across all past runs. If you remember the rough date, I can pull that specific run. Otherwise you'd grep `user_data/runs/*.md` yourself."*
- **Do NOT delete or rename archived files** without explicit confirmation. Even "clean up old runs" should trigger a list + confirmation pattern, not silent deletion.
- **Do NOT invent runs** when the user references something that doesn't exist. Always cross-check the filesystem.
- **Do NOT use chronological-ascending numbering** when the user references "#N" (e.g. don't interpret "#3" as "the 3rd run ever archived"). 1 = newest, always.
- **Do NOT auto-pick** when a date reference matches multiple runs. Always ask.

## Step 3b — Updating the user's profile mid-conversation

If the user asks to change their capital, categories, hard excludes, or distribution channels at any point AFTER initial onboarding (most commonly after seeing flagged ideas), use `scripts/update_profile.py` instead of re-running setup.

### CRITICAL — Three hard rules

These rules apply to ALL profile-change interactions. Violating any of them produces the bad UX we've seen (hallucinated category names, leaked internal terms like "weight" and "includes", missing numbered lists).

**Rule 1: ALWAYS read `user_data/profile.yaml` first** before showing the user their "current" settings or listing what's available. Never recall categories/channels/excludes from memory — read the file. Same goes for capital ceiling.

**Rule 2: ALWAYS show numbered lists with human-readable labels** — never raw snake_case keys, never canonical keys without their human label. The canonical key lists below are for YOUR mapping work; the user sees the human labels.

**Rule 3: NEVER expose profile.yaml internals** — do not mention `weight`, `includes`, `preset`, `custom`, `1.0`, `0.0`, `score: 2`, or any other internal config term in user-facing messages. The user knows nothing about these and should never need to.

**Rule 4: When the user asks "what are my settings" (or any variant), ALWAYS run `update_profile.py --show` and output its result.** Do NOT compose the answer by Reading profile.yaml directly and improvising — that leaks internal fields the user shouldn't see. The script is the single source of truth for what counts as "settings."

The ONLY user-editable settings are:
- Capital ceiling (`market.capital_ceiling_lakhs`)
- Picked categories
- Picked distribution channels
- Hard excludes (preset + custom)

All other config (AOV floor, gross margin minimum, scoring weights, filter thresholds, model IDs, max ideas per run) lives in `scripts/utils/config.py` and `scripts/utils/models.py` — NOT in profile.yaml. So when the user asks "what are my settings," there's literally nothing internal to leak from profile.yaml. If a user asks "can I change my min AOV?" — say honestly: *"That's not a user-changeable setting in this skill. AOV floor is fixed at ₹800 (the D2C unit-economics floor). If you genuinely need a different value, you'd need to edit `scripts/utils/config.py`."*

### Canonical key lists (use for mapping, never show to user)

**16 categories** (`canonical_key` → Human label):
- `beauty` → Beauty
- `personal_care` → Personal Care
- `food_cpg` → Food & CPG
- `home` → Home
- `pet` → Pet
- `wellness` → Wellness
- `sexual_wellness` → Sexual Wellness
- `innerwear_loungewear` → Innerwear & Loungewear
- `fashion_apparel` → Fashion & Apparel
- `footwear` → Footwear
- `jewellery_watches` → Jewellery & Watches
- `eyewear` → Eyewear
- `accessories` → Accessories
- `consumer_electronics` → Consumer Electronics
- `baby_kids` → Baby & Kids
- `fitness_nutrition` → Fitness & Nutrition

**10 distribution channels** (`canonical_key` → Human label):
- `paid_digital_ads` → Paid digital ads
- `organic_social` → Organic social content
- `influencer` → Influencer marketing
- `marketplace_ppc` → Marketplace PPC
- `direct_email_whatsapp_sms` → Email + WhatsApp + SMS
- `seo_content` → SEO / content marketing
- `referral_virality` → Referral / virality
- `quick_commerce` → Quick commerce
- `affiliate_coupon` → Affiliates / coupon sites
- `pr_press` → PR / press

**5 hard excludes** (`canonical_key` → Human label):
- `cold_chain` → Cold chain / perishables
- `complex_formulation` → Complex formulation
- `tight_regulation` → Tight regulation
- `over_10kg` → Items over 10kg
- `ingestible` → Anything ingestible

### Flow A — User is SPECIFIC (named the change directly)

If the user names the exact change ("add fashion and footwear", "bump capital to 15L", "drop pet"), map to canonical keys and run the command directly. No need to ask anything.

| User says | Run |
|---|---|
| "Bump my capital to 10 lakh" | `./venv/bin/python scripts/update_profile.py --capital 10` |
| "Increase my budget to 15L" | `./venv/bin/python scripts/update_profile.py --capital 15` |
| "Add fashion and footwear to categories" | `./venv/bin/python scripts/update_profile.py --add-category fashion_apparel --add-category footwear` |
| "Drop pet from my categories" | `./venv/bin/python scripts/update_profile.py --remove-category pet` |
| "Remove tight regulation from excludes" | `./venv/bin/python scripts/update_profile.py --remove-exclude tight_regulation` |
| "Add over-10kg to my excludes" | `./venv/bin/python scripts/update_profile.py --add-exclude over_10kg` |
| "Add paid ads as a channel I can use" | `./venv/bin/python scripts/update_profile.py --add-channel paid_digital_ads` |
| "I can't do PR, drop that" | `./venv/bin/python scripts/update_profile.py --remove-channel pr_press` |
| "Show me my current settings" | `./venv/bin/python scripts/update_profile.py --show` |

**Casual phrase mapping** (when user uses informal names):
- "fashion" → `fashion_apparel`
- "food" → `food_cpg`
- "personal care" → `personal_care`
- "innerwear" / "loungewear" / "innerwear & loungewear" → `innerwear_loungewear`
- "wellness" → `wellness` (NOT `wellness_non_ingestible` — that key doesn't exist)
- "electronics" / "consumer electronics" / "tech" → `consumer_electronics`
- "watches" / "jewellery" / "jewelry" → `jewellery_watches`
- "baby" / "kids" / "baby and kids" → `baby_kids`
- "fitness" / "nutrition" / "supplements" → `fitness_nutrition`

If you can't map a user's word to one of the 16 canonical category keys, say so honestly: "I don't have a category called '[name]'. The 16 categories are: [show numbered list]. Which one did you mean?" Do NOT invent a new category key.

### Flow B — User is VAGUE ("I want to add categories", "change my channels", etc.)

When the user expresses intent without naming specifics, run an interactive prompt — same format as onboarding.

**For each of the 4 mutable fields**, the flow is:
1. Read `user_data/profile.yaml` to see current state
2. Tell user what they currently have (in human labels)
3. Show a numbered list of what they could add/change (in human labels)
4. Wait for reply (numbers like "1, 3, 5" or names)
5. Map reply to canonical keys and run `update_profile.py`

### Unified format for B.1, B.2, B.3

All three of these flows use the SAME pattern as onboarding Q1/Q2/Q3:
- Read `user_data/profile.yaml` first to know current state.
- Show ALL canonical items in a numbered list with brief descriptions (same descriptions as onboarding).
- Mark already-picked items with `✓` at the end of the line.
- Tell the user they can say "add 3, 5" or "drop 2" or just name what they want.
- Map their reply to canonical keys and run the right command.

The point: mid-conversation changes should LOOK and FEEL like onboarding — same visual format, same numbered options, same descriptions. The user shouldn't have to learn a different UI just because they're updating an existing profile.

#### B.1 — "I want to add/change/remove categories"

1. Read `user_data/profile.yaml`. Note which categories have `weight > 0` (picked).
2. Show the **same numbered list as onboarding Q1** — all 16 categories with their descriptions. Append `✓ (picked)` to ones the user already has.
3. Tell the user:
   > "Tell me what to add or drop — e.g. 'add 4, 7' or 'drop wellness'. You can mix numbers and names."
4. Parse the reply: numbers/names that are NEW → add. Items they ask to drop → remove. Run `update_profile.py --add-category X` and/or `--remove-category Y` accordingly.

#### B.2 — "I want to change my distribution channels"

1. Read `user_data/profile.yaml`. For each of the 10 channels, check if it's available (score > 0) or not (score 0).
2. Show the **same numbered list as onboarding Q2** — all 10 channels with their descriptions:

   ```
   1. **Paid digital ads** — Meta, Google, YouTube ads
   2. **Organic social content** — Instagram, YouTube, Reels
   3. **Influencer marketing** — Instagram/YouTube creators
   4. **Marketplace PPC** — Amazon, Flipkart, Nykaa sponsored placements
   5. **Email + WhatsApp + SMS** — Klaviyo, WhatsApp Business, SMS
   6. **SEO / content marketing** — Blog, organic search, Reddit/Quora answers
   7. **Referral / virality** — Refer-a-friend, viral mechanics, word of mouth
   8. **Quick commerce** — Getting listed on Blinkit, Zepto, Instamart, BBNow
   9. **Affiliates / coupon sites** — Performance partners and cashback networks
   10. **PR / press** — Earned media, press placements
   ```

   Append `✓ (you have this)` to lines for channels with score > 0.
3. Tell the user:
   > "Tell me what to add or drop — e.g. 'add 4, 5' or 'drop PR'. You can mix numbers and names."
4. Parse the reply, run `update_profile.py --add-channel X` and/or `--remove-channel Y` accordingly.

#### B.3 — "I want to change my hard excludes"

1. Read `user_data/profile.yaml`. Note preset excludes + any custom excludes.
2. Show the **same numbered list as onboarding Q3** — all 5 preset excludes with descriptions:

   ```
   1. **Cold chain / perishables** — products needing refrigerated logistics
   2. **Complex formulation** — novel actives, R&D-heavy, requires specialist chemistry
   3. **Tight regulation** — FSSAI nutraceutical cat 8, AYUSH, baby formula, medical devices
   4. **Items over 10kg** — shipping cost eats into margin, returns and damage risk rise sharply
   5. **Anything ingestible** — food, supplements, drinks, anything consumed
   ```

   Append `✓ (excluded)` to lines for excludes already on. If the user has custom excludes (anything beyond these 5), list those after the numbered list as: *"Plus custom excludes: [list]"*.
3. Tell the user:
   > "Tell me what to add or drop — e.g. 'add 3' or 'drop tight regulation'. You can also add a custom one — just type it."
4. Parse reply: numbers/names that map to canonical → `--add-exclude X` / `--remove-exclude Y`. Custom free-text → also `--add-exclude` (the script accepts arbitrary strings as custom excludes).

#### B.4 — "I want to bump/change my capital"

1. Read profile.yaml `market.capital_ceiling_lakhs`.
2. Tell user:
   > "Your current capital ceiling is ₹[X] lakh. What would you like to change it to?"
3. Wait for a number. Apply same validation as onboarding:
   - <8L: strong warning, ask to confirm or change
   - 8-10L: lighter warning, confirm
   - 10L+: continue normally
4. Run `update_profile.py --capital N`.

### After any change

After running the script, confirm to the user what changed (the script prints a summary). Then ask in plain English if they want to find new ideas with the updated settings. Do NOT say "re-run find_ideas.py" or "use update_profile.py" in your reply to the user — those are internal command names. Say things like *"want to find new ideas now?"* or *"want me to run the pipeline with the updated categories?"*

**Never tell the user to "edit profile.yaml"** — they shouldn't need to know that file exists. All changes happen through your commands above; the user only ever speaks plain English.

## Step 4 — Handle user reactions

When the user reacts to ideas, there are only two cases:

### Case A — Reaction maps to a specific profile change

If the user's reaction implies a concrete `profile.yaml` edit, call `update_profile.py` directly (per Step 3b). Examples:

| User says | Action |
|---|---|
| "Skip pet forever" | `update_profile.py --remove-category pet` |
| "I want fashion ideas too" | `update_profile.py --add-category fashion_apparel` |
| "Drop tight regulation" | `update_profile.py --remove-exclude tight_regulation` |
| "Bump my capital to 15L" | `update_profile.py --capital 15` |
| "Add paid ads as a channel" | `update_profile.py --add-channel paid_digital_ads` |
| "Drop influencer marketing" | `update_profile.py --remove-channel influencer` |

After applying, confirm the change to the user in plain English (e.g. *"Added paid digital ads to your channels."*) and ask if they want to find new ideas with the updated settings. Do NOT phrase it as "re-run find_ideas.py" in your reply to the user — that's the internal command name, not user-facing language.

### Case B — Reaction is just taste commentary

If the user says something like "I like #3," "the wellness angle is interesting," or "more like the underarm serum one" — **acknowledge it in chat and ask if they want it translated into a concrete profile change.** Be honest about whether their reaction actually maps to anything tunable.

Example where it DOES map cleanly:

> User: "Skip pet forever, those ideas didn't land."
> You: "Got it. I'll drop pet from your categories — that's `update_profile.py --remove-category pet`. After that, the next run won't surface pet ideas. Want me to do it?"

Example where it does NOT map cleanly (be honest):

> User: "I really liked #2's premium positioning."
> You: "Brand positioning (premium / playful / scientific / etc.) is something Sonnet picks per idea — there's no profile knob that says 'always go premium.' What I can change: bump your capital ceiling (premium products usually need ₹15-20L+ to launch well), or add/remove categories that lean premium vs mass. Want either of those, or shall we leave the profile as-is?"

**Do NOT log anywhere or claim to remember.** There is no feedback log, no learning loop, no hidden state. The user's reaction lives in this chat conversation only. If they want a durable effect, it has to land as an explicit `update_profile.py` change.

**Important: the skill does NOT have a black-box learning system.** If you (Claude in chat) catch yourself saying "the system learned X about you" or "I'll remember that for next time" — stop. That's not how this skill works. Cross-session memory only exists in `profile.yaml`, and only changes when the user explicitly approves an edit.

## Notes on behaviour

- **Background mode + status file polling is required for find_ideas.py.** Pipeline takes 12-18 minutes. Read `user_data/progress.txt` every ~30 seconds while the shell is alive; relay each new status line. Stop when you see `DONE:` in the status file or the shell exits.
- **Present ideas verbatim.** When the pipeline completes, output the full `user_data/latest_ideas.md` to chat without summarising or excerpting. The file is formatted for direct chat display.
- **Never modify `user_data/` files directly** — always go through `setup.py` or `update_profile.py`.
- **The `.env` file contains the Exa key in plaintext.** It is gitignored. Do not commit it, do not paste it into chat, do not write it to other locations.
- **Setup is idempotent** — if the user re-runs setup, it overwrites the profile and reseeds learned context.
- **NEVER cite the user's settings from memory or from this SKILL.md.** When the user asks about their current capital ceiling, picked categories, distribution channels, or hard excludes — OR when you're suggesting changes ("bump from X to Y") — ALWAYS Read `user_data/profile.yaml` first using the Read tool. The numbers and lists in THIS file (SKILL.md) are examples and placeholders, NOT the user's actual values. The user's profile.yaml is the single source of truth.
- **Do NOT confidently state a specific user setting without having just read it.** If you don't know whether the user's capital ceiling is X or Y, read the profile. Saying "currently ₹20L" when you haven't verified is a hallucination — treat it as a bug, never do it. Reading the profile is cheap and always available.

---
> Source: [sanketn/india-d2c](https://github.com/sanketn/india-d2c) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
