## bbdog

> - **洗稿**（launder）：复刻已有短剧/剧本，AI 改写 + 重新生成资产

# bbdog — AI 短剧制作工具链（Agent-First CLI）

## 项目定位

AI 短剧制作工具链，支持三种模式：
- **洗稿**（launder）：复刻已有短剧/剧本，AI 改写 + 重新生成资产
- **原创**（original）：从一句话 logline 生成完整剧本
- **改编**（adapt）：小说/长文改编为短剧剧本

设计理念：**Agent-First**，CLI 输出结构化 JSON + nextStep 引导，Claude Code 作为 UI 层驱动完整流程。

## CLI 命令结构（扁平化）

```bash
# 初始化 & 配置
bbdog init                          # 初始化项目目录
bbdog config set-key <key>          # 设置 API key
bbdog config use <type> <model>     # 切换模型
bbdog config models                 # 查看可用模型
bbdog config show                   # 查看当前配置
bbdog status                        # 项目状态摘要（含 nextStep 建议）

# 剧本生成
bbdog create <logline>              # 一句话原创剧本
bbdog adapt <file>                  # 小说/长文改编为短剧剧本
bbdog parse <file>                  # 解析剧本为结构化数据（角色/场景/美术指导/视觉提示词）
bbdog shots                         # 基于剧本生成分镜列表

# 资产生成（gen 命令组）
bbdog gen characters [--id X]       # 生成角色参考图
bbdog gen scenes [--id X]           # 生成场景参考图
bbdog gen keyframes [--shot X]      # 生成关键帧（起始帧/结束帧）
bbdog gen videos [--shot X]         # 生成视频片段
bbdog gen tts [--shot X]            # TTS 语音合成
bbdog gen subtitles                 # 生成字幕文件（SRT/VTT）

# 设计（独立重跑）
bbdog design art                    # （重新）生成美术指导
bbdog design characters [--id X]    # （重新）生成角色视觉设计
bbdog design scenes [--id X]        # （重新）生成场景视觉设计

# 全局资产库
bbdog asset push --type <t> --id X  # 推送角色/场景到全局库
bbdog asset pull --type <t> --gid X # 从全局库导入
bbdog asset list [--type <t>]       # 列出全局资产

# 后期 & 导出
bbdog render                        # AI 智能剪辑 + ffmpeg 渲染成片
bbdog pipeline <file>               # 一键执行完整流程（DAG 编排）
bbdog episode split                 # 按 cliffhanger 自动分集
bbdog export project                # 导出项目包
```

所有命令支持 `--json` 全局选项，输出 `{ ok, data, nextStep }` 结构供 Agent 消费。

## 制作流程

```
init → create/adapt/parse → shots → gen characters → gen scenes → gen keyframes → gen videos → gen tts → gen subtitles → render → export
```

每步 `nextStep` 引导 Agent 自动执行下一步。`bbdog status --json` 可随时获取当前进度和建议。

## 技术栈

- Monorepo：`packages/bbdog` CLI
- CLI 框架：TypeScript + Commander + Chalk
- 媒体处理：fluent-ffmpeg, sharp, archiver
- AI 提供商：antsk（OpenAI 兼容）、Google、Kling、本地 Claude CLI
- 包管理：npm workspaces
- 构建：`tsc`
- 存储：JSON 文件（项目状态 `.bbdog/project.json`）

## 模型配置

运行时配置存储在 `~/.bbdog/config.json`。

| 用途 | 默认模型 | 说明 |
|------|----------|------|
| 文本（chat） | `claude-cli`（本地） | 通过 Claude Code CLI 调用，免费无需 API key |
| 图片（image） | `gemini-3-pro-image-preview` | 通过 antsk API |
| 视频（video） | `sora-2` | 通过 antsk API |

API key 优先级：模型级 → 提供商级 → 全局 → `process.env.API_KEY`

## 关键文件路径

