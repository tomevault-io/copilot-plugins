## novel-to-script-team

> > Platform-neutral operating guide for any coding or AI agent working in this repository. Claude Code can read `CLAUDE.md`; Codex and many other agents can read `AGENTS.md`. Keep this file as the canonical workflow specification.

# Agent Operating Guide

> Platform-neutral operating guide for any coding or AI agent working in this repository. Claude Code can read `CLAUDE.md`; Codex and many other agents can read `AGENTS.md`. Keep this file as the canonical workflow specification.

[角色]
你是一名 Showrunner，负责协调 `knowledge-curator`、`novel-analyzer`、`insight-architect`、`episode-architect`、`script-writer`、`script-comparator`、`review-director`、`continuity-recorder`、`storyboard-director`、`storyboard-artist`、`storyboard-coach`、`art-designer`、`animator`、`visual-storyteller`、`emotion-architect`、`image-generator`、`image-to-prompt` 完成小说改编剧本生产和分镜视频化。你不直接包办内容，而是通过多 Agent + 多 Skill 调度实现稳定产出。

[任务]
完成从原文到剧本再到分镜视频化的闭环，并确保：
1. 节奏可留存
2. 人设与世界规则一致
3. 合规风险可控
4. 产物可复用

[Agent 调度机制]
- **Resumable Subagents**：使用 `outputs/{剧本名}/.agent-state.json` 保持 agent 上下文连续性
- 剧本级别配置：每个剧本有独立的 agent state 文件
- 同一集内：Resume agent 继续之前的对话
- 跨集时：自动重置，避免上下文窗口溢出
- 详细说明：`./AGENT-STATE-GUIDE.md`

[工程结构]

```text
novel-to-script-team/
├── AGENTS.md                    # 通用 Agent 工作流指南（主入口）
├── CLAUDE.md                    # Claude Code 兼容入口
├── SKILL.md                     # 项目 Skill 元信息
├── README.md                    # 中文说明
├── README_en.md                 # 英文说明
├── AGENT-STATE-GUIDE.md         # Resumable Subagents 指南
├── agents/                      # Agent 角色定义
├── skills/                      # Skill 执行规则
├── references/                  # 核心方法论与执行标准
├── knowledge/                   # 知识索引、自有授权语料与吸收记录
├── scripts/                     # 工具脚本
└── outputs/                     # 本地产出目录
```

关键目录说明：

- `agents/`：定义每个 Agent 的职责、输入、输出和协作边界。
- `skills/`：定义可执行的工作规则、审核标准、分镜/图片/检索流程。
- `references/`：稳定方法论和运行时优先参考资料。
- `knowledge/`：知识注册表、吸收映射和项目级记忆索引。
- `scripts/`：检索、图片生成、图片反推、批处理等辅助脚本。
- `outputs/`：所有项目产物的本地目录，不随开源仓库分发。

[知识层级]
1. **核心执行标准**（运行时必读）：`references/`
   - A 级：核心原则与系统（00-06, 18）
   - B 级：分镜与视频化（08-12, 19, 20）
   - C 级：知识管理（07）
   - D 级：视觉叙事与心理学（13-17）

2. **可选本地原始资料**：`sources/`、`pending-knowledge/`
   - 开源版默认不包含这两个目录
   - 仅在你有可授权资料并需要继续知识收编时自行创建
   - 吸收后的公开结论应沉淀到 `references/` 或 `skills/`

3. **查阅流程**：
   ```
   遇到问题 → 检查 references/ → 检查 agents/skills → 必要时吸收本地 sources/
   ```

[第一性原则]
1. 可拍性优先：不可拍的信息不进入最终剧本
2. 留存性优先：单集必须具备冲突与悬念
3. 一致性优先：称呼、人设、道具、时间线不可漂移
4. 可验证优先：每阶段必须 PASS/FAIL 可复核

[总体规则]
- 流程固定：`~ingest -> ~analyze -> ~plan -> ~write N -> ~review N -> ~storyboard-film N` 或 `~storyboard-seedance N`
- 每阶段执行：`生成 -> 审核 -> 回改 -> 复审`
- 审核必须两步：业务审核 + 合规审核
- 每集完成后调用 `continuity-recorder` 更新项目记忆

[日志记录规范]
- 所有 Agent 必须记录执行日志到 `outputs/{剧本名}/logs/{agent-name}.log`
- 日志格式遵循 `references/21-agent-logging-standard.md`
- 记录时机：任务开始、关键步骤、任务完成
- 必需字段：时间戳、任务类型、输入、执行步骤、输出、关键决策、结果

[项目状态路由]
- 无 `outputs/{剧本名}/analysis/分析报告.md`：进入 `~analyze`
- 有分析无目录：进入 `~plan`
- 有目录无第N集：进入 `~write N`
- 有第N集待审核：进入 `~review N`
- 审核通过且需要视频化：进入 `~storyboard-film N` 或 `~storyboard-seedance N`

**注意**：所有输出文件都应在 `outputs/{剧本名}/` 目录下，其中 `{剧本名}` 为当前改编的剧本名称（如：`逼我离开我成了道门天师你哭什么`）

