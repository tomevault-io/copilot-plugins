## ankiprepper

> Java CLI tool that preprocesses Noji CSV exports for import into Anki. Adds unique sequential ID prefixes to prevent Anki import collisions.

# AnkiPrepper

Java CLI tool that preprocesses Noji CSV exports for import into Anki. Adds unique sequential ID prefixes to prevent Anki import collisions.

## Card Formatting Philosophy

### Front side disambiguation rule

Year-front cards always need a category label — a bare year gives no signal about what to recall. Name-front and rank-front cards only need a label when genuinely ambiguous across decks.

- **Year as front** → always prefix: `Heisman: 2024`, `Emmy Best Drama: 2019`, `Oscar Best Actor: 2021`
- **Name as front** → add `[Tag]` bracket only if the name appears in multiple deck contexts: `Russia [Area]` vs `Russia [Population]`, `California [Nickname]`
- **Rank as front** (`#1`, `#2`, …) → no label needed; back content makes the category obvious
- **Nickname/fact as front** → no label needed; the content is self-identifying

### Back side rule

Back sides are always clean — no category prefix. The front carries the label; the back is the answer only.

### Ranked list format

For ranked lists (cities, countries by population/area): `Entry → #RANK · VALUE` and `#RANK → Entry · VALUE`. Drops value-as-front direction to avoid collisions when multiple entries share the same value.

### Examples by deck

| Deck | Front pattern | Back pattern |
|------|--------------|--------------|
| Oscar Best Actor | `Oscar Best Actor: 2021` | `Anthony Hopkins · The Father` |
| Emmy Best Drama | `Emmy Best Drama: 2019` | `Game of Thrones` |
| Heisman Winners | `Heisman: 2024` | `Travis Hunter (CB/WR)\nColorado (2/2)` |
| Billboard Year End | `Billboard Year End: 2020` | `Blinding Lights · The Weeknd` |
| VP of | `VP of: George Washington\n1789-1797` | `John Adams` |
| Most Populous Countries | `India [Population]` | `#1 · 1.45 billion` |
| Largest Countries by Area | `Russia [Area]` | `#1 · 6.6m sq mi` |
| US State Nicknames | `California [Nickname]` | `The Golden State` |
| People Magazine Sexiest Man | `Sexiest Man Alive: 2020` | `Michael B. Jordan` |
| EU/NATO Nations | `EU: 1993` / `EU Member: Germany` | clean year or country |
| SNL Five Timers | `SNL Hosting: 5 Times\n2008–2021` | clean name |

---
> Source: [joevandevusse/AnkiPrepper](https://github.com/joevandevusse/AnkiPrepper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
