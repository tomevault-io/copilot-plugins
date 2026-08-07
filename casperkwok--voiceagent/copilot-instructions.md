## voiceagent

> 实时 AI 语音外呼 Agent。级联管线 FreeSWITCH→VAD→ASR→EOU→LLM→TTS，靠投机生成 + 帧级流式把命中路径压到 ~500ms。

# CLAUDE.md — VoiceAgent

实时 AI 语音外呼 Agent。级联管线 FreeSWITCH→VAD→ASR→EOU→LLM→TTS，靠投机生成 + 帧级流式把命中路径压到 ~500ms。

## 开工先读（治理文档）

| 想知道 | 看 |
|---|---|
| 快速召回（决策/偏好/踩坑/事实/目标） | `project/project.mneme`（agent 召回层，先读 spine：grep `^@`/`^gist`/`^state`） |
| 方向 / 目标 / 达标线 | `project/OKR.md`（唯一权威纲领） |
| 为什么这么定、否决了什么（完整） | `project/DECISIONS.md`（决策日志 ADR） |
| 怎么做的、踩了啥坑 | `project/NOTES.md`（迭代备忘） |
| 文档体系关系 / Mneme 格式 | `project/README.md` / `project/mneme-manual.md` |
| 设计细节（延迟/投机/打断/并发） | `docs/`（VitePress） |

### 记忆：Mneme + DECISIONS 并存（两边都写）
- `project/project.mneme` = agent 召回层（cell：gist/state/cue + 指回 D 编号），格式见 `mneme-manual.md`。
- `project/DECISIONS.md` = 完整 prose / 审计层（保持现状）。
- **产生持久决策/偏好/踩坑/事实时：DECISIONS.md 写完整条目 + project.mneme 追加对应 cell**（自包含 gist、写 cue、since=seen=今天、按需 supersede/link）。**两边都要有。**
- 用前看 cell 的 state/seen：`live+high+新` 直接用；`stale/low/久` 先核实再用、核实后把 seen 更新为今天。内容永不就地改，改认知靠新增 + supersede。
- **分配新 cell 的 ID 用 `python3 project/mneme_tool.py new-id PREFIX`**（别手数，防撞号）。提交时 pre-commit 钩子自动 lint（结构错误挡提交、seen 衰减告警）；也可手动 `mneme_tool.py lint`。

## 项目定位（已对齐 2026-06-02）

- **小规模内部用**（几路~十几路并发），起步学习、**后期推生产**。
- **坚持级联管线**，不转 speech-to-speech。
- **当前先压延迟**；生产健壮性作并行副线。
- 别堆大厂方案（不上 k8s/集群/Prometheus 等）。重大部署动作（上云、接真实线路）**先和用户确认**。

## 衡量口径（重要）

延迟一律用**用户视角的感知口径**：「用户真正停止发声 → 听到 AI 第一个字」，按 **S1–S5 分段**。
**不要**用工程内部指标 `speech_end→首帧 PCM`（它漏掉 VAD 尾 + 播放/链路税，会自欺）。详见 OKR「衡量基准」。

## 架构纪律（不加深单通话焊死）

整个进程目前是单通话单管线实例。做任何改动守 4 条（详见 `docs/reference/concurrency-readiness.md`）：
1. 可共享状态放**进程级**，通话状态放 **Controller / Stream 实例**——别像 `TTSWorker.last_heard_text` 那样把通话状态塞进共享 worker。
2. 固定话术缓存放进程级、播放走 per-call FIFO。
3. 新路径显式带 `call_uuid`，别依赖"反正只有一通"。
4. 别在 audio_loop / event_loop 里堆全局可变状态。

## 工作流约束

- **KA0（指标落库）是 gate**：基线没出前不写其他优化；任何 KA 先有基线、做完看数验证收益。
- 改业务代码前先说清方案，得到确认再动手。

## 常用命令

```bash
# 浏览器调试模式（绕开电话网）：后端 + web
TRANSPORT=browser uv run python -u main.py        # 后端 (DebugWS :8890)
cd web && npm run dev                              # 调试台 :3000
#   内网可访问：web 用 `npx next dev -H 0.0.0.0`，前端按 location.hostname 自动拼 ws

# 电话模式（需 FreeSWITCH 在跑，拨 9999）
./start_voice_agent.sh

# 文档站
cd docs && npm run dev
```

环境变量：`TRANSPORT=freeswitch|browser`、`EOU_BACKEND=model|rule`、`BARGE_SOFT_DUR`（软打断时长阈值）。ASR 固定走火山（本地 FunASR 已弃用，见 DEC-0023）。

## 个人偏好（与全局 ~/.claude/CLAUDE.md 一致）

- Git 提交只署名用户本人，**绝不加 `Co-Authored-By: Claude`**；PR body 不加 Claude 署名。

---
> Source: [casperkwok/VoiceAgent](https://github.com/casperkwok/VoiceAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