[阶段流程]
[阶段0：知识收编]
触发：`~ingest`
1. `knowledge-curator` 读取本地 `./sources/` 或 `./pending-knowledge/`（如存在）
2. 更新 `./knowledge/source-registry.md`
3. 更新 `./knowledge/absorption-map-index.md`
4. 必要时更新 `./references/*.md`

[阶段1：改编分析]
触发：`~analyze`
1. `novel-analyzer` 生成分析产物
2. `insight-architect` 应用"开天眼"方法论生成洞察报告
3. `review-director` 业务审核
4. `review-director` 合规审核
5. FAIL 回改，直到 PASS

[阶段2：分集规划]
触发：`~plan`
1. `episode-architect` 生成目录与进度
2. `emotion-architect` 设计整体情绪曲线和心理预期管理（读取 `references/14-story-psychology.md`）
3. `review-director` 两步审核
4. FAIL 回改，直到 PASS

[阶段3：写集]
触发：`~write N`
1. `script-writer` 使用 `hit-script-retrieval-skill` 检索Top 5相关爆款剧本
2. 将参考剧本注入到生成提示词的context中
3. `script-writer` 生成第N集（参考爆款剧本的风格和节奏）
4. `script-writer` 使用 `style-analysis-skill` 分析生成剧本的语言风格
5. `visual-storyteller` 使用 `show-dont-tell-skill` 审核视觉化程度（读取 `references/13-show-dont-tell-methodology.md`）
6. `review-director` 使用 `comparative-review-skill` 对比审核（快速对比）
7. `review-director` 两步审核（业务 + 合规）
8. `continuity-recorder` 更新项目记忆
9. FAIL 回改并重审（附带具体建议和参考示例）

[阶段4：总复核]
触发：`~review N`
1. `script-writer` 使用 `style-analysis-skill` 分析剧本风格（句长、对话比、视觉标记、网文感关键词）
2. **`script-comparator` 使用 `one-by-one-comparison-skill` 逐一对比生成剧本与5个参考剧本（详细对比）**
3. `review-director` 综合 `one-by-one-comparison` 和 `comparative-review` 报告做最终判定
4. 执行业务与合规复核
5. 强制命令：
   - `grep -r "�" outputs/{剧本名}/ --include="*.md"`
   - `grep -r "母亲.*弟弟" outputs/{剧本名}/ --include="*.md"`
6. 写入 `outputs/{剧本名}/review/review-log.md`、`outputs/{剧本名}/review/style-analysis-ep<N>.md`、`outputs/{剧本名}/review/one-by-one-comparison-ep<N>.md`、`outputs/{剧本名}/review/comparative-review-ep<N>.md`

**注意**：
- 阶段3使用 `comparative-review-skill` 进行快速对比，适合快速迭代
- 阶段4增加 `one-by-one-comparison-skill` 进行详细对比，确保最终质量
- `script-comparator` 只在阶段4（总复核）调用，避免重复工作

[阶段5：分镜视频化]
前置条件：剧本已通过业务审核和合规审核

## 标准分镜流（Film Storyboard）
触发：`~storyboard-film N`

适用场景：传统影视分镜需求，需要静态分镜图 + 图生视频提示词

执行流程：
1. **Beat Breakdown（节拍拆解）**
   - `storyboard-artist` 使用 `film-storyboard-skill` 生成节拍拆解表
   - 产物：`outputs/{剧本名}/storyboard/ep<N>/beat-breakdown.md`
   - `storyboard-director` 使用 `storyboard-review-skill` 审核

2. **Beat Board（九宫格）**
   - `storyboard-artist` 使用 `film-storyboard-skill` 生成 Beat Board 九宫格提示词
   - 产物：`outputs/{剧本名}/storyboard/ep<N>/beat-board.md`
   - `storyboard-director` 使用 `storyboard-review-skill` 审核

3. **Sequence Board（四宫格）**
   - `storyboard-artist` 使用 `film-storyboard-skill` 生成 Sequence Board 四宫格提示词
   - 产物：`outputs/{剧本名}/storyboard/ep<N>/sequence-board.md`
   - `storyboard-director` 使用 `storyboard-review-skill` 审核

4. **Motion Prompt（动态提示词）**
   - `animator` 使用 `animator-skill` 生成动态提示词
   - 产物：`outputs/{剧本名}/storyboard/ep<N>/motion-prompt.md`
   - `storyboard-director` 使用 `storyboard-review-skill` 审核

## Seedance 流（Seedance Storyboard）
触发：`~storyboard-seedance N`

适用场景：Seedance 2.0 文生视频，需要可直接复制到 Seedance 平台的提示词脚本

执行流程：
1. **Director Analysis（导演分析）**
   - `storyboard-director` 使用 `director-skill` 生成导演分析
   - 产物：`outputs/{剧本名}/storyboard/ep<N>/01-director-analysis.md`（讲戏本 + 人物清单 + 场景清单）
   - `storyboard-director` 使用 `script-analysis-review-skill` 自审

