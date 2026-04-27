## nattydate

> This file is the working guide for an AI agent about to modify this codebase. Read it before touching anything.

# NattyDate — Codebase Guide for AI Agents

This file is the working guide for an AI agent about to modify this codebase. Read it before touching anything.

---

## What This Project Does

NattyDate is a deterministic, fuzzy-scoring natural language date/time preprocessor. Given messy human strings like `"nxt fri 14:00"` or `"last monday at noon"`, it returns normalized structures, formatted timestamps, or a JSON token AST. No ML, no network calls, no system clock in tests.

---

## Key Files

| File | Role |
|---|---|
| `src/lib.rs` | Everything: pipeline, scoring, resolution, formatting |
| `src/main.rs` | CLI wrapper and test runner — thin; no business logic |
| `tests.json` | 104-case test suite with fixed `mock_now: 2026-03-18` |
| `SKILL.md` | User-facing quick reference (how to use the tool) |
| `CONTRIBUTING.md` | How to add words, holidays, modifiers |

---

## Build and Test

```bash
cargo build                          # debug build
cargo build --release                # optimised binary
cargo test                           # runs all 104 cases via #[test] in lib.rs
cargo run -- test --verbose          # same suite via CLI runner
cargo run -- "tomorrow at 3pm" -f "YYYY-MM-DD HH:mm:ss"
cargo run -- "nxt fri 14:00" -o json
cargo run -- test --test-file custom.json
```

All tests are deterministic: `mock_now = 2026-03-18` (a Wednesday) is injected via `ParseConfig.mock_now`. Never rely on `Local::now()` in tests.

---

## Pipeline Architecture

```
Input string
    │
    ▼
normalize()          — lowercase, phrase substitution, whitespace collapse
    │
    ▼
tokenize()           — split into RawToken variants: Word / Number / DateNumeric / Time / TimeZone
  ├─ convert_compound_numbers()   "nine thirty" → "9 30"
  └─ resolve_time_phrases()       "half past 10" → "10:30"
    │
    ▼
tokenize_and_classify()          — for each raw token, call evaluate_token()
  ├─ evaluate_token()             returns Vec<ScoredToken>, sorted by score desc
  │   ├─ parse_time_scored()
  │   ├─ parse_date_numeric_scored()
  │   ├─ parse_timezone_scored()
  │   ├─ i32 parse
  │   ├─ HashMap exact dict lookup   ← O(1), short-circuits fuzzy
  │   └─ Levenshtein fuzzy scan      ← only runs on dict miss
  └─ recombine adjacent tokens if combined score > individual avg + RECOMBINE_GAIN
    │
    ▼
apply_context_boosts()           — 4-phase, accumulate then apply
  Phase 1: collect score boosts (temporal adjacency, modifier→weekday, num+unit)
  Phase 2: apply boosts, sort Unknown candidate lists once each
  Phase 3: At + Number → Time structural conversion
  Phase 4: promote Unknown ≥ THRESHOLD_PROMOTE to Known
    │
    ▼
resolve()            — consumes Modifier/Weekday/Holiday/RelativeDay into DateNumeric
  preprocess_ago_patterns()      — converts "N unit ago" triples to DateNumeric first
    │
    ▼
to_canonical() / format_custom() — produce output string
```

---

## Token Types

```rust
// After resolve(), you will mostly see:
Token::Known(ScoredToken { token: KnownToken::DateNumeric { y, m, d }, score })
Token::Known(ScoredToken { token: KnownToken::Time { hour, min, sec, formatted }, score })
Token::Known(ScoredToken { token: KnownToken::TimeZone(TimeZoneKind::Named("EDT")), score })
Token::Noise(String)           // filtered out before output
Token::Unknown { word, candidates }  // score < threshold, passed through
```

`Modifier`, `Weekday`, `RelativeDay`, and `Holiday` tokens are consumed by `resolve()` and do not appear in final output — they are converted to `DateNumeric`. The JSON `-o json` output reflects the resolved state.

---

## Scoring Constants (top of lib.rs)

