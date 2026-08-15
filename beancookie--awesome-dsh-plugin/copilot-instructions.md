## awesome-dsh-plugin

> DeepSeek Harness（`dsh`）插件的精选列表（awesome list）。两个 README 是内容唯一来源，其余文件均由它们生成。

# AGENTS.md

DeepSeek Harness（`dsh`）插件的精选列表（awesome list）。两个 README 是内容唯一来源，其余文件均由它们生成。

## 内容来源（source of truth）

- `README.md`（中文，GitHub 默认展示）与 `README.en.md`（英文）是唯一需要手动编辑内容的文件。二者必须保持同步：插件集合相同、分类相同、每个插件各占一行。
- 语言映射：`site/locales.mjs` 中 `zh` locale 的 `readme` 指向 `README.md`（中文内容）、`en` locale 指向 `README.en.md`（英文内容）——`zh` 是默认 locale（`x-default`），站点 `/` 为中文、`/en/` 为英文。改动时保持这个映射。
- `docs/`（生成的网站）、`data/npm-map.json`、`data/added-dates.json` 都是自动生成的——切勿手动编辑。
- 插件计数行由构建脚本自动改写（`README.md` 用 `**N** 个插件`、`README.en.md` 用 `**N** plugins`），不要手动改动。

## 目录结构（每个文件夹的作用）

- `site/` — 网站源码：`template.html`（页面骨架，含 CSS 样式与前端交互 JS，用 `__TOKEN__` 占位）与 `locales.mjs`（语言与文案的唯一来源：分类名、标题、界面字符串）。build-site.mjs 据此生成页面。
- `scripts/` — 构建与探测脚本：`build-site.mjs`（解析两个 README，生成站点、`data/` 产物并同步计数）与 `probe-npm.mjs`（探测各仓库对应的 npm 包，写入缓存；需联网）。
- `docs/` — 生成的网站（GitHub Pages 部署目录）：`index.html`、各分类页、`plugins.json`、`sitemap.xml`、`feed.xml`、`logo.png`、`badge.svg`。自动生成，切勿手动编辑。
- `data/` — 自动生成的账本/缓存：`npm-map.json`（repo → npm 包映射）与 `added-dates.json`（插件收录日期）。由脚本维护，切勿手动编辑。
- `images/` — 源素材：`logo.png`（站点与 README 使用的 logo 源文件）。
- `.github/` — GitHub 配置：`workflows/build-site.yml`（推送 `main` 时自动构建并提交生成文件）与 `pull_request_template.md`（PR 提交模板）。
- `.git/` — git 内部元数据，无需改动。

## 添加插件

每个插件占一行，由 `scripts/build-site.mjs` 中的严格正则解析：

```
- [owner/repo](https://github.com/owner/repo) - 以句号结尾的一句话描述。
```

- URL 必须是 `https://github.com/...`，其他域名不会被解析。
- 分类取自最近的 `###` 标题，标题必须包含 `site/locales.mjs`（`categories`）中对应的分类名。共 11 个固定分类（`ui`、`theme`、`session`、`memory`、`tools`、`skill`、`workflow`、`notify`、`model`、`dev`、`fun`）——不要新增分类，除非同时更新 `scripts/build-site.mjs` 中的 `CAT_IDS`。
- 必须在两个 README 的对应分类下各加同一行，否则 `build-site.mjs` 会报 `missing: <url>` 并失败。
- 分隔符为 ` - `（英文）或 ` — `（中文），解析器两者都接受。

## 构建 / 验证

没有 npm scripts、测试或 lint。需要 Node 22（脚本使用 ESM、`fetch`、顶层 await）：

```
node scripts/probe-npm.mjs   # 探测 npm 仓库（需联网；失败时不会改动缓存）
node scripts/build-site.mjs   # 重新生成 docs/ 与 data/，并同步 README 计数
```

先跑 probe，再跑 build。CI（`.github/workflows/build-site.yml`）在推送到 `main` 时会依次运行两者并自动提交生成的文件——所以不要手动提交 `docs/`/`data/`；注意 CI 只在 `main` 分支触发，PR 分支不会触发。

## 收录规则（见 contributing.md）

- 被收录的插件必须在 `package.json` 中声明 `dsh.bundle` manifest（仅有 `dsh.client` 不行），否则无法安装。
- 描述只说明插件功能——不加营销词/最高级措辞；以句号结尾。
- 仓库应打上 `dsh-plugin` topic。

---
> Source: [beancookie/awesome-dsh-plugin](https://github.com/beancookie/awesome-dsh-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
