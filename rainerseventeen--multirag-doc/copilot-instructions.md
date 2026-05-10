## multirag-doc

> 面向自动化 Agent（Codex、Copilot 等）的仓库指引。

# AGENTS.md

面向自动化 Agent（Codex、Copilot 等）的仓库指引。

## 项目简介

MultiRAG-Doc 是一个用于自学多模态 RAG 的个人研究项目，目标是构建面向科研论文的多模态检索问答系统，支持文本、图像、表格、公式的统一检索与 citation 输出。

项目架构参考 `.ai_docs/rule/project_overview.md`。

## 环境激活

执行任何项目命令前必须先激活 conda 环境：

```bash
source /Users/rainer/Dev/miniconda3/etc/profile.d/conda.sh
conda activate multirag-doc
```

## 远程开发工作流

- 远程相关配置写在项目根目录 `.env`，至少包含 `SSH_SERVER_NAME` 和 `REMOTE_REPO_DIR`。
- 如果远端登录后还需要激活 conda，可额外配置 `REMOTE_PRELUDE`。

### 远端命令执行

执行远端命令时，直接使用 ssh，每次从 `.env` 读取环境变量，不要使用 `remote/run_remote.sh`：

```bash
source .env
ssh "$SSH_SERVER_NAME" "cd $REMOTE_REPO_DIR && $REMOTE_PRELUDE && <command>"
```

### 标准流程

1. 本地修改代码。
2. 用 `bash remote/sync_to_remote.sh` 同步到远端。
3. 用上述 ssh 命令在远端执行。
4. 需要取回远端产物时，用 `bash remote/pull_from_remote.sh <remote_subpath> [local_subpath]`。

### 远程限制

- 远端工作目录固定为 `.env` 中的 `REMOTE_REPO_DIR`。
- 除非用户明确要求，不要在该目录之外执行操作。
- 本地只做 smoke test；耗时长、依赖 GPU、成本高的任务优先放到远端。
- 不要擅自启动完整训练、长时间评测或大规模实验，除非用户明确授权。

## 仓库总览

这个仓库当前已经从“按 phase 分散的实验脚本”逐步收敛为“`src/` 下的正式实现 + `experiments/` 下的验证脚本”两条线：

- `src/` 是主开发区，承载正式的 ingest / query pipeline、索引、检索、生成与评测逻辑。
- `experiments/` 是阶段性验证脚本、消融实验、demo 和调参脚本，一般用于研究与复现实验，不是功能开发的首选落点。
- `database/` 存放输入 PDF、落盘 chunk、FAISS 索引、图像索引和测试集。
- `docs/materials/` 存放各阶段学习笔记与 TODO。
- `libs/DocsGPT/`、`libs/paper-qa/` 是参考/上游代码，默认视为 vendor 或对照实现。

## 实际代码结构

### 主入口

稳定入口是 `python -m src.cli`，当前包含三个主要子命令：

- `ingest`：单篇论文入库。
- `ingest-all`：批量入库，可选择普通模式或 staged ingest。
- `query`：检索，可选生成带 citation 的答案。

### `src/` 模块分层

按职责看，主代码大致分为下面几层：

| 目录 | 主要职责 |
|------|-----------|
| `src/cli.py` | CLI 入口，解析参数并调度 pipeline |
| `src/config.py` | 从 `config.yml` 读取全局配置单例 `CFG` |
| `src/pipeline/` | 编排主流程，包含 `ingest.py`、`query.py`、`staged_ingest.py` |
| `src/ingestion/` | PDF 解析、文本/多模态分块、caption、embedding |
| `src/index/` | 文本与图像索引持久化依赖的底层存储抽象 |
| `src/retrieval/` | 文本检索、图像检索、融合、rerank、多文档排序 |
| `src/generator/` | prompt 构造、LLM 调用、答案格式化 |
| `src/evaluation/` | citation 检查、guardrail、召回指标 |

### 主流程

1. `src.cli` 读取命令行参数。
2. `src.pipeline.ingest` 或 `src.pipeline.query` 负责主链路编排。
3. ingest 侧调用 `src.ingestion.*` 解析 PDF、构造 chunk、生成 embedding，再通过 `src.index.*` 写入索引与元数据。
4. query 侧调用 `src.retrieval.*` 做文本检索、多文档排序、图像融合与 rerank。
5. 如果启用生成，`src.generator.*` 负责提示词、LLM 请求、答案格式化；`src.evaluation.*` 负责 guardrail 与 citation 校验。

