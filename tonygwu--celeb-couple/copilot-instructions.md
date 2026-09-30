## celeb-couple

> Several agents work this repository at the same time, one agent per checkout.

# Notes for coding agents working in this repo

Several agents work this repository at the same time, one agent per checkout.
Git is the only channel between them. Read this file before your first git
command.

## Checkout layout on this machine

`~/Code/misc/celebrity-couple/` is a **container**, not a checkout. It holds
several checkouts of this same repository:

| Checkout | Role |
|---|---|
| `repo-0` | Working checkout for agent sessions. Edit here. |
| `repo-1` | Working checkout for agent sessions. Edit here. |
| `repo-2` | Working checkout for agent sessions. Edit here. |
| `repo-3` | Working checkout for agent sessions. Edit here. |

There is no deploy or install checkout yet. If this project ever gains one,
add a `repo-prod` checkout, give it the deploy-only role, and record that role
in this table in the same commit. A role that lives only in an agent's memory
does not exist.

Your checkout name is your **handle**. Use it when you claim work.

## The rules

1. **Sync before you touch.** `git fetch && git rebase` before you read or
   edit. Your local HEAD is a hypothesis about the repo, not a fact. A session
   that resumes after idle time re-fetches before its *first* git command, not
   before its first push.

2. **Push every completed unit, immediately.** Never end a turn with a
   publishable change sitting unpushed. If something blocks the push, say what
   it is. A held commit and a hung agent look identical from outside.

3. **Stage by name, never by sweep.** `git add -A` and `git add .` are
   forbidden here. Untracked files you did not create are another agent's work
   in progress. A tree dirty with foreign files is normal, not a mess to tidy.

   `git add -u` is the obvious thing to reach for once `-A` is ruled out, and
   it is only *safer*, not safe: it skips untracked files but still sweeps
   every TRACKED file that has been modified, including ones another agent is
   part-way through. Naming the files costs a few seconds and cannot take
   somebody else's work with it.

4. **Claim before you work.** Push the claim before you start. Add
   `[claimed by: repo-N]` to the item's line, commit that file by name, push.
   If the claim push is rejected, another agent won the item. Fetch and pick
   another.

5. **Divergence is a blocker.** More than 5 commits ahead, or any commits
   behind for more than a day, gets surfaced to the operator. Never let it
   accumulate quietly.

6. **Committed code must run from any checkout, any machine, any
   environment.** Never write a checkout-absolute path into committed code.
   Derive the repo root at runtime: `git rev-parse --show-toplevel`, or
   `Path(__file__).resolve()` in Python. A hardcoded path is correct in one
   checkout and wrong in three.

   Two other flavours of the same defect, both found in this repository:

   - **A home-absolute path.** All six quota-spending scripts defaulted
     `--account` to `/Users/tonygwu/.claude-e`, which exists on one machine
     and, on the night it was found, had 0% quota left. Use
     `packages/llmkit/accounts.py`, which refuses rather than guessing.
   - **A path that exists here and not in CI.** Three tests spawned
     `repo/.venv/bin/python`. CI installs with setup-python and has no
     `.venv`, so all three would have failed on its first run — and CI has
     never run, so nobody found out. Use `sys.executable`.

   The test for the first is that a fresh clone on another machine works. For
   the second it is `python3 -m venv` plus plain `pip install -r
   requirements.txt`, which is what the workflow does.

7. **A refusing hook is another agent talking to you.** Read its text and the
   sentinel file it names. Never `--no-verify`, never delete the hook.

8. **History rewrites: stop, fetch, reconcile.** Unknown SHAs or a
   non-fast-forward rejection mean the remote history may have moved under
   you. Stop, fetch, and reconcile against the coordinator notice at the top
   of this file. Never force-push over `main`.

9. **One session, one terminal.** Never resume the same session id from two
   checkouts at once. Concurrent appends corrupt the transcript.

## Staging by name is not enough: check the index

`git add <names>` stages those names. **`git commit` then commits the WHOLE
INDEX**, including anything another agent staged and had not yet committed. In a
shared clone that is a real and easy mistake, and rule 3 above does not prevent
it.

It happened on 2026-09-15. A commit meant to carry two files carried eight: 502
lines of `expand_roster.py`, 270 of its tests, `modules/records/wikidata.py` and
three more, all another agent's in-flight work sitting staged in the index.

**Treat a populated index in this clone as somebody else's work in progress.**
Before every commit:

```sh
git diff --cached --name-only        # is anything here not yours?
git commit -- path/one path/two      # commits ONLY these paths, index or not
```

`git commit -- <paths>` is the safe form: it ignores the rest of the index
entirely, so a foreign staged file cannot ride along.

The same incident showed a second thing worth knowing: because the commit took
the index, it took the INDEX's copy of a file that was also dirty in the working
tree. The two differed. Check `git show HEAD:<file>` after committing a file two
agents are editing, rather than assuming your version is the one that landed.

## Git identity

Every checkout is configured with the GitHub noreply author address:

```sh
git config user.email 446441+tonygwu@users.noreply.github.com
```

This is set so the history never needs an author rewrite if the repository is
made public later.

## Setup, in a fresh clone

