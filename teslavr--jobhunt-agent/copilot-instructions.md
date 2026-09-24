## jobhunt-agent

> This project is a local job-search agent. If you are Claude Code (or a similar

# Working on this repo with a coding agent

This project is a local job-search agent. If you are Claude Code (or a similar
coding agent) opened in this directory, this file tells you how to help.

## Ground rules — these are not negotiable

1. **You do not submit applications; `submit.py` does, when the user runs it.** Never
   click Apply in a browser yourself, and never send a message on the user's behalf.
   The user drives sending — your job is the drafts and the tooling.
   If you change `submit.py` or `jobhunt/forms.py`, the four invariants below are the
   point of the module and must survive any refactor:
   - never type a value that isn't from `profile.yaml` or `apply.answers` in config;
   - never report `submitted` without positive evidence (see `forms.verify_submitted`);
   - never interact with a CAPTCHA, and never try to look human to a bot check;
   - never send twice to the same URL, and respect `apply.daily_cap`.
   `tests/test_submit.py` locks all four in. Run it after touching either file.
2. **Never scrape LinkedIn** or any site that forbids it. Sources are public ATS APIs
   only. The LinkedIn layer generates links for the human to click in their own browser.
3. **Never invent facts about the user.** Resumes and cover letters may only contain
   things present in `profile.yaml`. If a role wants something the profile doesn't
   support, leave it out and flag the gap.
4. **`profile.yaml`, `config.yaml` and `companies.yaml` are gitignored** — they hold the user's contact
   details, salary, and resume. Don't commit them, don't paste their contents into
   anything public, and don't include them in example output.
5. Honour `confidential:` in `profile.yaml`: those things are never named in generated
   documents, only referred to as the profile says.

## If the user asks you to set them up

You can do the whole onboarding conversationally — it's usually better than the
scripted `python3 onboard.py`, because you can read their resume directly and ask
follow-ups. Do this:

1. Ask for their resume (or find it if they name a file) and read it.
2. Interview them for the things a resume doesn't say:
   - where they're based, work authorization, whether they'd relocate, remote-only?
   - salary floor and target, and in what currency
   - **what roles they're hunting** — titles, seniority, the kind of work
   - **what would make a role wrong for them** — this matters as much as the wants
   - any industries, company types, or regions to exclude
   - anything confidential that must never appear in a document
3. Write `profile.yaml`, `config.yaml` and `companies.yaml`, using the matching
   `*.example.yaml` files as the schema (and `sources.yaml` for the source catalogue). Read those first — they document every
   key, and the comments explain the intent behind each one.
4. The parts that need real thought, not defaults:
   - `tracks` — the title substrings that actually appear in real postings for the
     roles they want. Include the variants ("sales engineer", "solutions engineer",
     "pre-sales"). Too narrow and the digest is empty; too broad and it's noise.
   - `screen` — prose for the LLM screen. Base `reject_when` on the specific things
     this person doesn't want to do, in their words.
   - `fit.strengths` / `fit.soft_gaps` — what lines up and what they'll need to
     address, drawn from their actual background.
   - `location.eligible` / `ineligible` — substrings matched against a posting's
     location field, so think about how postings are actually worded.
5. Then run `python3 run.py --doctor`, fix any dead company slugs, and do a first
   `python3 run.py` together. Read the funnel line in the digest with them: if a
   stage is dropping everything, that's a config problem, and now is the moment to
   tune it.

Also offer to rebuild `companies.yaml` around the companies they actually care about —
the shipped list is a generic starter.

## If the user asks you to tailor an application

`apply.py --tailored <file.json>` takes a JSON file with `{summary, experience,
cover_letter}` written by you in-session, and uses it instead of calling an API. That
lets the user get a real rewrite without exporting an API key. Read the job from
`digests/jobs_<date>.json`, write the JSON, then run apply.py with it.

Keep the tone understated. No hype adjectives, no invented metrics, no exclamation
marks. Plain sentences about real work read better than enthusiasm.

If the user is hitting "handed to you" a lot, the fix is usually to help them write
`apply.answers` in `config.yaml` — notice period, salary expectation, visa status,
"why us". Offer to draft those from their profile, in their words, for them to approve.

## Layout

| Path | What it is |
|---|---|
| `run.py` | The pipeline: fetch → filter/score → fit → digest |
| `apply.py` | Builds one application package (resume + cover + checklist) |
| `web.py` | Local click-through UI over the same CLIs, no new deps |
| `onboard.py` | The scripted interview that writes profile/config/companies |
| `linkedin_searches.py` | Generates Boolean feed-search links to open manually |
| `jobhunt/sources.py` | Fetchers. Add new boards here — one function each |
| `jobhunt/score.py` | Filtering and scoring, all config-driven |
| `jobhunt/fit.py` | Two-way fit: "do I pass?" and "does it suit me?" |
| `jobhunt/llm.py` | Optional LLM: tailoring, role screen, onboarding extraction |
| `jobhunt/render*.py` | Digest markdown, and DOCX/PDF output |
| `submit.py` | Fills and (with `--send`) submits a form. Optional Playwright extra |
| `jobhunt/forms.py` | Field detection, value planning, submit verification |
| `tests/` | Fixture-based tests. No network, no real employer touched |

Everything user-specific lives in YAML. If you find yourself hard-coding a person's
city, employer, or salary into a `.py` file, that's a bug — move it to config.

---
> Source: [teslavr/jobhunt-agent](https://github.com/teslavr/jobhunt-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