### 数据落盘位置

默认路径由 `config.yml` 控制，当前主要落盘位置为：

- `database/pdf/`：输入论文 PDF。
- `database/chunks/`：每篇论文 chunk 文件，以及聚合后的 `all_chunks.json`。
- `database/index/text_index.faiss`：文本索引。
- `database/index/image_index.faiss` 与 `image_metadata.json`：图像索引与元数据。
- `database/index/figures/<paper_id>/`：抽取出来的 figure 图片。
- `database/staging/<paper_id>/`：staged ingest 的阶段中间产物。

## 修改边界

这是本仓库最重要的默认规则：

- 新功能、正式能力扩展、可复用逻辑重构，优先落在 `src/`。
- `experiments/` 里的代码一般不作为功能开发主战场，默认不修改，除非用户明确要求修实验脚本，或某个实验脚本本身就是本次任务目标。
- 如果实验中验证出的能力需要“产品化/正式化”，应把实现迁移或沉淀到 `src/`，而不是继续堆在 `experiments/`。
- `libs/DocsGPT/` 和 `libs/paper-qa/` 默认只作为参考，不应随手改动；只有用户明确要求或任务必须修改 vendor 代码时才动。

### 对 `experiments/` 的具体约束

- 可以阅读、运行、参考，但不要默认在其中实现 `feat`。
- 可以在必要时新增很小的 demo 或验证脚本，但前提是 `src/` 中已有对应正式实现，或用户明确要求先做实验验证。
- 若某个 bug 同时出现在 `src/` 和实验脚本，优先修 `src/`；只有在保持实验可运行确有必要时，才同步修 `experiments/`。

### 对 `src/` 的默认落点建议

- 入口和参数改动：优先改 `src/cli.py`。
- ingest 流程改动：优先改 `src/pipeline/` 与 `src/ingestion/`。
- 索引和元数据结构改动：优先改 `src/index/`。
- 检索、融合、rerank、多文档排序改动：优先改 `src/retrieval/`。
- 回答生成、citation 输出、拒答逻辑改动：优先改 `src/generator/` 和 `src/evaluation/`。

## Agent 工作准则

- 改动前先判断目标属于“正式能力”还是“实验验证”。
- 如果用户说“加一个 feat”或“实现一个能力”，默认理解为改 `src/`，不是改 `experiments/`。
- 如果只是为了验证论文想法、跑 ablation、做一次性脚本，才优先考虑 `experiments/`。
- 修改 `config.yml`、`database/` 路径约定、chunk schema、索引格式时，要检查是否影响已有 ingest 数据和 query 流程。
- 修改 ingest / query 主链路时，要优先保持 CLI 用法稳定，除非用户明确要求改接口。

## 目录速览

| 路径 | 说明 |
|------|------|
| `src/` | 正式实现代码，所有 `feat` 默认落在这里 |
| `experiments/00_data_prep/` | 数据准备与模型下载脚本 |
| `experiments/01_text_retrieval/` 到 `experiments/08_multi_paper_ranking/` | 分阶段 demo、消融实验与验证脚本 |
| `experiments/caption_tuning/` | caption 调参实验 |
| `database/` | 输入 PDF、索引、chunks、testset、staging 中间产物 |
| `docs/materials/` | 每周学习笔记、TODO、实验计划 |
| `.ai_docs/rule/` | 参考规范与文件架构说明 |
| `remote/` | 本地与远端同步脚本 |
| `libs/DocsGPT/` | 上游参考项目，默认不改 |
| `libs/paper-qa/` | 上游参考项目，默认不改 |

## 代码规范

- 详细规范见 `./.ai_docs/rule/code_rules.md`
- Python 遵循 PEP 8，4 空格缩进，公共函数加 type hint
- 文件/函数/变量用 `snake_case`，常量用 `UPPER_CASE`
- 实验脚本保持确定性，使用显式路径、固定随机种子
- 公共逻辑不要散落复制到多个实验脚本，优先抽到 `src/`

## Commit 规范

遵循 Conventional Commits：`feat:`、`refactor:`、`docs:`、`init:`。

若改动只涉及文档或 Agent 指引，优先使用 `docs:`。

---
> Source: [RainerSeventeen/MultiRAG-Doc](https://github.com/RainerSeventeen/MultiRAG-Doc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
