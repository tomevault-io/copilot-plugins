## dsh-live2d-pets

> DSH 的 Live2D 桌宠插件项目（文档驱动开发）。

# dsh-live2d-pets

DSH 的 Live2D 桌宠插件项目（文档驱动开发）。

## 必读文档

| 需求 | 文档 |
|------|------|
| 产品意图 | docs/intent/ |
| 行为规格 | docs/spec/ |
| 架构决策 | docs/adr/ |

改用户可感知行为 → 同步 spec；改产品意图/范围 → 同步 intent；架构/技术取舍 → 新增或更新 ADR。
纯样式/文案/重构且行为不变可不写文档；若已有 spec 写到相关细节则顺手改。
意图不清时先澄清再落文档，禁止先写代码再补文档。

## 工程约定

- 依赖安装统一用 **bun**（`bun install`），**不要用 npm**（npm 在本机缓存目录有权限问题）。
- 打包统一用 **`bun pm pack --destination dist`**（自动跑 prepack/prepare 构建）。**不是 `bun pack`**——当前 bun 1.3.x 没有 pack 内置子命令，会报 `Script not found "pack"`；也**不要用 `npm pack`**——会撞 `G:\Cache\npm\_cacache` 的权限问题（即使 `--cache` 重定向到工作区也只是绕道，非首选）。
- 本环境（DSH 沙盒）下 bun 写系统临时目录会被拒：安装时把 `BUN_TMPDIR`/`TMP`/`TEMP` 指向工作区 `.bun-tmp`，`BUN_INSTALL` 指向工作区 `.bun-home`（两者已 gitignore，装完可删）。
- 本环境（DSH 沙盒）禁止创建子进程（named-pipe 限制），vitest 默认 forks pool 会 `spawn EPERM`：`test` 脚本固定 `--pool=threads`（worker_threads 不 spawn 子进程），不要改成 forks，也不要加会被 vite 打包的 vitest.config.*（其配置加载阶段同样 spawn）。

## 发版流程

用户说「发版」「发布新版本」时按此编排执行；其中**发版说明一环调用全局 `release-notes` skill**（该 skill 只管说明，不由「发版」直接触发）：

1. 前置：工作区干净（有未提交改动让用户先 commit/stash，勿代提交无关改动）。
2. 版本类型从待发提交推断（含 `feat` → minor，仅 `fix`/`refactor` → patch，破坏性 `!` → major），**发版不可逆，须用户确认后继续**。
3. bump `package.json` 版本并提交 `chore: release vX.Y.Z`。
4. `bun pm pack --destination dist` 验证打包完整（会跑 prepack 构建）。
5. 打 **annotated** tag 并推送：`git tag -a vX.Y.Z -m "Release vX.Y.Z"` + `git push --follow-tags`（轻量 tag 不会被 `--follow-tags` 推送，勿用）。
6. **手动 `bun publish`——助手禁止代跑**（npm 鉴权常需浏览器跳转）；若默认 registry 是镜像，显式 `--registry https://registry.npmjs.org/`。
7. 发版说明：调用 `release-notes` skill（起草 → 用户确认 → `gh release create`，最新版不加 `--latest=false`）。
8. 验证：`gh release view vX.Y.Z`、`npm view dsh-live2d-pets version`。

---
> Source: [cyanfish-x/dsh-live2d-pets](https://github.com/cyanfish-x/dsh-live2d-pets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
