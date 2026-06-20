## speech-paper-wechat

> 每日语音论文速递-公众号版。搜索当天 arXiv 语音与音频领域新论文，先下载候选论文 PDF 到 /tmp，再逐篇阅读 PDF/首页文本并把分类、作者机构、评分和评语追加写入 JSONL，最后统一生成 reviewed.json、精选版和全量版公众号草稿；范围包含语音大模型、语音前端、说话人任务、空间音频、长音频理解，以及音乐信息检索/歌声转换/音乐生成等音乐相关方向；使用固定分类 taxonomy、方向/序号/论文/评分/关键词总览表、emoji 小标题和内容感知封面图，并通过仓库内置微信发布脚本一次性推送为同一个多图文草稿。


# 每日语音论文速递 — 公众号版

搜索当天 arXiv 语音与音频领域新论文 → 下载 PDF 到 `/tmp` → 逐篇读 PDF 并追加 JSONL → 统一生成公众号 markdown（默认双文章：精选版 + 全量版）→ 自动补封面 → 推送草稿箱。

## 什么时候用

当用户有以下意图时直接触发：
- “生成今天的语音论文公众号草稿”
- “把今天的语音论文速递推到草稿箱”
- “今天的语音论文发公众号”
- “跑每日语音论文公众号”

## 先读的参考文件

如需具体命令与字段，读取：
- `references/workflow.md`

## 路径约束

- 本仓库根目录：`speech-paper-wechat`
- 下文命令里的 `REPO_ROOT` 默认指向本仓库根目录
- Markdown 生成脚本：`$REPO_ROOT/scripts/build_wechat_markdown.py`
- arXiv 日期块抓取脚本：`$REPO_ROOT/scripts/collect_arxiv_recent.py`
- 封面 prompt 脚本：`$REPO_ROOT/scripts/make_cover_prompt.py`
- 默认封面图生成脚本：`$REPO_ROOT/scripts/image/generate_nanobanana.py`
- 微信发布脚本目录：`$REPO_ROOT/scripts/wechat`
  - `scripts/wechat/wechat-api.ts` 已包含 `--multi-manifest` 支持
  - `scripts/wechat/vendor/baoyu-md` 和 `scripts/wechat/vendor/baoyu-chrome-cdp` 已随仓库内置
- 微信凭据只允许放在本地未跟踪文件：`$REPO_ROOT/scripts/wechat/.baoyu-skills/.env`
- 图片 API Key 只允许来自环境变量 `NANOBANANA_API_KEY`、`~/.nanobanana/config.json` 或运行时参数
- 所有中间产物写到：`/tmp/papers_YYYYMMDD/`
- 不要把运行产物、PDF、封面图、真实 `.env` 写回仓库提交

## 固化后的执行顺序

### 1) 获取当天候选论文

默认从 arXiv recent 页面抓取当天日期块：

```bash
REPO_ROOT="/path/to/speech-paper-wechat"

python3 "$REPO_ROOT/scripts/collect_arxiv_recent.py" \
  --date-heading "Wed, 20 May 2026" \
  --output /tmp/papers_20260520 \
  --download-pdf \
  --extract-text
```

强约束：只解析目标日期的 `<h3>...showing ... entries</h3>` 块，不解析 replacement 或其它日期块。

### 2) 筛选论文

剔除：
- 纯图像/视频音频边缘论文
- 纯理论声学且与语音/音频系统无关

保留：
- ASR / TTS / Speech LLM / codec / diarization
- speech enhancement / separation / beamforming / dereverb
- speaker verification / spoofing / voice conversion
- audio system / hearables / spatial audio / room acoustics
- long-form audio understanding / spoken summarization / clinical conversation audio
- **音乐相关方向也保留**：music information retrieval、singing voice conversion、歌声合成、symbolic music generation、music tagging / classification、music-grounded audio intelligence 等

### 3) 收录规则（强约束）

除了上面明确剔除的无关稿件外，**当天两个源里的所有相关论文都要收录**。

也就是说：
- 不是只挑 4–8 篇
- 不是只写高分论文
- 而是 **全收录、全精读、全入稿**


