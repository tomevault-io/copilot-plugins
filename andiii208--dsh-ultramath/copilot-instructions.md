## dsh-ultramath

> > 本文件是 dsh-ultramath 插件包自身的开发约定。领域资产（模型库/角色指令/论文模板）内部维护在作者的 UltraMath 私密工作区，本插件包是其公开发布镜像——**插件包本身完全自包含**，用户安装使用无需访问任何外部仓库。

# dsh-ultramath 插件开发约定

> 本文件是 dsh-ultramath 插件包自身的开发约定。领域资产（模型库/角色指令/论文模板）内部维护在作者的 UltraMath 私密工作区，本插件包是其公开发布镜像——**插件包本身完全自包含**，用户安装使用无需访问任何外部仓库。

## 单一权威源（Single Source of Truth）

- 领域资产的**唯一权威源**是作者的 UltraMath 私密工作区（不对外公开）；插件包内的 `skills/模型库/`、`presets/*/agent.cordis.yml`、`templates/论文/` 均为其发布镜像。
- 对外发布时，插件包内所有领域资产必须**字节级一致、自包含可运行**：模型库 33 篇、5 个预设、论文模板与审稿脚本全部随包分发。
- 修改领域资产：先改内部工作区权威源，再同步进插件包；禁止在插件包内直接改模型库正文或角色指令（会破坏单一权威源）。
- **运行期装载路径（关键）**：本机运行中的 DSH 经 `~/.dsh/profiles/<profile>/node_modules/dsh-ultramath` 的 Junction 装载**内部工作区**（而非本公开仓库）。因此**运行期 bug（预设切换失败、技能不可见等）必须同时改三处**：①内部工作区（Junction 目标，DSH 实际加载）②本公开仓库 ③磁盘同步目标 `~/.dsh/.agent-presets/`；只改本仓库或磁盘，DSH 重启后会从内部工作区的旧文件覆盖回磁盘，修复被冲掉（v0.6.3 的 `sampleOverCapGlobResults` 必填字段修复即踩此坑）。结构性配置修复（非模型库正文/角色指令，如 DSH 主机 schema 升级导致的必填字段补齐）同样遵循三处同步；定位 Junction 目标可用 `dev_plugin_status` 或读 `<profile>/node_modules/dsh-ultramath` 的链接目标。
- 论文模板（`templates/论文/` 的 `.tex/.cls` 与 `merge_tex.py`，不含 fonts）与确定性审稿脚本（`scripts/check_*.py`、`scripts/review_loop/*`）自 v0.5.0 起随包分发；fonts 与 `figure_style.py` 不进包，由内部工作区维护（插件运行时字体走降级链，无需 fonts）。

## 形态与合规

- 双形态共存：`dsh.plugin.json`（registry 元数据；plugin_check 实测判定 kind=bundle，即 registry 元数据仅作 hub 目录收录、运行时装载走 `cordis.patch.yml` 的 bundle 机制）+ `package.json` 的 `main` + `dsh.bundle.patch` → `cordis.patch.yml` 的 `- insert:` 行（id=ultramath，避开官方核心 row）作为运行时装载机制（node 半侧经此路径把 presets 幂等同步到 ~/.dsh/.agent-presets/）。
- registry 元数据辅助：`dsh.plugin.json` 声明 `contributes.skills=["ultramath"]`，供 hub 目录收录与 plugin_check 校验。
- presets 同步只写 `~/.dsh/.agent-presets/{ultramath, ultramath-*}` 自有目录，幂等（字节相同跳过），并校验 `agent.cordis.yml` 结构（缺 name / name 前缀非法 / 重复 id 视为失败）。
- skills 同步只写 `~/.dsh/skills/ultramath/`（根 `SKILL.md` + `模型库/` + `templates/` + `scripts/` 随包审稿脚本，v0.6.1 起含 scripts/），幂等并校验 SKILL.md frontmatter/必要章节；`~/.dsh/skills` 是 DSH skill-filesystem 的用户根，不要依赖 npm 包内 `skills/` 被自动扫描。
- 不改 DSH 核心源码。

## 开发检查

- `plugin_check check <本插件包路径>` 必须 `verdict=pass`；实测判定 `kind=bundle`（`dsh.plugin.json` 的 registry 元数据仅作 hub 目录收录，运行时装载走 `cordis.patch.yml` 的 bundle 机制）。当前 plugin_check 要求 `types`/`peerDependencies`/`files` 字段（v0.5.0 起已补齐）。
- 发版前必须运行 `node scripts/sync-standalone.mjs --check` 通过（在插件包权威源副本下，确保公开仓与内部副本字节一致）。
- `dsh --profile web --dump-config | grep ultramath` 确认 row-id 已注入。
- 发布前 README 必须含 `dsh plugin --profile web add …` 示例。

## 目录约定

- 中文文件名与注释保持（与内部工作区一致）。
- `lib/index.js` 为纯 ESM，运行期依赖仅 schemastery（Config schema）。
- skills 由 `lib/index.js` 在 apply 时幂等同步到 `~/.dsh/skills/ultramath/`（根 `SKILL.md` + `模型库/`，v0.4.0 已实现；templates/ 由 v0.5.0 起一并同步；scripts/ 随包审稿脚本由 v0.6.1 起一并同步），DSH 的 `dsh-skill-filesystem` 会从 `~/.dsh/skills` 发现；不要依赖 npm 包内 `skills/` 被自动扫描。
- Windows 兼容：`lib/index.js` 的文件拷贝/删除一律用逐文件实现（`copyTree`/`removeTree`/`unlinkFile`），禁止用 `cpSync(recursive)`/`rmSync(recursive)` 处理含中文路径的目录（GitHub Issue #1：部分 Windows 环境下会硬崩溃进程）。
- 5 个新增技能（出版级图表规范 / LaTeX排版与编译 / 凸优化与非线性规划 / 全局敏感性分析 / 启发式优化）为 t2 调研新增，同样以内部工作区模型库为回写目标。

---
> Source: [Andiii208/dsh-ultramath](https://github.com/Andiii208/dsh-ultramath) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