```sh
uv venv --python 3.12 .venv
uv pip install --python .venv/bin/python -r requirements.txt
.venv/bin/python -m pytest tests -q      # offline; spends no model quota
```

`requirements.txt` gained `jsonschema` on 2026-09-14. It validates every stored
judge verdict against `rubrics/standing/estimate.schema.json`, which was only
safe once the schema's nullable enums were fixed: before that, the first thing a
validator did was reject all 52 raw verdicts.

**CI does not run.** `.github/workflows/tests.yml` exists, and every one of its
runs — over a hundred now — has been refused at GitHub's billing gate before
reaching pytest, so the repository's Actions state is red while the suite is
green. Do not read a green
Actions badge as verification and do not read a red one as a test failure. Run
the suite locally; that is the only gate that has ever executed. See
`docs/BACKLOG.md`.

The suite is offline and deterministic. The count is deliberately not written
down here: a hand-typed number goes stale and then lies, which is the defect
this project's sibling repo paid for five separate times. Run it and read it. **Treat any non-zero exit as failure,
including 5**, which pytest returns when it collects no tests at all.

**Read pytest's own exit status, not something downstream of it.** Both of
these report the wrong command's status and will happily let you commit a red
suite:

```sh
pytest tests -q; echo $?              # the status of echo
pytest tests -q | tail -2 && git commit    # the status of tail
```

Use this instead, which is what the rest of this repository's tooling does:

```sh
.venv/bin/python -m pytest tests -q > /dev/null 2>&1; RC=$?
[ $RC -eq 0 ] && git commit ...
```

That is not a hypothetical. The pipeline form pushed a failing test twice in
one night, and it caught a third on 2026-09-14: `git push origin main | tail -3;
echo "push RC=$?"` reported 0 while the push had not happened, and the clone sat
one commit ahead with a green-looking result.

**Do not run two `pytest tests` invocations at once.** The suite takes about 13
seconds. Two concurrent runs, plus a git operation, took 17 and 34 minutes on
2026-09-14 and looked exactly like a performance regression. Collection is
0.43s and no single test exceeds 2.9s, so if the suite ever seems slow, check
what else is running before looking for the cause in the code.

## Checking that the reports reproduce

`tests/test_doc_claims.py` proves a generated document is unchanged by
re-running its generator IN PLACE. That does not prove it is independent of the
checkout it was generated in, which is AGENTS.md rule 6 applied to the
deliverable rather than to code.

To prove that, regenerate in a different clone over the same artifacts:

```sh
D=$(mktemp -d) && git clone -q . "$D" && cp -R data "$D/data"
cd "$D" && python3 -m venv .venv && .venv/bin/pip install -q -r requirements.txt
bash scripts/run_chain.sh reports
diff docs/M0-REPORT.md <original clone>/docs/M0-REPORT.md
```

**Measured 2026-09-14:** all five generated documents came out byte-identical
-- `M0-REPORT.md`, `SCALING.md`, `REACHABLE-PRODUCTS.md`, `GROUNDING-AUDIT.md`
and `RELATIONSHIP-REVIEW.md` -- including the report's input fingerprint
`1d539901f1b8`. Nothing in the reports depends on where they were generated.

This is deliberately NOT in the suite: it clones the repository and builds a
virtualenv, which is the wrong cost to pay on every `pytest` run. Do it when a
generator changes, or before handing the reports to anyone.

## The tools, and how to invoke them

Run every one from a clone root with `.venv/bin/python`. The ones marked **yes**
in the last column spend model quota; every other one is read-only and offline
apart from the Wikidata and Wikipedia fetchers. Check `quotapick status` before
running anything in the first group.

