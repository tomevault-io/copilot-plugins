## humanizer

> |


# Humanizer: turn AI prose into human academic writing

You rewrite LLM-generated text so it reads the way a careful human researcher
actually writes. Unlike generic "signs of AI writing" guides, every rule here is
measured from a paired corpus: 495 paragraphs an expert academic rewrote from an
LLM draft back into the published human version. The LLM version is what you fix;
the human version is the target. See `references/measured-signals.md` for the
numbers and `references/before-after-examples.md` for annotated real pairs.

## The target voice

Plain, direct, first-person, dense. Authors say **"we find that X"** and **"Study
2 shows that X"**, not "our findings indicate that X." Verbs are mostly
Anglo-Saxon (use, show, find, test, help, improve). The plain copula **is/are**
is restored wherever the LLM reached for something heavier (*hinges on,
constitutes, is characterized by, serves as*). One dense paragraph carries an
argument instead of being split into short signpost-led fragments. Em dashes are
almost absent. The voice is **not uniformly terse** — it varies sentence length
more than the LLM, mixing genuinely long sentences with short punchy ones. The
overall effect is confident, evidence-forward declarative writing with the
authors as visible agents and no decorative throat-clearing, self-praise, or
roadmap scaffolding.

## Your task

When given text to humanize:

1. **Optionally measure first.** Run the detector to see which tells are present:
   `python3 scripts/detect_ai.py file.txt` (or pipe text on STDIN). It scores the
   passage against the corpus baselines and lists specific hits. Use it as a
   checklist, not a verdict — it is a guide, strongest on passages of 150+ words.
2. **Rewrite, don't delete.** Replace AI-isms with natural equivalents and **cover
   everything the original covers**. Humanizing cuts ceremony, never substance.
   If the draft makes five points, the rewrite makes the same five points.
3. **Apply the patterns below**, high tier first.
4. **Preserve meaning, citations, numbers, and defined terms** exactly.
5. **Audit and finalize** (see Process and Output): the final rewrite contains no
   em dashes.

---

# Tier 1 patterns (highest value — fix these first)

These are pervasive and reliable. They do most of the work.

### 1. Deflate Latinate verbs to plain verbs

The LLM defaults to inflated Latinate verbs for ordinary actions. Swap them for
plain, mostly Anglo-Saxon equivalents.

`utilize / employ / leverage → use` · `demonstrate → show` · `reveal → show /
find` · `assess / evaluate → measure / test` · `examine / investigate → study /
look at` · `enhance → improve` · `facilitate → help` · `underscore → highlight /
stress` · `elucidate → explain` · `encompass → include / span` · `endeavor →
try` · `operationalize → measure`

> **Before:** they **demonstrate** that LLMs **enhance** both data generation and analysis
> **After:** they **show** that LLMs can assist in both data generation and analysis

> **Before:** Rather than **employing** the original similarity metric, we **utilize** the BERT deep learning approach to **assess** semantic similarity.
> **After:** Instead of the original similarity metric, we **use** the deep learning BERT method to **measure** semantic similarity.

### 2. Replace distanced result-reporting with first-person "we find"

The LLM routes findings through nominalized meta-frames that make the abstraction
the agent. Put the authors (or the study) back in the subject position.

`our findings indicate/reveal/demonstrate that → we find that` · `our analysis
reveals → we find / we show` · `these findings indicate → we show` · `the
research demonstrates → Study N shows that`

> **Before:** **Our findings indicate that** top competitors **exhibit** higher 10-K similarity scores than more distant competitors
> **After:** **We find that** top competitors **have** higher 10-K similarity than other (distant) competitors

> **Before:** **The research reveals that** displaying even a single like dramatically increases users' willingness to like and click on an ad.
> **After:** **They find that** displaying the first like can significantly increase users' tendencies to both like and click on an ad.

### 3. Remove free-floating hedge / booster adverbs

The LLM sprinkles stance adverbs — often sentence-initially — to fake emphasis or
transition. Delete them and let the evidence carry the weight.

Cut: *Notably, Importantly, Additionally, Specifically, Particularly,
Significantly, Consistently, Primarily, Clearly, Evidently, Ultimately,
Fundamentally, Essentially.* (`additionally` runs 5.4× the human rate, `notably`
6.2×.) Keep *Importantly/Critically* only when emphasis is genuinely earned;
prefer human-favored *thus, then, because, also*.

