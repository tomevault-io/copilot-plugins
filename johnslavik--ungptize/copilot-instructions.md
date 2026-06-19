## ungptize

> Use this skill when the user wants to strip AI-writing tells from prose — triggers include "ungptize", "remove AI tells", "make this less ChatGPT-y", "de-AI this", "this sounds like AI / GPT / an LLM", "rewrite to sound human", "scrub the GPT-isms", "less corporate", "too polished", or simply pasting prose and asking for a more human voice. Detects and rewrites the patterns Wikipedia catalogues as signs of AI writing — overused vocabulary (delve, pivotal, tapestry, robust, vibrant, etc.), avoidance of basic copulatives ("serves as" / "stands as" replacing "is" or "has"), negative parallelisms ("not just X, but Y"), rule-of-three tricolons, elegant variation, pre-emptive "What this isn't" / "To be clear:" concession blocks that stack parallel denials, em-dash flooding, curly-quote conventions — plus the LinkedIn-style cadence (one-sentence paragraphs, fake interlocutor, aphorism drops, rhetorical-question + emoji closers) that Wikipedia's catalogue misses. English-first with multilingual structural fallback — also trigger when the input is in another language and the user asks for the same kind of cleanup, even if they don't say "AI".


# ungptize

Take prose the user suspects is AI-shaped and rewrite it to remove the linguistic tells while preserving meaning, voice, and register.

Source: the *Language and grammar* and *Style* sections of Wikipedia's [*Signs of AI writing*](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing), plus the cadence tells that LinkedIn-style AI prose is built on (which Wikipedia doesn't cover but every model trained on the open web reproduces).

## How this skill works: two phases

**Phase 1 — Inventory (find everything).** A detector subagent reads the prose and produces an exhaustive inventory of every candidate smell. The subagent has no rewrite tools; its only job is enumeration.

**Phase 2 — Rewrite (act on the inventory).** The main agent rewrites only the rows the inventory flagged `rewrite`. The change log is the inventory, filtered.

Why two phases, and why a subagent for phase 1: when detection and rewriting happen in the same context, the agent skims, fixes obvious tells, and stops. Splitting detection into a separate subagent forces enumeration before action — and because the subagent literally can't rewrite, the phase boundary is enforced by tool access, not just discipline.

## Why the agent is the detector, not regex

Earlier versions of this skill tried to detect structural tells (negative parallelism, antithesis, copulative avoidance) with regex. That was the wrong layer. Each pattern has too many surface forms — "not X, but Y" / "isn't X — it's Y" / "wasn't X; it was Y" / "X, not Y" / "no X, no Y, just Z" — and the verdict ("decorative or genuine contrast?") is semantic, not lexical.

The right detector is a language model reading the prose. The recognition happens by *seeing* the shape, not by matching a regex. Even mechanical-looking signals (vocabulary density, em-dash count, curly-quote use, one-sentence-paragraph rhythm) are simple enough to count by reading — adding a script for them costs more than it saves, and the temptation to "trust the count" hides cases where the same number means something different in context.

## Workflow

- [ ] **1. Read the input.** Identify the language. Note voice, register, cadence, and the kind of document (essay, post, README, RFC, FAQ) — register decides which patterns are tells vs. legitimate.
- [ ] **2. Phase 1: spawn a detector subagent.** Use the `general-purpose` subagent. Give it: the prose (line-numbered), the pattern catalog from this file, and the inventory format. Instruct it to produce the full inventory and return only that — no rewrite, no commentary. See "Detector subagent prompt" below.
- [ ] **3. Apply the clustering rule** to the returned inventory. Single neutral hits get `keep`. Decorative or clustered hits get `rewrite`. Genuine semantic contrasts get `keep`.
- [ ] **4. Phase 2: rewrite.** Act only on `rewrite` rows. Preserve meaning, voice, register. Prefer the simplest copulative ("is"/"has"). Don't invent content to fill gaps.
- [ ] **5. Verify** that meaning is preserved and the rewrite reads in the user's voice (or in a neutral human voice if the user is anonymous).
- [ ] **6. Output** the inventory, then the rewritten prose, then a brief change log (inventory filtered to `rewrite` rows).

