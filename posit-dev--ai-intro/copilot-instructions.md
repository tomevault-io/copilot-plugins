## ai-intro

> Materials for the "Introduction to AI" workshop: a Quarto website (`website/`) and

# Working in this repository

Materials for the "Introduction to AI" workshop: a Quarto website (`website/`) and
a set of Shiny apps (`apps/`) that attendees run or visit during the course. The
apps are teaching artifacts first — if a change makes an app tidier but blunts the
point it demonstrates, it's the wrong change.

- `apps/` — one directory per app. Six R apps plus `clearbot` (Python).
- `R/`, `python/` — canonical shared helpers, copied into each app.
- `website/` — the Quarto site, including slides.
- `tools/` — `setup.sh`, the guided one-time setup behind `make setup`, and
  `deploy.sh`, which deploys to Posit Connect (see `README_deploying.md`).

## Conventions

**Provider and model come from the environment, never hard-coded.** Apps read
`AI_INTRO_PROVIDER` and `AI_INTRO_MODEL`, each overridable per app as
`AI_INTRO_<APP>_PROVIDER` / `AI_INTRO_<APP>_MODEL`. Apps needing more than one
model ask for a size (`AI_INTRO_MODEL_SMALL` / `_MEDIUM` / `_LARGE`, also with the
per-app prefix). Sizes are additive — the full order is app size var, global size
var, app model var, `AI_INTRO_MODEL`, our built-in per-provider default, then the
provider's own default. Setting provider and model alone is always enough.

Call sites spell out the variables they consult, in order, rather than hiding the
chain in a helper. Someone reading `app.R` should be able to see what it reads.

**Edit the canonical helpers, not the copies.** `R/chat_env.R` and
`python/chat_env.py` are copied into every app directory by `make sync-helpers`.
Change the copy in `R/` or `python/`, run `make sync-helpers`, and confirm with
`make check-helpers` before committing.

**Get model IDs from the provider, not from memory or documentation.** Use
`ellmer::models_anthropic()`, `models_openai()`, `models_google_gemini()`,
`models_aws_bedrock()`, `models_posit()`. A wrong ID stops an app from starting.
Note that a listing enumerates documented models, which is not the same set as
accepted IDs — a bare alias can work without appearing there.

Some model IDs look wrong and aren't. The Bedrock entries are deliberately
asymmetric, the Gemini entries are deliberately floating aliases, and
`apps/counting-code` pins older IDs because they match its committed
`model-responses/`. Each is commented where it lives; read the comment before
"fixing" it.

**Two things worth knowing about providers.** `posit` works for local development
only — it authenticates by OAuth device flow, so there's no key to set on a
deployed app. `clearbot` reads the usual variables, but they select the entry its
Model dropdown starts on rather than fixing the model: it asks each provider it
has a key for which models it offers, and lets you switch between them mid-demo.

**Docs are split by audience.** `README.md` is for people who land on GitHub:
running locally, and self-hosting your own run of the workshop.
`website/setup.qmd` is for attendees of an already-deployed instance, who install
nothing and need no API key of their own. Don't move developer detail into the
attendee page.

**Don't run the apps to check your work.** They need live API keys and network,
and real calls cost money. Verify by parsing/compiling, by `make check-helpers`,
and by exercising helpers directly. Say plainly what you could not verify.

Dependencies are declared in `DESCRIPTION` (R, package `ai.intro`) and
`pyproject.toml` (Python, project `ai-intro`); `apps/clearbot` has its own
`pyproject.toml`. Per-app `renv` lockfiles are intentionally untracked.

Commits are small, logically grouped, and use conventional-commit subjects.
Comments explain why, in plain prose, addressed to a workshop attendee reading
the source — they are part of the teaching material.

---
> Source: [posit-dev/ai-intro](https://github.com/posit-dev/ai-intro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