| Command | What it does | Spends quota |
|---|---|---|
| `scripts/fetch_records.py` | relationship candidates + birth dates for the cohort | no |
| `scripts/build_episodes.py` | merges progressions, flags defects, applies the adult window | no |
| `scripts/build_partner_universe.py` | derives the outside-roster partners from scorable episodes | no |
| `scripts/fetch_onscreen_candidates.py` | films where a male and a female roster member co-star. Batched, and REFUSES a batch that comes back at its row cap | no |
| `scripts/resolve_roster.py` | turns a name list into a roster file with Wikidata ids | no |
| `scripts/derive_pairings.py` | derives every pairing gap by SUBTRACTING two cached person-year scores, so an already-judged pair costs nothing; a pairing whose two people are not both cached is emitted with `gap: null` and a reason rather than dropped | no |
| `scripts/expand_roster.py` | snowballs a roster over co-stars (P161) and real-life partners (P26/P451), bounded, recording what each round added and what each bound cut. `--roster docs/roster-100.json --out docs/roster-expanded.json` | no |
| `scripts/build_imdb_graph.py` | seed filmographies + full billed cast from the local IMDb dumps. Needs `--dumps <dir>` or `CELEB_IMDB_DIR`; the ~2GB `.tsv.gz` files are NEVER committed. Person-first on purpose: billing order is unreliable for picking a couple and reliable for listing a person's films | no |
| `scripts/build_person_gradings.py` | averages EVERY grading of a person-year into one canonical score, carrying `n`, `spread` and the models behind it. `--exclude-model claude-sonnet-5` drops one without re-grading anything | no |
| `scripts/resolve_imdb_people.py` | bridges IMDb `nconst` to Wikidata `QID` via P345, so a person found through IMDb can be scored against evidence the project already holds. Joins on the id, NEVER on a name | no |
| `scripts/build_imdb_pairings.py` | joins couples + the P345 bridge + film years into pairings the boards read. Counts every drop by reason; sex comes from Wikidata P21, never from IMDb's actor/actress credit | no |
| `bash scripts/refresh_boards.sh` | the whole free chain over whatever has landed: bridge (only if new people appeared) → pairings → canonical gradings → HTML. Safe to run WHILE the paid runs are still writing | no |
| `bash scripts/deploy_site.sh` | publishes the board `refresh_boards.sh` last rendered to https://celebrities.tonygwu.com/ , as Cloudflare Worker `celebrity-couple`. Renders nothing itself, so run `refresh_boards.sh` first. Refuses a source under 400KB or missing its title or DATA blob, then re-fetches the live URL and refuses if what came back is not this board. `--dry-run` stages `./site` and publishes nothing | no |
| `scripts/build_reallife_pairings.py` | EVERY scorable episode becomes a real-life pairing. Not `select_m1_slice.py`, which narrows to 10 focal actors and is why the real-life boards showed 4 people when the episodes file held 228 relationships | no |
| `bash scripts/render_og.sh` | renders `web/og-card.html` to `web/og.png` at 1200x630, the picture Twitter and LinkedIn show. Deliberately carries NO scores so it cannot go stale; re-run only when the card changes | no |
| `scripts/scaling_report.py` | pilot vs full roster; does coverage scale | no |
| `scripts/source_requirement.py` | how deep a source would have to be | no |
| `scripts/reachable_products.py` | what can be built with the evidence that exists | no |
| `scripts/fetch_observations.py` | list/award observations from Wikipedia tables, for cohort **and** partners | no |
| `scripts/merge_prose_mentions.py` | folds prose mentions in, deduping on person+year+publisher | no |
| `scripts/joint_coverage.py` | are BOTH sides scorable in the same period, and is the pairing comparable | no |
| `scripts/evidence_density.py` | observations per person-period — the actual bottleneck | no |
| `scripts/alignment_gap.py` | what the nearby-period bound costs, per pairing | no |
| `scripts/partner_eligibility.py` | splits "no evidence found" from "never rate" | no |
| `scripts/shape_confound.py` | how much of the estimate is evidence format | no |
| `scripts/gender_shape_confound.py` | is that format effect aligned with gender — the project's most consequential finding | no |
| `scripts/offset_diagnostic.py` | cross-gender offset sensitivity; exits non-zero if the identity breaks | no |
| `scripts/grounding_audit.py` | automated grounding checks + the human review sheet | no |
| `scripts/write_m0_report.py` | renders `docs/M0-REPORT.md` from the JSON artifacts | no |
| `scripts/corroborate_relationships.py` | pulls Wikipedia's own sentences about each relationship so the review sheet shows evidence, not just Wikidata links | no |
| `scripts/relationship_review.py` | the human review sheet for relationship claims, load-bearing ones first | no |
| `scripts/conclusion_robustness.py` | do the capstone's claims survive other settings of the bound and the romance filter | no |
| `scripts/verify_trap.py` | re-derive THE-TRAP.md's three structural claims; exits 1 if any stopped holding | no |
| `scripts/cross_run_stability.py` | do two separate runs score an identical dossier the same | no |
| `scripts/audit_doc_numbers.py` | cross-checks measured numbers typed into any tracked Markdown against the artifacts; exits 1 on a stale one | no |
| `scripts/verify_observations.py` | re-checks every observation against the live Wikipedia page it came from; exits 1 if one stopped matching | no |
| `scripts/verify_identities.py` | checks every roster Wikidata id is the person named, and is a human; exits 1 if not. `--roster docs/roster-100.json` for the roster scale | no |
| `scripts/absence_audit.py` | for cohort members with no observations, whether any permitted source names them at all — separates a pipeline gap from a source gap | no |
| `scripts/identify_couples.py` | which characters in each seed film are a couple, with a `central`/`substantial`/`incidental` weight. Cached per (film, family); `--backlog-only` spends nothing. Defaults to `claude-opus-5`, NOT Fable: this is factual recall and must not spend Fable's separate weekly allowance | **yes** |
| `scripts/run_stress.py --out data/pilot/stress` | the eight measurement stress cases | **yes** |
| `scripts/score_evidenced.py` | scores person-periods that have evidence | **yes** |
| `scripts/classify_romance.py` | is a co-starring pair actually a romance in the film | **yes** |
| `scripts/extract_prose_mentions.py` | list memberships from biographical prose | **yes** |
| `scripts/measure_rater_noise.py` | repeat-scores unchanged dossiers | **yes** |
| `scripts/run_pilot.py` | bounded pairing selection and coverage | **yes** |

A re-score does not destroy the previous estimates any more: `score_evidenced.py`
archives both score artifacts under `data/pilot/run/history/` before overwriting,
content-addressed so an unchanged re-run does not pile up copies. To compare two
runs of the same corpus, point `cross_run_stability.py --b-scores` at a history
file. Run-to-run variance is real — 2 of 3 identical dossiers moved by
2.0 points — so both runs have to survive for it to be measurable.

