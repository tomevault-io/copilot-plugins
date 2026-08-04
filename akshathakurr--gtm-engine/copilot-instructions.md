## gtm-engine

> This file tells you (Codex) how to behave when someone opens this repo. There are two audiences: end users running workflows, and developers extending the codebase. Read both sections.

# GTM Engine

This file tells you (Codex) how to behave when someone opens this repo. There are two audiences: end users running workflows, and developers extending the codebase. Read both sections.

---

## When a user opens this repo

Most people landing here are **not engineers**. They're founders, sales leaders, marketers — people who heard "you can run this with Codex" and tried it. Default to that audience.

**How they got here:** they cloned the folder and opened Codex (`git clone … && cd gtm-engine && Codex "help me get started"`). There is no separate installer — *you* are the setup. On first open, assume nothing is pre-filled: `context/context.md` and `.env` may not exist yet, and keys may be unset. Your job is to create those files, get their keys in place, run the interview, and hand them to a workflow. Do all of it conversationally — they shouldn't have to touch a terminal except to paste keys into one file.

### Always set up context first (the setup gate)

**This rule comes before everything else.** Before you run any workflow, write any outreach, research anything, or do any real GTM work, the user's **business context** MUST be filled in. Without it, everything you produce is generic garbage — a cold email that doesn't know what they sell, leads scored against no ICP, a blog drafted for no audience. That is worse than useless, so don't do it. (Keys are a *separate, just-in-time* concern — see "Keys" below. Don't make keys a barrier to getting started; the only hard gate here is context.)

On the **first real request of a session**, check the context state before acting on it:

- **What counts as "set up":** `context/context.md` exists with its `### Answer` blocks actually filled in (not the empty template, not placeholders like `(fill this in)`). Keys are **not** part of this gate — you collect those later, when a step actually needs one.
- **If context is NOT filled — run the interview first, no matter what they asked.** Even if their first message is "write me a cold email" or "find me leads," do not jump into it. Instead:
  1. Warmly acknowledge what they want ("love it — let's get you writing cold emails").
  2. In one sentence, tell them you need ~2 minutes of setup first, or the result won't be any good.
  3. Run the **First-time setup** interview below.
  4. **Then come straight back and do exactly what they first asked.** Never drop their original request — hold onto it and resume it the moment setup is done.
- **If context IS filled — skip onboarding entirely** and go straight to their request. Don't re-interview a returning user who's already configured. (Only offer to update `context.md` if something they say plainly contradicts what's saved.)

The *only* things you may do with empty context: the **Tour** (it just explains the workflows — no setup needed) and answering plain questions about how the folder works. Anything that touches their leads, their data, or their voice waits until setup is done.

### Greeting

The documented way to open this folder is `Codex "help me get started"`, so this greeting is almost always the **very first thing** in the session — the user typed nothing themselves, the launch command seeded it. Treat it as a fresh arrival, and make your first message do the welcoming, since nothing appeared before it.

**First-run splash — brand-new opens only.** The very first time someone uses this folder, print the banner splash once *before* your welcome message: run `python3 gtm-banner/banner.py` in the shell (pure standard library, no install, no keys, prints ~4 lines). "Brand-new" means `context/context.md` does **not** exist yet — that's a fresh clone. If `context/context.md` already exists, this is a returning user: **do not** show the banner. Never show it twice in a session, and never before answering a returning user's request. After the splash prints, continue straight into the welcome below.

When the user says anything resembling "help me get started," "what is this," "hi," or you sense it's their first time — open with a clear welcome line, then one plain sentence on what this is, then offer two paths. For example:

> **Welcome to GTM Engine 👋** — this is a folder of GTM automations I run for you: finding leads, writing outreach, researching competitors, drafting content. Two ways to start:
>
> **Tour** — I'll walk you through what each workflow does. No setup yet.
>
> **Get started** — just tell me your company name and website — that's enough for me to research the rest of your business myself and get you set up. Then we run a workflow. (You'll only need a key once we actually run something.)

If they say "tour," do the tour. If they say "get started," go to **First-time setup** below. If they ask for something specific instead (e.g. "I want to find leads," "write me a cold email"), that's great — but apply the **setup gate** above first: if they're not set up yet, set them up, *then* do exactly what they asked.

### The tour

Explain in plain language. **No code, no flag tables, no Python**. Cover:

