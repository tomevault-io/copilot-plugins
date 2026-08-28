## dsh-plugin-conversation-outline

> `@deepseek-ai/dsh-client-ui-conversation-outline` is a browser-only DeepSeek Harness plugin. It renders a collapsible outline rail to the left of the Web chat transcript: a clickable, flow-ordered index of the current session's user questions and agent answers, plus a keyword search box that filters the index over each message's full text (case-insensitive). Clicking an entry scrolls the transcript to that message and flashes it.

# AGENTS.md — Deployment Guide (agent-facing)

## What this package is

`@deepseek-ai/dsh-client-ui-conversation-outline` is a browser-only DeepSeek Harness plugin. It renders a collapsible outline rail to the left of the Web chat transcript: a clickable, flow-ordered index of the current session's user questions and agent answers, plus a keyword search box that filters the index over each message's full text (case-insensitive). Clicking an entry scrolls the transcript to that message and flashes it.

## Runtime contract

- **Platform**: `web` — client bundle only. The host half (`lib/index.js`) is a no-op `apply()`.
- **Slot**: `conversation.outline` — a `session`-scoped `single` slot **declared** by `@deepseek-ai/dsh-client-ui-conversation` (ui-conversation). This package only **occupies** it via `ctx.slots.inject('conversation.outline', ...)`; composing this package out leaves the seat empty at zero cost.
- **Services**: `slots`, `locale`.
- **Locale namespace**: `conversation-outline` (zh + en dictionaries).
- **Data access**: the occupant reads the session chat snapshot through the framework `useSession` hook only; it never calls the model, the host API, or any other plugin's service.
- **Rendering**: the rail renders only while a current session exists and is non-blank (a blank session has no conversation to index).

## Mounting in a deployment

Declare a row in the deployment's cordis.yml, like the shipped example in `packages/bundle/web-app/cordis.patch.yml`:

```yaml
plugins:
  - id: ui-conversation-outline
    name: '@deepseek-ai/dsh-client-ui-conversation-outline'
```

Requirements:

- The config-tree package must declare this package as a dependency.
- Loader edges (`dsh.client.inject`): `@deepseek-ai/dsh-client-locale`, `@deepseek-ai/dsh-client-runtime`, `@deepseek-ai/dsh-client-ui-conversation`.
- The bundle export (`exports["./client"]`) must be built before boot (see below); the client-modules service serves it at `/plugins/ui-conversation-outline/client.js`.

## Building

From the repository root:

```sh
pnpm run build:lib:client   # type-checks tsconfig.client.json and bundles all client packages
```

Artifacts land in the package directory:

- `lib/client.js` — browser bundle (served by the client-modules service)
- `lib/index.js`, `lib/invariant.js` — host loader stubs
- `lib/types/**` — type declarations

Single-package rebuild: `pnpm --filter @deepseek-ai/dsh-client-ui-conversation-outline bundle`. Distributable tarball: `npm pack` inside the package directory.

## Verifying

- Boot `dsh web` and open a non-blank session.
- The rail appears left of the transcript with the localized title (对话目录 / Outline) and a search box in its header.
- Clicking a row scrolls the transcript to that message and flashes it; typing a keyword filters the index over full message text.
- Plugin-set changes take effect on **restart** (the client-modules scan caches per package name).

## Removing

Delete the cordis.yml row (and optionally the dependency) and restart; the `conversation.outline` seat renders nothing when unoccupied.

## Known limitations

- The jump scrolls by chat node key, so it reaches only the currently loaded transcript window; a message outside the loaded history page needs that page loaded first.
- Search is a case-insensitive substring match over text blocks only; reasoning and tool output are not indexed.
- Assistant entries show their first text blocks only.

---
> Source: [XavierMarquis93/dsh-plugin-conversation-outline](https://github.com/XavierMarquis93/dsh-plugin-conversation-outline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
