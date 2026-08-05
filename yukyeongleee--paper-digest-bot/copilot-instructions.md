## paper-digest-bot

> This directory is the workdir of a cron job that sends an AI-paper digest to Slack every morning.

# Paper Digest Bot — Project Rules

This directory is the workdir of a cron job that sends an AI-paper digest to Slack every morning.
These rules are separate from SOUL.md (personality) and MUST be followed for this task.

*한국어 버전: `AGENTS.ko.md` — 한국어로 운영하려면 `cp AGENTS.ko.md .hermes.md` (`.hermes.md`가 이 파일을 오버라이드합니다).*

**Personal settings live in two files. Read them FIRST, before doing anything:**
- `profile/config.md` — target channel ID, group→emoji mapping, output language, tone guide
- `profile/interests.md` — interest groups with keywords, plus a low-interest list

(If either is missing, tell the user to copy it from `profile/*.example.md` and stop.)

## Mission

1. Run `python3 scripts/collect_reactions.py` first to collect reactions on past messages.
   - Based on the latest entries in `state/feedback.jsonl`, update `profile/interests.md`.
   - Reaction interpretation (weights):
     - ❤️ 🔥 🤩 💯 = strong interest (+2): more of this topic
     - 👍 👏 🙌 ⭐ = interest (+1)
     - 👀 🔖 = want to read (+1) — good candidate for a recap in the weekly digest
     - 👎 😴 🥱 = not interested (-1)
     - ❌ 🚫 = stop this topic (-2): move the topic toward the low-interest list
   - When updating, append one line to the "feedback history" section at the bottom of the file.
2. Gather today's paper candidates.
   - Rankings on HF Daily Papers and alphaXiv are **UTC-based**. Always use the most
     recently COMPLETED UTC day (= "yesterday" in KST terms) — never an in-progress ranking.
   - alphaXiv trending/hot list (do not rely on keyword search alone)
   - Hugging Face Daily Papers ranking for that completed day
   - X/Twitter researcher posts as discovery signals only
3. Exclude papers already sent, by checking against `state/sent-papers.jsonl`.
4. Select papers in two tracks:
   a. **Interest picks (3–5)** — match against `profile/interests.md`; the "low interest"
      section MUST be reflected. Papers mentioned by the influence accounts listed in
      `profile/config.md` get a ranking bonus — verify mentions via web search and cite
      only confirmed ones in the buzz line. Influence-mentioned papers are eligible even
      if they are NOT from that day's arXiv/ranking — recency is not required for them
      (still dedup against sent-papers.jsonl).
   b. **Exploration seeds (up to 3)** — papers ranked **top-3 on HF Daily or alphaXiv**
      are ALWAYS shared even if they fall outside the user's interests (skip any already
      chosen in 4a). Use the exploration emoji from `profile/config.md` and write the
      interest line as `_Exploration: HF Daily #N_`. Positive reactions on these are a
      signal to ADD a new topic to `interests.md` — they exist to expand the profile.
