## vox

> Turns one topic into a finished narrated motion-graphics explainer video. One

# vox — agent rules

Turns one topic into a finished narrated motion-graphics explainer video. One
approval gate, one backend: the **Higgsfield MCP**.

`skills/vox-style/SKILL.md` is the pipeline. This file is the project contract.

## Layout

Four skills under `skills/`, each symlinked from `.claude/skills/`:

| Skill | Role |
|---|---|
| `vox-style` | **Main.** The pipeline and the five modes. |
| `higgsfield-video-explainer` | Vendored. Owns every tool call and the assembly contract. |
| `higgsfield-generate` | Vendored. Model catalogue and prompt-engineering reference. |
| `higgsfield-soul-id` | Vendored. Identity models. |

`vox-style` is a **front end**, not a reimplementation. It adds input handling
(trend / article / no topic), three locked house styles, one approval gate with a
credit estimate, four surrounding modes in `references/modes.md`, and a durable
record under `output/`. Everything else belongs to the vendored workflow.

Run folders are named for the subject, not the date — `output/<topic-slug>/` —
because a date does not distinguish two runs on the same day and a slug says what
the run actually was. The date lives in `brief.md` and `manifest.json`.

`output/` is gitignored **deny-by-default**: everything is ignored except
`brief.md`, `script.md`, `storyboard.md` and `manifest.json`. The text record of
every run is tracked; not one byte of media is. Adding a new media format needs
no gitignore change, which is the whole reason the rule is written that way.

## Precedence

When two sources disagree: **live MCP > vendored skill > vox-style.** If a tool's
real behaviour contradicts a document, the tool is right and the document gets
fixed — with a note saying it was verified on a real run, since that is the only
thing that distinguishes a correction from a guess.

`vox-style` overrides upstream in exactly two documented places, both in its
SKILL.md: the wizard, which replaces upstream's Phase 0, and the one-gate rule.

## Vendored skills are CLI-era

Upstream `higgsfield-ai/skills` was `0.12.0` at vendor time and is written in
`higgsfield ...` shell commands. **There is no CLI here.** Each vendored skill
carries a banner and a `references/mcp-mapping.md` with the translation table.

- `higgsfield-video-explainer/SKILL.md` is the hand-adapted MCP version
  (`1.2-mcp`), not upstream's text.
- `higgsfield-generate` and `higgsfield-soul-id` are upstream verbatim plus the
  banner. Their `marketing-studio`, `workflow` and `soul-id` subcommands have MCP
  equivalents this project has not exercised — **do not guess a tool name.** Read
  the connected tool list, or say it is unmapped.

Re-vendor:

```bash
git clone --depth 1 https://github.com/higgsfield-ai/skills /tmp/hf
cp -R /tmp/hf/higgsfield-generate skills/
# then restore the banner and references/mcp-mapping.md
./scripts/build-skills.sh
```

## Always

- **Paths anchor on `PROJECT_ROOT`** (this directory, marked by `.vox-root`). The
  project must stay exportable and work when opened directly. Skill folders are
  read-only.
- **One approval gate, and it is real.** The gate is about money, not turns. The
  wizard asks seven questions every run and research, script and storyboard run
  to completion after it — all of it free. Then stop, show everything, quote the
  credit cost against the live balance, and wait. Never submit a paid job before
  the user replies. The gate is the only place a credit is committed and the only
  stop that needs a yes.
- **The wizard runs every time.** Never skip a question because the request
  already answered it. What the user said becomes that question's pre-selected
  first option instead, so accepting it costs one keystroke.
- **Quote real numbers.** `balance` for the balance, `get_cost: true` for the
  price. Preflight is free. Never guess.
- **The style key attaches to every clip.** This is the entire reason the output
  reads as one production. A clip without it is a defect, not a variation.
- **Pass `aspect_ratio` explicitly on every clip.** It does not inherit from the
  key image. Verified on a real run.
- **Voice before clips, strictly.** Each take's real duration decides whether a
  line needs shortening before 30 credits go on its clip.
- **Assemble server-side** with `explainer_video`. `scripts/assemble.py` is the
  offline fallback only.
- **Save every job id to `manifest.json` as you go**, so a crashed run resumes
  instead of re-paying.
- **Rebuild `dist/` after touching any skill** — `./scripts/build-skills.sh`. The
  bundles are the download path for Cowork and stale ones are worse than absent.

## Never

- Never run a `higgsfield` shell command.
- Never invent a number, a source, or an MCP tool name.
- Never generate the same block twice to get two aspects. Dual aspect is one 16:9
  render plus a free center-cropped re-assembly; see the dual-aspect rule in
  `vox-style/SKILL.md`.
- Never accept a server `preset_recommendation`. Resubmit with
  `declined_preset_id`.
- Never deliver loose clips. Assembly happens in the same run.
- Never add a credential to this project. It has none, by design.

## Scope

Delivery stops at the MP4 and its source list. No titles, descriptions, hashtags,
thumbnails, or uploads.

---
> Source: [EricTechPro/vox](https://github.com/EricTechPro/vox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
