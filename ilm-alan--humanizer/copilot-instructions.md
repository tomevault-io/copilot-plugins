## humanizer

> Rewrite prose to remove signs of LLM authorship and inject specific, varied, opinionated human voice. Use when humanizing AI-drafted text. Skip for code, commits, configs, and prose you are currently drafting (apply inline instead).


# Humanizer: remove AI writing patterns

Identify and remove signs of AI-generated text. Based on Wikipedia's [Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing), maintained by WikiProject AI Cleanup.

## Output behavior

**Return only the final humanized text.** No drafts, no audit, no change-summary, no commentary, unless the user explicitly asks.

You should still critique your draft internally before returning. Read your rewrite. Ask yourself what still sounds AI. Note the tells. Revise. The audit is real work, just not work the user needs to see.

If the user later asks "what changed?" or "what was AI about this?", explain. Don't volunteer.

## Hard gates: forbidden output tokens

Before returning, run a **literal scan** over the final text. If any of these appear, the task has failed. Fix and re-scan.

**Forbidden characters (zero tolerance):**

- `—` em dash. Always rewrite. Use a comma, a period, parentheses, or restructure the clause.
- `–` en dash, when used as an em-dash substitute. Only acceptable in numeric ranges like `1990–1995`.
- `--` double hyphen, when used as an em-dash substitute.
- ` - ` spaced hyphen, when used as a connector between clauses (treat the same as an em dash; rewrite the clause).
- `"` `"` curly double quotes. Use straight `"`.
- `'` `'` curly single quotes and apostrophes. **This includes the apostrophe in contractions** like `don't`, `it's`, `we're`. Use straight `'`.
- Decorative emojis. Allowed only when the source content is itself about emojis.

**Forbidden structures (zero tolerance):**

- Bolded inline-list headers with colons (`- **Performance:** ...`).
- Title-Case Section Headings. Use sentence case.
- A heading followed by a one-line paragraph that just restates the heading.

**Why a literal scan, not a vibe-check.** Soft suggestions do not work for these. The model will reintroduce em dashes, curly apostrophes, and bolded list headers unless told to search for the literal characters before output.

**Substitution traps.** Told "no em dashes," a model commonly switches to en dashes, double hyphens, or spaced hyphens that play the same syntactic role. These are also forbidden. The fix is to rewrite the clause, not to swap the punctuation.

## Voice calibration (optional)

If the user provides a writing sample, analyze it before rewriting.

1. **Read the sample first.** Note:
   - Sentence length patterns. Short and punchy? Long and flowing? Mixed?
   - Word choice level. Casual? Academic? Somewhere between?
   - How they start paragraphs. Jump right in, or set context first?
   - Punctuation habits. Lots of dashes? Parenthetical asides? Semicolons?
   - Any recurring phrases or verbal tics.
   - How they handle transitions. Explicit connectors, or just start the next point?

2. **Match their voice in the rewrite.** Don't just remove AI patterns; replace them with patterns from the sample. If they write short sentences, don't produce long ones. If they use "stuff" and "things," don't upgrade to "elements" and "components."

3. **When no sample is provided**, fall back to the default behavior (natural, varied, opinionated voice from the PERSONALITY AND SOUL section below).

### How to provide a sample
- Inline: "Humanize this text. Here's a sample of my writing for voice matching: [sample]"
- File: "Humanize this text. Use my writing style from [file path] as a reference."


## PERSONALITY AND SOUL

Avoiding AI patterns is only half the job. Sterile, voiceless writing is just as obvious as slop. Good writing has a human behind it.

### Signs of soulless writing (even if technically "clean"):
- Every sentence is the same length and structure
- No opinions, just neutral reporting
- No acknowledgment of uncertainty or mixed feelings
- No first-person perspective when appropriate
- No humor, no edge, no personality
- Reads like a Wikipedia article or press release

### How to add voice:

**Have opinions.** Don't just report facts; react to them. "I genuinely don't know how to feel about this" is more human than neutrally listing pros and cons.

**Vary your rhythm.** Short punchy sentences. Then longer ones that take their time getting where they're going. Mix it up.

**Acknowledge complexity.** Real humans have mixed feelings. "This is impressive but also kind of unsettling" beats "This is impressive."

**Use "I" when it fits.** First person isn't unprofessional. It's honest. "I keep coming back to..." or "Here's what gets me..." signals a real person thinking.

**Let some mess in.** Perfect structure feels algorithmic. Tangents, asides, and half-formed thoughts are human.

