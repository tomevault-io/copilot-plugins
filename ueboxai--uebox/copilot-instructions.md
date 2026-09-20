## uebox

> Instructions for AI coding agents working in this repository.

# AGENTS.md

Instructions for AI coding agents working in this repository.
中文版：[AGENTS.zh-CN.md](AGENTS.zh-CN.md)（两份必须同步更新，CI 会检查）

---

## 1. What this project is

**Unreal Box (虚幻盒子)** — an agent harness for Unreal Engine: a local runtime that lets an AI
agent actually operate an Unreal project. Electron + Vue 3 + TypeScript.

The agent is the product. The harness around it is model access, the Unreal tool set, the
skills, and the constraints and verification that keep it in bounds; the engine connection
itself runs through the bundled UnrealAgentLink plugin.

The other modules — project library, asset library, blueprint and material libraries,
notebooks and wikis — each have a full standalone UI, but structurally they exist to serve the
agent: they register what it acts on, feed it material, and receive what it produces. Treat
them that way when deciding where a feature belongs.

This repository is the open-source desktop application. Local features work fully offline,
without an account or region gate. Optional AI and network vault features use services
explicitly configured by the user.

**Never** add telemetry, remote calls, or an account requirement to a local feature.

## 2. Setup

```bash
pnpm install
pnpm dev          # start the app (Electron)
```

Windows is the primary platform. pnpm 10.28.2, Node 24.21.0.
The project pins Node in `.npmrc`; use `pnpm exec node` for direct Node commands so they use
the project runtime even when the system Node differs.

## 3. The gate — two tiers, and which one you owe

**Finishing a single task:**

```bash
pnpm verify:changed     # ~30s — only what you touched
```

**Before the work is published** — you are about to `git push`, cut a release, or hand the whole
batch over:

```bash
pnpm verify             # ~3.5 min — the full gate
```

**Do not run the full gate after every small task.** Those 3.5 minutes are almost all fixed cost:
the full lint ratchet is 49s, both typecheck projects 45s, and the whole test suite 126s — of which
only 43s is actually executing tests (the rest is 386 test files each building an environment).
Paying that per task is the biggest single waste of time in this repo. With several sessions
working at once it is also actively misleading: the gate is repo-wide, so another session's
half-written file turns **your** gate red (§5 rule 11).

`verify:changed` is a snapshot, not a substitute — the full gate still has to pass before anything
is published. Two things make deferring it safe:

- Every task got its own 30-second check, so a red full gate points at _this_ batch, not at a task
  someone finished three hours ago and can no longer reconstruct.
- **CI is the backstop.** `.github/workflows/quality.yml` does nothing but `pnpm verify --ci`, and
  it runs on **every push** — the full gate plus packaging and the offline-boot check. The gate is
  defined in exactly one place (`scripts/verify.mjs`), so local and CI cannot drift. Forgetting to
  run it locally costs you a red run on GitHub, not a broken release.

Every step tells you what to fix when it fails. You can leave the app running while it runs — the
gate no longer touches the `better-sqlite3` native binding.

| Step | What it checks |
|---|---|
| `security:secrets` | No credentials committed |
| `verify:official-endpoints` | Ratchet: the community core must not gain new official-server calls |
| `lint` | Ratchet: full-repo ESLint, per-file counts must not exceed `scripts/lint.baseline.json` |
| `lint:changed` | **Full ESLint rules on the lines you changed** — zero tolerance |
| `docs:check` | Bilingual doc pairs were updated together |
| `verify:skills` | `resources/skills/**` conforms to `resources/skills/SKILL_STANDARD.md` |
| `verify:ue-file-reads` | Engine-authored files (`.uproject` / `.uplugin` / engine `ini`) are never read with a hard-coded encoding — see §5 |
| `plugin:check` | Plugin source changed ⇒ the zip must be rebuilt — this step checks UE 5.5 only |
| `typecheck` | `tsconfig.node.json` (main) + `tsconfig.web.json` (renderer) + `tsconfig.cli.json` (`packages/cli`) |
| `test:run` | Vitest, whole-repo unit tests (no count here — it changes daily and would just go stale) |
| `audit:prod` | CI only — no high-severity vulnerabilities in production deps |
| `build:unpack` | CI only — production build + packaging + packaged-dependency closure |
| `verify:offline-boot` | CI only — boots the packaged app, walks every page, asserts zero official-server requests |