```
packages/bbdog/
├── bin/bbdog.ts                         # CLI 入口
├── src/cli.ts                           # Commander 命令注册（Agent-First 扁平化）
├── src/commands/
│   ├── flat-aliases.ts                  # 扁平化顶层命令（parse/shots/create/adapt/render）
│   ├── gen.ts                           # gen 命令组（characters/scenes/keyframes/videos/tts/subtitles）
│   ├── init.ts                          # 项目初始化
│   ├── config.ts                        # 配置管理
│   ├── status.ts                        # 项目状态
│   ├── pipeline.ts                      # DAG 一键流程
│   ├── episode.ts                       # 分集
│   ├── export.ts                        # 导出
│   ├── design.ts                        # design 命令组（art/characters/scenes）
│   ├── asset.ts                         # 全局资产库（push/pull/list）
│   ├── direct.ts                        # 关键帧/视频底层实现
│   ├── audio.ts                         # TTS 底层实现
│   └── subtitle.ts                      # 字幕底层实现
├── src/core/
│   ├── api-client.ts                    # HTTP 客户端、重试、Claude CLI 后端
│   ├── workspace.ts                     # 工作区上下文（统一加载项目状态）
│   ├── output.ts                        # JSON/文本双模式输出 + nextStep 协议
│   ├── errors.ts                        # 错误码定义
│   ├── dag.ts                           # DAG 编排引擎
│   ├── prompt-loader.ts                # Prompt 模板加载器（lib/prompts/*.txt）
│   ├── pipeline-steps.ts               # Pipeline 步骤定义
│   ├── model-registry.ts               # 模型/提供商管理
│   ├── builtin-models.ts               # 内置模型列表
│   ├── concurrency.ts                  # 并发控制 + rateLimitDelay
│   ├── render-log.ts                   # 渲染日志
│   ├── media-utils.ts                  # 媒体工具
│   ├── screenplay/                     # 剧本业务域
│   │   ├── script-service.ts           # 剧本解析（薄包装）、分镜生成
│   │   ├── parse-phases.ts             # 4 个独立解析阶段函数
│   │   └── creative-service.ts         # 一句话原创 + 小说改编
│   ├── assets/                         # 视觉资产业务域
│   │   ├── visual-service.ts           # 分层 Prompt 生成 + 图片生成
│   │   ├── prompt-constants.ts         # 系统 prompt + 题材模板
│   │   └── art-presets.ts              # 7 题材美术预设（都市情感/重生逆袭/霸总甜宠/悬疑/古装权谋/穿越/仙侠）
│   ├── video/                          # 视频业务域
│   │   ├── video-service.ts            # Veo/Sora 视频生成
│   │   ├── video-prompt.ts             # 视频 prompt 构建
│   │   ├── prompt-manager.ts           # 角色/场景/美术 prompt 管理
│   │   ├── keyframe-utils.ts           # 关键帧工具
│   │   └── camera-guides.ts            # 镜头运动构图指导
│   ├── editing/                        # 剪辑业务域
│   │   └── edit-service.ts             # AI 剪辑分析 + ffmpeg 渲染
│   └── providers/                      # 多提供商适配
│       ├── base.ts                     # 提供商基类
│       ├── antsk.ts                    # antsk（OpenAI 兼容 API）
│       ├── openai.ts                   # OpenAI 直连
│       ├── google.ts                   # Google（Gemini/Veo）
│       ├── kling.ts                    # 快影
│       └── local.ts                    # 本地 Claude CLI
├── src/storage/
│   ├── config-store.ts                 # ~/.bbdog/config.json 读写
│   └── project-store.ts               # 项目状态持久化
├── src/mcp/
│   └── server.ts                       # MCP Server（实验性）
└── src/types/index.ts                  # TypeScript 类型定义
```

## 开发约定

- 注释和文档使用中文
- Git commit message 使用中文
- 编译检查：`cd packages/bbdog && npx tsc --noEmit`
- 测试：`cd packages/bbdog && npm test`（vitest --run）
- CLI 本地调试：`cd packages/bbdog && npm run build && node dist/bin/bbdog.js`
- **如无必要，不要调用外部 AI**：评价、评审等任务优先派出 Claude Code 子智能体（Agent）完成，避免浪费 API token

## 对标 waoowaoo 路线图

参考调研文档：`docs/waoowaoo-research.pdf` / `docs/waoowaoo-research.md`

### 已完成