### 4) 下载 PDF 与逐篇结构化整理

**强约束：先下载到 `/tmp/papers_YYYYMMDD/`，再逐篇处理。不要先凭摘要批量分类。**

推荐目录：
- `/tmp/papers_YYYYMMDD/pdf/`：PDF
- `/tmp/papers_YYYYMMDD/firstpage/`：PDF 首页或前两页文本
- `/tmp/papers_YYYYMMDD/fulltext/`：全文抽取文本（能抽就抽）
- `/tmp/papers_YYYYMMDD/reviewed.jsonl`：逐篇追加结果
- `/tmp/papers_YYYYMMDD/reviewed.json`：最终数组版，供 markdown 脚本读取

逐篇流程：
1. 下载该论文 PDF
2. 用 `pdftotext` 或可用 PDF 解析工具抽取首页/全文
3. 先从 PDF 解析文本里取作者与机构
4. 根据论文主贡献确定固定 `direction` 标签
5. 写完整结构化对象，**立即追加一行 JSON 到 `reviewed.jsonl`**
6. 再处理下一篇

默认要求：**所有通过过滤的论文都要读取 PDF 首页；正文判断尽量读全文或接近全文，不要只看 abstract。**

追加 JSONL 的好处是：过程可恢复、可检查、分类不靠最后统一回忆。每篇只追加一次；如果要修某篇，重建最终 `reviewed.json` 前先去重保留最后一次同 `arxiv_id` 记录。

### 4.0) 固定分类 taxonomy（强约束）

`direction` 只能从下面固定集合中选择，不要临时发明大类：

- `语音大模型与生成`：TTS、speech LM、spoken dialogue、audio codec / tokenizer、voice conversion、speech generation、LLM-based enhancement
- `ASR与说话人`：ASR、diarization、speaker verification / recognition、accent / dialect、spoken language understanding
- `语音前端与声学系统`：enhancement、separation、dereverb、AEC、beamforming、array / microphone、room acoustics、spatial audio、underwater / environmental acoustic systems
- `音频安全与评测`：deepfake / spoofing detection、watermarking、fairness、benchmark、XAI、quality / appropriateness evaluation、clinical / biomarker screening
- `音乐与声音创作`：MIR、music generation / editing、singing voice、timbre transfer、symbolic music、music recommendation、sonification
- `多模态音视频理解`：audio-visual LLM、AVQA、audio-driven retrieval/search、audio-video generation/synchronization
- `其他相关音频`：确实相关但不落入以上类别的音频论文；使用时要在 `review` 里说明为什么没有更合适的大类

分类原则：
- 按论文**主贡献**分类，不按 arXiv subject 或模型组件分类
- TTS / speech generation 不再简单归到“语音大模型”；使用 `语音大模型与生成`
- deepfake 检测、公平性、watermark、评测基准统一进 `音频安全与评测`
- beamforming、声场重建、房间声学、环境声学系统统一进 `语音前端与声学系统`
- 纯音乐理论只有在和音频/音乐信息处理/创作工具有明确关系时才收录，否则剔除

### 4.1) 作者与机构信息（发布前强约束）

**作者与机构信息必须在推送前补齐，不能留空，不能写占位文本，不能带着“待补充”进入草稿箱。**

默认流程要求：
1. **优先使用解析后的 PDF 首页/前两页文本**
2. 如果 PDF 首页没有机构，再看 PDF 全文开头、脚注、最后的 author information / acknowledgements
3. 如果 PDF 确实只列作者不列机构，可写 `姓名（论文未列机构）`，并只在这种情况下允许
4. 不默认联网搜索机构；只有 PDF 解析完全不足且用户明确需要发布时，才补抓 arXiv abs 或其他学术元数据源
5. **只有作者与机构补齐后，才能进入 markdown 生成与公众号推送步骤**

禁止做法：
- `待补充`
- `作者信息缺失`
- `作者A, 作者B 等`
- 只写机构，不写作者
- 只写作者，不写机构（除非论文确实未提供机构）

如果确实无法可靠获取作者/机构信息：
- **不要推送**
- 明确向用户报告卡在哪一篇
- 保留中间产物，等待人工确认或补齐

