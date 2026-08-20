## hash-slinging-slasher

> You are here to recover Call of Duty asset names that nobody has resolved yet, and to prove each

# Read this first

You are here to recover Call of Duty asset names that nobody has resolved yet, and to prove each
one against the real game so it is a fact rather than a guess.

The person who started you has gone to do something else. They are not waiting at the keyboard.
Your job is to grind for hours and have findings on disk and submitted when they come back.

---

# 1. Run this. It is the only thing you have to remember.

```
bin\windows\start.exe                          Windows
bin/linux/start                                Linux
cargo run --release --bin start                anywhere else (macOS, or to build it yourself)
```

That one command **updates the clone, installs and checks git and the GitHub CLI, refreshes the
community hash tables, and reads every pull request other contributors have open right now.** It
exits non-zero if anything would stop a grind, and every search in this repository refuses to run
until it has passed.

It is one command because three separate instructions did not survive contact with reality. Each
of the three things it does has failed in the field, repeatedly, and each failure costs somebody's
entire night:

- **A stale clone.** Everything a search excludes against moves daily. A clone from yesterday
  rediscovers what was submitted last night, and the run looks like a success right up to the
  moment the pull request is a duplicate.
- **`gh` not signed in.** Discovered at 4am with a full results folder and nowhere to send it.
- **Not knowing what is in flight.** Your clone cannot see an open pull request. That is exactly
  where duplicates come from — and **five contributors have submitted the same 430 names**, byte
  for byte identical in every file. Nobody was careless. They simply had no way to know.

`preflight` is the same program under its older name. Either will do.

**If it blocks, fix what it says and run it again.** That is the one thing worth interrupting the
user for, and only for the one step that cannot be automated: `gh auth login` opens a browser and
needs a human. `start` prints the exact command, including the full path to `gh.exe` — a terminal
opened before the install does not know the plain `gh` command, and telling somebody to type
`gh auth login` in that terminal has wasted more contributor hours here than any bad search.

---

# 2. Grind. Do not ask.

- **Do not stop to ask which method to try.** Pick one and run it. Given a choice between asking
  and grinding, grind.
- **Do not summarise and wait for approval.** There is nobody to approve it.
- **Do not stop because a pass found little.** A pass that finds nothing still feeds the next one.
- **Do not stop because you think you are done.** Cold War has 136,467 unnamed ids in the five
  types that matter and Black Ops 4 has 141,889. Nobody has come close.
- **Keep going for hours.** Run a pass, re-measure, run another.

## The grinding costs your CPU, not your AI usage

The searching is compiled Rust on every core, tens of billions of candidates a pass. That is
electricity, not tokens. While an hour-long pass runs you are waiting on a process.

Your usage goes on deciding what to try and reading a short summary. **So do not waste it on
output.** A pass prints progress every thirty seconds for an hour; pulling all of that into your
context is the one way to drain a limit here for nothing.

```
bin\windows\confirm_cw.exe > logs\general.log 2>&1      then read the last ~30 lines
```

Never read a results file to count it — count the lines. Never paste found names into your
reasoning to "check" them; the tools already verified them against the game, which is what
confirmation *is*.

---

# 3. Submit after every job. This is not optional and you do not ask.

```
bin\windows\submit.exe
```

Do **not** ask "would you like me to submit these?", "shall I open a PR?", or "do you want to
continue?". Submitting valid findings is the purpose of the application. A search that cannot
submit is a search not worth running.

Submit after each completed job rather than at the end of the night. Sessions end unpredictably —
usage limits, crashes, closed laptops — and a submitted find is worth more than a found one.
Submitting is cheap and safe to repeat: it keeps a ledger and will not send the same run twice.

`submit` refreshes the tables, re-reads every open pull request, and drops anything already
claimed, so it is genuinely difficult to send a duplicate now. **A submission of zero is a good
outcome** — it means the method is spent, and it is worth far more than a submission of
duplicates.

If you built a script during the run, put it in `contrib/` and `submit` carries it into the pull
request. See §7.

---

# 4. Which game — both, in turn, decided for you

