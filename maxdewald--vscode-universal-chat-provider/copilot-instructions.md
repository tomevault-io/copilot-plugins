## vscode-universal-chat-provider

> A VS Code extension that exposes the chat-capable models from a local

# Universal Chat Provider

A VS Code extension that exposes the chat-capable models from a local
[CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) server in GitHub
Copilot Chat. Those models can also be set as Copilot's utility model, so its
background flows (commit messages, chat titles, summaries) run through them.

## Layout

- `src/chat/` — the language-model chat provider: model registry, requests, completion.
- `src/cliproxy/` — talks to CLIProxyAPI (client, SSE, credentials, and the `managed/` server lifecycle).
- `src/extension/` — activation, commands, menus, status bar.
- `src/shared/` — small cross-cutting helpers.

## Rules

- Add comments only when absolutely necessary. Code and variable names should
  explain intent on their own; reach for a comment only when something cannot be
  made clear through naming and structure.

---
> Source: [maxdewald/vscode-universal-chat-provider](https://github.com/maxdewald/vscode-universal-chat-provider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