| `scripts/score_roster_joint.py` | scores only what the roster-scale joint pairings need | **yes** |

### Why `identify_couples.py` does not use Fable

It answers a factual question about a film, so it does not need the model the
appearance scoring needs, and Fable has its own weekly allowance that is
usually the scarcest thing on the account. Measured on the same ten films:

| check | fable | sonnet | opus |
|---|---|---|---|
| recovers Minnie Driver, unbilled in Good Will Hunting | yes | yes | yes |
| rejects Armageddon's father/daughter pair | yes | **no** | yes |
| finds Cristin Milioti, unbilled in The Wolf of Wall Street | yes | **no** | yes |
| finds Julia Roberts + Alec Baldwin in Notting Hill | yes | **no** | yes |
| Just Go with It: pairs Brooklyn Decker correctly | yes | **no** | yes |
| Notting Hill: who ends up with Honey | **no** | - | yes |

Opus matched Fable everywhere and beat it once. Sonnet failed four of six, and
two of those failures silently delete real board rows rather than erroring.

**`weight` and `confidence` are different axes and both are required.** Alec
Baldwin's Jeff King really is Anna Scott's boyfriend, so confidence is `high`,
and he has one scene, so weight is `incidental`. A filter on confidence alone
put that pair on the board as a Julia Roberts row. Board rows come from
`central` and `substantial`; `incidental` is recorded and excluded so the
exclusion stays countable. `tests/test_couples.py` pins this.

### One canonical score per person-year, averaged (2026-09-16)

A person-year may be graded several times by several models. The operator's
rule: keep every grading, and make the canonical score their **mean**.

**Opus may now grade person-years, and it is the default.** Measured on five
tuples: Opus differed from Fable by a mean of 0.18, while Fable differed from
ITSELF by 0.12 on a re-run, against a Fable-to-astra floor of 0.307. So Opus is
inside the noise. Sonnet measured 0.35 and failed 1 of 5 on schema, so it is
named in `NOISY_MODELS` -- its four gradings still count, and
`--exclude-model claude-sonnet-5` reverses that without re-grading anything.

This matters because **Fable has its own weekly allowance**, separate from the
general pool and usually the scarcest window on the account. Moving person-year
scoring to Opus takes the whole pipeline off it. Keep Fable as a second family:
the cross-family spread is what made this measurable.

Three things the canonical score carries, deliberately:

- **`n`** -- 18 person-years rest on one grading, 187 on two, five on four or
  more. Those are not equally precise and the artifact says so.
- **`spread`** -- max minus min. Mean 0.226 over the 192 multiply-graded
  tuples, max **2.0** (Kathy Bates 2002, fable against astra).
- **`models`** -- so a later decision to drop one costs no calls.

**An unjudged grading is a refusal, not a zero.** astra refused 23 of its 215,
and averaging those in as 0.0 would drag every one to the bottom of the board.
`canonical_scores` skips them; `tests/test_canonical.py` pins it.

**Gradings written before 2026-09-16 carry no model and no timestamp.** 430 of
444. The model is recovered from the family through `MODEL_FOR_FAMILY`, which
is what those runs pinned. The time is NOT recoverable and is NOT back-filled
from file mtime, which records when a file was touched rather than when a
judgment was made. `tests/test_grading_provenance.py` forbids that back-fill.

**The real-life bar is 2, not 3.** An actor makes far more films than they have
relationships, so the same number is a much harsher test there.
`--min-pairings-real-life`.

**Real life does NOT come from `pairing_scores.json`.** That graph's on-screen
half is raw co-starring from `fetch_onscreen_candidates.py` with no romance
check, and merging it whole put Idris Elba on the board as Scarlett Johansson's
partner in Avengers: Age of Ultron and The Jungle Book. 21 of its 75 on-screen
pairs were never checked at all, including a Banksy documentary and a film
dated 2028. Real life now comes from `build_reallife_pairings.py`, which reads
relationship RECORDS.

### Two rules decide who appears (2026-09-16)

1. **Only films from 1980 on are graded.** `--min-year`, defaulted in BOTH
   `build_imdb_pairings.py` and `rebuild_boards.py`. It lives in the pairings
   stage as well as the board so `score_person_periods.py` never pays for a
   person-year the board cannot show. The per-person trajectory obeys it too,
   or a 1978 score stretches the shared x-axis to 1978.
2. **A person needs 3 scored romances to be ranked.** `--min-pairings`.
   **DISPLAY ONLY**: the pairings stay in the data and a person below the bar
   is still a partner on someone else's row. Each board prints how many people
   the bar hides, because a threshold nobody can see is the same problem as a
   silent exclusion.

**Nobody is removed by name.** An earlier version deleted Audrey Hepburn,
Sophia Loren, Paul Newman and Robert Redford from the roster because their
careers are mostly pre-1980. It worked, and it was roster selection for
presentation, which `docs/SOURCE-HUNT.md` warns against: the roster was chosen
on prominence BEFORE anything was scored, deliberately. The two rules reach a
similar place without anyone choosing who. Measured under them: Hepburn has 1
post-1980 romance and is not ranked, Loren 3, Newman 9, Redford 15.