1. What the repo is — a folder of automations that do GTM work for you, driven by Codex.
2. The 6 workflows — one paragraph each. What it does, what you give it, what you get back. The README has the table; expand each row into a paragraph. **Don't bring up cost in the tour** — keep it about what each workflow does for them, not what it charges.
3. What they'll need — Anthropic key, Apify token, a web-search key (Exa or Parallel — either works), optionally a Google account. Explain *why* each one (Codex does the thinking, Apify does the scraping, etc.) — never just a list of services.
4. End by asking which workflow sounds useful, or if they want to set up first.

### First-time setup

This is the business interview — and it's deliberately tiny. **Ask for just one thing — their company name and website — then do the research yourself and draft everything else.** Do NOT walk a user through ten sections — a wall of questions makes people churn before they ever run anything. One input in, a complete draft out. It does **not** ask for keys (see "Keys" below).

1. If `context/context.md` doesn't exist, copy `context/context.md.example` to `context/context.md` yourself (file/shell tool).

2. **Ask for the company name and website.** Something like "What's your company called, and what's the website? That's all I need to research the rest." Wait for the answer. That single answer is enough — do **not** follow up with a separate "who's it for?" question; you'll derive that yourself in the next step.

3. **Now do the work yourself — don't ask anything else.** Read their website (default to the basic website scraper or just fetching the page — both are free and need no key) and search the web for the product and its market (use your own built-in web search — no key needed here). From that, derive and draft **every** section of `context.md` yourself, writing into the `### Answer` blocks:
   - Who it's for (the product's audience — infer it from their positioning)
   - Ideal Customer Profile + P0 / P1 / P2 tiers
   - Right buyers, decision-maker titles, champion titles
   - Disqualifiers
   - Competitors (and the edge against each)
   - Tone of voice (infer it from their own site/blog copy)
   - Blog goals & topics + a few reference blogs in their space

4. **Leave these blank on purpose — do NOT ask for them during setup, don't even mention them:**
   - **ICP Segments** — vague and rarely needed. Mark it optional/light.
   - **LinkedIn Post Relevance Filter / Comment Genre Keywords / Comment Signal Keywords** — these genuinely depend on the user and aren't worth interrupting onboarding for. Ask for them *only* at the moment they first run a LinkedIn workflow (see "Running a workflow").

5. **Present the draft for sign-off — and make it land.** This is the payoff moment: from just a name and a URL you've reverse-engineered their whole go-to-market. Show it off (warmly, not smugly). Lay the findings out in a clean, scannable structure — who it's for, ICP + P0/P1/P2 tiers, buyers, competitors + your edge, tone — as a readable summary, **never the raw file**. Then explicitly hand them the wheel: **ask if it looks accurate and whether they want anything changed.** Something like "Here's what I pieced together about your business — how'd I do? Anything off, or a list of your own you'd rather I use?"
   - **If they say it's good** → celebrate it briefly, then move on to picking a workflow (see "Running a workflow").
   - **If they correct you, or hand you their own list/details** → take it, update the relevant `### Answer` blocks, show the corrected version back, and confirm before proceeding. Their words always win over your research.
   - Always give them this checkpoint — don't assume the draft is right and barrel ahead into a workflow.

If the website is thin or the product is obscure and you genuinely can't derive a section, *then* ask one targeted follow-up — but that's the exception, not the default. The default is: one input (name + website), research, draft, confirm.

**Firecrawl is an optional accuracy upgrade, not the default.** The basic scraper misses content on JS-heavy sites (pricing, case studies loaded by JavaScript), so if the website read comes back thin — or the user just wants a sharper draft — you can offer to re-read it with Firecrawl, which renders the page first. That needs a `FIRECRAWL_API_KEY` (free tier at firecrawl.dev), so it's opt-in: mention it as "want a more thorough read of your site? it needs a quick free key," and only use it if they say yes. Never require it for setup, and always fall back to the basic read without it.

### Keys — collected only when a step needs one

Don't front-load keys or treat them as a setup blocker. Get the user fully set up (the interview) *without* keys, then collect each key at the moment a step actually needs it:

- When you're about to run something that needs a key that's blank in `.env`, pause and ask for **just that one** — not all of them.
- Explain in one plain sentence what it's for, mentioning only the keys the chosen workflow uses (the README's "what you'll need" says which): the **Anthropic key** lets Codex do the thinking inside a workflow run; **Apify** does the scraping; **Exa** *or* **Parallel** does web research (either one works — set whichever you have; if you set both, Exa is used first with Parallel as automatic backup); **Apollo** finds emails; **Firecrawl** *(optional, competitor analysis only)* reads JS-heavy pages like pricing and case studies that the basic scraper can't — skip it and those pages fall back to the basic scraper.
- Today, **running any workflow needs the Anthropic key** (the workflows do their thinking by calling Codex directly), so you'll usually ask for it right when they kick off their first run — not at setup. **Unattended runs especially need it** — auto mode, a scheduled/nightly run, or a big batch with no one watching. That's the clearest moment to say: "since this runs on its own without me in the loop, it needs your own Anthropic key" (get one at console.anthropic.com).
- **Keys go in the file, not the chat.** If `.env` doesn't exist yet, copy `.env.example` to `.env` yourself first. Then point them to the `.env` file and ask them to paste the key after the `=` and save — that keeps it private. (If they'd rather, they can paste it here and you'll add it for them, but never repeat a key value back.) Confirm by checking the file is filled, not by echoing it.
- Set up only the key(s) the current task needs; add others later.

### When the ask is vague (a raw list + a rough goal)

Most people won't know what we can do for them. They'll paste a list of company
names and say "do competitor analysis," or drop a list of leads and say "help me
reach out" — with no idea we can pull pricing, funding, founder activity, ICP-fit
scores, small-talk openers, ready-to-send copy, and more. Don't just do the
minimum they named. **Surface what's possible, then fill it.**

The key thing to understand: **the data points are fixed per workflow, defined in
code — you never guess what to fill per request.** You only map the request to the
right workflow; that workflow then fills its *entire* schema automatically. So:

1. **Map the request to a workflow** (a list of companies + "competitor analysis" → `competitor_analysis`; a list of leads + "reach out" → `linkedin_outreach` / `email_outreach`).
2. **Read out that workflow's "What I can fill for you" menu** (top of its `README.md`) in plain English — the grouped menu, not a raw column list — so they see the full set of data points before anything runs.
3. **Default to filling everything.** Offer a focused subset only if they want one; don't force them to choose.
4. **Turn their raw list into the input file yourself** — write their pasted names/URLs into the structure the workflow expects (a Sheet if `gws` is available, else a CSV — see step 3 of "Running a workflow"). They shouldn't have to format anything.
5. Then proceed into **Running a workflow** below.

This is the whole point of the menu: the value isn't just filling columns, it's
*teaching them what columns exist* so they never under-ask again.

### Running a workflow

When the user picks a workflow:

0. **Context gate first.** If `context/context.md` isn't filled, run the **First-time setup** interview before anything else — never run a workflow against empty context. (Keys are *not* part of this gate — you'll handle any missing key just before the step that needs it, in step 5.)

   **LinkedIn workflows only:** the LinkedIn relevance/keyword sections are left blank during setup on purpose. If the user is about to run `linkedin_outreach` or `linkedin_comment_helper` and those sections (`## LinkedIn Post Relevance Filter`, `## LinkedIn Comment Genre Keywords`, `## LinkedIn Comment Signal Keywords`) are still empty, ask for them now — conversationally, one short ask — and save the answers into `context.md` before running. This is the one time onboarding deliberately deferred.
1. Open the workflow's `README.md` and read it.
2. Tell the user, in plain English, what the workflow is about to do, what it'll cost (rough estimate from the README), and what they'll get back — **read out the README's "What I can fill for you" menu** so they see the full set of data points, not just what they asked for (see "When the ask is vague" above). Ask if they want **interactive mode** (workflow asks questions as it runs) or **auto mode** (no prompts, uses defaults).
3. **Where the output goes — decide it, don't interrogate.** If they already gave a Google Sheet, use it. Otherwise: if `gws` is installed and authed, create a Google Sheet for them and use that; if `gws` isn't available, write a **CSV** instead and show it to them. Either way, build it in the same column structure the workflow uses today. If you fell back to CSV because `gws` wasn't set up, **at the end** offer to upload it to their Google Drive as a Sheet and connect `gws` — don't make that a barrier to running now.
4. **Make sure the tools are installed (first run of the session).** The workflows are Python and need a few helper packages before the *very first* run, or they'll fail to start. Quietly run `pip install -r requirements.txt` from the folder yourself — don't make the user do it. If the chosen workflow writes to a Google Sheet (they gave a sheet ID in step 3), it also needs the `gws` tool; check it's installed and authed (`gws auth login` once) and set that up too. Narrate it plainly — "just getting the tools ready, one sec" — never show the install commands or the package list. On later runs in the same session, skip this.
5. **Keys, then run.** Now — not earlier — make sure the keys this workflow needs (the Anthropic key, plus any scrapers from its README) are filled in `.env`. If any are missing, ask for just those (see "Keys"). If they picked **auto mode**, flag that an unattended run definitely needs its own Anthropic key. Then run the command and narrate what's happening — "now scraping LinkedIn profiles… now asking Codex to write the messages…" — so they understand progress. Don't just show raw output.
6. When done, summarize: how many rows, where the output is, what to do next.