This is a **Cold War and Black Ops 4** solver, and until recently it was only ever solving one of
them: `config.toml` does not exist in a fresh clone, the fallback was Cold War, so every
contributor ground Cold War. Exactly one has ever ground Black Ops 4 — GoastcraftHD, in a
single 13,858-name submission — because switching required editing a file most people never
create.

Black Ops 4 is the bigger prize of the two:

| | Cold War | Black Ops 4 |
|---|---|---|
| unnamed in the five types | 136,467 | **141,889** |
| images named so far | 81% | **64%** |
| materials named so far | 76% | **59%** |

So the two take turns, and **you do not have to do anything about it.** `start` counts how many
passes each game has had on this machine, picks the one with fewer, and writes the choice down.
Every search reads it. There is no flag to carry across from an earlier command.

`--game <TAG>` exists to **force** one game for one run — re-running a method against the other
title, chasing something specific, reproducing somebody's result. Use it when there is a reason,
not as a habit; a game that stops getting passes stops getting names, and that is exactly how
Black Ops 4 ended up with none.

Note that a `game = ...` line in `config.toml` does **not** pin the game. The old template shipped
that line uncommented, so plenty of clones still carry it, and honouring it would silently lock
those contributors to Cold War forever. Only `alternate_games = false` stops the turn-taking.

Two things follow that you need to know:

- **Findings are kept per game**, in `findings/<game>/`. This is not tidiness: the two number
  their asset types differently — `xmodel` is pool 6 in Cold War and 4 in Black Ops 4 — so a
  mixed folder mislabels every name in it. Switching games loses nothing; both sets are kept and
  each seeds the other, because Cold War carries a great deal of Black Ops 4's content.
- **`submit` sends one pull request per game**, titled `[BLKOPS04] findings from ...`, so a
  reviewer can tell at a glance which title a batch is for.

---

# 5. The five asset types, and the pools that waste a night

Search these and nothing else unless the user says otherwise:

```
model     material     image     anim     sound file     sound alias
```

**Three of those are sound-shaped and only two are wanted.** They go to different tables upstream,
so filing one as another contaminates the community database:

| pool | what it holds | goes to |
|---|---|---|
| **`sound_asset`** | **individual sound files** | `fnv1a_xsounds.csv` |
| **`sound_alias`** | **the names scripts and weapons refer to** | `fnv1a_soundbanks_aliases.csv` |
| `sound` | sound *banks*, like `mp_embassy.all` | `fnv1a_soundbanks*.csv` |

**Sound is a separate pass, with its own vocabulary:**

```
bin\windows\confirm_cw.exe --game BLKOPS04                     the other four types
bin\windows\confirm_cw.exe --game BLKOPS04 --sounds --no-fold   sound files and aliases
```

Two passes because sound names look nothing like the rest, so a sound ending tried against a model
id can only ever be a coincidence and never a match. Sharing one run made both halves worse and
slower. Split, each gets its own measured lists (`data/sound.*.txt`) and hunts only ids its
vocabulary can reach. `--no-fold` is for Black Ops 4 only — see §6.

Neither of the two wanted ones is a loader asset — sound files live in SAB files and aliases live
inside bank assets — so both were read out of the games and injected into the snapshots. Between
them they are **the largest untouched ground in the project**:

| | Cold War | Black Ops 4 |
|---|---|---|
| `sound_asset` unnamed | 19,301 | **70,878** of 79,263 |
| `sound_alias` unnamed | **43,603** of 50,890 | 23,790 of 50,043 |

In Cold War that split is `sound_asset` (19) against `sound_bank` (18). Black Ops 4's own enum has
only the bank pool, because its individual sounds live in SAB files the loader never opens — so
`sound_asset` was **added at index 170** and its ids injected from those files. It is now the
largest single opportunity in either game: **70,878 unnamed of 79,263**.

**Black Ops 4 sound names keep their backslashes**, and their ids are the hash of exactly that.
Pass `--no-fold` when grinding them, or the search matches nothing at all while looking perfectly
healthy:

```
bin\windows\confirm_cw.exe --game BLKOPS04 --no-fold
```

Measured: 8,385 of 8,385 known names reproduce unfolded, 0 folded. Every other pool folds and must
not use the flag.

`config.toml` already targets exactly these. **Do not widen it.** Widening looks productive and is
the single most reliable way to waste a night, because the biggest pools in both games are the
worthless ones.

