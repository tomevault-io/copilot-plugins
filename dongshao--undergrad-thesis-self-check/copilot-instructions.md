## undergrad-thesis-self-check

> 给南京大学软件工程专业本科毕业论文送审前的自检 skill。学生提供论文 PDF，先从 PDF 抽题目并请学生确认，自动判定"工程型 / 学术型"并请学生确认，再询问"是否团队子模块"（如是追问总项目+本人模块），按对应类别 32-34 条 checklist（含 3-4 条红线 + 6 条参考文献规范）逐项审查，给出带原文页码定位的反馈、红线高亮；并对软件工程图表规范性（用例图 / 架构图 / ER 图 / 类图 / 流程图等）做 UML 规范检查；最后可选追加 AI 文风+错别字检查。报告写到 <论文同目录>/<论文题目>-review.md（题目来自 PDF 内容而非文件名）。**不做**优/良/中/及格/不及格的总分估计。仅在用户明确说"本科论文自检""本科论文自查""本科送审前自查""我的本科论文有什么问题"等触发词且提供 PDF 路径时启用。


# NJU SE 本科论文自检 skill

给本科生送审前自查用。本 skill 是学生视角的 checklist 逐项审查。

## 何时使用

仅在用户明确说出以下触发词之一**且**提供论文 PDF 路径时启用：
- 「本科论文自检」「本科论文自查」「本科送审前自查」
- 「我的本科论文有什么问题」「我的本科论文能过吗」
- 「undergrad self-check」「pre-defense check」（且提供 PDF）

不要只看到 PDF 就触发。

## 调用格式

```
本科论文自检 /path/to/paper.pdf
本科论文自检 /path/to/paper.pdf --type=engineering   # 跳过类型判定
本科论文自检 /path/to/paper.pdf --type=academic
本科论文自检 /path/to/paper.pdf --type=engineering --team   # 标记团队论文（仍会追问总项目+模块）
本科论文自检 /path/to/paper.pdf --auto                # 全自动模式（任何 Gate 不再询问）
```

`--type` 可选值：`engineering`（工程型）/ `academic`（学术型）。
`--team` 标记团队子模块；不带则进入团队 Gate 询问。
`--auto` 触发**全自动模式**：用户消息中含「全自动」「自动模式」「auto」「--auto」「-y」「yes to all」任一关键词即视为开启。

## 全自动模式（AUTO_MODE）

开启后，整条流程从步骤 1 跑到步骤 11 一气呵成，**任何 Gate 都不再向用户索要确认**——本 skill 的 Claude 用自己的判断直接通过。具体差异如下：

| 环节 | 缺省（人工模式） | 全自动模式 |
|---|---|---|
| 步骤 2 抽题确认 | 把抽到的题目贴出来，等 y / 纠正 | 直接用抽到的 `title_clean` 作为最终题目，不询问；如四种策略全部失败回退到文件名，写入报告"附注：题目抽取退化"段 |
| 步骤 1 骨架异常 | 列出问题，问是否继续 | 直接继续；**不在报告里加诊断附注**（最终结论按真实结构给出，读者无需感知中间诊断） |
| 步骤 3 类型判定 | 给判定+证据，等 y / 纠正 | 用判定结果直接进入步骤 4；**不在报告里加"类别判定依据"段**（判定结果通过整篇报告所用 checklist 类型隐含体现）；仅当信号严重对半、按"工程型默认"处理时，在报告头部加一行「类别存疑，建议人工复核」 |
| 步骤 4 团队判定 | 询问是否团队 + 追问总项目/模块 | 默认按 solo 处理（不读 overrides）；如骨架元信息或题目里出现"团队 / SELABS / 模块"等强信号，按 team=yes 处理但 `team_project` / `my_module` 设为"未提供（AUTO_MODE 跳过追问）"，并在报告头部写明 |
| 步骤 8 图表能力探针失败 | 提示用户并跳过本步 | 自动跳过；在报告第七节写明"当前模型不支持图像输入，跳过 UML 图形评估，仅做基于图注的文本检查" |
| 步骤 11 是否做文风检查 | 询问用户 y/n | 默认进行；如未安装 humanizer-zh 则跳过文风部分但仍执行错别字检查，并在第十一节顶部写明"未安装 humanizer-zh，本节仅含错别字检查；如需补充 AI 文风扫描请安装后重跑" |

全自动模式下需保留的硬停点（**不能跳过**）：
- 论文 PDF 路径不存在 / 不是 PDF → 报错并停止
- `pdfplumber` 未安装 → 报错并提示 `pip install pdfplumber`，停止
- 步骤 1 骨架抽取脚本崩溃（非"骨架不完整"，是真的崩） → 报错并停止
- 步骤 8 PyMuPDF 未安装 → 不停止，跳过步骤 8 并在报告第七节写明"PyMuPDF 未安装，请运行 pip install pymupdf 后重跑此步骤"
- 步骤 11 切块脚本崩溃 → 报错并停止；脚本本身正常但 `split_failed=true` → 不停止，进入"骨架降级模式"（文风仅扫骨架可见的段落），并在副报告头部加警示

全自动模式下需在**对话中实时输出**关键节点，让用户看到进度（一句话一节点即可）：

