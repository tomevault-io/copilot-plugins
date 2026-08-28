## wasl

> **New here? Read [HANDOFF.md](HANDOFF.md) first** — state, map, traps, open threads.

# CLAUDE.md — working rules for Wasl

## What this is

**New here? Read [HANDOFF.md](HANDOFF.md) first** — state, map, traps, open threads.
This file is the rules.
**Wasl** (وصل, "the link") — a verifiable genealogy (nasab) explorer. Source of truth is two JSONL files; the HTML is
generated output. Everything asserted must be traceable to an Arabic primary text.

## Non-negotiable rules

1. **No uncited assertion.** Every parent edge, name, alias, kunya and date lives in
   `claims.jsonl` with `work`, `ed`, `vol`, `page`, `loc`, `ar` and `en`. A person row in
   `people.jsonl` carries identity only — no relationships, no dates.
2. **Quotes must be verbatim from `corpus/`.** `validate.py` re-reads the cited page in the
   pinned OpenITI text and fails if the Arabic string is not there. Never hand-type Arabic
   from memory; copy it out of the corpus file.
3. **Never resolve a disagreement by deleting one side.** If sources differ, add every
   reading as its own claim and link them with `variant_of`. The UI shows all of them.
4. **Never invent a date.** Every date claim carries `date_basis`:
   `attested` | `attested_relative` | `derived_from_age_at_death` | `generation_estimate` |
   `unknown`.
   If the source gives no year, the field stays empty. A blank is correct; a guess is not.
5. **New source ⇒ new row in `sources.tsv`** with version URI, URL, editor and edition, and a
   re-run of `fetch.sh`. No claim may cite a work absent from `sources.tsv`.
6. **Model proposes, script verifies, human approves.** An LLM may draft rows. `validate.py`
   decides whether they are real. Never let a draft land without a passing validate.
7. **Source files carry `SPDX-License-Identifier: GPL-3.0-or-later`.** Keep the header when
   adding a file; see LICENSING.md for why the data and the corpus are not the same thing.
8. **`validate.py` shares code with `nasab.py`, so it can agree with its own bug.**
   `test_wasl.py` re-derives page boundaries from the raw file with plain string operations and
   confirms the checker rejects fabricated quotes and wrong pages. Run both. This is not
   theoretical: it is how the repeated-page-marker bug in Ibn Sa'd was found.

## Workflow
```
./fetch.sh          # pull pinned corpus into corpus/ (gitignored, checksummed)
python3 validate.py  # MUST pass before every commit - proves every quote against the corpus
python3 test_wasl.py # independent checks: re-derives page boundaries without nasab.py
python3 test_parsers.py # focused parser regression checks
python3 build.py     # regenerate index.html
python3 tools/build_entries.py --write   # re-pin the biographical entries
python3 tools/build_summaries.py --write # check the anchors, write summaries.jsonl
python3 tools/build_bios.py              # bio/*.html - CI-built, never committed
```

`bio/` is gitignored. The biography pages carry a quarter of a million words of
OpenITI's Arabic, so CI builds them from the fetched corpus and deploys from the artifact;
the staging step deletes `corpus/` and fails if it survives.

`./fetch.sh` verifies the committed checksum by default. Only
`./fetch.sh --refresh-checksums` may deliberately replace the pin after a reviewed source update.

## English references: tried, and dropped

Guillaume's translation of Ibn Hisham was pinned and then removed. Keep it removed unless a
better source appears, and do not repeat the attempt on the same scans:

- The archive.org scan is **431 leaves for 813 pages** - two printed pages per image - so page
  boundaries cannot be recovered from it at all. Any page number derived by locating text
  would be a guess wearing a citation's clothes.
- Guillaume's own index of proper names was the way round that, since the page numbers are
  then the book's rather than ours. But the OCR recovers about 40% of it, and the stretch
  under 'ayn is essentially gone - which is where 'Ali, 'A'isha, al-'Abbas, 'Abd al-Rahman
  and 'Abd al-Muttalib all live. 25 of 47 people at best.
- The OCR runs entries together on one line, so "'Abdul-Malik b. Hassan, 416, 499* 'Abdu
  Yalil b. 'Amr, 614-15" silently gave one man the other's pages until the parser was taught
  to stop at a name.