```rust
SCORE_EXACT         = 1.0   // dictionary exact match
SCORE_DICT_TIME     = 0.9   // exact match for named time words (morning, noon…)
SCORE_EXPLICIT      = 0.95  // unambiguous: ISO, HH:MM, am/pm
SCORE_NUMERIC       = 0.9   // plain integer
SCORE_DATE_YMD      = 0.95  // three-part date with year, unambiguous
SCORE_DATE_YMD_AMBIG= 0.9   // three-part date with year, ambiguous m/d order
SCORE_DATE_MD       = 0.75  // two-part date (no year)
SCORE_DATE_AMBIG    = 0.7   // two-part date, both parts ≤ 12
SCORE_TIME_COLON    = 0.95  // HH:MM with colon
SCORE_TIME_DEFAULT  = 0.8   // other separators, no am/pm
SCORE_TIME_SINGLE   = 0.7   // bare hour with no suffix

FUZZY_BONUS_PREFIX  = 0.1   // candidate word starts with the query
FUZZY_BONUS_FIRST_CHAR = 0.05
FUZZY_MAX_SCORE     = 0.95  // cap so fuzzy never equals exact
FUZZY_MIN_ACCEPT    = 0.65  // minimum to surface a fuzzy candidate
FUZZY_MAX_LEN_DIFF  = 2     // skip pairs whose lengths differ by more

BOOST_TEMPORAL_ADJ  = 0.1   // Time adjacent to Date/TZ
BOOST_MODIFIER_CTX  = 0.2   // Weekday candidate following a Modifier
BOOST_RELATIVE_TIME = 0.15  // RelativeDay candidate adjacent to Time
BOOST_NUM_UNIT_PAIR = 0.2   // Number directly before a Unit

THRESHOLD_PROMOTE   = 0.65  // Unknown promoted to Known
THRESHOLD_KNOWN     = 0.65  // Single-candidate accepted as Known
THRESHOLD_MARGIN    = 0.09  // Score gap required for top candidate to win
RECOMBINE_MIN       = 0.8   // Min score for recombined token to win
RECOMBINE_GAIN      = 0.15  // Combined must beat per-token avg by this
```

### Critical invariant: THRESHOLD_MARGIN

`THRESHOLD_MARGIN = 0.09`, **not** `0.10`. The gap between `SCORE_DATE_YMD_AMBIG (0.9)` and `SCORE_TIME_DEFAULT (0.8)` in `f32` arithmetic is `≈ 0.09999996`, which is less than `0.10`. Raising this constant above `0.099` will cause date strings like `"18.3.26"` to be misclassified as times. Do not change it.

---

## Dictionary (get_dict / get_dict_map)

`get_dict()` returns a `&'static [(&'static str, KnownToken)]` — the full vocabulary.
`get_dict_map()` returns a `&'static HashMap<&'static str, KnownToken>` built from it — used for O(1) exact lookup in `evaluate_token()`.

**Both are initialised once via `OnceLock` and are immutable at runtime.**

To add a word: append to the `vec![]` inside `get_dict()`. The map is derived automatically. No other changes required for a simple alias.

### Modifier synonyms (all currently mapped):

| Word | Maps to |
|---|---|
| `next`, `nxt`, `following` | `Modifier::Next` |
| `last`, `lst`, `past`, `previous`, `prev`, `prior` | `Modifier::Last` |
| `this` | `Modifier::This` |
| `ago` | `Modifier::Ago` (consumed by `preprocess_ago_patterns`) |

---

## Holiday Resolution

`resolve_holiday(h, year)` maps each `Holiday` variant to a `NaiveDate`. Static holidays are `NaiveDate::from_ymd_opt`. Floating holidays use:
- `nth_weekday_of_month(year, month, weekday, n)` — nth occurrence
- `last_weekday_of_month(year, month, weekday)` — last occurrence

After computing `h_date`, `resolve()` checks `if h_date < now` and if so, calls `resolve_holiday(h, current_year + 1)`. This auto-advance logic is the only place the current date affects holiday output.

---

## Past Date Resolution

### Prefix modifiers (last monday, previous friday)
`current_modifier` is set when a `Modifier` token is encountered. The next `Weekday` token picks it up via `effective_mod = current_modifier.as_ref().or(postfix_mod.as_ref())`.

