## foxit-pmm-agent

> Project context and standing rules for any agent working in this repo. Read this before writing code or prose.

# Foxit PMM workflow agent

Project context and standing rules for any agent working in this repo. Read this before writing code or prose.

## What this is

A Claude Code agent plus a Next.js showcase site that runs a product marketing manager's launch workflow on Foxit's MCP Host and PDF API stack. A PMM drops a press release draft PDF into `./inbox`. The agent reads it, pulls structured launch facts, generates a one-page launch brief PDF, files a review task in Notion, sends the brief for legal sign-off over eSign, and prints a run summary. The web page frames the whole thing as a developer marketing asset Foxit could publish.

This is portfolio work for a Foxit Senior PMM interview. It has to run end to end on a laptop, be screen-recordable, and ship a public page at foxit.teddycastro.me.

## Writing rules

Every piece of prose in this repo follows these rules. That means the README, template copy, the showcase page, error messages, log summaries, commit messages, all of it. If output could have come from any AI answering any prompt, it is not good enough.

This set was consolidated from research into what actually signals machine-written prose. The goal is simple. If a sentence could have come from any model answering any prompt, rewrite it. Teddy can extend or override any of this; treat his edits as final.

### Hard constraints

These are not negotiable.

- No em dashes anywhere. Not one. Use a period for a separate thought, a comma for a continuing one, or rewrite. The em dash is the single loudest AI tell.
- Sentence-case headings. "What this is", never "What This Is".
- Prose over lists when the content is explanation. Lists are for real enumerations: config keys, ordered steps, a banned-word reference. Never break a paragraph into bullets to look tidy.
- No bolded label paragraphs. The "**Term:** explanation" pattern repeated down a page is a dead giveaway. Write the sentence instead.
- Contractions are fine and preferred. "It's", "don't", "won't". They read like a person.

### Banned vocabulary

Do not use these words. They cluster in AI output far past their natural rate.

delve, dive into, navigate (figurative), underscore, bolster, foster, harness, leverage, unpack, shed light on, pave the way, pivotal, crucial, vital, groundbreaking, cutting-edge, transformative, game-changer, game-changing, innovative, robust, comprehensive, seamless, intricate, nuanced, vibrant, multifaceted, holistic, testament, landscape (figurative), realm, embark, synergy, endeavor, enlighten, esteemed, tapestry, treasure trove, peril, amplify, beacon, convey, resonate, interplay, adhere, paramount, profound, indelible, bespoke, cognizant, encompass, hitherto, utilize (write "use"), elevate, unlock, supercharge, revolutionize, boasts, notably, ever-evolving, fast-paced.

Plain swaps: use not utilize, build not foster, use not leverage, look at or dig into not delve, key not pivotal, strong not robust, full not comprehensive.

### Banned transitions and openers

Do not start sentences or paragraphs with these, and do not use them as connective filler.

Furthermore, Moreover, Additionally, Indeed, Importantly, Notably. Also kill the stock openers: "It's important to note", "It's worth noting", "In today's [fast-paced / digital / evolving] world", "At its core", "When it comes to", "At the end of the day", "Let's break it down", "Let's explore", "Now let's turn to", "This is where X comes in".

### Banned sentence shapes

These false-insight constructions are the most recognizable pattern of all. Never use them.

- "It's not just X, it's Y"
- "Not only X, but also Y"
- "This isn't about X. It's about Y."
- "No X. No Y. Just Z."

### Structural tells to avoid

Watch for these even when the words are clean.

- The rule of three. AI defaults to three adjectives, three examples, three-item lists. Use three only when three is the real count. Otherwise use two, or four, or one.
- Uniform rhythm. Vary sentence and paragraph length on purpose. Follow a long sentence with a short one.
- Sweeping context openers and tidy summary closers. Start with the point. Stop when the point is made. Do not restate the question before answering it.
- Hedge stacking. Cut "may", "might", "could potentially", "generally speaking", "to some extent" when you mean the plain claim. Say the thing.
- "Just" as a softener. Usually delete it.

### Specificity

Replace abstractions with facts. If a draft says "significant improvement", give the number. If it says "many companies", name them. Concrete beats balanced. A real example beats a general claim every time.

## Code rules

Python uses type hints on every function. Lint with ruff. Log structured JSON, never bare print statements for anything that matters. TypeScript runs in strict mode. Secrets load from `.env` only and never get hard-coded. Every step in the agent catches its own errors so one failure never crashes the whole run. A failed step logs clearly and the run continues with what worked.

## Repo map

`agent/` holds the Python agent. `main.py` is the orchestrator that watches `./inbox`. `foxit_doc_gen.py` and `foxit_esign.py` are thin REST wrappers over the Foxit Document Generation and eSign APIs. `extractors.py` turns raw PDF text into a typed dict of launch facts. `templates/` holds the launch brief docx template.

`inbox/` is the watched drop folder. `outputs/runs/` holds one timestamped folder per run with a `log.jsonl`. `samples/` holds the demo press release. `web/` holds the Next.js showcase.

## Secrets

All secrets live in `.env`, modeled by `.env.example`. Keys: `FOXIT_CLIENT_ID`, `FOXIT_CLIENT_SECRET`, `FOXIT_API_HOST`, `LEGAL_REVIEWER_EMAIL`, `NOTION_DATABASE_ID`. Do not guess `FOXIT_API_HOST`. Confirm it against the Foxit PDF API MCP Server README.

## Build phases

The build runs in ten phases. Stop after each and wait for Teddy to verify before moving on. The full phase list lives in `PROMPT.md`.

---
> Source: [tcast1000/foxit-pmm-agent](https://github.com/tcast1000/foxit-pmm-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
