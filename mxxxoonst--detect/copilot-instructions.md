## detect

> 多源异构文件的**数据预处理与信息抽取流水线**：从混合格式文件（JSON/CSV/TSV/SQL/XLSX/TXT/日志）中

# CLAUDE.md — PII Detect 项目上下文

## 项目说明

多源异构文件的**数据预处理与信息抽取流水线**：从混合格式文件（JSON/CSV/TSV/SQL/XLSX/TXT/日志）中
按后缀路由 / 嗅探 .txt 真实格式、容错解析，并提取五类结构信息及 PII 种子。三阶段管线：

```
corpus_root → [阶段0 嗅探 sniff] → [阶段1 分级解析 parse] → [阶段2 Schema 提取 extract]
```

- **语言/包管理**: Python 3.12+ / uv
- **数据规模**: ~45,473 文件、GB 量级（真实数据在云服务器，当前不可下载；开发用 `test_data/samples/`）
- **文档分工**：
  - 本文件 = 稳定的高价值上下文（约束/命令/结构/约定/禁止事项）
  - `docs/guides.md` = 详细业务逻辑、API、数据流、各阶段算法（实现层，按需查阅）
  - `todo_list.md` = 演进中的模块/阶段设计、占位接口、待办与未来方向（易变）

## 关键环境约束

- **⚠ 「本地编辑 + 远程执行」架构**：Claude Code 运行在本地 Windows，但**代码实际跑在远程服务器，GB 级数据也只在远程**——
  **本地没有数据、没有 `.venv`，本地直接运行任何脚本都会失败，不要在本地验证**。
  - 需要运行/验证时一律通过 SSH 在远程执行：
    `ssh root@172.17.66.200 "cd data/header_parser/zlf/PII_detect/detect/ && uv run python <脚本>"`
  - 处理 GB 数据**优先用小样本调试**，别一上来跑全量。
  - 本地仅用于编辑代码、读 `test_data/samples/` 小样本、跑单元测试逻辑（不依赖远程数据的部分）。
- **流式/抽样**：禁止整文件 load，一律流式读或只读头部（JSON 大数组用 ijson，JSONL 逐行，CSV/xlsx 每文件 `LIMIT SAMPLE_PER_FILE`）
- **编码混合**：GBK/UTF-8 混存，chardet 探编码，`errors='replace'` 解码；UTF-16/32 先靠 BOM（`detect_bom`）识别编码再判二进制；分片阶段忠实使用 `grade.encoding`
- **嗅探只管 .txt**：真实语料 json/csv/jsonl/sql/tsv/xlsx 等结构化文件 ~99% 后缀与内容一致，故 `sniff_file()` **对这些扩展名直接信任**（噪声交给阶段1 容错解析暴露，不在嗅探期纠格式）；仅 `.txt`/`.log`/无扩展名走 `vote_format()` 内容投票路由。`.db`/`.sqlite` 不再特殊处理（真实样例太少，已移除）。
- **禁止持久化 PII 原值（默认）**：value 画像只存统计摘要（长度分布、字符宏类分布、脚本直方图、模式模板），默认不存原始值。
  **唯一例外**：`extract`/`pipeline` 显式加 `--keep-samples` 才按 pattern 去重保留 ≤5 个样本（owner 授权的调试/下游用途）；`--mask-samples` 进一步脱敏（保留分隔符与长度、内容字符打码）。`profile_value()` 永不持有原值，样本仅在 `aggregate_profiles()` 显式开启时落地。

## 环境与依赖

```bash
cd F:\zlf\paper_dataPipeline\detect
uv sync              # 安装依赖
```

- 核心依赖：`chardet`（编码探测）、`ijson`（JSON 流式）、`json5`（容错解析）、`pytest`（测试）
- 依赖与版本声明集中在 `pyproject.toml`（uv 管理）

## 常用命令

```bash
# 生成测试数据 (~27 文件，覆盖 格式×质量×编码 矩阵)
uv run python test_data/generate.py [--synth_data test_data/samples] [--seed 42]

# 流水线子命令 (root 支持目录或单文件)
uv run python main.py sniff    <corpus_root> -o synth_data [-v]
uv run python main.py parse    <corpus_root> -o synth_data [-v]
uv run python main.py extract  <corpus_root> -o synth_data [-v] [--field-mode template|fold]
uv run python main.py pipeline <corpus_root> -o synth_data [-v] [--field-mode template|fold]

# 测试
uv run python -m pytest tests/ -v
```

CLI 标志（`root`/`-o,--output-dir`/`-f,--output-file`/`-v,--verbose`/`--restart` 由共享 parent parser 提供，四命令通用）：
- `-v`：开 DEBUG 级日志（默认 INFO）
- `--restart`：清空已有 `grades.jsonl` / `schema_units.jsonl` 重新开始（**默认续跑**）
- `--field-mode {template,fold}`（仅 `extract`/`pipeline`）：字段主干方案，`template`(默认) / `fold`
- `--keep-samples`（仅 `extract`/`pipeline`，**默认关**）：信息三按 pattern 去重保留 ≤5 个样本值（⚠ 落原值，破红线，需显式开启）
- `--mask-samples`（仅 `extract`/`pipeline`）：配合 `--keep-samples`，样本脱敏（保留分隔符与长度、内容字符打码），不落原始字符
- 产出文件见 `docs/guides.md`（`extract`/`pipeline` 各产出 `*_report.json` + `vocab_table.json`，并写 `<output-dir>/<command>.log`）

