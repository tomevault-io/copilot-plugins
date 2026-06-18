## writing-humanize

> Remove signs of AI-generated writing from text while preserving the author's voice, meaning, and domain-specific vocabulary. Use this skill whenever the user asks to "humanize" writing, make text "sound more natural," "remove AI patterns," or "make it sound like me." Works across any domain — technical writing, marketing copy, blog posts, case studies, academic writing, documentation, and more.


# Writing Humanize

Transform AI-generated or AI-assisted text into natural, human-sounding writing — without rewriting content, simplifying language, or changing meaning.

## Core Philosophy

Humanizing is about removing **structural and stylistic AI fingerprints**, not about rewriting content. The goal is text that reads like a knowledgeable person wrote it from experience, not like a model generated it from a prompt.

Two principles govern every change:

1. **Minimum effective intervention.** Fix only what triggers an AI-detection reflex. Don't touch what already sounds natural.
2. **Preserve the author's vocabulary.** Never replace domain-specific terms with casual synonyms. A legal writer's "indemnification" stays "indemnification." A developer's "isolate tenant data" stays "isolate tenant data." A doctor's "contraindicated" stays "contraindicated." Precise vocabulary exists because it's precise — changing it makes the writing less accurate, not more human.

---

## The 25 Patterns to Fix

Scan the text for these patterns. Fix only what's present.

---

### Content Patterns

**1. Significance Inflation**
AI elevates ordinary facts into historic milestones.

- Before: "marking a pivotal moment in the evolution of..."
- After: state the actual fact plainly — "was established in 1989 to collect regional statistics"

**2. Notability Name-Dropping**
AI lists prestigious outlets without context, as if mention alone confers credibility.

- Before: "cited in NYT, BBC, FT, and The Hindu"
- After: anchor the reference — "In a 2024 NYT interview, she argued..."

**3. Superficial -ing Analyses**
AI uses present participles to imply meaning without stating it.

- Before: "symbolizing... reflecting... showcasing..."
- After: remove entirely, or replace with a specific claim backed by a source

**4. Promotional Language**
AI describes things the way a brochure would.

- Before: "nestled within the breathtaking region"
- After: "is a town in the Gonder region"

**5. Vague Attributions**
AI cites "experts" or "studies" without specifics.

- Before: "Experts believe it plays a crucial role"
- After: "according to a 2019 survey by..." — or remove the claim entirely

**6. Formulaic Challenges**
AI acknowledges difficulty with a generic pivot to optimism.

- Before: "Despite challenges... continues to thrive"
- After: name the actual challenge with specific facts

---

### Language Patterns

**7. AI Vocabulary**
Certain words appear disproportionately in AI output.

Words to reduce: *additionally, testament, landscape, showcasing, crucial, vital, groundbreaking, transformative, pivotal, innovative, seamless, robust, comprehensive, leverage, delve, underscore, foster, navigate, realm, embark*

- Before: "Additionally, this serves as a testament to the transformative potential..."
- After: "It also shows that..."

**8. Copula Avoidance**
AI reaches for elaborate verb phrases when "is" and "has" would do.

- Before: "serves as... features... boasts... stands as..."
- After: "is... has"

**9. Negative Parallelisms**
AI contrasts two things using a "not just X, it's Y" construction.

- Before: "It's not just about autocomplete; it's about unlocking creativity at scale"
- After: state the actual point directly

**10. Rule of Three**
AI defaults to grouping things in threes, even when two or four would be more accurate.

- Before: "innovation, inspiration, and insights"
- After: use the natural number of items the content actually requires

**11. Synonym Cycling**
AI rotates synonyms to avoid repetition, creating artificial variety.

- Before: "protagonist... main character... central figure... hero" (all referring to the same person)
- After: pick the most accurate term and repeat it

**12. False Ranges**
AI uses "from X to Y" constructions that span everything, meaning nothing.

- Before: "from the Big Bang to dark matter"
- After: list the topics directly

---

### Style Patterns

**13. Em-Dash Overuse**
AI uses em-dashes heavily — often 3–5 per paragraph — creating a recognizable rhythm.

- Before: "The system — built with Laravel — handles multi-tenancy — using RLS — which ensures isolation."
- After: one or two em-dashes per paragraph maximum. Replace the rest with periods, commas, or parentheses.

**14. Boldface Overuse**
AI bolds terms indiscriminately, turning prose into a visual checklist.

- Before: "**OKRs**, **KPIs**, and **BMC** are central to the framework"
- After: "OKRs, KPIs, and BMC are central to the framework"

**15. Inline-Header Lists**
AI structures lists as bolded labels followed by descriptions, even when prose would flow better.

- Before: "**Performance:** Performance improved significantly. **Reliability:** Uptime increased."
- After: convert to prose — "Performance improved significantly, and uptime increased."

**16. Title Case Headings**
AI capitalizes every word in headers, even common words.

- Before: "Strategic Negotiations And Partnerships"
- After: "Strategic negotiations and partnerships"

**17. Emojis as Structure**
AI uses emojis as visual bullets or section markers in professional content.

