## cfps-panel-data-cleaning-pipeline

> Orientation for any coding agent (or human) working on this repository. Read this

# Agent guide: CFPS panel cleaning pipeline

Orientation for any coding agent (or human) working on this repository. Read this
before editing a `.do` file, then open the task guide under `docs/agent-guide/`
that matches what you are doing.

This repo harmonizes CFPS (China Family Panel Studies / 中国家庭追踪调查)
individual-level data from six waves — 2010, 2012, 2014, 2016, 2018, 2020 — into
one long-format panel, `cfps_ind.dta`, keyed by `pid` and `wave`.

The code is hand-written Stata with strong internal conventions and no test
suite. Nothing validates that an edit is correct — a variable that violates the
naming convention is dropped silently rather than flagged, so "it ran without
error" proves very little here.

## Layout and execution order

| File | Role |
| --- | --- |
| `00_run.do` (project root) | Entrypoint: checks working directory, checks dependencies, runs everything in order |
| `scripts/01_config.example.do` | Template → copy to `scripts/config.do` (gitignored) |
| `scripts/config.do` | Local paths: `$data_raw`, `$data_gen` |
| `scripts/02_paths_globals.do` | **Every** raw file path (`$f_raw_*`) and output path (`$f_out_*`) |
| `scripts/11_CFPS_Panel_Ind10.do` | Wave 2010 |
| `scripts/12_CFPS_Panel_Ind11.do` | Wave 2011 — **not run** by `00_run.do` (2011 is not a core wave) |
| `scripts/13`…`17_CFPS_Panel_Ind{12,14,16,18,20}.do` | Waves 2012–2020 |
| `scripts/20_CFPS_Panel_Individual_Crosswave.do` | Parent/spouse linkage, cross-wave backfill, append to panel, cognitive interpolation |
| `scripts/30_CFPS_Panel_Individual_Labels.do` | Chinese + English variable and value labels, final save |
| `ado/local/` | `csco2isco_cfps`, `csco2isei_cfps` (shipped) |
| `ado/third_party/` | gitignored; drop your own `iskoisei.ado` etc. here |
| `data_dictionary/01_individual.csv` | Codebook, 6 columns, sectioned by `Part` |

**Wave scripts must run in order.** Every script from `13_` onward merges the
generated `cfps{YY}_ind.dta` files of *earlier* waves, and `17_` (2020) merges all
five. You cannot run a later wave standalone against an empty `$data_gen`.

## The three conventions everything depends on

**1. The `a_` staging prefix.** Inside a wave script every harmonized variable is
built as `a_<name>`. The block at the end of each questionnaire section does:

```stata
keep a_*
rename a_* *_10
rename pid_10 pid
```

So a variable you create without the `a_` prefix is dropped by `keep a_*` without
any error. This is by far the most common way an edit to this repo fails — the
script runs clean and the variable simply isn't in the output.

**2. The `_YY` wave suffix.** After staging, variables carry a two-digit wave
suffix (`edu_10`, `edu_20`). `20_` strips it before appending the waves together,
so the final panel has bare names plus a `wave` column.

**3. Child questionnaire first, adult appended onto it.** Each wave script
processes the child (or childproxy) file, saves `cfps{YY}_ind.dta`, then processes
the adult (or person) file and ends with `append using` that same file. A variable
that only exists in the adult questionnaire is therefore missing for child
records. That is expected, not a bug.

One more thing that trips up automated edits: **the `Part` section numbering is
not consistent across waves.** In `11_` (2010), `Part8` is Health and `Part10` is
Political Experience; in `13_`–`17_`, `Part8` is Political Experience and there is
no Health section. Place new code in the section that actually exists in the file
you are editing.

## Running it

```stata
cd "/path/to/CFPS-Panel-Data-Cleaning-Pipeline"
do "00_run.do"
```