```
[1/11]  抽骨架完成（{pages} 页 / {chapters} 章）
[1/11]  题目：《...》
[2/11]  题目确认（AUTO_MODE 默认通过）
[3/11]  类别：工程型（依据：第三章"系统架构"+ 第五章"系统测试"）
[4/11]  团队：solo / team={team_project} - {my_module}
[5/11]  已加载 engineering-30.md（34 条）+ overrides（仅 team 模式）
[6/11]  红线扫描：命中 X 条
[7/11]  全量审查完成：✅ A / ⚠️ B / ❌ C
[8/11]  图表规范性：识别 N 张图 / 命中 M 类图 / 警告 K 条
[9/11]  报告写入 {report_path}
[10/11] 摘要已输出到对话
[11/11] 文风+错别字检查：N 块 chunks / 错别字 X 条 / 文风 Y 条
完成。
```

人工模式下保持原行为不变。

## 前置依赖

- `pdfplumber`：如缺失提示用户 `pip install pdfplumber`
- `PyMuPDF`（包名 `pymupdf`，import 名 `fitz`）：步骤 8 渲染整页为 PNG。如缺失提示 `pip install pymupdf`；缺失只导致步骤 8 跳过，不影响其他步骤
- `humanizer-zh` skill（github.com/op7418/Humanizer-zh）：步骤 11 AI 文风检查。如未安装，AUTO_MODE 下跳过文风部分仅做错别字检查；人工模式下询问用户后决定
- 本 skill 自带 `scripts/extract_skeleton.py`（步骤 1 抽骨架）与 `scripts/extract_chunks.py`（步骤 11 切块），无需依赖外部 skill

## 输出位置约定

- **主报告** → `<论文 PDF 所在目录>/<论文题目>-review.md`（学生第一眼能看到）；如已存在则顺延为 `<论文题目>-review-1.md` / `-review-2.md` / ...，**不覆盖前次报告**，便于学生对照修改进度。具体路径由步骤 9 生成的 `$REPORT_PATH` 决定
- **文风副报告** → `<论文题目>-style.md`（与主报告共用同一下标顺延，如 `-review-2.md` 配 `-style-2.md`）；仅当步骤 11 实际执行时生成
- **中间产物** → `<论文 PDF 所在目录>/<TMP_DIR>/`，`<TMP_DIR>` 由步骤 1 顺延决定：默认 `tmp`，若已存在则改用 `tmp1` / `tmp2` / ... 直到找到未占用的目录名。**不覆盖前次产物**，学生可对比多次跑的差异。下文为可读性以 `tmp/` 占位指代当前实际目录。
  - `<TMP_DIR>/skeleton.md` — 步骤 1 抽出的骨架
  - `<TMP_DIR>/figs/page-NNN.png` — 步骤 8 渲染的关键图所在页
  - `<TMP_DIR>/chunks/<range>.txt` + `chunks-index.json` — 仅步骤 11 触发时生成（lazy 切块）

不要再往 `/tmp/` 系统临时目录写——重启会丢，也不便于学生回看；也不要把中间产物直接散落在论文目录下。`<paper_dir>/<TMP_DIR>/` 是唯一中转区。

## 步骤执行原则（**强制**）

1. **不得为节省 token / 时间跳过任何步骤或子步骤**——只要前置依赖满足且未触发 SKILL.md 明文允许的退化分支，本 skill 就必须把每个步骤完整跑完。学生付费跑一次自检的预期是「完整审查」，跳过等于交付残缺产品。
2. **仅以下情形允许跳过**：
   - PDF 文件不存在 / 不是 PDF → 整体停止
   - 必需依赖缺失（pdfplumber / PyMuPDF）→ 按 SKILL.md 各步骤明文规定的退化路径处理（如步骤 8 缺 PyMuPDF 时跳过本步骤；不是绕过，必须在最终报告里写明跳过原因 + 安装命令）
   - 模型图像能力探针失败（VISION_CAPABLE=false）→ 步骤 8 进退化路径，仍要做基于图注的文本检查
   - 步骤 11 学生明确选「n 跳过」（人工模式下询问后），或 humanizer-zh 未安装（AUTO_MODE 下降级为仅错别字）
3. **不得编造"未执行"措辞**：报告里如果写了某步骤"未执行"或"跳过"，必须给出具体原因（依赖缺失 / 探针失败 / 用户拒绝），不能是"为节省时间"。
4. **token 不是借口**：本 skill 设计上预期一次完整跑可能消耗较多 token——这是学生选择跑自检时已知的成本。跳步骤换"看起来跑完了"是对学生的欺骗。

## 主流程（11 步）

| 步骤 | 内容 |
|---|---|
| 1 | 抽骨架 + 抽取论文题目（含 tmp 目录顺延） |
| 2 | 题目确认 Gate（AUTO_MODE 跳过） |
| 3 | 类型判定 Gate（工程型 / 学术型；AUTO_MODE 跳过） |
| 4 | 团队子模块 Gate（AUTO_MODE 跳过） |
| 5 | 加载对应 reference + 应用 overrides |
| 6 | 红线扫描（工程 4 条 / 学术 3 条） |
| 7 | 全量审查（剩余 ~30 条） |
| 8 | **软件工程图表规范性检查（新增）** |
| 9 | 生成主报告 |
| 10 | 对话输出摘要 |
| 11 | **（可选）文风与错别字检查（新增）** |

### 步骤 1：抽骨架 + 抽取论文题目

