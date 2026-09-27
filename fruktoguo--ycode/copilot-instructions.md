## ycode

> All Codex sessions in this checkout share one development stack:

# YCode Workspace Instructions

## Shared Development Server

All Codex sessions in this checkout share one development stack:

- API: `http://127.0.0.1:7921`
- Web: `http://127.0.0.1:4323`

Before browser or API verification, run `bun run dev`. The command is
idempotent: it reuses the existing backend and Vite processes instead of
allocating another port.

- After backend source changes, run `bun run dev:restart`. If YCode
  self-iteration is available, call the visible `restart_host` /
  `ycode.dev.restart` tool instead of that shell command.
- Frontend source changes use the existing Vite HMR process for `4323`; do not
  restart Vite. HMR does not publish the change to `7921` or FRP.
- Use `bun run dev:status` to inspect the shared stack.
- Do not run `ycode serve`, `packages/cli/src/index.ts serve`, `vite`, or
  `vite preview` on another persistent port for normal development.
- An isolated port is allowed only for an explicitly isolated smoke/E2E check.
  Stop that instance in the same turn and never present it as the main URL.
- Always hand off `http://127.0.0.1:4323` as the development URL.

## Packaged Web And FRP Deployment

Unless the user explicitly says a frontend change is local-only, every frontend
handoff must also update the packaged Web served by API port `7921` and the FRP
deployment. A working Vite page on `4323` is development verification only.

Run this sequence after the frontend source is ready:

1. `bun run --cwd apps/web build`
2. `bun run scripts/generate-web-assets.ts`
3. Confirm `packages/cli/src/generated-web-assets.ts` contains the new hashed
   `assets/index-*.js` and `assets/index-*.css` entries from `apps/web/dist`.
4. Bump the product version first when the change qualifies (see Versioning).
   Then restart: `restart_host` / `ycode.dev.restart` inside YCode, or
   `bun run dev:restart` from an external shell.
5. `bun run dev:status`

Do not report the update as complete until all of these deployment checks pass:

- `http://127.0.0.1:7921/api/health` returns success.
- `https://frp.yuohira.com/ycode/api/health` returns success.
- An authenticated browser opens or reloads
  `https://frp.yuohira.com/ycode/`, loads the new hashed JavaScript asset, and
  shows the requested change with no new console errors.

Build success, source inspection, `4323` HMR, or an unauthenticated login page
alone never proves that the `7921`/FRP Web has been updated.

## Versioning

Product version is `MAJOR.MINOR.PATCH`. The only source of truth is
`packages/cli/package.json` (`scripts/dist-targets.ts readCliVersion()`).
Keep `npm/ycode` and the seven `@yuohira/*` platform packages in lockstep,
including the main package `optionalDependencies`. Do not invent a fourth
version field and do not bump by gut feel.

`apiVersion` (plugin RPC) and `schemaVersion` (SQLite migrations) increment
on their own tracks. They are not encoded in these three digits.

While the product stays on `0.x`, `MAJOR` remains `0` until an explicit `1.0.0`
decision. `0` means "not yet publicly stable", not "MINOR may break users".
Apply the same three-digit meanings as after `1.0.0`.

| Digit | Meaning | Bump when |
| --- | --- | --- |
| `MAJOR` | Upgrade is no longer seamless | User data/settings cannot auto-migrate; a published CLI flag or REST behavior breaks old clients; install, update, or plugin-loading mechanics change so an old package cannot be swapped in place |
| `MINOR` | Backward-compatible new capability | A user-visible command, settings surface, Agent/tool capability, or workflow is added, and old sessions/config/plugins on the same `apiVersion` still work |
| `PATCH` | Same capability surface | Bug, performance, security, copy, packaging, or platform-compat only; no new user-facing capability switch |

Never skip a digit or jump to a marketing number. After a `MAJOR` bump, reset
`MINOR` and `PATCH` to `0`. After a `MINOR` bump, reset `PATCH` to `0`.
Optional suffixes (`-rc.1`, `+build.12`) do not replace a required bump.

When a change qualifies for a bump and the session will restart the shared
dev host (`restart_host` / `ycode.dev.restart`, or `bun run dev:restart`
from an external shell), bump first, persist the version files, then restart.
A restarted `/api/health` and `ycode --version` must show the new version.
Do not restart first and bump later. Do not bump on a no-op restart or a
local-only frontend tweak that is not being packaged.

---
> Source: [fruktoguo/ycode](https://github.com/fruktoguo/ycode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
