## literature-downloader-skill

> 中文文献检索、筛选、批量采集和合法全文获取助手。用于用户需要查找论文、下载可合法获取的 PDF/HTML/XML 全文、生成关键词和检索式、查询 DOI/PMID、筛选高影响因子或高分区期刊、检查开放获取、做引用链扩展、批量文献候选表、下载日志、去重清单、Zotero/BibTeX 整理，或解决文献难下载问题；禁止绕过付费墙、盗版下载、共享账号或规避版权限制。


# 中文文献下载与采集助手

## 基本原则

用中文和用户协作，把“找文献、筛文献、判断期刊质量、获取合法全文、整理记录”做成一条可复现流程。

不要提供或推荐盗版、绕过付费墙、共享学校账号、破解 DRM、影子图书馆等做法。遇到无法直接下载的文献，改用合法路线：出版社页面、开放获取版本、PubMed Central、Europe PMC、Unpaywall、预印本、机构仓储、作者主页、学校图书馆、馆际互借、邮件联系作者。

影响因子、JCR 分区、Scopus/SJR、收录状态、撤稿/更正、全文可用性都会变化。只要用户要“最新”“高影响因子”“近五年”“能下载全文”，先核对来源，再给结论。

## 普通检索流程

1. 明确需求：主题、学科、年份、目标篇数、文献类型、语言、是否必须全文、是否要求高影响因子/分区/SCI/SSCI/EI/PubMed/Scopus。
2. 拆关键词：把自然语言题目拆成 2-5 个概念块，补同义词、缩写、英文表达、中文表达、控制词。详细方法见 `references/search-strategy.md`。
3. 分层检索：
   - 广泛发现：Google Scholar、Semantic Scholar、OpenAlex、Crossref、Lens、Dimensions。
   - 学科数据库：PubMed/Europe PMC、IEEE Xplore、ACM DL、Web of Science、Scopus、CNKI、万方、维普等。
   - 全文入口：DOI 页面、出版社页面、PubMed Central、Unpaywall、机构仓储、预印本、作者主页、合法的 ResearchGate 作者上传版本。
4. 筛选质量：相关性、年份、期刊/会议质量、影响因子或分区、引用、文章类型、是否同行评议、是否撤稿、更正或有利益冲突。详细规则见 `references/source-quality.md`。
5. 合法获取全文：按 `references/full-text-access.md` 的顺序找 PDF/HTML/XML 或记录下一步合法获取方式。
6. 整理结果：按 DOI/PMID/题名去重，保存引用信息、全文来源、访问状态、PDF 文件名、Zotero/BibTeX 信息。整理规则见 `references/organization.md`。

## 一键生成检索计划

先用 `scripts/litplan.py` 生成中文可读的检索入口、Boolean 检索式和追踪表：

```bash
python scripts/litplan.py --topic "LLM 在环境领域中的应用" --keywords "large language model|LLM|ChatGPT,environmental science|climate|sustainability,application|decision support" --years 2021-2026
python scripts/litplan.py --doi "10.1038/s41545-025-00509-8"
python scripts/litplan.py --title "ChatGPT and Environmental Research"
```

这个脚本不绕过付费墙，也不盗版下载。它只生成合法检索路线和可复现记录表。

## 其他 Agent 使用方式

Codex 可以原生识别本 skill，并使用 `Use $literature-downloader ...` 触发。Claude Code、Trae、CodeBuddy、Cursor 等工具不一定识别 Codex Skill 自动触发机制，但仍然可以把本仓库当作“agent 指令文件 + 脚本工具包”使用。

给其他 agent 的推荐提示词：

```text
请阅读当前仓库的 README.md 和 SKILL.md，并严格按其中的 literature-downloader 工作流执行。

我的任务是：下载“[研究主题]”的近五年文献，优先高影响因子，最多 [数量] 篇，下载到 <下载目录>。

要求：
1. 只允许使用合法可访问的全文来源，不要绕过付费墙，不要使用盗版来源。
2. 先创建批量采集任务目录。
3. 生成候选文献表，并在每篇文献行中列出题名、年份、期刊或来源、影响因子/分区、指标年份、指标来源、DOI、文章地址、全文状态和下一步建议。
4. 尝试下载开放 PDF、HTML 或 XML；遇到登录页、403、订阅页、短 HTML 跳转页时，不要算成功，写入待处理清单。
5. 最后运行 scripts 生成中文下载报告、文章地址总表、已下载文献清单、待处理文献清单，并整理输出目录。
```

结论：Codex 是原生 Skill 体验；其他工具是“读 README/SKILL.md + 调 scripts”的体验。

## 批量采集模式

用户要“找 20 篇”“批量下载”“做一个文献包”“保存日志”时，使用批量采集模式。先读 `references/batch-harvest.md`。

1. 创建任务目录和配置：

```bash
python scripts/prepare_harvest_run.py --topic "LLM 在环境领域中的应用" --keywords "large language model|LLM|ChatGPT,environmental science|climate|sustainability" --years 2021-2026 --output-root literature_runs --run-name llm_environment_20260426
```

2. 把检索到的候选文献填入 `candidate_table.csv`。来源可以是 Crossref、OpenAlex、PubMed、Europe PMC、Zotero 导出、出版社页面或人工核对结果。
3. 自动分级候选文献：