```bash
PAPER_PDF="/path/to/paper.pdf"
PAPER_DIR=$(dirname "$PAPER_PDF")

# tmp 目录顺延：默认 tmp；已存在则 tmp1 / tmp2 / ... 找首个未占用的
# 不要复用或覆盖前次 tmp 目录——学生可能还要回看上一次的 chunks / figs / skeleton
TMP_DIR="$PAPER_DIR/tmp"
i=1
while [ -e "$TMP_DIR" ]; do
  TMP_DIR="$PAPER_DIR/tmp$i"
  i=$((i+1))
done
mkdir -p "$TMP_DIR"
echo "TMP_DIR=$TMP_DIR"   # 后续步骤的 Python 代码块从这里读取实际目录

python3 ~/.claude/skills/undergrad-thesis-self-check/scripts/extract_skeleton.py \
  "$PAPER_PDF" \
  --out "$TMP_DIR/skeleton.md"
```

**重要**：本次会话内所有后续步骤（步骤 8 渲染图 / 步骤 11 切块）必须用同一个 `$TMP_DIR`。Bash 工具的 shell state 不跨 call 持久，所以**记住步骤 1 echo 出的路径**，在后续每个 Bash / Python 代码块的开头**显式重赋值**（Bash 用 `TMP_DIR="..."`；Python 用 `TMP_DIR = "..."`）。**不要**在后续步骤里再跑一次顺延逻辑——那会建一个新的空目录。

读骨架。如脚本失败 / 骨架 < 30 页 / 章节切分退化（出现"未识别章节"）：
- 先告知用户骨架异常，列出问题，问是否继续（基于不完整骨架审查会漏检）。
- 本科论文一般 ≥ 35 页（15k 字 ÷ 约 400 字/页 ≈ 38 页），骨架 < 30 页提示偏低。
- **AUTO_MODE**：不询问，直接继续；不在最终报告里加诊断附注，按真实章节边界手工修正判定（脚本崩溃属硬停点，那种情况必须停）。

**抽取论文题目**（用于步骤 9 的报告文件名）。4 策略按优先级依次尝试：

```python
# 直接从 PDF 前 10 页提取头部文本来找题目
# 策略 1：NJU SE 本科封面 "题目：" 行（回归测试 8/9 命中，优先）
# 策略 2：摘要首页"毕业论文题目：" 行（模板备用）
# 策略 3：封面"论 文 题 目" 区段多行合并（专硕封面模板）
# 策略 4：fallback 到 PDF 文件名

import re, pdfplumber, os
PDF = "/path/to/paper.pdf"
title = None

with pdfplumber.open(PDF) as pdf:
    head_text = '\n'.join((p.extract_text() or '') for p in pdf.pages[:10])

# CID 编码检测：若头部 CID 标记过多或中文字符极少，提前警告
cid_count = head_text.count('cid:')
cjk_count = sum(1 for c in head_text if '一' <= c <= '鿿')
if cid_count > 50 or cjk_count < 100:
    print(f"警告：PDF 疑似 CID 编码（cid: 出现 {cid_count} 次，中文字符仅 {cjk_count} 个），"
          f"文字无法正常提取，题目抽取将大概率失败，建议直接使用文件名作为题目。")

# 策略 1（NJU SE 本科模板）：题目[：:] … 至下一个字段
m = re.search(
    r'题目[：:]\s*([\s\S]{1,150}?)(?=\n院系|\n学院|\n专业|\n本科生姓名)',
    head_text
)
if m:
    title = re.sub(r'\s+', '', m.group(1)).strip()

# 策略 2（备用模板）：毕业论文题目[：:] … 至下一个字段
if not title:
    m = re.search(
        r'毕业论文题目[：:]\s*([\s\S]{1,150}?)(?=(?:本科|学士学位|学生姓名|指导教师|学\s*校\s*代\s*码|软件工程专业|计算机.{0,4}专业|.{0,8}\d{4}\s*级\s*本科生))',
        head_text
    )
    if m:
        title = re.sub(r'\s+', '', m.group(1)).strip()

# 策略 3（专硕封面模板）：论 文 题 目 多行区段
if not title:
    m = re.search(r'论\s*文\s*题\s*目\s*([\s\S]{0,200}?)(?=作\s*者\s*姓\s*名|学位类别|研\s*究\s*方\s*向|学\s*生\s*姓\s*名)', head_text)
    if m:
        title = re.sub(r'\s+', '', m.group(1)).strip()

# 策略 4：fallback 到文件名
if not title or len(title) < 5:
    title = os.path.splitext(os.path.basename(PDF))[0]
    print(f"警告：未能从 PDF 内容抽取题目，回退到文件名: {title}")

# 清洗
title_clean = re.sub(r'[\\/:*?"<>|\r\n\t]', '', title).strip()
if len(title_clean) > 120:
    title_clean = title_clean[:120]
print(f"PAPER_TITLE={title_clean}")
```

把得到的 `title_clean` 存为环境变量 `PAPER_TITLE` 或 shell 变量，供步骤 9 拼接报告路径使用。

### 步骤 2：题目确认 Gate

告诉用户抽取到的题目，让用户确认或纠正：

```
从 PDF 抽取到的论文题目：
《{title_clean}》

报告将写到：<论文同目录>/{title_clean}-review.md（如已存在则顺延为 -review-1.md / -review-2.md / ...）

确认输入 y 继续；如题目错误请直接给出正确题目。
```

**AUTO_MODE**：不询问，直接采用 `title_clean` 作为最终题目继续步骤 3。如果走到了策略 4（fallback 到文件名），在最终报告"附注：题目抽取退化"段写明，仍继续。

### 步骤 3：类型判定 Gate（工程型 / 学术型）

读骨架的元信息 + 各章首页文本，按以下规则判别：