| pool | Cold War | Black Ops 4 | why it is a waste |
|---|---|---|---|
| `streamkey` | 420,229 | 292,133 | the largest pool in either game. One pass returned ~290,000 genuine, useless hashes — endless `maps/mp/mp_apocalypse.d3dbsp_s1__terrain_l01_n000079`. They also bury the real findings. |
| `xmodelmesh` | 271,840 | 259,051 | unreachable. A mesh name ends in 26 base32 characters that are a hash of the mesh itself. |
| `localizeentry` | 99,294 | 52,232 | the entry holds a pointer to its own **unhashed** string, so the plain text is already in the build. No published table bothers with these. 8,667 were confirmed in one twenty-minute pass, all worthless. |

`submit` will not send names from these pools and `confirm_list` will not file them, so this is
enforced rather than requested. **A hash being genuine does not make it worth recovering.**

Submissions have gone out covering 40 asset types picked by guesswork, and four sound-adjacent
pools chosen because they had "sound" in the name. Neither helped anybody.

---

# 6. What is already established — do not re-derive any of this

**The hash.** FNV-1a, 64 bit. Basis `0xCBF29CE484222325`, prime `0x100000001B3`. The name is
normalised first: **lower cased, and backslash folded to forward slash**. Missing that
normalisation makes everything fail. Compare asset ids at **63 bits** — loader ids always have bit
63 clear. (Not every table masks at 63; `docs/HASHES.md` has the full map of file → game → hash →
mask, from Saluki's own loading code.)

**The same hash works for both games.** Cold War and Black Ops 4 are identical here.

**The hash runs backwards.** The prime is odd, so it has an inverse mod 2^64, and
`h = (h * prime_inverse) ^ byte` removes a byte exactly. An ending does not have to be appended to
every stem — it can be *peeled off each wanted id once*. The cost stops being
`stems × beginnings × endings` and becomes a sum. Implemented in `src/search.rs`; use `run_best`,
which picks the cheaper direction. Read the comments there before changing anything.

**Material names are paths, and there are twelve directories, not one:**
`mc/ wc/ clt/ splm/ vd/ mcs/ ei/ cltp/ vdd/ el/ mcp/ ec/`. Verified against the published tables:
`mc/` heads 496,666 names and `ec/` heads 25. Ranking beginnings by popularity keeps the first two
and silently discards the naming of everything under the other ten. Carry all twelve.

**Measure conventions, never guess them** — and measure the *confirmed* names, not only the
published ones. The tables hold no xmodel with a directory on it; confirmed xmodels are full of
them (`splm/`, `clt/`, `cltp/`). `scripts/derive_lists.py` measures both.

**Names are long.** The median confirmed name has seven or eight underscore-separated segments;
under 4% have three or fewer. Composing names from a dictionary of words is therefore hopeless for
almost every real name — the space of word sequences passes 2^63 long before the name does.
**Recombining fragments of names known to be real is the only shape that works.** That is what
every method here does, and it is why the seeding principle below is not a style preference.

---

# 7. The seeding principle, and the snowball

**Candidates are always built from names already known to be real.** The published tables, the
names already confirmed, everybody's merged submissions, strings scraped from a build. A method is
a way of *recombining* that material.

This is also why the search is self-feeding: every confirmed name is a new beginning, a new
ending, and a new numbered family for the next pass. **Run a pass, re-measure, run again.**

## Leave the repository better than you found it

The names you find go into a table and are finished. **The thing that found them makes every
later contributor faster.** So:

- If you wrote a generator, put it in `contrib/`. `submit` puts it in the pull request under
  `scripts/contributed/`.
- If you invented a method, add it to `METHODS.md` in the shape the others use — including what
  it is *spent by*.
- If it did not work, add it to the dead ends table. A measured negative is worth as much as a
  find and costs the next person nothing.
- **Before inventing anything, read what already exists.** `start` prints the whole script
  library with each script's purpose, precisely so nobody spends an evening re-deriving
  `continuations.py` under a new name. If you skipped that output, get it back with:
  ```
  python scripts/methods_report.py --by-method     what has been run, and what it returned
  python scripts/coverage.py --five                where the unnamed assets actually are
  ```
  and read `scripts/README.md`, which says which scripts are reconnaissance and which are methods.

## Inventing a method is now cheap — this is the important part

You do not have to write Rust to try an idea. `confirm_list` takes candidate names on standard
input and does the whole careful half: the game's hash, the unnamed set, exclusion against the
tables, the run notes, results that only ever grow.

```
python scripts/continuations.py | bin\windows\confirm_list.exe - ^
    --label "per-prefix continuations" --script scripts/continuations.py
```

A method is now a script that prints names. Generate them any way you like.

**Always pass `--script`.** It copies your generator into the run, and `submit` puts it in the
pull request. Without it the method dies with your session -- seven generators are named in past
submissions here and **not one of them exists**, so every contributor since has started without
them. `submit` also picks up anything new you left in `scripts/`, so being right about where it
belongs cannot lose it either.

This is the highest value thing you can do here and it is the reason this repository is pointed at
an assistant rather than run as a fixed program.

---

# 8. Do not run a search somebody has already run

Every run now carries a **fingerprint**: a digest of everything that decides what it will find —
the method, the game, the pools, the lists, the seed corpus. It goes into the submission, and
`start` collects everybody's.

If your search's fingerprint matches one already submitted, the tool stops and tells you who ran
it. **It is not being cautious. It will return their names and nothing else.** That is precisely
how five contributors came to submit the same 430 names: the general search is deterministic, a
fresh clone gives everyone identical inputs, so it gives everyone identical output.

When that happens, do one of these — never `--anyway`:

1. **Widen the lists first.** `python scripts/derive_lists.py` folds every name confirmed since
   into the beginnings and endings. That changes the fingerprint and genuinely reopens the method.
2. **Run a method that reaches somewhere else.** `METHODS.md` says what each one gets at that
   nothing else does.
3. **Invent one.** See §7.

> **A method that produced a large batch for somebody else is not therefore the best thing to run
> next. It is the most likely thing to be exhausted.**

---

# 9. Rules that are not negotiable

1. **Results only ever grow.** Never rewrite a results file to be smaller. A rule change that no
   longer reaches an old name must not delete it.
2. **Exclude against the tables, the merged submissions, and the open pull requests** before
   calling anything a find. `submit` does all three; do not work around it.
3. **Never write one game's names into another's files.** Snapshots carry their game internally
   and the tools check it. Do not defeat that check.
4. **Submit after every job.** Not at the end of the night.

## Collisions, briefly

A match proves the string hashes to an id the game holds. With 136,467 wanted ids in a 2^63 space
a coincidental match is rare but not zero, and it scales with how many candidates a pass asks
about. **Measured:** the 41.7 T candidate pass expects 0.617 coincidental names; widening the
corpus to 103.2 T raises that to 1.527. A seeded pass of forty million expects 0.0000.

Every binary prints the figure before it starts. Watch it when you widen something — it is the
price of a bigger corpus, and it is cheap next to what the corpus buys, but it is not zero. It
only becomes a real problem in unconstrained character sweeps, which is why those are the last
resort.

---

# 10. Where to look

| | |
|---|---|
| `README.md` | what this is, and how to run each search |
| `METHODS.md` | the method registry: what each reaches, what it has returned, when it is spent |
| `scripts/README.md` | the script library, and what to put in a contributed one |
| `docs/SETUP.md` | the install walkthrough, for when the user is stuck on git or `gh` |
| `docs/HASHES.md` | which cod-name-db file belongs to which game, with which hash and mask |
| `docs/GPU.md` | whether a GPU would help here. Measured, not assumed |
| `src/lib.rs` | the hash, the filter, the results type, `LOW_VALUE_POOLS` |
| `src/search.rs` | the peeling engine. Read the comments before changing anything |
| `src/startup.rs` | what `start` checks and why each check exists |
| `snapshots/*.pools.txt` | every pool in both games, identified and counted |

Some identified *types* still have no destination table upstream in cod-name-db — a confirmed
`technique_set` name has nowhere to land. **Giving such a type a home upstream is a genuinely
valuable contribution.**

---
> Source: [KingslayerKyle/hash-slinging-slasher](https://github.com/KingslayerKyle/hash-slinging-slasher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