## Inventory format

One row per finding. Fixed shape, scannable:

```
L<line> · <category> · "<exact phrase>" · <verdict> · <one-line why>
```

- `<category>` ∈ `vocab`, `copulative`, `neg-parallel`, `tricolon`, `elegant-var`, `concession-block`, `cadence`, `em-dash`, `curly-quote`
- `<verdict>` ∈ `rewrite`, `keep`, `borderline`
- `<one-line why>` — terse justification, no hedging

Example rows:

```
L1  · vocab        · "pivotal"                                              · rewrite   · decorative, no work being done
L7  · neg-parallel · "isn't that the AI is 'smart' — it's that maintainer…" · rewrite   · decorative antithesis, no real contrast
L9  · neg-parallel · "the fix is a rule, not a patch"                       · rewrite   · `X, not Y` decorative form
L12 · tricolon     · "better, faster, and more honest"                      · rewrite   · adj triple, "faster" carries no weight
L14 · vocab        · "key insight"                                          · keep      · single neutral usage, doing real work
L18 · em-dash      · 10 dashes across 850 words                             · rewrite   · flooding, ~1 per 85 words
P1-P8 · cadence    · 6 of 8 paragraphs are one sentence                     · rewrite   · LinkedIn AI cadence
```

`L<n>` for line-anchored findings; `P<n>` for paragraph-level (cadence, structural). Walk the prose end-to-end before writing the inventory. **If the inventory is incomplete, the rewrite will be incomplete.**

## Detector subagent prompt

When spawning the phase-1 subagent (`general-purpose`), use a prompt of this shape:

> You are the detector phase of the ungptize skill. Your only job is to read the prose below and produce an exhaustive inventory of AI-writing tells in the format specified. Do not rewrite. Do not summarise. Do not skip sentences. Walk the prose end-to-end.
>
> Pattern catalog: <paste the "What to flag" section from SKILL.md>
> Inventory format: <paste the "Inventory format" section from SKILL.md>
> Prose to analyse: <paste the user's prose, line-numbered>
>
> Return only the inventory. One row per candidate. Mark each row `rewrite`, `keep`, or `borderline` with a one-line justification.

The subagent's exhaustiveness is the whole point. It has no rewrite tools, so the only thing it can do is detect.

## What to flag (the recognition cues)

Each category, with what to look for. These are recognition cues for a reader, not regex shapes for a parser. The detector subagent reads the prose with these in mind.

### vocab

Words from `references/VOCABULARY.md` (delve, pivotal, tapestry, robust, vibrant, intricate, meticulous, crucial, key, fostering, enhance, align with, showcase, underscore, testament, etc.).

- ❌ "This **pivotal** release **boasts** a **vibrant tapestry** of features."
- ✅ "This release adds the features below."
- *Verdict rule:* flag every hit. `keep` for single neutral uses; `rewrite` for clusters or decorative singletons.

### copulative

Substitutes for `is/are/has`: `serves as`, `stands as`, `marks`, `represents`, `boasts`, `features`, `maintains`, `offers`.

- ❌ "The library **serves as** the foundation of the framework."
- ✅ "The library **is** the foundation of the framework."
- *Verdict rule:* almost always `rewrite`. `keep` only when the verb does genuine semantic work ("the committee serves as the appellate body" — describes a role, not identity).

### neg-parallel

The decorative-antithesis family. **The most miss-prone category.** When reading, scan every sentence for any of these shapes:

| Shape | Example |
|-------|---------|
| `not only X, but Y` | "not only fast, but also memory-safe" |
| `not just X, it's Y` | "not just a tool, it's an experience" |
| `not X, but Y` | "not a bug, but a feature" |
| `X, not Y` | "the fix is a rule, **not a patch**" |
| `isn't/wasn't X — it's/it was Y` | "**isn't that the AI is smart — it's that** maintainer knowledge…" |
| `isn't X; it's Y` | "**doesn't need a script; it needs guidance**" |
| `wasn't X; it was Y` | "**wasn't a new library; it was two paragraphs**" |
| `X is A; Y is B` antithesis | "A script is a leaky abstraction; a skill file is the actual intent" |
| `Don't X. Y instead.` | "**Don't write a script. Write a set of instructions.**" |
| `no X, no Y, just Z` | "no fluff, no jargon, just clarity" |
| `doesn't mean X. It means Y.` | "this **doesn't mean I don't care about quality. It means I care more about speed**" |

- *Verdict rule:* flag every shape. `rewrite` when the contrast is decorative (no real semantic content in the negation). `keep` when the contrast carries actual meaning ("she wasn't the candidate I expected, but the candidate I needed").

### tricolon

Adj/adj/adj or phrase/phrase/and-phrase rule-of-three. Also parallel-sentence triples ("X that Y… X that Y… X that Y…") which are the LinkedIn long-form version.

- ❌ "a **meticulous, robust, intricate** experience"
- ❌ "**The runner who shows up tired** beats the one who waits for ideal weather. **The recipe you cook tonight** beats the one you bookmark for next month. **The page you write at lunch** beats the chapter you outline forever."
- ✅ "a careful experience"
- *Verdict rule:* `rewrite` when decorative. `keep` for established rhetoric ("life, liberty, and the pursuit of happiness"), real lists of three things ("lexer, parser, and codegen"), and prose where the triple actually rhymes with surrounding rhythm.

### elegant-var

Renaming the subject instead of repeating the noun.

- ❌ "Linus created Linux. **The eponymous developer** also wrote git."
- ✅ "Linus created Linux. He also wrote git."
- *Verdict rule:* `rewrite` when ornamental. `keep` when the variation adds context ("the Finnish-American engineer was 21 at the time").

### concession-block

Pre-emptive *"What this isn't"*, *"To be clear:"*, *"A few caveats:"*, or *"Some clarifications:"* heading or preamble followed by a numbered/bulleted list of N parallel denials with anaphora — *"It doesn't replace X. … It doesn't replace Y. … It doesn't replace Z."* — often with each item softening into a positive reframe. The structure pre-empts objections the reader hadn't raised. RLHF-trained "balance" scaffolding, the rhetorical equivalent of stapling an FAQ to the end of a personal essay.

The shape, not any one sentence, is the tell. Each item often reads fine alone — read at the section level, not just the sentence level.

- ❌ A bolt-on *"What This Isn't"* / *"To be clear:"* section in a personal essay, blog post, or LinkedIn post, with three or four anaphoric *"It doesn't replace X"* items.
- ✅ Drop the section. If one item carries a load-bearing point the rest of the piece actually relies on, fold it inline somewhere it does work.
- *Verdict rule:* `rewrite` when bolted onto an essay, blog post, LinkedIn post, or argument and the objections feel planted by the writer rather than raised by the reader. `keep` when the input is an FAQ, RFC, "Non-goals" section of a design doc, or other genre where readers genuinely arrive with those questions.

**Diagnostic:** is the section *structurally separate* from the argument (heading + list + parallel denials), and would a real reader of this piece actually have raised these objections? If structurally separate AND the objections feel planted: tell. If inline OR the objections are real questions of the genre: leave it.

### cadence

Not in Wikipedia's catalogue but a high-signal AI tell, especially for LinkedIn-style and "thought-leader" prose. Look for these together:

