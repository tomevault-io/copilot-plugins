## android-use

> This repository builds one Windows-first AI Agent with four bounded capabilities:

# Android Use Agent Guidance

## Product Boundary

This repository builds one Windows-first AI Agent with four bounded capabilities:

- workspace file inspection and text patching inside one frozen workspace root;
- Android observation and input through the internal Android Use MCP child;
- explicit-request-only model-managed memory under the protected Agent state root;
- whitelisted QQ task ingress and plain-text QQ message egress through a loopback OneBot connection.

QQ moderation, Qzone, profile, media, browser, and every non-text-message operation remain Android
Use tasks. OneBot must not expose those operations.

The Agent must never expose arbitrary PowerShell, `cmd`, host shell, arbitrary `adb shell`, root,
lock bypass, permission bypass, file deletion, rename, chmod, or binary file editing.

## Required Reading

- Read this file and `docs/current.md` in full before repository work.
- Read `docs/reference/tool-contracts.md` in full before every Android Use MCP or JSON CLI use.
- Treat `docs/roadmap.md` as the approved execution plan and future-work authority.
- TypeScript contracts remain authoritative for machine schemas.

## Development Workflow

- Use tests first for production behavior changes and verify the intended RED before implementation.
- Keep one TypeScript package until a real release boundary requires otherwise.
- Use argument arrays with `shell: false` for every child process.
- Keep MCP stdout protocol-only; diagnostics go to stderr or ignored local artifacts.
- Keep screenshots, UI XML, device logs, and local plans under ignored `.artifacts/` or `.local/`.
- Keep retained Android artifacts bounded to 20 complete snapshots per device hash by default.
- Do not add hosted CI unless the user explicitly requests it.
- Do not explain why changes were made; record only what changed.

## Workspace Boundary

- Freeze one canonical workspace root at startup.
- Accept only relative paths and reject absolute, UNC, drive-relative, ADS, traversal, control-character,
  symlink, junction, reparse-point, and hard-link escapes.
- Deny `.git`, `.local`, `.artifacts`, `node_modules`, build outputs, environment files, keys, and the
  Agent's own control-state files.
- File patches require a matching SHA-256 base and unique exact-text hunks. Do not add a shell tool.

## Git Workflow

- Every implementation task ends with a reviewable Git commit.
- TDD checkpoint commits are allowed when they preserve a verified RED/GREEN boundary.
- Commit messages use `git(<type>): 中文解释` without a business scope.
- Before committing, inspect status and staged diff; stage only task-owned files.
- Never stage an API key, device serial, personal screenshot, local absolute ADB default, or ignored
  artifact. Report commit hashes and verification results at handoff.

## Real Device Boundary

- Read-only connection diagnostics may run when relevant.
- Obtain explicit consent before capturing a personal screen or UI hierarchy.
- Any tap, swipe, key, text input, app launch, install, uninstall, IME switch, permission change, or
  other device mutation requires explicit approval for that testing step.
- Prefer the fixture app. Never test mutations against personal settings, messages, payments, photos,
  accounts, or other personal applications.
- Never commit an external ADB path or device serial as a runtime default.

## Verification

Run the local gate:

```powershell
pnpm check
```

For UI changes, also inspect Playwright screenshots at desktop and compact viewports and verify the
Canvas is nonblank, the layout stays within 100vh, text does not overlap, and no non-green accent is
introduced. Report skipped gates or environment limitations explicitly.

---
> Source: [lzdycx/android-use](https://github.com/lzdycx/android-use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
