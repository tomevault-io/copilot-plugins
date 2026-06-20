## mlb-announcer-1930s

> Use when the user asks for a 1930s-style radio broadcast transcript, period-accurate old-time announcer call, vintage play-by-play, "Red Barber / Graham McNamee version", or any "make it sound like a 1930s broadcast" transformation of a game. Consumes the Markdown product of the `mlb-game-report` skill (with its structured ATMOSPHERE, SCORING, PLAY-BY-PLAY + Statcast sections) and produces a transcript written in the voice of a Golden Age radio sportscaster, with modern Statcast metrics translated into period-appropriate color commentary.


# 1930s Radio Broadcast Transcript

Transforms a structured MLB game report into a transcript of a *live* radio broadcast, as if called by a 1930s-era sportscaster **who has time-traveled to the present and is gamely calling a contemporary game**. Takes the structured product of the `mlb-game-report` skill (PBP + Statcast + atmosphere + captivating-moment data) and re-voices it in period-accurate radio English.

## The conceit

The announcer's voice is always Depression-era. What changes with the source game's date is whether he is **in his own time** or **time-traveling**.

**Always read the `date` field from the source frontmatter before writing and pick the right mode.**

### Mode A — Source date 1925–1949 (his own era)

He is simply calling a game he was scheduled to call. **Drop the time-travel framing entirely.** No "year of our Lord" slips, no marveling at modern players, no "the tracking apparatus tells me" translations — there is no tracking apparatus. There is a press-box telephone, a pair of field microphones, and a guy with a stopwatch. Use the full period vocabulary from `references/vintage-phrases.md` naturally. Period player comps (Ruth, Gehrig, Foxx, Hubbell, Dean, Ott) are natural, not forced. The masthead date matches the source date.

### Mode B — Source date 1950 or later (time-traveler)

He is in our century but keeps his 1930s voice and vocabulary. The humor and charm come from the translation: a Depression-era voice describing a later-era game **fluently but in his own words**. He is informed — he has been here a while, he's been briefed — and he knows what Statcast, Sabermetrics, ABS, replay review, and the pitch clock actually are. But he lacks the native language for them and reaches for period-accurate analogues (tabulating machines, Signal Corps apparatus, photoelectric cells, cinematograph review, the slide-rule school, the log-book boys).

- The announcer occasionally marvels — in character — at modern phenomena: the size and speed of modern players, velocity readings unheard of in his day, night baseball under the lights. A light touch, not a bit every inning. The further from his own era (2020s vs. 1960s), the more he has to reach.
- Modern player names used as-is — Moncada, Trout, Tatis Jr. He can say them confidently.
- **Modern technology and concepts are translated, not avoided.** See `references/translations.md` for the full vocabulary table. Examples:
  - Statcast → "the Signal Corps tracking apparatus" / "the Bell Labs boys with their radio-ranging gear"
  - ABS → "the electric umpire" / "the photoelectric strike zone"
  - Sabermetrics → "the slide-rule school" / "the number men" / "the IBM-tabulator fellows"
  - Replay review → "the cinematograph review" / "the newsreel confab" (never "video review")
  - Exit velocity → "the off-the-bat reading" / "the tracking tells us"
  - Pitch clock → "the electric pitch timer" / "the chronometer"

### Mode C — Source date before 1921 (no radio baseball yet)