**Two floors were tried and rejected, and why is recorded so they are not
tried again.** 1990 cut 101 pairings and the loss was concentrated: Michelle
Pfeiffer lost 33% of her romances, Richard Gere 27%, Tom Cruise 25%, while 17
of 26 seeds lost nothing. It took Dangerous Liaisons, Scarface, Top Gun and An
Officer and a Gentleman. 1985 cut 39 with the same shape. **1980 cuts 8**, all
three of Richard Gere's earliest films, and nobody else loses anything.

The floor is not what fixed the axis. Removing the pre-1980 outliers did, and
the rules now do it: the corpus runs continuously from 1980 with no gap.

## Plan v4: the pairing boards

`docs/PLAN-v4.md`. The scoring layer changes and everything downstream carries
over. One call judges one pairing and returns the GAP directly, because two
absolute scores made in separate calls drift against each other and labelling
that drift is the only reason `modules/analytics/comparability.py` exists.

| Command | What it does | Spends quota |
|---|---|---|
| `scripts/select_m1_slice.py` | picks the M1 focal actors and their pairings by a fixed rule, before any judging | no |
| `scripts/rebuild_boards.py` | cache → gaps → within-sex normalization → four boards → one HTML file. Free; re-running after grading more person-years costs nothing | no |
| `scripts/score_person_periods.py` | judges each (person, year) once, film-blind and evidence-anchored, and CACHES it; `--backlog-only` spends nothing | **yes** |
| ~~`scripts/score_pairings.py`~~ | judged each pairing and returned the gap. **Superseded 2026-09-15**: it told the judge which film it was scoring, so the same person in the same year came back 9.0 for one film and 9.5 for another. See docs/PLAN-v4.md §4a | **yes** |
| `scripts/build_boards.py` | renders the four leaderboards from judged gaps, and refuses to call a ranking established without measured spread. Also renders the within-sex normalized SECOND view below them and writes its numbers to `data/roster100/run/normalized_view.json` | no |

The v4 rubrics are `rubrics/person/` (current) and `rubrics/pairing/` (superseded). `modules/pairing/judge.py` parses a verdict
and REFUSES one whose two absolute scores contradict its own gap.

Two constraints from plan v3 are kept and asserted by
`tests/test_no_age_formula.py`: no age arithmetic anywhere in the scoring path,
and no image fetched or sent to any judge.

### The normalized second view

`modules/pairing/normalize.py` restates every absolute score as its distance
from the mean of its own sex, in units of that sex's spread. It is a SECOND
view rendered below the raw boards and it never replaces them, which is how
plan v3 shipped `same_shape_view` beside the main board. Invoke it through
`scripts/build_boards.py`; there is no separate command.

Four things to know before reading it:

- The population is **(judge family, sex, person, period) tuples**, because a
  person is judged at the time of each pairing. A person judged in three
  periods is three observations; a person judged five times inside one period
  is one, carrying their mean.
- **It reads the RAW verdicts, not the scored artifact.** `pairing_scores.json`
  keeps the gap and drops `f_absolute` and `m_absolute`, and a within-sex
  normalization cannot be built without the absolutes.
- **Parse raw verdicts with `parse_pairing_verdict`, never `json.loads`.** One
  judge family fences its JSON in a code block, and a bare `json.loads` drops
  every one of those files, raises nothing, and reports a smaller corpus.
  `repeat_spread()` in `scripts/build_boards.py` still has that defect; it has
  no caller, which is the only reason it has not cost anything.
- **The mean normalized gap is zero by construction**, so the normalized view
  can never be asked whether the judges score women higher than men. The raw
  board carries that measured offset and stays the default. The zero holds over
  the whole observation population and NOT inside any subset; the rendered
  document has the per-domain table showing where it does not.

The quota-spending scripts take `CELEB_JUDGES` (default `fable,astra`) and
`CELEB_MAX_CALLS`, or equivalent flags. Run the whole chain in this order after
changing any source: `fetch_records` → `build_episodes` → `build_partner_universe` →
`fetch_observations` → `merge_prose_mentions` (once per
mentions file) → `verify_identities` → `verify_observations` → `absence_audit` → `corroborate_relationships` → `score_evidenced` → `joint_coverage` → `evidence_density` →
`alignment_gap` → `shape_confound` → `grounding_audit` → `offset_diagnostic` →
`source_requirement` → `scaling_report` → `reachable_products` → `relationship_review` → `conclusion_robustness` → `verify_trap` →
`cross_run_stability` →
`write_m0_report` → `audit_doc_numbers`.

Run the free stages and the reports with the chain script, which encodes the
dependency order so a stage cannot run before the one it needs:

```sh
bash scripts/run_chain.sh free      # no model calls
bash scripts/run_chain.sh reports   # regenerate every report from artifacts
bash scripts/run_chain.sh all
```

The `reports` pass runs a stale-corpus preflight first and EXITS rather than
continuing. `run()` records a stage failure and carries on by design, so without
the preflight the chain refuses at every scored stage and still reaches the
report renderer at the end. That happened during the 2026-09-14 contract bump
and rewrote `docs/M0-REPORT.md` from a corpus no rubric on disk produced.

The quota-spending stages are deliberately NOT in it. They need an account and a
cap chosen by a human who has looked at `quotapick status`.

## Long runs: caffeinate, not just nohup

**`nohup` survives a closed terminal. It does not survive the machine sleeping.**