每篇整理成一个 JSON 对象，至少包含这些字段：
- `kept`
- `title`
- `direction`
- `score`
- `novelty_score`
- `impact_score`
- `evidence_score`
- `audience_fit_score`
- `authors_org`
- `abs_url`
- `pdf_url`
- `code_url`
- `summary`
- `review`
- `architecture`
- `innovation`
- `training`
- `results`
- `why`

### 4.2) 评分与毒舌点评口径（强约束）

**默认按 ML 顶会 reviewer 视角打分，不按“arXiv 看起来还行”或“工程做完了”给高分。**

参考口径：
- 以 `NeurIPS / ICML / ICLR` 主会审稿标准为默认标尺
- 语音任务可结合 `ACL / EMNLP / ICASSP / Interspeech` 语境理解，但**分数尺度仍按 ML 顶会标准校准**
- 优先看：新意、问题重要性、证据强度、泛化价值、分析深度
- 明确压低对“工程堆料、训练更大、模块拼装、只刷熟数据集”的默认评价

默认分数校准：
- `9-10`：极少出现；通常得是当天最强几篇里最像顶会强 accept / spotlight 的稿子
- `8`：明确强于平均 accepted paper；有清楚新意，实验也站得住
- `7`：可以算不错，但仍有明显短板；更像 strong accept 边缘，不是随手就给
- `6`：合格可读，但偏 incremental、证据一般，或贡献点较窄；**这应该是很多“还行”论文的默认落点**
- `5`：borderline reject；点子旧、分析浅、实验支撑不够，或者只是在熟套路里微调
- `4` 及以下：问题价值弱、方法说服力差、实验不可信，或整体完成度不足

额外约束：
- **不要把分数堆在 `7-9`。** 如果你发现大多数稿子都被打成高分，说明标尺放水了，要主动下调
- **`8+` 必须稀缺。** 一天里没有 `8` 分论文是正常的
- **`7` 不是“还不错”的口头鼓励分，而是接近顶会 strong accept 的分**
- `score` 默认等于 `novelty_score + impact_score + evidence_score + audience_fit_score`
- 四个子分必须真的拆开打，不要机械平均，更不要先拍脑袋给总分再倒推

`review` 字段的写法要求：
- 口吻像顶会 reviewer 的内部评语：直接、具体、专业
- 优先指出：贡献是否成立、基线是否公平、ablation/泛化/统计显著性是否足够、是否只是系统堆料
- 不要泛泛地夸“很有意思”“工程完整”“效果不错”
- 如果给了高分，必须明确说清楚为什么它配得上高分

`architecture` / `innovation` / `training` / `results` 的展开度要求：
- **`🔧 技术方案` 默认比 `📌 简介` 更展开**，不要只写一句“用了某某 backbone + 某某模块”
- `architecture` 至少交代：主干结构、输入输出或信息流、关键模块怎么接
- `innovation` 要说清：相对强 baseline 到底改了哪里，为什么可能有效，而不是只重复论文标题
- `training` 尽量补充：训练范式、数据来源或规模、损失设计、蒸馏/对齐/多阶段训练、推理或系统约束；不要只写“预训练+微调”
- **`📊 实验结果` 默认也要更扎实**，至少尽量覆盖：主 benchmark/真实场景、对比对象、核心数字增益、关键 ablation、泛化/鲁棒性观察、失败点或局限中的 3 项
- 如果论文实验本身很薄，就直接指出薄在哪里，不要把这段也写得很空

其中 `authors_org` 是**强约束字段**：
- 不能写“作者1, 作者2 等”
- 不能只写机构名
- 必须尽量写全作者
- 推荐格式：`姓名（机构）；姓名（机构）；姓名（机构）`
- 如果作者很多，也优先全写；确实超长时再做压缩，但默认不要省略成“等”

将结构化结果写到：
- 逐篇追加：`/tmp/papers_YYYYMMDD/reviewed.jsonl`
- 最终数组：`/tmp/papers_YYYYMMDD/reviewed.json`

### 5) 生成公众号 markdown

使用自带脚本：

