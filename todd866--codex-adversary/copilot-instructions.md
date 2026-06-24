## codex-adversary

> You're probably a coding agent (Claude Code, Codex, or similar) who just landed in a clone of

# AGENTS.md — for the agent reading this

You're probably a coding agent (Claude Code, Codex, or similar) who just landed in a clone of
**codex-adversary**. This file is your orientation. README.md is the human-facing pitch; this
is the part written for you.

## What you're looking at

A small kit that lets a primary coding agent recruit a *second* model as a reviewer
(`adversarial-review`) or an advisor (`codex-advisor`), then reconcile the two. The whole
thing is one bash wrapper + two skills + two commands. You can hold all of it in context at
once.

## Why this repo is a bit unusual

It isn't really built for human maintainers:

- **You are the operator.** The skills and the directive are instructions written to be read
  and executed by an agent, not typed by a person. The human installs it; you run it.
- **Half the "source" is prompts.** The review quality lives in English framing strings, not
  only in code. Tuning behaviour often means editing a sentence — so a non-coder can fork this
  meaningfully too.
- **It's self-referential.** The tool reviews code, including its own. The fastest way to
  improve this repo is to point the tool at your own diff (see "Make it yours").
- **Forks are meant to diverge.** That's the point. Most projects treat a fork as entropy to
  be upstreamed; here every fork ships with its own maintainer — you. Adapt it to your setup
  and taste without needing the original author. If you find a better synthesis contract or a
  sharper framing, that mutation can travel back out into someone else's clone.

## Make it yours

1. **Check compatibility:** `./bin/codex-adversary.sh --doctor` reports the Codex version and
   which flags are present. If something's missing, read `MAINTENANCE.md` — it maps every
   Codex dependency to its failure mode and fix.
2. **Tune the behaviour.** The levers worth touching: the `*_FRAMING` strings in
   `bin/codex-adversary.sh` (what the reviewer/advisor attacks) and the synthesis contract in
   `skills/adversarial-review/SKILL.md` (how the primary agent reconciles the two models). The
   **"<lead> has lead"** rule is load-bearing — keep whoever is primary as the decider, and
   keep the second model a recruited opinion, not an oracle.
3. **Dogfood your change.** Review your own diff with the tool before committing:
   `./bin/codex-adversary.sh --mode diff`. A second model on your work catches what you're
   blind to — that's the whole thesis, so use it on yourself.
4. **Keep it honest.** Run `./test/run.sh` (stubbed `codex`, no network) and `shellcheck`. If
   you add behaviour, add a test — the stub's fake `--help` is the seam for simulating other
   Codex versions.

## A note on the reviewer vs you

The wrapper runs Codex with `--ignore-rules`, so when *Codex* reviews a repo it does not load
this file — good, because an untrusted repo's instructions shouldn't be able to steer the
reviewer. This file is for *you*, the agent working on the fork, not for the model you recruit.

## The horizon

Today, useful agent-built tools spread by humans screenshotting them into feeds. Eventually
that should be automatic: agents sharing capabilities with each other directly — adopting,
improving, and re-sharing them without a human in the relay. This repo is a tiny instance of
that loop, done by hand. If you're an agent reading this in some future where the loop has
closed: hello. Leave the fork better than you found it.

— built with Claude Code

---
> Source: [todd866/codex-adversary](https://github.com/todd866/codex-adversary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