- **The check could not have caught the worst of it.** Pinning the headword `Husayn` returned
  `Husayn, al, b. al-Humam (P), 43` - a poet - and every automated test passed, because
  re-reading the index line proves the line exists, not that it is the right person's. A
  citation apparatus whose errors are invisible to its own checker is worse than none.

The other English translations need no scan and are honest as plain bibliography: Bewley's
*The Women of Madina* is Ibn Sa'd's volume of women (abridged), Moinul Haq and Ghazanfar
cover volumes I-II, and Landau-Tasseron's *al-Tabari* vol. 39 is a biographical dictionary.
Usd al-Ghaba and al-Isti'ab - which carry most of our entries - have no English translation
at all. None of that can be machine-verified, so if it is ever added it goes in as
bibliography, from a copy in hand, and labelled as such.

**A page milestone can have four digits.** `PAGE_RE` read `\d{3}` and al-Isti'ab paginates
1..1969 in one run across its four volumes, so 286 published claims named a page a tenth of
the true one. Every quote still verified, because the claim and the checker read the same
truncated index and agreed. `test_wasl.py` now re-derives the last page of every volume with
a plain scan. A citation nobody can look up is the failure this project exists to prevent.

**An entry heading is not a unique key.** `locate()` is sound for a claim, whose quote is
long and distinctive, and unsound for a heading: 'Umar b. al-Khattab' as a string occurs on
hundreds of pages of Usd al-Ghaba, so searching for it put his entry on page 13 spanning 665
pages. Entry spans come from milestone POSITIONS in the file, never from a text search. And
Ibn Sa'd files one man once per tabaqa, so a heading can legitimately repeat - `tools/
entries.py` pins with `=exact` and `#ordinal` and refuses to guess between matches.
`index.html` is generated and committed so the repo is browsable without a toolchain. Never
hand-edit it — edit `build.py`, `template.html`, or the JSONL.

Palette (gold into deepening green) lives in `PALETTE` in `build.py`; the 10-fold girih rosette
is generated by `rosette()` there, not stored as an asset.

## Conventions
- Person ids: `p.` + lowercase ALA-LC-ish slug, hyphens (`p.abd-al-muttalib`).
- Claim ids: `c` + 5 digits, never reused after deletion.
- Transliteration: ALA-LC with diacritics (`ʿAbd al-Muṭṭalib`, `Muḥammad`).
- Arabic strings in JSONL are stored exactly as they appear in the corpus, minus mARkdown
  markers (`#`, `~~`, `PageV..`, `ms\d+`) and with newlines collapsed to single spaces.
- Commit at every stage. Message body states what was verified, not just what changed.

## Extraction rules (tools/)

The books are written in a small number of fixed shapes, so one parser opens many of them:
`fa-walada X: A, B, C` covers Ibn Hazm and Ibn al-Kalbi almost entirely, and much of
al-Baladhuri and Ibn Sa'd. `tools/extract_walad.py` reads that shape; `tools/ingest.py` mints
people and refuses any claim the corpus does not carry.

Three rules keep automated extraction from inventing relatives. All were added because the
draft violated them and the sample caught it:

1. **A chain resolves only if the WHOLE chain matches one path already in the tree.** Matching
   a suffix hung Qahtani clans (Sa'd Hudhaym, Ka'b b. al-Khazraj) under Quraysh, because the
   resolver latched onto any `Zayd` it could find. A partial match is a wrong answer, so it
   must be no answer.
2. **Ask the corpus how ambiguous its own phrase is** (`continuations()`). `Qusayy b. Kilab` is
   only ever continued `b. Murra` - one man. `Muhammad b. Abd Allah` is continued 32 different
   ways in Ibn Sa'd alone, so a bare mention of it identifies nobody. Without this test, three
   other men's sons were hung on the Prophet. Note it correctly rejects `Hashim b. Abd Manaf`
   too - Ibn Hazm records two of them.
3. **Scope each pass with `under=`.** A pass grows outward from a chosen trunk, so a phase has
   a boundary that can be stated: Phase 2 is `under=p.fihr`, which Ibn Hazm defines as exactly
   the set of people called Qurashi.

Recall is deliberately sacrificed to precision. Statements the rules reject are counted and
reported, never guessed at. `store.person(name, father=...)` treats identity as (name, father),
and `child_of` also matches recorded aliases, so `Amir wa-huwa Mudrika` resolves to the existing
Mudrika instead of minting a twin.

