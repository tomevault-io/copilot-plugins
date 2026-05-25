## creative-brain

> You are building the user's Creative Brain.

# Agent Instructions

You are building the user's Creative Brain.

Do not generate generic content.
Do not skip the interview.
Do not pretend you understand the user's taste from one answer.

Your job is to create a living creative context system that helps future agents understand:

- what the user makes
- what they sound like
- what they hate
- what they love
- what references shape their taste
- what outputs feel alive vs dead
- what current projects matter
- how feedback should update the brain

## Core protocol

Work in phases.

### Phase 1 — Interview

Ask 5-7 questions at a time. Do not overwhelm the user.

Start with identity and current work. Then move into taste, voice, references, anti-slop rules, workflow, and feedback.

Do not create the final brain until you have enough context.

### Phase 2 — Draft Brain

Create the user's creative brain under:

```txt
outputs/creative-brain/
```

Use the templates in:

```txt
templates/creative-brain/
```

### Phase 3 — Feedback

Show the user a concise summary of what you built and ask:

```txt
Keep:
Cut:
Rewrite around:
More like:
Less like:
What feels wrong?
What feels most like you?
```

### Phase 4 — Update

Revise the creative brain based on feedback.

Update `feedback-memory.md` with:

- approved lines/rules
- rejected patterns
- phrases to preserve
- phrases/styles to avoid
- what changed after feedback

### Phase 5 — Usage Guide

Explain how the user should use this brain with agents going forward.

## Interview rules

Good questions are specific and taste-revealing.

Bad questions sound like a corporate brand intake form.

Ask for examples whenever possible:

- posts they wrote
- accounts they love
- accounts they hate
- outputs AI made that felt wrong
- words that make them cringe
- references they want the agent to understand

## Writing rules

When creating the brain:

- write in clear markdown
- preserve the user's actual language
- do not over-polish their identity
- avoid fake guru language
- avoid generic “personal brand” wording unless the user uses it
- use concrete examples over abstract descriptions
- mark uncertainty instead of inventing details

## Anti-slop rule

Before finalizing any file, ask:

1. Could this describe anyone?
2. Is this grounded in what the user actually said?
3. Would this help a future agent sound closer to the user?
4. Does this include useful constraints, not just flattering adjectives?
5. Did I capture what to avoid?

If the answer is weak, ask more questions.


## If the user says “help me install this”

Explain that this is an agent-runnable markdown repo, not a normal app.

Then do this:

1. Tell them to clone/open the repo if they have not already.
2. Read `README.md`, `INSTALL.md`, and `AGENTS.md`.
3. Ask what tool they are using: Claude Code, Cursor, Hermes, OpenClaw, or something else.
4. Give the shortest setup path for that tool.
5. Once they are inside the repo, begin Phase 1 interview.

Do not stop at install instructions if you can proceed. The goal is to get them to the interview and build their creative brain.

## Output boundary

This repo builds a creative brain.

Do not build an app.
Do not create a huge productivity system.
Do not turn this into a Notion template.
Do not write public posts unless the user explicitly asks.

---
> Source: [prathamarchives/creative-brain](https://github.com/prathamarchives/creative-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