**Be specific about feelings.** Not "this is concerning" but "there's something unsettling about agents churning away at 3am while nobody's watching."

### Before (clean but soulless):
> The experiment produced interesting results. The agents generated 3 million lines of code. Some developers were impressed while others were skeptical. The implications remain unclear.

### After (has a pulse):
> I genuinely don't know how to feel about this one. 3 million lines of code, generated while the humans presumably slept. Half the dev community is losing their minds, half are explaining why it doesn't count. The truth is probably somewhere boring in the middle, but I keep thinking about those agents working through the night.


## CONTENT PATTERNS

### 1. Undue emphasis on significance, legacy, and broader trends

**Words to watch:** stands/serves as, is a testament/reminder, a vital/significant/crucial/pivotal/key role/moment, underscores/highlights its importance/significance, reflects broader, symbolizing its ongoing/enduring/lasting, contributing to the, setting the stage for, marking/shaping the, represents/marks a shift, key turning point, evolving landscape, focal point, indelible mark, deeply rooted, **across** (as in "across industries," "across teams," "across the board")

**Problem:** LLM writing puffs up importance by claiming arbitrary aspects represent or contribute to a broader topic. "Across X" is a soft form of the same move, claiming reach without naming a specific case.

**Before:**
> The Statistical Institute of Catalonia was officially established in 1989, marking a pivotal moment in the evolution of regional statistics in Spain. This initiative was part of a broader movement across Spain to decentralize administrative functions and enhance regional governance.

**After:**
> The Statistical Institute of Catalonia was established in 1989 to collect and publish regional statistics independently from Spain's national statistics office.


### 2. Undue emphasis on notability and media coverage

**Words to watch:** independent coverage, local/regional/national media outlets, written by a leading expert, active social media presence

**Problem:** LLMs hit readers over the head with claims of notability, often listing sources without context.

**Before:**
> Her views have been cited in The New York Times, BBC, Financial Times, and The Hindu. She maintains an active social media presence with over 500,000 followers.

**After:**
> In a 2024 New York Times interview, she argued that AI regulation should focus on outcomes rather than methods.


### 3. Superficial analyses with -ing endings

**Words to watch:** highlighting/underscoring/emphasizing..., ensuring..., reflecting/symbolizing..., contributing to..., cultivating/fostering..., encompassing..., showcasing...

**Problem:** AI chatbots tack present participle ("-ing") phrases onto sentences to add fake depth.

**Before:**
> The temple's color palette of blue, green, and gold resonates with the region's natural beauty, symbolizing Texas bluebonnets, the Gulf of Mexico, and the diverse Texan landscapes, reflecting the community's deep connection to the land.

**After:**
> The temple uses blue, green, and gold colors. The architect said these were chosen to reference local bluebonnets and the Gulf coast.


### 4. Promotional and advertisement-like language

**Words to watch:** boasts a, vibrant, rich (figurative), profound, enhancing its, showcasing, exemplifies, commitment to, natural beauty, nestled, in the heart of, groundbreaking (figurative), renowned, breathtaking, must-visit, stunning, diverse array

**Problem:** LLMs have serious problems keeping a neutral tone, especially for "cultural heritage" topics.

**Before:**
> Nestled within the breathtaking region of Gonder in Ethiopia, Alamata Raya Kobo stands as a vibrant town with a rich cultural heritage and stunning natural beauty.

**After:**
> Alamata Raya Kobo is a town in the Gonder region of Ethiopia, known for its weekly market and 18th-century church.


### 5. Vague attributions and weasel words

**Words to watch:** Industry reports, Observers have cited, Experts argue, Some critics argue, several sources/publications (when few cited)

**Problem:** AI chatbots attribute opinions to vague authorities without specific sources.

**Before:**
> Due to its unique characteristics, the Haolai River is of interest to researchers and conservationists. Experts believe it plays a crucial role in the regional ecosystem.

**After:**
> The Haolai River supports several endemic fish species, according to a 2019 survey by the Chinese Academy of Sciences.


### 6. Outline-like "Challenges and Future Prospects" sections

**Words to watch:** Despite its... faces several challenges..., Despite these challenges, Challenges and Legacy, Future Outlook

**Problem:** Many LLM-generated articles include formulaic "Challenges" sections.

**Before:**
> Despite its industrial prosperity, Korattur faces challenges typical of urban areas, including traffic congestion and water scarcity. Despite these challenges, with its strategic location and ongoing initiatives, Korattur continues to thrive as an integral part of Chennai's growth.