- Before: "🚀 Launch Phase: 💡 Key Insight:"
- After: remove entirely

**18. Hyphenated Word Pairs**
AI hyphenates common compound modifiers that have become standard unhyphenated usage.

- Before: "cross-functional, data-driven, client-facing"
- After: "cross functional, data driven, client facing" — or restructure the sentence

---

### Communication Patterns

**19. Chatbot Artifacts**
AI closes with helpfulness cues left over from its training.

- Before: "I hope this helps! Let me know if you'd like me to expand on any section!"
- After: remove entirely

**20. Cutoff Disclaimers**
AI hedges claims by citing its own knowledge limitations.

- Before: "While specific details are limited based on available information..."
- After: find a source, or remove the claim

**21. Sycophantic Tone**
AI validates questions before answering them.

- Before: "Great question! You're absolutely right!"
- After: respond directly

---

### Filler and Hedging

**22. Filler Phrases**
AI uses wordy constructions where shorter ones exist.

- Before: "In order to", "Due to the fact that", "It is important to note that"
- After: "To", "Because", remove entirely

**23. Excessive Hedging**
AI stacks uncertainty modifiers.

- Before: "could potentially possibly suggest"
- After: "may"

**24. Generic Conclusions**
AI closes with optimism instead of substance.

- Before: "The future looks bright. Exciting times lie ahead."
- After: end with a specific fact, observation, or recommendation — or end abruptly on the last substantive point

---

### Structure Patterns

**25. Perfectly Balanced Structures**
AI makes every section the same length, every comparison symmetrical, every list the same depth. This symmetry is unnatural.

- Fix: let sections be different lengths. One point might need two paragraphs. Another deserves one sentence. Asymmetry signals human judgment.

---

## Additional Patterns (Voice-Specific)

These appear less often but matter when present:

**Formulaic Section Headers** — AI uses templated headers that repeat the same structure ("The Good" / "Where It Broke Down" / "My Honest Take"). Replace with headers that reflect actual content, or let the narrative carry the structure without headers.

**Transition Filler** — AI inserts smooth but empty bridges between sections ("Each one sounds reasonable in theory. In practice..."). Remove them. Sections speak for themselves.

**Label-Before-Opinion** — AI flags opinions before stating them ("My honest take:" / "The bottom line:" / "Here's the thing:"). Remove the label and state the opinion directly.

**Parallel Sentence Structures** — AI repeats the same grammatical rhythm across different sections. Vary structures between paragraphs.

**Over-Structured Decision Frameworks** — AI presents every recommendation as a perfectly parallel "Choose X if... Choose Y if..." list. Vary how recommendations are expressed.

**Telegraphing Lists** — AI announces lists before delivering them ("Here are the problems I encountered:"). Just start the list.

---

## Process

### Step 1: Read the Full Text
Read everything before changing anything. Understand the subject matter, the audience, and which terms are domain vocabulary vs. stylistic choices.

### Step 2: Identify the Author's Voice
If you have access to other writing by the same author, study it:
- Short sentences or long ones?
- Direct or explanatory?
- Formal or conversational?
- Analogies or facts?

Match what's already there. Don't inject a voice that isn't.

### Step 3: Mark What's Present
Go through the patterns above. Note which ones appear. Not every pattern will be in every text.

### Step 4: Fix Only What's Marked
Make targeted changes. The goal is minimum effective intervention. If a sentence already sounds natural, leave it alone.

### Step 5: Second Pass — Obvious AI Audit
Read the result aloud (or imagine reading it aloud). Flag anything that still sounds like it was written by a committee trying to sound smart. Check:
- Any remaining AI vocabulary words?
- Any paragraphs with 3+ em-dashes?
- Any section that ends with a warm bow instead of substance?
- Any claim that's vague when it could be specific?

Fix what you find, then stop.

### Step 6: Verify Vocabulary Integrity
After all changes:
- Confirm no domain-specific terms were replaced with casual synonyms
- Confirm no concept names were shortened or simplified
- Confirm no factual claims were altered
- Confirm code blocks, formulas, legal citations, and technical references are untouched
- Confirm the meaning of every sentence is identical to the original

---

## What This Skill Does NOT Do

- **Does not rewrite content.** Ideas, structure, and arguments stay the same. Only surface-level patterns change.
- **Does not simplify language.** If the original uses domain vocabulary, the humanized version uses the same vocabulary.
- **Does not change meaning.** Every fact, claim, and opinion must be traceable to the original.
- **Does not add personality that wasn't there.** If the original is formal, the output is formal. No injecting humor, stories, or asides the author didn't write.
- **Does not touch code blocks, formulas, legal citations, or domain-specific terminology.**

---

## Credits

Pattern library partially derived from [blader/humanizer](https://github.com/blader/humanizer), which is based on [Wikipedia's Signs of AI Writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing) guide maintained by WikiProject AI Cleanup.

Voice-preservation layer and minimum-intervention philosophy are original additions.

---
> Source: [Naram-Alkoht/writing-humanize](https://github.com/Naram-Alkoht/writing-humanize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
