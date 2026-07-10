## handprint

> This file tells an AI agent how to install and configure handprint for transcript extraction **without any interactive prompts**.

# AGENTS.md: handprint install guide for automated agents

This file tells an AI agent how to install and configure handprint for transcript extraction **without any interactive prompts**.

---

## 1. Install

```sh
npm i -g handprint-sh
handprint init --global
```

`init --global` creates a signing key and a default config file at `~/.handprint/config.json`. It does not require a network connection or an account.

> **No Cloudflare account, no API key, and no sign-up of any kind is required** to run handprint locally.

---

## 2. Choose an extraction provider

handprint supports three extraction providers. Set one before running `grab`:

### Option A: `local` (private, free, no quota)

Runs a small language model entirely on the machine. No data ever leaves the host. No API key. No usage quota.

```sh
handprint config set extraction.provider local --global
```

Then choose a model. Pick the **smallest model the machine can run**:

| Model id | Size | RAM needed | Notes |
|---|---|---|---|
| `qwen2.5-1.5b-instruct-q4` | ~1.1 GB | 2 GB | Lightest. Good for short structured extraction on 8 GB machines / no GPU. |
| `qwen2.5-3b-instruct-q4` | ~2.0 GB | 4 GB | **Recommended default.** Strong JSON adherence; fast on Apple Silicon. |
| `llama-3.2-3b-instruct-q4` | ~2.0 GB | 4 GB | Alternative 3B if Qwen output quality is poor for your transcripts. |

Default is `qwen2.5-3b-instruct-q4`. To override:

```sh
handprint config set extraction.model qwen2.5-1.5b-instruct-q4 --global
```

The local model uses the optional `node-llama-cpp` package, which is **not** installed by the default `npm i -g handprint-sh`. Install it once to enable local extraction:

```sh
npm i -g node-llama-cpp
```

If it is missing when `grab` runs in local mode, handprint prints this exact command. The model file itself is downloaded automatically on first `grab` if not already present at `~/.handprint/models/`.

### Option C: `ollama` (local via Ollama, or any OpenAI-compatible server: LM Studio, llama.cpp, vLLM)

Routes extraction through a local OpenAI-compatible HTTP server. handprint does not download or manage models: you pull them with Ollama (or another server) and handprint POSTs to the server. Fully on-machine, no cloud, no API key required.

```sh
handprint config set extraction.provider ollama --global   # 'openai' is accepted as an alias
handprint config set extraction.model qwen2.5:3b --global
```

Default server: `http://localhost:11434/v1` (Ollama). Override:

```sh
handprint config set extraction.baseUrl http://localhost:1234/v1 --global
```

Start the server and pull the model before running `grab`:

```sh
ollama serve
ollama pull qwen2.5:3b
handprint grab
```

For servers that require an API key (LM Studio, vLLM with auth):

```sh
handprint config set extraction.apiKey <your-key> --global
```

### Option B: `host` (uses the user's existing Claude / opencode / codex quota)

Routes extraction through the agent that is already running on the machine (whichever tool generated the transcripts). Uses the user's existing quota, with no additional API key required as long as the host tool is authenticated.

```sh
handprint config set extraction.provider host --global
```

Extraction is a structured task, not open-ended generation, so the claude host engine defaults to the cheap, fast `haiku` model rather than inheriting your Claude Code default (which could be Opus). Override per machine with `extraction.model` (passed as `--model` to the claude CLI), for example `sonnet` for higher fidelity:

```sh
handprint config set extraction.model sonnet --global   # default is haiku
```

Without this setting the agent uses its own default model.

The grab plan names the detected agent by brand (e.g. `host:claude (Claude Code)`) and shows a rough input-token estimate so you can see what will be billed before confirming.

---

## 3. Scope discovery to specific sources

By default `grab` reads all implemented sources. To limit discovery to specific tools:

```sh
handprint config set extraction.sources '["claude-code","opencode"]' --global
```

Supported values: `claude-code`, `opencode`. (Additional sources `codex` and `cursor` are stubs, not yet functional.)

---

## 4. Run extraction

```sh
handprint grab
```

Use `--dry-run` to preview what would be extracted without writing anything:

```sh
handprint grab --dry-run
```

Filter to a single source at grab time (overrides config):

```sh
handprint grab --source claude-code
```

---

## 5. Summary: fully non-interactive setup

```sh
npm i -g handprint-sh
handprint init --global
handprint config set extraction.provider local --global
handprint config set extraction.model qwen2.5-3b-instruct-q4 --global
handprint config set extraction.sources '["claude-code","opencode"]' --global
handprint grab
```

No account. No cloud. No prompts.

> `config set` writes machine-wide config with `--global`. Omit `--global` to write project-scoped config, which requires a project to be initialized first (`handprint init`).

---

## 6. Visibility

Visibility (private / unlisted / public) is a hub concept, not part of the protocol or the signed record. The CLI does not set or read a visibility field.

When you push, the hub assigns the handprint a visibility level:

- unlisted (default): metadata is searchable but not listed on your public profile.
- public: listed on your public profile and discoverable.
- private: visible only to you.

Set or change visibility from the hub dashboard after pushing.

## 7. Acceptable use

`grab` and local extraction run entirely offline and require no account. Publishing to the hub (`handprint push`) is opt-in and requires `handprint login`; hub access is rate-limited and governed by the [Human Provenance Covenant](./COVENANT.md). Use a person's provenance to benefit that person rather than to build systems that route around them.

## The handprint skill

For interactive use inside an agent, handprint ships a Claude Code skill at `skills/handprint/SKILL.md` (also published in the npm package).

**The skill installs and stays in sync automatically.** `handprint init` installs it into `~/.claude/skills/handprint/`, and every subsequent CLI invocation silently re-syncs it when the recorded version does not match the running CLI. Running `npm i -g handprint-sh@latest` upgrades the skill on the next invocation.

For explicit control:

```sh
handprint skill install           # install or re-sync globally
handprint skill install --project # install into .claude/skills/ in the current project
handprint skill uninstall         # remove the global copy
HANDPRINT_NO_SKILL_SYNC=1 handprint <cmd>  # disable auto-resync for one run
```

It orchestrates the CLI: it asks how far back to capture, previews the size with `grab --dry-run`, runs `grab` (extract, sign, store) then `push` (publish as unlisted), and logs in if needed. It adds no background behavior and no new CLI commands.

---
> Source: [CameronWhiteside/handprint](https://github.com/CameronWhiteside/handprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