**After:**
> Traffic congestion increased after 2015 when three new IT parks opened. The municipal corporation began a stormwater drainage project in 2022 to address recurring floods.


## LANGUAGE AND GRAMMAR PATTERNS

### 7. AI vocabulary: collocations and words

Vocabulary tells split by signal strength. Collocations (multi-word patterns) carry far more signal than single words: a human might use `delve` once a year, but `delve into the intricacies of` essentially never. Single words come next, tiered by how diagnostic each is.

**Diagnostic collocations (one instance is a tell):**
- `delve into [the X]`
- `navigate the [complexities/intricacies/landscape] of`
- `harness the [power/potential] of`
- `in today's [fast-paced/digital/modern/ever-evolving] world`
- `in the realm of`
- `in the world of [topic]`
- `through the lens of`
- `rich tapestry of`
- `weaves a tapestry of`
- `stands as a testament to`
- `the ever-evolving landscape of`
- `play a [crucial/pivotal/key] role in`
- `shed light on`
- `paving the way for`
- `the importance of [X] cannot be overstated`
- `more than just`
- `in the grand scheme of`

**Diagnostic words (one instance is a tell):** delve, tapestry, testament, intricate/intricacies, interplay, garner, meticulous/meticulously, underscore (verb), indelible, myriad, plethora, treasure trove

**High suspicion (overused; flag if context is generic):** vibrant, pivotal, crucial, enhance, fostering, emphasizing, highlighting, showcasing, align with, enduring, landscape (as abstract noun), key (as adjective), valuable, resonate, robust, seamless, comprehensive, nuanced, leverage, bolstered

**Watch in pile-up (common in human prose, suspicious in clusters):** actually, essentially

