## momojee-github-io

> > 本文件是项目文档系统的**唯一入口（索引）**。任何开发者、AI 助手或维护者在动代码之前，请先阅读本文件并按需跳转到对应文档。

# AGENTS.md — MoMoJee 个人主页项目索引

> 本文件是项目文档系统的**唯一入口（索引）**。任何开发者、AI 助手或维护者在动代码之前，请先阅读本文件并按需跳转到对应文档。

## 项目一句话

Vue 3 + Vite + TypeScript 构建的**纯静态个人主页**，内容与展示分离、数据驱动渲染，通过 GitHub Actions 自动构建并部署到 GitHub Pages 及多个镜像域名。

## 新人入场 60 秒

1. 跑一遍下面"快速开始"，本地能看到页面。
2. 通读 [docs/architecture.md](docs/architecture.md)（系统怎么运转）+ [docs/development.md](docs/development.md)（怎么改代码）。
3. 记住三条铁律：**改内容只动 `profile.ts`；改文档必须与代码同步；`npm run docs:check` 与 `npm run build` 双通过才允许推 main**。

## 快速开始

```bash
npm install        # 安装依赖
npm run dev        # 本地开发（Vite Dev Server）
npm run build      # 类型检查 + 构建（vue-tsc --noEmit && vite build）
npm run docs:check # 校验文档系统一致性（推 main 前必跑，CI 也会跑）
npm run preview    # 本地预览构建产物
```

## 目录结构速览

```
.
├── AGENTS.md                 # ← 文档索引（本文件，唯一入口）
├── docs/                     # 全部文档（每份带元信息头，改动代码须同步）
│   ├── README.md             # 文档导航
│   ├── documentation.md      # ★ 文档维护手册（怎么保持文档不腐烂）
│   ├── architecture.md       # 架构与数据流
│   ├── development.md        # 开发与工程约定
│   ├── content.md            # 站点内容维护指南
│   ├── deployment.md         # 部署、域名与镜像同步
│   ├── operations.md         # 运维手册与排障
│   └── evolution.md          # 升级与改造路线图（含 ADR 决策记录）
├── scripts/
│   └── docs-check.mjs        # ★ 文档校验器（npm run docs:check）
├── src/
│   ├── App.vue               # 唯一页面级组件，组织所有区块；持有语言/主题状态 + GSAP 动效
│   ├── main.ts               # 入口，挂载 App
│   ├── style.css             # 全部样式（纯 CSS，:root 浅色 / [data-theme=dark] 深色变量主题）
│   ├── data/profile.ts       # ★ 站点内容数据源（双语 Localized，改内容改这里）
│   ├── composables/          # 有状态逻辑：useTheme（☀/🌙）、useI18n（中/EN）、useGithub（动态拉取 GitHub 数据）
│   └── components/           # 展示型组件（只负责渲染，不持有数据）
├── public/                   # favicon.svg / og-cover.svg（原样拷贝）
├── .github/workflows/        # deploy.yml（含 docs:check 门禁）+ sync-mirrors.yml
├── index.html                # HTML 入口与 SEO meta
├── CNAME                     # GitHub Pages 自定义域名：momojee.cn
└── vite.config.ts / tsconfig.json / package.json
```

## 核心约定（改代码前必读）

1. **内容与代码分离**：所有页面文案、项目、技能、雷达数据都在 `src/data/profile.ts`，改内容**不要**动 Vue 组件。文案支持中英双语（`Localized`），界面文字统一集中在 `ui`。详见 `docs/content.md`。
2. **组件只渲染不持数据**：组件通过 props 接收数据，数据统一由 `App.vue` 从 `profile.ts` 注入；有状态逻辑放 `src/composables/`（`useTheme` / `useI18n`），仅 App.vue 层使用。
3. **样式纯 CSS**：不使用 UI 框架与 CSS 预处理器。主题走 `style.css` 里的 CSS 变量（`--accent` / `--accent-2` / `--bg` / `--text` 等），浅色在 `:root`、深色在 `[data-theme="dark"]`，页面右上角可手动切换 ☀/🌙（持久化到 localStorage），改色先改变量。动效用 GSAP，须尊重 `prefers-reduced-motion`。
4. **无测试、无 lint**：本项目没有配置 test 与 lint 脚本，代码交付标准 = `npm run build` 类型检查通过。
5. **主分支直推**：`main` 分支即发布分支，push 即触发部署，**请勿直接向 main 推送未经 `npm run build` 验证的改动**。
6. **文档即代码（活文档）**：任何代码改动必须在同一提交内同步相关文档；`npm run docs:check` 通过是硬性门禁（本地 + CI），目的是让每个后来者看到的都是最新、完整、无失效链接的文档。