```bash
python scripts/rank_candidates.py --run-root literature_runs/llm_environment_20260426
```

4. 只尝试下载合法、直接可访问的全文链接：

```bash
python scripts/download_accessible_fulltexts.py --run-root literature_runs/llm_environment_20260426 --retry-failed
```

5. 生成普通人能看懂的报告，优先输出 PDF：

```bash
python scripts/generate_readable_report.py --run-root literature_runs/llm_environment_20260426 --title "LLM 在环境领域中的应用：文献下载报告" --pdf
```

6. 如果用户要求影响因子/分区，先核验并填写 `journal_metrics.csv` 或候选表里的 `impact_factor`、`jcr_quartile`、`metric_year`、`metric_source`、`indexing`。不要让脚本凭空猜指标。
7. 汇报结果：先给用户 `下载报告.pdf` 或 `下载报告.html`，再给 CSV 日志路径。报告里要说明候选总数、成功 PDF、HTML/XML 全文、无法访问数量、失败原因、影响因子/分区来源和下一步合法获取路线。

下载脚本不会绕过付费墙。它只保存能直接合法访问的 PDF/HTML/XML；如果遇到登录页、订阅页、403/401、付费墙，会记录为不可访问。

## 输出经验

面向用户时，不要让用户先读 CSV。CSV 是技术日志，不是交付物。

优先交付：

1. `下载报告.pdf`：最适合普通用户查看和转发。
2. `下载报告.html`：PDF 生成失败时的可读替代品，浏览器直接打开。
3. `README_先看我.md`：短说明，告诉用户下载了什么、哪些没下成、下一步怎么办。
4. `downloaded_fulltexts/` 或用户指定目录：实际 PDF/HTML/XML 文件。
5. `文章地址总表.csv`：只列题名、文章地址、是否下载成功、全文状态、DOI、下一步建议。
6. `候选文献总表.csv`、`高优先级文献.csv`、`已下载文献清单.csv`、`待处理文献清单.csv`：所有列出文献的表都必须直接包含题名、年份、期刊或来源、影响因子/分区/指标来源、DOI、文章地址、全文状态和下一步建议。英文 CSV 只作为脚本兼容备份。

除实际文献文件外，输出文件名默认用中文，让用户一眼知道用途：

- `README_先看我.md`：最短说明。
- `下载报告.pdf`：主报告。
- `下载报告.html`：可在浏览器打开的报告。
- `检索计划.md`：关键词、检索式、数据库入口。
- `配置文件.json`：本次任务设置。
- `文章地址总表.csv`：题名、地址、是否下载成功。
- `候选文献总表.csv`：所有候选文献。
- `高优先级文献.csv`、`中优先级文献.csv`、`低优先级文献.csv`：筛选结果。
- `期刊指标.csv`：仅作为后台核验来源或临时模板，不作为主要交付物。
- `下载日志.csv`：下载尝试记录。
- `总下载日志.csv`：合并后的最终状态。
- `去重清单.csv`：重复文献处理依据。
- `已下载全文/`：PDF、HTML、XML 全文文件夹。
- `内部数据_一般不用打开/`：英文兼容文件、原始日志和中间数据。

如果需要影响因子或分区，报告中显示“影响因子/分区”列。指标必须标注年份和来源；没有核验时显示“待核验”，不要编数字。

凡是列出文献的文件，都要把重要期刊指标并入每一行。不要让用户在文献清单和期刊指标表之间来回对照。

```bash
python scripts/make_student_outputs.py --run-root literature_runs/llm_environment_20260426
python scripts/organize_final_outputs.py --run-root literature_runs/llm_environment_20260426
```

报告必须用中文解释状态：

- `success_pdf` 写成“已下载 PDF”。
- `success_html` 写成“已保存网页全文”。
- `success_xml` 写成“已保存 XML 全文”。
- `inaccessible` 写成“平台限制直连下载，建议学校库/馆际互借/联系作者”。
- `metadata_only` 写成“只找到题录，尚未找到合法全文”。
- `failed` 写成“尝试失败，需人工复查入口”。

把“为什么有些没下载成功”写清楚：这通常不是程序坏了，而是 ACS、Elsevier、SpringerLink 等平台对非订阅、非浏览器、非机构网络的下载限制。不要把失败包装成成功。

## 输出给用户的格式

普通检索任务输出：

- 检索式：关键词块、同义词、Boolean 检索式、数据库版本。
- 推荐文献：题名、年份、期刊、DOI/PMID、推荐理由。
- 期刊质量：影响因子/JCR 或 SJR/Scopus/索引状态，说明来源和年份。
- 全文状态：已开放、可从出版社下载、可从 PMC/仓储下载、需要学校库、建议馆际互借、建议联系作者。
- 追踪表：题名、DOI/PMID、来源、状态、全文入口、备注。

批量任务额外输出：

- 任务目录路径。
- `下载报告.pdf` 路径；如果没有生成 PDF，则给 `下载报告.html`。
- `README_先看我.md` 路径。
- 实际全文文件夹路径。
- `combined_download_log.csv` 或 `download_log.csv` 路径。
- 未解决文献的下一步处理清单。

如果用户提供 DOI、PMID、题名或文献清单，先解析标识符和找全文，不要从泛泛关键词检索开始。

---
> Source: [Lucaswangzcx/literature-downloader-skill](https://github.com/Lucaswangzcx/literature-downloader-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