```bash
REPO_ROOT="/path/to/speech-paper-wechat"

python3 "$REPO_ROOT/scripts/build_wechat_markdown.py" \
  --input /tmp/papers_YYYYMMDD/reviewed.jsonl \
  --output /tmp/papers_YYYYMMDD/wechat-post.md \
  --date YYYY-MM-DD
```

脚本可直接读取 `reviewed.jsonl`，会按 `arxiv_id` 去重并保留最后一次记录；如果已经生成了 `reviewed.json`，也可以继续传 JSON 数组文件。

脚本现在默认会生成三份产物：
- `/tmp/papers_YYYYMMDD/wechat-post.md`：**精选版**（按 rubric 选 1-4 篇）
- `/tmp/papers_YYYYMMDD/wechat-post-all.md`：**全量版**（当天所有相关论文）
- `/tmp/papers_YYYYMMDD/wechat-post-multi.json`：多文章草稿 manifest，供微信发布脚本一次性发两篇

精选版默认 rubric：
- 新意 0–3
- 影响力 0–3
- 证据强度 0–2
- 受众匹配度 0–2

解释口径：
- `7`：接近 ML 顶会 strong accept，已经不算宽松分
- `8+`：默认视为稀缺高分

总分 **≥7** 才进入精选；若满足条件太多，则按总分排序取前 **1-4 篇**；不足则宁缺毋滥，不硬凑。

两个版本都会带新的 **📋 总览表**，列固定为：**方向 / 序号 / 论文 / 评分 / 关键词**。

其中：
- 序号按方向内部分组编号，不是全局混排
- 评分统一写成 `⭐ x/10`
- 关键词优先从结构化字段里取；没有时再用方向兜底
- 精选版额外写明精选规则

### 6) 排版细节

- 元信息区（评分 / 作者机构 / 论文链接 / PDF / 代码）统一使用 markdown 列表，不要依赖行尾两个空格做换行
- 因为微信 HTML 转换有时会吞掉软换行，列表最稳
- 如果某篇论文没有代码链接，明确写“暂无”
- 作者信息默认写全，优先显示为“姓名（机构）；姓名（机构）”
- 总览表用于快速扫描，不替代正文全收录
- **默认正文字号固定为 14px**，其它结构不变
- **总览表字号默认 13px**，让它更像高层摘要，不抢正文注意力
- 正文固定使用带 emoji 的小标题：`📌 简介` / `☠️ 毒舌点评` / `🔧 技术方案` / `📊 实验结果` / `💡 为什么值得看`
- 也就是说：标题层级、信息结构、论文分块都保持原样；正文与元信息做一档轻微缩小，而总览表再额外缩小两档

### 7) 自动补封面图（nanobanana + 内容感知封面）

如果用户没提供封面图，默认自动生成。

优先使用 `nanobanana`。

#### 前置要求

默认假设以下环境已就绪：
- `$REPO_ROOT/scripts/image/generate_nanobanana.py` 已存在
- `python3` 可用
- 已安装依赖：`pip install httpx`
- 默认模型固定为 `gemini-3.1-flash-image-preview`
- 以后执行本 skill 时，**一律优先使用 `gemini-3.1-flash-image-preview` 作为默认出图模型**
- **不要自动切回旧模型**；只有在用户明确同意时，才允许改用其他模型或其他出图方案
- 已通过以下任一方式配置 API Key：
  - `~/.nanobanana/config.json`
  - `NANOBANANA_API_KEY` 环境变量
  - `--api-key` 参数

#### 内容感知封面规则（默认启用）

封面图不再只是固定“语音科技风”，而是要**结合当天论文内容**生成。

默认策略：
- 保持栏目统一气质：**像素风格微信公众号封面图，宽幅横版构图，适合文章头图，复古 8-bit / 16-bit 像素艺术风格**
- 画面简洁但有细节，主体突出，围绕文章主题进行场景化设计，具有故事感和视觉焦点
- 背景有层次，色彩鲜明但不过于杂乱，不要预留标题留白，画面尽量铺满
- 整体干净、吸睛、适合社交媒体传播，现代像素插画质感
- 同时读取当天 **Top1 / Top3 论文主题**，将其转成封面视觉元素