**A probe must begin a chain.** Any search for a person by their chain will otherwise match
inside longer chains and attribute whatever follows to the wrong man - this is how Fihr b. Malik
was briefly given Umar's son's kunya. Check the preceding characters for `bn `.

**Honorifics are not names, and a chain link is not always `bn`.** `rasul Allah wa-sayyid walad
Adam` is one man under two epithets - splitting it on the `wa-` invented a son for the Prophet's
father. `bint` and `ibna` are chain links exactly as `bn` is; leaving them out of the splitter
hung granddaughters on their grandfathers and sexed them male. Both bugs reached the Prophet's
immediate family. `test_wasl.py` now asserts that family name by name, and that no name may
contain an honorific or be an unsplit chain.

**When a parser is wrong, fix the parser and REPLAY.** Patching the output leaves the same class
of error everywhere else in the 3,800 nodes you did not look at. Data resets cleanly to the
hand-seeded Phase 3 commit; phases 2, 5, 6, 6b, kunya, merge, prune and retranslit replay over
it in about ten minutes.

**A chain in the text is often broken by an honorific** - `Umar b. al-Khattab, radiya Allahu
anhu, ibn Nufayl`. A probe that assumes contiguity will miss the most famous men in the corpus.

**Always sample the output before writing.** Every bug above was invisible in the totals and
obvious in fifteen sampled lines. The recurring failure is a plausible-looking name attached to
the wrong man; totals cannot show it and `validate.py` cannot either, because the quote is
genuinely on the page. Only reading the lines shows it.

**Seed a spine by hand before parsing into new territory.** Parsers grow outward from a correct
backbone; they cannot find one. Phase 5 hung al-Aws on a `al-Harith b. Qahtan` until the real
Qahtan-to-Ansar chain was seeded from Ibn Hisham.

**Watch the cost of a scan inside a resolver.** `aliases_of` walked every claim per person,
making chain resolution O(people x claims) - a ten-minute hang that became 0.7s per pass once
indexed. Anything called per-statement must be O(1).

## What is proven and what is inferred

`validate.py` proves the QUOTE: the Arabic really is on the page cited. It cannot prove the
PLACEMENT: that the man the chain anchored to is the man the text meant. Parser-placed claims
carry `source_pattern` and their nodes are badged `auto` in the page. Never describe an `auto`
node as verified without that distinction - in the docs, in commit messages, or to the user.

## The replay pipeline

The data is generated. When a PARSER is wrong, do not patch people.jsonl - reset and replay,
or the same class of error stays in the thousands of nodes you did not inspect. Order matters:
each phase can only anchor onto what the previous ones put in the tree.

```bash
git checkout <phase-3-commit> -- people.jsonl claims.jsonl   # the hand-seeded base
python3 tools/phase2_quraysh.py   --write      # Quraysh, under Fihr
python3 tools/phase5_tribes.py    --write      # Qahtan seed + tribes
python3 tools/phase6_ansar.py     --write      # Ansar clans + companion entries
python3 tools/phase6b_notables.py --write      # marquee companions, by hand-quoted chain
python3 tools/phase7_wives.py     --write      # the Ummahat al-Mu'minin, hand-quoted
python3 tools/extract_kunya.py    --write      # kunyas from entry bodies
python3 tools/kunya_notables.py   --write      # kunyas for the directory people
python3 tools/prune.py            --write      # repair names, then drop what was never a name
python3 tools/merge.py            --write      # collapse duplicates - AFTER prune, see below
python3 tools/retranslit.py       --write      # re-read Latin forms in place
python3 validate.py && python3 test_wasl.py && python3 build.py
```

Phase 1 and Phase 3 are hand-seeded and live in the base commit; `tools/seed_phase1.py` and
`tools/phase3_ahlbayt.py` are kept as the record of how they were built. Roughly ten minutes end
to end.

**`prune` must run before `merge`.** Prune *repairs* names before dropping anything - `Umar
amuhu al-Shayba` becomes `Umar` - and a repair can turn two differently-named siblings into
duplicates. Run the other way round, merge never sees them: Ali ended up with two sons called
Umar exactly this way. `retranslit` is last because it only relabels.

## Marriage is not descent

`married_to` is the one claim type that is not a tree edge, and it must stay out of `kids`:
a wife hung under her husband would make the page assert that she descends from him. It is
indexed both ways in the template and rendered as its own row.