What the per-task tier actually does: `verify:changed` skips the full lint ratchet (new code is
still covered by `lint:changed`, zero tolerance), typechecks only the side you changed, and runs
only the tests related to your diff. Everything else in the table above still runs — those steps
are cheap.

Its comparison base is `$VERIFY_BASE` → `origin/main` → `main` → `HEAD`. So as an unpushed batch
grows, `verify:changed` slowly widens to cover the whole batch. To check strictly the task in
front of you, pin it: `VERIFY_BASE=HEAD pnpm verify:changed`.

Two other variants: `pnpm verify:fast` (the whole gate, but the test step skips the native-ABI
preflight) and `pnpm verify --with-build` (adds packaging and the offline-boot gate locally — use
it when you touched the main process, build config, or dependencies).

Plugin package freshness comes in two tiers. `plugin:check` inside `pnpm verify` covers **UE 5.5
only** (the version you build during development); rebuild with
`pnpm plugin:build --engine 5.5 --project <uproject path>`. The strict tier is
`plugin:check:all`, wired into the official installer scripts (`build:win` / `build:mac` /
`build:linux`); it requires **every version, 5.0–5.8, to be
fresh** — ship a release with one version behind and those users really do install a stale
plugin. Rebuild the whole set with `node scripts/build-all-plugins.mjs`.

Shipping an official installer is one command, `pnpm build:win`, with no channels (the old
personal / enterprise split is gone, along with `build:win:personal`, `sync:update-feed`, and the
channel marker file). There is exactly one update source: **public GitHub Releases**. Set
`updateGithubRepo` in `package.json` to `owner/repo` and the build script injects the publish
config and emits `latest.yml`; leave it empty and you get a package with no update source at all —
it makes no update request on boot, which is also what anyone who clones and builds this repo
gets. To release, upload `dist/`'s `.exe`, `.blockmap`, and `latest.yml` to that repository's
Release; there is no other sync step. Runtime resolution lives in
`src/main/services/updater/updateFeed.ts`.

**Do not** make the gate green by skipping, deleting, or `.skip`-ing tests, by widening ESLint
ignores, or by loosening the coverage threshold. If a gate is genuinely wrong, say so in the PR
instead of routing around it.

## 4. Where code goes — the vertical slice

Almost every feature crosses the same seven layers. Skipping one is the most common failure mode.
Using tags as the example:

| # | Layer | Path | Responsibility |
|---|---|---|---|
| 1 | Model | `src/main/sqliteDataBase/models/tag.ts` | Table DDL + SQL |
| 2 | IPC handler | `src/main/sqliteDataBase/ipc/tag.ts` | `ipcMain.handle('db:tags:*')` |
| 3 | Preload bridge | `src/preload/index.ts` | `contextBridge` → `window.api.database.tag.*` |
| 4 | Preload types | `src/preload/index.d.ts` | **Must change together with #3** |
| 5 | Renderer API | `src/renderer/src/api/tag.ts` | `tagAPI.*`, unwraps via `unwrapResult()` |
| 6 | Store | `src/renderer/src/store/modules/*` | Pinia, only if state is shared |
| 7 | View | `src/renderer/src/views/**` | Vue components |

Full walkthrough with a real worked example: [docs/contributing/vertical-slice.md](docs/contributing/vertical-slice.md).

For project vocabulary, read [CONTEXT.md](CONTEXT.md).

Other entry points: main-process IPC in `src/main/ipc/`, AI agents in `src/main/agent-v3/`
(`agent-v2/` is gone; it only survives in historical docs), shared types in `src/shared/`.

Four areas sit outside that slice. Each carries its own obligation, and a change there is not done
until it is met:

| Area | Path | Obligation |
|---|---|---|
| Agent tools | `src/main/agent-v3/tools/` | Define with `defineTool` / `defineUeTool` and register in `registry.ts`; mind the `appSettingsManager` trap in §7 |
| Runtime skills (loaded by the in-app agent and the CLI) | `resources/skills/**`, `packages/cli/skills/**` | Conform to `resources/skills/SKILL_STANDARD.md` — gate step `verify:skills` |
| UE plugin | `plugin/UnrealAgentLink/` | Source changed ⇒ rebuild the UE 5.5 zip, or gate step `plugin:check` fails (§3) |
| `uebox` CLI | `packages/cli/` | Typechecked by the gate via `tsconfig.cli.json`; `pnpm verify:cli-package` (build + package check) is not in the gate — run it before publishing the package |

## 5. Hard rules

1. **No hard-coded colors or spacing.** Use the CSS variables in
   `src/renderer/src/assets/styles/theme.css`. See [docs/UI-Design-Standards.md](docs/UI-Design-Standards.md).
   The app has dark / light / high-contrast themes; a raw HEX breaks two of them.
2. **Check [docs/ui-components.md](docs/ui-components.md) before building UI.** The renderer has
   two component sources **on purpose**: our own `App*.vue` and what is left of `ant-design-vue`.
   The line: anything the user clicks or that pops up a layer is ours; form inputs are still antd.
   ESLint stops you if you pick wrong and names the right component in the error. Never wrap an
   antd component we already replaced.
3. **User-visible strings are bilingual.** Add the key to **both**
   `src/renderer/src/i18n/locales/zh-CN.ts` and `en-US.ts`. Never hard-code Chinese in a
   `.vue` template for new code.
4. **`src/preload/index.ts` and `src/preload/index.d.ts` change together.** Typecheck fails otherwise.
5. **The renderer never calls `ipcRenderer` directly.** Go through `window.api.*`, and wrap it in
   `src/renderer/src/api/*` using `unwrapResult()` so errors surface consistently.
6. **New behaviour needs a test.** Put it in `tests/` or beside the source as `*.test.ts` — both
   are picked up. See [docs/contributing/testing.md](docs/contributing/testing.md).
7. **No new dependencies, network calls, or telemetry** unless the issue explicitly asks for it.
   `pnpm audit:prod` runs in CI and a new package will be scrutinised.
8. **Don't reformat files you didn't change.** A 900-line diff for a 10-line feature will not be merged.
9. **Read before you write.** Check `docs/` for an existing spec on the area you're touching
   (`docs/资产库避坑指南.md`, `docs/ContextMenu组件使用规范.md`, `docs/router-meta.md`, …).
10. **For anything the user authored, the file on disk is the single source of truth.** The database
   is an index you can delete and rebuild; localStorage only holds UI preferences and one-shot flags.
   That is what makes the community edition's "your data stays yours" promise real — it has to be
   copyable, backup-able, and still there on another machine. Same for credentials: localStorage is
   plaintext, so use the main process's secure storage. Gate: `scripts/check-local-storage.mjs`;
   existing debt is registered in `scripts/local-storage.baseline.json` and may only shrink.

11. **This repo is often edited by several agent sessions at once.** Three rules:
    - **Stage your own files by path.** `git add -A` / `git add .` / `git commit -a` sweep
      another session's uncommitted work into your commit — nothing is lost, but the history
      is now wrong and the change is untraceable. These commands are in the `deny` list in
      `.claude/settings.json`.
    - **Run the full `pnpm verify` only when handing off, and only one session at a time.**
      Use `pnpm verify:changed` while you work. The gate is repo-wide: a file another session
      is halfway through will turn your gate red, and that is not yours to fix.
      **Do not edit files outside your task just to make the gate green** — say in your handoff
      whose work-in-progress is blocking it. That is more useful than finishing it for them.
    - **The four ratchet baselines are shared** (lint / colors / local-storage /
      official-endpoints under `scripts/`). They can only be updated one session at a time;
      concurrent updates always conflict. Before updating one, confirm you are the only
      session wrapping up.