5. For each paper, write the message using the template below, extract the concept figure
   with `python3 scripts/extract_figure.py <arxiv_id>`, then send with
   `python3 scripts/post_digest.py` (payload schema in the script's docstring).
   - Channel: use the channel ID from `profile/config.md`
   - One call sends the main message + the thread (figure, details).
   - Never send via cron delivery. Sending happens ONLY through this script.
6. Using the `ts` printed by post_digest.py, record each sent paper in
   `state/sent-papers.jsonl` (one line = one paper, schema below). Never omit `ts` —
   it is required for reaction tracking.

## Weekly digest (Saturdays)

On Saturdays, run only step 1 (reactions) as usual, then instead of the daily flow send
**ONE message + thread**.
- Channel: the **weekly channel** in `profile/config.md` (not the daily channel).
- The main message is an overview only; ALL content goes into thread comments —
  minimize reader fatigue.
- Pick one of two types: Type 1 when a major conference just ended (or just announced
  awards/orals/spotlights); Type 2 otherwise.

### Common rules
- Papers already shared in a daily digest MAY reappear — mark with 👀 + a short recap
  (`sent-papers.jsonl` dedup does NOT apply to the weekly digest).
- Never show which roundup/source a paper came from.
- Paper titles are hyperlinks to arXiv (`<url|title>`).
- Keyword/field names stay in English as-is — never awkwardly translated
  (e.g. "Agentic Memory", "Long-Context Efficiency").
- Do NOT record the weekly message in `sent-papers.jsonl` (it is not a per-paper send).
- Send with ONE post_digest.py call (main + `thread` array). Figures optional.

### Type 1: Conference highlights

Main message:
```
🏛️ *{Conference} highlights* — weekly digest

> *TL;DR*: X and Y dominated this {conference}.

*Trends*
- trend 1 (one line)
- trend 2 (one line)

*Notable papers*: N — one per thread comment
```
- Deriving trends: combine ① topic distribution of the official award/oral/spotlight
  lists and ② social reception (X write-ups, conference specials in weekly roundups)
  into 2–4 currents. No speculation — verified only.
- Selecting the 5 notable papers: `interests.md` match + hype (social buzz, award/oral).
- Thread: ① 1–2 sentence conference overview ② one comment per paper — `[oral]`/
  `[best paper]` tag + why it matters in 1–2 sentences ③ last comment: related links.

### Type 2: Weekly roundup

- Selection: the **union** of DAIR.AI "Top AI Papers of the Week" and TuringPost's
  weekly top N.
- Main message: 📚 title (with the covered date range) + `>` TL;DR + an overview of
  the selection. **Never mention paper titles in the main message** — describe
  field-level currents only.
- Thread: group the papers by field/keyword, **one comment per keyword**, formatted:
```
*Keyword 1: {name}*
> TL;DR: why this cluster matters this week
- <arxiv-url|Paper A> — one-line description
- <arxiv-url|Paper B> — one-line description 👀
```

## Message format

### Type: Paper Digest (main message)

```
{group emoji} *Paper Title* — Institution/Group

> *TL;DR*: one sentence on what this paper does

- *Problem*: what it tries to solve (1 sentence)
- *Key idea*: how it differs from prior work (1–2 sentences)
- *Evaluation*: benchmark name + one headline result

*Buzz*: HF Daily #N · attention on X · conference acceptance/oral (verified facts only)
*Links*: arXiv · alphaXiv
_Interest: Group X (specific keywords)_
```

Emoji rules (MUST follow):
- Exactly ONE emoji, in front of the title. No emoji in body bullets or the buzz/links lines.
- The title emoji follows the group→emoji mapping in `profile/config.md`.
- The TL;DR line must be a quote (`>`).

### Thread rules (Paper Digest)

One topic per comment. Do not cram everything into one long bulleted comment.
The order of the `thread` array in the post_digest.py payload is the comment order:

1. **Comment 1** (`{"file": ...}`): concept figure + one-line caption
2. **Comment 2**: research group/author introduction + the paper's impact
   (HF Daily rank, alphaXiv reactions, conference acceptance/oral, social reception — verified only)
3. **Comments 3–N**: the content — method, experimental setup, results, why it's worth reading, etc.
   One topic per comment. Each comment is a self-contained 2–4 sentence paragraph. No bullets; prose.
4. **Last comment**: related links (GitHub, HF, project page, related papers/posts)

Thread writing style follows the tone guide in `profile/config.md`.

### Type: AI News (phase 2 — not in use yet)

```
📰 *Issue title*

*What happened*: 1–2 sentences
*Why it matters*: 1–2 sentences connecting to the user's interests
🔗 Source · HN discussion · related papers
```

## Writing principles

- Do not try to make the reader understand the whole paper. Provide only the high-level
  insight needed to decide whether to look it up.
- If it gets long, keep the main message to the core and move details to the thread.
- Never state unverified facts (acceptance status, author background, etc.). No speculation.
- Only include URLs you have actually accessed and verified.
- Use emoji, line breaks, and bullets only to the extent they reduce reader fatigue.

## State file schemas

`state/sent-papers.jsonl` — one line = one paper:
```json
{"arxiv_id": "2607.01234", "title": "...", "date": "2026-07-15", "channel": "C0XXXXXXXXX", "ts": "1752537600.123456", "interest_group": "B"}
```
- `ts` is the Slack message timestamp. Required for reaction lookup — always record it.

`state/feedback.jsonl` — written by collect_reactions.py. Never edit directly.
One snapshot per message per run (the same arxiv_id appears multiple times) —
always judge by the LATEST entry per arxiv_id; do not double-count older snapshots.

## Forbidden

- Never access `.hermes/`, `.env`, or any token/key files.
- Never modify files outside `state/` and `profile/` (scripts/ and AGENTS.md are read-only).
- Never re-send the same paper (always check sent-papers.jsonl).
- Never send more than 5 interest picks + 3 exploration papers per day.

---
> Source: [yukyeongleee/paper-digest-bot](https://github.com/yukyeongleee/paper-digest-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