Baseball radio broadcasting did not meaningfully exist before August 1921 (KDKA's Pirates-Phillies broadcast). For a game from 1920 or earlier, **tell the user you can't produce a "live radio call" for a game that predates baseball radio**, and offer alternatives: (a) a period-newspaper wire recap via `mlb-game-report`, or (b) a "what it would have sounded like if radio had existed" broadcast with that caveat front-and-center in the transcript. Don't silently produce a live-radio call for a pre-1921 game.

### Always (all modes)

- **LIVE PRESENT TENSE.** This is a radio broadcast happening *right now*, as the game unfolds. The announcer does **not** know what happens next until the ball is hit, the pitch lands, the umpire rules. **Default voice is present tense**: "He swings — *he misses* — strike two!" / "Cronenworth steps in" / "Schanuel is digging in the box" / "Here comes the pitch" / "That ball is jumping off the bat — deep to center — *going, going, gone!*" — not "Moncada homered" or "Schanuel went 3-for-5". Past tense creeps in easily; be vigilant. Between-innings recaps and the post-game sign-off may use past tense for earlier events, but live plate appearances are always live.
- **The source date is the masthead date.** Never substitute a different year.
- **Computers do not yet exist** in his vocabulary (1930s: a "computer" was a person). Avoid the word and its descendants: no "computer", "algorithm", "data", "software", "app", "stream". Prefer "tabulating machine", "records", "the books", "the wire service".
- **Facts from the source, voice from the references.** Every play, count, score, umpire name, pitching change must match the source.
- **Sparse sources** (historical games where the source report lacks detailed PBP) → produce a **highlights-style broadcast** that walks through the line-score innings and scoring plays, rather than pretending to call every batter. Make this mode explicit to the listener: the announcer is summarizing rather than live-calling. Use "our correspondent at the park has wired us…" / "we gather from the press-box wire…" as cover — and even in summary, keep the core calls in present tense when describing a specific play ("Thomson *hits* one deep" rather than "Thomson hit one deep").

### Tense rules in detail

- **During an at-bat (live):** present tense. "Soriano comes set." / "Here's the pitch — *ball one*, outside." / "Moncada digs in." / "*He gets all of it!*"
- **Describing ball flight:** present progressive or present. "That ball *is* going, *is* going — gone!" / "High fly to right — Tatis races back — he's at the warning track — at the wall — *off the top!*"
- **Immediately after the play resolves:** present perfect or simple past for the just-finished action, then back to present. "That's a base hit through the hole. / Trout *scores* from second. / Neto *stands* at first." Keep moving forward in time.
- **Between-innings recaps:** past tense is allowed for recapping earlier innings. "Soriano *set 'em down* in order in the first. / The Angels *plated* three in the second on that Moncada poke and the Frazier double."
- **Closing sign-off:** past tense is fine for the game-wide summary.
- **Atmospheric color:** present tense. "The sun *is just slipping* behind the grandstand" — not "the sun slipped". "The flags *hang* limp" — not "hung limp".

## Preferred workflow (as of v1.1): scaffold-first

The skill is now part of a **four-stage pipeline**. Three stages are mechanical scripts; this skill owns the one non-mechanical stage (voice).

1. **Scaffold** — `scripts/scaffold-broadcast.py <dataset>` produces a factual `.md` trellis with every inning divider, pitching change, and per-play HALLIDAY stub already placed (real timestamps from `plays.csv.start_time_utc`, real counts, real events). Every line needing voice is marked `<ENRICH: hint>` or `<ENRICH>`.
2. **Enrich** — *this skill*. Open the scaffold, rewrite each `<ENRICH>` marker into 1930s radio voice per the style guide and references. Keep the factual spine (batter/pitcher/count/event) intact; replace the stub sentence with live present-tense color. Remove `<ENRICH: …>` placeholders as you go.
3. **Tag** — `scripts/tag-broadcast.py <dataset> <md>` injects `<span class="player">`, `<span class="team-name">`, `<span class="run">` and `<span class="sb-tag">` pills linking each play back to the game-log anchor.
4. **Render** — `scripts/render-broadcast <dataset>` runs tagging → pandoc → CSS/nav inject in one shot, producing the final `.html`.

**Why scaffold-first:** facts-gospel becomes automatic (timestamps, batter order, counts, pitching changes come from CSV), so the announcer can't accidentally flip a score or miscount strikes. The voice-writing shrinks to "make each stub sound like 1936 radio" on a trellis that's already structurally correct.

**If no scaffold exists**, fall back to writing the whole .md from scratch following the rest of this document. The scaffold is preferred but not required.

## Voice-depth rubric (hard requirements)

A broadcast is not done when every play has a line. It's done when every line *sounds like 1930s radio*. Before handing off, verify:

**Length.** A 9-inning game produces **2,500–4,000 words** of enriched narrative. Extra innings add ~300 words per frame. Anything under 2,000 is a scaffold, not a broadcast.

**Per half-inning.** Every half-inning has at least:
- **One sensory line** — crowd reaction, weather beat, umpire call, vendor cry, flag movement, sunlight shift, or a groundskeeper note. Not every paragraph — at least one somewhere in the half.
- **Descriptive action on each PA**, not just the event. "Moore lines a single into left, Powell plays it on a hop" beats "Moore singles." A fly ball names the fielder's route; a strikeout names the pitch type; a walk names the count shape.

**Big moments.** Any play flagged as `is_scoring_play` or with high `captivating_index`, or any HR, gets a **three-line dramatic hold**:
1. Setup line (batter, situation, pitcher check)
2. The swing-and-ball-flight line (italics on the verb: *"HIGH DRIVE, DEEP TO RIGHT!"*)
3. The reaction line (crowd SFX or HALLIDAY follow-up with the score recited)

For the play marked `captivating_play_idx` in `game.csv`, stretch to four lines and add a SFX break.

**Opening and closing.** Each gets **5–10 script lines**:
- Opening: MUSIC → STATION ID → greeting with venue + weather + crowd size → storyline/context → pitchers-on-the-hill → lineups → SFX → national anthem MUSIC → PLAY BALL → "we are underway" handoff.
- Closing: final-out call → SFX → final-score recap → decisions (W/L/SV) → batting stars → series/season context → sign-off → MUSIC → STATION ID.

**Verb variety across a series.** Don't reuse the same home-run call or the same ball-flight metaphor twice in a single broadcast. Rotate: "a drive!" / "there she goes!" / "off the bat like a cannon!" / "over the wall!" / "into the bleachers!" / "goodbye, baseball!" (anachronistic if pre-1947; use "that ball is gone!" / "a tremendous wallop!").

**Red flags that you're under-writing.**
- Same verb twice in one half ("a fly to X", "a fly to Y", "a fly to Z"). Vary: fly, lift, drift, loft, chase, drop, pop.
- "out" or "grounds out" as the entire action. What kind of grounder? Who fielded? What happened?
- No crowd/weather reference for an entire middle inning.
- The MOMENT OF THE GAME rendered in fewer than 3 script lines.
- A closing under 200 words.

If any of the above apply, you haven't finished — go back and enrich.

## When to use

Triggers:
- "give me a 1930s radio call of [game]"
- "Red Barber version" / "Graham McNamee version" / "Bill Stern style"
- "vintage announcer transcript" / "old-time radio broadcast"
- "make [report] sound like a 1930s broadcast"
- "narrate the [team] game like it's 1935"

## How to use

1. **Find the source report.** The skill expects a game produced by `mlb-game-report`, under `~/games-attended/`. Each game has two co-located artifacts:
   - The **dataset directory**: `~/games-attended/<slug>/` containing `game.csv`, `plays.csv`, `pitches.csv`, `batting.csv`, `pitching.csv`, `linescore.csv` — **this is your primary data source.**
   - The **rendered Markdown**: `~/games-attended/<slug>.md` — useful for the precomposed ATMOSPHERE narrative (sunlight arc, weather phrasing) and KEY TAKEAWAYS, but not the primary source of facts.

   If the user hasn't specified a game, list candidate slugs with `ls -d ~/games-attended/*/` and ask which one. If they reference a team + date, find the matching slug.

   **Backward compat:** if only a `.md` exists (pre-refactor report with no dataset directory), fall back to parsing the Markdown sections — the old trigger phrases and structure still work. Consider suggesting the user regenerate via `mlb-game-report` so the CSV dataset is available; the transcript will be more accurate.

2. **Read the CSV dataset first (primary data).** The CSVs are structured rows — far cleaner than regex-parsing the `.md`.

   | File | What's in it | How the announcer uses it |
   |---|---|---|
   | `game.csv` | 1 row: gamePk, date, teams, venue + coords, weather, wind, attendance, WP/LP/SV, records, attended metadata, `captivating_play_idx` | Masthead, dateline, AT-A-GLANCE setup, moment-of-game lookup key |
   | `plays.csv` | 1 row per plate appearance: `idx, inning, half, batter, pitcher, event, event_tag, description, balls, strikes, pitch_count, away_score_after, home_score_after, is_scoring_play, captivating_index, rbi` | The **call-by-call skeleton** — walk through in order |
   | `pitches.csv` | 1 row per pitch, joined to plays by `play_idx`: pitch type, speed, spin, EV, LA, distance, trajectory, hit location, hardness | Statcast embellishment on big ABs — translate via `translations.md` |
   | `batting.csv` / `pitching.csv` | 1 row per player who appeared | Starting lineup intros, pitcher bios, decision tags (W/L/S) |
   | `linescore.csv` | 1 row per inning + R/H/E summary | Inning-by-inning context, between-innings recaps |

3. **Consult the rendered `.md` only for narrative text you want to borrow.** The `.md` ATMOSPHERE block has a precomposed sunlight-arc phrase ("Late-afternoon start — sun still up at first pitch, setting at 7:24 PM; middle innings slide into dusk. Full dark around 7:50 PM.") and a one-sentence HOW IT HAPPENED. Both are handy to weave into the call. Don't re-parse the .md for play-level facts — use the CSVs.

4. **Optional — extra color via the MLB MCP.** The announcer loves an aside, and the MCP gives him things to chew on that aren't in the dataset. Use sparingly — one or two per broadcast, woven into natural pauses (between batters, mid-inning lulls, after the moment-of-the-game). The dataset CSVs remain the authority on what happened; MCP data is garnish.

   | Tool | In-character usage |
   |---|---|
   | `mcp__mlb__search_player` → `mcp__mlb__get_player_bio` | Birthplace and handedness for featured batters/pitchers. *"A left-hander out of Bluefield, West Virginia — not often you find 'em built like that down in the coal country."* |
   | `mcp__mlb__get_player_stats` (season or career) | Contextualize a performance. *"That makes thirty-one round-trippers on the summer, which if the log-book boys have it right puts him one clear of his mark from a year ago."* |
   | `mcp__mlb__get_stat_leaders` | League leaderboard placement. *"Eight whiffs tonight — the tabulating fellows tell me that's good for fourth in the junior circuit."* |
   | `mcp__mlb__get_standings` | Post-game pennant-race line. *"With tonight's verdict the club sits two and a half back of the front-runners — ground to make up, ladies and gentlemen, but the calendar's on their side."* |
   | `mcp__mlb__get_transactions` (around game date) | Roster-move explanations. *"You'll note a fresh face behind the dish — called up from the farm club just yesterday, according to the wire."* For IL activity: *"back from the sick bay"* / *"off the hospital list"*. |
   | `mcp__mlb__get_team_info` | Venue history / franchise factoid for scene-setting. |

   **Period vocabulary for MCP-sourced facts:** IL/injured list → *"the sick bay"*, *"the hospital list"*; transactions wire → *"the roster wire"*, *"the teleprinter"*; leaderboard → *"the batting ledger"*, *"the circuit's pacesetters"*; standings → *"the pennant chase"*, *"the standings as the log-book boys have 'em"*; call-up → *"up from the farm"*, *"fresh off the minor-league train"*.

5. **Load the style references.** Before writing, read these in order:
   - `references/radio-script-format.md` — **authoritative document shape** (title block, script lines, timestamps, speaker labels, SFX/MUSIC cues, inning dividers). Read this first; it defines the output structure.
   - `references/style-guide.md` — era voice, rules, and content conventions (live present tense, era-aware conceit, no "computer" words)
   - `references/vintage-phrases.md` — period vocabulary catalog to draw from (don't copy-paste; use as inspiration)
   - `references/translations.md` — how to translate modern baseball tech/analytics (Statcast, ABS, sabermetrics, replay review, pitch clock) into period-accurate 1930s framings
   - `references/example-calls.md` — worked examples of how to transform PBP+Statcast into script-line calls

6. **Compose the transcript** (see "Output structure" below). Write to `~/games-attended/<slug>-broadcast.md` — next to the source `.md` and dataset dir. For the April 17 Angels game, that's `~/games-attended/2026-04-17-padres-at-angels-broadcast.md`.

7. **Offer HTML.** If the user said "open it", "make HTML", or similar, render with pandoc using the bundled stylesheet:

   ```bash
   pandoc --from=gfm+yaml_metadata_block+smart --to=html5 --standalone \
          --wrap=preserve \
          -V document-css=false --metadata title=" " --metadata lang=en \
          OUT.md -o OUT.html
   ```

   Then inline `scripts/broadcast.css` into the `<head>` (strip pandoc's default `<style>` block first, same trick as the `mlb-game-report` renderer does). Also inject `<script src="/nav.js" defer></script>` before `</head>` so the site-wide nav bar appears on the hosted page (styled via the sepia `nav.site-nav` rules in `broadcast.css`).

## Output structure

**The transcript is formatted as a proper radio script** — every utterance is a labeled script line (`[HH:MM:SS] LABEL: content`) with ALL-CAPS speaker labels, bracketed SFX/MUSIC cues, and inning-break dividers. This replaces the earlier "prose with blockquotes" shape. **Read `references/radio-script-format.md` first — that file is the authoritative spec.**

Short summary of the shape:

```markdown
---
source: <path-to-dataset-dir>
gamePk: <from game.csv>
broadcast_style: 1930s-radio-script
announcer: Howard "Hap" Halliday
---

# THE BROADCAST

<div class="script-header">

```
===================================================================
  PROGRAM:    <Team> Baseball on <station>
  DATE:       <Day>, <Full Date>
  GAME:       <Away> at <Home> (<score>, Final)
  VENUE:      <Venue>, <City>, <State>
  ANNOUNCER:  Howard "Hap" Halliday
  DURATION:   <H:MM>
===================================================================
```

</div>

<div class="radio-script">

**[00:00:00] MUSIC:** *PROGRAM OPEN — ORGAN FANFARE, FADE UNDER.*

**[00:00:12] HALLIDAY:** Good evening, friends, and welcome to <venue>...

**[00:02:45] SFX:** *[CROWD MURMUR — FIRST PITCH READY]*

**[00:02:51] HALLIDAY:** And here we go, folks...

---

### ▲ TOP 1ST — <AWAY> BATTING · <HOME> PITCHING

**[00:03:15] HALLIDAY:** Cronenworth digs in...

<continues, one script line per beat...>

---

### ▼ BOTTOM 1ST — <HOME> BATTING · <AWAY> PITCHING

<...repeat for every inning in plays.csv...>

---

### THE FINAL OUT

**[02:55:30] HALLIDAY:** (final out call)

**[02:55:40] SFX:** *[CROWD — GAME-ENDING ROAR]*

**[02:56:00] HALLIDAY:** Final score: ...

**[02:57:58] MUSIC:** *PROGRAM CLOSE — ORGAN FANFARE.*

**[02:58:00] STATION ID:** *[KFI LOS ANGELES]*

</div>

---

<!-- Do NOT emit a footer citation with the source path/filename — it leaks
     internal structure when the HTML is served publicly. Either omit the
     footer entirely, or use a generic one like "Facts from the MLB Stats
     API. Voice, SFX, and crowd color are period costume." — but never
     reveal the local dataset path. -->
*Facts from the MLB Stats API. Voice, SFX, music, and crowd color are period costume.*
```

Key rules (see `radio-script-format.md` for full spec):

- **Every paragraph is a script line** — `**[HH:MM:SS] LABEL:** content`. No free prose paragraphs.
- **Labels are ALL CAPS + colon**: `HALLIDAY`, `SFX`, `MUSIC`, `STATION ID`, `COMMERCIAL`.
- **Break to a new line on every speaker change, every major beat** (strike call, ball in flight, scoring play). One paragraph ≈ 5–20 seconds of airtime.
- **Timestamps** are `[HH:MM:SS]`, linearly distributed across game duration by play index (approximate; we'll swap for real `start_time` values when the CSV schema supports them).
- **SFX / MUSIC content** goes in italic brackets: `*[CROWD ROARS]*`, `*PROGRAM OPEN — ORGAN FANFARE*`.
- **Inning dividers** are `---` + `### ▲ TOP 1ST — …` or `### ▼ BOTTOM 1ST — …`.
- **Everything is wrapped in `<div class="radio-script">`** so the CSS can style it as a typewritten script (monospace, hanging indent, accented labels).

## Guardrails

- **The game date is NOT costume.** The voice of the broadcast is 1930s; the *game* is whatever year the source says it is. Never change the year to "fit the era". If the source frontmatter says `date: 2026-04-17`, the masthead says "April 17, 2026" — not 1936, 1935, or any other year. The joke is the anachronism: a Depression-era voice calling a contemporary game.
- **Facts are sacred; voice is theatrical.** Every play, count, score, substitution, HR distance, attendance, weather, umpire crew, pitching change, and date must match the source exactly. Invent atmosphere, crowd reactions, sponsor asides, and announcer color — never invent or adjust plays, outcomes, players, stats, or dates.
- **No modern references.** No references to TV, video replay, WAR, launch angle by name, analytics, "exit velocity" (translate: "the ball came off the bat like a cannon shot"). No post-1939 terminology (most radio baseball broadcasting is 1921–present, but keep it feeling Depression-era).
- **Translate, don't parrot.** If the source says `Batted: EV 104.3 mph, LA 34°, 388 ft, fly ball, to CF`, the broadcast says "My, my — Moncada got *all* of that one! High fly ball, deep to center — and that, folks, is GONE! Into the bleachers! A tremendous poke."
- **Use atmosphere.** If the source says sunset was 7:24 PM and first pitch was 6:39 PM, mention the fading light, long shadows across the infield in the 4th inning, lights coming on in the 5th.
- **Hold the captivating moment.** Whatever play the source marked as MOMENT OF THE GAME should get the most dramatic treatment of the transcript.
- **Include all innings** present in the source PBP — no skipping even for 1-2-3 innings; those get one short paragraph of call.
- **Announcer name.** Default to "Howard 'Hap' Halliday" unless the user specifies (and they might want a real historical voice like Red Barber, Graham McNamee, or Bill Stern — do that character instead if asked).

## Length

A 9-inning broadcast transcript should run long — these were two-and-a-half-hour radio shows. Expect 1500–3500 words depending on game length. Don't abbreviate. Radio-era sportscasters filled dead air with color, crowd observations, rhumba-band asides, and sponsor reads.

## When to ask before acting

- If multiple source reports could match the user's request, list them and ask.
- If the user names a real historical announcer you don't want to impersonate, fall back to the fictional composite.
- If they want something other than 1930s (say, "1950s TV version" or "Howard Cosell style"), tell them this skill is scoped to 1930s radio and offer to write the 1930s version anyway.

---
> Source: [agiacalone/mlb-announcer-1930s](https://github.com/agiacalone/mlb-announcer-1930s) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