| 信号 | 工程型 | 学术型 |
|---|---|---|
| 题目关键词 | "系统/平台/工具的设计与实现"、"模块"、"管理系统" | "方法研究"、"基于 X 的 Y 方法"、"X 算法/模型"、"面向 X 的 Y" |
| 第二章篇幅与重心 | 偏"技术栈介绍"（Spring、K8s、LLM 框架等） | 偏"相关工作综述"（按方法/路线分类） |
| 第三/四章 | "需求分析 / 系统架构 / 详细设计 / 实现" | "问题形式化 / 方法设计 / 实验设计" |
| 是否有"系统测试"或"运行截图"章 | 通常有 | 通常无 |
| 是否有"实验对比"章 | 弱（功能验证为主） | 强（基线对比 + 评价指标） |
| 关键句信号词 | "本系统"、"模块"、"部署"、"接口" | "提出"、"实验表明"、"结果表明"、"基线" |
| 摘要核心 | 系统的功能与工程价值 | 方法的创新与实验结论 |

**输出格式（无 emoji）**：

```
论文类型判别
------------------------------------------------------------
论文标题：《...》
我判定为：【工程型 / 学术型】
判定依据：
1. [一条具体证据，含章节或页码]
2. [第二条]
3. [第三条]

请确认：
- 输入 y / 工程 / 学术 来确认或纠正
- 如果是混合型，请告诉我重心偏向哪一类
```

如果用户提供了 `--type=...` 跳过本 Gate。

**AUTO_MODE**：不询问，直接采用本 skill 自己的判定结果进入步骤 4。**不在最终报告里加"类别判定依据"段**——判定结果通过整篇报告所用 checklist 类型隐含体现。仅当判定证据不足以单边定论（信号严重对半），按工程型处理时，在报告头部加一行「类别存疑，建议人工复核」。

### 步骤 4：团队子模块 Gate

**默认问，不自动猜**：

```
团队论文判定
------------------------------------------------------------
本论文是否为团队项目中的一个子模块？
（团队项目 = 多人协作、各自负责一个模块、各写一份论文，
  例如 SELABS 类的"统计安全 / 空间预约 / 资产管理"四模块拆分）

输入 y / 是 / team / n / 否 / solo
```

如答 yes，追问：

```
请提供两条信息（用 / 分隔）：
  1. 团队总项目名称（如：SELABS）
  2. 你负责的模块名（如：统计与安全管理模块）

格式示例：SELABS / 鉴权与数据中心管理模块
```

把 `team_project` / `my_module` 存为 shell 变量供后续使用。

**AUTO_MODE**：不询问。默认按 solo 处理（不读 overrides）；如骨架元信息或题目里出现"团队 / SELABS / 模块（且本论文为子模块）"等强信号，按 team=yes 处理，`team_project` / `my_module` 标为"未提供（AUTO_MODE 跳过追问）"并在报告头部说明。

### 步骤 5：加载对应 reference + 应用 overrides

按确认的类别读取：
- 工程型：`~/.claude/skills/undergrad-thesis-self-check/references/engineering-30.md`（34 条）
- 学术型：`~/.claude/skills/undergrad-thesis-self-check/references/academic-30.md`（32 条）

如团队 = yes：再读 `~/.claude/skills/undergrad-thesis-self-check/references/team-submodule-overrides.md`，对若干条目应用 patch（替换 / 放宽 / 追加 UND-TEAM-DECL）。

**学术型读取的特殊性**：academic-30.md 中前 27 条（RED-1/2/4 + UND-ENG-A-01..10 + UND-ENG-B-01..04,B-10 + UND-ENG-C-02..04 + UND-ENG-F-01..F-06）是"占位指引"，加载时应回到 engineering-30.md 读取对应 ID 条目的完整 yaml；只有 UND-ACA-S-01..05 的 5 条专属条目在 academic-30.md 中完整定义。

具体实现：审查每条时，按 ID 前缀决定从哪个文件取条目内容：
- UND-RED-* / UND-ENG-* → engineering-30.md
- UND-ACA-S-* → academic-30.md

### 步骤 6：红线扫描

先把红线（工程 4 条 / 学术 3 条）跑一遍。每条按 reference 文件里的"检查方法"操作：
- 若全文有 grep 类检查 → 用 Bash 在骨架文件上 grep（路径为 `$TMP_DIR/skeleton.md`）
- 若涉及篇幅 → 从骨架元信息读 page_start / page_end + 字符密度估算
- 若涉及章节内容 → Read 骨架对应章节段落

记录红线命中数，**不要在此停下**，继续步骤 7。

### 步骤 7：全量审查（剩余 ~30 条）

按 reference 文件中的顺序逐条审查。每条遵循"评判 + 定位 + 修改建议"模式：

**单条审查的内部模板**（合格项写一行；警告/不合格项展开）：

```yaml
- id: UND-ENG-A-05
  状态: ✅ / ⚠️ / ❌
  位置: P.42-44, §3.2.1（仅警告/不合格时给）
  评语: "评审专家口吻，1-3 句"（仅警告/不合格时给）
  修改建议: "具体怎么改"（仅警告/不合格时给）
```

**评语口吻参考**：
- 不写"你应该……"，写"建议在 X 章节补充 Y……"
- 不写"这里错了"，写"X 表述与 Y 不一致，建议……"
- 不堆套话；每条建议必须包含至少一处可定位的位置（章节号或页码）

**当某条审查需要更细节时**：用 Read 跳读骨架对应章节，必要时 Read 原 PDF（用页码范围）。但 80% 的判断应该在骨架上完成。

