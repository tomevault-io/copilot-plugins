## chatgpt-local-mcp-kit

> Read `AGENT-DEPLOYMENT.md` in full before deployment. Read `docs/INSTALL.md` for Windows or `docs/MACOS.md` for macOS before modifying a running installation.

# Instructions for agents working with this deployment kit

Read `AGENT-DEPLOYMENT.md` in full before deployment. Read `docs/INSTALL.md` for Windows or `docs/MACOS.md` for macOS before modifying a running installation.

## Scope

- This is a Windows x64 and macOS Universal packaging/deployment repository, not the upstream AgentDock implementation.
- Detect the operating system first. Never run Windows PowerShell deployment scripts on macOS or macOS `.command`/zsh entries on Windows.
- Distinguish preparing a distributable from installing it. A packaging request does not authorize reinstalling or restarting the user's live MCP service.
- Do not create another public repository, push commits, publish a release, register a domain, create scheduled jobs, or send a ChatGPT message without the user's corresponding authorization.
- No subagent delegation is required by this file.

## Security and user control

- Use the signed-in ordinary user. Do not use Windows administrator, macOS root, `sudo`, LaunchDaemon, or a second service lifecycle as a shortcut.
- Keep loopback binding, authentication, OAuth, HTTPS, TLS verification, Gatekeeper and organization controls.
- Never copy real runtime/configuration files, DPAPI data, `.codex`, `.agentdock`, `~/Library/Application Support/AgentDock`, browser profiles, cookies, credentials, private logs, screen captures or macOS privacy databases into Git or a release.
- Do not show passwords, Bearer Tokens, Tunnel Tokens, OAuth signing keys or authenticated request headers. The macOS App and Windows upstream installer can reveal secrets; leave viewing and entry to the user.
- User logins, OAuth entry, UAC, macOS Login Items approval, Accessibility and Screen Recording decisions belong to the user.
- A new machine or user account needs new credentials. This package contains no reusable account access.

## Preserve existing state

- Inspect before acting. Windows `Install-AgentDock.ps1` and macOS `01-Install.command` must not overwrite an existing installation.
- On macOS, AgentDock.app and its `SMAppService` entries are the lifecycle authority. Do not add competing plist files, cron jobs or login items. Use the App for enable/disable operations.
- `Enable-Features.ps1` must preserve unrelated Windows settings and existing enabled ACP by default.
- Backups stay in private platform locations, never under Git.
- Prefer a Core-only recovery when settings change. A Quick Tunnel regeneration can change the public URL and requires advance disclosure.
- Recovery helpers must no-op when the target is already healthy and perform at most one bounded unhealthy-Tunnel repair per run.
- Never broadly terminate all Node/cloudflared processes or delete an entire user, workspace, Applications, state or support tree.

## Platform baselines

- Windows: AgentDock 0.8.0 and cloudflared 2026.8.2; optional isolated Node 24.15.0, Codex 0.148.0 and Codex ACP 1.7.0.
- macOS: official AgentDock.app 0.8.1 Universal DMG, macOS 13+, arm64 and x86_64. The App bundles its matching Core, cloudflared and core Skills.
- Keep Windows dependency pins in `config/dependencies.lock.json` and macOS pins in `config/dependencies.macos.lock.json`. `config/macos-release.env` is a shell-safe mirror and tests must prove it matches the JSON lock.
- Never replace a digest merely to accept an unexplained file. Check the official upstream release metadata and behavior first.

## Features and validation

- AgentDock 0.8.1 replaced `server_info` with `agentdock_context`. Cross-platform prompts should try `agentdock_context`, then use `server_info` only for the locked Windows 0.8.0 baseline.
- macOS browser tools are enabled in AgentDock.app Advanced Settings. Do not edit `agentdock.env` behind the App for settings the App manages.
- macOS desktop control requires the official digest-pinned `desktop` Skill and user-granted Accessibility/Screen Recording permissions. Do not infer those permissions from tool discovery.
- Keep PowerShell/CMD files compatible with Windows PowerShell 5.1 and ASCII. Keep zsh/`.command` files UTF-8, LF, executable and compatible with macOS 13's system zsh.
- Run `tests/Run-Tests.ps1`, `scripts/Test-Repository.ps1`, `tests/Run-Mac-Tests.zsh` on a real macOS runner, and the appropriate dependency/app verification.
- Do not run real model prompts, restart a live healthy Tunnel, grant privacy permissions, or write to private projects just to test packaging.
- Health, anonymous 401, OAuth, tool discovery, browser navigation, desktop observe/action, ACP handshake and real web read/write are separate checks. Report only what actually ran.

## Publication

- Use the platform exporters, which select committed source plus explicitly verified dependency assets.
- Never archive an entire workspace, home directory, live installation or `.git` directory wholesale.
- Keep binaries out of ordinary Git history. Full releases carry vendor assets; Git history travels as a reviewed bundle.
- Before delivery, inspect archive members, Git history, links, executable bits, app signature/Gatekeeper results and secret/private-path scans.
- Published release assets are immutable in normal operation. Build a new version instead of replacing a published release.

---
> Source: [zararogers338-hash/chatgpt-local-mcp-kit](https://github.com/zararogers338-hash/chatgpt-local-mcp-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