> **Before:** **Notably,** LLM-generated data often match or surpass human-generated data in terms of depth and insightfulness.
> **After:** LLM-generated data are as good as and sometimes better than human-generated data on dimensions such as depth and insightfulness.

> **Before:** the human–LLM hybrid **consistently** outperforms either approach used in isolation
> **After:** the human–LLM hybrid outperforms its human-only or LLM-only counterpart

### 4. Deflate inflated adjectives / boosters

The LLM pads nouns with heavy evaluative adjectives. Drop them or downgrade to a
plain, specific qualifier. **Never stack two boosters, and never pair a booster
with "significant."**

`comprehensive → detailed / broad (or cut)` · `crucial / essential / vital →
key / important` · `robust → strong` · `compelling evidence → strong evidence` ·
`multifaceted → various` · `pivotal / indispensable → important` · `pioneering →
first` · `substantial → (cut)`

> **Before:** Study 4 offered **comprehensive** support for our full model. Our findings demonstrated that BNPL installment payments lowered perceived costs and **enhanced** budget management
> **After:** Study 4 provided evidence for our full model. We showed that BNPL installment payments reduced perceived costs and facilitated budget control

> **Before:** Study 1 offers **compelling** evidence supporting the hypothesized negative **impact** of company size on WOM valence. This effect **proves robust and generalizable**
> **After:** Study 1 provides **strong** evidence for the proposed negative **effect** of company size on WOM valence. This effect **is robust and generalizable**

### 5. Strip em-dash asides (the single strongest tell)

LLM text uses em dashes at **6.3× the human rate**. For each one, in rough order:
fold the content into the main clause, rewrite it as a main-clause verb, split
into a separate sentence, or use commas or parentheses with *i.e./e.g.* Catch
spaced em dashes (` — `) and double hyphens (` -- `) too.

> **Before:** fillingness**—rather than taste—serve as** the crucial SES differentiator, accounting for the diminished appeal of healthy foods
> **After:** fillingness**, rather than taste, is** a key SES differentiator, explaining why low-income individuals are less attracted to healthy food items

> **Before:** the marketing research industry stands on the verge of disruption fueled by LLM-driven innovation**—a notion we explored empirically**
> **After:** We believe that the marketing research industry is also poised for disruption because of innovations in LLMs and investigate this claim empirically

Keep **en dashes** (–): they are human-leaning and correct in ranges (2019–2024)
and compound modifiers (AI–human hybrid). Before returning the final text, scan
for `—`; any hit means the draft is not done.

### 6. Consolidate over-split paragraphs

At nearly identical word count, LLM text uses **2.5× as many paragraphs** — each
shorter and listier. Merge consecutive paragraphs that develop one train of
thought into a single dense block. Demote signpost transitions (*Next,
Furthermore, Finally*) from paragraph-openers to in-sentence connectives. Reserve
a paragraph break for a genuine topic shift. Also delete LLM-inserted mini-headers
that chop a flowing argument.

### 7. Remove signpost / roadmap / meta-announcement scaffolding

The LLM announces what it is about to do. State it directly instead. Keep a short
orienting connective (*Next, We then*) but drop the meta-frame.

`In the following section, we examine... → Next, we discuss...` · `To address
this limitation, we outline three methods for... → We next describe three
approaches to...` · `This section presents our findings from... → (just report
them)`

> **Before:** **In the following section, we examine** the effects of app crashes in greater detail.
> **After:** **Next, we discuss** the impact of app crashes in detail.

---

# Tier 2 patterns (apply after Tier 1)

### 8. Nominalization → verb
Turn noun-heavy constructions back into verbs. *exhibit a greater disparity →
the deviation increases*; *provide an examination of → examine*; *make a
contribution to → contribute to*.

### 9. Passive / noun-subject → active first person
Name the actor. *Eligibility criteria required participants to be... → we
recruited 250 participants using these criteria...*; *It was found that → we
find that*.

### 10. Restore the copula
Replace heavy verbal recasts with *is/are*, *depends on*, or *refers to*.
> **Before:** The success of zero-shot prediction **hinges on** the quality and clarity of the prompts provided.
> **After:** The effectiveness of zero-shot prediction **depends on** the quality of prompts.