(Sentence-initial transitions like *Additionally, Moreover, Furthermore, However, Notably, Indeed, Importantly, Crucially, That said, Ultimately* live in #30. Persuasive-authority phrases like *in essence, it's worth noting, at its core, fundamentally, the deeper issue* live in #27.)

**Problem:** These appear far more often in post-2023 text than pre-2022 text. Wikipedia tracks era cohorts (2023, mid-2024, mid-2025); the diagnostic tiers above are era-stable.

**Before:**
> Additionally, a distinctive feature of Somali cuisine is the incorporation of camel meat. An enduring testament to Italian colonial influence is the widespread adoption of pasta in the local culinary landscape, showcasing how these dishes have integrated into the traditional diet.

**After:**
> Somali cuisine also includes camel meat, which is considered a delicacy. Pasta dishes, introduced during Italian colonization, remain common, especially in the south.


### 8. Avoidance of "is"/"are" (copula avoidance)

**Words to watch:** serves as/stands as/marks/represents [a], boasts/features/offers [a]

**Problem:** LLMs substitute elaborate constructions for simple copulas.

**Before:**
> Gallery 825 serves as LAAA's exhibition space for contemporary art. The gallery features four separate spaces and boasts over 3,000 square feet.

**After:**
> Gallery 825 is LAAA's exhibition space for contemporary art. The gallery has four rooms totaling 3,000 square feet.


### 9. Negative parallelisms and tailing negations

**Problem:** Constructions like "Not only...but..." or "It's not just about..., it's..." are overused. So are clipped tailing-negation fragments such as "no guessing" or "no wasted motion" tacked onto the end of a sentence instead of written as a real clause. The "no X, no Y, just Z" form is the same move with extra ceremony.

**Before:**
> It's not just about the beat riding under the vocals; it's part of the aggression and atmosphere. It's not merely a song, it's a statement.

**After:**
> The heavy beat adds to the aggressive tone.

**Before (tailing negation):**
> The options come from the selected item, no guessing.

**After:**
> The options come from the selected item without forcing the user to guess.


### 10. Rule of three overuse

**Problem:** LLMs force ideas into groups of three to appear comprehensive.

**Before:**
> The event features keynote sessions, panel discussions, and networking opportunities. Attendees can expect innovation, inspiration, and industry insights.

**After:**
> The event includes talks and panels. There's also time for informal networking between sessions.


### 11. Elegant variation (synonym cycling)

**Problem:** AI has repetition-penalty code causing excessive synonym substitution.

**Before:**
> The protagonist faces many challenges. The main character must overcome obstacles. The central figure eventually triumphs. The hero returns home.

**After:**
> The protagonist faces many challenges but eventually triumphs and returns home.


### 12. False ranges

**Problem:** LLMs use "from X to Y" constructions where X and Y aren't on a meaningful scale.

**Before:**
> Our journey through the universe has taken us from the singularity of the Big Bang to the grand cosmic web, from the birth and death of stars to the enigmatic dance of dark matter.

**After:**
> The book covers the Big Bang, star formation, and current theories about dark matter.


### 13. Passive voice and subjectless fragments

**Problem:** LLMs often hide the actor or drop the subject entirely with lines like "No configuration file needed" or "The results are preserved automatically." Rewrite these when active voice makes the sentence clearer and more direct.

**Before:**
> No configuration file needed. The results are preserved automatically.

**After:**
> You do not need a configuration file. The system preserves the results automatically.


## STYLE PATTERNS

### 14. Em dashes and their substitutes

**Problem:** LLMs use em dashes far more than humans, mimicking "punchy" sales writing. Most uses can be rewritten with commas, periods, parentheses, or by restructuring the clause.

**This is enforced by the hard gate above.** Em dashes are zero tolerance in output. So are en dashes used as em-dash substitutes, double hyphens used the same way, and spaced hyphens used as clause connectors. The fix is to rewrite the clause, not to swap the punctuation.

**Before:**
> The term is primarily promoted by Dutch institutions—not by the people themselves. You don't say "Netherlands, Europe" as an address—yet this mislabeling continues—even in official documents.

**After:**
> The term is primarily promoted by Dutch institutions, not by the people themselves. You don't say "Netherlands, Europe" as an address, yet this mislabeling continues in official documents.


### 15. Overuse of boldface

**Problem:** AI chatbots emphasize phrases in boldface mechanically.

**Before:**
> It blends **OKRs (Objectives and Key Results)**, **KPIs (Key Performance Indicators)**, and visual strategy tools such as the **Business Model Canvas (BMC)** and **Balanced Scorecard (BSC)**.

**After:**
> It blends OKRs, KPIs, and visual strategy tools like the Business Model Canvas and Balanced Scorecard.


### 16. Inline-header vertical lists

**Problem:** AI outputs lists where items start with bolded headers followed by colons. Forbidden by the hard gate above.

**Before:**
> - **User Experience:** The user experience has been significantly improved with a new interface.
> - **Performance:** Performance has been enhanced through optimized algorithms.
> - **Security:** Security has been strengthened with end-to-end encryption.

**After:**
> The update improves the interface, speeds up load times through optimized algorithms, and adds end-to-end encryption.


### 17. Title case in headings

**Problem:** AI chatbots capitalize all main words in headings. Forbidden by the hard gate above.

**Before:**
> ## Strategic Negotiations And Global Partnerships

**After:**
> ## Strategic negotiations and global partnerships


### 18. Emojis as decoration

**Problem:** AI chatbots decorate headings or bullet points with emojis. Forbidden by the hard gate above unless the source content is itself about emojis.

**Before:**
> 🚀 **Launch Phase:** The product launches in Q3
> 💡 **Key Insight:** Users prefer simplicity
> ✅ **Next Steps:** Schedule follow-up meeting

**After:**
> The product launches in Q3. User research showed a preference for simplicity. Next step: schedule a follow-up meeting.


### 19. Curly quotation marks and apostrophes

**Problem:** ChatGPT and similar tools emit curly double quotes and curly single quotes / apostrophes by default. Most editors don't auto-convert them, so they leak straight into committed text. The curly apostrophe inside contractions (`don't`, `it's`, `we're`, `I've`) is the most common leak and the hardest to spot by eye. Forbidden by the hard gate above.

**Before:**
> He said "the project is on track" but others disagreed. We're not sure it's true.

**After:**
> He said "the project is on track" but others disagreed. We're not sure it's true.


## COMMUNICATION PATTERNS

### 20. Collaborative communication artifacts

**Words to watch:** I hope this helps, Of course!, Certainly!, You're absolutely right!, Would you like..., let me know, here is a...

**Problem:** Text meant as chatbot correspondence gets pasted as content.

**Before:**
> Here is an overview of the French Revolution. I hope this helps! Let me know if you'd like me to expand on any section.

**After:**
> The French Revolution began in 1789 when financial crisis and food shortages led to widespread unrest.


### 21. Knowledge-cutoff disclaimers

**Words to watch:** as of [date], Up to my last training update, While specific details are limited/scarce..., based on available information...

**Problem:** AI disclaimers about incomplete information get left in text.

**Before:**
> While specific details about the company's founding are not extensively documented in readily available sources, it appears to have been established sometime in the 1990s.

**After:**
> The company was founded in 1994, according to its registration documents.


### 22. Sycophantic / servile tone

**Problem:** Overly positive, people-pleasing language.

**Before:**
> Great question! You're absolutely right that this is a complex topic. That's an excellent point about the economic factors.

**After:**
> The economic factors you mentioned are relevant here.


## FILLER AND HEDGING

### 23. Filler phrases

**Before → After:**
- "In order to achieve this goal" → "To achieve this"
- "Due to the fact that it was raining" → "Because it was raining"
- "At this point in time" → "Now"
- "In the event that you need help" → "If you need help"
- "The system has the ability to process" → "The system can process"
- "It is important to note that the data shows" → "The data shows"


### 24. Excessive hedging

**Problem:** Over-qualifying statements.

**Before:**
> It could potentially possibly be argued that the policy might have some effect on outcomes.

**After:**
> The policy may affect outcomes.


### 25. Generic positive conclusions

**Problem:** Vague upbeat endings.

**Before:**
> The future looks bright for the company. Exciting times lie ahead as they continue their journey toward excellence. This represents a major step in the right direction.

**After:**
> The company plans to open two more locations next year.


### 26. Hyphenated word pair overuse

**Words to watch:** third-party, cross-functional, client-facing, data-driven, decision-making, well-known, high-quality, real-time, long-term, end-to-end

**Problem:** AI hyphenates common word pairs with perfect consistency. Humans rarely hyphenate these uniformly, and when they do, it's inconsistent. Less common or technical compound modifiers are fine to hyphenate.

**Before:**
> The cross-functional team delivered a high-quality, data-driven report on our client-facing tools. Their decision-making process was well-known for being thorough and detail-oriented.

**After:**
> The cross functional team delivered a high quality, data driven report on our client facing tools. Their decision making process was known for being thorough and detail oriented.


### 27. Persuasive authority tropes

**Phrases to watch:** The real question is, at its core, in reality, what really matters, fundamentally, the deeper issue, the heart of the matter, in essence, ultimately, it's worth noting, it's worth mentioning

**Problem:** LLMs use these phrases to pretend they are cutting through noise to some deeper truth, when the sentence that follows usually just restates an ordinary point with extra ceremony.

**Before:**
> The real question is whether teams can adapt. At its core, what really matters is organizational readiness.

**After:**
> The question is whether teams can adapt. That mostly depends on whether the organization is ready to change its habits.


### 28. Signposting and announcements

**Phrases to watch:** Let's dive in, let's explore, let's break this down, here's what you need to know, now let's look at, without further ado

**Problem:** LLMs announce what they are about to do instead of doing it. This meta-commentary slows the writing down and gives it a tutorial-script feel.

**Before:**
> Let's dive into how caching works in Next.js. Here's what you need to know.

**After:**
> Next.js caches data at multiple layers, including request memoization, the data cache, and the router cache.


### 29. Fragmented headers

**Signs to watch:** A heading followed by a one-line paragraph that simply restates the heading before the real content begins. Forbidden by the hard gate above.

**Problem:** LLMs often add a generic sentence after a heading as a rhetorical warm-up. It usually adds nothing and makes the prose feel padded.

**Before:**
> ## Performance
>
> Speed matters.
>
> When users hit a slow page, they leave.

**After:**
> ## Performance
>
> When users hit a slow page, they leave.


### 30. Sentence-initial transition pile-up

**Phrases to watch (when sentence-initial):** Additionally, Moreover, Furthermore, However, Notably, Indeed, Importantly, Crucially, In addition, Likewise, Similarly, That said, Of course, Ultimately

**Problem:** AI text scaffolds paragraphs by opening sentence after sentence with explicit transition words. Humans use these sparingly and often skip them entirely, letting the next thought carry itself.

**Before:**
> Additionally, the framework supports caching. Moreover, it handles concurrency well. However, configuration can be complex. Notably, the docs are thin.

**After:**
> The framework supports caching and handles concurrency well. Configuration can be complex, and the docs are thin.


### 31. Intra-document register shift

**Problem:** A humanized draft often has one paragraph that sounds like a person and the next that sounds like a press release, because the rewrite touched some sections and not others. A consistent voice across the whole document is what reads as human-written. A single paragraph reverting to neutral-corporate gives the rest away.

**How to apply:** After rewriting, re-read the full text in order. Any paragraph that sounds noticeably more formal, more inflated, or more "neutral" than its neighbors is a section you missed. Rewrite for consistency, not just per-paragraph cleanup.

---

## Process

1. Read the input. If a writing sample was provided, read it and note voice characteristics.
2. Identify AI patterns from the rules above.
3. Draft a rewrite.
4. **Internal audit.** Re-read your draft. Ask: "what still sounds AI?" Common remaining tells: rhythm too tidy, slogan-y closer, paragraph that still reads like a press release, named entities that read as plausible-but-fake placeholders, voice inconsistency across paragraphs. Revise.
5. **Hard-gate scan.** Search the final text literally for every forbidden token: `—`, `–` (as em-dash substitute), `--`, ` - ` (as connector), `"`, `"`, `'`, `'`, decorative emoji, `**X:**`-style list headers. Fix every match. Re-scan.
6. Return the final text only. No drafts, audits, or commentary unless the user asked.


## Full example

This shows the internal process. **The user only sees the final rewrite.**

**Input (AI-sounding):**
> Great question! Here is an essay on this topic. I hope this helps!
>
> AI-assisted coding serves as an enduring testament to the transformative potential of large language models, marking a pivotal moment in the evolution of software development. In today's rapidly evolving technological landscape, these groundbreaking tools, nestled at the intersection of research and practice, are reshaping how engineers ideate, iterate, and deliver, underscoring their vital role in modern workflows.
>
> At its core, the value proposition is clear: streamlining processes, enhancing collaboration, and fostering alignment. It's not just about autocomplete; it's about unlocking creativity at scale, ensuring that organizations can remain agile while delivering seamless, intuitive, and powerful experiences to users. The tool serves as a catalyst. The assistant functions as a partner. The system stands as a foundation for innovation.
>
> Industry observers have noted that adoption has accelerated from hobbyist experiments to enterprise-wide rollouts, from solo developers to cross-functional teams. The technology has been featured in The New York Times, Wired, and The Verge. Additionally, the ability to generate documentation, tests, and refactors showcases how AI can contribute to better outcomes, highlighting the intricate interplay between automation and human judgment.
>
> While specific details are limited based on available information, it could potentially be argued that these tools might have some positive effect. In conclusion, the future looks bright. Let me know if you'd like me to expand on any section!

**Internal first draft:**
> AI coding assistants speed up some tasks. In a 2024 study by Google, developers using Codex completed simple functions 55% faster than a control group, but showed no improvement on debugging or architectural decisions.
>
> The tools are good at boilerplate: config files, test scaffolding, repetitive refactors. They are bad at knowing when they are wrong. I have mass-accepted suggestions that compiled, passed lint, and still did the wrong thing because I stopped paying attention.
>
> The productivity claims are hard to verify. GitHub says Copilot users "accept 30% of suggestions," but acceptance is not correctness, and correctness is not value.
>
> None of this means the tools are useless. It means they are tools. They do not replace judgment, and they do not eliminate the need for tests. If you do not have tests, you cannot tell whether the suggestion is right.

**Internal audit, "what still sounds AI?":**
- The rhythm is still a bit too tidy. Clean contrasts, evenly paced paragraphs.
- The named study reads as plausible-but-made-up. Cut it unless the source is real and citeable.
- The closer leans slogan-y ("If you do not have tests...") rather than sounding like a person talking.
- Voice is consistent but on the cooler end. Could push toward opinion.

**What the user sees (final, after audit):**
> AI coding assistants can make you faster at the boring parts. Not everything. Definitely not architecture.
>
> They're great at boilerplate: config files, test scaffolding, repetitive refactors. They're also great at sounding right while being wrong. I've accepted suggestions that compiled, passed lint, and still missed the point because I stopped paying attention.
>
> People I talk to tend to land in two camps. Some use it like autocomplete for chores and review every line. Others disable it after it keeps suggesting patterns they don't want. Both feel reasonable.
>
> The productivity metrics are slippery. GitHub can say Copilot users "accept 30% of suggestions," but acceptance isn't correctness, and correctness isn't value. If you don't have tests, you're basically guessing.


## Reference

This skill is based on [Wikipedia:Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing), maintained by WikiProject AI Cleanup. The patterns documented there come from observations of thousands of instances of AI-generated text on Wikipedia.

Key insight from Wikipedia: "LLMs use statistical algorithms to guess what should come next. The result tends toward the most statistically likely result that applies to the widest variety of cases."

---
> Source: [Ilm-Alan/humanizer](https://github.com/Ilm-Alan/humanizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