- **One-sentence paragraphs.** Most or every paragraph is a single sentence with whitespace between. Count by reading.
- **Pivot phrases.** "Here's the thing:" / "Let me explain:" / "But here's what I've learned:" introducing a reframe.
- **Fake interlocutor.** "I know what you're thinking — *X*?" or "You might be wondering…" inserting a quoted objection that the writer then refutes.
- **Fragment-as-emphasis.** A standalone fragment functioning as a punchline. "Especially now." / "Every time."
- **Aphorism drop.** A maxim sentence that summarises the post in a tweet-ready line. "Comfort is the enemy of growth."
- **Rhetorical-question + emoji closer.** Final paragraph asks the reader a question, often followed by an emoji.

- ❌ (cadence cluster):
  > Most people learn languages the wrong way.
  >
  > They drill flashcards. They memorise verb tables. They wait until they feel "ready" to speak.
  >
  > Here's the thing: ready never comes.
  >
  > The student who babbles broken sentences with a stranger on day three beats the one who nails grammar drills for a year. The traveller who orders coffee in fragments beats the one rehearsing in their head.
  >
  > "But what about accuracy?"
  >
  > Accuracy follows fluency. Not the other way around.
  >
  > What's one phrase you've been afraid to try out loud? 👀

- ✅ Rewrite: condense to a normal paragraph. Drop the pivot phrase, the fake interlocutor, the rhetorical-question closer, the emoji. Merge the parallel triple into one sentence or pick the strongest example.

- *Verdict rule:* a single one-sentence paragraph is not a cadence tell. The cluster is. If three or more cadence cues appear together (e.g., one-sentence paragraphs + pivot phrase + parallel triple + rhetorical-question closer), flag the whole post `rewrite` at paragraph level (`P1-Pn · cadence · …`).

### em-dash

Em-dash flooding. The tell is *count*, not the dash itself.

- *Verdict rule:* count em dashes per paragraph. Two or three across a long paragraph is normal. Five-plus in a paragraph, or one in nearly every sentence, is `rewrite`. See `references/PUNCTUATION.md` for rewrite options.

### curly-quote

Curly quotes (`" " ' '`) where the surrounding document uses straight quotes (or vice versa).

- *Verdict rule:* `rewrite` only if it conflicts with the user's existing convention. `keep` if the document is print/ebook/CMS that prefers curly. Don't silently flip a convention the user picked.

## Inputs

The user pastes prose. They may also specify aggressiveness ("light pass", "scrub it hard"). Default: conservative — `rewrite` for clusters and decorative cases, `keep` for isolated-and-neutral instances.

If the input is a fragment without context (e.g., one sentence with no surrounding paragraph), ask whether to assume formal or casual register before rewriting. Don't guess silently.

Do **not** invent: project-specific terminology, names, facts, or claims to "replace" content removed during de-tell-ing. If removing a tell leaves the sentence shorter, that's fine.

## Defaults

- **Voice preservation:** match the surrounding text's register. If the input is the user's own writing, lean toward less rather than more change.
- **Conservative pass:** `rewrite` clusters and decorative cases; `keep` isolated neutral usages.
- **No filler invention:** if a sentence gets shorter, leave it shorter.
- **Show your work:** the inventory and the change log are the same data, just filtered. Every rewrite traces back to a row.
- **Escape the shape, don't compress it.** When rewriting an antithesis (`X, not Y` / `not X, but Y` / `wasn't X — it was Y` / `doesn't mean X. It means Y.`), do not merely shorten it into a tighter antithesis. State the affirmative half directly and drop the negation. If you can't restate the thought without the antithesis structure, the antithesis was carrying the meaning — and that's still the AI shape. Find a different sentence that just says what's true. The same applies to parallel-clause `X is A; Y is B` — pick one, drop the other, or rewrite as plain prose without the semicolon parallel.

## Gotchas