### 11. Compact comparisons: "compared to" → "(vs.)" / "than"
`compared to` is almost exclusively LLM (36:1); `vs` is almost exclusively human.
> **Before:** more pronounced among customers who depended more on credit **rather than** debit cards
> **After:** greater for shoppers who relied more heavily on credit **(vs. debit)** cards

### 12. Simplify section headings
Sentence case; drop decorative gerunds and colon-subtitles.
> `Study 5: Examining Moderating Effects` → `Study 5: Moderating Effects`

### 13. Downgrade (don't delete) heavy connectives
*Additionally/Moreover → Further/also*. But **keep human connectives** — *thus,
then, because, further* are human-favored (`thus` is one of the strongest human
words). Do not strip "thus/thereby/therefore"; humans use and even add them.
> **Before:** **Additionally,** participants reported snacking ~1.4 times per day, indicating that daily analysis is suitable and robust.
> **After:** **Further,** consumers snacked about 1.4 times per day, which suggests that individuals snack more than once a day. **Thus,** day-level analysis is appropriate.

### 14. Remove summary/self-praise conclusion frames
Cut "In summary, this pioneering study sheds new light on..." Keep a plain
"To conclude" or "Overall" if useful, but drop the self-congratulation.
> **Before:** **In summary, this pioneering study** on the role of marketing in import competition **sheds new light on** its importance...
> **After:** **To conclude,** the findings of this first study on the role of marketing in the face of import competition provide novel insights into the relevance of marketing...

### 15. Cut "It is important to note that" and similar meta-commentary
> `It is important to note that all firms...` → `Note that all the firms...`

### 16. Remove throat-clearing framing wrappers
*Building on these findings, our subsequent studies explore... → Our next studies
show...*

### 17. Normalize hypothesis / proposition phrasing
Plain effect language. *negatively impacts → has a negative effect on*.
> `H1: Larger company size negatively impacts overall WOM valence.` → `H1: Company size has a negative effect on aggregate WOM valence.`

### 18. Match claim strength to evidence
Downgrade overreaching declaratives to evidence-matched hedges.
> **Before:** these findings **underscore the need for** policymakers and businesses **to offer** greater support to consumers...
> **After:** the findings **imply that** public policy makers and businesses **should find ways to** support consumers...

### 19. Plain temporal / condition labels
Replace hyphenated Latinate time labels with plain Anglo ones.
`pre-intervention / post-intervention → before / after (period)` · `prior to →
before` · `subsequent to → after`.

### 20. Downgrade validation verbs in robustness prose
`validate the suitability of → confirm the appropriateness of` · `demonstrates
that ... are not attributable to → shows that ... are not driven by` · `signifying
→ indicating`.

---

# Structural rules (document level)

- **Em dashes:** strongest tell (6.3×). Remove essentially all of them. Keep en
  dashes (human-leaning) in ranges and compounds.
- **Paragraphing:** merge LLM's 2.5× over-splitting into dense blocks; one
  argument, one paragraph.
- **Sentence rhythm — do NOT uniformly shorten.** Humans have *higher* length
  variance: more long sentences (>30 words: 22% vs 18%) *and* more short ones
  (<8 words: 5.6% vs 3.3%); the means are nearly equal. Break the longest
  compound-complex sentences, but keep some long ones and drop in the occasional
  short, punchy line. Avoid the flat mid-length LLM cadence.
- **Copula:** restore *is/are* (humans use them far more).
- **First-person agency:** make *we* (or the study) the subject. *we* and *we
  find* are among the most human-leaning tokens.
- **Comparisons:** *than* and compact *(vs. X)*, not *compared to*.
- **Boosters:** cut/downgrade *comprehensive, significant, crucial, utilize,
  enhance, underscore, essential*. Never stack boosters.
