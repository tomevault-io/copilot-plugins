## naive-text

> 本项目是一个 Rust 命令行工具，从六朝正史文本语料（晋书、宋书、南齐书、梁书、陈书、魏书）中提取三层结构化信息：

# person_extract — 六朝史书结构化提取工具

## 项目概述

本项目是一个 Rust 命令行工具，从六朝正史文本语料（晋书、宋书、南齐书、梁书、陈书、魏书）中提取三层结构化信息：

| 层级 | 内容 | 说明 |
|------|------|------|
| Layer 1 | 传主信息 | 从传记开头解析姓名、字、籍贯等 |
| Layer 2 | 篇内人名 | 识别正文中提及的其他人名 |
| Layer 3 | 事件提取 | 提取任命、战役、薨卒等带时间的事件 |

## 构建与运行

```bash
# 构建
cargo build --release

# 全量提取（输出到 output/*.json）
cargo run -- extract corpus

# 按年号查询
cargo run -- query "太和三年"
cargo run -- query "太和元年-太和六年"

# 按政权查询
cargo run -- query "@东晋"

# 打印年号时间轴
cargo run -- timeline
```

### 依赖

- Rust Edition 2024，需 rustc 1.85+
- clap 4（命令行解析）、regex 1、serde / serde_json、walkdir 2

### 输出文件（`output/` 目录）

- `persons.json` — 传主摘要 + 篇内人名
- `events.json` — 结构化事件
- `timeline.json` — 年号时间轴、时间索引、统计

## 代码结构

```
src/
├── main.rs      # CLI 入口与输出编排
├── types.rs     # 核心数据结构（Person, Book, Section 等）
├── parser.rs    # 传记开头正则解析
├── scanner.rs   # 语料目录扫描
├── surname.rs   # 姓氏拆分（复姓优先）
├── regime.rs    # 政权与年号定义（23 个政权）
├── event.rs     # 事件提取与时间索引
├── intext.rs    # 篇内人名识别
├── titles.rs    # 官职名数据库
└── extract.rs   # 引用统计汇总
```

### 语料目录结构

```
{书名}/
  {NN_section}/        # 本纪 / 列传 / 载记 / 志
    {NN_卷}/
      {NN_人名}.txt    # 单篇传记文本
```

## 代码质量

```bash
# 格式化
cargo fmt --all

# 静态分析（应零警告）
cargo clippy --all-targets -- -D warnings

# 单元测试
cargo test
```

### pre-commit 安装

```bash
pip install pre-commit
pre-commit install
```

安装后每次 `git commit` 会自动运行 `cargo fmt`、`cargo check`、`cargo clippy` 以及文件格式检查。

## 提取常见陷阱

### 1. 正则贪婪匹配

传记开头格式高度模式化，但正则表达式的贪婪特性容易导致过度匹配。例如：

```
柳世隆字彦绪，河东解人也。
```

若 `{Origin}` 部分的正则不加限制，可能吞掉句号后的内容。**始终用否定字符类（`[^，。]`）约束每个捕获组。**

### 2. 复姓必须优先匹配

名字拆分时，若先按单字姓处理 `司马褧`，会得到姓 `司` 名 `马褧`。代码中 `split_name()` 和 `build_name_regex()` 均先检查复姓列表（`COMPOUND_SURNAMES`），再回退到单字姓。**任何新增的姓名解析逻辑都必须保持此顺序。**

### 3. 年号歧义——同名年号属于不同政权

"太和"同时出现于东晋与北魏。若不按来源书籍消歧，会把晋书中的"太和"错归北魏。代码通过 `Book → Regime` 映射解决：晋书条目只匹配西晋/东晋年号，魏书条目只匹配北魏年号。**切勿绕过此消歧逻辑。**

### 4. 时间作用域传播

本纪等叙事文本中，一个时间标记（如"太和三年"）后续段落若无新时间标记，则默认延续该时间。`TimeScope` 机制用字节偏移量标记每个时间标记的管辖范围。**如果修改事件提取逻辑，必须保持"到下一个时间标记为止"的作用域语义。**

### 5. 篇内人名的误报过滤

古文中大量两三字组合"看起来像人名"但实际是官职、地名或普通词组。`intext.rs` 维护了一个黑名单来过滤误报。**新增识别模式后需检查误报率，必要时补充黑名单。**

### 6. `scanner.rs` 跳过非传记文件

扫描器会跳过"目录""史论""史评""论""评""赞""序""注""正文""附录"以及以"年"结尾的文件。**如果语料增加新类型的非传记文件，需更新跳过规则。**

## 古文提取的特殊问题

### 姓名系统的复杂性

一个人可能有多种称谓：**讳**（given name）、**字**（courtesy name）、**小字**（childhood name）、**庙号**（temple name）、**谥号**（posthumous title）。代码通过 `PersonKind` 枚举区分四种人物类型，各有不同的名字字段组合：

- `Emperor` — 庙号 + 谥号 + 讳 + 字 + 小字
- `Official` — 姓 + 名 + 字 + 籍贯
- `Deposed` — 称号 + 讳 + 字 + 小字
- `Ruler`（载记人物）— 姓名 + 字（部分因避讳缺字）

### 避讳与缺字

载记中部分少数民族政权人物的字未被记录（原文直接写描述而非字），`parser.rs` 中对此有专门的回退模式。`CourtesyName::NotRecorded` 标记此类情况。**不要假设所有人物都有字。**

### 异体字与罕见字

古文中存在大量异体字和罕见字（如"㔨"），这些字在 Unicode CJK 扩展区。正则中的 CJK 字符类 (`CJK_CHAR`) 使用否定排除法（排除标点和空白），而非正面枚举 CJK 范围，以确保覆盖所有可能的字符。**不要用 `\p{Han}` 之类的 Unicode 属性，它可能遗漏扩展区字符。**

### 中文数字解析

年份使用中文数字：元年=1，二年=2，十三年=13，二十一年=21。`event.rs` 中有完整的中文数字解析逻辑。**注意"元年"是特例，不要遗漏。**

### 干支纪日

日期采用干支表示（甲子、乙丑……），共 60 组合。代码用天干（甲-癸）和地支（子-亥）的笛卡尔积构建正则。

### "姓X氏"模式

部分皇帝传记中姓氏以"姓X氏"格式单独标注（如"姓司马氏"），需额外正则提取。

### 僑置地名

东晋南朝设有大量侨置郡县（如侨置的"琅邪郡"并非在山东而在江苏），这些地名在事件提取中通过 `is_qiao` 标志标记。**地名不可简单等同于现代地理位置。**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Feiyang472) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
