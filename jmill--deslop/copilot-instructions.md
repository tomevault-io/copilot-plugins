## deslop

> deslop is a [Vale](https://vale.sh) style package that flags AI-slop in prose: the stock

# deslop — Agent Guide

deslop is a [Vale](https://vale.sh) style package that flags AI-slop in prose: the stock
phrases, hedges, and structural tics that mark machine-written text. It ships as a release
asset (`Deslop.zip`) that other repos consume through Vale's `Packages` mechanism.

## Layout

| Path | What it is |
| --- | --- |
| `styles/Deslop/*.yml` | The rules. One file per tell family (e.g. `SlopVocab.yml`, `HollowCloser.yml`). Each is a Vale rule extending `existence`, `substitution`, or `occurrence`. |
| `styles/Deslop/meta.json` | Package metadata Vale reads on `vale sync`. |
| `.vale.ini` | Lints this repo's own prose. `Deslop` only, no `Vale` base style. Excludes `tests/`, which holds deliberate slop. Consumers use a different `.vale.ini` (see below). |
| `tests/run.sh` | The rule suite. Run it after every rule change. |
| `tests/should-flag.md` | Deliberate slop. Every rule must fire here at least once. |
| `tests/should-pass.md` | Ordinary technical prose. Any alert here is a false positive and fails the build. |
| `tests/expected.tsv` | Pinned `<rule>`/`<phrase>` pairs, mostly inflections and narrowed tokens. |
| `tests/expected-branches.tsv` | Pinned branch lists for `occurrence` rules, which nothing else protects. |
| `tests/check.py` | The assertions behind `run.sh`. |
| `docs/ADOPTING.md` | Copy-paste integration guide: `.vale.ini`, CI, customizing, composing private styles. |
| `.github/workflows/test.yml` | Runs the rule suite on every push and PR. |
| `.github/workflows/vale.yml` | Lints this repo's `*.md` on PRs. |
| `.github/workflows/release.yml` | On a `v*` tag, runs the suite, builds `Deslop.zip`, publishes a GitHub release. |
| `CHANGELOG.md` | What changed between released versions. |
| `README.md` | Human-facing overview and the full rule table. |

## How to use it (consuming the package)

A downstream repo adds a `.vale.ini`:

```ini
StylesPath = styles
MinAlertLevel = suggestion
Packages = https://github.com/JMill/deslop/releases/latest/download/Deslop.zip

[*.{md,mdx}]
BasedOnStyles = Deslop
```

Then `mkdir -p styles && vale sync && vale "**/*.md"`. The `mkdir -p styles` must come before
`vale sync` or Vale stages to a temp path and leaves `StylesPath` empty. Full guide:
[docs/ADOPTING.md](docs/ADOPTING.md).

## Writing a rule

The common change is adding a tell. Create one file under `styles/Deslop/`. Most rules extend
`existence` (flag any match) and take a list of regex `tokens`:

```yaml
# styles/Deslop/SlopVocab.yml (excerpt)
extends: existence
message: "'%s' is an AI-slop tell. Cut it or say the thing plainly."
level: warning
ignorecase: true
tokens:
  - 'delv(?:e|es|ed|ing)'
  - 'plethora'
  - 'when\s+it\s+comes\s+to'
```

Use `occurrence` for density caps (cap how often a `token` appears in a `scope`, as
`HollowIntensifier.yml` does) and `substitution` for "prefer X over Y" swaps. A new file is
picked up automatically once it is placed under `styles/Deslop/`, because `.vale.ini` lists
the `Deslop` style directory in `BasedOnStyles` (you list style directories there, not
individual rule stems). The rule name is the file stem (`Deslop.SlopVocab`). See the
[Vale styles docs](https://vale.sh/docs/topics/styles/).

**Always run `tests/run.sh` after touching a rule.** Every gotcha below produces a rule that
loads without error and silently matches nothing or too much. Vale reports no diagnostic for
any of them; the suite is the only thing that will tell you.

### Gotchas that cost real debugging time

- **`tokens` entries are wrapped in `\b...\b`.** So `delve` does not match `delved`, and
  `seamless` does not match `seamlessly`. Spell inflections out: `delv(?:e|es|ed|ing)`.
  This is the single most common way to ship a rule that looks right and under-fires.
  It applies to verb-headed idioms too, where the base form is the *least* common one in
  practice: `move the goalposts` misses `moved the goalposts`, and `circle back` misses
  `circled back`. Write `mov(?:e|es|ed|ing)\s+the\s+goalposts`.
- **Cover the whole paradigm, including irregulars and the gerund.** `navigate(?:s|d)?`
  silently drops `navigating`, because the gerund changes the spelling of the stem.
  `(?:driv(?:e|es|ing)|drove)` drops `driven`. And an alternation like `remain(?:s|ed)`
  drops the *base* form, since the suffix group is mandatory without a trailing `?`.
- **A `tokens` entry ending in punctuation never matches.** The trailing `\b` cannot be
  satisfied after `!` or `,`. Use `raw` for those.
- **Multiple `raw` entries are concatenated, not alternated.** A list of five `raw`
  patterns compiles to one impossible regex and the rule flags nothing, forever. Write a
  single `raw` entry with top-level alternation, as `AssistantOpener.yml` does.
- **Do not mix `tokens` and `raw` in one rule.** The rule stops matching entirely.
- **`raw` is not wrapped in `\b`.** Add the boundaries by hand or the pattern matches
  inside words.
- **Lookarounds are accepted and then ignored.** `(?<!test\s)harness\s+the` compiles
  without complaint and still matches "test harness the". You cannot carve out exceptions
  this way; narrow the pattern with real context instead, or let consumers use a Vale
  vocabulary (`accept.txt` entries are filtered out of every rule's matches).
- **Typographic apostrophes are not normalised.** `don't` written with a plain `'` does
  not match `don’t`, and editors rewrite quotes routinely. Spell the class out:
  `don\s*['’]?t`. This applies to `tokens` and `raw` alike.
- **Substitution swaps support capture backreferences.** `'utiliz(e|es|ed|ing)': 'us$1'`
  gives the right suggestion for all four forms on one line. Irregular forms
  (`comprehended` -> `understood`) still need their own entry.

### One tell, one home

A word or phrase must appear in exactly one rule file. `robust` lived in both `SlopVocab.yml`
and `Substitutions.yml` and so reported twice on every hit — the same double-flagging the repo
`.vale.ini` avoids by not enabling Vale's base style. The split is:

- **`Substitutions.yml`** owns anything with a clean one-word replacement.
- **`SlopVocab.yml`** owns phrases and words with no single swap.
- Family phrases live with their family: the whole `in today's ...` set is `OpenerCliche.yml`,
  the whole `unlock <noun>` set is `SlopVocab.yml`.

The suite fails on any two rules that flag overlapping text, so a collision shows up as a
`double-flag` error naming both rules.

### Precision

Anchor a tell that is only a tell in one position. `great question` is throat-clearing
when it opens a reply and ordinary prose in `the design raises an interesting question`,
so the pattern requires sentence-initial position. Prefer a narrowed token over a bare
word whenever the bare word has an ordinary technical meaning. `realm of` not `realm` (Realm is a database), `landscape of` not `landscape`
(landscape orientation), `harness the power` not `harness the` (test harness),
`unlock <noun>` not `unlock` (unlock a mutex). Add the ordinary usage to
`tests/should-pass.md` so it stays safe.

Keep this repo public-safe. Private or brand-specific phrasing does not belong here. Consumers
add those through composition: a second style directory listed alongside `Deslop` in
`BasedOnStyles`. The mechanism is documented in [docs/ADOPTING.md](docs/ADOPTING.md) section 8.

## Commands

```sh
# The rule suite. Run this after every rule change.
tests/run.sh

# Lint this repo's prose (uses the repo .vale.ini; tests/ is excluded)
vale --config .vale.ini .

# Build the release asset locally
cd styles && zip -rq /tmp/Deslop.zip Deslop
```

## Cutting a release

1. Land the rule change on the default branch with `tests/run.sh` green.
2. Add a `CHANGELOG.md` entry describing what now fires (or stops firing).
3. Push a `v*` tag. The release workflow reruns the suite, builds `Deslop.zip` from
   `styles/Deslop`, and attaches it to a GitHub release.

Consumers pinned to `releases/latest/download/Deslop.zip` pick it up on their next
`vale sync`, so **an unreleased rule change reaches nobody**. This has bitten the repo
before: `v0.1.0` shipped 13 of 17 rules for two months because the ruleset was expanded
and no tag followed. The suite's packaging check now compares the archive against the
tree, but it cannot make you tag.

## Conventions

- One tell family per file. Name the file for what it catches.
- One tell, one file. Never the same word in two rules.
- `warning` for tells, `suggestion` for density signals. Nothing ships as `error`:
  which tells should block a build is the consumer's call, and they can promote any rule.
- Messages name the fix, not just the problem. Point at a plainer rewrite.
- Every example slop word in docs is backtick-quoted so deslop does not flag its own guide.
- Every new tell gets a line in `tests/should-flag.md`; every narrowed token gets its
  ordinary usage in `tests/should-pass.md`.
- A fixture line must be matched by exactly one token of its rule, which the suite now
  enforces as an `ambiguous fixture` failure. If a sibling token
  also matches it, deleting a branch leaves the line still alerting and the loss goes
  unnoticed — `The evolving landscape of tooling` fired through `landscape of` too, so
  it became `The evolving landscape changed`.
- Write inflections as spelled-out alternatives, not character classes. `indicat(?:e[sd]?|ing)`
  hides `indicate`/`indicates`/`indicated` inside one class; the branch audit expands classes
  now, but the flat form is what a reader can check.
- One tell per fixture line. Every prose line in `should-flag.md` must produce an alert
  on its own, and every *token* in every rule must match some line, so a dead token
  cannot hide behind its neighbours. Headings and HTML comments are the only exemptions.
- Give each inflected form its own fixture line. For verb-headed idioms the base form is
  the least common one in running prose, so `moved the goalposts` earns a line more than
  `move the goalposts` does.
- Editing an `occurrence` rule means editing `tests/expected-branches.tsv` too. Those
  rules fire on density across the whole file, so one fixture line covers every branch at
  once and removing a single one changes nothing the suite can see. The pinned list is
  the only thing that notices, and the suite checks it in both directions: a pinned
  branch must still exist in the rule, and every branch in the rule must be pinned.

## Where this sits

deslop is the public, generic anti-slop floor: mechanical line-level tells any prose can hit.
It is one input to the private [writers-room](https://github.com/JMill/writers-room) pipeline,
which runs deslop as a deterministic pre-pass and layers judgment-level critics on top.
sam-bot is the front door that routes work to capabilities like that pipeline. deslop itself
knows nothing about either; it is a standalone Vale package.

---
> Source: [JMill/deslop](https://github.com/JMill/deslop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