A wife reaches the tree only by her FATHER's chain, so every route in is a route in wrong.
Most of their fathers are outside what the parsers built - Banu Asad b. Khuzayma, `Amir b.
Sa`sa`a, Khuza`a and the Jewish Banu al-Nadir are not Quraysh and not Ansar - so where a
chain does not anchor, its own top name becomes a root. **Forcing an anchor is worse than
having none.** A one-name suffix match put Safiyya bt. Huyayy of Banu al-Nadir onto
`p.al-nadir`, which is al-Nadir b. al-Harith of `Abd al-Dar - a Qurashi - and would have
placed the Prophet's Jewish wife inside Quraysh. Below three names the corpus decides, not
the tree: `identifies()`, never `find_by_chain` alone. `test_wasl.py` names that trap.

`ingest.Store.person(force=True)` mints a root. Only a hand-quoted pass may use it, because
it asserts 'the top of this chain is nobody we already hold' - which a parser cannot know.

## Summaries: the one place prose is written, not quoted

`tools/summaries.py` holds a hand-written brief per Who's who person. It is the only composed
prose in the repository, so it is the only place a plausible sentence could pass unchecked,
and the format exists to stop that. A summary is not a paragraph: it is a list of lines, and
an `anchored` line carries the Arabic phrase from its own entry that it rests on. `validate.py`
re-reads that phrase and fails if it is not inside the pages the entry claims; `test_wasl.py`
re-reads it again from the raw page slice, sharing no code with the index, and asserts that a
genuine phrase from a DIFFERENT entry is rejected.

What that proves: every statement points at text that exists where it says. What it does not
prove: that the English is a fair rendering of the Arabic. That stays a human judgement -
Rule 6 exactly - and the docs should never imply otherwise.

Drafting rules, all of which exist because the alternative is worse:

- **Anchor first, sentence second.** These lives are well known; writing from memory produces
  fluent detail that anchors to nothing and reads perfectly.
- **`editorial` lines carry no number and no name**, are counted, and are capped at a fifth.
- **Where the entry disagrees with itself, the summary says so.** Ibn Sa'd gives Safiyya's
  death as 50 and, in the same chapter, as 52. Ibn al-Athir reports four different counts of
  who preceded 'Umar into Islam and two readings of his mother's name that turn on one letter.
  Ibn Sa'd gives three accounts of how Qusayy got the House. Smoothing any of that is editing
  the source, which Rule 3 forbids.
- **Nothing enters that is not in THIS entry**, however well attested elsewhere.

An anchor must be at least 25 normalised characters. `locate()` returns the FIRST occurrence
in a work, so a phrase common enough to appear earlier fails the span check - which is the
point: it was not distinctive enough to cite.

## Translations

`tools/i18n.py` holds the interface strings and gloss templates; `tools/translate_claims.py`
writes an `id` and `ms` gloss onto every claim. Regenerate after any data change:

```bash
python3 tools/translate_claims.py --write
```

The summaries are the exception to the rule below, and deliberately: there the English is the
original, not a gloss of Arabic, so `id` and `ms` are translated FROM it. Everywhere else:

Templated glosses are GENERATED per language from the structured fields, never translated from
the English - a translation of a translation drifts for no reason when the fact is 'X, son of
Y'. Only bespoke prose is hand-translated, and from the Arabic. A claim with no translation
makes the generator exit nonzero and refuse the write, so stale text cannot survive silently.

## Corpus gotchas
- OpenITI mARkdown: `#` starts a paragraph, `~~` continues it, `### |` is a heading,
  `PageV01P002` is a milestone placed at the **end** of the page it closes, `ms0031` is a
  Shamela milestone and is noise for our purposes.
- **A page marker can repeat.** Ibn Sa'd's text closes every numbered report with the page
  marker, so one page is several disjoint segments. Anything that slices by page must gather all
  of them, not the first.
- Text quality varies between versions of the same edition. `IbnAbdAlBarr` deliberately uses
  `Shamela0012288` rather than `JK000778`: same Bajawi edition, same pagination, far fewer
  OCR errors. Check quality before adding a version.
- Ibn Isḥāq's own recension in the corpus (Zakkār's *Siyar wa-Maghāzī*) does **not** contain
  the opening nasab. Ibn Isḥāq's chain survives through Ibn Hishām, who names his isnād for
  it explicitly. Cite it as Ibn Hishām transmitting from al-Bakkāʾī from Ibn Isḥāq.

---
> Source: [hendrasaputra/wasl](https://github.com/hendrasaputra/wasl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
