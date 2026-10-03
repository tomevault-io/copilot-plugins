## multilogin-cloud-phone-agent

> manages the mobile profile and keeps the launcher available in the background);

# AGENTS.md — operating a Multilogin cloud phone

Portable instructions for **any** AI agent (Claude Code, OpenClaw, Cursor, …) that
can run shell commands and look at a screenshot. This file is the contract; the
`mlxphone/` toolkit is the implementation. Read this, then drive the phone with
the commands below.

## What you are doing

Three steps:

1. **Start** the selected cloud phone, wait for it to boot, and activate ADB — *deterministic, handled for you.*
2. **Act**: connect to the phone and perform the requested task over ADB — *this is your job.*
3. **Always stop** the phone and save results/logs — *deterministic, guaranteed.*

You own step 2. Steps 1 and 3 are code in `mlxphone.session`; do not reimplement
them, and never skip the stop — mobile minutes are billed.

## Setup (once, from a fresh clone)

```bash
make setup                 # creates .venv + installs deps (uses `python -m pip`)
cp .env.example .env       # fill in real values (email/password or MLX_TOKEN, MOBILE_PROFILE_ID)
make doctor                # validate deps, adb, .env, sign-in, profile — spends NO minutes
```

**Run `make doctor` and clear every `[FAIL]` before you start a phone.** It catches
a missing `adb`, leftover `.env` placeholders, and bad credentials up front, so no
mobile minutes are wasted on an environment that isn't ready.

Requirements: a Multilogin account with the API-token feature, a cloud phone, and
remaining mobile minutes; the **Multilogin desktop app installed and running** (it
manages the mobile profile and keeps the launcher available in the background);
Python 3.9+ (3.11+ recommended); and the `adb` binary on PATH. Without `make`:
`./scripts/setup.sh`, then `.venv/bin/python -m mlxphone.session doctor`.

## Lifecycle commands (steps 1 & 3 — deterministic)

```bash
# Step 1: start + wait for boot + enable ADB + connect. Leaves the phone running.
# --max-minutes arms a watchdog that force-stops the phone on a deadline even if
# you crash. Always pass it.
python -m mlxphone.session start --phone "$MOBILE_PROFILE_ID" --max-minutes 20

# Step 3: ALWAYS run this when done, on success or failure.
python -m mlxphone.session stop --phone "$MOBILE_PROFILE_ID"
```

Wrap your work so stop always runs — the watchdog is a backstop, not a substitute
for calling stop yourself.

## Action tools (step 2 — your perception→act→verify loop)

```bash
python -m mlxphone.tools screenshot results/step1.png   # LOOK: capture the screen
python -m mlxphone.tools dump-ui results/ui.xml         # read the UI hierarchy
python -m mlxphone.tools find <keyword>                 # locate nodes: text/desc/id + bounds
python -m mlxphone.tools tap <x> <y>                    # ACT: tap real device pixels
python -m mlxphone.tools text "hello world"            # type (spaces handled)
python -m mlxphone.tools shell <args...>                # any adb shell command
```

The loop for every meaningful step:

1. **Look** — `screenshot`, and read the image.
2. **Find** — `find <keyword>` (or `dump-ui`) to get exact `bounds="[x1,y1][x2,y2]"`. Do not guess pixels for important buttons.
3. **Act** — `tap` the **center** of the bounds, or `text` to type. Coordinates are real device pixels (`shell wm size` for the resolution), independent of screenshot display size.
4. **Verify** — screenshot again and confirm the expected change before moving on. Bounds shift after navigation; re-find them.

Notes:
- glogin auth is refreshed automatically before each tool call (the device session expires in ~30–60s).
- For typing, avoid apostrophes, `&`, emoji, and shell/input control symbols; rephrase if needed. To clear a field: tap it, `shell input keycombination KEYCODE_CTRL_LEFT KEYCODE_A`, `shell input keyevent KEYCODE_DEL`, then type.

## Compliance (read before acting)

Operate within the target platform's rules and the law. This tool is for
**transparent** automation of accounts you are authorized to run.

- Do not impersonate a real, identifiable person or pose as a genuine real individual. Label created/persona/brand/managed accounts transparently where appropriate.
- Be transparent about automation; follow platform automation rules. Do not try to hide automation, "look human to avoid detection", or bypass safeguards.
- No platform manipulation: no fake engagement, warm-up/account-aging to dodge spam systems, metric inflation, follow/unfollow churn, or coordinated/bulk actions.
- Keep any engagement genuine and in normal human volumes.
- Refuse ban evasion, scraping private data, harassment, exposing PII, or illegal activity. If asked to conceal automation, impersonate a real person, or run inauthentic engagement, decline and offer a compliant alternative.

## Reporting

When done, summarize: what you did, any actions skipped and why, failures and how
you handled them, and anything that needs the user. Then confirm the phone was
stopped.

## Examples

- `examples/hello-phone/task.md` — a neutral first task (recommended starting point).
- `examples/x-avatar/recipe.md` — an optional social-persona recipe (extra ToS caveats).

---
> Source: [multilogin/multilogin-cloud-phone-agent](https://github.com/multilogin/multilogin-cloud-phone-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