## 文档索引

| 文档 | 内容 | 何时看 |
| --- | --- | --- |
| [docs/README.md](docs/README.md) | 文档导航与阅读顺序 | 快速定位该读哪篇时 |
| [docs/documentation.md](docs/documentation.md) | 文档维护手册：如何保持文档不腐烂、变更→文档映射表、校验器说明 | 任何改动前必读（维护纪律） |
| [docs/architecture.md](docs/architecture.md) | 整体架构、数据流、组件职责、技术选型 | 需要理解系统如何运转时 |
| [docs/development.md](docs/development.md) | 开发环境、构建流程、工程约定、目录职责 | 开始写代码 / 改代码前 |
| [docs/content.md](docs/content.md) | profile.ts 数据模型与每个区块的维护方法 | 只改文案、项目、数据时 |
| [docs/deployment.md](docs/deployment.md) | 部署流水线、镜像仓库、域名与 DNS、密钥 | 发布、加域名、加镜像时 |
| [docs/operations.md](docs/operations.md) | 日常运维、发布清单、排障手册 | 线上出问题、日常巡检时 |
| [docs/evolution.md](docs/evolution.md) | 已知局限、升级方向、决策记录（ADR） | 规划改造 / 升级时 |

## 常用命令

| 命令 | 作用 |
| --- | --- |
| `npm run dev` | 启动本地开发服务器 |
| `npm run build` | 类型检查 + 生产构建到 `dist/` |
| `npm run preview` | 预览 `dist/` 构建产物 |
| `npm run docs:check` | 校验文档系统一致性（链接/索引/元信息/数据覆盖），**改任何东西后必跑** |
| `npm ci` | CI 中使用，按 lockfile 精确安装 |

## 文档维护纪律（每个改代码的人都要遵守）

**核心规则：文档与代码同源、同步、同生命周期。`npm run docs:check` 是推 main 前不可跳过的校验。**

1. **改了什么，就同步哪份文档**：对照 [docs/documentation.md](docs/documentation.md) 第 4 节的"变更→文档映射表"更新对应文档，并刷新其元信息头的 `updated` 日期。
2. **新增脚本必须在 `AGENTS.md` 与 `docs/development.md` 的命令表登记**，否则 `docs:check` 会失败（命令覆盖检查）。
3. **新增 `profile.ts` 导出必须在 `docs/content.md` 记录**，否则 `docs:check` 会失败（数据覆盖检查）。
4. **新增/删除文档必须登记索引**：在 `AGENTS.md` 文档索引表与 `docs/README.md` 各加/删一行，否则 `docs:check` 会失败（索引同步检查）。
5. **结构性/依赖/CI 决策先写 [docs/evolution.md](docs/evolution.md)**（ADR + 变更登记），再动代码。
6. 部署/域名/密钥变更必须同步更新 `docs/deployment.md` 与 `docs/operations.md` 资产清单。

> 详细规则、映射表与校验器说明见 [docs/documentation.md](docs/documentation.md)。

## 变更纪律

- 每次改动遵循**单一职责**：改内容 → `profile.ts`；改样式 → `style.css`；改结构 → 对应组件。
- **完成的定义（DoD）**：代码改动 + 对应文档同步 + `npm run docs:check` 通过 + `npm run build` 通过，四者缺一不可。
- 涉及新增依赖、路由、页面结构、CI/CD 时，先在 `docs/evolution.md` 记录决策，再动代码。
- 部署/域名/密钥变更必须同步更新 `docs/deployment.md`。

---
> Source: [MoMoJee/MoMoJee.github.io](https://github.com/MoMoJee/MoMoJee.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