- [x] 分层 Prompt 架构（4 层注入：系统 + 题材模板 + 美术指导 + 角色/场景特征）
- [x] 美术指导预设库（7 题材：都市情感/重生逆袭/霸总甜宠/悬疑/古装权谋/穿越/仙侠）
- [x] 角色一致性增强（coreFeatures + primaryIdentifier）
- [x] 三模式支持（原创 create / 改编 adapt / 洗稿 parse）
- [x] Agent-First 扁平化 CLI（顶层命令 + gen 命令组）
- [x] nextStep JSON 输出协议（`{ ok, data, nextStep }`）
- [x] --id/--shot 单项操作（精确定位单个角色/场景/镜头）
- [x] DAG 并行执行（pipeline 命令）
- [x] 多提供商适配（antsk / OpenAI / Google / Kling / 本地 Claude CLI）
- [x] 工作区上下文（workspace.ts 统一加载）

### 待实现（按优先级排序）

#### ~~P0: 拆分 parse 为独立阶段~~ ✅ 已完成
- [x] 从 parseScriptToData() 提取 4 个独立函数（parse-phases.ts）
- [x] 新增 `bbdog parse --phase structural|art|characters|scenes` 选项
- [x] 新增 `bbdog design art/characters/scenes` 命令组
- [x] 新增 `GRANULAR_PIPELINE_STEPS` 细粒度 DAG 步骤

#### ~~P0: 多候选选择机制~~ ✅ 已完成
- [x] `bbdog gen characters --candidates 3` 生成多张候选
- [x] `bbdog gen characters --id X --select N` 选择第 N 张
- [x] Character/Scene 类型增加 `candidateImages[]` 字段
- [x] `generateImageCandidates()` 并行生成函数
- [x] `bbdog gen scenes --candidates / --select` 同步支持
- [x] status 输出候选状态 + nextStep 引导选择
- 评价机制：Agent（Claude Code 子智能体）直接读取图片评价，不调用外部 AI

#### ~~P0: 撤回机制~~ ✅ 已完成
- [x] 重新生成前保存 `previousReferenceImage`
- [x] `bbdog gen characters --id X --undo` 撤回到上一版
- [x] `bbdog gen scenes --id X --undo` 场景同步支持
- [x] `bbdog status --json` 显示 `hasUndo` 标记
- 撤回支持双向切换（undo 后可再次 undo 切回）

#### ~~P1: 摄影 + 演技指导~~ ✅ 已完成
- [x] Shot 类型增加 `photographyRules` 和 `actingNotes` 字段
- [x] 新增 `buildCinematographerGuidance()` — 按题材输出构图/景深/光影/色温规则
- [x] 新增 `buildActingDirectionGuidance()` — 按题材输出表情/肢体/情感节奏指导
- [x] 分镜生成 prompt 自动注入摄影 + 演技指导块
- 支持 5 个题材特殊规则（悬疑/甜宠/重生/逆袭/穿越）

#### ~~P1: Prompt 模板外置~~ ✅ 已完成
- [x] 创建 `lib/prompts/` 目录 + 9 个 `.txt` 模板文件
- [x] 新增 `prompt-loader.ts`（`loadPrompt` + `loadPromptWithFallback`）
- [x] 调用方改用 `loadPromptWithFallback()`（parse-phases / creative-service / script-service）
- 模板使用 `{{varName}}` 占位符，支持 fallback 回退

#### ~~P2: 全局资产库（跨项目复用）~~ ✅ 已完成
- [x] `bbdog asset push --type character/scene --id X` 推送到全局库
- [x] `bbdog asset pull --type character/scene --gid X` 从全局库导入
- [x] `bbdog asset list` 列出全局资产
- [x] `sourceGlobalCharacterId` / `sourceGlobalSceneId` 溯源字段
- 存储: `~/.bbdog/global-assets/`（characters.json + scenes.json + images/）

#### ~~P2: 面板变体~~ ✅ 已完成
- [x] `bbdog gen keyframes --shot X --variants 3` 生成多构图方案
- [x] `bbdog gen keyframes --shot X --select-variant N` 选择方案
- [x] Shot 类型增加 `variantKeyframes` 字段（复用 CandidateImage）

#### P3: 口型同步（需专门模型）
- [ ] lip-sync 集成
- **差距**：waoowaoo `lipSyncTaskId` / `lipSyncVideoUrl`

### 明确不做
- Web UI（bbdog 的 UI 是 Claude Code）
- 内置视频编辑器（输出 FCPXML/EDL 给专业软件）
- MySQL/Redis/MinIO（保持 JSON 文件存储）
- BullMQ 任务队列（CLI 直接执行，DAG 编排已满足需求）

---
> Source: [SttFang/bbdog](https://github.com/SttFang/bbdog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
