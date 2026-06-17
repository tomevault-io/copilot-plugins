## handoff-baton

> Watch conversation context size and, when it crosses a threshold, hand the session off to a fresh one via a saved handoff document. Generates the handoff doc and manages per-session thresholds.


# handoff-baton

A relay baton for long conversations: when the context grows large, write a
handoff document so a fresh session (`claude` or `codex`) can pick up the work
with a small, cheap context — instead of paying to re-send a huge transcript
every turn.

## When you are invoked

There are two paths:

- **Automatically**, by the `handoff-baton` Stop hook, when context crosses the
  threshold. The hook injects the current token count, the `session_id`, the
  target handoff directory, and the exact `hb-state` commands to run. Present the
  options menu below.
- **Manually**, when the user runs the skill themselves. Skip the menu and
  generate a handoff document immediately. If the user passed an argument, treat
  it as the focus for the next session.

## Options menu (hook-triggered only)

Ask the user to choose, then act on their choice. **Never choose for them.**

1. **Handoff now** — Write the handoff document (see below), then tell the user
   to resume with `hresume claude` (or `hresume codex`).
2. **Extend +10K** — Run the `hb-state extend <session_id> <value>` command the
   hook provided, then continue the user's current work.
3. **Disable for this conversation** — Run the `hb-state disable <session_id>`
   command the hook provided, then continue.
4. **Skip** — Do nothing; the hook will ask again on the next turn. Just
   continue the work.

## Writing the handoff document

Save to the directory the hook provided:
`<data>/handoffs/<project>/handoff-<YYYYMMDD-HHMM>.md`. When invoked manually,
use this repo's `handoffs/<project>/` folder with the current timestamp.

Include:

- **Goal / current focus** — what we're trying to achieve. If the user gave an
  argument, tailor the doc toward that next focus.
- **State** — what's done, what's in progress, what's next.
- **Key decisions & constraints** — non-obvious choices and the reasoning.
- **Open questions / blockers.**
- **Pointers** — reference PRDs, plans, ADRs, issues, commits, diffs, and key
  files by path or URL. Do **not** duplicate their content.
- **Suggested skills** — skills the next session should invoke.

Rules:

- **Redact secrets** — API keys, passwords, tokens, PII.
- **Be concise.** The whole point is a small starting context. Link, don't paste.

## After writing — how to resume

Once the doc is written, print the file path and a copy-pasteable resume
command. Do **not** try to open or launch anything automatically (the current
session can't relaunch itself, and guessing the user's terminal is unreliable).

Use the **agent the user is actually running** for the command. The hook's
context names it (`current agent: …`); if invoked manually, detect it from
`$AI_AGENT` / `$CLAUDECODE` (Claude Code) or `$CODEX*` (Codex), defaulting to
`claude`. Substitute that for `<tool>` below.

Tell the user exactly this (prefer the **bare** command — no path to paste):

> Handoff saved: `<full path to the doc>`
>
> To continue, exit this session (`/exit` or Ctrl-D), then from this folder run:
>
> ```
> hresume <tool>
> ```
>
> That loads the newest handoff for this project. If you've changed folders,
> pass the file explicitly: `hresume <tool> "<full path to the doc>"`.

---
> Source: [Rorogogogo/handoff-baton](https://github.com/Rorogogogo/handoff-baton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