Four things must be in place first: the raw CFPS data arranged by wave (the repo
ships none — it is licensed and cannot be redistributed), `scripts/config.do`
copied from the example and pointed at your paths, the `isko` and `iscogen`
packages installed from SSC, and the stray `B` deleted from the SSC copy of
`iskoisei.ado`.

`00_run.do` stops at the first failure with a distinct exit code (601 config or
working directory, 604 missing ado, 605 broken `iskoisei`, 610–617 individual
scripts), so the exit code alone identifies the stage.

## Verifying a change

There is no test suite, so verification is manual and you should always do it.
To re-run a single stage without the rest of the pipeline:

```stata
cd "/path/to/CFPS-Panel-Data-Cleaning-Pipeline"
do "scripts/config.do"
do "scripts/02_paths_globals.do"
adopath ++ "ado/local"
adopath ++ "ado/third_party"
do "scripts/17_CFPS_Panel_Ind20.do"     // requires waves 10-18 already generated
```

Then confirm the variable survived, with its wave suffix:

```stata
use "$data_gen/cfps20_ind.dta", clear
describe newvar_20
count if newvar_20 < 0        // should be 0: raw CFPS negatives must become .
```

After a full run, this is the check that catches a wave you missed:

```stata
use "$data_gen/cfps_ind.dta", clear
bysort wave: count if !missing(newvar)   // a wave with 0 is a red flag
```

Raw CFPS encodes missingness as negative values (`-1` through `-10`). Every
harmonized variable must convert these to Stata missing, via `if rawvar > 0` (or
`>= 0` when zero is meaningful), `mvdecode …, mv(-10/-1)`, or
`recode … (-10/-1=.)`. Follow whichever idiom the surrounding lines use.

## Task guides

Read the matching file before making changes — each lists the full set of places
that must be edited in sync, which is the part that is easy to get wrong.

| Task | Guide |
| --- | --- |
| Setting up a fresh clone, or diagnosing a failed run | [`docs/agent-guide/setup-and-reproduce.md`](docs/agent-guide/setup-and-reproduce.md) |
| Adding or modifying a harmonized variable | [`docs/agent-guide/add-variable.md`](docs/agent-guide/add-variable.md) |
| Adding a new survey wave (e.g. 2022) | [`docs/agent-guide/add-wave.md`](docs/agent-guide/add-wave.md) |
| CFPS republished the data; files or variables renamed | [`docs/agent-guide/data-version-update.md`](docs/agent-guide/data-version-update.md) |

## Things to raise rather than decide alone

Some changes are research-design decisions, not maintenance. Surface these
instead of picking an answer:

- Switching occupation coding between ISCO-88 and ISCO-08. The waves are not
  uniform: 2010/2012 read a precoded `qg307isco`, while 2014/2020 call
  `csco2isco_cfps … , csco(qg303code)`. `iskoisei` expects ISCO-88 and will
  produce wrong ISEI scores from ISCO-08 codes without complaining.
- Changing the cross-wave backfill rules in `20_`, which overwrite earlier waves'
  time-invariant variables using later waves.
- Changing the cognitive-score interpolation in `20_`, which fills missing waves
  with the mean of adjacent waves.

These are documented in README under "Core Cleaning Logic" and materially affect
downstream estimates.

## Keeping this guide accurate

This document and the task guides cite specific files, conventions and line
regions, so they go stale quietly when the code moves. If you change something
described here, update the corresponding guide as part of the same change —
`CONTRIBUTING.md` has a table mapping code areas to the file that documents them.

Line numbers cited in the task guides are approximate and given as "near line N"
on purpose. Locate the code by the surrounding pattern (the `foreach var in
birthy_ …` list, the `keep pid edu_10 …` lines) rather than by the number, and if
you find the number has drifted far, correct it while you are there.

---
> Source: [tianbing1230/CFPS-Panel-Data-Cleaning-Pipeline](https://github.com/tianbing1230/CFPS-Panel-Data-Cleaning-Pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