### 步骤 8：软件工程图表规范性检查

软工本科论文的用例图、系统架构图、ER 图、类图、流程图等是否符合 UML / 工程规范，是评审常考点之一。本步骤产出的所有问题计为"本步骤警告"，不计入红线，不影响步骤 6 的红线计数。

加载评估清单：

```
~/.claude/skills/undergrad-thesis-self-check/references/diagrams.md
```

#### 8.1 模型图像能力探针

**先确认模型能不能 Read 图像**——能力不足的模型直接跳过视觉评估，避免空跑。

```python
import fitz, os
PDF = "/path/to/paper.pdf"
TMP_DIR = os.environ["TMP_DIR"]   # 来自步骤 1 的 echo；不要重新计算 tmp 顺延，会建一个空的新目录
FIGS_DIR = os.path.join(TMP_DIR, "figs")
os.makedirs(FIGS_DIR, exist_ok=True)

# 用 PyMuPDF 渲染论文第一页作为探针
doc = fitz.open(PDF)
probe_path = os.path.join(FIGS_DIR, "_probe.png")
pix = doc[0].get_pixmap(dpi=150)
pix.save(probe_path)
doc.close()
print(f"PROBE_PATH={probe_path}")
```

**接下来本 skill 的 Claude 用 Read 工具读这张 _probe.png**：

- 能 Read 成功并描述出页面内容 → `VISION_CAPABLE=true`，进入 8.2
- Read 报错（如"current model does not support image input"）或返回空 → `VISION_CAPABLE=false`，跳到 8.6 退化路径
- PyMuPDF 未安装 → 跳过整个步骤 8，在报告第七节写一行"PyMuPDF 未安装，请运行 `pip install pymupdf` 后重跑此步骤"，进入步骤 9

探针成功后**删除 _probe.png**（不污染 figs/ 子目录）。

#### 8.2 图注扫描

用 grep 在骨架上找所有图注，识别图所在页和图标题：

```bash
grep -nE "^图\s*[0-9]+[\-．.][0-9]+\s+\S+|^Figure\s+[0-9]+[\-．.][0-9]+\s+\S+" "$TMP_DIR/skeleton.md"
```

骨架行格式形如 `[P.42] 图 3-2 系统架构图`，从中提取：
- `figure_id`：如 "3-2"
- `chapter`：3
- `page`：42
- `caption`：系统架构图

把所有图注存为列表 `figures = [{id, chapter, page, caption}, ...]`。

#### 8.3 整页渲染

**只渲染含图的页面**，不要全文渲染（80 页论文渲染慢且无意义）。

```python
import fitz, os
PDF = "/path/to/paper.pdf"
TMP_DIR = os.environ["TMP_DIR"]   # 来自步骤 1
FIGS_DIR = os.path.join(TMP_DIR, "figs")
os.makedirs(FIGS_DIR, exist_ok=True)

doc = fitz.open(PDF)
for fig in figures:
    page_idx = fig['page'] - 1  # 1-based → 0-based
    if 0 <= page_idx < len(doc):
        out = os.path.join(FIGS_DIR, f"page-{fig['page']:03d}.png")
        if not os.path.exists(out):  # 一页可能多图，避免重复渲染
            doc[page_idx].get_pixmap(dpi=150).save(out)
        fig['img_path'] = out
doc.close()
```

DPI 用 150（清晰度够辨识 UML 元素，体积可控）。

#### 8.4 分类与评估

对每张图：

1. **粗分类**：根据 `caption` 关键词初判类型
   - 含"用例" → use_case
   - 含"架构 / 体系结构 / 部署 / 拓扑" → architecture
   - 含"E-R / ER / 实体 / 表结构" → er
   - 含"类图 / Class" → class
   - 含"流程 / 活动 / Flow / Activity" → flowchart
   - 含"时序 / 顺序 / Sequence" → sequence
   - 含"状态 / State" → state
   - 其他 → unknown
2. **图像识别二次确认**：用 Read 工具读 `img_path`，让本 skill 的 Claude 直接看图判断
   - 类型与图注一致 → 用图注分类
   - 类型与图注不符（如图注写"系统架构"实际画的是流程图） → 记一条"图注与图内容不符"警告，按实际图类型评估
   - **PDF 渲染伪影禁报**：如果在渲染 PNG 上看到字符双重叠影（如"AAddPPllaann"）、字体回退、字符重影等显示异常，**不要**记入报告——这是 PyMuPDF 渲染管线的副作用而非论文问题，详见 `references/diagrams.md` 顶部「评估范围」段
3. **按 references/diagrams.md 对应类别评估**：
   - 通用前置：图注、章内编号连续性、正文引用、清晰度
   - 类别专项：DGM-1 ~ DGM-6
4. **4+1 视图特别处理**：DGM-2 走四层判定——(1) 关键词 grep 触发，未命中即跳过；(2) 声明强度 grep（"采用/参考/基于…4+1/Kruchten" 共现），未命中记失败模式 A；(3) 五视图覆盖度（标题锚点 + 内容信号双条件），缺失记 B；(4) 概念混用与图文一致性启发式，命中记 C / D。详见 `references/diagrams.md` 的 DGM-2 段

**输出格式**（每条问题）：

```yaml
- 图: 图3-2 系统架构图（P.42）
  类型: architecture（图注: 架构 → 实际: 用例图，类型不符）
  状态: ⚠️
  问题: actor 画成方框 / use case 画成菱形 / 缺 system boundary
  修改建议: 改用 stickman 表示 actor、椭圆表示 use case；为系统加矩形边界
```

