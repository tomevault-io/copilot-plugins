## voice-agent-platform

> LiveKit-based 語音 Agent 平台。Profile-driven Agent 系統，admin UI 管理 profile / 工具 / 部署 / sessions。

# voice-agent-workshop

LiveKit-based 語音 Agent 平台。Profile-driven Agent 系統，admin UI 管理 profile / 工具 / 部署 / sessions。

主要架構：
- `agents/agent.py` — LiveKit Agent entrypoint（pipeline + realtime 雙模式）
- `agents/agent_factory.py` — 動態 Agent class 建立（讀 profile YAML/DB）
- `agents/agent_tools.py` — Tool factory registry
- `agents/api/` — FastAPI 管理 API（profiles / sessions / deploy / tools / test）
- `frontend/` — Next.js admin UI + voice 測試頁
- `agents/db/` — SQLAlchemy（SQLite，profile / session / cost 持久化）
- `agents/profiles/` — YAML profile 定義（fallback 來源）
- `agents/Dockerfile` + `agents/livekit.toml` — `lk agent deploy` 從 `agents/` 執行

---

## Profile Editor（v2 → default，2026-05-14 啟動）

> V1 Admin UI 重新設計已完成，歸檔於 `docs/archive/ADMIN_UI_REDESIGN_V1_2026-05.md`

### 現況

Profile Editor v2 已取代舊 prototype，default route 是 `frontend/app/admin/(authenticated)/profiles/[id]/page.tsx`；舊 `/v2` 路由已退場/redirect。OpenSpec 的最新真相在 `openspec/specs/`，近期 `openspec/changes/` 全數歸檔。

核心能力：
- **Prompt / Graph 雙模式**：prompt-first split panel；graph mode 使用 editable React Flow canvas + Node Inspector。
- **Global-as-base-layer IA**：global prompt、identity、QA、hours、tools、model/voice stack 是 always-on base layer；graph 只是 branching layer。
- **Model & Voice full-width view**：Header Stack chip 開啟全頁視圖，編輯 `models.mode`、LLM/STT/TTS provider+model、voice、language、`via`。
- **Catalog source of truth**：`GET /api/model-defaults` 由 backend constants 提供 LiveKit Inference catalog、voice suggestions、language matrix、pricing flags；frontend fallback 由 shared fixture 雙邊測試防 drift。
- **AI Generate**：prompt create/enhance 已完成，使用 LM Studio endpoint；graph 草稿生成仍是延伸項。
- **Flow Test + run log**：Profile Editor 右側 panel 可對已存 profile 跑單輪 deterministic flow smoke/debug；後端保存 `profile_test_runs` / `profile_test_run_events`，事件含 graph path、tool mode、handoff/fallback、sanitized payload。這不打 LLM，也不是 voice Try 替代品。

### 已落地 changes

| Change | 狀態 | 重點 |
|---|---|---|
| `multi-model-runtime` | ✅ archived | `models` block、provider runtime、cost |
| `graph-agent-builder` | ✅ archived | editable graph editor + schema |
| `graph-runtime-executor` | ✅ archived | pipeline graph runtime + fallback |
| `profile-editor-model-voice-ux` | ✅ archived | Stack UI first pass |
| `profile-model-catalog-runtime` | ✅ archived | catalog endpoint + direct Google LLM + voice |
| `profile-stack-language-selection` | ✅ archived | STT/TTS language matrix + controls |
| `profile-editor-visual-refinement` | ✅ archived | full-width Model & Voice view + browser QA |
| `profile-test-runs` | ✅ archived | flow test panel + structured run log |
| `llm-text-test` | ✅ archived | LLM Text Test（`kind: llm_text`）多輪真實 LLM 文字測試 |

### 目前殘項