Measured 2026-09-15: two scoring runs were launched with `nohup ... &`, confirmed
alive, and five hours later had made no progress. The processes were orphaned to
PID 1 with nothing supervising them, so nothing noticed and nothing restarted
them. A 131-pairing run that should have taken 40 minutes produced nothing for
most of a day.

Start any run longer than a few minutes under both:

```sh
nohup caffeinate -i -m -s .venv/bin/python -u scripts/score_pairings.py ... &
```

`-i` blocks idle sleep, `-m` disk sleep, `-s` system sleep. To protect a run
that is ALREADY going, attach by pid — `caffeinate` holds until it exits:

```sh
for pid in $(pgrep -f "[s]core_pairings.py"); do
  nohup caffeinate -i -m -s -w "$pid" > /dev/null 2>&1 &
done
```

Two more habits that go with it, both learned the same day:

- **Arm a watcher.** An orphaned run that dies is indistinguishable from one
  that is working, until someone asks. Check that verdicts are still landing,
  not just that the process exists.
- **Write per-unit output.** `score_pairings.py` writes one raw file per pairing
  as it lands, so a run killed at 60% keeps 60% of its work. A run that only
  writes at the end loses everything the machine sleeps through.

Quota rules for the ones that spend:

- **Subscription only. API billing is asserted off at start-up** — a visible
  `ANTHROPIC_API_KEY` aborts the run.
- Judges are `fable` (Claude, via `claude -p`) and `astra` (via `codex exec`).
  **Gemini via `agy` does not work headlessly**: it reaches for a shell tool,
  headless mode auto-denies it, and the turn returns empty. Do not fix that by
  granting the permission — a judge with filesystem access is not isolated from
  the corpus it is being kept away from.
- Check `quotapick status` first, and pass what it tells you. **There is no
  default CLAUDE account**: every paid script takes `--account <config dir>` or
  reads `CELEB_ACCOUNT`, and refuses to start without one, naming the config
  dirs it can see.
- **The CODEX account is picked for you, from live usage (2026-09-15).** Leave
  `--astra-account` and `CELEB_CODEX_HOME` unset and `resolve_codex_home` asks
  `quotapick` which Codex home has headroom, then announces its choice on
  stderr:

  ```
  [accounts] quotapick chose Codex account codex_b (CODEX_HOME=/Users/tonygwu/.codex-b)
  ```

  This is not the hardcoded default that AGENTS.md spends a section warning
  about. That default was a value frozen into a file; this is a question asked
  at the moment of the run. Both overrides still outrank it, and a named home
  is never second-guessed.

  Four things worth knowing before you rely on it:

  - **It still refuses.** No router on PATH, a router that names no account, a
    router whose contract version it has not read, or a router answer it cannot
    parse, all raise `RouterUnavailable`, which IS an `AccountNotChosen`.
    Nothing falls back to `~/.codex`.
  - **A returned account is not a spendable one.** `quotapick pick` exits 0 and
    names an account even when nothing is spendable, marking it `fits: false`.
    The resolver refuses on that flag. A real capture is kept at
    `tests/fixtures/quotapick_pick_all_exhausted.json`.

    **`fits: false` does not mean the call would fail**, and the first version
    of this note said it did. A smoke test spent a real `codex exec` against an
    account reading `fits: false` and got a normal verdict back. The account is
    held back on purpose: the operator reserves weekly quota on `~/.codex` for
    the Codex DESKTOP app, which cannot use any other home. At the time of
    measurement the vendor had 13% of the week left, the reserve held 51%, and
    0% was spendable by automation. `quotapick status --explain` prints the
    split, and `pick --explain` does not, because `pick` only ever emits JSON:

    ```
    reserve codex: 13% left - 51% held for manual use (0.09/day x 5.6d) = 0% spendable
    ```

    So the guard honours a policy rather than dodging an outage, which is the
    stronger reason to keep it. A failed run is loud; quietly eating the
    operator's interactive quota is not.
  - **`used_fraction` is the vendor reading, and `held_fraction` is the hold**
    (quotapick v0.1.5 and later). They net to the whole window. Measured
    2026-09-15 on the codex 7d window: `used_fraction 0.89`, `held_fraction
    0.11`, summing to 1.0, with `manual_reserve.spendable 0.0`.

    **This reverses the advice that stood between v0.1.4 and v0.1.5**, which
    this file carried and which said to ignore `used_fraction` and read the
    `manual_reserve` block instead. Under v0.1.4 the reserve OVERWROTE the
    vendor value, so the same window read `used_fraction: 1.0` while the vendor
    still had 13% left, and a held account was indistinguishable from an
    exhausted one. `manual_reserve` still works and is still the place the
    reserve arithmetic is spelled out. It is no longer a workaround.

    Nothing in this repo reads either field, so nothing needed changing. The
    note is here because the next person to debug a 0%-spendable Codex account
    will read `used_fraction` and needs to know which version's semantics they
    are looking at.

    There is **no `remaining_fraction` field**, in `status --json` or in
    `pick --json`, although the v0.1.5 announcement named one. The netted value
    is `account.min_remaining`, and `manual_reserve.spendable` beside it.
  - **The candidate list is derived, never written down.** The ids come from
    `quotapick status --json` filtered on `provider == "codex"`. Do not replace
    that with `--only codex,codex_b`; a third Codex account would be invisible
    to the picker while `quotapick status` listed it.
  - **It costs about 5 seconds**, once per run, for two live calls. Name the
    home yourself to skip them.

  The Codex judge now records `config_dir` in its telemetry, the way the Claude
  judge always has, so the artifact says which account paid for a verdict.
  Before this, the Codex account behind a verdict was unrecoverable after the
  run.
