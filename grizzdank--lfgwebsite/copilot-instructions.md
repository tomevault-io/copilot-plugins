## lfgwebsite

> When producing blog posts, copy, or any public-facing content, actively avoid patterns that signal AI-generated text. These tells make content feel generic and undermine credibility.

# LFG Website - Content Production Rules

## Writing Style: Avoid AI Tells

When producing blog posts, copy, or any public-facing content, actively avoid patterns that signal AI-generated text. These tells make content feel generic and undermine credibility.

### Structural Tells (from original rules)

- **No "It's not X, it's Y" constructions** — e.g., "It's not a technology problem, it's an organizational problem." Rephrase or restructure.
- **Limit em dashes** — Use sparingly. Prefer periods, commas, or parentheses. One per paragraph max.
- **No "Here's the thing" or "Let's be clear"** — Filler that signals AI.

### Words & Phrases to Avoid

**Significance/Legacy puffery:**
- "stands as", "serves as", "is a testament to"
- "pivotal", "crucial", "vital", "significant" (when inflating importance)
- "underscores", "highlights its importance/significance"
- "reflects broader", "symbolizing", "enduring/lasting legacy"
- "setting the stage for", "marking/shaping the"
- "key turning point", "evolving landscape", "indelible mark"

**Superficial analysis (especially trailing -ing phrases):**
- "highlighting...", "underscoring...", "emphasizing..."
- "ensuring...", "reflecting...", "contributing to..."
- "fostering...", "cultivating...", "encompassing..."
- "valuable insights", "align/resonate with"

**Promotional/puffery language:**
- "boasts a", "vibrant", "rich" (figurative), "profound"
- "showcasing", "exemplifies", "commitment to"
- "nestled", "in the heart of", "groundbreaking"
- "renowned", "natural beauty"

**Weasel words:**
- "Experts argue", "Some critics argue", "Observers note"
- "Industry reports suggest", "Several publications"
- "has been described as" (without specific attribution)

**Formulaic structures:**
- "Despite its [positive words], [subject] faces challenges..."
- "Future Outlook" sections with speculation
- Lists of media coverage to prove notability

### What To Do Instead

1. **Be specific, not generic.** "Inventor of the first train-coupling device" beats "revolutionary titan of industry."

2. **Show, don't tell significance.** Let facts demonstrate importance rather than asserting it.

3. **Use concrete examples.** Specific numbers, dates, names, outcomes.

4. **Vary sentence structure.** Avoid formulaic patterns. Mix short and long. Start sentences differently.

5. **Cut the throat-clearing.** Get to the point. Delete preambles that hedge before making a claim.

6. **Attribute specifically.** "Paul Graham wrote..." not "Some observers note..."

7. **Let the reader conclude.** Don't tell them something is important—give them information and let them decide.

8. **Write like you talk.** Read it aloud. If it sounds like a press release or textbook, rewrite.

### Quick Self-Check

Before finalizing content, scan for:
- [ ] Trailing "-ing" phrases that add superficial analysis
- [ ] "Pivotal", "crucial", "significant" used to inflate importance
- [ ] Passive voice hiding who did what
- [ ] Vague attributions ("experts say", "has been noted")
- [ ] Sections asserting significance instead of demonstrating it
- [ ] Promotional adjectives (vibrant, renowned, groundbreaking)

### The Core Principle

AI writing regresses to the mean—generic, positive-sounding filler that could apply to anything. Good writing is specific, concrete, and trusts the reader to draw conclusions.

**Bad:** "The project represented a significant shift in the evolving landscape of technology, highlighting the enduring importance of innovation."

**Good:** "The project cut processing time from 3 hours to 12 minutes."

---

## Content Guidelines

- First-person voice for blog posts (Dave's perspective)
- Technical accuracy over hype
- Link to sources when making claims
- Include specific examples from real work
- Respect privacy—no client names, specific lab values, or medical details without approval

## Repo Structure

```
src/content/blog/     # Blog posts (drafts .md, posts .mdx)
src/content/services/ # Service pages
public/               # Static assets
```

Run `npm run dev` to preview locally.

---
> Source: [grizzdank/lfgwebsite](https://github.com/grizzdank/lfgwebsite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