12. **Never read engine-authored files with a hard-coded encoding.** UE saves
    `.uproject` / `.uplugin` through `SaveStringToFile` with AutoDetect: one non-ASCII
    character anywhere in the file (a Chinese project name or description) and the whole
    file is written as UTF-16LE. Reading it as `'utf-8'` yields mojibake and `JSON.parse`
    throws `Unexpected token '�'` — which reaches the user as "the plugin will not
    install, the AI cannot see the engine", wrapped in an error message they cannot read.
    Always read through `src/main/utils/ueTextFile.ts` (`readUeTextFile` / `readUeJsonFile` /
    `decodeUeText`); write back as UTF-8 without a BOM (with no BOM the engine decodes as
    UTF-8, identically on 5.0-5.8). Gate: `scripts/check-ue-file-reads.mjs`.

## 6. Commits and PRs

- Conventional Commits with a **Chinese** description, matching existing history:
  `feat: 支持自定义标签颜色`, `fix: 修复打包产物无法启动的两个缺陷`, `feat!: 卸载工作流 Nexus`
- Branch name: `feat/<slug>`, `fix/<slug>`
- Fill in `.github/PULL_REQUEST_TEMPLATE.md` honestly, including whether `pnpm verify` passed.
- **Stop and ask the human before `git push` or opening a PR.** Committing locally is fine;
  publishing is the contributor's decision, not yours.
- AI-assisted contributions are welcome and explicitly encouraged. Say so in the PR — it is not
  held against you. What is reviewed is the diff and the gate, not who typed it.

## 7. Traps

- **`better-sqlite3` ABI — no longer a trap.** Node and Electron need different ABIs,
  but both artifacts are now cached side by side: tests read the Node one through a vitest alias,
  `node_modules` keeps the Electron one. **You can run the tests with the app open.**
  If you ever see `NODE_MODULE_VERSION`, the cache is missing —
  `node scripts/better-sqlite3-abi.mjs ensure node` fixes it without touching `node_modules`.
- **`pnpm lint` is a ratchet, not a clean bill of health.** It lints the whole repo, but compares
  against a recorded backlog (`scripts/lint.baseline.json`: 341 files, 2435 errors, 3440 warnings).
  Existing violations are not your job to fix — they just must not grow.
- **If the gate says the backlog shrank**, run `pnpm lint:baseline:update` and commit the updated
  baseline. The ratchet only turns one way: `--update` refuses any increase.
- **`pnpm lint:changed` is what actually guards your code.** The ratchet still allows swapping 30
  existing problems for 30 different ones in the same file; the per-line gate does not.
- **Coverage thresholds (80%) apply to `src/renderer/**` only\*\*; main and preload are excluded
  from coverage but still must have tests for logic you add.
- **Never statically import `appSettingsManager` from an agent tool.** It starts with
  `import { logger } from './services'`, and that module drags in the WebSocket service and the
  task runner, ending at the `better-sqlite3` native binding. The tool registry is the entry point
  for every tool, so importing it statically makes **every** test that reaches the registry load
  that native module inside a vitest worker thread. The result is not "slow" — it is the
  **whole test process dying with a segfault** (0xC0000005 on Windows; individual tests pass but
  `pnpm test:run` crashes outright). Lazy-load it with `await import(...)` inside the function.
- **In `scoped` styles, `:global(...)` must be the entire selector.** Written as
  `:global([data-theme='dark']) .foo`, Vue drops everything after it and compiles to a bare
  `[data-theme='dark']` — that is `<html>` itself, so the rule hits the whole app. (This really
  happened: one `filter` turned the entire preferences screen white.) **Write the ancestor
  selector directly**; `scoped` appends `[data-v-xxx]` to the last segment, so it already matches
  only this component. Gate: `tests/unit/scoped-global-selector.test.ts`.

## 8. When you are stuck

Do not guess and do not silently narrow the task. Report what you tried, what the gate said, and
what you need decided. A half-finished PR with an honest description is more useful than a green
one that skipped the hard part.

---
> Source: [ueboxai/uebox](https://github.com/ueboxai/uebox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