优先级：
1. 取评分最高论文作为主视觉线索
2. 取前 3 篇论文作为辅助视觉线索
3. 保留“语音论文速递”栏目统一视觉母版

#### 主题映射示例

- open-ear ANC / hearables / 空间音频
  - 智能眼镜、声波抑噪、空间声场、环绕波束
- diarization / 多说话人 / 长音频理解
  - 多轨波形、对话气泡、分离的语音流、摘要结构
- brain-to-speech / BCI
  - 脑机接口、神经信号、语音重建、神经电路纹理
- singing voice / music generation / MIR
  - 音符、歌声频谱、旋律线、舞台感但不过度花哨
- source separation / audio frontend
  - 分离声源、频谱切片、阵列麦克风、降噪纹理

#### 推荐 prompt 生成方式

在生成封面前，先根据当天 Top 论文动态拼接 prompt。结构建议如下：

```text
为微信公众号文章《语音论文速递》生成一张内容感知封面图。
要求：像素风格微信公众号封面图，宽幅横版构图，适合文章头图，复古 8-bit / 16-bit 像素艺术风格。
画面简洁但有细节，主体突出，围绕当天重点论文主题进行场景化设计，具有故事感和视觉焦点。
背景有层次，色彩鲜明但不过于杂乱，不要预留标题区，不要留白，画面尽量铺满，但仍然保持主体清晰。
整体干净、吸睛、适合社交媒体传播，现代像素插画质感。
结合当天重点论文主题：{根据 Top1/Top3 自动填写，例如 open-ear 智能眼镜降噪、多说话人 diarization、长音频摘要、歌声转换、脑机语音等}。
把这些论文主题转化为抽象但可感知的视觉元素，不要做成拼贴海报，不要堆太多元素。
不要真实人物，不要任何文字，不要汉字，不要英文字母，不要数字，不要 Logo，不要水印。
封面比例默认使用 2.35:1。
```

#### 实际生成命令模板

```bash
REPO_ROOT="/path/to/speech-paper-wechat"

python3 "$REPO_ROOT/scripts/make_cover_prompt.py" \
  --input /tmp/papers_YYYYMMDD/reviewed.jsonl \
  --output /tmp/papers_YYYYMMDD/cover_prompt.txt

python3 "$REPO_ROOT/scripts/image/generate_nanobanana.py" \
  -p "$COVER_PROMPT" \
  -ar 21:9 \
  -s 2K \
  -o /tmp/papers_YYYYMMDD/cover.png
```

#### 失败回退

- 如果 `nanobanana` 未安装、API Key 缺失或脚本失败：
  1. 明确告诉用户卡点
  2. 可建议用户先配置 `nanobanana`
  3. 默认先重试 `gemini-3.1-flash-image-preview`，不要自动切回旧模型
  4. 不要静默改成固定封面
  5. 只有在用户明确允许时，才退回别的图像生成方案

#### 输出路径

- 默认封面输出到：`/tmp/papers_YYYYMMDD/cover.png`
- 如果输出文件扩展名与真实格式不一致，后续上传前应检查并修正

### 8) 推送公众号草稿箱

**前置检查（强约束）**：
1. 检查 `reviewed.json` 中所有保留论文的 `authors_org` 已真实补齐
2. 检查 `$REPO_ROOT/scripts/wechat/vendor/baoyu-md` 目录存在
3. 检查 `$REPO_ROOT/scripts/wechat/.baoyu-skills/.env` 已由 `.env.example` 复制并填写，不能提交真实值

如果任一论文出现以下情况，**禁止推送**：
- 空字符串
- `待补充`
- `作者信息缺失`
- `unknown`
- 其他明显占位文本

#### 推送命令

```bash
REPO_ROOT="/path/to/speech-paper-wechat"

cd "$REPO_ROOT/scripts/wechat"
npx -y bun ./wechat-api.ts /tmp/papers_YYYYMMDD/wechat-post.md \
  --author "JusperLee" \
  --cover /tmp/papers_YYYYMMDD/cover.png \
  --multi-manifest /tmp/papers_YYYYMMDD/wechat-post-multi.json
```