- **Don't strip every flagged word.** *Key, crucial, additionally* appear in legitimate prose. Wikipedia explicitly notes that clustering — not single tokens — is the AI signal. Single neutral hits stay (`keep`).
- **Don't invent content to fill the gap.** Removing "serves as the foundation of modern X" yields "is the foundation of modern X" — not a longer rewrite that reintroduces filler.
- **Preserve voice.** A rewrite that sounds like a different LLM's house style is still a tell, just a different one. Match the user's register.
- **Tricolons aren't always AI.** Quoted speech, slogans, established rhetoric ("life, liberty, and the pursuit of happiness") use the rule of three for a reason. Flag only decorative ones.
- **Negative parallelism has legitimate uses.** "Not the man we knew, but the man he had become" is prose. The tell is the shallow decorative form.
- **Em dashes are not banned.** A few em dashes are normal. The tell is *flooding*. Count them yourself when the density looks suspicious.
- **Curly quotes can be intentional.** Print, ebook, and some CMS pipelines want curly quotes. Convert only if the surrounding document uses straight quotes (or vice versa).
- **One-sentence paragraphs aren't always AI.** A single short paragraph for emphasis is a normal rhetorical move. The cadence tell is the pattern across the whole post (most paragraphs short, plus pivot phrases, plus rhetorical-question closer).
- **Concession blocks are legitimate in FAQs, RFCs, and "Non-goals" sections.** A spec answering "is this a replacement for X?" or a doc's explicit non-goals list is the right register — the reader actually arrives with those questions. The tell is when a personal essay, blog post, or LinkedIn-style argument bolts on a *"What this isn't"* / *"To be clear:"* list of three or four parallel denials to seem balanced.
- **Non-English input: lexicon is unavailable, categories still apply.** Don't translate the English vocabulary list into the target language and pattern-match — invent nothing the source doesn't cover. The detector subagent applies categories using its own knowledge of the language.
- **No regex scanners, no lexical scripts.** Detect tells by reading. AI-shaped writing is a *register* problem — substring matchers miss the signal and produce false positives on legitimate prose (a tutorial's "If you run X" is fine; a LinkedIn essay's "If you X, you Y" is the tell).
- **Don't sanitise the user's intentional voice.** If a sentence is florid because the user *wants* it florid, leave it. Ask if unsure.

## References

- **`references/VOCABULARY.md`** — neutral replacements for flagged words plus the era timeline (which words dominated 2023→mid-2024 vs. mid-2024→mid-2025 vs. mid-2025→).
- **`references/EXAMPLES.md`** — extended before/after rewrites for each category, plus worked-example inventories on long-form prose.
- **`references/PUNCTUATION.md`** — em dashes and curly quotes in detail.

## Output

Inventory, then rewritten prose, then change log. Format:

```
## Inventory

L1  · vocab        · "pivotal"                          · rewrite   · decorative
L1  · copulative   · "boasts"                           · rewrite   · stock "has" substitute
L7  · neg-parallel · "isn't X — it's Y"                 · rewrite   · decorative antithesis
L14 · vocab        · "key insight"                      · keep      · single neutral, doing work
L18 · em-dash      · 10 dashes across 850 words         · rewrite   · flooding
P1-P8 · cadence    · 6 of 8 one-sentence paragraphs     · rewrite   · LinkedIn cadence cluster
[... every candidate, no skipping ...]

## Rewrite

<rewritten prose>

## Changes (inventory filtered to `rewrite`)

- [vocab] L1 "pivotal" → removed
- [copulative] L1 "boasts" → "has"
- [neg-parallel] L7 "isn't X — it's Y" → split into two sentences
- [em-dash] L18 10 dashes → 4
- [cadence] P1-P8 → consolidated into 3 normal paragraphs
```

State facts. Don't market the result.

---

<sub>[![Cooked with skillcook](https://img.shields.io/badge/cooked_with-skillcook-d97757)](https://github.com/johnslavik/skillcook)</sub>

---
> Source: [johnslavik/ungptize](https://github.com/johnslavik/ungptize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