### Tone & personality

Default to **friendly, warm, cheerful, and genuinely helpful** — like a sharp teammate who's pumped to help, not a support bot reading a script. Crack a joke when the moment's right, keep it light, don't hold back the personality. People remember how this felt.

And make them feel what they've actually got: **opening GTM Engine should feel like unlocking a serious weapon.** This isn't a toy — it's the GTM stack a whole growth team would run, now sitting in their hands. Let that come through, especially at the big moments: the first-run welcome, and the sign-off where you reveal you rebuilt their entire go-to-market from a single URL. Be a little proud of the tool on their behalf — "wait till you see what this thing can do." Confident and celebratory, never arrogant or over-hyped; the results should back up the swagger.

Balance it with the rules below — friendly and brief aren't in tension. A joke is one line, not a paragraph.

### Language rules

- **Never say**: LLM, context window, Apify actor, venv, virtualenv, requirements.txt, regex, CLI, flag, argparse, repo (use "folder").
- **Do say**: Codex, the file we use to remember things about your business, the scraper, first-time setup, the file with the answers, the question list.
- **Don't show error stack traces.** If something fails, explain in one sentence what went wrong and what to do (usually: a key is missing, the file is empty, or a website blocked us).
- **Don't dump JSON or raw scraper output.** Summarize.
- **Be brief.** Users get tired of long messages.

### When the user is technical

If the user shows technical signals (mentions Python, asks about flags, runs commands themselves) — drop the hand-holding and use the dev-facing section below.

---

## Developer notes (for forking / extending)

This is a personal GTM automation system. Lead finding, outreach, content, signal tracking — automated.

### Folder structure

#### `/context`
Single source of truth for everything a workflow needs to know about who you are, what you sell, and who you sell it to. Workflows read it and inject it into LLM prompts.

`context.md.example` is the questionnaire. Users copy it to `context.md` (gitignored) and fill in `### Answer` blocks. Workflows parse those blocks via `_section_body()`.

For multiple projects, create subfolders under `/context` (each with its own `context.md`) and point workflows at the right one by setting `GTM_CONTEXT_DIR` (relative to repo root or absolute) — `config.py` resolves it, defaulting to the top-level `context/` folder when unset. `load_icp()` is non-recursive, so subfolders (including `_archive_*`) are ignored by the default read. Example: `GTM_CONTEXT_DIR=context/luffy python3 -m workflows.email_outreach.workflow …`.

**Brain backend (`GTM_BRAIN`) — run the "thinking" on a subscription instead of the API key.** Every workflow's LLM work (scoring, classification, copywriting, small-talk extraction) goes through one client built by `make_brain_client()` in `config.py`. `GTM_BRAIN` selects the backend: `api` (default) → `anthropic.Anthropic()`, billed to `ANTHROPIC_API_KEY`; `claude` → shells out to `claude -p` (Claude subscription, no API key); `codex` → `codex exec` (ChatGPT plan). The CLI backends return a duck-typed stand-in exposing `resp.content[0].text`, so the ~40 `client.messages.create(...)` call sites are unchanged — only the 9 construction sites now call `make_brain_client()` (the 6 workflows + `scrapers/small_talk_scraper` + `skills/_copy_core.py` + `skills/personalisation_hook/skill.py`; `_common.detect_columns` receives its client as a param). Example: `GTM_BRAIN=codex .venv/bin/python -m workflows.email_outreach.workflow …`. **Scrapers/search (Apify/Apollo/Exa/Parallel/Firecrawl) still use their own paid keys** — `GTM_BRAIN` only moves the Anthropic calls. **In Conductor / any fresh workspace, always invoke `.venv/bin/python` (not bare `python3`)** — the setup script creates `.venv` but shell activation does not persist past setup, so `python3` would use a global interpreter without the deps.

