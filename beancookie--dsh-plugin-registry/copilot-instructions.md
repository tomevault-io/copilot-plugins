## dsh-plugin-registry

> 一个 DeepSeek Harness（DSH）插件：从 `awesome-dsh-plugin` 精选 registry 检索并安装插件。TypeScript，ESM。

# AGENTS.md

一个 DeepSeek Harness（DSH）插件：从 `awesome-dsh-plugin` 精选 registry 检索并安装插件。TypeScript，ESM。

## 命令

- `pnpm install`
- `pnpm build` — 唯一的构建步骤：`tsc` 编译 `src/` → `lib/`，再由 `scripts/build-client.mjs` 打包 `client/` → `lib/client.js`。无 test / lint / typecheck 脚本。

## 架构

- `src/*.ts` 是源码；`lib/*.js` 是**由 `tsc` 生成但被提交**的产物，也是包的 `main`。切勿直接编辑 `lib/` —— 先改 `src/` 再执行 `pnpm build`。
- 模块体系为 ESM + `moduleResolution: NodeNext`，因此相对导入**必须带 `.js` 后缀**（例如 `import { loadRegistry } from './registry.js'`）。
- `src/index.ts` — 插件的 `apply(ctx)`；`inject = ['tools', 'loader']`。创建唯一一个 `ProfileStore`，供 HTTP API 与 DSH 工具共享（安装状态来自单一事实源），然后注册工具，并通过 `ctx.inject(['webServer'], …)` 挂载仅 Web 使用的 HTTP 路由。
- `src/registry.ts` — 拉取/缓存远程 `plugins.json`（1 小时 TTL、4 秒超时），失败回退 `data/registry-snapshot.json`；`searchPlugins`（按 `name`/`owner`/description 检索，`sort` 参数支持 `relevance`/`stars`/`recent` 三种排序，默认相关度评分）、`installSpec`（把 `install` 归一化为 pnpm spec，含 `github:owner/repo#path:/sub`）、`describe`、`resolvePlugin`（按 URL → name → `owner/repo` 把用户 ref 解析为唯一条目，遇到歧义时报错）。
- `src/profile.ts` — `ProfileStore`：发现当前 profile 目录（loader 的 `cordis:include` 配置路径，回退 `resolveProfileDir('web')`），通过 `@deepseek-ai/dsh-app-boot` 读写 profile 清单，执行 `corepack pnpm add/remove`，对账 `dsh.profile.bundles`（镜像 `dsh plugin add/remove`），并**仅依据清单**（`dependencies` + `dsh.profile.bundles`）判定安装状态，从不依赖 loader/HMR。含 `install`/`uninstall` 及 spec 归一化辅助函数。
- `src/tools.ts` — 通过 `@deepseek-ai/dsh-tools` 的 `defineTool` 注册两个 DSH 工具：`search_dsh_plugins`（检索 + 安装状态，支持 `sort` 参数）与 `install_dsh_plugin`（解析 + `profile.install`）。
- `src/api.ts` — 在 `ctx.webServer` 上注册单个 `prefix` 路由 `/api/dsh-plugin-registry`：`GET …/search`（支持 `sort=relevance|stars|recent`）、`GET …/installed`、`POST …/install`、`POST …/uninstall`，复用 `registry.ts`/`market.ts`/`profile.ts`。
- `src/market.ts` — `searchHits`：检索 + 附加服务端算好的安装状态（含 `stars`/`createdAt`）；HTTP API 与工具共用的同一视图。
- `src/types.ts` — 共享类型（`RegistryPlugin`、`SearchHit`、`InstalledEntry`、`InstallResult`、`UninstallResult`）。
- `client/client.tsx` — 浏览器 bundle 源码：在 `settings.plugins.tab` 槽位（id `market`）注册的 React「插件市场」页面。搜索 + 排序下拉（相关度/按 Star/按最新，默认按 Star 降序）+ 分类过滤 + 已安装状态过滤 + 安装/卸载（带确认弹窗），全部通过 `fetch` 调 `/api/dsh-plugin-registry/*` 路由。由 esbuild 打包，**不是** `tsc`；无类型检查（仅 esbuild）。`react`/`react/jsx-runtime` 由 DSH web shell 提供的外部依赖。
- `data/registry-snapshot.json` — 随包分发的离线回退（336 个插件、11 个分类，含 `stars`/`createdAt`）。若过期，从 `src/registry.ts` 中的远程 URL 刷新。

## 注意事项

- 同伴依赖 `@deepseek-ai/cordis`、`@deepseek-ai/dsh-tools`、`@deepseek-ai/dsh-app-boot`、`@deepseek-ai/dsh-home-paths` 是内部 scoped 包；不要把它们作为普通依赖添加（`react` 也是 peer）。本地构建时它们在 `devDependencies` 中重复声明。
- `profile.ts` 有 Windows 专用的 `pnpm` 执行路径（通过 `node` 运行 `corepack/dist/corepack.js`，回退 `cmd.exe /c corepack pnpm …`，因为 `.cmd` shim 无法被 `execFile`）；改动安装流程时请保留。`stdio` 为 `['ignore', 'pipe', 'pipe']`，避免非交互 pnpm 下 stdin 阻塞。
- `dsh` CLI 不是本地依赖（`.bin` 里只有 `tsc`/`cordis`）；开发加载用 `dsh plugin --profile web add .` 再 `dsh web`。
- `dev.cordis.patch.yml` 被 gitignore（开发模式补丁）；`cordis.patch.yml` 是已提交的 bundle 补丁（一条 `dsh.bundle` insert），用于启用本插件自身。
- 在 `--patch` 覆盖层中，相对 entry 的 `name` 解析是相对于 **profile 目录**（`$DSH_HOME/profiles/<name>`），而非补丁文件或 cwd。关键点：`--patch`（即便是绝对 `file:///` URL）只会加载**宿主侧** —— Web UI 的客户端 bundle 由 `dsh-client-modules` 按**包名**解析 entry 的包（`require.resolve(name + '/package.json')`）来发现，所以必须按名安装。开发 UI 用 `pnpm build && dsh plugin --profile web add . && dsh web`。
- Web UI 需要 `package.json` 中的 `dsh.client` 清单 + `exports["./client"]`；bundle 必须通过 `window.__ModuleLoader__.load({ id: <包名>, factory })` 自注册。`scripts/build-client.mjs` 用 esbuild 生成这层包装 —— 不要手动编辑 `lib/client.js`。
- `src/api.ts` 的路由注册在 `ctx.inject(['webServer'], …)` 之下，因此无头 profile（无 `webServer`）仍能工作；该子 fiber 保持 PENDING 即可。
- `pnpm-workspace.yaml` 设置 `allowBuilds: { esbuild: true }` —— 在 pnpm 11 下必需，否则 esbuild 的 postinstall 被跳过，`pnpm build` 会因依赖检查失败。
- 安装状态完全取自持久化的 profile 清单（`dependencies` + `dsh.profile.bundles`），刻意忽略 loader/HMR —— 以保证「已安装」徽标与卸载行为一致（仅存在于 loader、未持久化的条目不应显示为已安装）。

---
> Source: [beancookie/dsh-plugin-registry](https://github.com/beancookie/dsh-plugin-registry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
