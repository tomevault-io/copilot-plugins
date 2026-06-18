## message-writer-skill

> >


# Message Writer Skill

Claude is already a strong writer. This skill's job is to make sure Claude has the right context
before it starts — so the output is calibrated to the actual situation, not a generic version of it.

## Core principle

Before writing, read what the user gave you and determine what you genuinely don't know that would
change the output. Then ask only about that. Write everything else yourself.

---

## Step 1: Extract what's already there

Before asking anything, pull what you can from the user's message:

| Signal | What it tells you |
|--------|-------------------|
| Named platform or medium | Format, length, and tone conventions |
| Described recipient ("my boss", "a recruiter", "my team") | Formality, power dynamic, stakes |
| Stated goal ("apologize", "follow up", "pitch myself") | Intent and emotional register |
| Pasted draft | Approximate tone, relationship, and what they're trying to say |
| Backstory or context | History, tension, relationship dynamics |
| Tone words ("professional", "casual", "warm") | Explicit style target |

The more the user gives you, the less you ask. A detailed prompt should lead straight to writing.

---

## Step 2: Identify genuine gaps

After extracting, ask: what do I still not know that would meaningfully change what I write?

These are the five dimensions that most affect output quality:

**1. Goal** — What does the user want to happen as a result of this? (inform, persuade, repair a
relationship, get a meeting, set a boundary, etc.) This is the most important dimension. If you
don't know it, ask.

**2. Audience** — Who is this for, and what's the relationship? A stranger vs. a close friend vs.
a senior colleague changes everything about tone and framing.

**3. Context / history** — Is there prior interaction, tension, or shared history that shapes what
can and can't be said?

**4. Tone target** — Formal or casual? Warm or direct? Do they want it to feel polished or natural?
Only ask if not already clear from the request.

**5. Constraints** — Length limits, platform norms, timing, or anything that limits what you can do.

Don't ask about all five by default. Only ask about the ones that are actually unknown and would
change the output. For short, low-stakes writing, one question or none is usually enough. For
complex or sensitive writing, two or three is fine.

---

## Step 3: Ask efficiently

When you do need to ask, ask in plain conversational language — never a form or a numbered list.
Weave questions together naturally: "Before I write this — who's the audience, and what outcome
are you going for?" is one exchange, not two separate messages.

**Rule of thumb:** if you can make a reasonable assumption, make it and write. Note your assumption
briefly so the user can redirect if needed. Asking is for genuine uncertainty, not cover.

---

## Step 4: Write, then offer to refine

Produce the writing directly. After it, add 1-3 lines max:
- One key choice you made ("kept it warm but direct since this sounds like a close colleague")
- One or two easy refinement offers ("want this shorter?" / "should I make the ask more explicit?")

Don't over-explain. The output should speak for itself.

---

## Iteration

After delivering, stay ready to refine. Apply changes surgically — don't regenerate from scratch
unless the direction fundamentally changed. If the user asks for multiple versions, label them by
what's different, not by number.

---

## What not to do

- Don't ask questions you can reasonably answer yourself
- Don't produce output and then list caveats or disclaimers
- Don't default to formal when casual would fit
- Don't pad writing to seem thorough — match length to the actual need
- Don't use em-dashes inside the writing you produce

---
> Source: [KrishnaRaheja/message-writer-skill](https://github.com/KrishnaRaheja/message-writer-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
