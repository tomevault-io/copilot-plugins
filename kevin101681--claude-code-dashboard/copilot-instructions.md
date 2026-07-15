## claude-code-dashboard

> A self-hosted Node/Express web app for persistent, roamable Claude Code sessions

# Claude Dashboard

A self-hosted Node/Express web app for persistent, roamable Claude Code sessions
across a Windows PC, Chromebook and phone. No build step, no framework — `server.js`,
a few modules in `lib/`, and vanilla JS in `public/`.

## Shape of it

- **Sessions are ConPTY processes**, owned by the server itself (`@lydell/node-pty`),
  not tmux — there's no WSL distro, and a WSL Claude would have its own `~/.claude`.
  `lib/sessions.js` is the manager; `this.live` is a `Map<id, session>`.
- A session has a **kind**: `claude`, `shell` (PowerShell) or `codex`. Only a Claude
  session has a transcript, so only it gets the chat view. Don't reintroduce the
  assumption that "not a shell" means "has a conversation" — that's now false.
- **The transcript is the source of truth for anything historical.**
  `lib/discovery.js` parses `~/.claude/projects/**/*.jsonl` (head/tail chunks only —
  these files get huge). The context gauge, the model pill, chat, and search all read
  from there. The PTY stream is the source of truth for anything *live*.
- **Status is scraped from the raw PTY** (`SessionManager.computeStatus`) because
  neither the spinner nor a permission prompt ever reaches the transcript. Those
  regexes are Claude-specific; anything that isn't a Claude session reports `idle`
  rather than being read with them.
- Runs on **port 4310**. Auth is Clerk when configured, with a shared token as
  fallback (`lib/auth.js`). Both live in `config.json`, which is **gitignored**.

## Things that will bite you

- **`config.json` must never be written by PowerShell.** `Set-Content -Encoding utf8`
  adds a BOM and breaks `JSON.parse` on the next start. This has already caused data
  loss once. Write it with Node.
- **The composer must send prompt text and `\r` as separate PTY writes**, ~200ms
  apart. A trailing `\r` in the same chunk trips Claude Code's paste detection and
  the prompt just sits in the input box, unsubmitted.
- **Never `existsSync()` a UNC path on the `/api/state` path.** An unreachable SMB
  host blocks for *seconds* on Windows, and that endpoint is polled every 5s — one
  dead network share would freeze the whole dashboard. (A `\\host\share` UNC path can
  be a pinned project, and that host does go down.)
- **Windows keeps a trailing separator on a root path.** Launch a session in
  `\\host\share` and Claude reports its cwd as `\\host\share\`, which munges to a
  *different* project-directory name. `normalizeCwd` / `projectDirFor` in
  `lib/discovery.js` exist for exactly this; use them rather than munging a path
  yourself.

## Conventions

- Match the surrounding code: vanilla JS, no framework, comments that explain *why* a
  constraint exists rather than narrating what the next line does.
- Client assets are cache-busted by mtime, so a `public/` change is live on refresh.
  Anything in `server.js` or `lib/` needs a server restart — and a restart kills every
  live session, so say so rather than doing it silently.

---
> Source: [kevin101681/claude-code-dashboard](https://github.com/kevin101681/claude-code-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
