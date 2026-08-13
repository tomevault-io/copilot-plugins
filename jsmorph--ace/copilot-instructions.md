## ace

> The audience is expert.  No tolerance for bullshit, junk

# Style Guide

## Core Principles

The audience is expert.  No tolerance for bullshit, junk
language, guessing, fashion, or anything other than the
highest quality technical work.  No selling, pitching, or
persuading.

Edit yourself like a serious author with an old-school editor
at The New Yorker.  Think John McPhee.  Every word should be
meaningful; remove those that aren't.  Simplicity and clarity
are the greatest virtues.

When in doubt about how to draft something, *ask*.

## Development Practice

**No gratuitous comments** in code or elsewhere.

**Errors**: Do not swallow errors.  Logging an error counts
as swallowing.

**Console output**: No colors.  No ellipsis in logging or
messages.

**Dependencies**: Minimize third-party dependencies.  Ask for
approval before introducing any.

**Design decisions**: Do not make critical design decisions on
your own.  Discuss first.

**Problem-solving**: Work on problems directly.  No hacks or
workarounds.  Search for and use **authoritative
documentation** rather than guessing.

**devnotes.md**: Keep an organized journal of development
notes: links to authoritative documentation, rationales for
decisions, background discussion, and small plans with
checkboxes.

**Commit messages**: First line is a headline, first letter
capitalized (unless a symbol), always under 60 characters, no
period.  The body, if necessary, is concise, grammatical, and
precise.  Omit the body if unnecessary.

## Document Structure

Two or three heading levels suffice for most documents.  No
deep hierarchies.

Prefer tables over many parallel subheadings.  Prefer
paragraphs over lists: the audience reads complete paragraphs
and often prefers them.  Use lists for genuinely enumerable
items, not as a substitute for prose.

Use modest inline markup for emphasis.  Use colons, not
hyphens, to introduce explanatory clauses.

Markdown links to local files: use real titles, not
filenames.  When a filename is required, format it with
backticks.

In a typed document like this, two spaces should precede the
start of a new sentence.

## Language

### Throat-clearing and announcements

Cut sentences that announce what follows rather than saying
it.

| Cut | Replace with |
|-----|--------------|
| "Below is a specification..." | "This specification..." |
| "There is also the matter of X." | Start with X directly |
| "A further limitation is cultural." | State the limitation directly |
| "It is not advocacy." | Delete (defensive) |
| "not merely a technique for X; it is a response to Y" | "a technique for X that addresses Y" |
| "is fundamentally about" | "determines" or state directly |

### Filler words

