## claude-code

> Core source lives in `src/`. Entry points and CLI wiring are under files such as `src/dev-entry.ts`, `src/main.tsx`, and `src/commands.ts`. Feature code is grouped by area in folders like `src/commands/`, `src/services/`, `src/components/`, `src/tools/`, and `src/utils/`. Restored or compatibility code also appears in `vendor/` and local package shims in `shims/`. There is no dedicated `test/` directory in the restored tree today; treat focused validation near the changed module as the default.

# Repository Guidelines

## Project Structure & Module Organization
Core source lives in `src/`. Entry points and CLI wiring are under files such as `src/dev-entry.ts`, `src/main.tsx`, and `src/commands.ts`. Feature code is grouped by area in folders like `src/commands/`, `src/services/`, `src/components/`, `src/tools/`, and `src/utils/`. Restored or compatibility code also appears in `vendor/` and local package shims in `shims/`. There is no dedicated `test/` directory in the restored tree today; treat focused validation near the changed module as the default.

## Build, Test, and Development Commands
Use Bun for local development.

- `bun install`: install dependencies and local shim packages.
- `bun run dev`: start the restored CLI entrypoint interactively.
- `bun run start`: alias for the development entrypoint.
- `bun run version`: verify the CLI boots and prints its version.

If you change TypeScript modules, run the relevant command above and verify the affected flow manually. This repository does not currently expose a first-class `lint` or `test` script in `package.json`.

## Coding Style & Naming Conventions
The codebase is TypeScript-first with ESM imports and `react-jsx`. Match the surrounding file style exactly: many files omit semicolons, use single quotes, and prefer descriptive camelCase for variables and functions, PascalCase for React components and manager classes, and kebab-case for command folders such as `src/commands/install-slack-app/`. Keep imports stable when comments warn against reordering. Prefer small, focused modules over broad utility dumps.

## Testing Guidelines
There is no consolidated automated test suite configured at the repository root yet. For contributor changes, use targeted runtime checks:

- boot the CLI with `bun run dev`
- smoke-test version output with `bun run version`
- exercise the specific command, service, or UI path you changed

When adding tests, place them close to the feature they cover and name them after the module or behavior under test.

## Commit & Pull Request Guidelines
Git history currently starts with a single `first commit`, so no strong conventional pattern is established. Use short, imperative commit subjects, for example `Fix MCP config normalization`. Pull requests should explain the user-visible impact, note restoration-specific tradeoffs, list validation steps, and include screenshots only for TUI/UI changes.

## Restoration Notes
This is a reconstructed source tree, not pristine upstream. Prefer minimal, auditable changes, and document any workaround added because a module was restored with fallbacks or shim behavior.



![Preview](1.png)
![Preview](2.png)
![Preview](3.png)

该项目100%开源，可直接npm发布即运行。
npx heibai 运行即可。

或直接部署源码。

前提条件：
  - Bun 1.3.5 或更高版本
  - Node.js 24 或更高版本

 安装依赖：
 ```bash
 bun install
 ```

 运行：
```bash
bun run dev
```

便捷运行：
```bash
npm install -g heibai
```

启动 （与claude启动一样）：
```bash
heibai
```

或 
```bash
npx heibai
```

注意：
无论直接运行npx heibei还是部署运行 bun run dev 都需要环境保障  Bun大于1.3.5 与 Node 大于24版本。

与官方2.1.88一样，
新增1：/model 切换heibai模型，实现本站网页实时切换Claude，GPT，Gemini，Grok等模型。
新增2：/login 实现本站登录实时切换key,登录成功后，适用于heibai，key同步官方claude终端插件。
（源码中有一个链接 https://heibai.cd.xdo.icu，这只是获取登录接口）

学习用，你可以直接拿去把我的信息改了，拿去npm发布。

本站官网：claude.hk
免费体验领取：120刀额度 约6000万token 
支持模型：Claude，GPT，Gemini，Grok。

后期会独立维护，加入类似cursor 4x能力，以及mem0记忆体。

···················································

This project is 100% open source and can be deployed and run directly via npm.

Run `npx heibai`.

Or deploy the source code directly.

Prerequisites:

- Bun 1.3.5 or higher

- Node.js 24 or higher

Install dependencies:

```bash

bun install

```
Run:

```bash

bun run dev

```
Shortcut run:

```bash

npm install -g heibai

```
startup (same as claude startup)：

```bash
heibai
```
or 
```bash
npx heibai
```

Note: Whether running `npx heibai` directly or deploying and running `bun run dev`, both require a Bun version greater than 1.3.5 and a Node.js version greater than 24.

Similar to the official 2.1.88,

New Feature 1: `/model` switches the Heibai model, enabling real-time switching between Claude, GPT, Gemini, Grok, and other models on this website.

New Feature 2: `/login` enables real-time key switching for login on this website. After successful login, the key is applicable to Heibai and synchronized with the official Claude terminal plugin.

(There is a link in the source code: https://heibai.cd.xdo.icu, which only retrieves the login interface.)

For learning purposes, you can directly modify my information and publish it via npm.

Official Website: claude.hk
Free Trial: $120 (approximately 60 million tokens)

Supported Models: Claude, GPT, Gemini, Grok

Later, it will be maintained independently, adding features similar to cursor 4x and mem0 memory.

---
> Source: [claudehk/claude-code](https://github.com/claudehk/claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
