---
name: basecamp-cli
description: Manage Basecamp (via bc3 API / 37signals Launchpad) projects, to-dos, messages, and campfires via a TypeScript CLI. Use when you want to list/create/update Basecamp projects and todos from the terminal, or when integrating Basecamp automation into Clawdbot workflows. Use when this capability is needed.
metadata:
  author: openclaw
---

# Basecamp CLI

This repo contains a standalone CLI.

## Install

```bash
npm i -g @emredoganer/basecamp-cli
```

## Auth

Create an integration (OAuth app) in 37signals Launchpad:
- https://launchpad.37signals.com/integrations

Then:
```bash
basecamp auth configure --client-id <id> --redirect-uri http://localhost:9292/callback
export BASECAMP_CLIENT_SECRET="<secret>"
basecamp auth login
```

## Notes

- This uses the Basecamp API docs published under bc3-api: https://github.com/basecamp/bc3-api
- `BASECAMP_CLIENT_SECRET` is intentionally NOT stored on disk by the CLI.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openclaw)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/openclaw)
<!-- tomevault:4.0:skill_md:2026-04-08 -->