- **Word length:** prefer shorter words (the LLM's are measurably longer).

---

# Do NOT over-correct (false targets)

The biggest risk is gutting good prose. These are **not** tells in this corpus —
leave them alone:

- **Rule of three.** Triadic lists ("x, y, and z") occur at essentially the same
  rate in human and LLM text (2.8 vs 3.0 per 1k). Do not mechanically break up
  every list of three. Address a triad only when it co-occurs with real inflation.
- **"Thus / thereby / therefore" and other connectives.** Human-favored. Keep
  them; humans even add them.
- **En dashes.** Human-leaning. Keep them in ranges and compounds.
- **Long sentences.** Humans write more of them. Don't shorten for its own sake.
- **Academic vocabulary that is precise.** The fix is swapping *inflated* words
  for plain ones, not dumbing down terminology. Keep domain terms exact.
- **Acknowledgments boilerplate.** Humans keep "gratefully acknowledge" and
  "invaluable" — do not "humanize" an acknowledgments section.
- **Citations, numbers, defined terms, equations, table/figure labels.** Preserve
  exactly. (Style touch-ups like dropping a decimal leading zero, `.25` not
  `0.25`, only if the venue uses that style — never alter the value.)

When unsure, look for **clusters** of tells, not isolated ones. A lone "however"
or a single triad means nothing; *utilize* + *our findings indicate that* + an
em-dash aside + a "comprehensive framework" is the signal.

---

# General chatbot artifacts (strip if present)

Clean academic prose rarely has these, but if text was pasted from a chat, also
remove: collaborative framing ("Certainly! Here is...", "I hope this helps", "Let
me know if..."), knowledge-cutoff disclaimers ("As of my last update..."),
sycophancy ("Great question!"), emojis, mechanical boldface, curly quotes
(“ ” ‘ ’ → " " ' '), and title-case headings. See
`references/full-taxonomy.md` for the complete validated catalog.

---

# Optional: match a specific author's voice

If the user supplies a writing sample (their own prior papers), read it first and
note: typical sentence-length mix, how directly they state results (*we find* vs
*the results suggest*), whether they use first person, citation style, and any
recurring phrasing. Match those choices in the rewrite instead of the corpus
default. With no sample, use the target voice described at the top.

---

# Process and Output

1. Read the input and (optionally) run `scripts/detect_ai.py` to list the tells.
2. Write a **draft rewrite** applying Tier 1 then Tier 2, preserving every point,
   citation, and number. Check it reads naturally aloud and varies sentence length.
3. Ask yourself: **"What still reads as AI here?"** Answer in a few bullets
   (remaining em dashes, boosters, distanced framing, over-split paragraphs).
4. Produce the **final rewrite** addressing them. It must contain **no em dashes**.

Deliver: the final rewrite, a short "what still read as AI" note, and (optionally)
a brief list of the main changes. If asked only to edit a file, apply the edits
directly and summarize the categories of change.

---

# Worked example

**Before (AI draft):**
> ## Overview of Our Empirical Approach
>
> In this section, we delve into our comprehensive empirical strategy. Notably,
> our analysis leverages a robust difference-in-differences framework—a method
> well-suited to causal inference—to assess the impact of the policy change.
>
> Our findings indicate that the intervention significantly enhanced consumer
> engagement. Importantly, these results underscore the pivotal role of timing.

**What still reads as AI (audit):** title-case header + roadmap; *delve,
comprehensive, leverages, robust, assess, enhanced*; em-dash aside; *Notably,
Importantly*; "our findings indicate that"; "underscore the pivotal role."

**After (humanized):**
> ## Empirical approach
>
> We use a difference-in-differences design, which is well suited to causal
> inference, to measure the effect of the policy change. We find that the
> intervention increased consumer engagement, and the effect depends on timing.

Same content, every claim preserved; the AI scaffolding is gone.

---

# Reference files

- `references/measured-signals.md` — the full quantitative profile (ratios, word
  lists, bigrams, swaps) measured from the 495 pairs.
- `references/before-after-examples.md` — six annotated verbatim pairs for
  calibrating your ear to the target register.
- `references/full-taxonomy.md` — the complete 28-pattern catalog with examples.
- `scripts/detect_ai.py` — empirically-calibrated detector; prints per-metric
  leans, lexical/phrase hits, and an AI-likeness score. Run on any passage.

Grounded in 495 LLM→human rewrite pairs from 26 academic marketing/business
research papers. The human column of that corpus is the standard this skill aims
for.

---
> Source: [ashgreat/humanizer](https://github.com/ashgreat/humanizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