- **To use the account bare `claude` uses, pass `--account default`.** It is a
  keyword, not a path, and this is not a style choice. That account's config
  file is `~/.claude.json`, which sits OUTSIDE `~/.claude/`, so pointing
  `CLAUDE_CONFIG_DIR` at `~/.claude` makes Claude Code look inside, find
  nothing, and scaffold a brand-new EMPTY account — after which the run fails as
  `auth_or_quota` and reads like a broken judge. The refusal message used to
  list `~/.claude` among the valid choices; it no longer does, and
  `tests/test_default_account.py` asserts that. Found 2026-09-14, when that
  account held 100% of its fable window and no script could ask for it. The six scripts used to default to `~/.claude-e`, which on
  2026-09-14 was at 0% on its 5-hour window while `~/.claude-c` had 66% fable
  headroom — a default that is wrong is worse than no default, because the run
  fails as `auth_or_quota` and reads like a broken judge. A `--dry-run` needs
  no account; resolution happens where the judge is built.
- Every runner takes `--max-calls` style caps and **halts at the cap**, reporting
  the halt and the work it did not reach.
- **The paid stages have an order too**, and it is not in `run_chain.sh` because
  they are not in `run_chain.sh`: `score_evidenced.py` →
  `measure_rater_noise.py` → `run_stress.py`. `run_stress.py` reads the
  rater-noise floor from a SEPARATE scoring run and refuses a stale one, so
  running it first wastes the calls. It checks before building a judge, so the
  refusal costs nothing, but only when the floor is already stale.

## Running the roster-scale chain

The pilot chain defaults to `data/pilot/`. The 100-name roster uses the same
scripts with different paths:

```sh
.venv/bin/python scripts/fetch_records.py       --cohort docs/roster-100.json --out data/roster100/records
.venv/bin/python scripts/build_episodes.py      --records data/roster100/records/relationship_candidates.json \
                                                --cohort docs/roster-100.json --out data/roster100/records/episodes.json
.venv/bin/python scripts/build_partner_universe.py --episodes data/roster100/records/episodes.json \
                                                --cohort docs/roster-100.json --out data/roster100/records/partner_universe.json
.venv/bin/python scripts/fetch_observations.py  --cohort docs/roster-100.json \
                                                --partners data/roster100/records/partner_universe.json \
                                                --out data/roster100/observations
```

Everything above is free. `scripts/score_roster_joint.py` then scores only the
dossiers the jointly covered pairings need, which is about nine rather than the
127 a full pass would cost.

## IMDb is in scope as of 2026-09-15

Plan v3 §3 listed IMDb as `Out`, on its terms clause about data "repurposed to
create any kind of online/offline database of movie information". **The
operator reversed that on 2026-09-15, for full use including the published
page.** The reasoning, and what did not change, are in `docs/SOURCE-HUNT.md`.

Two things an agent needs from that decision:

- **Never commit a `.tsv.gz` dump.** They are ~2 GB and not ours to
  redistribute. Derived artifacts commit normally.
- **The restricted publishers are unaffected.** People Inc./people.com, Ziff
  Davis/askmen.com, Maxim and Condé Nast/Glamour stay out by every route,
  archives included. IMDb was never on that list.

One correction worth keeping, because it cost a turn: an agent asserted this
restriction was written in `AGENTS.md` and in a `docs/PLAN-v3.md`. Neither was
true. Plan v3 lives outside the repo, in the plan-mode file, so a subagent
grepping the checkout found nothing and the agent then over-corrected to "the
restriction never existed". **A constraint you remember but cannot cite is
neither confirmed nor refuted by its absence from the repo.** Say which it is.

## Two ways Wikidata answers wrongly and looks fine

Both were found on 2026-09-15 and both had already shipped a number. Use
`batched_query` in `modules/records/wikidata.py` for any new SPARQL fetch; it
carries both guards. Do not write a bare `_query` loop.

**A LIMIT that is reached is data loss, not a limit.** Wikidata returns the
first N rows and says nothing about the rest. `fetch_onscreen_candidates.py`
ran one query with `LIMIT 400` against a 100-name roster that produces 2570
rows, and published 136 co-starring pairs across 74 films. The real figure is
808 pairs across 502 films, and 25 of the 100 roster members had NO pair at
all. Gigli, Armageddon and Ghosted each have two roster members in the cast and
all three were in the discarded tail, so the product looked like it was missing
famous couples that Wikidata knows about perfectly well. Set the cap far above
any honest answer, batch the query so that is possible, and REFUSE a batch that
comes back at its cap.

Two smaller things that made it worse and are worth copying:

- **`SELECT DISTINCT`.** A film whose `P31` values each reach `Q11424` by a
  different path yields one solution per path, and `P577` repeats per country
  on top. Those duplicates counted against the cap: 2570 rows for the same 808
  pairs, 2473 with DISTINCT.
