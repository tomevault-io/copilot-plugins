## vsc

> This repo contains a tiny Bash tool named `vsc`.

# Agent Notes

This repo contains a tiny Bash tool named `vsc`.

## Purpose

`vsc` starts or stops browser VS Code access for a repo using `code-server`.

It is for people who already manage their devbox, SSH, tmux, byobu, and coding agents themselves.

## Hard rules

- Do not manage tmux or byobu.
- Do not start, stop, attach to, or kill terminal sessions.
- Do not start or stop coding agents.
- Do not create a central project registry.
- Do not store secrets.
- Do not add authentication or reverse proxies beyond `tailscale serve`. TLS is provided only via `tailscale serve` on a separate https port (`port + offset`). Never put serve on code-server's own port: serve binds real sockets on the tailscale IPs and races a `0.0.0.0` bind on the same port — the loser gets EADDRINUSE after a restart (verified empirically; this was the old "bind addr conflict").
- Tailscale must stay optional. If `tailscale`/`tailscale serve` is unavailable or unauthorized, warn and continue — plain `http://ip:port` access must always work.
- Do not modify a project's `.gitignore` automatically. Use `.git/info/exclude` for local ignores.
- Keep project-local editor state in `.code-server-web/`.
- Keep the tool small.

## Preferred behavior

- Prefer readable Bash over clever Bash.
- Prefer environment variables for optional overrides.
- Prefer stateless behavior where practical.
- Use deterministic ports derived from the repo path.
- Allow manual port override with `VSC_PORT`.
- Bind `code-server` to `0.0.0.0` — host is assumed reachable only via private network (tailnet). Print the tailnet IP in URLs when `tailscale` is available.
- When tailscale is running, also configure `tailscale serve --bg --https=<port+offset> http://127.0.0.1:<port>` on `up` and remove it on `down`. The https URL is printed first.
- Use transient `systemd --user` services (`systemd-run --user`). There is no `.service` file on disk. The unit exists only while `code-server` is running and is gone after stop or crash.

## Non-goals

No web terminal. No project database. No workspace orchestration. No team management. No public internet exposure by default. No editor-state syncing between repos.

## Before changing behavior

```bash
bash -n vsc        # syntax check
shellcheck vsc     # lint (if available)
```

Core design: `vsc = code-server launcher only` — SSH, tmux, byobu, and agents are user-managed.

---
> Source: [alantian/vsc](https://github.com/alantian/vsc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
