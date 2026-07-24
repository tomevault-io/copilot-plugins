## agenticmail

> This file is read by Claude Code when working in this repository.

# Claude Code instructions for AgenticMail

This file is read by Claude Code when working in this repository.

**For all instructions, see [`AGENTS.md`](./AGENTS.md)** — the tool-agnostic
[agents.md](https://agents.md) file used by every major AI coding assistant
(Claude Code, ChatGPT, Cursor, Aider, Codex, …).

Quick reference for the most common asks:

**"Install AgenticMail":**

```bash
# Option A — one-line installer (recommended)
curl -fsSL https://raw.githubusercontent.com/agenticmail/agenticmail/main/install.sh | bash

# Option B — manual
npm install -g @agenticmail/cli@latest && agenticmail bootstrap
```

Either path: zero prompts, ~2 minutes. See [`AGENTS.md`](./AGENTS.md)
section 1 for the full runbook.

**"Show me what my agents have been doing":** point them at the
Gmail-style web UI or the interactive REPL — both work.

```bash
agenticmail web      # opens http://127.0.0.1:3829/ in the browser
agenticmail shell    # interactive terminal REPL
```

See [`AGENTS.md`](./AGENTS.md) section 6 for the decision table.

**"Coordinate two or more agents":** use the email-thread pattern.
One kickoff email, everyone on CC, `wake: ["alice"]` to scope which
agents actually get a Claude turn from the dispatcher. Add `[FINAL]`
to the subject to close the thread. See [`AGENTS.md`](./AGENTS.md)
section 2 for the worked example.

**"Set up Twilio for outbound calls" (non-interactive — secrets
piped via env, never typed on the command line):**

```bash
TWILIO_ACCOUNT_SID='<your Twilio Account SID>' \
TWILIO_AUTH_TOKEN='<your Twilio Auth Token>' \
AGENTICMAIL_PHONE_NUMBER='<your number in E.164, e.g. +15555550100>' \
  agenticmail setup-phone --provider twilio
```

Twilio needs to webhook back into the machine, but you don't need
to think about that: if `--webhook-url` / `AGENTICMAIL_WEBHOOK_URL`
is not provided, `setup-phone` automatically opens a free Cloudflare
quick-tunnel (no account, no signup, free `*.trycloudflare.com` URL)
and uses that. The tunnel persists across runs.

If you do have your own public HTTPS URL — e.g. you ran `agenticmail
setup` in domain mode, or you have your own reverse proxy — pass
`AGENTICMAIL_WEBHOOK_URL=https://your-domain.example/` and the
auto-tunnel is skipped.

Same shape for 46elks (use `ELKS_USERNAME` / `ELKS_PASSWORD` and
`--provider 46elks`). Webhook secret is auto-minted if omitted.
Secrets are stored encrypted at rest under the master key.

**"Set up Telegram for the agent":**

```bash
TELEGRAM_BOT_TOKEN='<token from @BotFather>' \
TELEGRAM_CHAT_ID='<your numeric chat id>' \
  agenticmail setup-telegram
```

Writes the standalone Telegram bridge's three config files
(`~/.agenticmail/telegram/{telegram-token,agent-key,telegram-allowed-ids}`)
so `agenticmail start` auto-spawns the bridge alongside the API.

---
> Source: [agenticmail/agenticmail](https://github.com/agenticmail/agenticmail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