- Live 語音 e2e：`tool_result → handoff/transfer_to_human` 已修並在 Try log 驗證；外部 LINE 單據已確認收到。剩餘風險是建單 endpoint side effect 成功但 response 超過 agent HTTP tool 的 10s timeout，需調整 quick-ack / timeout / async job contract。
- Flow Test：已支援單輪 deterministic flow smoke/debug 與 structured run log。dry-run 預設不觸發外部副作用。LLM Text Test（`kind: llm_text`）已落地：同 panel 打真實 LLM 的多輪文字測試（無 room / 語音 / 副作用），graph 路由由 LLM 呼叫合成 `goto_*` 工具決定，realtime profile fallback 至 pipeline LLM + warning。
- CI workflow 已建立：root `.github/workflows/test.yml` 跑 backend pytest 與 frontend vitest。
- Per-node model UI、AI Generate graph 草稿、end-node hang-up 仍是 follow-up，不是現有 blocker。

### AI Generate Prompt 設計

- 後端 endpoint：`POST /api/admin/generate-prompt`
- 目前走 LM Studio OpenAI-compatible endpoint（`http://192.168.2.100:1234/v1`，`google/gemma-4-26b-a4b`）
- 輸入：使用者描述（business type + 功能需求）
- 輸出：完整 system prompt 初稿
- 前端：System Prompt 區域右上角 `Generate` 按鈕 → modal 輸入描述 → preview → 使用者確認 Replace

---

## 工程慣例

- 安全 / 中間件 / auth：Apr 30 review 後已 hardened，動到 `frontend/middleware.ts`、`frontend/app/api/admin/*`、`api/routes_test.py` 前先看 git log 理解
- Admin 路由結構：`frontend/app/admin/(authenticated)/` route group 包住所有登入後頁面（共用含 sidebar 的 layout），`frontend/app/admin/login/` 維持平行、不套 sidebar。新增登入後頁面請放進 `(authenticated)/` 內
- 登入跳轉用 `window.location.assign()` 而非 `router.replace()`：login 頁與 dashboard 在不同 layout 下，但 App Router 仍會 prefetch dashboard 的 RSC payload；prefetch 發生時還沒 cookie，middleware 回 redirect 並被 client cache，導致登入後 `router.replace` 拿到的是舊的 redirect。hard navigation 直接繞過 client cache
- Test：`cd agents && uv run pytest tests/ -q`；`cd frontend && pnpm test`（目前 84 passed）。`tests/test_api.py` 的 `client` fixture 會 unset `ADMIN_API_TOKEN`，auth 強制驗證用 `secured_client`
- DB：SQLite，profile 透過 `db.profile_store` 持久化；YAML 是 fallback。`db.engine` 是 module-global singleton，conftest.py 已將測試 DB 指向 `:memory:`
- Profile 改名 / tool 拿掉時記得同步更新 `agents/tests/test_agent_system.py` 的 assertion
- Cost：realtime 用 Gemini Live token rates（`db/cost.py`），pipeline 走 LLM/STT/TTS 分開計費

### Realtime mode 架構（TextInputRealtimeModel）

Realtime mode **不是**純 Gemini audio-in → audio-out。Gemini Live API 接收原始音頻時，audio token 隨對話時間累積，延遲從 1–2 秒遞增到 20–30 秒（已在所有 observability 資料確認）。

解法：`TextInputRealtimeModel`（`agent.py`）封裝 Gemini Live，攔截 `push_audio`，改由外部 Deepgram STT 轉寫後以文字輸入 Gemini。

實際信號流：
```
語音 → Silero VAD → Deepgram STT → text
     → on_user_turn_completed → session.generate_reply(user_input=text)
     → Gemini Live（文字輸入）→ 語音輸出
```

因此 `AGENT_STT_PROVIDER` 在 realtime 和 pipeline 兩種 mode 下**都有效**：
- `inference`（預設）— 走 LiveKit Inference gateway（Cloud 部署用）
- `deepgram` — 直連 Deepgram API key（Try button 本機測試用，避免吃 free plan STT concurrency quota）

### LiveKit Agents multi-process 陷阱

LiveKit Agents 的 worker 用 multi-process 模型：`entrypoint()` 是在 SDK fork 的 **child process** 執行，不繼承 parent 的 `sys.argv`。