合格的图只记一行 `- 图3-2 ✅`，不展开。

#### 8.5 关键图缺失检查

**仅工程型论文做此检查**。学术型跳过本小节。

按 references/diagrams.md 末尾"工程型论文的关键图缺失检查"的方法：用例图 / 系统架构图 / ER 或表结构图 至少应有两类。

完全缺 → 一条警告；缺一两类 → 对应警告。本检查即使 `VISION_CAPABLE=false` 也能做（基于图注关键词）。

#### 8.6 退化路径（VISION_CAPABLE=false）

模型不支持图像输入时：

1. 跳过 8.3 整页渲染、8.4 视觉评估
2. 仅做基于图注文本的检查：
   - 图注格式是否规范（"图 X-Y 标题"）
   - 章内图编号是否连续
   - 正文是否有"如图 X-Y 所示"的引用
   - 关键图是否缺失（基于图注关键词命中）
3. 在报告第七节顶部写明：

```
⚠️ 当前模型不支持图像输入，本节仅做基于图注文本的规范检查。
UML 图形元素（如 actor 是否画成 stickman、关系箭头方向是否正确等）建议人工抽查。
```

#### 8.7 收集结果

把本步骤产出的所有警告（无论来自视觉评估还是图注文本检查）暂存为 `diagram_warnings` 列表，供步骤 9 写入报告第七节。**本步骤的警告不计入第一节的"警告"统计**。

### 步骤 9：生成主报告

按下面格式拼装报告。**不要在对话里贴完整报告**，写到磁盘。

**报告路径顺延**（与步骤 1 的 `$TMP_DIR` 同样规则，不覆盖前次产物；主报告与文风副报告**配套**顺延，下标一致）：

```bash
PAPER_DIR=$(dirname "$PAPER_PDF")

# 主报告
REPORT_PATH="$PAPER_DIR/${PAPER_TITLE}-review.md"
i=1
while [ -e "$REPORT_PATH" ]; do
  REPORT_PATH="$PAPER_DIR/${PAPER_TITLE}-review-$i.md"
  i=$((i+1))
done
echo "REPORT_PATH=$REPORT_PATH"

# 文风副报告路径（与主报告共用同一下标，避免 review-2 配 style-5 这种错配）
suffix=$(echo "$REPORT_PATH" | sed -E "s|.*-review(-[0-9]+)?\.md|\1|")  # 取 -N 或空
STYLE_PATH="$PAPER_DIR/${PAPER_TITLE}-style${suffix}.md"
echo "STYLE_PATH=$STYLE_PATH"     # 步骤 11 写入文风+错别字详情用这个路径
```

命名约定：
- 首次：`<题目>-review.md`（+ `<题目>-style.md` 仅在步骤 11 执行时）
- 第 N 次（N ≥ 1）：`<题目>-review-N.md`（+ `<题目>-style-N.md`）

#### 报告模板

```markdown
# 本科论文自检报告 — YYYY-MM-DD

**论文**：<原 PDF 文件名>
**抽取题目**：《...》
**类别**：工程型 / 学术型（已确认）
**团队论文**：是 / 否
  - 团队项目：SELABS（仅 yes 时）
  - 本人模块：统计与安全管理模块（仅 yes 时）
**生成**：本报告由 undergrad-thesis-self-check skill 生成，仅供学生自查参考

---

## 一、首页摘要

| 项 | 数 |
|---|---|
| 🔴 红线命中 | X |
| ⚠️ 警告 | Y |
| ✅ 通过 | Z |
| 总条目 | 28（工程型）/ 26（学术型） |
| 📊 图表警告（步骤 8，不计入总警告） | K |

### 必须先修的红线项

[命中时逐条展开；未命中写"✅ 无红线命中"]

---

## 二、🔴 红线条目（工程 4 / 学术 3 条全列）

[逐条展开]

---

## 三、学术质量（10 条）

按 ❌ → ⚠️ → ✅ 顺序排列，警告与不合格在前。

[逐条展开]

---

## 四、结构内容（工程 10 / 学术 5 条）

[逐条展开]

---

## 五、格式细节（工程 4 / 学术 3 条）

[逐条展开]

---

## 六、参考文献问题（6 条 UND-ENG-F）

按 ❌ → ⚠️ → ✅ 顺序排列，警告与不合格在前。

- F-01 双向引用抽样
- F-02 编程书 / arXiv 上限
- F-03 英文 / 中文文献门槛 + 防伪
- F-04 [N] 数字编号 / 中括号位置 / et al. 用法
- F-05 非同行评议来源处理
- F-06 arXiv / 预印本条目著录格式（兼容 GB/T 7714 新旧国标）

[逐条展开]

---

## 七、软件工程图表规范性（步骤 8 产出，不计入红线/警告总数）

> 本节由步骤 8 生成。**所有问题不计入第一节的"警告"统计**，仅供修改参考。
> 模型不支持图像输入时，本节仅含基于图注的文本检查；UML 图形元素的规范性请人工抽查。

### 7.1 检测能力
- 模型图像识别:✅ 支持 / ❌ 不支持（已退化到文本检查）
- 图注扫描:识别 N 张图，覆盖第 X-Y 章
- 整页渲染:`$TMP_DIR/figs/` 子目录（如已生成）

### 7.2 各图评估

按章节顺序列出。合格图只记一行 `图 X-Y ✅`；问题图按 `图 / 类型 / 问题 / 修改建议` 展开。

### 7.3 工程型关键图缺失（仅工程型）

如全部到位写「✅ 用例图 / 系统架构图 / ER 或表结构图 三类齐全」；缺失则逐条列出。

### 7.4 小结

- 图表警告:共 K 条
- 图类型分布:用例图 a / 架构图 b / ER 图 c / 类图 d / 流程图 e / 其他 f
- 修改优先级:先处理"图注与图内容不符""关系箭头方向错"等语义问题，再处理样式细节

---

## 八、（仅学术型）实验与方法专项（5 条）

[仅学术型展开]

---

## 九、（仅团队论文）团队论文专项检查

[override 命中条目 + UND-TEAM-DECL 状态]

---

## 十、修改 checklist（待勾选）

- [ ] [UND-RED-2] 字数补到 15,000 以上
- [ ] [UND-ENG-A-07] 图 3-2 / 图 4-5 在正文未被引用，补"见图 X-Y"或删图
- [ ] [UND-ENG-F-05] 参考文献 [12] 为 CSDN 博客，建议替换为同行评议论文或加 [Z] 在线资源标记
- ...

<!-- 第十一节由可选的步骤 11（humanizer-zh + 错别字检查）追加生成；如学生跳过该步骤则不出现 -->
```