**⚠ 阶段间落盘流式传递（内存恒定 + 断点续跑）**：阶段1 逐文件追加 `grades.jsonl`，阶段2 流式消费它、
逐文件追加 `schema_units.jsonl`，再两遍流式聚合 `vocab_table` / 全局视图。**不在内存累积全量 Grade / SchemaUnit**，
适配 ~45k 文件 / GB 级；进程中断后重跑同一命令自动跳过已处理文件（崩溃截断的尾行被容错跳过）。`--restart` 强制重来。

## 项目结构

```
detect/
├── main.py                       # CLI 入口 (argparse 子命令) + resolve_input() 容错路径解析
├── pyproject.toml                # uv 包管理 + 依赖声明
├── src/
│   ├── constants.py              # 阈值常量、PII 正则、魔数定义
│   ├── sniff/                    # [阶段0] 内容嗅探
│   │   ├── sniffer.py            # sniff_file(path) → (fmt, enc, conf)
│   │   ├── voting.py             # vote_format(lines, text) → {fmt: score}
│   │   └── profiler.py           # profile_corpus(root, files=None) → 交叉表
│   ├── parse/                    # [阶段1] 容错分级解析
│   │   ├── grade.py              # Grade dataclass + grade_parse() 路由
│   │   ├── json_parser.py        # JSON/JSONL: strict(ijson) + tolerant(json5)
│   │   ├── csv_parser.py         # CSV/TSV: strict(列一致) + tolerant(skip bad lines)
│   │   ├── sql_parser.py         # SQL 文本: regex 抽 CREATE/INSERT 头部 + 方言标注
│   │   └── xlsx_parser.py        # xlsx: openpyxl read_only 读 sheet 表头 (二进制只读)
│   ├── extract/                  # [阶段2] Schema 单元化提取 (仅 tier1)
│   │   ├── extractor.py          # stream_schema_units()(流式构建) + finalize_ir_from_units()(IR聚合) / finalize_from_units()(非IR+vocab) + _aggregate_global_view()
│   │   ├── schema_types.py       # 共享 TypedDict: SchemaPartition/SchemaUnit/FieldInfo/VocabTable
│   │   ├── schema_partition.py   # partition_file() — 文件内 Schema 分片
│   │   ├── schema_unit.py        # build_schema_unit() — 单遍折叠遍历组装五类信息
│   │   ├── vocab_table.py        # build_vocab_table() — 跨表同义倒排
│   │   ├── skeleton.py           # 信息一: 结构骨架 + union-schema 归一聚类
│   │   ├── value_profile.py      # 信息三: value 画像 (统计摘要，不存原值)
│   │   └── pii_seed.py           # 信息五: PII 种子 (key 名推断 + free_text 标记)
│   └── utils/
│       ├── logger.py             # setup_logger()/get_logger()，pii_detect 根命名空间
│       ├── jsonl.py              # append_jsonl/iter_jsonl — 阶段间落盘+断点续跑(容忍截断尾行)
│       ├── encoding.py           # chardet 探编码 + safe_decode
│       ├── file_utils.py         # is_binary(), 读头, walk_files
│       └── text_utils.py         # 括号平衡, 列稳定性, 非空行, JSON 试探
├── tests/                        # pytest (test_sniff / test_parse / test_extract)
├── test_data/
│   ├── generate.py               # 测试数据生成器
│   └── samples/                  # 生成的测试样本
├── output/                       # 流水线运行结果输出 (gitignored)
├── docs/guides.md                # 详细业务逻辑/API/数据流
├── todo_list.md                  # 演进设计与待办
└── CLAUDE.md                     # 本文档
```

## 代码约定

- **数据结构用 TypedDict（非 dataclass）**：结构以 dict 形态流动并序列化为 JSON，TypedDict 在保持
  `obj["key"]` 运行态不变的前提下提供静态类型契约。共享类型集中在 `extract/schema_types.py`。
- **日志统一走 `pii_detect` 根命名空间**：子模块顶部 `log = get_logger(__name__)`（自身无 handler，向上
  propagate）；CLI 在 `main.py:_configure_logging()` 一处 `setup_logger("pii_detect", ...)` 挂 file handler。
  默认 INFO，`-v` 开 DEBUG。**不要用 `print`**（`main.py` 内报表/阶段标题均已收口为 `log.*`）。
- **日志级别约定**：INFO = 阶段起止/进度/全局摘要/WARNING/ERROR；DEBUG = 逐文件吞错、投票得分、分片方法、骨架解析。
  `except` 吞错处必须记日志（WARNING 实质失败 / DEBUG 可接受的回退）。
- **ID 作用域**：`SchemaUnit` id（`sch_NNNNN`）与 `field_id`（`f_NNNNN_NN`）在单次 pipeline 运行内自增，
  不跨 run 持久化；`reset_unit_counter()` 仅供测试。
- **骨架签名非 JSON**：`structure_signature()` 输出含裸类型标记（`<int>` 等），不是合法 JSON；解析前需 `re.sub` 归一化。

## 禁止事项

- ❌ **对结构化扩展名再做内容投票**——`.txt`/`.log`/无扩展名才投票；json/csv/sql/xlsx 等后缀直接信任，格式错配交由阶段1 容错解析暴露为 tier2/3。
- ❌ **依赖已删除的旧入口/模块**——`extract_all()` / `extract_five_infos()`（内存版主入口 + 扁平包装）及 `vocabulary.py` / `topology.py` 已删；阶段2 一律走流式 `stream_schema_units` + `finalize_ir_from_units`(IR) / `finalize_from_units`(非 IR)。`parsers/` 目录也已不存在。

---
> Source: [mxxxoonst/detect](https://github.com/mxxxoonst/detect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