### Postfix modifiers (monday last, friday next)
In the `Weekday` arm of `resolve()`, if `current_modifier` is `None`, the code peeks at `tokens[i+1]` for a `Modifier` token. If found, it's used as `postfix_mod` and the index is advanced to skip it.

### Ago patterns (3 days ago, 1 week ago)
`preprocess_ago_patterns()` runs **before** the main resolve loop. It scans for `Number + Unit + Ago` triples and replaces them with a single `DateNumeric` computed as `now - Duration::days(n)`. Month/Year use approximate values (30/365 days).

---

## Normalization Phrases

`normalize()` performs phrase substitution before tokenization. Multi-word holiday names are collapsed to single tokens (e.g. `"new years day"` → `"newyearsday"`). This must happen before the tokenizer splits on whitespace. Phrases are applied in order — longer phrases should come first to avoid partial matches. If you add a new multi-word holiday, add its phrase here.

---

## format_custom Placeholders

`format_custom(tokens, template)` scans for `DateNumeric` and `Time` tokens and replaces:

| Placeholder | Source | Default if absent |
|---|---|---|
| `YYYY` | DateNumeric.y | unchanged (literal `YYYY`) |
| `YY` | DateNumeric.y % 100 | unchanged |
| `MM` | DateNumeric.m | unchanged |
| `DD` | DateNumeric.d | unchanged |
| `HH` | Time.hour | `"00"` |
| `mm` | Time.min | `"00"` |
| `ss` | Time.sec | `"00"` |
| `TZ` / `Z` | TimeZone | removed (with leading space) |
| `{RELATIVE}` | RelativeDay or Weekday | removed |

Note: `HH`, `mm`, `ss` default to `"00"` when no Time token is found. `YYYY`, `MM`, `DD` are **not** defaulted — they remain as literal strings in the output, which is how negative test cases (`"xyzzy blob flargh"` → `"YYYY-MM-DD"`) are verified.

---

## Common Gotchas

1. **Modifier consumption**: `current_modifier` is set to `None` after a Weekday/RelativeDay/Holiday consumes it. If no date token follows, the modifier is flushed back into `resolved` as an orphan token (it won't affect output via `format_custom` since Modifier has no placeholder, but it will appear in `-o json`).

2. **"coming" is noise**: The word `"coming"` is in the hard-coded noise list in `tokenize_and_classify()`. If a user says `"coming monday"`, `"coming"` is silently dropped and the weekday resolves normally (as next Monday). This is intentional.

3. **Month tokens**: `KnownToken::Month(Month::Jan)` etc. are classified correctly but are not resolved by `resolve()`. Month-name expressions like `"jan 15"` do not currently produce a date. Do not write tests that assume they do.

4. **Two-part dates roll forward**: A yearless `DateNumeric { y: None, m, d }` — like `"9-1"` (September 1) — is given the current year. If that date is already in the past relative to `now`, it rolls to next year. This happens in the `DateNumeric` arm of `resolve()`.

5. **ISO 8601 parsing**: The ISO path goes through `parse_iso()` → splits on `T` → `parse_date_numeric_scored` + `parse_time_scored` + `parse_timezone_scored`. The formatted time field is overwritten with the raw ISO time substring to preserve sub-second precision.

6. **Recombination**: Adjacent tokens `s1` and `s2` are evaluated as `"s1s2"` and `"s1 s2"`. If the combined score beats the per-token average by `RECOMBINE_GAIN (0.15)` and reaches `RECOMBINE_MIN (0.8)`, the two tokens are merged into one. This handles cases like `"9 am"` → `"9am"`.

---

## Adding a New Feature — Checklist

- [ ] Add to `get_dict()` if it's a new word/alias
- [ ] Add to `normalize()` phrases if it's multi-word
- [ ] Handle in `resolve()` or a new pre-pass if it has semantic meaning
- [ ] Add at least one test case in `tests.json`
- [ ] Update `SKILL.md` user documentation
- [ ] Update `CHANGELOG.md` under `[Unreleased]`
- [ ] Run `cargo test` and `cargo run -- test --verbose`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/troy74) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