### 步骤 10：对话输出摘要

只贴：

```
本科论文自检完成。

报告路径：/Users/.../<题目>-review.md

类别：工程型（团队论文 — SELABS / 统计与安全管理模块）
统计：🔴 红线 1 条 / ⚠️ 警告 5 条 / ✅ 通过 28 条 / 共 34 条
📊 图表规范性：警告 K 条（不计入红线/总警告，仅供修改参考）

🔴 必须先修：
  1. [UND-RED-2] 正文字数约 12,800，未达 15,000 底线
     建议在第 3 章需求分析扩充约 2,200 字

详细分项与修改建议见报告。
```

**不输出**：
- 评议结论估计（本科版不做）
- 34 条逐条状态（避免刷屏）
- 独立的免责声明段（报告头部一句话即可）

### 步骤 11（可选）：文风与错别字检查

步骤 10 输出摘要后，**主动**询问用户是否进行 AI 文风检查与错别字检查。这一步会跑 humanizer-zh（github.com/op7418/Humanizer-zh）+ 错别字扫描。

#### 11.1 询问用户

按以下文案询问（必须把开销讲清楚，让学生有预期）：

```
是否对全文做 AI 文风痕迹与错别字检查？（可选）

说明：
- 工具：humanizer-zh（按 24 种 AI 写作模式扫描）+ 错别字/标点检查
- 流程：将正文按章节切块，仅扫正文章节（chN.txt），逐块检查
- 论文当前 {pages} 页，预估耗时约 5-15 分钟，会消耗较多 token
- 结果会写入独立副报告：<论文题目>-style.md，主报告会留摘要指针

输入 y 进行检查；输入 n 跳过（不会影响已有报告）。
```

**AUTO_MODE**：不询问，默认进行；如检测到 humanizer-zh 未安装，则跳过文风部分但仍执行错别字检查，并在第十一节顶部写明「未安装 humanizer-zh，本节仅含错别字检查；如需补充 AI 文风扫描请安装后重跑」。

#### 11.2 检测 humanizer-zh 是否可用

```bash
ls ~/.claude/skills/humanizer-zh/SKILL.md 2>/dev/null && echo "INSTALLED" || echo "MISSING"
```

如未安装，**人工模式**告知用户并停在这里：

```
未检测到 humanizer-zh skill。请先安装：

  npx skills add https://github.com/op7418/Humanizer-zh.git

安装并重启 Agent 会话后再次运行本科论文自检即可。
```

不要尝试 fallback 到内置规则——humanizer-zh 的 24 类模式是它的核心资产，本 skill 不重复实现。

**AUTO_MODE** 下未安装时不停止：跳过文风部分，仅做错别字检查，在第十一节顶部写明已跳过文风扫描及安装命令，让流程跑完。

#### 11.3 切块（lazy 切，仅本步骤需要时执行）

调用 `extract_chunks.py` 把 PDF 按章节切成独立 .txt：

```bash
TMP_DIR="<步骤 1 echo 出的绝对路径>"
python3 ~/.claude/skills/undergrad-thesis-self-check/scripts/extract_chunks.py \
  "$PAPER_PDF" \
  --out "$TMP_DIR/chunks"
```

产物：
- `$TMP_DIR/chunks/<range>.txt`：每个固定范围一份原文（如 `ch1.txt` / `abstract-cn.txt` / `references.txt`）
- `$TMP_DIR/chunks/chunks-index.json`：含 `chunks` 列表（每个文件的页码、字符数、章标题）+ `stats.split_failed` 标志

**切块结果分支**：

| `stats.split_failed` | `stats.body_chars` | 处理 |
|---|---|---|
| false | ≥ 5000 | ✅ 正常进入 11.4 |
| true | 任意 | ⚠️ 骨架降级：仅扫骨架可见的段落，副报告头部加警示「全文切分失败，文风检查质量受限」 |
| false | < 5000 | ⚠️ 同上，且额外提示用户检查 PDF 是否扫描版（无文本层） |

读 `chunks-index.json` 一次，缓存正文章节列表（id 以 `ch` 开头）。

#### 11.4 逐块检查

遍历正文章节 chunks。每个文本块**先错别字、后文风**两步做：

**(a) 错别字与标点检查**（由本 skill 的 Claude 直接读块文本判断，重点关注）：