- **Read the cap off the artifact.** `rows_fetched` and `row_limit_per_batch`
  are stamped in `onscreen_candidates.json` now. The old artifact recorded
  neither, so nothing on disk could have revealed the truncation.

**A query timeout arrives as HTTP 200.** The service does not answer 503 when
it gives up at about sixty seconds. It streams result rows, stops mid-JSON, and
appends its own log to the same body:

```
"valSPARQL-QUERY: queryStr=
SELECT DISTINCT ?seed ...
java.util.concurrent.TimeoutException
```

That came back as a 595 KB body with a Java stack trace glued to the end, and
the status line, the byte count and a `urlopen` that raised nothing all said
success. `json.loads` rejecting it is the only reason it surfaced.
`WikidataQueryTimeout` names it, and `batched_query` HALVES the chunk rather
than asking the same too-big question again, because a retry of an identical
query costs a minute and cannot succeed. A real fixture of that body is kept at
`tests/fixtures/wdqs_timeout_body.txt`.

## What has been measured

`docs/M0-REPORT.md`, generated from `data/pilot/`. Read it before proposing
anything. The findings that should shape any next step:

1. **The rubric is not the problem.** On synthetic dossiers it spans 63 to 92
   and produces ten distinct values.
2. **Evidence density was the bottleneck.** Every real dossier once carried
   exactly one observation, so the corpus produced two distinct values. Adding
   prose mentions took it to twelve.
3. **Evidence SHAPE explains 40% of the estimate** (omega-squared,
   unbiased; the biased eta-squared that earlier documents quoted reads 39%). An editorial award pins
   near 92 by construction; ranked placements spread lower. 3 of 4
   jointly covered pairings pit one against the other, so their gaps are
   substantially about publication format.
4. **The one comparable gap is 0.5**, against a least significant difference of
   about 1.03 points for rank-shaped estimates, measured on four dossiers
   repeated four times each BY BOTH JUDGE FAMILIES.

   Two things about those numbers. The gap was 0.0 while one family scored the
   corpus and returned integers; it is 0.5 because the reducer now averages two
   judges and one of them said 93 where the other said 92. That is not a signal.
   And the floor is NOT pinned: the same judge on the same four dossiers gave a
   mean within-judge sd of 0.926 on one run and 0.269 on the next, so the LSD
   measured 2.56 and then 1.03. Quote the interval, not the point. There is no leaderboard here yet, and adding more
   award-shaped sources will not create one. The LSD is quoted PER SHAPE:
   pooling the award dossiers' zero measured variance with the ranked ones'
   halved it to 1.2, which several documents published. Every one of the four
   ranked dossiers moved between repeats; both award dossiers returned the
   same number eight times out of eight.

5. **S2 does NOT confirm the shape effect — it tests anchor-following.**
   Stress case S2 renders one judgment as an award, a rank and prose, and the
   estimates spread 4 points. Do not quote that as controlled evidence about
   format: the prose arm is the VERBATIM text of the rubric's calibration
   Example 3, anchored at 88, and the award arm has the shape of Example 1,
   anchored at 92. The judge returned 92, 92, 88 — the anchored values. The
   case is worth running, because a judge that stopped following its anchors
   would be a real finding, but finding 3 rests on the observational corpus
   alone. Redesign is filed in `docs/BACKLOG.md`.
6. **Every real estimate came from ONE judge family.** The plan decided J = 2
   so a one-family idiosyncrasy could be told from a property of the rubric.
   Codex reached 0% of its 7-day window mid-run, so all 40 person-periods were
   scored by `fable` alone and the `astra` column in the report is empty for
   that reason, not because the judges agreed. Both families DID run on the
   stress corpus. Re-scoring with codex when its quota returns is the single
   cheapest thing that would strengthen every conclusion above.

Do not treat a bigger confounded number as progress over a smaller comparable
one.

## Changing a rubric or schema

Do not edit anything under `rubrics/` without reading
[`docs/CONTRACT-BUMP.md`](docs/CONTRACT-BUMP.md). The grading contract is
`sha256(len(rubric) + rubric + len(schema) + schema)`, so a one-line typo fix
gives a new `contract_id`, and estimates under different ids must not be pooled.
A trivial edit therefore costs a full re-score.

**The three filed corrections landed together on 2026-09-14**, under
`standing-rubric-2.1` / `mentions-1.1`, with the operator approving the re-score
they cost. That bump also changed the hashing scheme to `v2-length-prefixed`,
which moved every contract id including romance's, whose bytes never changed.
`contract_id_scheme` is stamped beside the id in every artifact so a scheme
change cannot be misread as a rubric change.

The cost is bigger than "re-score the rubric you edited". A scheme change
invalidates EVERY rubric's artifacts at once: prose mentions for cohort and
partners, romance classification, evidenced scores, roster joint scores, rater
noise and the stress corpus. `docs/CONTRACT-BUMP.md` has the order and the
lessons from the first real bump.

## New shared tooling

A new tool is not done until it is committed **and** registered in this file
with one line saying how other agents invoke it. An unregistered tool exists
for nobody.

---
> Source: [tonygwu/celeb-couple](https://github.com/tonygwu/celeb-couple) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