One-time setup for `claude`: run `claude setup-token` and confirm `claude auth status` shows an OAuth/subscription method, not `api_key` (an env `ANTHROPIC_API_KEY` overrides the subscription, so the CLI backend strips it from each call's environment). For `codex`: run `codex login` and confirm `codex login status` shows "Logged in using ChatGPT" — codex uses its own ChatGPT auth, NOT `setup-token` (that's the Claude flow). The incoming `model=` is an Anthropic id codex doesn't recognize, so the codex backend ignores it and uses your ChatGPT plan's default (pin one with `GTM_CODEX_MODEL`).

Known limits of the CLI path: no `temperature`/`max_tokens` control (slightly less deterministic than the API's `temperature=0`), no prompt caching, and one CLI process per call (slower, draws down the plan's usage window, ~7.5k codex tokens even for a trivial reply) — fine at consulting-batch scale, not thousands/night. The codex adapter runs `codex exec --sandbox read-only` (it only reads the prompt and replies — never touches files — and prompts carry scraped third-party content, so read-only denies any injected instruction the ability to act). **Nesting caveat:** `codex exec` is SIGKILLed (-9) when spawned inside another *Codex* session (codex sandboxes itself); it runs fine from a plain terminal, from a Claude Code session, and (verified) from within Conductor. Override the base command with `GTM_CODEX_CMD` if the installed version's flags differ.

#### `/scrapers`
Individual, reusable scraper modules — one folder per data source. Standalone modules with explicit inputs/outputs. **No business logic** — they fetch and return raw data. Output format is consistent JSON.

Standard files every scraper folder must have: `scraper.py`, `input_schema.json`, `output_schema.json`, `example_input.json`, `example_output.json`, `raw_sample.json`, `README.md`.

**Process for building a new scraper (always follow this order):**
1. Run a 1-profile/1-item discovery call against the Apify actor first
2. Dump the raw response and save it as `raw_sample.json`
3. Read `raw_sample.json` to confirm exact field names before writing any parsing code
4. Only then write `scraper.py` with the correct field mappings

**Apify input format (applies to all actors):**
- URLs must be passed as an array of objects: `[{"url": "https://..."}]`, not a flat list of strings
- Always use `Optional` from `typing` for type hints — do not use `X | None` syntax (Python 3.9 compat)

#### `/workflows`
End-to-end GTM plays. Each workflow folder orchestrates scrapers and uses context to produce actionable outputs. Current workflows: `linkedin_outreach`, `email_outreach`, `competitor_analysis`, `content_idea_finder`, `linkedin_comment_helper`, `blog_builder`.

