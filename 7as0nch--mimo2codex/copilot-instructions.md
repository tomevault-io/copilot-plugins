## mimo2codex

> This repo is a local proxy that lets the latest OpenAI Codex CLI / desktop talk

# Agent instructions for the mimo2codex repo

This repo is a local proxy that lets the latest OpenAI Codex CLI / desktop talk
to **Xiaomi MiMo V2.5** by translating the Responses API to MiMo's Chat
Completions API. When you (the agent) run inside Codex pointed at this proxy,
the chat backend is **MiMo, not OpenAI** — adjust your assumptions accordingly.

## Hard rules

1. **Never `pip install openai` and never `import openai`.** This project
   intentionally avoids the OpenAI Python SDK. The user's API key is for
   Xiaomi MiMo, not OpenAI — `openai` SDK calls would either fail
   authentication or hit endpoints that don't exist. The sandbox also blocks
   network installs.

2. **Never assume image generation is available natively.** MiMo V2.5 does not
   have an image generation endpoint. Codex's `/hatch` (which calls OpenAI's
   `gpt-image-1`) does not work when Codex is pointed at MiMo. There is a
   ready-made workaround in `mimoskill/`; use that instead of writing fresh
   code to call `gpt-image-1`.

3. **Don't fight the sandbox by asking the user to install packages.** If you
   would normally write code that needs a Python dependency, first check
   `mimoskill/scripts/` — most things you need are already there using only
   stdlib (`urllib.request`, `json`, etc.). If you genuinely need a new
   dependency, say so explicitly with the user before adding code that
   depends on it.

## Where things are

- `src/` — TypeScript source for the mimo2codex proxy itself (Node 18+).
  Compiled to `dist/` via `npm run build`. Tests in `test/`.
- `scripts/install.sh` and `scripts/install.ps1` — bootstrap scripts (clone or
  in-repo run, install deps, build, test).
- `mimoskill/` — a self-contained directory with helpers for MiMo + workarounds
  for what MiMo doesn't do. **This is where you should look first when the
  user asks for anything beyond plain chat.**

## Specifically: generating a Codex pet from an image

The user almost certainly hits this when they say "generate a pet" /
"`/hatch` doesn't work" / "the image generation tool isn't available". The
flow you should follow is **already implemented** — just invoke it, do not
rewrite it from scratch:

```bash
# 1. Generate the pet image. --provider auto picks the best available backend
#    given the env vars: gpt-image-1 if PET_OPENAI_API_KEY is set, otherwise
#    pollinations.ai (free, no key, no signup). With only MIMO_API_KEY,
#    auto = pollinations.
python3 mimoskill/scripts/generate_pet.py \
    --description "<short chibi pet description>" \
    --out /tmp/pet.png

# 2. Optional: bundle of three states (idle / working / done) from one call
python3 mimoskill/scripts/generate_pet.py \
    --description "<...>" \
    --bundle /tmp/my-pet/

# 3. Install into Codex's pet directory (probes macOS/Linux/Windows paths)
bash mimoskill/scripts/install_pet.sh /tmp/pet.png "<pet-name>"
# or for the bundle:
bash mimoskill/scripts/install_pet.sh --bundle /tmp/my-pet/ "<pet-name>"
```

If the sandbox blocks the network call (Pollinations / OpenAI / Replicate),
do NOT respond with "please install openai". The correct response is:

> "I can't reach the network from inside the sandbox. Please run the
> following in a regular terminal (outside Codex), then tell me when it's
> done and I'll continue:
>
>     python3 mimoskill/scripts/generate_pet.py --description "..." --out /tmp/pet.png
>     bash mimoskill/scripts/install_pet.sh /tmp/pet.png "<pet-name>"
>
> No `pip install` is needed — the script uses only the Python standard
> library."

## Other MiMo capability gaps

When the user asks for something MiMo doesn't natively support, the answer
is in `mimoskill/references/models.md`. Quick rules:

- **Image input (vision)** — only `mimo-v2.5` and `mimo-v2-omni` accept it.
  `mimo-v2.5-pro` does NOT. mimo2codex auto-strips images on non-vision
  models.
- **Image generation** — none of MiMo's models do this. Use
  `mimoskill/scripts/generate_pet.py` (works for any image-gen task, not
  just pets).
- **TTS / ASR** — separate MiMo endpoints (`mimo-v2.5-tts`, `mimo-v2.5-asr`).
  Out of scope for the chat completions proxy; call them directly.
- **Code interpreter / sandboxed Python** — not provided by MiMo. Run code
  locally if needed.
- **`computer_use_preview` / `file_search`** — server-side OpenAI tools with
  no MiMo equivalent. mimo2codex silently drops them.

## Field quirks when calling MiMo directly

These bite people; use them when writing chat-completions calls:

- Use `max_completion_tokens`, NOT `max_tokens`.
- When you send `image_url` content, you MUST also send a `text` content part
  in the same array — image-only messages return 400 "Param Incorrect:
  `text` is not set". An empty space `" "` is enough.
- Reasoning is `reasoning_content` on the assistant message (DeepSeek-style),
  not `reasoning_summary`.
- Web search is a builtin tool of `type: "web_search"` — **not** a function
  tool. Requires the user to have activated the Web Search Plugin in their
  MiMo console (separately metered).
- `tool_choice` other than `"auto"` is silently ignored upstream right now.

For the canonical reference, hit
`https://platform.xiaomimimo.com/docs/api/chat/openai-api`.

## When in doubt

- Read `README.md` (English) or `README.zh.md` (Chinese) for the proxy itself.
- Read `mimoskill/SKILL.md` for the MiMo helper library and pet workflow.
- Both `node dist/cli.js print-config` and `node dist/cli.js print-cc-switch`
  emit ready-to-paste config snippets — prefer those over hand-crafting
  TOML / JSON.
- If you find yourself writing a new `import openai` or `pip install
  openai` line, stop and use `mimoskill/scripts/mimo_chat.py` (chat) or
  `mimoskill/scripts/generate_pet.py` (image gen) instead.

---
> Source: [7as0nch/mimo2codex](https://github.com/7as0nch/mimo2codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