影響：`uv run agent.py connect --profile restaurant` 的 `--profile` flag 只對主 worker process 有效；SDK fork 出來跑 `entrypoint()` 的 child process 找不到 `--profile`，會 fallback 到 `AGENT_PROFILE` env var。

修法（已在 `api/routes_test.py` 實作）：`child_env` 必須同時帶 `AGENT_PROFILE=profile`（env var child process 會繼承）。**不可移除這一行**，否則 Try button 全部 fallback 到預設 profile。

```python
child_env = {**os.environ, "AGENT_STT_PROVIDER": "deepgram", "AGENT_PROFILE": profile}
```

### Graph 執行架構（graph-runtime-executor）

`editor_mode: graph` 的 profile 可把對話 graph 直接在 runtime 跑起來，而非只用攤平的 `instructions`。入口是 `agent_factory.build_root_agent(profile, mode)`（`agent.py entrypoint` 在 `session.start` 前呼叫），內含**策略來源 gate**：

```
editor_mode == 'graph' AND 後端 validate_graph().valid AND effective_mode == 'pipeline'
  → build_graph_root_agent：每個 node 一個 Agent instance，回傳 start node
否則
  → create_agent_class(...)()：現有單一 instructions 路徑（fallback，不 raise）
```

關鍵約束：
- **graph 與 realtime 互斥**：graph 執行**僅 pipeline**。realtime 下 node 切換要 mid-session 換 instructions，會被 Gemini Live 1007 拒絕（native-audio）或靜默忽略（flash-live）。save 時 `editor_mode: graph` + `models.mode: realtime` 由 `routes_profiles` 回 422；runtime 誤配（env `AGENT_MODE=realtime` 蓋過）降級跑 flatten `instructions` + warning，**絕不掛電話**。
- **edge 轉移 = SDK 原生 handoff**：function tool 回傳 target node 的 Agent instance 即觸發 handoff（`generation.py make_tool_output`）。`user_turn` 邊 → 生成 `goto_<target>` 工具，description 帶 NL condition，LLM 判斷後呼叫。`tool_result` 邊 → 包裝 node domain tool 回傳 `(result, target_agent)`（`functools.wraps` 保留參數 schema），LLM 講完結果即 handoff，v1 一律無條件。
- **後端 validator 是唯一真相**：`runtime/graph.py validate_graph` 規則集對齊前端 `frontend/lib/agent-graph.ts validateGraph`（前端僅 UX，可被直接打 API 繞過）。改任一邊的結構規則時兩邊一起改。
- **全域能力注入 global 層**：`global_prompt` + QA inline + services hours 組成 preamble，前置到每個 node；`transfer_to_human` 只掛在 `handoff` node（非全域）。
- **per-node 模型**：`Agent(llm=/tts=/stt=)` 可接，v1 只留介面點（node 宣告 model spec 時 log + 用 session 模型），未實作 per-node 切換。

## 部署

- LiveKit Cloud：`lk agent deploy`，`AGENT_PROFILE` env 控制預設 profile
- Docker：`docker compose up -d --build`（frontend port 3004 對外；api 用 `expose: 8083`，僅 compose 內網可達，由 frontend 透過 `http://api:8083` 連）
- Realtime 模式（預設）：Gemini Live + Deepgram STT（避開 audio token 累積延遲 bug，見上方架構說明）

### SIP 接入的 profile 限制

**LiveKit Cloud dashboard 設定的 SIP dispatch rule 沒有 metadata 欄位**，無法動態切換 profile。SIP 來電的 profile 永遠固定在 `AGENT_PROFILE` secret。

若需要不同電話號碼對應不同 profile，只能：
1. 透過 LiveKit API（非 dashboard）建立 dispatch rule，可在 `room_config.agents[].metadata` 帶 `{"profile": "dental_clinic"}`，agent 的 `_get_runtime_profile_name()` 會從 `ctx.job.metadata` 讀取
2. 或部署多個 Cloud agent（不同 `agent_name`），各自對應不同 `AGENT_PROFILE` secret

---
> Source: [Robert-Kung/voice-agent-platform](https://github.com/Robert-Kung/voice-agent-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