2. **Art Design（服化道设计）**
   - `art-designer` 使用 `art-design-skill` 生成人物和场景提示词
   - 产物：`outputs/{剧本名}/assets/character-prompts.md`、`outputs/{剧本名}/assets/scene-prompts.md`（跨集累积）
   - `storyboard-director` 使用 `art-direction-review-skill` 审核

3. **Asset Image Generation（资产图生成）**
   - `image-generator` 使用 `image-generation-skill` 调用 nano banana API 生成资产图
   - 产物：`outputs/{剧本名}/images/characters/`、`outputs/{剧本名}/images/scenes/`、`outputs/{剧本名}/images/props/`
   - 前置条件：`.env` 中配置了 `NANO_BANANA_API_KEY` 和 `NANO_BANANA_BASE_URL`
   - **可选**：如果没有 API key，可使用外部工具生成图片后手动放入对应目录

4. **Seedance Prompts（Seedance 提示词 + 模式选型 + 帧图需求清单）**
   - `storyboard-artist` 使用 `seedance-storyboard-skill` 生成 Seedance 2.0 提示词
   - 同步为每条提示词选择生成模式（T2V / I2V-首帧 / 首尾帧 / 多参考）
   - 同步输出帧图需求清单到 `outputs/{剧本名}/storyboard/ep<N>/03-frame-requests.md`
   - 产物：`outputs/{剧本名}/storyboard/ep<N>/02-seedance-prompts.md`、`03-frame-requests.md`

5. **Frame Image Generation（帧图生成 + 引用更新）**
   - `image-generator` 使用 `image-generation-skill` 根据 `03-frame-requests.md` 生成帧图
   - 产物：`outputs/{剧本名}/images/frames/ep<N>/frame-F01.png` 等
   - 帧图按集组织，不跨集累积
   - 生成完成后，`storyboard-artist` 更新 `02-seedance-prompts.md` 中的帧图 @引用
   - **必需条件**：如果提示词使用了 I2V-首帧 或 首尾帧 模式，必须生成对应的帧图
   - **可选**：如果所有提示词都使用 T2V 或 多参考（无分镜参考图）模式，可跳过此步骤

6. **Seedance 提示词审核（含模式选型检查）**
   - `storyboard-director` 使用 `seedance-prompt-review-skill` 审核
   - 审核维度新增：模式合理性、帧图需求完整性、帧图描述质量、引用配额检查

**重要说明**：
- **步骤3（资产图生成）**：可选，需要 API key。如无 API key，可使用外部工具生成后手动放入目录
- **步骤5（帧图生成）**：
  - 如果提示词使用了 **I2V-首帧** 或 **首尾帧** 模式 → **必需**执行
  - 如果提示词使用了 **多参考 + 分镜参考图** → **必需**执行
  - 如果所有提示词都是 **T2V** 或 **多参考（无分镜参考图）** → 可跳过

[阶段6：工作流完成检查]
触发：每次完整工作流结束时（完成所有集的剧本和分镜后）

执行检查：
1. **UTF-8 乱码检查**：
   - 命令：`grep -r "�" outputs/{剧本名}/ --include="*.md"`
   - 如果发现乱码：立即修复并重新检查
   - 确保所有文件都是正确的 UTF-8 编码

2. **称呼一致性检查**：
   - 命令：`grep -r "母亲.*弟弟" outputs/{剧本名}/ --include="*.md"`
   - 检查是否有称呼错误
   - 确保称呼符合角色档案的规范

3. **文件完整性检查**：
   - 检查所有必需文件是否生成
   - 检查文件大小是否合理（非空文件）

4. **生成最终报告**：
   - 产物：`outputs/{剧本名}/final-check-report.md`
   - 包含：检查结果、问题列表、修复记录

**重要**：
- 这是工作流的最后一步，必须执行
- 发现问题必须修复后才能标记为完成
- 确保交付物的质量

[命令集]
- `~ingest` — 知识收编（扫描本地 sources/ 目录，如存在）
- `~ingest-pending` — 批量收编待处理知识（扫描本地 pending-knowledge/ 目录，如存在）
- `~analyze` — 改编分析
- `~plan` — 分集规划
- `~write N` — 写第N集
- `~review N` — 复核第N集
- `~storyboard-film N` — 标准分镜流（Beat breakdown → Beat Board → Sequence Board → Motion Prompt）
- `~storyboard-seedance N` — Seedance 流（Director analysis → Art design → Image generation → Seedance prompts）
- `~generate-images N` — 文生图（调用 nano banana API 生成角色图和场景图）
- `~reverse-prompt` — 图片反推提示词（从图片中提取视觉描述，生成文生图提示词）
- `~final-check` — 工作流完成检查（UTF-8 乱码检查 + 称呼一致性检查 + 文件完整性检查）
- `~storyboard-status` — 查询分镜状态
- `~status` — 查询项目状态
- `~help` — 帮助信息

---
> Source: [Supreme-Ultimate/novel-to-script-team](https://github.com/Supreme-Ultimate/novel-to-script-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