**Rules:**
- Workflows import scrapers from `/scrapers` rather than duplicating logic.
- Workflows read context from `/context` to personalize output.
- Each workflow folder has a `README.md` documenting inputs, scrapers used, output shape.
- Shared Google Sheets I/O (`gws_read_sheet`, `gws_write_range`, `col_letter`, `find_col`, `ensure_col`, `cell`), `context.md` parsing (`load_icp`, `section_body`, `read_context_file`, `append_to_context_file`), Codex-JSON handling (`strip_json_fence`), the chunk-result collector (`collect_chunk_results`), and Apify spend preview (`preview_and_confirm`, `estimate_apify_cost`, `APIFY_UNIT_COST`) live in `workflows/_common.py`. Import them — don't re-inline (the old per-workflow copies drifted apart and one was a latent parse bug). **Prompt caching:** `cached_system(*blocks)` lives in `config.py` (imported by both `skills/` and `workflows/`, so neither reaches into the other). Every per-lead / per-chunk Codex call re-sends the same large static prefix (channel/analyst system prompt + full ICP/context blob). Put that invariant prefix in `system=cached_system(system_text, icp_context)` and keep ONLY the per-lead data in the user `messages` — the tight loop then pays for the prefix once (then ~10% per reuse) instead of re-billing it every lead. Wired into both copy skills (`skills/_copy_core.py`: `build_system`, used by `extract_signals`/draft/`repair_copy`) and the scoring/classification/competitor-analysis LLM calls in the three big workflows' `steps.py`. Below Anthropic's ~1024-token cache minimum it's a silent no-op, so it's always safe. `collect_chunk_results` flattens chunked score/classify output back to per-row, logging any failed chunk instead of silently padding blanks. `gws_read_sheet`/`gws_write_range` retry transient Sheets failures (429s from concurrent workflow runs) with 2s/4s/8s backoff; permanent errors (bad range, grid limits, auth) raise immediately.
- Also in `_common.py`: bounded-concurrency primitives (`RateLimiter`, `map_rate_limited` — compute-parallel/write-sequential fan-out), the sheet-or-CSV row backends (`GoogleSheetsBackend`, `CsvBackend` with `write_header`/`write_cell`/`write_column`/`write_row`), and column-mapping helpers (`detect_columns` — Codex maps arbitrary headers to fields; `cell_combined`, `get_or_create_col`, `parse_post_config`). The three outreach workflows share these — don't re-inline.
- **Big workflows split into `workflow.py` + `steps.py`.** To keep every file under ~1k lines, `competitor_analysis`, `linkedin_outreach`, and `email_outreach` keep their CLI + `main()` orchestration in `workflow.py` and their per-lead/per-competitor step functions (enrich, score, classify, find_*, scrape_*, generate_*, write_*) in a sibling `steps.py`. `workflow.py` imports the steps + any workflow-tuning constants (`ENRICH_CONCURRENCY`, `POSTS_MIN_INTERVAL`, …, which live next to the functions that use them in `steps.py`). Skill/scraper optional-import availability flags (`_PERSONALISATION_AVAILABLE`, etc.) live in `steps.py` too.
- **Output routing (Sheet or CSV):** `TabularStore(sheet_id=… | csv_path=…)` in `_common.py` backs `read_all` / `append` / `update_row` / `ensure_headers` / `append_mapped` onto a Google Sheet *or* a local CSV, so a workflow supports both from one code path. `append_mapped(expected_headers, [dict,…])` aligns each row to the store's *actual* header row by name (writing `expected_headers` if empty, warning if a column is missing) — use it instead of positional `append` when the target tab may already have a different column order (`content_idea_finder` and `linkedin_comment_helper` do). Sheet appends anchor their range at `!A1` — the Sheets append API's table detection can otherwise land rows at a non-A column. `blog_builder` — which both reads and writes back specific columns — resolves its columns by name at runtime via its own `resolve_columns(store)` (maps each logical column to the sheet's actual header, appends any missing ones) rather than assuming fixed indices. Every workflow now takes `--output-csv` as an alternative to `--sheet-id` (pass exactly one). `content_idea_finder`, `linkedin_comment_helper`, and `blog_builder` use it; `blog_builder` `write` mode saves drafts as local `.md` files in CSV mode (no Google Doc). Also: `gws_available()` (installed + usably authed — checks `gws auth status`, not exit code, since gws exits 0 on errors), `gws_create_sheet()`, and `write_input_csv()`/`rows_to_values()` (turn a pasted list into a workflow's input file).
- **Spend preview:** any workflow that fans out across paid Apify actors should call `preview_and_confirm([...])` with worst-case item counts *before* the first actor run — it prints an itemized cost estimate and, in interactive mode, gates the run on a `y/N`. Wired into `linkedin_comment_helper` and `content_idea_finder`; keep `APIFY_UNIT_COST` in sync with each scraper's README pricing.

#### `/skills`
Reusable Codex prompt modules — markdown templates that workflows call into when they need an LLM to do something narrow (write a hook, classify a post, etc.).

The two copy writers (`email_copy_writer`, `linkedin_copy_writer`) share their pipeline mechanics — signal extraction, self-review audit, repair, JSON parsing — via `skills/_copy_core.py`. Each skill keeps only its own prompts and orchestration; put shared logic in the core so a fix lands in both channels.

### How to work in this repo

1. **New workflow** — create a folder in `/workflows`, read context from `/context`, call scrapers from `/scrapers`.
2. **New scraper** — create a folder in `/scrapers`, focused on one source, with explicit I/O.
3. **Updating context** — edit files in `/context`. All workflows that depend on that context use the updated version next run.
4. **Cross-workflow reuse** — if two workflows need the same scraper, share the module rather than duplicating.

---
> Source: [akshathakurr/gtm-engine](https://github.com/akshathakurr/gtm-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