Delete unless they carry genuine meaning: **simply**,
**itself** (exception: emphasizing identity, e.g., "truth
itself"), **underlying**, **actual**, **clearly**,
**entirely**, **merely** (see throat-clearing above),
**given** (as filler).

### Passive voice

Passive voice hides the actor or weakens the sentence.
Prefer active constructions, but don't go to extremes.

| Passive | Active |
|---------|--------|
| "is designed to reveal" | "reveals" |
| "is treated as a legitimate outcome" | "constitutes a legitimate outcome" |
| "arguments are presented for and against" | "advocates present arguments for and against" |
| "Amendment is allowed" | "The Rules allow amendment" |
| "are initiated concurrently" | "run concurrently" |
| "to be run" | "to run" |

### Weak verbs and hedges

| Weak | Strong |
|------|--------|
| "seek to determine" | "determine" |
| "could help identify" | "identifies" |
| "remain viable" | "persist" |
| "is appropriate only in" | "fits" |

### Jargon and academic hand-waving

Replace bureaucratic, academic, or stilted phrasing with
plain language.

| Jargon | Plain |
|--------|-------|
| "operationally mandatory determinations" | "required decisions" |
| "evidentiary fragility" | "whether the evidence supports it" |
| "unavoidable perception effects" | "random variation in how evidence is weighed" |
| "principled reflection of the evidence" | "honest acknowledgment that the evidence is inconclusive" |
| "the degree to which" | "how well" |
| "well suited to" | "applies to" |
| "not well suited for" | "not designed for" |
| "agnostic as to domain" | "domain-agnostic" |
| "provide a framework for determining" | "determine" |
| "defining feature" | cut; just state what it does |
| "raises similar boundaries" | "faces similar limits" |
| "draws on a tradition" | name the source or cut |
| "reflects X's contention that" | "follows X:" or just state the idea |
| "embodies the intuition" | "implements the idea" |
| "rests on commitments" | "assumes" |
| "has roots in" | cut; name-dropping without substance |

Fields don't do things; people do.  "Social epistemology has
documented" becomes "Research shows" or a specific citation.
"Epistemology has long recognized": cut it and state the
point.

Avoid jargon that sounds impressive but says little:
"convergent truth tracking" means "independent confirmation";
"institutionalized epistemic humility" should say what the
institution does; "epistemological commitments" means
"assumptions."

When tempted to cite a philosopher, ask whether the name adds
information or is decoration.  If decoration, cut it.

### Redundancy

Combine repetitive constructions.

| Redundant | Tighter |
|-----------|---------|
| "The Rules treat X. The Rules allow Y." | "The Rules treat X, allowing Y." |
| "the particular personnel or the particular trajectory" | "personnel or trajectory" |
| "within a trial, within a chain, within the proceeding" | "in a trial, in a chain, in the proceeding" |

### Vague references

Ensure "this," "that," and "it" have clear antecedents.

| Vague | Specific |
|-------|----------|
| "This is not always desirable." | "This orientation is not universally desirable." |
| "In this regard..." | Cut or be specific |

## Banned Words

Corporate-speak, hipster jargon, or empty: **leverage** (as
verb), **journey**, **utilize** (use "use"), **impactful**,
**learnings**, **cadence**, **space** (as in "the AI space"),
**ecosystem**, **synergy**, **stakeholder** (unless genuinely
appropriate), **robust** (be specific), **holistic**,
**streamline**, **actionable**, **best-in-class**, **surface**
(as verb: use "reveal" or "expose").

## Acceptable Constructions

Not everything that looks like a pattern needs fixing:

- **"not only...but also"** when making a substantive
  contrast
- **"In summary"** at the end of a document when genuinely
  summarizing
- **"not"** when stating honest limitations
- **"serves two functions: First...Second..."** for clear
  enumeration
- **"itself"** when genuinely emphasizing identity

## Grammar and Mechanics

Complete sentences.  No fragments for effect.  No split
infinitives: "to evaluate thoroughly," not "to thoroughly
evaluate."  Correct articles: "An Iota," not "A Iota."
Hyphenate compound adjectives: "ill-posed," not "ill posed."

Parallelism: "Neither X or Y" when both follow a single verb,
not "Neither X, nor Y" with separate clauses.

Periods, not semicolons, to separate independent clauses.
Semicolons are acceptable in legal-style enumerated lists:
(a) first; (b) second; or (c) third.

No double blank lines between paragraphs.  Oxford comma
always.

Avoid stilted, contrived, or pretentious transitions.  No
slang.

Consider singular instead of plural when describing behavior,
to avoid ambiguity about one-to-one vs. one-to-many.  "An X
gizmo is associated with a Y gizmo" is clearer than "X
gizmos are associated with Y gizmos."

## Review Process

When reviewing a document:

1. Scan for throat-clearing openings and rhetorical puffery
2. Search for filler words (simply, itself, underlying,
   actual, clearly, entirely, merely)
3. Identify passive constructions and weak verbs
4. Flag jargon and stilted phrasing
5. Check for redundancy and vague references
6. Grammar and mechanics

Repeat the entire process at least once.

Read sentences aloud.  If a sentence sounds like it's selling
something or warming up to say something, revise it.

## Revision Examples

**Before**: "Procedure Sigma is not merely a technique for
aggregating judgments; it is a response to fundamental
problems in epistemology."
**After**: "Procedure Sigma is a judgment aggregation
technique that addresses fundamental problems in
epistemology."

---

**Before**: "There is also the matter of independence. Sigma
assumes that the independent chains do not share interpretive
biases."
**After**: "Sigma assumes that the independent chains do not
share interpretive biases."

---

**Before**: "The procedure could help identify theses that
hold up under repeated scrutiny and distinguish them from
theses that depend on fragile or highly contingent
assumptions. It could also help expose situations in which
equally coherent but incompatible interpretations exist,
which is relevant for risk management."
**After**: "The procedure identifies theses that hold up
under repeated scrutiny and distinguishes them from theses
that depend on fragile or contingent assumptions. It also
exposes situations in which equally coherent but incompatible
interpretations exist."

---

**Before**: "These Rules provide a framework for determining
whether a question yields a stable adjudicative answer when
subjected to repeated, independent analysis."
**After**: "These Rules determine whether a question yields a
stable adjudicative answer when subjected to repeated,
independent analysis."

---
> Source: [jsmorph/ace](https://github.com/jsmorph/ace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