当前默认行为：**同一次草稿里发布两篇文章**
- 第 1 篇：精选版（1-4 篇）
- 第 2 篇：全量版（当天所有相关论文）

> **注意**：本仓库内置的 `scripts/wechat/wechat-api.ts` 已包含 `--multi-manifest` 支持，不需要额外补丁。

### 9) 异常处理

- `40164 invalid ip`：告诉用户把当前出口 IP 加入公众号白名单，然后重试
- `No cover image`：先检查 `nanobanana` 是否生成成功，再重试
- `unsupported file type`：优先检查封面是否是真实可上传的 PNG/JPG，不要只改扩展名糊弄微信
- `Cannot find package 'baoyu-md'`：vendor 目录丢失，按下方「已知问题与修复」步骤恢复
- `Format mismatch: cover.png declared as image/png, actual image/jpeg`：nanobanana 输出的封面实际是 JPEG 但存为 .png，微信会警告但仍接受，不影响发布
- API 不可用：保留 `/tmp/papers_YYYYMMDD/wechat-post.md`，通知用户稍后重试
- **作者/机构缺失**：不要推送，先补齐 `authors_org`，补不齐就明确报错给用户

## 已知问题与修复

### 1) vendor 依赖丢失（baoyu-md / baoyu-chrome-cdp）

**症状**：推送时报 `Cannot find package 'baoyu-md'`

**原因**：仓库内置的 `scripts/wechat/vendor/` 被删掉，或 `node_modules/` 状态损坏，导致 bun 无法解析 `file:./vendor/baoyu-md` / `file:./vendor/baoyu-chrome-cdp` 依赖。

**修复步骤**：

```bash
REPO_ROOT="/path/to/speech-paper-wechat"
SCRIPTS_DIR="$REPO_ROOT/scripts/wechat"

cd "$SCRIPTS_DIR" && rm -rf bun.lock node_modules
npx -y bun install
```

**预防**：每次新会话推送前，先检查 `vendor/baoyu-md` 是否存在：

```bash
REPO_ROOT="/path/to/speech-paper-wechat"

ls "$REPO_ROOT/scripts/wechat/vendor/baoyu-md/package.json" 2>/dev/null || echo "MISSING"
```

### 2) --multi-manifest 失效

**症状**：传了 `--multi-manifest` 参数但草稿箱只出现一篇文章

**原因**：仓库里的 `scripts/wechat/wechat-api.ts` 被外部覆盖，或本地修改被回退。

**检测**：

```bash
REPO_ROOT="/path/to/speech-paper-wechat"

grep -c "multiManifest\|multi-manifest\|Multi-article manifest" "$REPO_ROOT/scripts/wechat/wechat-api.ts"
```

### 3) 封面格式 mismatch

**症状**：`Format mismatch: cover.png declared as image/png, actual image/jpeg`

**原因**：封面生成脚本有时会输出 JPEG 数据但文件名仍是 `.png`。

**影响**：微信 API 会警告但仍接受上传，不影响发布；如需消除警告，可在生成后检查真实格式并重命名。

## 写作风格要求

- 全程中文，论文标题保留英文
- 口吻：毒舌但专业，别写空话；**默认参考 ML 顶会审稿视角，不要用行业媒体吹稿口吻**
- 真正值得看的放前面
- 优先强调：是否解决真实问题、是否有可信实验、是否只是比赛/拼装工程
- 不要因为“作者强 / 参数大 / 任务热”就自动高分，先看贡献和证据
- 默认接受“多数论文分数不高”这件事，宁可刻薄一点，也不要失去区分度
- 标题别太长，适合公众号草稿箱
- 排版默认采取“**正文 14px，别的不变**”策略，不要擅自改成更大字版或重排结构，除非用户明确要求
- 其中**总览表默认 13px**，作为高层摘要区呈现

## 最终回复用户

成功时至少汇报：
- 标题
- 是否成功推草稿箱
- media_id

失败时明确卡点和下一步动作，不要只说“失败了”。

---
> Source: [JusperLee/speech-paper-wechat](https://github.com/JusperLee/speech-paper-wechat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
