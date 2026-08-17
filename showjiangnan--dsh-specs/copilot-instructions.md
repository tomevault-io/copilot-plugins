## dsh-specs

> 本仓库是供 Codex、Claude Code 等智能编码工具读取的 DeepSeek Harness 独立文档与规格快照，不包含产品源码。开始分析或编辑前先阅读 [README.md](README.md)、[智能编码工具接管指南](docs/开发/智能编码工具接管指南.zh.md)、[ARCHITECTURE.md](ARCHITECTURE.md)、[文档导航](docs/文档导航.zh.md)和[来源与同步](来源与同步.md)。

# AGENTS.md

本仓库是供 Codex、Claude Code 等智能编码工具读取的 DeepSeek Harness 独立文档与规格快照，不包含产品源码。开始分析或编辑前先阅读 [README.md](README.md)、[智能编码工具接管指南](docs/开发/智能编码工具接管指南.zh.md)、[ARCHITECTURE.md](ARCHITECTURE.md)、[文档导航](docs/文档导航.zh.md)和[来源与同步](来源与同步.md)。

## 智能编码任务

- 把本仓库用于定位产品边界、模块归属、扩展点、术语、质量要求和来源证据；不要把它当作可运行的 DSH checkout。
- 开始实现前读取目标 DSH 源码 checkout 的 `AGENTS.md`，记录其 commit，并在代码、测试和配置中核对与任务相关的文档结论。
- 区分代码事实、文档声明和推断。重要结论引用具体路径，以及函数、类、类型、事件、包或配置键。
- 按任务加载最小上下文，不要一次读取全部生成目录。先读入口文档，再进入所属子系统。
- 构建、测试、类型检查、生成 freshness 和运行验证只在目标 DSH 源码 checkout 中执行；本仓库的绿色文档检查不能替代它们。

## 文档归属

- `docs/产品/` 负责产品目的、用户、规格、术语和假设。
- `docs/架构/` 负责跨模块结构、依赖方向、运行流程与决策导航。
- `docs/前端/` 和 `docs/后端/` 负责对应实现面的当前状态说明。
- `docs/质量/` 负责测试、安全、可靠性、防御性模式和事故复盘。
- `docs/使用指南/` 与 `docs/开发/` 分别负责产品任务和二次开发流程。
- `docs/参考/` 负责子系统、框架接口和生成目录。
- `docs/规划/` 只保存有边界的进行中计划和仍有维护价值的完成记录。
- `.agents/notes/` 保存本仓库信息架构自身的决策理由。

每项事实只有一个归属文档；其他页面只保留读者需要的摘要和链接。不要把教程、参考目录、计划和决策理由混在同一页面。

## 命名与双语

- `docs/` 下每个目录和 Markdown 语义主文件名必须包含简体中文。
- `中文语义名.md` 保存英文内容，`中文语义名.zh.md` 保存简体中文内容，`中文语义名.i18n.yaml` 保存两侧确认一致时的 Git blob hash。
- 修改任一语言时同步更新对侧，并重新记录 sidecar。
- 文本段落一段一行；不要手工折行。表格、列表和代码围栏保持标准 Markdown 结构。
- 首次使用项目术语时，用一句普通语言解释，然后保持术语一致。

## 独立仓库边界

源码、包 README、测试、生成器和完整 Agent Note 历史仍归上游 DeepSeek Harness 仓库。指向这些证据的链接固定到[来源与同步](来源与同步.md)记录的公开上游基线；不要改成指向本仓库中并不存在或含义不同的相对路径。

`docs/参考/生成/`、`docs/参考/框架接口/`中的生成页，以及子系统页中的生成区域都是上游生成器产物。更新它们时必须先在匹配的上游源码 checkout 中运行生成器，再把结果同步到本仓库；禁止在本仓库手工修改生成内容。

## 验证

提交前运行：

```sh
npm run docs:check
git diff --check
```

检查必须证明中文路径、相对链接、Markdown 锚点、双语结构和 sidecar hash 一致。本仓库不安装运行时依赖，也不声称替代上游的构建、测试、类型检查、生成 freshness 或 VitePress 门禁。

---
> Source: [showjiangnan/dsh-specs](https://github.com/showjiangnan/dsh-specs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