- 同音字误用（"做/作"、"在/再"、"的/地/得"、"原/源"、"以/已"、"即/既"）
- 形近字（"末/未"、"己/已/巳"、"戊/戌/戍"、"撤/撒"、"暴/爆"）
- 专业术语前后写法不一致（"K8s / Kubernetes / k8s" 同一文中混用、"OpenTelemetry / Open-Telemetry" 等）
- 英文词中英标点错位（句号是 `.` 而段尾用了 `。`，或反之）
- 全角/半角混用（数字、字母、括号、引号）
- 引号成对错误（中文 `"` 配 `"`，英文 `"` 配 `"`，不要混用）
- 缺字、多字、重字（"的的"、"了了"、漏字时上下文不通）

每块产出列表：每条给 `原文片段 / 建议修正 / 章节定位`。

**(b) AI 文风检查**（调用 humanizer-zh skill）：

通过 Skill 工具调用：

```
Skill: humanizer-zh
args: 严格按你 SKILL.md 的 24 种 AI 模式扫描以下论文片段。
仅列出命中的模式，每条给出：模式编号+名称 / 原文片段（≤80 字）/ 建议改写 / 命中理由（一句话）。
不要改写整段，不要给概括性总评，只列条目。
来源：第 {chapter} 章，P.{p_start}-{p_end}

<文本块内容>
```

把每块的输出收集到内存中（标注其章节定位），不要在对话里逐块打印。

如某章字符数 > 12000，可在内存中再按"节"切（按 `§X.1` / `§X.2` 标记拆段），但**不要把切分结果写回 chunks/**——避免污染步骤 11.3 的产物。

#### 11.5 写入独立 style 副报告 + 在主报告里追加指针

文风+错别字结果**不要**追加到主 review 报告里——条目可达数十条，会冲淡主报告核心结论。改为：

**(a) 独立写到 `$STYLE_PATH`**（步骤 9 已确定路径，与主报告共用下标）：

```markdown
# 文风与错别字检查 — YYYY-MM-DD

**论文**：<paper.pdf 文件名>
**配套主报告**：`<论文题目>-review.md`（或 -review-N.md）
**工具**：humanizer-zh + 内置错别字扫描
**切分**：步骤 11.3 切出的 N 个正文 chunk，覆盖 P.X-Y

---

## 一、错别字与标点问题

按章节列出。每条格式：

- **[第 X 章 P.YY]** 原文：「……」 → 建议：「……」 — 类型：同音字 / 形近字 / 标点 / 全半角 / 引号开关 / 术语不一致 / 重字漏字

如全文未发现，写「✅ 未发现明显错别字与标点问题」。

## 二、AI 文风痕迹（humanizer-zh 24 类模式）

### 2.1 命中频次表

按模式编号汇总，给一张统计表：模式编号 / 模式名 / 命中次数 / 重灾章节。

### 2.2 按章节展开

每章一节，列出该章命中的所有条目，每条格式：

- **[模式 N — 模式名]** P.YY：原文「……」 → 建议改写「……」（命中理由：……）

如某章未命中，写「✅ 本章未见明显 AI 写作痕迹」。

## 三、小结与修改建议优先级

- 错别字与标点问题：共 X 条
- AI 文风痕迹：共 Y 条（涉及 Z 类模式）
- 重灾区：列出 3-5 个最值得优先重写的位置
- 修改建议：先处理高频命中模式（≥3 次同一模式），再处理零散错别字

> ⚠️ humanizer-zh 与错别字扫描均为辅助工具，可能漏检或误报。本副报告条目较多并不直接等于论文质量差——AI 痕迹常集中在过渡句、本章小结、通用宣告等"低信息密度"区域；建议先做高频区域集中重写，再做全文标点统一。
```

**(b) 在主 review 报告里追加第十一节**——只追加一段简短摘要：

```markdown

---

## 十、文风与错别字检查（可选）

> 详细列表已写入独立副报告：`<论文题目>-style.md`（或 -style-N.md）
> 检查时间：YYYY-MM-DD

**统计**：
- 错别字与标点：共 X 条
- AI 文风痕迹：共 Y 条（涉及 Z 类模式）
- 命中前 3 类模式：模式 N1（K1 处）/ 模式 N2（K2 处）/ 模式 N3（K3 处）
- 重灾区：……（3-5 个最值得优先重写的位置）

详细条目（含原文片段+建议改写）请打开 `$STYLE_PATH`。
```

#### 11.6 对话输出

只贴：

1. 错别字 X 条 / 文风痕迹 Y 条
2. 命中前 3 的 humanizer-zh 模式（如"模式 10 三段式 8 处 / 模式 4 宣传性 7 处 / 模式 7 AI 词汇 5 处"）
3. 两句话：
   - "主报告第十一节已加摘要指针：`$REPORT_PATH`"
   - "详细条目（含原文片段+建议改写）见副报告：`$STYLE_PATH`"

不要在对话里贴每一条原文片段——会刷屏。

## 红线触发后的特别提醒

报告顶部和对话里都要明确提醒：

> 🔴 命中 X 条红线。这些是评审"一票否决"或"严重影响通过"的项，建议优先修改完毕后再处理其他警告项。

## 检查方法的复用

reference 文件中每条 checklist 均自带 `检查方法` 段（含 grep 模板、章节定位、模块名提取等），无需外部依赖；如发现多条重复的检查动作，可在本节后续补充统一模板。

---
> Source: [dongshao/undergrad-thesis-self-check](https://github.com/dongshao/undergrad-thesis-self-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
