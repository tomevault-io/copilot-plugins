## openclaw-x-signal

> Read X/Twitter market intelligence, analyze crypto and broader market narratives, classify sentiment, cross-validate with market data, and generate Telegram-ready daily reports.


# X Agent Skill

Use this skill when the user asks for X/Twitter-based crypto, AI, macro, or market sentiment intelligence.

This skill is designed for read-heavy market analysis. It should not place trades. It should not post, like, reply, follow, or perform engagement actions unless the user explicitly asks and approves the exact action.

## What This Skill Does

X Agent collects and analyzes market-relevant X/Twitter content from:

- A curated X List
- X Explore Trending
- Narrative keyword searches
- High-signal searches such as whale moves, ETF flows, regulation, on-chain data, and project launches

It produces:

- Structured JSON for other agents
- Human-readable Telegram reports
- Historical keyword heat tracking
- Cross-validation against market data

## Main Commands

Run from the X Agent project directory:

```bash
cd /path/to/x-agent
```

### Full Daily Report

Use this by default for user-facing Telegram reports:

```bash
node daily-run.mjs --telegram-parts
```

This prints JSON with a `parts` array:

```json
{
  "status": "ok",
  "parts": [
    "Part 1...",
    "Part 2..."
  ],
  "savedDaily": "/path/to/daily-output.json"
}
```

Send every item in `parts` to the user, in order. Do not merge all parts into one Telegram message.

### Short Telegram Report

```bash
node daily-run.mjs --telegram
```

Use this only when the user explicitly wants a shorter report.

### Raw Scrape and Analysis

```bash
node x-agent.mjs --search
```

This performs the full X scrape and prints structured JSON.

### X List Only

```bash
node x-agent.mjs
```

Use this for quick list-only checks.

## Report Style

The default report should be detailed. Do not shorten it just to fit one Telegram message.

If the report is long, split it into parts:

- Part 1/2
- Part 2/2

or:

- Part 1/3
- Part 2/3
- Part 3/3

The report should include:

1. Market sentiment overview
2. Sentiment classification summary
3. Crypto / On-chain narrative board
4. AI / Macro / Geopolitics narrative board
5. High-frequency tokens
6. X x Market cross-validation
7. Most engaged tweets
8. Highest viewed tweets
9. Data source note
10. Saved JSON path

## Interpretation Rules

### Sentiment Labels

Each tweet may be classified as:

- `bullish`
- `bearish`
- `panic`
- `hype`
- `news`
- `shill`
- `neutral`

When explaining the report, be clear that this is a rule-based sentiment layer unless an LLM classification pass has been explicitly added.

### Narrative Boards

Separate narratives into:

```text
Crypto / On-chain
AI / Macro / Geopolitics
```

Do not let high-reach AI or macro posts bury crypto-specific signals.

### Cross-Validation Labels

Use these labels carefully:

- `early narrative`: X heat is rising, but price and volume have not moved much
- `possible pump`: X heat is rising and price or volume is surging
- `high risk`: X is hot but DEX liquidity is weak
- `risk-off`: bearish or panic X sentiment appears with price/funding stress
- `watching`: no strong confirmation yet

Never present these labels as trading instructions.

## Safety Rules

- Do not expose cookies, API keys, or tokens.
- Do not print secret file contents.
- Do not commit secrets to the repository.
- Do not claim that X sentiment predicts price with certainty.
- Do not place trades from this skill.
- Do not give financial advice.
- Do not automate posting, liking, replying, following, or unfollowing unless the user explicitly requests that action and approves the exact content.

## Expected Files

The project should contain:

```text
x-agent.mjs       Main scraper and analyzer
daily-run.mjs     Daily pipeline and Telegram formatter
package.json      Node project config
README.md         Public documentation
SKILL.md          This skill file
```

Optional legacy or helper files may include:

```text
analyzer.mjs
runner.mjs
```

## Secrets

The X session cookie should live outside the repository:

```text
~/.openclaw/secrets/x-twitter.json
```

Expected shape:

```json
{
  "x_auth_token": "YOUR_X_AUTH_TOKEN",
  "x_ct0": "YOUR_X_CT0_TOKEN"
}
```

Never send real values to the user unless the user explicitly asks to inspect their own secret file and understands the risk.

## Data Outputs

Default data directory:

```text
~/.openclaw/data/x-agent/
```

Typical outputs:

```text
latest.json
latest-daily.json
x-agent-*.json
daily-*.json
history/YYYY-MM-DD.json
```

When answering user questions, prefer summarizing these files rather than pasting raw JSON.

## When To Use This Skill

Use this skill when the user asks:

- What is X saying about crypto today?
- Summarize my X List.
- What narratives are heating up?
- Which tokens are being discussed?
- Is sentiment bullish or bearish?
- Are there panic or risk-off signals?
- Cross-check X hype with market data.
- Generate my daily X market intelligence report.

## When Not To Use This Skill

Do not use this skill for:

- Direct trade execution
- Portfolio management
- Exchange account actions
- Posting to X without explicit approval
- Financial advice

## Recommended Future Enhancements

- LLM-based sentiment classification for high-impact tweets
- Semantic narrative clustering
- Account tier classification
- Confidence scores
- Historical signal backtesting
- Alert mode for sudden narrative spikes
- Trader-agent-ready JSON schema

---
> Source: [TINZY0612/openclaw-x-signal](https://github.com/TINZY0612/openclaw-x-signal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
