## openjellyfish

> This is a FastAPI + DeepAgents project (renamed to JellyfishBot) with the following structure after refactoring:

# JellyfishBot Project Rules

## Architecture
This is a FastAPI + DeepAgents project (renamed to JellyfishBot) with the following structure after refactoring:
Two-tier system: Admin Management + Consumer Consumption.

```
app/
  core/           - settings.py, security.py (auth), observability.py (langfuse)
                  - path_security.py (ensure_within / safe_join — unified path traversal prevention)
  storage/        - 可选 S3 文件存储后端 (STORAGE_BACKEND=local|s3 环境变量切换)
                  - config.py (S3Config, is_s3_mode)
                  - base.py (StorageService ABC)
                  - local.py (LocalStorageService — 封装 os.* 本地磁盘操作)
                  - s3.py (S3StorageService — boto3 S3 API)
                  - s3_backend.py (S3Backend — deepagents BackendProtocol for S3)
                  - __init__.py (工厂: get_storage_service, create_agent_backend, create_consumer_backend)
  schemas/        - requests.py (admin models), service.py (service/consumer models)
  routes/         - auth.py, conversations.py, chat.py, files.py, scripts.py,
                    models.py, settings_routes.py, batch.py
                  - services.py (admin service CRUD + key mgmt)
                  - consumer.py (consumer chat APIs: OpenAI compat + custom SSE)
                  - consumer_ui.py (standalone chat page at /s/{service_id})
  services/       - agent.py, tools.py, conversations.py, prompt.py,
                    subagents.py, ai_tools.py, script_runner.py
                  - _sandbox_wrapper.py (runtime file I/O sandbox for script execution)
                  - published.py (service CRUD, API key mgmt, consumer conversations)
                  - consumer_agent.py (consumer agent factory, with memory subagent)
                  - memory_tools.py (Memory subagent tools + soul config + short-term memory injection)
  voice/          - router.py (WebSocket S2S proxy)
  deps.py         - get_current_user (admin), get_service_context (consumer via sk-svc-)
  main.py         - FastAPI app assembly (64 routes) + startup/shutdown
frontend/                — Vite + React 18 + TypeScript + Ant Design 5
  index.html              — Vite entry point
  vite.config.ts          — dev server (port 3000), proxy /api → FastAPI :8000
  tsconfig.json           — strict TS, path alias @/* → src/*
  src/
    main.tsx              — React root mount
    App.tsx               — ConfigProvider (antd dark theme) + AuthProvider + Router
    router/index.tsx      — BrowserRouter: /login, /, /services, /scheduler, /wechat
    layouts/AppLayout.tsx — Sider (nav menu) + Content <Outlet/>
    pages/
      Login.tsx           — 品牌分栏登录/注册：左 40% 渐变品牌区（`/media_resources/jellyfishlogo.png` + 呼吸动画 + 像素点底纹），右 60% 表单区 `#1c1c27`；主色 Primary `#E89FD9`、Secondary `#8B7FD9`、Accent `#5FC9E6`；表单项聚焦描边与渐变按钮；样式以 inline + 组件内 `<style>`（keyframes）为主，窄屏约 900px 以下改为上下堆叠
      Chat/index.tsx      — conversation list + message area + SSE streaming
      AdminServices/index.tsx — Service 管理页：左 30% 服务列表（卡片 `#1c1c27`、圆角 8px、悬停边框 `#E89FD9`、选中左侧 3px `#E89FD9`、发布/草稿用绿/灰圆点）；右 70% 详情区 Tabs：Basic Info | API Keys | WeChat Channel | Test；API Keys 表斑马纹深色行；复制按钮用 Phosphor `Copy` + 点击高亮 Accent；图标 `@phosphor-icons/react`（Plus/Trash/PencilSimple/LinkSimple/ArrowsClockwise/GridFour）；品牌色与 `FLOAT_SHADOW`、`MODAL_RADIUS` 12px 与 Login/Chat 一致
      Scheduler/          — placeholder (migrating from scheduler.html)
      WeChat/             — placeholder (migrating from admin-wechat overlay)
    services/api.ts       — typed API client (port of legacy js/api.js)
    stores/authContext.tsx — React Context for auth state
    styles/
      global.css          — scrollbar, markdown, antd overrides
      theme.ts            — antd ThemeConfig matching original dark palette
    types/index.ts        — shared TS interfaces
  public/                 — static assets (legacy HTML kept for FastAPI template pages)
    service-chat.html     — consumer chat (served by FastAPI /s/{service_id})
    wechat-scan.html      — WeChat scan (served by FastAPI /wc/{service_id})
  server.js               — legacy Express proxy (kept, use `npm run legacy`)
  public/js/              — legacy vanilla JS (reference during migration)
  public/css/             — legacy CSS (reference during migration)
```

## Storage Backend (可选 S3)
- 通过环境变量 `STORAGE_BACKEND=local|s3` 切换，默认 `local`
- `local` 模式：所有文件操作使用本地磁盘（行为与原代码一致）
- `s3` 模式：文件存取走 S3 API（兼容 AWS S3、MinIO、R2、OSS 等）
- **工厂函数**：`get_storage_service()` 返回 StorageService 实例，`create_agent_backend()` 返回 deepagents BackendProtocol 实例
- **S3 键映射**：用户文件 `{prefix}/{user_id}/fs/{path}`，消费者生成文件 `{prefix}/{admin_id}/svc/{svc_id}/{conv_id}/gen/{path}`
- **媒体访问**：S3 模式下使用 presigned URL（302 重定向），local 模式使用 FileResponse
- **脚本执行**：S3 模式下临时下载脚本到本地执行，结果上传回 S3
- 新增依赖：boto3, aioboto3（仅 S3 模式使用）

## Two-Tier Design
- **Admin**: registers, manages docs/scripts/prompts, publishes Services
- **Service**: a published config selecting model + docs + scripts + capabilities
- **Consumer**: authenticates via per-service API key (`sk-svc-...`)
- **Isolation**: consumer generated content isolated per `conversation_id` under
  `users/{admin_id}/services/{service_id}/conversations/{conv_id}/generated/`
- **Consumer APIs**:
  - `POST /api/v1/chat` — custom SSE (same event format as admin)
  - `POST /api/v1/chat/completions` — OpenAI-compatible streaming/non-streaming
  - `POST /api/v1/conversations` — create conversation
  - `GET /api/v1/conversations/{conv_id}` — get conversation history
  - `GET /api/v1/conversations/{conv_id}/files` — list generated files
- **Admin Service APIs**: `POST/GET/PUT/DELETE /api/services/{service_id}`, key mgmt at `/keys`
- **Standalone Chat**: `GET /s/{service_id}` serves `service-chat.html`

## Startup
```bash
# 推荐：跨平台 Python 启动器（自动端口检测 + 旧实例清理）
python launcher.py              # 生产模式
python launcher.py --dev        # 开发模式（uvicorn --reload + vite dev）
python launcher.py --port 9000  # 自定义后端端口

# 快捷脚本
./start_local.sh      # Mac/Linux
start_local.bat       # Windows

# Docker
./start.sh            # Docker 容器内启动

# 手动启动后端（调试用）
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

## Key Technical Details
- **Checkpointer**: `AsyncSqliteSaver` — DB at `data/checkpoints.db`; package `langgraph-checkpoint-sqlite` is installed in venv
  - **WAL 模式**：`init_checkpointer` 在 `aiosqlite.connect` 后执行 `PRAGMA journal_mode=WAL` + `PRAGMA synchronous=NORMAL`，降低多路由/多 bridge 并发读写 checkpoint 时的锁冲突。PRAGMA 失败（极少数网络盘场景）不阻塞启动。
- **deepagents**: installed in venv at d:\semi-deep-agent\venv
- **No `deepagents` in system Python** — always use `.\venv\Scripts\python.exe`
- **Auth (admin)**: JSON-based, stored in `users/users.json`; bcrypt with sha256 fallback
- **Auth (consumer)**: per-service API keys (`sk-svc-...`), sha256 hashed, stored in `users/{admin}/services/{svc}/keys.json`
- **Langfuse**: optional, env-based; SDK v3 (reads from env vars directly)
- **S2S Voice**: WebSocket proxy to OpenAI Realtime API; tools injected at `session.update`
- **Research tools (rag_query, market_data)**: removed in commit a301b31

## API Key / Base URL Configuration
- **Centralised in `app/core/api_config.py`** — `get_api_config(capability)` returns `(key, base_url)`
- **Provider defaults**: `OPENAI_API_KEY` / `OPENAI_BASE_URL`, `ANTHROPIC_API_KEY` / `ANTHROPIC_BASE_URL`
- **Per-capability overrides** (fall back to provider defaults if unset):
  - `IMAGE_API_KEY` / `IMAGE_BASE_URL` — image generation
  - `TTS_API_KEY` / `TTS_BASE_URL` — text-to-speech
  - `VIDEO_API_KEY` / `VIDEO_BASE_URL` — video generation
  - `S2S_API_KEY` / `S2S_BASE_URL` — realtime voice
  - `STT_API_KEY` / `STT_BASE_URL` — speech-to-text (Whisper)
- **LLM base_url**: `_resolve_model` in `agent.py` passes `base_url` to `init_chat_model` when `OPENAI_BASE_URL` or `ANTHROPIC_BASE_URL` is set
- **Never hardcode API URLs** — always go through `api_config` or env vars

## Supported Models
- Claude Opus/Sonnet/Haiku 4.x with optional Thinking variants
- GPT-5.x, GPT-4o, o3-mini
- Model IDs use `provider:model-id` format (e.g., `anthropic:claude-sonnet-4-5-20250929`)
- Thinking models configured in `app/services/agent.py::THINKING_MODEL_CONFIG`

## Dev Guidelines
- All imports should use `app.*` package paths
- No top-level imports of `deepagents` outside `app/services/agent.py`, `app/services/consumer_agent.py`, and `app/voice/router.py`
- Route files use `APIRouter` — assembled in `app/main.py`
- Circular imports avoided by doing lazy imports inside functions (e.g., `clear_agent_cache` in prompt.py/subagents.py)
- Consumer agent uses `get_service_context` dependency (not `get_current_user`)

## Security Architecture
- **Path traversal prevention**: all user-facing path operations use `app.core.path_security.safe_join()` / `ensure_within()` which resolve via `pathlib.Path.resolve()` + separator-aware boundary check
- **XSS prevention**: `consumer_ui.py` uses `html.escape()` for template injection; frontend uses DOMPurify to sanitize `marked.parse()` output
- **Script sandbox (two layers, defense-in-depth)**:
  1. **AST static analysis** (`script_runner._check_script_safety`): blocks dangerous modules (subprocess, pathlib, ctypes, io, pickle, threading, posix/nt/_posixsubprocess, etc.), dangerous builtins (exec, eval, getattr, setattr, globals, etc.), only *absolutely dangerous* os functions (system/popen/exec*/spawn*/fork/kill/chown/setuid/chroot/chdir), and access to `__builtins__`/`__subclasses__`/`__globals__`/`__dict__`/`__mro__`/`__bases__`. **File I/O functions (remove/rename/mkdir/listdir/chmod/...) are intentionally NOT in the AST blacklist** — they pass AST and are enforced at runtime via path whitelist.
  2. **Runtime wrapper** (`_sandbox_wrapper.py`): monkey-patches `builtins.open`, `io.open`, `os.listdir`, `os.scandir`, `os.walk`, `os.chdir` + **`os.open` / `os.readlink`** to enforce directory-level read/write restrictions; **writes** (`remove/unlink/rmdir/removedirs/rename/renames/replace/mkdir/makedirs/chmod/fchmod/lchmod/truncate/mkfifo/mknod/utime/link/symlink`) are wrapped to require their path args be inside `allowed_write` before delegating to the original function. **Absolutely dangerous** functions (`system/popen/exec*/spawn*/posix_spawn*/fork/forkpty/kill/killpg/chown/lchown/setuid/setgid/se[te]uid/se[te]gid/setres*id/chroot/pipe/pipe2/dup/dup2`) are *replaced* with `PermissionError`-raising stubs — `os.__dict__["system"]` and `os.system` both resolve to the blocked stub (since they reference the same module-level binding).
- **沙箱加固 (2026-04-16 第一轮)**：AST 层——`ast.Call` 检查新增对 `func` 为 `ast.Subscript` 或 `ast.Call` 的拒绝，封堵 `os.__dict__['system'](...)` / `(lambda:...)()(...)` 等动态构造调用；新增 `__dict__`/`__mro__`/`__bases__` 属性黑名单。
- **沙箱加固 (2026-04-16 第二轮，运行时纵深)**：真实 PoC 验证发现 `os.__dict__["popen"]` 在未加固前可拿到原函数并执行任意命令（且 root 进程风险）、`os.__dict__["remove"]` 绕过 builtins.open 补丁删除任意文件。加固：**(a)** 运行时直接覆盖 `os.system/popen/exec*/spawn*/fork/kill/chown/setuid/chroot/pipe/dup` 等为 `PermissionError` 报错函数，即使攻击者通过 `os.__dict__[name]` / `vars(os)[name]` / 未来未知的 AST 绕过拿到引用，调用时仍然抛错；**(b)** 写操作（`remove/rename/mkdir/chmod/link/symlink/truncate/utime/...`）统一走 `_check_write` 路径白名单，允许合法脚本在 `allowed_write` 内使用；**(c)** 新增 `os.open` / `os.readlink` 低层 FD 拦截，之前 `fd = os.open("/etc/passwd", os.O_RDONLY)` 完全裸奔；**(d)** 从 AST 黑名单移除 `os.remove/rename/listdir/mkdir/chmod/symlink` 等，否则合法写文件脚本会被误杀。**IfExp 残留绕过**（`fn = os.system if True else None; fn(...)`）由运行时层兜底，AST 层不再追加。综合验证 29/29 用例全通过。
- **Script allowed directories**:
  - Admin: read = `scripts/ + docs/`, write = `scripts/ + generated/`
  - Consumer: read = `scripts/ + docs/` (filtered by allowed_scripts), write = conversation `generated/`
- **超管脚本免限制开关 `SUPERADMIN_SCRIPT_UNRESTRICTED`（部署级，2026-06-08）**：
  - 运维/超管在服务器上设置的环境变量（取值 `1/true/yes/on` 视为开启，默认关闭）。由 `script_runner.superadmin_script_unrestricted()` 读取（call-time 读 env，改值无需重启）。
  - 开启后 **仅 admin 侧** 脚本执行解除四项限制：**(1)** AST 静态分析（允许 `import subprocess/socket`、`os.system/eval/exec` 等）；**(2)** 环境变量白名单（透传完整 `os.environ`，脚本可读取 API 密钥等，仅强制 `PYTHONIOENCODING=utf-8`）；**(3)** 资源限制（不设 `RLIMIT_AS/RLIMIT_NPROC`、不注入 `_BLAS_THREAD_ENV`）；**(4)** 超时上限（不再 clamp 到 `MAX_TIMEOUT=120`，honor 调用方传入值）。
  - **保留**：脚本路径必须在 `scripts_dir` 内、拒绝符号链接、只允许 `.py`、文件存在校验，以及 `_sandbox_wrapper.py` 的运行时读写路径白名单（`--allowed-read/--allowed-write` 仍生效）。⚠️ 注意：AST 解除后脚本可 `subprocess` 派生子进程，而 wrapper 的 monkey-patch 只作用于本进程，子进程可绕过路径白名单——这是超管开关的已知代价，符合"免限制"语义。
  - 实现：`run_script(..., unrestricted: bool=False)` 新增参数；`_build_safe_env(unrestricted)` / `_get_preexec_fn(unrestricted)` 分支；`unrestricted=True` 时打 `_log.warning` 审计。
  - **传入点（admin 侧，开关生效）**：`tools.py::create_run_script_tool`（覆盖主 agent + subagent + S2S + batch，全部共用此工厂）、`scheduler.py::_run_script_task`（定时脚本任务）、`routes/scripts.py::api_run_script`（手动运行）。
  - **永不传入（service 侧，始终完整限制）**：`consumer_agent.py` 内的 `run_script` 硬编码 `unrestricted=False`——即使部署开关开启，消费者/服务端脚本执行也保持 AST/env/资源/超时/路径全部限制。
- **Script 资源限制（2026-04-18 调优）**：
  - `_MAX_NPROC = 256`（原 16，numpy/OpenBLAS 在 16 配额下 `pthread_create failed`，崩在 `import numpy` 阶段）
  - `_MEMORY_LIMIT_BYTES = 1024 MB`（原 512MB，给 pandas/matplotlib 留余量）
  - `_BLAS_THREAD_ENV` 强制注入 `OPENBLAS_NUM_THREADS=2` / `OMP_NUM_THREADS=2` / `MKL_NUM_THREADS=2` / `NUMEXPR_NUM_THREADS=2` / `VECLIB_MAXIMUM_THREADS=2`，避免单脚本启动数十线程占满 CPU
  - **Linux 注意**：`RLIMIT_NPROC` 限制的是 **当前 uid 的进程/线程总数**（pthread = LWP），切到 `jellyfish (uid=1000)` 非 root 后更敏感
- **Script 全局排队（2026-04-18 新增）**：
  - `script_runner._SCRIPT_SEMAPHORE = threading.BoundedSemaphore(SCRIPT_CONCURRENCY)` 进程内信号量
  - 默认 `SCRIPT_CONCURRENCY=4`、`SCRIPT_QUEUE_TIMEOUT=180s`，可通过环境变量覆盖
  - 排队中的脚本只占一个 Python 线程，不消耗子进程 / 内存 / NPROC 配额
  - 超时返回友好错误：`"脚本执行队列繁忙，已等待 180 秒。当前 N 个脚本运行中，前面还有 M 个等待。"`
  - `_active_count` / `_pending_count` 由 `_SCRIPT_STATS_LOCK` 保护，可通过 `get_script_runtime_stats()` 读取（未来若加 `/api/admin/script-stats` 端点可直接复用）
  - **当前规模 (admin ≤ 20, uvicorn workers=1) 进程内 semaphore 足够**；扩到多 worker 需替换为 cross-process 队列 (Redis / file lock)
- **Sandbox read exemptions** (automatically applied, not user-configurable):
  - `_PYTHON_READ_ROOTS`: `sys.prefix`, `sys.base_prefix`, `sys.exec_prefix`, `site.getsitepackages()` — allows `import matplotlib` etc.
  - `_SYSTEM_READ_DIRS`（2026-04-21 大幅放宽）: 整个 `/usr`、`/etc`、`/opt`、`/Library`、`/System`、`/Applications`、`/private/etc`、`/private/var/folders`、`/bin`、`/sbin`、`/lib`、`/lib64`、`/var/cache/fontconfig`、`~/.fonts`、`~/.local/share/fonts`、`~/Library/Fonts` — 覆盖 matplotlib 字体扫描、PIL codec、SSL CA、locale/timezone 等几乎所有库 import 时遍历的系统目录（写权限不变，依然只允许 `_ALLOWED_WRITE`）
  - `_TEMP_DIR` (write): `tempfile.gettempdir()` — allows library cache writes (matplotlib font cache etc.)
  - 环境变量预设（启动时强制写入沙盒可写目录）：`MPLCONFIGDIR=$TMPDIR/mpl_sandbox`（matplotlib 配置/字体缓存）、`MPLBACKEND=Agg`（无头后端，避免 Tk/Qt/Cocoa 探测）、`FONTCONFIG_PATH=/etc/fonts`、`XDG_CACHE_HOME=$TMPDIR/xdg_cache_sandbox`
- **Sandbox 静默扫描 (2026-04-21)**：`os.listdir` / `os.scandir` / `os.walk` 对越权路径**返回空集**而非抛 `PermissionError`。原因：matplotlib `font_manager` 等库会主动遍历一堆系统/家目录，遇到一个就抛会让整个 import 链挂掉；扫描型调用静默返回空让库以为「目录是空的」从而跳过；真正读文件内容仍走 `open()` 硬拦截，敏感文件（如 `~/.zsh_history`、`~/.ssh/`）依旧被保护
- **Sandbox 检查函数防递归 (2026-04-21 关键)**：`_check_read` / `_check_write` / `_is_read_allowed` / `_is_write_allowed` 都用 `_check_guard = threading.local()` 守护 —— 因为它们内部要 `os.path.realpath(path)` 解析符号链接，而 realpath 会调 `os.readlink`（已被 patch），patched readlink 又会回头调 `_check_read` → realpath → readlink…。必须在最外层置 `busy=True`，递归进来直接放行（外层兜底）。错误信息里二次调用 realpath 也必须包在 guard 里（用 `_safe_realpath` helper），否则一旦失败路径触发就死循环成 `RecursionError`。**任何新增的 patched os 函数若内部调用了路径检查或 realpath，都必须遵守这个 guard 协议**

## Cross-platform (Windows / Linux)
- **Path prefix checks** (`path_security.ensure_within`, `script_runner` scripts_dir guard, `_sandbox_wrapper._is_within`): use `str.startswith(root + os.sep)` — on Windows (case-insensitive FS) mixed-case paths can theoretically false-negative if resolved strings differ in case from `root_resolved`.
- **script_runner**: `preexec_fn` is intentionally `None` on Windows — no `resource.setrlimit` (memory/NPROC); Unix/macOS get extra limits. Subprocess uses `encoding="utf-8"` / `PYTHONIOENCODING=utf-8` (good vs locale/GBK).
- **`_sandbox_wrapper`**: monkey-patches are OS-agnostic in CPython; `os.fchdir` gated with `hasattr`. Pipe `|` as dir delimiter is fine (Windows paths use `:` in drive letters, not `|`).
- **`files.py` / `security.py`**: text I/O is explicitly UTF-8; non-UTF-8 files (e.g. legacy GBK) surface as decode errors / “binary” preview — more common on Chinese-locale Windows.
- **Known gap**: `run_script(..., input_data=...)` documents stdin but does not pass `stdin` to `subprocess.run` (all OS).

## WeChat iLink Bot 集成（wechat-bot/）

### 协议概要
腾讯 iLink Bot API（`ilinkai.weixin.qq.com`），基于 `@tencent-weixin/openclaw-weixin@1.0.2` 源码还原。

### 关键发现（踩坑记录）
1. **`qrcode_img_content` 是扫码 URL**，不是 base64 图片，需要用 qrcode 库生成 QR 码
2. **`sendmessage` 必须包含 `base_info` 和 `client_id`**，缺少任何一个都会导致消息静默丢失（HTTP 200 但不送达）
3. **`msg.from_user_id` 必须是空字符串 `""`**，填 bot ID 会导致消息不送达
4. **`client_id` 格式**: `openclaw-weixin:{timestamp_ms}-{random_hex_8}`
5. **`SendMessageResp` 正常就是 `{}`**（空对象），不含 ret 字段
6. **`ilink_user_id` 只用于 `getconfig` 和 `sendtyping`**，不能加到 `sendmessage`/`getupdates`
7. **`getconfig` 需要 `ilink_user_id` + `context_token` + `base_info`**
8. **网络分流**：iLink（国内）必须直连（`proxy=None`），OpenAI 走 SOCKS5 代理

### sendmessage 正确格式（对齐源码 send.ts + api.ts）
```json
{
  "msg": {
    "from_user_id": "",
    "to_user_id": "xxx@im.wechat",
    "client_id": "openclaw-weixin:1774427815868-44e6cc41",
    "message_type": 2,
    "message_state": 2,
    "context_token": "<从 inbound 消息取>",
    "item_list": [{"type": 1, "text_item": {"text": "回复内容"}}]
  },
  "base_info": {"channel_version": "1.0.2"}
}
```

### 文件结构
```
wechat-bot/
├── bot.py              # 主程序入口 + 消息路由 + GPT 多轮对话
├── ilink_client.py     # iLink 协议客户端（对齐 npm 源码）
├── chat.py             # OpenAI GPT-4o 对话管理 + Vision
├── media.py            # AES-128-ECB 加解密
├── requirements.txt    # httpx[socks], openai, pycryptodome, qrcode, python-dotenv
└── .env                # OPENAI_API_KEY, OPENAI_PROXY
```

### 集成到 semi-deep-agent 的方向（完整设计文档：docs/wechat-channel-design.md）
- 两层二维码架构：Service QR（稳定 URL）→ iLink QR（动态刷新）
- 每个微信用户扫码后分配独立 conversation_id，复用 consumer conversation 机制
- consumer_agent.py 需支持 humanchat capability → 注入 send_message 工具
- consumer_agent.py `_build_consumer_system_prompt`：支持 `user_profile_version_id` 注入 Profile 版本内容到 `{user_profile_context}` 占位符
- WeChat Bridge 拦截 send_message tool_result → 通过 iLink sendmessage 转发
- 新增模块：app/channels/wechat/（client, media, session_manager, bridge, router）
- 中间页路由：app/routes/wechat_ui.py → /wc/{service_id}
- 中间页前端：frontend/public/wechat-scan.html
- Service config 扩展 wechat_channel 字段（enabled, expires_at, max_sessions）
- Service config 支持 `system_prompt_version_id` 和 `user_profile_version_id`（均可选），指定后 consumer agent 使用对应版本内容
- main.py 已注册路由 + startup/shutdown 钩子（restore_sessions, start_all_polling, cleanup_task）
- app 路由总数从 64 增至 70
- admin-services.html 新增微信渠道管理面板（启用/禁用/过期设置/QR 链接/活跃会话列表/断开/查看对话记录）
- session_manager: 指数退避重连（max 20次后自动移除）；有 `from_user_id` 的会话**不**参与 24h 无活动清理（长期保留）；无 `from_user_id` 的空会话才按 inactive 清理；空轮询仅对「未建立用户」会话在 50 次后丢弃
- **delivery.py（统一投递层）**：`app/channels/wechat/delivery.py`
  - `deliver_tool_message(content_json, session, client)` — 解析 send_message 工具结果，投递文本+媒体到微信
  - `send_media_to_wechat(session, client, media_path)` — 发送生成的媒体文件（图片/视频/语音/文件）
  - `extract_media_tags(text) -> (cleaned_text, [paths])` — **从 send_message 的 text 中提取 `<<FILE:path>>` 标签**（去重 + 折叠空行），转为额外媒体投递；admin_bridge / consumer bridge 共享
  - Bridge 和 Scheduler 共享此模块，避免重复代码
- **微信媒体投递的 `<<FILE:...>>` 兼容（2026-04-18 修复）**：
  - 背景：`tools.py::generate_image/speech/video` 工具返回提示语 `"请使用 <<FILE:{path}>> 展示给用户"`，system prompt 也教 agent 用此标签（web 端 markdown.ts 渲染契约）。Agent 在微信对话里会把 `<<FILE:...>>` 直接塞进 `send_message(message=...)` 的 text，没用 `media_path` 参数 → 投递层之前只看 `payload.media`，结果用户在微信端看到字面字符串
  - 修复：投递层做"宽容解析"，主动从 text 抽 `<<FILE:...>>` 标签作为额外 media 投递，剩余 cleaned_text 作为文本发出。同时支持 `media_path` 参数和 `<<FILE:...>>` 标签两种用法
  - 涉及：`delivery.py::extract_media_tags` + `deliver_tool_message`（consumer bridge / scheduler 自动获益）；`admin_bridge.py` 内联 send_message 处理（admin/consumer 媒体路径解析逻辑不同，仅共享解析 helper）
- **定时任务微信投递**：scheduler.py `_run_service_agent_task(reply_to=...)` 在 agent 执行循环中拦截 send_message 工具调用，通过 delivery.py 实时发送到微信（支持文本+媒体）；`_deliver_reply` 仅在 agent 未使用 send_message 工具时作为兜底（纯文本摘要）
- bridge: 图片消息双向支持
  - 接收：CDN GET 下载 + AES 解密 → base64 multimodal → Agent Vision
  - 发送：getuploadurl(filekey/media_type/rawsize/rawfilemd5/filesize/aeskey_hex) → POST encrypted to CDN → x-encrypted-param header → sendmessage(image_item.media.encrypt_query_param + aes_key=base64(hex))
- bridge: 语音消息支持（CDN 下载 → AES 解密 → SILK→WAV 转码 → Whisper 转文字 → Agent 处理）
- bridge: 发送 TTS 音频：/generated/audio/ 目录下的 MP3 文件通过 send_file 作为文件附件发送
  - SILK 语音条方案暂停（pysilk 编码的 SILK 在 WeChat 客户端始终静音，24kHz/16kHz 均无效，待进一步调研）
  - voice_item 字段中 encode_type/sample_rate 会导致消息被静默丢弃，只能用 media+playtime
  - TTS 文件名不可预测（hangzhou_intro.mp3 etc），按目录 `/audio/` 检测而非文件名前缀
### iLink 媒体下载协议要点（重要！）
- 接收的图片/语音/视频**没有 cdn_url 字段**，下载方式和发送完全不同
- 字段名：image_item.aeskey（直接hex）、media.aes_key（base64编码）、media.encrypt_query_param
- CDN 下载：GET https://novac2c.cdn.weixin.qq.com/c2c/download?encrypted_query_param={url_encode(value)}
  - 注意参数名是 encrypted_query_param（有d），不是 encrypt_query_param
  - 值必须 URL 编码（urllib.parse.quote）
- AES key 三种格式：直接hex(32字符)、base64(原始16字节)、base64(hex字符串)
- 微信语音使用 SILK 格式（WeChat 变体：首字节 0x02），需用 pysilk 解码为 PCM 再转 WAV
- logging 配置：main.py 需要 logging.basicConfig(level=logging.INFO) 否则 wechat.* 日志不输出
- Go SDK 参考：github.com/openilink/openilink-sdk-go（cdn.go 有完整实现）
- 协议文档：https://www.wechatbot.dev/zh/protocol
- rate_limiter: 单用户消息频率限制（10条/60s）、QR 生成频率（5次/60s）、全局 session 上限（config max_sessions）
- 多 Admin 隔离：list_sessions/remove_session 按 admin_id 过滤，防止跨 Admin 越权
- router: 新增 GET /api/wc/{service_id}/sessions/{session_id}/messages 端点（Admin 查看对话历史）
- frontend/server.js: 新增 /wc 代理规则（此前缺失导致 /wc/ 被 SPA fallback 拦截到登录页）
- frontend/server.js SPA fallback：**不要**用 `app.get('/{*path}', ...)`，Express 5 + path-to-regexp v8 下对深层路径（如 `/settings/services`）匹配不稳定，会触发 "Cannot GET /xxx/yyy"。改用 `app.use((req,res,next)=>{...})` 兜底，并显式排除 `/api`、`/s/`、`/wc/`、`/assets/`（不存在的 assets 应当 404 而不是被错当 HTML 返还，否则浏览器解析旧 chunk 时会报令人困惑的语法错误）
- frontend/server.js 代理 pathFilter 用 `'/wc/'` 而非 `'/wc'`：http-proxy-middleware v3 的 pathFilter 是前缀匹配，加结尾斜杠避免误拦截 `/wcanything`

## Consumer Agent — 渠道感知（channel-aware）
- `create_consumer_agent(..., channel: str = "web")` —— 调用方必须显式传渠道：
  - `channel="web"`（`/api/v1/chat`、`/api/v1/chat/completions`，consumer 直连 SSE）—— **不**注入 `send_message`，即便 humanchat capability 启用。原因：web 上 agent 输出已经直接流给浏览器，再调 send_message 既无投递目标，又会让消费者看到不该看的工具事件
  - `channel="wechat"`（`bridge.py`）—— 注入 send_message，工具结果由 delivery 层反向投递给微信
  - `channel="scheduler"`（`scheduler.py`）—— 同理，定时任务用 send_message 推送
- cache_key 中加入 `::ch={channel}`（仅当非默认 web 时），避免不同渠道复用错误的 agent

## service-chat — React 入口（已统一渲染管线）
**❗ 旧的 `frontend/public/service-chat.html` 已删除，不要重新引入**。当前架构：

### 文件布局
- `frontend/service-chat.html` — vite multi-entry 顶层 HTML（含 `<!-- SVC_INJECT -->` 占位）
- `frontend/src/service-chat/main.tsx` — React entry，`createRoot(...).render(<ServiceChatApp/>)`
- `frontend/src/service-chat/ServiceChatApp.tsx` — 主组件
- `frontend/src/service-chat/ServiceToolBadge.tsx` — service 端友好状态条（替代 admin 的 ToolIndicator）
- `frontend/src/service-chat/streamHandler.ts` — 轻量 SSE handler（hook 形式，无 admin 的 subagent/HITL/interrupt 状态）
- `frontend/src/service-chat/serviceApi.ts` — consumer-side API（与 `services/api.ts` 解耦，避免 token 串台）
- `frontend/src/service-chat/serviceChat.module.css` — 服务端专属样式
- `vite.config.ts` 的 `rollupOptions.input` 必须同时声明 `main` 和 `service-chat` 两个 entry
- `app/routes/consumer_ui.py` 读 `frontend/dist/service-chat.html`，把 `<!-- SVC_INJECT -->` 替换为 `<script>window.__SVC__={...}</script>`
- dev 模式（dist 不存在）：`consumer_ui.py` 返回引用 `http://localhost:3000/src/service-chat/main.tsx` 的兜底 HTML，依赖 vite dev server 跨域 ESM

### 跨 admin / service 共享组件（**改一次两边都生效**）
- `frontend/src/pages/Chat/markdown.ts` — 唯一的 markdown 渲染管线，含 `<<FILE:path>>` 媒体标签处理 / DOMPurify 配置 / hljs 语言注册
- `frontend/src/pages/Chat/components/StreamingMessage.tsx` — 共享渲染组件，接受 `toolRenderer` / `hideSubagents` / `avatarSrc` props 让两端定制差异部分
- `frontend/src/pages/Chat/types.ts` 的 `StreamBlock` 数据结构

### 渲染差异（设计意图）
- **工具块**：admin 用默认 `ToolIndicator`（真实工具名 + 可展开 args/result，调试用）；service 传 `toolRenderer={ServiceToolBadge}`（白名单友好文案，未在白名单的统一「思考中…」，不展示 args/result）
- **subagent**：service 传 `hideSubagents` 隐藏，避免向消费者泄露内部子流程；admin 显示完整 `SubagentCard`
- **媒体 URL 鉴权**：`markdown.ts` 暴露 `setMediaUrlBuilder(fn)`，admin 默认走 admin token（`adminMediaUrl`），service-chat `main.tsx` 启动时调用 `setMediaUrlBuilder(buildConsumerMediaUrl)` 覆盖为 `/api/v1/conversations/{conv_id}/files/{path}?key=...`（query 参数携带 service API key，因为 `<img src>` 不支持 Authorization header）

### 改动检查清单（修 admin chat 时）
- 改 `markdown.ts` → service 自动同步 ✅
- 改 `StreamingMessage.tsx` 的 props/逻辑 → 同步影响 service；新增 prop 务必给默认值或在 `ServiceChatApp.tsx` 中传值
- 新加 SSE 事件类型 → 同时改 `streamContext.tsx`（admin）和 `streamHandler.ts`（service）；后者意图保持轻量，不要把 admin 的 HITL/interrupt 引入
- 千万别为图省事在 `service-chat/` 复制一份 `markdown.ts` 之类——这就是之前 `<<FILE:>>` 媒体 bug 在 wechat 和 web consumer 各坏一次的根因

### 历史踩坑（已修复）
- 旧 `public/service-chat.html` 用 CDN 的 `marked + DOMPurify`，没处理 `<<FILE:path>>` 媒体标签，consumer 只看到字面文本——同 wechat delivery 的同源 bug
- 旧版 sanitize 没加 iframe/audio/video 白名单，即便补了媒体处理也会被剥
- 旧版 `pathFilter: '/wc'` 是前缀匹配，`/wcanything` 也会被代理；改成 `/wc/`
- **ServiceToolBadge 白名单初版工具名拍脑袋写错**（`read_doc` / `list_docs` 等）：实际工具名以 `consumer_agent.py` (`ls`, `read_file`) + deepagents 内置 (`glob`, `grep`, `write_file`, `edit_file`, `write_todos`, `task`) + `tools.py` 的 `generate_image/speech/video` 和 `run_script` 为准。修工具时务必同步该白名单，否则 consumer 端全部显示"思考中…"
- **streamHandler.ts onDone 闭包陷阱**：早期版本 `onDone: () => stream.blocks` 会拿到首次 render 闭包到的 stale state（永远是 `[]`），导致 streaming 完成后整段 assistant 消息消失。修复：让 hook 直接把 `finalBlocks` 作为参数传给 `onDone`，**消费者绝对不要从外层 stream 引用读 blocks**。同时 hook 内部用 `optsRef.current.xxx` 而非闭包到 opts，避免 send 函数因 opts 每次新对象而重建
- **架构差异提醒**：admin /chat 的"流结束 → 消息持久化"走的是后端 `save_message(blocks=...)` + 前端 `getConversation(convId)` refetch 的路径，没有客户端 commit step；service-chat 没用同样的 refetch（消费者侧没有"会话历史"页面），所以必须在 onDone 客户端 commit 一份到 messages list
- Admin 微信接入：admin_router.py + admin_bridge.py（独立于 Service 渠道）
  - Admin 扫码连接自己的主 Agent（完整 docs/scripts/tools 权限 + humanchat）
  - 对话存入 Admin 的 conversations 目录（和普通对话共存）
  - 端点：POST /api/admin/wechat/qrcode、GET qrcode/status、GET/DELETE session、GET messages
  - **会话持久化**：`users/{user_id}/admin_wechat_session.json`，Docker/服务重启后自动恢复
  - `_save_admin_session()` 在会话创建、`from_user_id` 首次捕获时写入
  - `restore_admin_sessions()` 在 `main.py` startup 时调用，扫描所有用户目录恢复
  - `shutdown_admin_sessions()` 仅停止轮询和关闭连接，**不删除**持久化文件
  - `_remove_admin_session()` 用于主动断开或连续错误 20 次，会删除持久化文件
  - 前端：index.html 侧边栏"微信接入"按钮 → 弹窗面板（QR/状态/对话记录）
  - JS：frontend/public/js/admin-wechat.js
  - CSS：style.css .awc-* 样式

### 多模态 Vision 支持（全链路）
- 后端 schema：ChatRequest.message 和 ConsumerChatRequest.message 改为 Any（接受 str 或 multimodal list）
- chat.py / consumer.py：_extract_text() 提取纯文本用于持久化，原始 content 直接发给 LangChain agent
- 微信 bridge/admin_bridge：_build_multimodal_content() 将图片 base64 编码为 image_url block
- 前端 Admin（chat.js）：_pendingImages + 粘贴/拖拽/按钮上传 → sendMessage 构建 multimodal list
- 前端 Consumer（service-chat.html）：同上逻辑
- 格式：OpenAI compatible [{"type":"text","text":"..."}, {"type":"image_url","image_url":{"url":"data:...base64"}}]
- GPT-4o / Claude Sonnet/Opus 均原生支持

## Soul 记忆系统（memory_tools）
- **soul config 目录**：`users/{user_id}/soul/config.json`（应用层元数据，agent 不可直接访问）
  - `config.json`：`memory_enabled`、`include_consumer_conversations`、`max_recent_messages`（默认 5）、`memory_subagent_enabled`（Memory Subagent 写入开关）、`soul_edit_enabled`（Soul 文件系统暴露开关）
- **soul 内容目录**：`users/{user_id}/filesystem/soul/`（agent 可读写的笔记/人格文件）
  - 物理存放在 filesystem 根目录内，避免 deepagents FilesystemBackend 路径逃逸检查
  - 旧版使用 symlink `filesystem/soul → ../soul`，在 Docker/Linux 上因 `Path.resolve()` 跟随符号链接导致路径逃逸报错，已废弃
  - `sync_soul_symlink()` 现在负责：移除旧 symlink、创建 `filesystem/soul/` 目录、自动迁移旧 `soul/` 内容文件
- **Memory Subagent**：
  - Admin：内置 memory subagent（`subagents.py` DEFAULT_SUBAGENTS），基础工具 = `list_conversations`/`read_conversation`/`list_service_conversations`/`read_service_conversation`/`read_inbox`
  - `memory_subagent_enabled=true` 时追加 soul 写入工具：`soul_list`/`soul_read`/`soul_write`/`soul_delete`
    - 对话记录保持只读，Subagent 不能修改聊天历史
    - 只能在 `filesystem/soul/` 内创建、编辑、删除文件（`config.json` 禁止修改）
    - soul 工具的 `soul_root` 指向 `_soul_content_dir()`（= `filesystem/soul/`）
  - Consumer：内置 memory subagent，工具 = `read_my_conversation`（仅读自己对话）
  - Admin 的 `include_consumer_conversations=false` 时，Service 对话工具返回提示
- **Soul 文件系统暴露**：
  - `soul_edit_enabled=true` 时，`filesystem/soul/` 自然可被 Agent 的 `FilesystemBackend` 访问，文件面板也可见
  - `create_user_agent` 和 API `PUT /api/soul/config` 都会调用 `sync_soul_symlink`
- **Soul 能力提示词**（`CAPABILITY_PROMPTS` 扩展）：
  - `memory_subagent`：当 `memory_subagent_enabled=true` 时注入，告诉 Agent 有 memory subagent 可委托、soul/ 可写入
  - `soul_edit`：当 `soul_edit_enabled=true` 时注入，告诉 Agent 有 /soul/ 目录可直接读写
  - 注入逻辑在 `agent.py::create_user_agent` 中，读取 `soul_config` 判断
- **前端 Soul 设置**：
  - 设置 → Prompt → 第三 Tab「Memory & Soul」（`SoulSettings.tsx`）
  - 两个 Switch：Memory Subagent 写入（`memory_subagent_enabled`）、Soul 文件系统（`soul_edit_enabled`）
  - 每个 Switch 下方附带可编辑的提示词（仅在开关打开时显示），支持自定义/恢复默认
  - 附加 Switch：包含消费者对话（`include_consumer_conversations`）
  - API：`GET /api/soul/config`、`PUT /api/soul/config`（`settings_routes.py`）
  - 更新配置后自动 `clear_agent_cache` 使 Agent 重建时生效
- **能力提示词系统（Capability Prompts）**：
  - 所有 `CAPABILITY_PROMPTS`（`tools.py`）支持 per-user 自定义覆盖
  - 存储：`users/{user_id}/capability_prompts.json`（仅存覆盖项，不存默认）
  - 后端：`prompt.py` 的 `get_capability_prompts`/`save_capability_prompts`/`get_resolved_capability_prompt`
  - API：`GET /api/capability-prompts`（列出所有 + 标记是否自定义）、`PUT /api/capability-prompts/{key}`（保存覆盖）、`DELETE /api/capability-prompts/{key}`（恢复默认）
  - `agent.py::create_user_agent` 使用 `get_resolved_capability_prompt` 替代直接读 `CAPABILITY_PROMPTS`
  - 前端操作规则 Tab（`SystemPromptEditor.tsx`）：主 prompt 编辑器下方增加「能力提示词」折叠面板，展开可逐条编辑/恢复默认
  - 前端 Soul Tab（`SoulSettings.tsx`）：soul 相关提示词直接嵌入开关下方
  - Soul 相关 key（`memory_subagent`、`soul_edit`）不在操作规则 Tab 显示，避免重复
- **短期记忆注入**：
  - `scheduler.py` 的 `_run_service_agent_task` / `_run_agent_task`：从 conversation JSON 读最近 N 条消息，拼入 prompt 前缀
  - `inbox.py` 的 `_trigger_inbox_agent`：注入最近 3 条 inbox 历史
- **消息来源标注**：
  - Service 定时任务 prompt 头部加 `[系统指令 - 来自管理员]`，明确 send_message 发给用户、contact_admin 发给管理员
  - Inbox agent prompt 头部加 `[系统指令 - Service 收件箱通知]`，明确 send_message 发给管理员本人
- **任务结果持久化**：`_run_agent_loop` 结束后调用 `save_message` / `save_consumer_message` 写入对话 JSON（含 `source: "scheduled_task"` 或 `"admin_broadcast"` 标记）
- **Inbox thread_id 稳定化**：`inbox-{admin_id}`（同一 Admin 共用，有累积记忆）
- **publish_service_task**：
  - `service_ids` 支持 ID 和名称匹配（大小写不敏感），未匹配时返回可用 Service 列表
  - `session_ids` 可选参数，精确到单个微信会话
  - `run_now` 调度使用 `_schedule_coro` 线程安全模式（先尝试 `get_running_loop()`，失败回退 `run_coroutine_threadsafe`）
- **工厂函数**：`create_admin_memory_tools(user_id)` 返回 5~9 个工具（取决于 `memory_subagent_enabled`），`create_consumer_memory_tools(admin_id, svc_id, conv_id)` 返回 1 个工具
- **soul/ 初始化**：`_create_user_dirs` 调用 `ensure_soul_dir(user_id)` 自动创建
- **Subagent 可用工具**（`SHARED_TOOL_NAMES` + `MEMORY_TOOL_NAMES`）：
  - 通用工具：`run_script`、`web_search`、`web_fetch`、`generate_image`、`generate_speech`、`generate_video`、`schedule_task`、`manage_scheduled_tasks`、`publish_service_task`、`send_message`
  - 记忆工具：`list_conversations`、`read_conversation`、`list_service_conversations`、`read_service_conversation`、`read_inbox`、`soul_list`、`soul_read`、`soul_write`、`soul_delete`
  - `build_subagent_tools` 按需创建：仅实例化 subagent 配置中列出的工具，避免不必要的初始化
  - 前端 SubagentManager 显示全部可选工具（`GET /api/subagents` 返回 `available_tools`）

## Per-User Python 环境（venv_manager）
- **模块**：`app/services/venv_manager.py`
- **目录**：`users/{user_id}/venv/`，每个 Admin 独立的 Python 虚拟环境
  - 使用 `--system-site-packages` 创建，继承系统预装包（numpy、pandas 等）
  - 用户自定义安装的包持久化到 `users/{user_id}/venv/requirements.txt`
- **脚本执行集成**：
  - `script_runner.run_script` 新增 `python_executable` 参数
  - `tools.py::create_run_script_tool` 和 `consumer_agent.py` 均使用 `get_user_python(user_id)` 获取用户 Python
  - Consumer 脚本使用 admin 的 venv（`get_user_python(admin_id)`）
- **API**：
  - `GET /api/packages` — 列出已安装包 + venv 状态
  - `POST /api/packages/init` — 初始化用户 venv
  - `POST /api/packages/install` — 安装包（含输入安全检查）
  - `POST /api/packages/uninstall` — 卸载包
- **启动恢复**：`main.py` startup 调用 `restore_all_venvs()`，扫描所有用户目录，对有 `requirements.txt` 的用户自动 `pip install -r` 还原
- **前端**：设置 → Python 环境（`PackagesPage.tsx`），支持初始化环境、安装/卸载包、查看已安装包列表
- **安全**：包名禁止 `;|&$\`` 等注入字符；pip 操作在用户 venv 内执行，不影响系统环境

## YOLO 模式（admin 自动批准, 2026-04-21）
- **痛点**：单人开发时 HITL 审批卡（write_file / edit_file / propose_plan）频繁打断流，每次都要手动点「批准」。
- **架构** — 同步前后端，复用现有 `Command(resume=...)` 协议，零侵入 deepagents：
  - `app/schemas/requests.py::ChatRequest/ResumeRequest` 各加 `yolo: Optional[bool]`
  - `app/routes/chat.py::_stream_agent(yolo=False)` —— 把原 `async for event in agent.astream(...)` 抽成内部 helper `_drain_one_pass(payload)`（async generator，通过 nonlocal 共享 closure state，保持原 body 缩进 0 改动），外层 try 内套 `while True:` 循环：
    1. `async for sse in _drain_one_pass(input_payload): yield sse`
    2. 用 `agent.aget_state(config)` 检测 interrupt
    3. `not has_interrupt` → break（正常退出 + done 事件 + 存盘）
    4. `not yolo` → 发 `interrupt` SSE + 写 `_interrupt_state` + return（**完全保留原行为**）
    5. YOLO 路径：构造 `[{"type":"approve"}] * N` decisions、push `auto_approve` block 到 blocks 持久化数组、yield `auto_approve` SSE、`input_payload = Command(resume=...)`、`continue`
    6. 防御性 `MAX_YOLO_LOOPS=50`，超出发 error 事件
  - cancel 处理：原 `return` 之前加 `_cancelled = True`（nonlocal），外层 while 检测 → 立即 return（避免 helper 的 `return` 让 while 死循环）
  - `api_chat` / `api_chat_resume` 透传 `yolo=bool(req.yolo)`
- **前端**：
  - `frontend/src/utils/yoloMode.ts` — `getYoloMode/setYoloMode`，localStorage key `yolo_mode_admin`，全局事件 `yolo-mode-changed` 让多个面板同步
  - `frontend/src/types/index.ts` — `ChatOptions.yolo`、`SSECallbacks.onAutoApprove`、`MessageBlock` 加 `auto_approve` variant（持久化历史可看）
  - `frontend/src/pages/Chat/types.ts` — `StreamBlock | AutoApproveBlock`
  - `frontend/src/services/api.ts` — body 里塞 `yolo: true`（streamChat / resumeChat），SSE switch 加 `case 'auto_approve': onAutoApprove?.(count, actions); break;`
  - `frontend/src/stores/streamContext.tsx` — `StreamOpts.yolo` 透传；`onAutoApprove` 回调 close thinking 后 push `{type:'auto_approve', count, actions}` block + scheduleFlush
  - `frontend/src/pages/Chat/index.tsx` — 两处 `startStream` / `resumeStream` 调用都带 `yolo: getYoloMode()`
  - `frontend/src/pages/Settings/GeneralPage.tsx` — 「YOLO 模式」卡片放在「界面样式」之后、「批量运行」之前，黄色 Tag「已开启」+ ⚠ 警告语 + 「仅作用于 Admin 端」说明。监听 `YOLO_EVENT` 同步状态
  - `frontend/src/pages/Chat/components/AutoApproveBadge.tsx` — 小徽章「⚡ YOLO 自动批准 N 项」可展开看具体 tool name 列表；橘黄色，参考 ToolIndicator 交互
  - `StreamingMessage.tsx` + `MessageBubble.tsx::BlocksRenderer` 都加 `case 'auto_approve'` → `<AutoApproveBadge>`
  - `chat.module.css` 末尾 `.autoApprove*` 样式（橘黄主色 #f59e0b，与 admin 蓝色主色形成视觉区分）
- **关键设计决策**：
  - **不影响 service / consumer**：consumer agent 本身就没配置 `interrupt_on`（见 `consumer_agent.py` 注释 "No HITL interrupts on writes"），不存在审批，YOLO 开关对其无意义
  - **不复用微信 admin_bridge 自动批准代码**：bridge.py 是同步 `ainvoke` 模式（非流式），chat.py 是 SSE 流式，两套接口；但**自动批准的语义/decisions 构造完全一致**（`[{"type":"approve"}] * N`）
  - **历史回看也保留 auto_approve 标记**：blocks 数组里持久化 `{type:'auto_approve', count, actions:[{name, args}]}`，刷新页面后仍能看到「⚡ 自动批准了哪些操作」
  - **YOLO 状态在前端，不持久化到后端**：localStorage 即可；用户每个浏览器/设备独立选择，不需服务端一致性
- **不要做的事**：
  - 不在 chat 顶部加横幅 banner（用户明确否决）
  - 不在 service-chat 端加任何 YOLO UI（service 默认就是无审批，加开关只会让消费者困惑）
  - 不重新拆 _stream_agent 函数（helper async generator + nonlocal 已经是最小侵入）
  - 不修改 deepagents `interrupt_on` 配置（HITL 中间件继续生效，前端只是看到再 resume）

### YOLO 行为调整（2026-04-22, 用户反馈：「不设上限，不要显眼徽章」）
- **后端 `app/routes/chat.py::_stream_agent`**：
  - 删除 `MAX_YOLO_LOOPS=50` 硬上限及超出后 yield error / save_message 的中止分支。
  - 改为 `YOLO_WARN_EVERY=50`，每达 50 次 `yolo_loops % 50 == 0` 就 `_log.warning(...)` 一次，便于排查死循环但不打断流。
  - **不再向 `blocks` 数组追加 `auto_approve` 块**（避免历史回看出现 ⚡ 大徽章）；仅保留 SSE `auto_approve` 事件（用于驱动前端底部小 tag）。
- **前端 `streamContext.tsx`**：
  - 新增 `yoloApprovedConvs: Set<string>` 状态（context 暴露），记录本次浏览器会话内发生过 YOLO 自动批准的 convId。
  - `onAutoApprove` 回调改为「只 close thinking + add convId 到 yoloApprovedConvs」，**不再 push `{type:'auto_approve', ...}` 到 blocks**。
- **前端 `Chat/index.tsx`**：
  - 新增 `yoloOn` 本地 state 监听 `YOLO_EVENT` 同步开关。
  - 输入区 `inputArea` 末尾（`inputWrapper` 之后）加一行小 tag `<div className={styles.yoloFooterTag}>` —— 仅在 `yoloOn && currentConvId && yoloApprovedConvs.has(currentConvId)` 时渲染，含一个橙色小圆点 + 文本 `yolo`，title 解释含义；浏览器刷新或后端没再 yield `auto_approve` 时不显示。
- **前端 `MessageBubble.tsx` / `StreamingMessage.tsx`**：`case 'auto_approve': return null`（彻底隐藏；包括反序列化的旧消息），并删除对 `AutoApproveBadge` 组件的 import。
- **删除文件**：`frontend/src/pages/Chat/components/AutoApproveBadge.tsx`、`chat.module.css` 中 `.autoApprove*` 一整段。
- **保留**：`StreamBlock` 联合中仍保留 `AutoApproveBlock` 类型（仅向后兼容旧消息反序列化），`types.ts` 的 JSDoc 已更新说明它现在不被渲染。
- **新增 CSS**：`chat.module.css::.yoloFooterTag` —— `align-self: flex-end`、`opacity: 0.7`、`font-size: 10px`、圆角 pill、橙色 5px 小点 `.yoloFooterDot`。极简、不抢视觉。
- **教训**：自动批准这种「持续 N 轮」的流程，不要在每轮 push 持久化块到消息流——会产生大量重复条目污染历史。状态信号放在「全局 UI 角落」（如本案的输入区底部 tag）即可。

## 聊天里 `<<FILE:>>` tag 一键定位（2026-04-22）
- **痛点**：agent 回复里出现 `<<FILE:/path/to/file.txt>>`，非媒体类型在 `markdown.ts::filePathToHtml` 返回 null 后退化为 `<FILE:...>` 文本（被 marked 渲染为单尖括号）；用户没有快捷方式跳到 FilePanel 对应位置。媒体（图片/音视频/pdf/html）虽已直接渲染，也缺少「在文件浏览器看一眼上下文」的入口。
- **方案**（admin /chat 与 service-chat 共用渲染层 `pages/Chat/markdown.ts`，但 admin 才有 FilePanel）：
  - **`stores/fileWorkspaceContext.tsx`**：
    - 把 FilePanel 的 local `currentPath` 提升到 context（`browserPath` + `setBrowserPath`），FilePanel 改为消费 context 中的字段（别名解构 `browserPath: currentPath, setBrowserPath: setCurrentPath` 保留组件内已有命名，最小改动）。
    - 新增 `revealInBrowser(path)`：`setBrowserPath(parentDir(path))` + `setFileBrowserOpen(true)` + `await openFile(path)` —— 一次点击同时「打开文件 + 跳到所在目录 + 展开右侧文件浏览器」。
    - 在 Provider 内 `useEffect` 安装 **document 级别的全局点击委托**：捕获 `target.closest('[data-jf-file]')`，读 `data-jf-file` 调 `revealInBrowser`，并 `e.preventDefault() / e.stopPropagation()`。这样 markdown 渲染（无 React 上下文）也能触发跳转，无需在每个 `dangerouslySetInnerHTML` 容器上挂 onClick。
  - **`pages/Chat/markdown.ts`**：
    - 新增 `_fileRevealEnabled` 开关 + `setFileRevealEnabled(enabled)` 导出。默认 `true`（admin 端）；service-chat 在 `ServiceChatApp.tsx` 的 mediaUrlBuilder useEffect 内 `setFileRevealEnabled(false)` 关掉，避免渲染出无效的可点击 pill。
    - 新增 `nonMediaFileToHtml(filePath)`：把非媒体 `<<FILE:>>` 渲染成 `<button type="button" class="jf-file-link" data-jf-file="${escaped}">📄 ${name}</button>` 内联 pill；关闭时回退到 `&lt;FILE:...&gt;` 文本。
    - 新增 `buildRevealAction(filePath)`：给媒体 `.jf-media-caption` 拼接一个 `<button class="jf-file-reveal" data-jf-file="...">📁</button>` 小按钮，让图片/音视频/pdf/html 也能一键定位到 FilePanel。
    - **重要**：`<a href="javascript:void(0)">` 会被 DOMPurify 默认按 URI 安全策略剥掉 href —— 改用 `<button type="button">` + `data-jf-file` 委托。`data-*` 属性 DOMPurify 默认放行，无需扩 ADD_ATTR。
  - **`chat.module.css`**：新增 `:global(.jf-file-link)` / `:global(.jf-file-link-icon)` / `:global(.jf-file-link-name)` / `:global(.jf-file-reveal)` 全局类（与 `.jf-media*` 同处一段，使用 `:global()` 规避 CSS Modules 哈希）。pill 用主色 `var(--jf-primary)` 浅紫调，name 限宽 320px 配 ellipsis；reveal 按钮 transparent 背景 hover 才上色，避免 caption 拥挤。
- **关键设计决策**：
  - **不在每个消费 dangerouslySetInnerHTML 的组件挂 onClick**：使用 document 级委托一次性覆盖 MessageBubble / StreamingMessage / SubagentCard / FilePreview 全部入口，零侵入。
  - **复用现有 openFile**：context 里的 openFile 已经处理 split 模式切换 + media 类型不读文本的分支；revealInBrowser 直接复用，错误 toast 也由 openFile 统一弹。
  - **在父目录跳转**：browserPath 设为 `parentDir(path)`（自实现 `lastIndexOf('/')`），让用户在 FilePanel 看到同级文件上下文；同时 FilePanel 内部 `editingFile === joinPath(currentPath, item.name)` 自动高亮当前打开项。
- **不要做的事**：
  - 不要在 markdown 输出里写 `onclick="..."` 内联 handler 调用全局函数 —— 路径里的 `'` 会破坏 JS 字符串字面量；用 data-attribute + 委托更鲁棒。
  - 不要试图扩展 DOMPurify 的 URI scheme 白名单去支持 `javascript:` —— 直接换 `<button>` 即可。

## write_file / edit_file 流式可视化（StreamingFilePreview, 2026-04-21）
- **痛点**：deepagents 的 write_file / edit_file 是 LLM token-by-token 生成 args，但前端一直把 args 累在折叠的 ToolIndicator 里看不到。HITL 审批卡又要等 LLM 写完才弹出，用户全程等待。
- **方案**：纯前端改造（后端 SSE 已经在推 `tool_call_chunk` + `args_delta`，零改动）。三层组件：
  - `frontend/src/utils/partialJson.ts::extractStreamingField(raw, field)` — 状态机扫 incomplete JSON，提取字符串字段当前到达部分。处理 \" \\n \\\\ \\u00xx 等转义；末尾孤立反斜杠/半个 unicode 容错；区分 key vs value（避免 `"content":"x"` 中的 `"content"` 在 value 里被误判）。13 个 case 全过。
  - `frontend/src/pages/Chat/components/StreamingFilePreview.tsx` — 双层：
    - `FilePreviewBody`（命名 export）：纯展示，接收 `{filePath, text, kind: 'write'|'edit', status: 'streaming'|'pending'|'done'|'error', errorMessage?, langOverride?, showCursor?}`，做语法高亮 + 状态徽章 + 打字机光标。ApprovalCard 直接复用。
    - `StreamingFilePreview`（默认 export）：包装层，从 ToolBlock.args 解析 partial JSON 后传给 FilePreviewBody。
  - 高亮：复用 `markdown.ts` 已注册到 highlight.js 全局单例的语言（py/ts/json/bash/yaml/md/html/css/java/go/rust/cpp/sql/toml/dockerfile）。后缀 → lang 映射表 `EXT_TO_LANG`，未匹配走 `hljs.highlightAuto`。**副作用 import `'../markdown'`** 保证语言一定注册（即使本组件被先于 markdown.ts 加载）。
- **路由入口**：在三处独立判断 `name === 'write_file' || name === 'edit_file'`：
  1. `StreamingMessage.tsx`（live admin/consumer 流式视图）— `isStreaming = isLast && isStreaming`
  2. `MessageBubble.tsx::BlocksRenderer`（历史消息回看）— `isStreaming = false`
  3. `ApprovalCard.tsx::FileActionCard`（HITL 审批）— write_file 用 FilePreviewBody（status='pending'）；edit_file 用 `EditDiffViewer`（status='pending'）。批准前可点「编辑」切到 TextArea 改 new_string
- **状态语义**：
  - `streaming`：流式中（光标 + 旋转图标 + 主色）
  - `pending`：HITL 等待审批（"待审批" + 警告色）
  - `done`：写入/编辑成功（"已写入"/"已编辑" + 成功色）
  - `error`：result 以 error/failed/失败/出错 开头自动识别（红色 + 错误信息）
- **样式**：`chat.module.css` 末尾新增 `streamFile*` 与 `approvalFileBlock`/`approvalActionBar`，全用 themes.css 变量；打字机光标 `streamCursor` 用 `▍` 字符 + 1s 闪烁动画。
- **service-chat 不需要改**：因为路由判断在 StreamingMessage 内部，consumer 端自动获益（流式打字机 + done 状态，无审批）。`ServiceToolBadge` 不会被 file 工具调用，不必为 write_file/edit_file 列白名单。
- **性能**：`useMemo(parseArgs(args), [args])` 避免每次重渲染重新解析；highlight 同步执行（hljs 很快），未做 web worker。文件 > 480px 高度自动滚动。
- **不要做的事**：
  - 不引第三方 partial-json 库（自己写状态机，~100 行，零依赖）
  - 不在 LLM 调用约定上加约束（不走 Cursor 模式）
  - 不取消 HITL 审批
  - 不对 edit_file 在 ApprovalCard 中改用 FilePreviewBody（diff 视图更直观）

## edit_file git 风格 diff（EditDiffViewer, 2026-04-21）
- **痛点**：`edit_file` 的预览只有 `old_string` ↔ `new_string` 两段对照，看不出在原文件哪一行、附近代码是什么；用户没法判断 agent 改的是不是正确位置。
- **方案**：新增 `EditDiffViewer` 组件，**自己拉原文件 + 算 unified diff + 渲染 git 风格 hunk**。流式中继续打字机 new_string，流完切到 diff（避免无效 fetch）。
- **算法（`frontend/src/utils/unifiedDiff.ts`）**：
  - `lineDiff(oldLines, newLines)` — 经典 LCS（dp 二维 Int32Array），回溯顺序保证「del 在 add 之前」（git 习惯）。文件 < 5000 行实测足够快。
  - `computeUnifiedDiff(originalText, oldString, newString, contextLines=3)` — 在原文中 `indexOf(oldString)` 定位 → 替换得到新文件 → `lineDiff(整个旧, 整个新)` → `sliceHunks` 按上下文行数切 hunk（相邻自动合并）。
  - `sliceHunks` — 标记每个改动行 `[i-ctx, i+ctx]` 范围 keep=1，连续 keep 段切成 hunk；hunk header 复刻 `@@ -3,4 +3,4 @@`。
  - 14 个单元测试覆盖：等长替换 / 插入 / 删除 / 多行 old / 远距离改动 / 上下文重叠 / not_found 容错。
- **组件（`frontend/src/pages/Chat/components/EditDiffViewer.tsx`）**：
  - 异步 GET `/api/files/read` 拿原文件 → useMemo 算 diff → 渲染。
  - 容错：原文找不到 old_string（agent 写错或文件已变） / 文件读不到 / 是新文件 → 降级到「双段对照」(old 全部 - / new 全部 +)，**不阻断 UI**。
  - UI：复用 `streamFileCard*` 头部样式（FileCode 图标 + 路径 + lang + 状态徽章）；diff 行用 4 段 grid（旧行号 / 新行号 / 符号 / 代码），代码段走 hljs 高亮（与 markdown 共享语言注册）。状态徽章带 `+N -M` 行数统计。
  - 「展开全文」按钮切换 hunk 视图 ↔ 完整 diff；max-height 480px 内滚动。
- **接入点**：
  - `StreamingFilePreview`：`!isWrite && status !== 'streaming' && filePath && oldString && newString` → 直接 `return <EditDiffViewer />`（流式期保留打字机 new_string）。这覆盖了 live 流（StreamingMessage） + 历史回看（MessageBubble.BlocksRenderer），因为后端 `chat.py` 在持久化 blocks 时把 `args` 累成完整 JSON 字符串保留。
  - `ApprovalCard.FileActionCard`：edit_file 分支由原 SBS diff 改为 `<EditDiffViewer status="pending" />`，按钮区与 write_file 一致（`approvalActionBar`）。`computeLineDiff` 旧函数与 SBS 渲染已删除。
- **样式**：`chat.module.css` 末尾新增 `diff*` 系列（`diffBody/diffHunk/diffHunkHeader/diffRow{Add,Del,Context}/diffRow{Old,New}Num/diffRowSign/diffRowText/diffExpandBtn/diffStat{Add,Del}/diffLoading/diffWarn/diffEmpty`）；颜色用 `--jf-success/--jf-error` 半透明背景。
- **不要做的事**：
  - 不在 write_file 上做 diff（即使覆盖现有文件也保持「全文新建」视图，逻辑简单清晰）
  - 不在流式期间做 diff（new_string 还在变 + 频繁 fetch 浪费）
  - 不上 jsdiff（~50KB gz，自己 LCS ~120 行够用）
  - 不在 EditDiffViewer 里加编辑功能（编辑 new_string 由 ApprovalCard 外层 TextArea 负责，组件只管展示）

## 文件元信息排序工具（list_files_sorted, 2026-04-21）
- **动机**：deepagents 内置的 `ls` 只返回文件名列表，agent 拿不到大小/时间，无法回答「最新生成的图是哪个」「找最大的 csv」之类问题；不动框架内置工具，新增一个语义更明确的工具。
- **后端实现**：
  - `app/services/tools.py::create_list_files_sorted_tool(user_id)` — admin 版本，走 `get_storage_service().list_dir(user_id, path)`（自动支持 local/S3 两种 backend）
  - `app/services/consumer_agent.py::_create_consumer_read_tools` 内 `list_files_sorted` — consumer 版本，仅扫描 `docs_dir` 并经过 `_is_allowed` 过滤，禁止访问 `generated/`
  - 共享辅助函数 `_format_size` / `_format_mtime_short` / `_SORT_KEYS` 都在 `tools.py`，consumer 版本直接 import
- **工具签名**：`list_files_sorted(path="/", order_by="modified", desc=True, limit=50)`，`order_by` 接受 `name|modified|mtime|size`（`mtime` 是 `modified` 的别名），`limit` 上限 500
- **输出格式**：`类型 大小 修改时间 名称` 表格，目录显示为 `DIR /`，文件大小走 `K/M/G/T` 单位，时间用 `YYYY-MM-DD HH:MM`（去秒/微秒/时区，节省 token）
- **注册入口**：`app/services/agent.py::create_user_agent` 在 `tools = [...]` 初始化时注入；consumer 通过 `_create_consumer_read_tools` 返回值自动注入
- **前端联动**：`frontend/src/service-chat/ServiceToolBadge.tsx` 的 `TOOL_LABELS` 白名单需同步加 `list_files_sorted: '正在整理文件…'`，否则 consumer 会兜底显示「思考中…」
- **不做的事**：不存储 `created_at`（macOS `st_birthtime` / Linux 不可靠 / S3 没有），只暴露 `modified_at`；不加 REST API 排序参数，前端用 `useMemo` 客户端排序

## 联网工具（web_tools）
- 新增 `app/services/web_tools.py`：`web_fetch(url)` / `web_search(query, count)`
- 双 provider：优先 CloudsWay（`CLOUDSWAY_SEARCH_KEY`），fallback Tavily（`TAVILY_API_KEY`）
- 环境变量：`CLOUDSWAY_SEARCH_KEY`、`CLOUDSWAY_READ_URL`（可选覆盖）、`CLOUDSWAY_SEARCH_URL`（可选覆盖）、`TAVILY_API_KEY`
- Admin agent：**永远注入** web 工具（无需在 capabilities 里填 "web"）
- Consumer/Service agent：`service_config.research_tools=true` 或 `capabilities` 含 "web" 时注入

## 定时任务（scheduler）
- `app/services/scheduler.py`：TaskScheduler 单例，asyncio 循环每 30s 检查所有用户任务

### Admin 任务
- 存储：`{user_dir}/tasks/{task_id}.json`（以 `task_` 前缀），含最近 20 条运行记录
- 支持调度类型：`once`（一次性 ISO 时间）、`cron`（需 `pip install croniter`）、`interval`（秒数）
- 支持任务类型：`script`（执行 scripts/ 脚本）、`agent`（Batch Agent + 可选文档上下文）
- API：`GET/POST /api/scheduler`、`GET/PUT/DELETE /api/scheduler/{id}`、`GET /api/scheduler/{id}/runs`、`POST /api/scheduler/{id}/run-now`
- Admin agent **默认**注入 `schedule_task`（创建）和 `manage_scheduled_tasks`（list/update/delete）工具
- `reply_wechat=True` 参数可让结果推送回管理员微信

### Service 任务（与 admin 任务分离）
- 存储：`{user_dir}/services/{service_id}/tasks/{task_id}.json`（以 `stask_` 前缀）
- 仅支持 `agent` 类型（无脚本），使用 consumer agent 执行
- `reply_to` 字段记录 {channel, admin_id, service_id, conversation_id, session_id}
- 执行完自动通过 `_deliver_reply` 推送结果到 WeChat
- Consumer agent 通过 `create_service_schedule_tool` 注入 `schedule_task`，通过 `create_service_manage_tasks_tool` 注入 `manage_scheduled_tasks`（仅 `"scheduler"` 在 capabilities 中时）
- Service 的 `manage_scheduled_tasks` 仅能操作当前 `conversation_id` 的任务（权限隔离）
- API：`GET /api/scheduler/services/all`、`GET/POST /api/scheduler/services/{svc_id}`、`GET/PUT/DELETE /api/scheduler/services/{svc_id}/{task_id}`

### 前端
- `scheduler.html` 侧边栏有 **管理员任务 / 服务任务** 两个标签页
- 服务任务显示 service_id、📬 推送标记、reply_to 信息

### 消息级时间戳注入（2026-04-13 重构）
- **设计**：精确时间不再写入 system prompt（按天缓存会冻结时间），改为每条用户消息前注入时间戳 `[YYYY-MM-DD HH:MM:SS]`
- **system prompt**：`DEFAULT_SYSTEM_PROMPT` 只保留 `{today}`（日期），移除 `{time}`
- **工具函数**：`prompt.py::stamp_message(content, user_id)` — 支持 str 和 multimodal list 两种 content 格式，按用户时区格式化
- **注入点**：`chat.py`（Web）、`admin_bridge.py`（微信 Admin）、`consumer.py`（Consumer，3处）、`bridge.py`（微信 Consumer）
- **优势**：(1) 不依赖缓存刷新，每条消息实时精确 (2) 历史回溯时知道每条消息的发送时间 (3) agent 缓存恢复按天粒度，不浪费资源
- **注意**：consumer 端使用 `admin_id` 获取时区（跟随 admin 设置）

### 时区处理（2026-04-10 修复）
- 每个任务 JSON 中存储 `tz_offset_hours` 字段，记录创建时的用户时区偏移
- **cron 表达式按用户时区解释**：`_next_cron` 将 UTC now 转为用户本地时间 → 传给 croniter → 结果转回 UTC
- **once 类型必须带时区后缀**（如 `+08:00`）：`_compute_next_run` 用 `fromisoformat` 解析带 tz 的 ISO 字符串；无 tz 时按 `tz_offset_hours` 补充
- **工具层防御**：`_ensure_tz_suffix(iso_str, tz_offset_hours)` 在 `tools.py` 中自动为无时区的 ISO 字符串补上用户偏移后缀
- **Agent prompt 已引导**：CAPABILITY_PROMPTS 中 `scheduler` / `service_scheduler` 明确要求 once 类型带时区后缀，cron 按用户时区
- **interval 类型不受影响**：直接按秒数偏移，与时区无关
- `create_task` / `create_service_task`：若 data 中无 `tz_offset_hours`，自动从 `preferences.get_tz_offset(user_id)` 获取
- **`_compute_next_run` 不可用 `task.get("tz_offset_hours", 0)`**：缺字段的旧任务会误按 UTC；应使用 `_resolve_task_tz_offset(task)`（缺字段时回退 `get_tz_offset(user_id)`，与 `preferences` 默认 +8 一致）
- React **Scheduler** 页创建/更新任务时 body 须带 `tz_offset_hours: getTzOffset()`（与设置页一致），避免任务 JSON 长期缺字段

### 通用
- main.py startup 启动调度器，shutdown 优雅停止
- **运行记录步骤日志**：每条 run record 含 `steps[]`，每步有 `type/content/ts` + 扩展字段
  - Script 步骤类型：`start`→`stdout`→`stderr`→`exit`（或 `error`）
  - Agent 步骤类型：`start`→`docs_loaded`→`loop`→`tool_call`→`tool_result`→`ai_message`→`auto_approve`→`finish`（或 `error`）→`reply`
- **沙箱权限配置**：`task_config.permissions = {read_dirs: [...], write_dirs: [...]}`
  - 路径相对用户文件系统根目录（如 "docs", "scripts", "generated", "tasks", "data/output"）
  - 默认权限：`_DEFAULT_READ_DIRS = _DEFAULT_WRITE_DIRS = ["docs","scripts","generated","tasks"]`
- **脚本路径规则**：脚本通过子进程执行，cwd 为 `scripts/` 的真实路径
  - 脚本内不能用 `/docs/`、`/scripts/` 等虚拟路径（会变成容器绝对路径）
  - 必须用相对路径：`open("../docs/file.csv", "r")`、`open("output.txt", "w")`

## 定时任务 v2 大版本迭代（已实施 — 2026-05-08 完成 Phase A–E + 后端冒烟）

> 设计源文件：`定时任务大版本迭代.md`。核心命题：把 scheduler 从「用户工具」升级为「Agent 元能力」，让 Agent 通过定时任务自我延续 / 自我生长。
>
> **状态**：Phase A（基础设施）/ B（scheduler 重构）/ C（agent 工具）/ D（API）/ E（前端）全部落地，三轮端到端冒烟测试 + frontend `tsc -b` build 全过。仅剩 F（用户向文档 + 真实数据 e2e 验证）。
>
> **已修复的真实 bug（不要再踩）**：
> 1. `_compute_next_run`：`schedule_type='once'` 且 `schedule=''` 现在解读为「立即执行」（spawn 默认行为依赖此）。旧调用约定也被静默兼容。
> 2. `delete_task` / `delete_service_task` **必须**先 `list_descendants(include_root=True)` 拿到 victim id 列表，再 `_heap_upsert(...None)` 从堆 index 中驱逐 — 否则被删任务的 next_run_at 残留在堆里会导致幽灵 dispatch。
> 3. spawn 工具的 ContextVar **必须**在 `_execute_*_task` 的 `finally` 块里 `reset(token)` —— 否则 ContextVar 跨 task 串味，下一次执行的 spawn 会归属错误的 parent。
> 4. `check_chain_quota` 是 4 参签名 `(scope, uid, root_task_id, service_id=None)`；`QuotaResult` 字段叫 `current` / `remaining` / `reset_at`（不是 `used` / `reset_in_s`）。

### 实际落地的最终决策（覆盖原计划）

### 已确认的设计决策（all_in_one 一次性交付）
1. **存储模型 → 文件系统树**：
   - `users/{uid}/tasks/{root_id}/{child_id}/.../_meta.json`（含 runs/ 子目录存 JSONL）
   - service 任务同结构：`users/{uid}/services/{svc}/tasks/{root_id}/.../`
   - root_task_id 即路径第一段；任意 task_id 反查路径用 `scheduler_tree.task_path_for(task_id)`
   - 旧 `task_xxx.json` + `task_xxx.steps/` lazy 迁移到 `task_xxx/_meta.json` + `task_xxx/runs/`
2. **Spawn 权限 → 全场景**：
   - admin agent / consumer service agent / 「正在执行定时任务」上下文里的 agent 都能 spawn 子任务
   - 通过 `_current_task_var: ContextVar` 传递当前 task_id 给工具（在 `_run_agent_loop` 进出时 set/reset）
3. **Spawn 限制 → rate limit（可配）**：
   - 单 spawn_chain 每小时 spawn 上限，**默认 30，环境变量 `SCHED_SPAWN_RATE_PER_HOUR` 覆盖**
   - 实现位置：`app/services/spawn_limits.py`（按 root_id 聚合 chain 上所有任务的最近 1h spawn 次数）
4. **调度机制 → heap 驱动**：
   - 取代当前 `TaskScheduler` 30s 轮询
   - 最小堆 `[(next_run_unix, task_path)]` + `asyncio.Event` 唤醒
   - 任何 next_run_at 变化（create/update/spawn/delete/run-now）都要 `_wake_event.set()`
   - 自然支持 1s ~ 1h 全档精度，无需「分档轮询」
5. **记忆策略 → 4 层**：
   - L1 持久化：写 `_meta.json::runs[]` + 父任务的 `children_summary`（现有）
   - L2 短期注入：`scheduled_inject` 把当前 thread 最近 N 次 scheduled_task tool 对注入 LangGraph state（现有）
   - L3 子代摘要滚动：父 task 的 `_meta.json` 维护 `descendants_summary` 字段，限长 1500 字 LRU（**新增**）
   - L4 长期目录翻阅：memory subagent 加 `read_task_tree(root_id, depth)` 工具（**新增**）
6. **UI → 谱系图**：
   - 技术选型 **reactflow**（自带 zoom/pan + react-friendly）
   - Scheduler 页拆 3 视图 Tab：[列表]（现有保留 + 加 parent 列） / [谱系图 reactflow] / [时间轴 timeline]
7. **Spawn 默认继承**：
   - `spawn_child_task` 默认继承父任务的 `reply_to + permissions + capabilities + tz_offset_hours`
   - schedule + name 必须显式指定，其余字段可覆盖

### 影响文件清单（13 个，含 4 个新建）
- 后端核心：`app/services/scheduler.py`（大改）、`app/services/scheduler_tree.py`（新建）、`app/services/spawn_limits.py`（新建）
- 工具层：`app/services/tools.py`（加 `create_spawn_child_task_tool`）、`app/services/memory_tools.py`（加 `read_task_tree`）
- subagent：`app/services/subagents.py`（`SHARED_TOOL_NAMES` += `spawn_child_task`；`MEMORY_TOOL_NAMES` += `read_task_tree`）
- 注入层：`app/services/scheduled_inject.py`（chain-aware 选项）
- 路由：`app/routes/scheduler.py`（新增 `/tree/{root_id}`、`/{task_id}/children`、`/{task_id}/ancestors`）
- 前端：`frontend/src/pages/Scheduler/index.tsx`（重构）、`Scheduler/GraphView.tsx`（新建）、`Scheduler/TimelineView.tsx`（新建）、`types/index.ts`（加 parent_task_id / spawn_chain / children_count）
- 依赖：`frontend/package.json` 加 `reactflow`
- 迁移：`scripts/migrate_tasks_to_tree.py`（新建：lazy migration helper + 一次性 dry-run 工具）
- 文档：本文件（`.cursorrules`）+ `docs/DEVELOPER_GUIDE.md` §8 同步更新

### 关键技术约束
- **task_id 全局唯一前缀不变**：`task_*` / `stask_*`，但**不再用作文件名**，而是路径段
- **rate limit 检查时机**：spawn 工具调用 → check_chain_quota(scope, uid, root_id, service_id) → 不通过返回友好错误信息让 Agent 知道
- **ContextVar 必须在 `_execute_*_task` finally reset**：否则跨任务复用时会污染上下文
- **谱系图节点状态**：用 next_run_at + last_run_at + children_count 判定，节点颜色按 status（success/error/timeout/pending/idle）
- **L3 descendants_summary 写入时机**：子任务 finish 后向上回溯所有祖先（沿 spawn_chain），每个祖先的 `_meta.json` append 一行 `[ts] icon task_id(d=N): name — first-line snippet`，LRU 截断 **1500 字**

### v2 实施 Snapshot（2026-05-08）

**新建后端模块**：
- `app/services/scheduler_tree.py` — 文件树存储 / 路径反查 / lazy migrate / walk_tree
- `app/services/spawn_limits.py` — 内存限频（key = `(scope, uid, service_id, root_id)`，1h 滑窗，env: `SCHED_SPAWN_RATE_PER_HOUR`，默认 30）
- `scripts/migrate_tasks_to_tree.py` — 一次性批迁移工具（`--dry-run`、`--users`、`--no-services`、`--verify`）

**重写后端模块**：
- `app/services/scheduler.py`：
  - 新增 `TaskContext` dataclass + `_current_task_var: ContextVar`
  - 新增 `create_child_task(parent_ctx, data)` / `_new_task_meta_v2(...)` / `_propagate_descendant_summary(...)` / `_build_task_context_from_meta(...)`
  - **删除**老 `TaskScheduler` 30s 轮询循环，**替换**为 `HeapScheduler`（min-heap + `asyncio.Event` 唤醒 + 每 600s 全量 rescan 兜底；`TaskScheduler = HeapScheduler` 别名保持兼容）
  - 全部 `_load_task / _save_task / _load_service_task / _save_service_task / list_tasks / list_service_tasks / delete_*` 改走 `scheduler_tree`
  - `_execute_task / _execute_service_task` 现在 set/reset ContextVar、写堆、做 L3 propagation

**新建前端模块**：
- `frontend/src/pages/Scheduler/types.ts` — 共享类型（含 v2 spawn 字段 + `TaskTreeNode` + `ChainQuota`）
- `frontend/src/pages/Scheduler/GraphView.tsx` — reactflow + dagre TB 布局谱系图，节点点击切换 selected
- `frontend/src/pages/Scheduler/TimelineView.tsx` — 纯 SVG swim-lane，24h / 7d / 全部 三档窗口

**修改前端模块**：
- `frontend/src/pages/Scheduler/index.tsx` — 引入 types.ts、加 `viewMode` segmented、列表项按 `spawn_depth` 缩进 + `↳` 标记 + `+N` 子计数
- `frontend/src/services/api.ts` — 加 5 个 v2 helpers：`getSchedulerTaskTree` / `getSchedulerTaskChildren` / `getSchedulerTaskAncestors` / `getSchedulerChainQuota` / `migrateAdminTasksV1ToV2`
- `frontend/package.json` — 加 `reactflow` + `dagre` + `@types/dagre`

**新增工具/能力**：
- `tools.py::create_spawn_child_task_tool()` — 无 user_id 参数，从 ContextVar 读 `(scope, uid, service_id, root, depth, chain)`；context 缺失返回友好错误；rate-limit 不通过返回 `current/limit/window/reset_at`
- `tools.py::CAPABILITY_PROMPTS["spawn"]` — capability prompt（agent.py 和 consumer_agent.py 在 scheduler 上下文里挂载）
- `memory_tools.py::list_task_trees(scope)` + `read_task_tree(task_id, scope, max_depth)` — admin memory subagent 读谱系
- `subagents.py` — `SHARED_TOOL_NAMES += {spawn_child_task}`、`MEMORY_TOOL_NAMES += {list_task_trees, read_task_tree}`、`build_subagent_tools` 加分发、默认 `memory` subagent 工具列表加这两个

**新增 API 端点**（`app/routes/scheduler.py`，admin + service 镜像）：
- `GET  /scheduler/{task_id}/tree?max_depth=5` → `TaskTreeNode`
- `GET  /scheduler/{task_id}/children` → 直接子节点列表
- `GET  /scheduler/{task_id}/ancestors` → root → … → parent 列表
- `GET  /scheduler/quotas/{root_task_id}` → 当前 chain 配额（peek，不消耗）
- `POST /scheduler/admin/migrate {dry_run: bool}` → 当前用户的 v1 → v2 迁移触发
- service 版：把 `/scheduler/...` 换成 `/scheduler/services/{service_id}/...`

**冒烟脚本**（都在 `scripts/`）：
- `smoke_phase_b.py` — 6 项：root 创建 / 列表 / 子任务 / L3 / 删除子树 / heap dispatch
- `smoke_phase_c.py` — 4 项：context-less 报错 / context 内派生 / rate limit / memory 树读
- `smoke_phase_d.py` — 7 项：5 个新端点 + v2 字段在现有 detail 端点 + 404 行为

## Frontend (React + Vite)

### 启动
```bash
cd frontend
npm run dev        # Vite dev server on :3000, proxy to FastAPI :8000
npm run build      # Production build → dist/
npm run legacy     # Old Express server (fallback)
```

### 技术栈
- **React 19** + **TypeScript 5.7** + **Vite 6**
- **Ant Design 5** (dark theme via ConfigProvider, 中文 locale)
- **React Router 7** (BrowserRouter, history mode — 干净 URL)
- **marked** + **DOMPurify** + **highlight.js** (markdown rendering, XSS safe)
- State: React Context (auth), component-local state (pages)

### 开发规范
- 组件使用函数式 + hooks，不用 class
- API 调用统一通过 `src/services/api.ts`
- 类型定义在 `src/types/index.ts`
- 页面组件放 `src/pages/<PageName>/index.tsx`
- 共享组件放 `src/components/`
- 样式优先使用 antd 组件 + inline style，复杂布局用 CSS Modules

### Chat 页面增强架构
```
pages/Chat/
  index.tsx           — 主页面：对话列表 + 消息区域 + SSE streaming + 模型/能力选择 + Plan Mode
  types.ts            — StreamBlock 联合类型 (thinking/text/tool/subagent)
  markdown.ts         — marked + highlight.js (按需导入 17 种语言, 83KB vs 1MB) + <<FILE:path>> 媒体嵌入 (image/audio/video/pdf/html)
  chat.module.css     — CSS Modules：JellyfishBot 暗色主题（设计 tokens 挂在 `.chatContainer` 的 `--jf-*` 变量上；主色 #E89FD9、辅色 #8B7FD9、强调 #5FC9E6；侧栏宽 240px；代码字体 JetBrains Mono；勿删组件已引用的类名）
  useSmartScroll.ts   — 智能滚动 hook (用户上滚暂停吸底, 回底部恢复)
  components/
    ThinkingBlock.tsx  — 思考过程折叠/展开
    ToolIndicator.tsx  — 工具调用 (流式参数预览 + 结果 + 折叠)
    SubagentCard.tsx   — Subagent 执行卡片 (timeline 时间顺序渲染: text/tool/thinking 交替)
    StreamingMessage.tsx — 流式消息容器 (管理交替的 blocks)
    MessageBubble.tsx  — 历史消息气泡 (含 tool_calls 回放)
    ApprovalCard.tsx   — HITL 审批 (文件操作 diff / Plan 审批 + 编辑)；按钮/标题图标用 Phosphor（FileCode、ListChecks、Check、X、PencilSimple 等），不再用 `@ant-design/icons`
    ImageAttachment.tsx — 图片附件 (粘贴/拖拽/选择 + 缩略图)
    VoiceInput.tsx     — 语音输入 (MediaRecorder + 转写)；**toggle 模式**：单击按钮开始 → 再次单击停止并自动转写填入；录音中 Esc 取消（不发送）；不绑任何全局开始快捷键（旧版 Tab 按住模式因 keydown `!e.repeat` 未阻止重复事件导致焦点切换 bug，且 Tab 破坏无障碍焦点导航，已废弃）
pages/AdminServices/
  index.tsx           — Service 完整管理 (CRUD + API Key + 微信渠道 + 内联测试)
pages/Scheduler/
  index.tsx           — 定时任务管理 (Admin/Service + 运行记录 + 步骤日志)
pages/WeChat/
  index.tsx           — 微信接入 (QR 生成/轮询 + 状态 + 消息)
components/
  FilePanel.tsx       — 文件浏览面板 (多选/内拖拽/虚拟剪贴板/系统粘贴/批量 zip/发送到/Diff)
  FolderPicker.tsx    — 单选文件夹选择器（懒加载，禁用源/源后代节点；用于「发送到/复制到」）
  FilePreview.tsx     — 多类型文件查看器（按 fileKind 分发渲染，详见下文「文件预览面板」）
  modals/
    SystemPromptEditor.tsx — System Prompt 版本管理 + Diff
    BatchRunner.tsx        — Excel 批量运行 (配置→进度→结果)；**入口在设置 → 通用 Tab 内嵌**，非独立路由
    SubagentManager.tsx    — Subagent CRUD + 工具/模型配置
    UserProfileEditor.tsx  — 用户画像 (投资偏好/自定义)
    SoulSettings.tsx       — Memory & Soul 高级设置 (Subagent 写入/文件系统暴露)
```

### Chat 组件视觉（JellyfishBot）
- 助手气泡头像：`public/media_resources/jellyfishlogo.png`，在 `MessageBubble` / `StreamingMessage` 中用 `<img>`（用户侧仍为字母 **U**）。
- 子组件图标：`@phosphor-icons/react`（如 ThinkingBlock=Brain + 流式时头部三点弹跳；ToolIndicator=Wrench、CircleNotch 旋转、CheckCircle；SubagentCard=Robot、CaretDown/Right、子工具行 Wrench；历史 tool 回放与流式一致）。
- 品牌色可参考：Primary `#E89FD9`、Secondary `#8B7FD9`、Accent `#5FC9E6`、Warning `#FFB86C`、Error `#FF6B9D`；Subagent 状态文案可用内联 `color` 与之一致。
- 若需 keyframes（弹跳、旋转）且不改 `chat.module.css`，可在组件内用局部 `<style>` 块，类名加 `jellyfish-` 前缀避免污染。

### Service 自助化（v2.x）
- **专属链接附带 Key**（`/s/{id}?key=sk-svc-xxx`）
  - 后端：`app/routes/consumer_ui.py` 注入模板变量；`service-chat.html` 启动时 `URLSearchParams` 读 `key` → 写 localStorage → `history.replaceState` 立即清掉 query（避免 referer/历史曝光，但仍残留在浏览器请求日志/网关访问日志中，**等同分享 Key**）
  - 前端 admin：Key Modal 生成成功后**额外**展示带 key 的完整链接 + 警告文案；只能在生成那一刻拿到（已存的 key 是哈希，无法回溯）
- **欢迎语 + 快速问题**（`welcome_message: str`、`quick_questions: List[str]`）
  - Schema：`app/schemas/service.py` `CreateServiceRequest` / `UpdateServiceRequest` 新增字段；持久化层用 `**data` 透传，无需改 `published.py`
  - 后端模板注入：`_safe_json_for_inline_script` 把 `</` 转义为 `<\/` 防 script breakout；占位符 `{{WELCOME_MESSAGE_JSON}}` `{{QUICK_QUESTIONS_JSON}}` 直接 inline JSON
  - 前端 chat 页：ChatGPT 风格首屏（大欢迎语+渐变 + chips 横排自动 wrap），发送第一条消息（含 chip 点击）后自动隐藏；为空字段则降级回旧的 empty-state
  - 前端 admin：Modal 中独立模块「聊天页定制」，欢迎语 = `TextArea(maxLength=300, showCount)`，快速问题 = `Form.List`（动态增删，单条 80 字限制）
- **可访问文件/脚本图形选择器**（替换原"逗号分隔字符串"输入）
  - 组件：`frontend/src/components/FileTreePicker.tsx` —— antd `Tree` checkable + `loadData` 懒加载（按需调 `api.listFiles`）+ 「全部 (*)」 Switch 快捷开关 + 缺失路径警告
  - 文件夹勾选 = 整个目录递归允许（key 以 `/` 结尾）；文件勾选 = 单文件
  - **根目录限定**：`allowed_docs` 只展示 `/docs`；`allowed_scripts` 只展示 `/scripts`（与项目文件系统约定一致）
  - Trigger 行：`PickerTrigger`（紧凑展示已选摘要，"全部" 用绿色 Tag，多选超 3 个截断为"X, Y, Z 等 N 项"）
  - Form 集成：用 `PickerField` 包装一层适配 antd Form 的 `value/onChange`，弹窗确定后 `form.setFieldValue` 写回
- **schema 字段保留逗号字符串兼容**：后端不再强制；前端老数据若是 `["doc1", "doc2"]` 列表自然兼容；空 `allowed_docs` 自动回落 `["*"]`，空 `allowed_scripts` 保持空数组（语义=禁止脚本）

### 文件预览面板（FilePreview）
- **kind 分类**：`utils/fileKind.ts` 按扩展名 → `image|audio|video|pdf|markdown|html|csv|json|text|binary`
- **openFile 优化** (`stores/fileWorkspaceContext.tsx`)：媒体/binary 跳过 `api.readFile`，直接 `setEditingFile(path)` 避免拉大文件文本
- **渲染策略**（`components/FilePreview.tsx`）：
  - 媒体（image/audio/video/pdf）：原生 `<img>/<audio>/<video>` 或 `<iframe>` 走 `mediaUrl(path)`，工具栏自动隐藏「保存」按钮
  - markdown：默认预览（复用 `pages/Chat/markdown.ts` 的 `renderMarkdown`，含 hljs/<<FILE:>> 处理）；样式用全局 `.jf-file-md-preview`（见 `styles/global.css`）
  - html：默认预览，`<iframe sandbox="allow-scripts">`（**无 same-origin** —— 允许 Plotly/ECharts 交互但禁止访问父页/cookies/localStorage）
  - csv/tsv：默认预览，`utils/csvParse.ts` 状态机解析（支持引号转义/CRLF/引号内换行），用 antd `Table` 渲染（最多 2000 行，超过显示「仅显示前 N 行」提示）；自动识别 `,`/`\t`
  - json/jsonl/ndjson：默认预览，`JSON.parse` + `hljs.highlight(..., {language:'json'})` 高亮；解析失败降级为源码 + 错误提示
  - text/code（py/js/ts/yaml/sh/...）：保持现状 textarea，无高亮编辑（极简设计）
  - binary：`Empty` 占位 + 下载按钮
- **工具栏切换**：toggle 类（md/html/csv/json）头部加 antd `Segmented`「预览/源码」；切换文件自动重置为预览
- **下载按钮**：所有 kind 工具栏统一新增；保存按钮仅在 `isEditableKind` 显示

### Markdown 标题锚点 + 右侧 TOC + 路径 clickable（聊天 / 文件预览共享）
> 所有改动集中在 `pages/Chat/markdown.ts`、`components/FilePreview.tsx`、`styles/global.css`，因为聊天气泡和 markdown 文件预览复用同一套 `renderMarkdown`。

- **标题锚点（`pages/Chat/markdown.ts` 自定义 `headingRenderer`）**
  - 用 `marked.use({ renderer: { heading: headingRenderer } })` 替换默认 h 渲染
  - `slugifyHeading(text)` 把标题转 URL 友好 id；**保留 CJK 字符**（不靠纯 `[a-z0-9-]` 过滤），重复标题加 `-2 / -3` 后缀
  - **每次 `renderMarkdown` 调用前先 `_resetHeadingIds()`** —— 否则模块级 `_headingIdSeen` Set 会跨多条聊天消息累积，造成同名标题不停退化为 `heading-2/3/4...`，破坏锚点稳定性
  - `this.parser.parseInline(tokens)`（`marked` v15 把 `this` 绑到 parser）保证标题里的 em/strong/code/link 仍正常渲染
  - 末尾追加 `<a class="jf-heading-anchor" href="#id">#</a>`，hover 标题才显形（CSS 在 `styles/global.css`）
  - DOMPurify 白名单默认放行 `id/class/href`，无需扩 SANITIZE_OPTS

- **TOC（`components/FilePreview.tsx > MarkdownPreview`）**
  - `extractToc(html)` 用 `DOMParser` 解析 h1-h3 → `[{id, depth, text}]`；**仅在 markdown 文件预览展示**，聊天气泡不渲染 TOC
  - 滚动高亮：`IntersectionObserver` 监听容器内所有 `[id]^=` 标题；`rootMargin: '-20% 0px -70% 0px'` 让"接近视口顶部"的标题成为 active
  - UI：右侧 `<aside>` 浮层，可点击 `UnorderedListOutlined` 折叠/展开；点击条目 `scrollIntoView({ behavior: 'smooth', block: 'start' })`
  - 折叠后保留圆形按钮入口；TOC 仅在 `toc.length >= 2` 时显示（避免单标题文件无意义 TOC）

- **路径 clickable（聊天里贴出来的工作区路径自动可点）**
  - `markdown.ts > postProcessInlinePaths(html)` 用 `INLINE_PATH_RE` 把 `<code>/docs/x.md</code>` 这类**含 `/` 且带扩展名**的内联代码替换成 `<button class="jf-file-link jf-file-link-inline" data-jf-file="..."><span class="jf-file-link-icon">📄</span><span class="jf-file-link-name">...</span></button>`，与已有 `[[FILE:..]]` 块级链接（`renderFileLinkButton`）的样式族保持一致，行内显示更紧凑
  - 触发条件刻意保守：必须含 `/`、不含空格/引号/反斜杠/冒号、扩展名 1-8 位字母数字 —— 避免误转 URL、Windows 盘符、命令行片段
  - 点击转跳：`stores/fileWorkspaceContext.tsx` 在 `document` 上挂全局点击委托，命中 `[data-jf-file]` 即调 `revealInBrowser(path)` 打开文件面板并定位 —— 因此 chat 气泡 / FilePreview / 任何把 markdown 渲染到 DOM 的位置都自动可用
  - DOMPurify 必须放行 `data-jf-file` 属性（`SANITIZE_OPTS.ADD_ATTR` 已含）

- **CSS（`styles/global.css`）**
  - `.jf-heading` 提供锚点定位的 `scroll-margin-top`
  - `.jf-heading-anchor` 默认 `opacity: 0`，`.jf-heading:hover .jf-heading-anchor` / `:focus-visible` 才显形；hover 变品牌色
  - `.jf-file-link-inline` 行内代码风格 + 主题色边框 hover 高亮，明确"可点"语义

### 文件面板批量操作 / 拖拽 / 剪贴板 / Zip 下载 / 发送到（FilePanel + 后端 zip & copy）
> 改动文件：`components/FilePanel.tsx`、`components/FolderPicker.tsx`（新增）、`services/api.ts`、`app/routes/files.py`、`app/storage/{base,local,s3}.py`、`app/schemas/requests.py`

#### 后端
- **`StorageService` 新增两个抽象方法**（`app/storage/base.py`）
  - `copy(user_id, source, destination) -> str`：单文件用 `shutil.copy2`，目录用 `shutil.copytree`；S3 用 `_copy_or_move(... delete_source=False)` 复用 move 的递归逻辑（`list_objects_v2` 分页 + `copy_object`）
  - `walk_files(user_id, path) -> Generator[(rel_path, bytes), None, None]`：local 用 `os.walk` 流式 yield；S3 用 paginator + `get_object` —— **不 buffer 整个目录**，配合 `BytesIO` zip 实现近常量内存
  - **拒绝把目录复制到自身/后代下**（`_is_descendant` 检查，与 move 行为一致）
- **`POST /api/files/copy`**（`CopyFileRequest{source, destination}`）
  - 复用 `safe_join` + `path_security` 防遍历；和 move 一样捕获 `FileExistsError` → 409、`ValueError` → 400
- **流式 Zip 下载**（`app/routes/files.py`）
  - `_iter_zip_stream(user_id, paths)`：把所有 path 的 `walk_files()` 输出写入 `zipfile.ZipFile(BytesIO, 'w', ZIP_DEFLATED)`，**每写一个文件就 yield 一次 BytesIO 当前缓冲并 truncate**，实现"逐文件刷出去"的伪流式（pure-Python `zipfile` 不支持真正的 streaming writer，但够用）
  - `GET /api/files/zip?paths=[json]`：标准 Bearer Auth，前端 fetch 拿 blob 触发下载
  - `GET /api/files/zip-token?paths=...&token=...`：**给 `<a download>` 直接用的 token-in-query 变体**——浏览器原生 `<a>` 下载无法附带 Authorization 头，所以专门走 query token；token 仍走 `verify_token` 校验，行为等同 Bearer
  - `_zip_filename_from_paths(paths)`：单根用根名，多根用 `selection-{N}.zip` 命名

#### 前端（FilePanel 重写要点）
- **多选状态机**（Finder 风格）
  - `selected: Set<string>`、`lastSelected: string | null`
  - 单击：清空只选当前；`Ctrl/Cmd+click`：toggle 单项；`Shift+click`：基于 `displayedFiles` 顺序的连续区间选择
  - `Ctrl+A` 全选当前目录、`Esc` 清空选择
  - 顶部条件渲染**批量工具栏**（仅 `selected.size > 0` 出现）：复制/剪切/发送到/打包下载/删除

- **虚拟剪贴板（与系统剪贴板互不冲突）**
  - `clipboard: { paths: string[]; mode: 'copy' | 'cut'; sourceDir: string } | null`
  - `Ctrl+C` / `Ctrl+X` 仅在 `selected.size > 0` 且**焦点不在输入框**时触发（用 `e.target.tagName` 排除）
  - `Ctrl+V` → `pasteHere()`：cut 模式用 `moveFile` + 清空 clipboard；copy 模式用 `copyFile` + 保留 clipboard 支持多次粘贴
  - 顶部还有"剪贴板提示条"：显示来源目录 + 项数 + 模式，提供 `粘贴到这里` / `清空` 按钮，让虚拟剪贴板的状态可视化

- **面板内拖拽（移动）**
  - 自定义 MIME `application/x-jf-file-paths`（JSON 字符串）—— 用于和系统拖入文件区分；外部拖入文件继续走 `dataTransfer.files` → 上传逻辑
  - `draggable` 仅在非重命名状态启用；拖动多选项目时一并带上整个 selection
  - drop 目标：**目录行 + 面包屑某段 + 「返回上一级」按钮**，拖拽悬停加 `.jf-drop-hover` 视觉反馈
  - 落在自身/源目录直接 no-op；冲突走 `performBulkMoveOrCopy` 的同名询问（`overwrite | rename | skip | abort`）

- **系统剪贴板粘贴**
  - 给面板根 div 绑 `onPaste`：从 `e.clipboardData.items` 读图片 → 自动命名 `clip-YYYYMMDD-HHMMSS.png` 上传到当前目录；普通文件走标准 `uploadFiles`

- **「发送到 / 复制到」（右键菜单 + 工具栏）**
  - 弹出 `FolderPicker`（新组件，`components/FolderPicker.tsx`）：antd `Tree`、`loadData` 懒加载、过滤掉非 directory；`disabledPaths` 把源目录 + 后代标灰防止自坑
  - 选中后调 `performBulkMoveOrCopy(targetDir, mode)`，统一走冲突解决逻辑

- **批量 Zip 下载**
  - 单文件直接 `mediaUrl(path)`；多个或包含目录走 `zipDownloadUrl(paths)`（`api.ts` 的工具函数，拼接 `/files/zip-token?paths=...&token=...`），用动态 `<a download>` 触发
  - 不需要前端 buffer，整个流程 backend StreamingResponse 推过去

- **右键菜单（`buildContextMenu`）**
  - 单项：打开 / 重命名 / 下载 / 复制 / 剪切 / 发送到 / 复制到 / 删除
  - 多选：复制 / 剪切 / 发送到 / 复制到 / 打包下载 / 批量删除
  - 在文件夹空白处：粘贴（含数量提示）/ 上传 / 新建文件 / 新建文件夹

- **冲突解决（`performBulkMoveOrCopy`）**
  - 遍历 selection 调 move/copy；命中 409 时弹 antd `Modal.confirm` 让用户选 `覆盖 / 重命名（追加 (1)/(2)）/ 跳过 / 全部中止`
  - "记住本次"开关把决策应用到剩余冲突，避免连点

- **API 客户端（`services/api.ts`）**
  - `copyFile(source, destination)` → `POST /files/copy`
  - `zipDownloadUrl(paths)` → 拼 `zip-token` 完整 URL（含 token），交给 `<a>` 用
  - 走原 `getToken()`，复用现有鉴权

#### 关键边界 / 已知约束
- 拖拽**仅识别 effectAllowed=move**；不支持跨用户/跨 service 拖拽（路径 prefix 受 `path_security` 限制）
- Zip 流式实质是"逐文件 chunk"，不是 zip64 真流式；超大目录（如几 GB）建议分批选；S3 模式 `walk_files` 已用 paginator，不会一次性 list 完
- 虚拟剪贴板**仅在 SPA 内**有效，关掉浏览器即失效；和系统剪贴板隔离避免混淆
- 复制/移动时 `source == destination` 或 src 是 dst 祖先，后端会抛 `ValueError` → 400 文案明确返回
- "发送到"目标为根 `/` 时也支持（拼接为 `/{name}`）

### 媒体文件嵌入（<<FILE:path>> 标签）
- **后端**: `tools.py` 的 `generate_image`/`generate_speech`/`generate_video` 返回 `<<FILE:/generated/images/xxx.png>>` 格式
- **前端 markdown.ts**: 三阶段渲染管线
  1. `preProcessMediaTags`: 正则匹配 `<<FILE:path>>` → 替换为带认证 URL 的媒体 HTML（image/audio/video/pdf/html）
  2. `marked.parse`: 标准 Markdown → HTML
  3. `postProcessMediaSrc`: 修正 `![](path)` 生成的 `<img src>` → 认证 media URL
- **DOMPurify**: 白名单含 `audio`, `video`, `source`, `iframe` + `controls`, `preload`, `onclick` 等属性
- **CSS**: `chat.module.css` 末尾 `:global(.jf-media*)` 系列样式（因通过 `dangerouslySetInnerHTML` 注入，不走 CSS Modules 编译）
- **认证 URL**: `mediaUrl(path)` → `/api/files/media?path=...&token=...`
- **Consumer chat** (`service-chat.html`): 旧版，暂未移植此功能

### 流式性能优化
- SSE 回调通过 `useRef` 直接修改 blocks 数组 (避免每 token 触发 setState)
- `requestAnimationFrame` 节流刷新 (~60fps), 批量更新到 React state
- `StreamingMessage` 使用 `React.memo` 避免不必要的 parent re-render

### 错误边界（ErrorBoundary）
- 组件：`frontend/src/components/ErrorBoundary.tsx`（**全仓唯一的 class 组件**，React 19 仍要求 class 形式的错误边界，无 hook 等价物，刻意豁免 "avoid classes" 规则）
- 三层部署（`router/index.tsx`）：
  - `scope="app-layout"` 包裹 `<AppLayout/>` — 兜底整个受保护区域
  - `scope="chat"` 包裹 `<ChatPage/>` — 聊天页崩溃不影响侧栏导航
  - `scope="settings"` 包裹 `<SettingsLayout/>` — 设置页崩溃不影响聊天
- 交互：友好提示 + **可展开错误详情** + **复制错误信息**按钮（Clipboard API，含 scope/时间/URL/UA/stack/componentStack），小团队 admin 便于把错误贴给维护者
- 操作按钮：刷新页面 / 回到首页 / 复制错误信息
- 样式用 Ant Design `Result` + `Collapse`，错误详情背景用 `--jf-bg-deep` 暗色主题变量保持一致性

### 迁移状态
- ✅ 登录/注册 (Login.tsx — 品牌分屏布局, 水母呼吸动画, 渐变按钮)
- ✅ 对话/聊天 (Chat/index.tsx — 完整流式: Thinking/Tool/Subagent blocks)
- ✅ Markdown + 代码高亮 (highlight.js 按需加载, JetBrains Mono 代码字体)
- ✅ 智能滚动 (useSmartScroll)
- ✅ 模型选择 + 能力开关 + Plan Mode
- ✅ HITL 审批卡片 (ApprovalCard.tsx — 文件操作 diff + Plan 审批 + 编辑)
- ✅ 图片附件 (ImageAttachment.tsx — 粘贴/拖拽/文件选择 + 缩略图)
- ✅ 语音输入 (VoiceInput.tsx — MediaRecorder + 转写)
- ✅ Service 管理 (AdminServices/index.tsx — 完整 CRUD + API Key + 微信渠道 + 内联测试)
- ✅ 定时任务 (Scheduler/index.tsx — Admin/Service 任务 + 运行记录步骤日志)
- ✅ 微信接入 (WeChat/index.tsx — QR 生成/轮询 + 连接状态 + 消息展示)
- ✅ 文件面板 (FilePanel.tsx — 文件浏览/编辑/上传/删除/重命名/移动/Diff)
- ✅ System Prompt 编辑器 (SystemPromptEditor.tsx — 版本历史 + Diff + 回滚)
- ✅ 批量运行 (GeneralPage 内嵌 BatchRunner.tsx — Excel 上传 + 进度 + 结果下载；`/settings/batch` 重定向到 `/settings/general`)
- ✅ Subagent 管理 (SubagentManager.tsx — CRUD + 工具/模型配置)
- ✅ 用户画像设置 (UserProfileEditor.tsx — 投资画像 + 自定义备注)
- 🔲 Voice Agent Mode
- 🔲 HumanChat Mode

### 设计系统 (Design System)
- **图标库**: `@phosphor-icons/react` (全站统一, 替代 @ant-design/icons)
- **圆角**: sm=4px / md=8px / lg=12px / bubble=16px
- **字体**: 正文 Segoe UI / 代码 JetBrains Mono (Google Fonts CDN)
- **Logo**: `/media_resources/jellyfishlogo.png` (像素水母)
- **theme.ts**: 集中导出 brandColors 等 JS 常量（仅作后备参考），主题色以 CSS 变量为准
- **chat.module.css**: CSS变量 `--jf-*` 前缀, 所有 Chat 组件 scoped 样式

### 多主题系统 (Theming)
- **核心文件**：`src/styles/themes.css`（CSS 变量定义）、`src/stores/themeContext.tsx`（状态管理）
- **原则**：所有颜色通过 `var(--jf-*)` 引用，硬编码仅在 `themes.css` 和 `themeContext.tsx`（Antd ThemeConfig）中
- **主题切换**：`[data-theme]` 属性在 `<html>` 上，由 ThemeProvider 控制
- **持久化**：localStorage `jf-theme`
- **已有主题**：`dark`（默认，暖粉紫深色）、`cyber-ocean`（青蓝浅色）、`terminal`（磷绿 CRT 终端风格）
- **terminal 主题特殊规则**：
  - 全局 monospace 字体（覆盖 body + Antd 组件）
  - `border-radius: 0`（所有 `--jf-radius-*` 变量 + `:root` 覆盖共享 token）
  - 磷光文字发光 `text-shadow: var(--jf-glow)` / `var(--jf-glow-strong)`
  - CRT 扫描线覆盖层（`#root::after` repeating-linear-gradient + mix-blend-mode）
  - 按钮：无圆角 + 大写 + hover 反转（绿底黑字）
  - 标题：`text-transform: uppercase` + `letter-spacing: 0.08em`
  - 输入框：无圆角 + 绿色聚焦发光
  - 选择色：绿底黑字
  - 配色：primary `#33ff00`（磷绿）、secondary `#ffb000`（琥珀）、error `#ff3333`、bg `#0a0a0a`
- **添加新主题**：在 `themes.css` 中复制一个 `[data-theme]` 块并调整值，在 `themeContext.tsx` 中添加 THEMES 项和 Antd 配置
- **CSS 变量命名**：
  - 品牌色：`--jf-primary/secondary/accent/highlight/legacy`
  - RGB 三元组：`--jf-primary-rgb` 等（用于 `rgba(var(--jf-primary-rgb), 0.12)` 写法）
  - 渐变：`--jf-gradient-from/to`、`--jf-user-bubble-bg/shadow`、`--jf-bot-avatar-bg/user-avatar-bg`
  - 背景：`--jf-bg-deep/panel/raised/code/inset`
  - 文字：`--jf-text/text-muted/text-dim/text-quaternary/text-on-primary/text-on-gradient`
  - 边框：`--jf-border/border-rgb/border-strong`
  - 语义色：`--jf-success/warning/error/info`（含 `-rgb` 后缀版本）
  - 阴影：`--jf-shadow-float/hover/brand`
  - Diff：`--jf-diff-add-bg/add-text/del-bg/del-text/eq-text`
  - Antd：`--jf-menu-selected-bg/menu-hover-bg/select-option-bg`
- **UI 入口**：侧栏底部 Sun/Moon/Terminal 三态循环切换（AppLayout.tsx）+ Settings 页详细选择
- **Antd 适配**：ThemeProvider 根据 themeName 返回对应 ThemeConfig（含 `algorithm: darkAlgorithm | defaultAlgorithm`），App.tsx 动态传入 ConfigProvider

### Agent 执行控制
- **Stop 按钮**: 流式输出时发送按钮变为红色 Stop 按钮 (Phosphor `Stop`)
- **后端取消**: `POST /api/chat/stop` 设置 `asyncio.Event` 取消标志, `_stream_agent` 每次迭代检查
- **SSE 断流处理**: `handleSSEStream` 中网络错误正确调用 `onError`, 不再静默卡死
- **错误时保存**: 后端异常时将已生成的部分回复 + 错误信息保存到对话历史
- **连接中断保存**: `_stream_agent` / `_stream_consumer` / `_stream_openai_compat` 的 `finally` 块检测未保存的部分回复，追加 "⚠️ [连接中断 — 已保存已生成内容]" 后持久化（`_saved` 标志位防止重复保存）。`_stream_consumer.finally` 的保存条件从「仅 `full_response`」放宽为「`full_response` 或 blocks 里有任意 content/args/result」，确保纯工具调用阶段被中断也能持久化、刷新后可见。
- **超长 streaming 中断后内容消失（service 页修复，2026-06-04）**: 根因在前端 `service-chat/streamHandler.ts`——实时流式 blocks 只通过 `stream.isStreaming && <StreamingMessage blocks={stream.blocks}/>` 渲染，且**只有成功结束才调 `onDone` 把 blocks 固化进 `messages`**；网络/超时中断走 catch 的网络错误分支时只 push 一条错误文本、**从不调 `onDone`**，而 `finally` 把 `isStreaming` 置 false → 实时消息瞬间从 DOM 消失、blocks 也未固化 → 用户看到「内容凭空消失」。修复两处：(1) `streamHandler.ts` catch 网络错误分支里把已生成内容（追加「⚠️ 连接中断，已保留以上内容。」提示）走 `onDone` 同路径固化进 `messages` 后再 `onError`；AbortError（用户切换/新建会话）仍不提交。(2) `ServiceChatApp.tsx` 渲染条件从 `stream.isStreaming` 改为 `stream.isStreaming || stream.blocks.length > 0`（防御性：未提交的 blocks 始终渲染，`onDone` 内会 `reset()` 清空避免重复）。admin 页本就靠 `handleStreamError` 600ms 后从后端 reload 恢复 + 渲染条件含 `streamBlocks.length>0`，无此问题。
- **活跃流追踪**: `_active_streams` 字典追踪当前正在 streaming 的 `{thread_id → {user_id, conv_id}}`；`GET /api/chat/streaming-status` 返回当前用户活跃 streaming 对话列表
- **前端断连恢复**: Chat 页面加载时调用 `checkServerStreaming()`，如果当前对话仍在后台 streaming，显示黄色横幅（"上一轮对话仍在后台运行中"），提供「终止并保存」和「刷新状态」按钮；发消息前检查 `serverStreaming` 防止写入冲突

### Consumer 页面（不在 SPA 路由中）
- `service-chat.html` / `wechat-scan.html` 仍由 FastAPI 直接返回（占位符替换）
- 位于 `frontend/public/`，Vite 直出静态文件

## Tauri 桌面启动器（tauri-launcher/）

### 架构
- **Tauri v2** (Rust + WebView) 封装为原生桌面应用（.dmg / .exe）
- 内嵌 Python 3.12 + Node.js 20 运行时 + 后端代码 + 前端构建产物
- 用户双击即启动，无需命令行

### 关键配置（踩坑记录）
- **`withGlobalTauri: true`** — 必须！否则 `window.__TAURI__` 为 undefined，JS 走 mock 函数
- **`server.js` 使用 ESM 语法** — `package.json` 有 `"type": "module"`，不能用 `require()`
- **Express 5 通配符** — 用 `'/{*path}'` 代替 `'*'`
- **路径必须绝对** — `launcher.py` 的 `_resolve_python/node()` 必须返回 `os.path.abspath()`
- **路径必须剥离 `\\?\` 扩展前缀（Windows 关键坑，2026-04-20）**
  - **症状**：Windows .exe 启动后 uvicorn 报 `OSError: Cannot load native module 'Crypto.Util._cpuid_c': Not found '_cpuid_c.cp37-win_amd64.pyd'`，但 `.pyd` 文件物理上明明在 `D:\JellyfishBot\python\Lib\site-packages\Crypto\Util\` 下
  - **根因**：Tauri 的 `app.path().resource_dir()` 在 Windows 上返回 `\\?\D:\JellyfishBot\` 形式的扩展长路径前缀；该前缀通过 `Command::new(python_exe)` 和 `JELLYFISH_PYTHON` 环境变量传给 Python 子进程，污染 `sys.executable` 和所有 site-packages 模块的 `__file__`；`pycryptodome` 的 `os.path.isfile()` 在 `\\?\` 前缀路径下查不到 sibling 的 `.pyd` 文件，整个 wechat 模块加载失败
  - **复现**：直接运行 `\\?\D:\JellyfishBot\python\python.exe -c "from Crypto.Cipher import AES"` 立即报错；plain 路径 `D:\JellyfishBot\python\python.exe -c "..."` 正常
  - **诊断脚本**：`tauri-launcher/scripts/verify_extended_path_bug.ps1`（5 个 Test 对比 plain vs `\\?\`）
  - **修复**：`src-tauri/src/lib.rs` 加 `strip_win_extended_prefix()` helper，在 `resolve_project_dir` / `find_bundled_python` / `find_bundled_node` 三处强制剥离前缀；`launcher.py` 加 `_strip_extended_prefix()` 双重防御 `JELLYFISH_PYTHON/NODE` 和 `SCRIPT_DIR`
  - **同样的坑也会影响 numpy/scipy/matplotlib 等其他依赖原生扩展的包**（虽然他们多数模块不像 pycryptodome 用 `os.path.isfile` 显式检查），所以剥离动作放在源头一次解决
  - **mac 完全无此问题**（`/Applications/JellyfishBot.app/Contents/Resources/` 路径正常无前缀）
- **macOS 签名** — 修改 Resources 后需 `codesign --force --deep --sign - <app_path>`
- **`express` + `http-proxy-middleware`** 必须在 `frontend/package.json` 的 `dependencies` 中

### 超管功能（Tauri 命令）
- `list_registration_keys` — 读 `config/registration_keys.json`
- `generate_registration_keys(count)` — 生成 `JFBOT-XXXX-XXXX-XXXX` 格式注册码
- `delete_registration_key(key)` — 仅删未使用的码
- `list_admin_users` — 读 `users/users.json`（脱敏：不含 token/password_hash）
- `reset_admin_password(userId)` — 生成临时密码（sha256 哈希，兼容 Python 后端）
- `delete_admin_user(userId)` — 删 JSON 条目 + 用户目录
- `get_admin_stats` — 统计：总用户/7 天活跃/已配 Key/可用注册码
- `set_admin_disabled(userId, disabled)` — 封禁/解禁（2026-06-30，详见下方 P2 段）
- `rename_admin_user(userId, newUsername)` — 改显示用户名（查重名，User ID 不变）
- `get_admin_storage(userId)` — 用户目录磁盘占用（总字节 + 顶层子目录/文件 breakdown，降序）
- `get_admin_key_status(userId)` — 已配供应商列表（读 api_keys.json 看密文非空，不解密）
- `get_system_health()` — 系统健康只读快照（运行态/checkpoints 大小/logs 列表，2026-06-30 P4）
- `get_disk_usage()` — project_dir 顶层磁盘占用（按需，walk）
- `tail_log(name, lines)` — 只读 logs/ 下指定日志末尾 N 行（路径校验，不删不清）

### 关于 / 工具命令（2026-04-20 新增）
- `open_project_dir` — 调 `open::that(project_dir)`
- `open_users_dir` — `users/` 不存在时先 `create_dir_all` 再打开（首次安装无 users/ 也能正常弹窗）
- `open_logs_dir` — `logs/` 同上 lazy 创建
- `open_release_page` — 浏览器跳转 `https://github.com/LiUshin/JellyfishBot/releases/latest`（公开镜像仓库；私有 dev 仓库 `LiUshin/semi-deep-agent` 不对外暴露）
- `get_app_version` — 返回 `env!("CARGO_PKG_VERSION")`，**和 Cargo.toml 一致，无需另存版本字符串**

### Launcher Bug 修复记录（2026-04-21）
- **Toast 反复弹窗**：`updateRunUI` 之前每次 polling（2s）只要 ready 就 `showToast('JellyfishBot 服务已就绪')`，导致用户被骚扰。修复：用闭包外的 `lastReadyState` 标志，仅在 false→true 转变瞬间弹一次；停止时复位
- **关闭窗口未杀子进程（核心坑）**：`launcher.py` 通过 `Popen` 启动 uvicorn + express 两个孙子进程；Tauri 默认窗口 X 关闭只触发 process exit，不会杀子进程，且 `Child::kill()` 在 Windows 上**只杀直接子**，孙子（实际占着 8000/3000 端口的进程）变成孤儿，下次启动报「端口被占」
  - 修复 (a)：`stop_jellyfish` 抽 `kill_process_tree(child)` helper —— Windows 用 `taskkill /T /F /PID <pid>` 递归杀整棵树；Unix 用 `SIGTERM` + 800ms 宽限期 + `SIGKILL` 兜底
  - 修复 (b)：Tauri Builder 加 `.on_window_event` 监听 `WindowEvent::CloseRequested`，关闭瞬间同步调 `shutdown_jellyfish(&state)`；不 `prevent close`，杀完直接让窗口关
  - 修复 (c)：双保险 —— `App::run(|app, event|)` 监听 `RunEvent::Exit`，即便 CloseRequested 没触发（系统强 Exit / 任务管理器结束）也再清一次。`shutdown_jellyfish(&AppState)` 是 `stop_jellyfish` 命令和窗口事件共享的 helper，避免逻辑重复
  - **不要回退到只用 `child.kill()`** —— Windows 实测会留 uvicorn.exe + node.exe 残留进程
- **Logo 不显示**：`dist/index.html` 中 `<img class="logo-img">` 用了 700 字节的 base64 占位（实际是个无效图片），需要把 `frontend/public/media_resources/jellyfishlogo.png` (324KB) 拷贝到 `tauri-launcher/dist/jellyfishlogo.png`，HTML 改用 `./jellyfishlogo.png` 相对路径（dist 目录会被 Tauri `frontendDist` 整体打包，无需额外配置 resources）。注意 logo 容器有 `image-rendering: pixelated` 样式 —— 真实水母图也保持像素风视觉，无需调整
- **Release URL 用户名拼写**：`open_release_page` 之前是占位 `jellyfishbot/jellyfishbot`，正确公开仓库是 `LiUshin/JellyfishBot`（注意大小写：U 大写、JellyfishBot 驼峰）。git remote 里 `public` 才是发布用，`origin` (`semi-deep-agent`) 是私有 dev

### Windows 本机 dev 环境要求（重要！踩坑记录）
- `npx tauri dev` / `npx tauri build` 都需要：
  1. **Rust toolchain**（cargo 在 `C:\Users\{user}\.cargo\bin\` —— 默认不在 PATH，PowerShell 里要手动 `$env:Path = "C:\Users\X\.cargo\bin;$env:Path"`）
  2. **MSVC linker（link.exe）** —— VS 2022 Build Tools 或 Community 都行
  3. **Windows SDK**（提供 `kernel32.lib` / `ntdll.lib` 等）—— **VS 安装时容易漏勾！** 没装会在编译期间死掉，报 `LNK1181: 无法打开输入文件"kernel32.lib"`。检测命令：`Test-Path "C:\Program Files (x86)\Windows Kits\10"`
  4. 解决：开 Visual Studio Installer → Modify → 勾选「使用 C++ 的桌面开发」工作负载（自动包含 SDK）
- 一行式启动 dev（注入 MSVC + cargo PATH）：
  ```powershell
  $vsBat = "C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\Tools\VsDevCmd.bat"
  & cmd /c "`"$vsBat`" -arch=x64 -no_logo && set" | ForEach-Object {
    if ($_ -match '^([^=]+)=(.*)$') { [Environment]::SetEnvironmentVariable($matches[1], $matches[2], 'Process') }
  }
  $env:Path = "C:\Users\$env:USERNAME\.cargo\bin;$env:Path"
  cd tauri-launcher; npx tauri dev
  ```
- **`tauri.conf.json` 不要保留 `devUrl`**：纯静态 `frontendDist: "../dist"` 项目没有 Vite/dev server，留着 `devUrl` 会让 `tauri dev` 一直卡在 `Waiting for your frontend dev server to start on http://localhost:1420/...` 死循环。已删除（2026-04-20）。
- **冷启动 Rust 全量编译约 4 分钟**（457 个 crate），后续增量 ~5-30 秒。改 `dist/index.html` 不需要重编译，**Tauri 窗口里 Ctrl+R 即时刷新**

### 迭代规则（更新版）
- 改 `dist/index.html` → **dev 模式 Ctrl+R 即时生效**；生产模式需 `npx tauri build`（HTML/CSS/JS 编译进二进制，**不可热修**）
- 改 `lib.rs` / `tauri.conf.json` → 必须 `npx tauri build`（dev 模式下文件 watcher 会自动重编）
- 改 `launcher.py` / `server.js` / `app/**` / `frontend/dist/**` → 这些是 Tauri **resources**（`bundle-resources/` 解压到磁盘），**可热修**：直接覆盖安装目录里的对应文件 + 重启 launcher 即可（macOS 还需 `codesign --force --deep --sign - <app_path>`）

### UI 结构（`dist/index.html`）
- 左侧 76px 窄导航栏（Logo + 4 页 Tab）—— 整页是**单文件 HTML（含内联 CSS + 内联 JS + base64 logo）**，无构建步骤；`window.__TAURI__` 不存在时走 `mockInvoke()`，所以拿浏览器直开 `file://.../dist/index.html` 也能预览样式
- Page 1 **控制台**（`page-launcher`）：环境检测 3 列 pill + 圆形 START 按钮 + API Keys 配置表单
- Page 2 **注册码管理**（`page-keys`）：表格 + 生成/删除 + 复制按钮
- Page 3 **账户管理**（`page-admin`）：统计摘要 4 卡 + 用户表格 + 重置密码/删除 + 确认弹窗
- Page 4 **关于 / 工具**（`page-about`，2026-04-20 新增）：渐变版本号 + 版本副标题 + 「查看最新 release」主按钮 + 4 张工具卡（项目目录 / 用户数据 / 日志目录 / 跳转 release）
  - 版本号通过 `invoke('get_app_version')` 读 `env!("CARGO_PKG_VERSION")`，同时回写侧栏底部 `#navVer`
  - 工具卡用 `.tool-grid` + `.tool-btn`（毛玻璃 hover 上浮，见 `.tool-btn:hover`）
  - 卡片图标统一在 `.tool-icon` 容器内（38px 圆角小框 + accent 色描边 SVG）
- 品牌视觉对齐主应用：`--jf-*` CSS 变量、JetBrains Mono 等宽字体、渐变按钮

### 构建
```bash
cd tauri-launcher
python scripts/build.py          # 完整打包（下载运行时 + staging + Tauri build）
npx tauri build                  # 仅重编译（需先 staging）
python scripts/version.py bump patch  # 版本管理
```

### 包大小
- DMG ~247 MB，解压 ~998 MB

## Docker 部署

### Dockerfile（多阶段构建）
- **Stage 1 (frontend-builder)**: `node:20-alpine`，`npm ci` + `npm run build` 编译 React → `dist/`
- **Stage 2 (production)**: `python:3.11-slim` + Node.js 20，安装 Python 依赖 + Express 运行时依赖（`express`, `http-proxy-middleware`），复制 `dist/` 产物
- React 源码和 devDependencies **不进入**最终镜像
- Express `server.js` 从 `dist/` 目录提供静态文件（不再是 `public/`）

### docker-compose
- Nginx(:80) → Express(:3000) → FastAPI(:8000)
- 用户数据通过 volume 挂载 `./data/users:/app/users`
- `.env` 注入环境变量

### 启动脚本 (start.sh)
- 先启 FastAPI(8000)，等待就绪后启 Express(3000)
- `wait -n` 监控两个进程

### .dockerignore
- 排除 `frontend/node_modules/`、`frontend/dist/`、`venv/`、`data/`、`.env`、`.git/` 等

## 用户附件存储（query_appendix）
- **目录结构**：与 `generated/` 对称，每个对话独立
  - Admin: `users/{user_id}/conversations/{conv_id}/query_appendix/images/...`
  - Consumer: `users/{admin_id}/services/{service_id}/conversations/{conv_id}/query_appendix/images/...`
- **消息格式**：`save_message` / `save_consumer_message` 新增 `attachments` 参数
  ```json
  {"role": "user", "content": "...", "attachments": [
    {"type": "image", "filename": "wx_abc123.jpg", "path": "images/wx_abc123.jpg"}
  ]}
  ```
- **文件服务端点**：
  - Admin: `GET /api/conversations/{conv_id}/attachments/{file_path}`
  - Consumer: `GET /api/v1/conversations/{conv_id}/attachments/{file_path}`
- **来源**：
  - 微信 Bridge（Consumer/Admin）：下载的用户图片存到 `query_appendix/images/`
  - React Chat：后端 `chat.py` 自动从 multimodal message 中提取 base64 图片并保存
- **前端渲染**：`MessageBubble` 通过 `attachments` 字段渲染图片缩略图画廊，点击可放大
- **前端 API**：`attachmentUrl(convId, path)` 构造附件访问 URL

## Advanced 页面可见性（前端 localStorage）
- `GeneralPage.tsx`「高级功能」卡片：两个 Switch 控制 Prompt 页的 Advanced Tab 是否显示
- localStorage key：`show_advanced_system`（操作规则）、`show_advanced_soul`（Memory & Soul），值 `'1'` 显示 / 其他隐藏
- 默认关闭（首次访问 localStorage 无值 → 不显示）
- `PromptPage.tsx` 通过自定义事件 `advanced-settings-changed` 实时响应 GeneralPage 的 Switch 变更
- 当 Tab 被隐藏时，若当前 activeKey 正好是被隐藏的 Tab，自动回退到 `'profile'`

## Remaining Review Notes
- `main.py`: CORS `allow_origins=["*"]` + `allow_credentials=True` — tighten origins in production
- `admin-services.html`: `startNewTest` creates a new API key per test — orphan keys without cleanup
- `app/schemas/requests.py`: many fields use untyped `list`; no max length on strings
- `frontend/server.js`: `API_TARGET` hardcoded — use env for deploys

### AppLayout（`frontend/src/layouts/AppLayout.tsx`）
- 侧栏导航与底部快捷操作：**图标统一使用 `@phosphor-icons/react`**（导航 `size={20}`），不再使用 `@ant-design/icons` 于该布局文件。
- 侧栏品牌区：`/media_resources/jellyfishlogo.png`（32×32）+ 文案 **JellyfishBot**；点击标题区仍可折叠侧栏。
- 侧栏宽度 **240px ↔ 64px**，Sider `style` 含 `transition: width 0.3s ease-in-out`；主菜单用 `inlineCollapsed={collapsed}` 与折叠宽度一致。
- 底部：**上半**为 System Prompt / Subagent / 用户画像 / 文件面板（均有 Tooltip）；Excel **批量运行**在 **设置 → 通用** 内嵌，不再单独侧栏项；**下半**为用户 Avatar + 用户名 + 退出（登出使用 Phosphor `SignOut`）。
- 文件面板按钮开启态：**品牌色 `#E89FD9`** + 浅粉半透明背景 `rgba(232, 159, 217, 0.14)`。

### AdminServices（`frontend/src/pages/AdminServices/index.tsx`）
- 内联 `style.borderRadius` 使用 `themes.css` 变量字符串：`var(--jf-radius-sm|md|lg|bubble)`（对应原 4/8/12/16px 档）；圆形指示点等保留 `borderRadius: '50%'`。

### Scheduler（`frontend/src/pages/Scheduler/index.tsx`）
- 内联 `borderRadius` 与 AdminServices 一致：`4→sm`、`8→md`、`12/10→lg`、`16+→bubble`；原 `6px` 类圆角用 `md`；圆形头像/时间轴点保留 `'50%'`。

### WeChat（`frontend/src/pages/WeChat/index.tsx`）
- 内联 `borderRadius` 与 AdminServices/Scheduler 相同；消息气泡四角简写（原 `14px`/`4px`）用 `lg`/`sm` 变量组合，无 `50%` 圆形元素。

### FilePanel / Modals / Chat 审批卡
- `FilePanel.tsx`、`components/modals/*`（SystemPromptEditor、BatchRunner、SubagentManager、UserProfileEditor）、`Chat/components/ApprovalCard.tsx`：内联 `borderRadius` 已统一为 `themes.css` 的 `var(--jf-radius-sm|md|bubble)` 字符串；`dropZone` 底部两角用 `borderBottom*Radius` + `md`。`ThinkingBlock` / `ToolIndicator` / `SubagentCard` 无内联数字圆角（由 `chat.module.css` 等控制）。
- **FilePanel 排序（2026-04-21）**：`SortKey` 联合类型 `name-asc|name-desc|mtime-desc|mtime-asc|size-desc|size-asc`，下拉按钮放在刷新右侧（`SortAscendingOutlined`），偏好持久化到 `localStorage['jf-filepanel-sort']`。`files` state 只保存 API 返回的原始顺序，渲染前用 `useMemo(sortFiles(files, sortKey))` 派生 `displayedFiles`，4 处旧的 `setFiles(... .sort(...))` 已全部去掉内联排序，否则会和 sortKey 冲突。当 sortKey 是 `mtime-*` 时文件行额外显示 `MM-DD HH:mm`。后端 `app/storage/{local,s3}.py` 的 `FileEntry.modified_at` 已经填好（`st_mtime` / `LastModified`），`api_list_files` 透传到前端无需改动；REST 不加排序参数（纯客户端排序，省一次往返）。

### Settings / Chat 主界面内联圆角
- `Settings/GeneralPage.tsx`、`PackagesPage.tsx`、`InboxPage.tsx`；`Chat/index.tsx`（如流式区缩略图）、`Chat/components/MessageBubble.tsx`（附件预览等）：内联样式中 `borderRadius` 使用字符串 `'var(--jf-radius-sm|md|lg)'`（4/8/12px 档），与 AdminServices 映射一致。

## 消息 Blocks 持久化（交错渲染）
- **目的**：流式输出和历史消息使用统一的交错渲染（thinking → text → tool → text → subagent → text…）
- **后端持久化**：`save_message` / `save_consumer_message` 新增 `blocks` 参数
  - `blocks` 是有序数组，每个元素为 `{"type": "thinking|text|tool|subagent", ...}`
  - `text`: `{"type": "text", "content": "..."}`
  - `thinking`: `{"type": "thinking", "content": "..."}`
  - `tool`: `{"type": "tool", "name": "...", "args": "...", "result": "...", "done": true}`
  - `subagent`: `{"type": "subagent", "name": "...", "task": "...", "status": "done", "content": "...", "tools": [...], "timeline": [{"kind":"text|tool|thinking", ...}], "done": true}`
- **后端构建**：`chat.py::_stream_agent` 和 `consumer.py::_stream_consumer` 在流式过程中同步构建 `blocks[]`
- **前端渲染**：`MessageBubble` 检测到 `msg.blocks` 时使用 `BlocksRenderer`（复用 `ThinkingBlock` / `ToolIndicator` / `SubagentCard` 组件），否则 fallback 旧逻辑（`tool_calls` 在上 + `content` 在下）
- **向后兼容**：旧消息无 `blocks` 字段时使用 fallback 渲染
- **类型定义**：`frontend/src/types/index.ts` 新增 `MessageBlock` 联合类型，`Message` 接口新增 `blocks?: MessageBlock[]`

## WeChat 投递可靠性修复（2026-04-09）
- **Bug 1 — inbox agent 线程池问题**：`contact_admin` 是 sync tool，LangGraph 通过 `run_in_executor` 在线程池执行，`asyncio.get_running_loop()` 失败。修复：`inbox.py` 缓存主事件循环 (`set_main_loop`)，sync 上下文用 `run_coroutine_threadsafe` 提交
- **Bug 2 — context_token 不持久化**：`admin_bridge.py` 更新 `context_token` 后未调用 `_save_admin_session`，重启后 token 过期。修复：每次 `context_token` 更新时调用 `_save_admin_session`
- **Bug 3 — send_message 投递静默失败**：`_resolve_wechat_client` 返回 None 时无日志。修复：`scheduler.py` 在 `_run_agent_task` / `_run_service_agent_task` 中添加 `wechat_warning` step
- **Bug 4 — from_user_id/context_token 空值**：`_handle_send_message_tool` 对空 `to_user` 静默发送。修复：空值检查 + 明确的 `wechat_error` step 和日志
- **关键机制**：LangChain sync tool → `BaseTool._arun` → `run_in_executor(None, self._run)` → 线程池线程无 event loop。所有需要 asyncio 的 sync tool 必须通过 `_main_loop` + `run_coroutine_threadsafe` 调度

## 跨平台启动器（launcher.py）

### 功能
- **旧实例检测**：通过端口扫描（lsof/netstat）检测已运行的 JellyfishBot 进程
- **用户确认后杀掉**：列出占用端口的进程，用户确认后 SIGTERM → SIGKILL
- **自动端口发现**：默认 8000(后端) + 3000(前端)，被占用时自动递增查找
- **双进程管理**：启动 uvicorn + Express(server.js)，子进程异常退出时全部清理
- **干净退出**：Ctrl+C / SIGTERM 时优雅终止所有子进程（5s 超时后 SIGKILL）
- **日志 tee（2026-04-20 新增）**：所有由 launcher 拉起的子进程（backend/frontend）的 stdout+stderr 同时写到 `{project_dir}/logs/{name}-YYYYMMDD.log`（按天滚动 + append 模式 + session header `==== JellyfishBot session start ... ====`）和原始 stdout
  - 实现：`_spawn_with_log(cmd, cwd, env, log_name)` 替代裸 `Popen`，subprocess 的 stdout/stderr 合并到 `PIPE` 后由 `_tee_pipe_to(fh, src)` 后台 daemon 线程逐行落盘
  - `_log_files` / `_log_threads` 用于 cleanup 时关闭文件句柄
  - 与 Tauri 的 `open_logs_dir` 命令配合，用户在「关于 / 工具」里可一键打开日志目录
  - **不要回退到 `subprocess.PIPE` + `communicate()`** —— 长流程会阻塞 `wait()`；当前实现的 daemon 线程 + `bufsize=0` 是经过验证的非阻塞方案

### 文件
- `launcher.py` — 主启动器（纯 Python 标准库，无额外依赖）
- `start_local.sh` — Mac/Linux 快捷脚本（自动检测 python3/python）
- `start_local.bat` — Windows 快捷脚本（双击启动）
- `start.sh` — Docker 容器启动脚本（支持 `BACKEND_PORT` / `FRONTEND_PORT` 环境变量）

### 端口配置
- `server.js`：通过 `FRONTEND_PORT` 和 `API_TARGET` 环境变量配置（原硬编码 3000/8000）
- `start.sh`（Docker）：通过 `BACKEND_PORT` / `FRONTEND_PORT` 环境变量配置
- `launcher.py`：通过 `--port` / `--frontend-port` 命令行参数配置

### 使用
```bash
python launcher.py                    # 生产模式
python launcher.py --dev              # 开发模式
python launcher.py --port 9000        # 自定义后端端口
python launcher.py --backend-only     # 仅后端
python launcher.py --skip-check       # 跳过旧实例检测
```

## Per-Admin API Key（用户级密钥管理）

### 设计概要
- 每个 Admin 可在 Settings → General 中配置自己的 OpenAI/Anthropic/Tavily 等 API Key
- Key **AES-256-GCM 加密**存储于 `users/{user_id}/api_keys.json`
- 优先级链：**用户配置 > 环境变量 > 未配置（提醒设置）**
- Admin 的所有 Agent（主 Agent、Subagent、Consumer Agent）统一使用该 Admin 的 key

### 核心文件
- `app/core/encryption.py` — AES-256-GCM 加解密，master key 自动生成于 `data/encryption.key`（或 `ENCRYPTION_KEY` 环境变量覆盖）
- `app/core/user_api_keys.py` — 用户 API key 的 CRUD（加密存储 + 脱敏返回）
  - `get_user_api_keys(user_id)` — 解密返回全部 key
  - `save_user_api_keys(user_id, keys)` — 加密保存
  - `get_masked_keys(user_id)` — 脱敏 + `*_configured` 标记
- `app/core/api_config.py` — 所有 `get_api_config`/`get_openai_llm_config`/`get_anthropic_llm_config`/`has_provider` 新增 `user_id` 可选参数

### 调用链变更
- `agent.py::_resolve_model(model_id, user_id=None)` — 通过 `api_key` 和 `base_url` 传给 `init_chat_model`
- `consumer_agent.py::create_consumer_agent` — 传 `admin_id` 给 `_resolve_model`
- `ai_tools.py::generate_*` — `get_api_config("image", user_id=user_id)`
- `web_tools.py::web_search/web_fetch` — 新增 `user_id` 参数，查找 tavily/cloudsway key
- `tools.py::create_web_tools(user_id=None)` — 传递 user_id
- `voice/router.py` / `routes/scripts.py` / `routes/models.py` — 传 user_id
- `subagents.py::build_subagent_tools` — 传 user_id 给 `create_web_tools`

### API 端点
- `GET /api/settings/api-keys` — 返回脱敏 key + configured 状态
- `PUT /api/settings/api-keys` — 保存 key（触发 `clear_agent_cache` + `clear_consumer_cache`）
- `POST /api/settings/api-keys/test` — 测试 provider 连通性（openai/anthropic/tavily/all）
- `GET /api/settings/api-keys/status` — 快速检查是否有 LLM provider 可用

### 前端
- `GeneralPage.tsx` — 顶部新增 **API Keys** 卡片（Anthropic/OpenAI/Tavily 三组折叠面板），支持编辑/保存/测试连接
- `ApiKeyWarning.tsx` — 无 LLM Key 时弹出引导 Modal（sessionStorage 防重复弹出）
- `AppLayout.tsx` — 集成 `<ApiKeyWarning />`
- `api.ts` — 新增 `getApiKeys/updateApiKeys/testApiKeys/getApiKeysStatus`

### 支持的 Key 字段
- 密钥：`openai_api_key`, `anthropic_api_key`, `tavily_api_key`, `cloudsway_search_key`, `image_api_key`, `tts_api_key`, `video_api_key`, `s2s_api_key`, `stt_api_key`
- URL：`openai_base_url`, `anthropic_base_url`, `image_base_url`, `tts_base_url`, `video_base_url`, `s2s_base_url`, `stt_base_url`

### 缓存失效
- key 更新后自动调用 `clear_agent_cache(user_id)` + `clear_consumer_cache(admin_id=user_id)`
- Agent 下次请求时重新创建，使用新 key

## Tauri 原生启动器 (tauri-launcher/)
- **目的**：为不会用命令行的用户提供一键启动体验（.exe / .dmg）
- **技术栈**：Tauri v2 + Rust + 单文件 HTML UI（无构建步骤前端）
- **结构**：
```
tauri-launcher/
├── package.json            # @tauri-apps/cli
├── dist/
│   └── index.html          # 自包含启动器 UI（品牌色 + JellyfishBot 风格）
└── src-tauri/
    ├── Cargo.toml           # tauri 2, reqwest, tokio, serde, open, libc(unix)
    ├── tauri.conf.json      # 窗口 720×580, NSIS(Windows), macOS ≥10.15
    ├── build.rs
    └── src/
        ├── main.rs          # entry point
        └── lib.rs           # 8 个 Tauri Command
```
- **8 个 Tauri Command**：
  - `detect_environment` — Python/Node.js/项目文件检测
  - `load_env_config` / `save_env_config` — 读写 `.env`（保留注释/未知键）
  - `test_api_key` — HTTP 测试 OpenAI/Anthropic/Tavily 连通性
  - `start_jellyfish` — 调用 `launcher.py --skip-check`（自动端口）
  - `stop_jellyfish` — SIGTERM(Unix) / kill(Windows)
  - `get_status` — 轮询进程 + 端口存活
  - `open_in_browser` — 打开浏览器到前端
- **UI 流程**：环境检测 → API Keys 填写+测试 → 保存到 .env → 一键启动 → 2s 轮询状态 → 浏览器打开
- **构建**：`cd tauri-launcher && python scripts/build.py`（一键完成）
- **前提**：需安装 Rust 工具链 + Node.js（Tauri CLI 自动通过 npm 安装）

## 打包分发 (Phase 4)
- **build.py** (`tauri-launcher/scripts/build.py`)：一键打包脚本
  - 下载嵌入式 Python (python-build-standalone 3.12.7) + Node.js 20.18.0 到 `.cache/`
  - 暂存 app/、config/、frontend dist、launcher.py、requirements.txt 到 `bundle-resources/`
  - 预装 pip 依赖到嵌入式 Python site-packages
  - 调用 `npx tauri build` 生成安装包 (.dmg / .exe)
  - 参数：`--version X.Y.Z`、`--clean`、`--no-pip`、`--no-frontend`、`--stage-only`、`--target <triple>`
- **version.py** (`tauri-launcher/scripts/version.py`)：版本管理
  - `show` / `bump patch|minor|major` / `set X.Y.Z` / `tag [--push]`
  - 同步更新 `tauri.conf.json` + `Cargo.toml` + `package.json`
- **launcher.py**：支持 `JELLYFISH_PYTHON` / `JELLYFISH_NODE` 环境变量
  - Tauri 启动时将嵌入式运行时路径传入，launcher.py 优先使用
- **lib.rs**：14 个 Tauri Command（base 9 + `open_project_dir` / `open_users_dir` / `open_logs_dir` / `open_release_page` / `get_app_version`）
  - 资源感知路径：`resolve_project_dir(app)` 先查 resource_dir 再 fallback dev 路径
  - `find_bundled_python/node`：先查 resources/{python,node}，再查系统 PATH
  - `detect_environment` 返回 `python_bundled`、`node_bundled`、`first_run`
  - `strip_win_extended_prefix(p)` —— 所有路径出口必经，剥离 `\\?\` 前缀，避免污染 Python 子进程 sys.executable（详见上面 pycryptodome 踩坑）
- **CI/CD** (`.github/workflows/release.yml`)：
  - 触发：push tag `v*` 或手动 workflow_dispatch
  - 三平台矩阵：macOS-arm64、macOS-x64、Windows-x64
  - 产物上传 + 自动创建 GitHub Release (draft)
- **发版流程**：`version.py bump patch` → `build.py` 本地测试 → `version.py tag --push` → CI 自动构建

## 数据备份与恢复 (2026-04-21)

每个用户可在 **Settings → 数据备份** 自助导出/导入自己的全部数据，用于
迁移机器或定期备份。launcher 端则提供整库一键打包按钮。

### Tauri Launcher：一键备份按钮 (lib.rs `pack_backup`)
- About 页底部 `打包备份 (config / data / logs / users)` 按钮 + 旁边问号 hover tooltip
- Rust 端：`rfd::AsyncFileDialog` 弹原生保存对话 + `zip` crate 流式打包
  - 默认文件名：`jellyfishbot-backup-YYYYMMDD-HHmmss.zip`
  - 始终跳过：`__pycache__`、`.pytest_cache`、`node_modules`、`venv`、`.venv`、`target`、`.git`
  - 不打包 `.env`（含明文 API Key，避免 ZIP 泄漏；新机器请重配）
  - 锁定中的日志文件单文件失败不会中断整个备份（计入 `skipped_dirs`）
  - 64KB 流式分块写入，支持 GB 级 users/ 目录
- 前端 `dist/index.html`：`runBackup()` JS handler，进度态 / 成功 toast 含路径与文件数
- 新增 Cargo 依赖：`zip = "2"`、`walkdir = "2"`、`rfd = "0.15"`

### 后端 Per-User 备份 (`app/services/backup.py` + `app/routes/backup.py`)
- 6 个模块（`MODULE_PATHS`）：
  - `filesystem` → `users/{uid}/filesystem/`
  - `conversations` → `users/{uid}/conversations/`
  - `services` → `users/{uid}/services/`
  - `tasks` → `users/{uid}/tasks/`
  - `settings` → `preferences.json` + `subagents.json` + `capability_prompts.json` + `system_prompt*.json[+versions/]` + `user_profile*.json[+versions/]` + `soul/`
  - `api_keys` → 特殊处理：导出时**解密为明文**写入 `api_keys.PLAINTEXT.json`，
    导入时用本机 master key 重新加密（实现跨机器迁移；ZIP 一定要保密！）
- 永久排除：venv/、__pycache__/、.git/、`*.pyc`、`.DS_Store`、`Thumbs.db`
- `include_media=False` 时跳过 `generated/` 与 `query_appendix/images/` 内的 png/jpg/mp4/silk 等媒体文件，保留 JSON/脚本/文档
- ZIP 内含 `_jellyfishbot_backup.json` manifest（kind / version / user_id / modules / created_at），用于导入校验
- Zip-Slip 防御：`_safe_extract_target` 强制目标路径必须在 user_dir 之内
- 4 个 REST 端点（统一 `/api/backup/*`，需登录态）：
  - `GET /api/backup/modules` → 模块列表 + 默认勾选
  - `POST /api/backup/preview` (Form: modules, include_media, include_api_keys) → 每模块文件数 + 未压缩字节数预估
  - `POST /api/backup/export` (Form 同上) → `StreamingResponse` 64KB 分块流式下载，临时文件用完自删
  - `POST /api/backup/import` (multipart: file + mode + password + modules) → 5GB 上传上限
- 两种导入模式：
  - **merge**：跳过已存在文件（只补缺）
  - **overwrite**：先把现有模块整体 `shutil.move` 到 `users/{uid}.pre-restore-{ts}/` 快照目录再解压，**强制要求登录密码确认**（`_verify_password_for_user`）

### 前端 (`frontend/src/pages/Settings/BackupPage.tsx`)
- 设置侧栏新增 `数据备份` Tab（Phosphor `Archive` 图标）
- 导出卡片：6 个模块 Checkbox + 每项问号 Tooltip + 「包含媒体」与「导出 API Keys」开关 + 「预估大小 / 导出 ZIP / 全选 / 重置默认」按钮 + 预估表格（按模块拆 + 合计）
- 导入卡片：`Upload.Dragger` 拖拽上传 ZIP + 模式 Radio (merge/overwrite) + 覆盖时显示密码输入框 + 二次确认 Modal + 结果 Alert（写入数 / 跳过数 / API Keys 重新加密数 / 快照路径）
- API 工具 (`api.ts` 末尾)：`listBackupModules` / `previewBackup` / `downloadBackup`（自动触发 `<a download>` 浏览器保存）/ `importBackup`

### 设计权衡
- **API Keys 明文导出 + 重新加密导入**：ENC: 密文用源机器 `data/encryption.key` 加密，跨机器无法解密，因此选择导出时解密、导入时用目标机器 master key 重新加密。代价是 ZIP 内含明文，需用户自行保密（UI 二次 Modal 确认 + 默认不勾选）。
- **覆盖模式不直接删除**：`shutil.move` 到 `<userdir>.pre-restore-<ts>/`，给用户留后悔药，可手动找回。
- **不强制 manifest**：导入时若 ZIP 缺 `_jellyfishbot_backup.json` 仅 warning，仍尝试按文件路径反查模块（便于从手工压缩的备份恢复）。
- **包含媒体开关**：默认开（数据完整），允许关掉减小备份体积；关掉后聊天记录里的引用图会丢，但 JSON / 脚本 / 文档保留。

## 项目文档

### 文档清单
- `README.md` — 项目总览（中英文双语：#english / #中文 锚点）、快速开始、架构图、技术栈、环境变量、两层架构说明
- `docs/USER_GUIDE.md` — 详细使用指南（中文）：登录注册、界面总览、对话/文件/设置/Service/定时任务/微信/收件箱全功能说明、Consumer API 集成、Docker 部署、FAQ
- `docs/USER_GUIDE_EN.md` — Full user guide (English): complete translation of USER_GUIDE.md
- `docs/DEVELOPER_GUIDE.md` — 开发文档（中文）：架构设计、后端/前端目录结构、路由/依赖注入/Agent 引擎/工具系统/存储层/安全架构详解、SSE 流式处理、设计系统 Token、组件规范、API 参考、扩展开发指南
- `docs/DEVELOPER_GUIDE_EN.md` — Developer Guide (English): complete translation of DEVELOPER_GUIDE.md
- `docs/WEBSITE_DESIGN_BRIEF.md` — JellyfishBot 官网设计需求：水母哲学核心叙事（身体=Admin/触手=Service/进出同口=文件驱动自循环）、竞品分析（OpenClaw/Dify差异化）、6大触手卖点、品牌视觉规范（深海意象）、Landing Page 7个Section（Hero/三句话/场景驱动/水母架构图/QuickStart/技术基石/Footer）、交互动效、文案语气指南
- `docs/filesystem-architecture.md` — 文件系统与 S3 键规范
- `docs/wechat-integration-guide.md` — iLink 微信集成实战
- `frontend/docs/DESIGN_SYSTEM.md` — UI 设计系统规范

## 模型与接口总览（适配/升级前整理，2026-04-22）

用于后续统一换型、对表 OpenAI/Anthropic 新 API 或第三方兼容网关时的**单一索引**；具体 ID 以 `app/routes/models.py`、`app/services/agent.py` 与 `.env` 为准。

### 凭据与解析顺序
- **核心逻辑**：`app/core/api_config.py` — `get_openai_llm_config` / `get_anthropic_llm_config`（LLM）；`get_api_config(capability)` 用于 image / tts / video / s2s / stt，顺序为**用户 per-capability 键 > 用户 provider 回退 > `{CAP}_API_KEY` / env > OpenAI/Anthropic 默认**。
- **用户存储**：`app/core/user_api_keys.py` — `ALL_FIELDS` 含 `openai_*`、`anthropic_*`、Tavily、CloudsWay、以及 `image/tts/video/s2s/stt` 的 `*_api_key` / `*_base_url`（设置页 `GeneralPage` 目前只显式展示 Anthropic / OpenAI / Tavily，其余可经 API 写入或走 env）。

### 聊天 LLM（主 Agent / 子 Agent / DeepAgents）
- **列表与默认**：`GET /api/models` → `app/routes/models.py` 的 `AVAILABLE_MODELS`；有 Anthropic 时默认 `anthropic:claude-sonnet-4-5-20250929`，否则 `openai:gpt-4o`。
- **解析与特化参数**：`app/services/agent.py` — `langchain.chat_models.init_chat_model`；`THINKING_MODEL_CONFIG` 中 Anthropic「thinking」系列映射到非 `-thinking` 的 `base_model` 并带 `thinking` / `max_tokens`；`openai:gpt-5.4` 带 `use_responses_api` 与 `reasoning` kwargs。
- **默认模型串**：`app/core/settings.py` 的 `DEFAULT_MODEL`；`config/agent_config.json` 的 `main_agent.model` 由 `_get_default_model()` 读取。
- **复用同一路径**：批处理 `create_batch_agent`、定时/技能链 `create_user_agent`、Consumer `create_consumer_agent`（`svc_config["model"]` 默认同左）、`app/services/inbox.py` 收件箱代答用 `_get_default_model()`。

### 多模态生成（OpenAI 兼容 HTTP，模型名在代码中写死）
- **图片**：`app/services/ai_tools.py` → `POST {base}/images/generations`，`model: "gpt-image-1"`，capability **`image`**。
- **TTS**：同文件 → `POST {base}/audio/speech`，默认 `model` 参数 `tts-1`（工具层可传），capability **`tts`**。
- **视频（Sora）**：同文件 → `POST {base}/videos` + 轮询 + `GET .../content`，`model: "sora-2-2025-12-08"`，capability **`video`**。

### 语音
- **S2S Realtime 代理**：`app/voice/router.py` — WebSocket `get_api_config("s2s")`，上游 URL `{wss}/realtime?model=gpt-4o-realtime-preview`，头 `OpenAI-Beta: realtime=v1`。
- **STT（聊天页上传等）**：`app/routes/scripts.py` `POST /api/audio/transcribe` — `get_api_config("stt")`，`POST {base}/audio/transcriptions`，`model=whisper-1`。
- **微信语音转写**：`app/channels/wechat/bridge.py`、`admin_bridge.py` — `AsyncOpenAI` / 默认 `OPENAI_BASE_URL`，`whisper-1`（与 per-user stt 配置路径不完全一致，升级时注意）。

### 联网检索
- **实现**：`app/services/web_tools.py` — 优先 `CLOUDSWAY_SEARCH_KEY` + `CLOUDSWAY_SEARCH_URL` / `CLOUDSWAY_READ_URL`（**用户级** `cloudsway_search_key`），否则 Tavily（`TAVILY_API_KEY` 或 `tavily_api_key`），请求官方 `api.tavily.com`。

### 可观测
- **Langfuse**：`app/core/observability.py` — `LANGFUSE_*` 环境变量，供 LangGraph 回调（非「模型」但属外部接口）。

### 子 Agent
- `app/services/subagents.py` — 用户 `subagents.json` 中可选字段 **`model`** 传入 deepagents 子图；未设则与主模型行为一致（由库默认）。

### 升级适配时的提示
- 同步改三处：`(1) app/routes/models.py` 白名单与默认 `(2) agent.py` 的 `THINKING_MODEL_CONFIG` / OpenAI 特参 `(3) 多模态/语音/视频里**硬编码**的 model 字符串`。
- 若增加「只改 env 的模型名」：优先考虑把硬编码提升为**配置或 env**，避免漏改工具层。

## Provider 适配层 + Hybrid Catalog（2026-04-22 Phase 0 上线）

> 目的：解决「新模型上架要改多处硬编码」「非 OpenAI 兼容厂商接入难」两个痛点。
> Phase 0 完成的是**骨架 + 默认 OpenAI 接入**，三家新 vendor（Kimi/MiniMax/豆包）将在 Phase 1-3 增量加入。

### 文件结构
- `app/services/providers/base.py` — 抽象 `ImageProvider/TTSProvider/VideoProvider/STTProvider`；统一 `invoke(model, credentials, **)` + `extras: dict`；`ProviderError` / `UnknownModelError`。
- `app/services/providers/registry.py` — `_REGISTRY[capability][provider_name]`；`register()`、`dispatch(capability, model_id, user_id=, **)`；`parse_model_id("openai:gpt-image-1")`。
- `app/services/providers/openai_image.py` / `openai_tts.py` / `openai_video.py` / `openai_stt.py` — 内置 4 个 OpenAI provider，模块底部 `register(...)` 自动登记；`providers/__init__.py::_bootstrap()` 触发导入。
- `app/services/model_catalog.py` — Hybrid catalog 加载（仓库 + 用户深合并）；`list_models / find_model / get_default_model / resolve_model`。
- `config/model_catalog.json` — 仓库内置默认 catalog；用户级覆盖在 `users/{uid}/model_catalog.json`（同 id 整条覆盖；新 id 追加）。
- `app/core/api_config.py` 新增 `get_provider_credentials(provider, user_id, capability=None)` 与 `has_provider_credentials(...)`：openai/anthropic 走原 `get_*_llm_config` 与 `get_api_config`；kimi/minimax/doubao 直接读 `user_api_keys`。
- `app/core/user_api_keys.py` 扩字段：`kimi_api_key/base_url`、`minimax_api_key/group_id`、`doubao_access_key/secret_key/region`。
- `app/services/preferences.py` 扩 `capability_defaults: {llm/image/tts/video/stt/s2s → model_id}`，`get_capability_default(uid, cap)` 给 catalog 用。

### 路由
- `GET /api/models` — 仍只返回 LLM + 当前默认（向后兼容形状），数据源改为 catalog。
- `GET /api/capabilities/{cap}/models` — 列表 + `available` 标记（按凭据过滤）+ `default_params/params_schema`。
- `GET /api/capabilities/defaults` / `PUT /api/capabilities/defaults` — 用户每 capability 默认 model。
- `GET /api/catalog` — 完整合并 catalog，前端模型管理页用。

### 调用流
- 工具层（`ai_tools.generate_image/speech/video`）公共签名不变，内部改：
  `model_id = resolve_model(cap, user_id=, explicit=model)` → `dispatch(cap, model_id, **)` → 持久化。
- `routes/scripts.py::api_transcribe_audio` 同样改走 `dispatch("stt", ...)`，丢弃硬编码 `whisper-1`。
- `voice/router.py` Realtime URL 中的 model 改为读 catalog `s2s` 默认（仍 `openai:gpt-4o-realtime-preview`）。
- `agent.py::_get_default_model(user_id=None)` 优先尊重 `capability_defaults.llm`，否则回退 `agent_config.json` → `DEFAULT_MODEL`。

### 增加新 vendor 的 5 步标准流程
1. `app/services/providers/<vendor>_<cap>.py` 继承对应抽象类，模块底部 `register(<vendor>Provider())`。
2. `providers/registry.py::_bootstrap` 末尾追加 import。
3. `config/model_catalog.json` 新增 `<provider>:<model>` 条目（含 default_params / params_schema）。
4. `user_api_keys.py` 与 `api_config.get_provider_credentials` 增凭据形态。
5. 前端 GeneralPage 加凭据卡片（凭据形态非 Bearer 时单独 UI）。

### 兼容/取舍
- LLM 仍走 `langchain.chat_models.init_chat_model`，**不**进 Provider 接口（深度依赖 LangChain 的 thinking/tool-use 等）。Kimi 通过 OpenAI-compat 接入也走 `init_chat_model(model="openai:moonshot-...", api_key=, base_url=)`。
- S2S Realtime 是有状态 WebSocket 代理，不进 Provider 接口；但 model 名走 catalog，便于换型。
- 历史会话兼容：catalog 中删除某 model 时由 `dispatch` 抛出结构化错误，不静默回退。
- 不引入新 SDK：所有 vendor 走直 HTTP；后续 vendor 模块可按需自行 `import` 自家 SDK，不影响其他 capability。
- **改图片模型时同步 4 处**：`ai_tools.py` 的 `model` 字段 + `tools.py::CAPABILITY_PROMPTS["image"]` + `tools.py::generate_image` 的 docstring + `.env.example` 注释；这些是 agent prompt 和工具说明里写死的字符串，改了真实 API 不改 prompt 会让 agent 在回复里说错型号。
- **Anthropic 新模型可能引入 breaking 参数 schema**：例如 Opus 4.7（2026-04-16 发布）只支持 `thinking: {"type": "adaptive"}`，传 `budget_tokens` 直接 400；同时 `temperature/top_p/top_k` 设非默认值也 400。新增到 `THINKING_MODEL_CONFIG` 时不要照抄旧条目的 `enabled` + `budget_tokens` 模板，按官方最新 API 写；`_resolve_model` 里只塞 `api_key/base_url`，不会注入采样参数，所以默认安全。

### 接入新模型的快速 checklist（2026-04-22）
- **LLM**：
  1. `app/routes/models.py::AVAILABLE_MODELS` 加条目（thinking 变体放 Anthropic — Thinking 区，普通版放 Anthropic — Latest 区，OpenAI 同理）
  2. 如果带 thinking / reasoning 特参 → `app/services/agent.py::THINKING_MODEL_CONFIG` 加配置（注意 Anthropic 新版 API 的 schema 变化）
  3. 默认模型是否要改？看 `app/routes/models.py::api_list_models` 的 `default_model` 逻辑 + `app/core/settings.py::DEFAULT_MODEL` + `config/agent_config.json::main_agent.model`
- **图片/TTS/视频**：直接改 `app/services/ai_tools.py` 的硬编码 `model` 字段（当前 image=`gpt-image-2`、tts 默认 `tts-1`、video=`sora-2-2025-12-08`）+ `tools.py` 里的 prompt/docstring。

## 定时任务结果作为 Tool Use 进入对话（2026-04-22）

> 目的：把定时任务（admin & service consumer）的执行结果当作「Agent 自己调用了一个名为 scheduled_task 的工具」一样，既在前端有专属卡片展示，也注入回 LangGraph 状态让 Agent 真正"记得"。

### 两层结构
- **L1 持久化 / 前端展示**：`app/services/scheduler.py` 调 `_build_scheduled_task_block(task_meta, output_text, success, error)` 把结果写成 `messages.json` 中的 `tool` block，`name="scheduled_task"`、`args` 是 JSON `{task_meta, status}`、`result` 是输出文本。前端 `ScheduledTaskCard.tsx` 渲染（信息蓝 `--jf-info`，失败红 `--jf-danger`）；管理端用全量元信息卡片，consumer 端通过 `scheduledTaskFriendlyMode={true}` 隐藏 task_id/scope。
- **L2 Agent 记忆注入**：`app/services/scheduled_inject.py` 把同一次结果做成合成 `AIMessage(tool_calls=[scheduled_task])` + `ToolMessage` 对，通过 `agent.aupdate_state(...)` 写进 LangGraph checkpoint。Agent 下一轮会直接看到这条「自己执行过的工具调用」。

### scheduled_inject 关键设计
- **每 thread 一个 asyncio.Queue**：`_pending[thread_id]`，常驻后台 `_drainer_loop` 在线程**空闲**时合并 drain。
- **active 引用计数 + 异步上下文管理器**：`mark_thread_active` / `mark_thread_inactive` 由 4 个流式入口（admin web `routes/chat.py`、consumer web + OpenAI 兼容 `routes/consumer.py`、admin 微信 `channels/wechat/admin_bridge.py`、consumer 微信 `channels/wechat/bridge.py`）调用。微信桥用 `async with scheduled_inject.thread_active(thread_id):` 包住整个流式 + 落库段，确保异常路径也会释放，避免 drainer 死锁。
- **超时兜底**：单 item 在队列里超过 10 分钟就放弃 L2，仅留 L1。
- **截断（避免上下文膨胀，2026-04-26 改造）**：state 里同名 pair 超过 `_MAX_LIVE_PAIRS=5`（通过 `additional_kwargs._sched_pair_id` + `_scheduled_task` 标识识别）时，通过 `RemoveMessage` 移除最老的若干对，并把它们摘要进**单个合成 `AIMessage`(tool_calls=[scheduled_task]) + `ToolMessage`** 对（`additional_kwargs._sched_summary=True` + `_sched_pair_id="summary_<rand>"`）。已存在的旧摘要里的行会被合并保留进新摘要；`evict_count==0` 时不重建摘要。**一定不要用 `SystemMessage` 做摘要**（见下文 Anthropic 坑）。
- **Agent factory**：`_admin_agent_factory(user_id, conv_id)` / `_consumer_agent_factory(service_id, conv_id)` 是闭包，drain 时才调用 `create_user_agent` / `create_consumer_agent`，因此走的是已有 LRU 缓存。
- **失败也注入**：scheduler 的 `_safe_persist_admin_failure` / `_safe_persist_service_failure` 已改为 async，同时写 L1 + 入 L2 队列（status=error）。

### ⚠️ Anthropic "multiple non-consecutive system messages" 坑（2026-04-26 修复）
- **症状**：`langchain_anthropic.chat_models._format_messages` 抛 `ValueError: Received multiple non-consecutive system messages.`；agent.astream 彻底崩掉，之后每次 astream 都复现（因为 bad state 已写进 checkpoint）。
- **根因**：旧版 `scheduled_inject.py` 的截断路径把老的 scheduled_task pairs 替换成 `SystemMessage(content="[历史定时任务摘要] ...", additional_kwargs={"_sched_summary": True})`，这条 SystemMessage 沉在**对话历史中部**。Anthropic 的 formatter 明确只允许**单个 leading SystemMessage**（agent factory 注入的那条），再遇到任何 SystemMessage 都 raise。LangGraph checkpoint 会把这条 SystemMessage 永久存下来，所以**一次坏注入 = 线程永久不可用**，直到被 `RemoveMessage` 清掉。
- **修复**（两层）：
  1. **断未来**：`_build_summary_pair(summary_text, folded_count)` 改返回 `[AIMessage(tool_calls=[scheduled_task]), ToolMessage(content=summary_text)]`，tool_name 仍是 `scheduled_task`，`additional_kwargs` 标 `_sched_summary=True`；`_inject_batch` 用这对替代旧的 SystemMessage。这样摘要和正常的 scheduled_task pair 在类型上完全一致，Anthropic formatter 根本不会把它当 system 看。
  2. **治存量**：新增 `async def repair_scheduled_state(agent, thread_id)`，用 `_find_legacy_system_summary_ids()` 找出所有 `isinstance(SystemMessage) and additional_kwargs["_sched_summary"]=True` 的消息，通过 `agent.aupdate_state(..., {"messages": [RemoveMessage(id) for id in ...]})` 删除。**幂等，状态干净时零成本**。
  3. **调用时机**：5 个 agent.astream 入口必须在调用前先 repair 一次：
     - `app/routes/chat.py::_stream_agent` — `await scheduled_inject.repair_scheduled_state(agent, thread_id)` → `mark_thread_active`
     - `app/routes/consumer.py` — 3 处（`_stream_consumer` / `_stream_openai_compat` / 非流式），全部加 repair 调用
     - `app/channels/wechat/admin_bridge.py` + `app/channels/wechat/bridge.py` — 改用 `async with scheduled_inject.thread_active(thread_id, agent=agent):`（`thread_active` 新增 `agent=` 可选参数，内部自动 repair）
- **辅助 helper**：
  - `_find_summary_ids(msgs)` — 返回**新形态** summary pair 的 id 列表（两条，AI + Tool）
  - `_find_legacy_system_summary_ids(msgs)` — 返回**旧形态** SystemMessage summary 的 id 列表（给 repair 用）
  - `_parse_summary_body(content)` — 从 ToolMessage content（新）或 SystemMessage content（老）中解析出 `["- A\n- B"]` → `["A", "B"]`，升级时保留老摘要内容
  - `_scan_existing_pairs(msgs)` — 显式**跳过** `_sched_summary=True` 的 pair，不进 budget 计数（否则 5 个 pair 的上限会被摘要吃掉）
- **为什么 L1（messages.jsonl 里的 tool block）没受影响**：L1 是前端渲染/持久化用的 `{"type":"tool","name":"scheduled_task",...}`，从来就不是 LangChain message object，和 Anthropic formatter 无关。L2 才是写进 LangGraph state 的真 message。
- **禁止回退**：任何时候都不要为了"省一对消息"而把摘要改回 `SystemMessage`，或在对话历史中部插入任何 SystemMessage。真需要给 Agent 推系统级信息时，用 `AIMessage` 或 ToolMessage 包装；要改全局规则请改 `config["system_prompt"]` 或让 `middleware/todo.py::awrap_model_call` 在最外层 rewrite system_message（这才是 Anthropic 接受的做法）。
- **Smoke test**：包含 7 个用例，覆盖 legacy 检测 / 新形态构造 / budget 排除 / 向前向后 summary body 解析 / Anthropic formatter 接受新形态 / Anthropic formatter 拒绝旧形态（reproduction）。未来修改 `scheduled_inject.py` 时务必回归。

### thread_id 约定
- Admin：`f"{user_id}-{conv_id}"`
- Consumer：`f"svc-{service_id}-{conv_id}"`
- scheduler 必须用同一规则才能命中正确的 LangGraph 线程；新增一种入口时务必同步。

### 排查要点
- Agent "不记得"任务 → 先看 `_pending[tid]` 是否长期非空 / `_active_refcount[tid]` 是否卡 >0（可能是某个流式入口忘了用 `thread_active` 上下文）。
- 摘要消息越来越长 → `_summarize_pair` 截断了 80 字符，但行数会随历史无限增长；如果用户产生数百条历史定时任务，可改 `_extract_prior_summary_lines` 加上行数上限。
- 前端没看到卡片 → 检查 `messages.jsonl` 里 block 的 `type==tool` && `name=="scheduled_task"`，老消息可能没这个结构（Phase 1 之前的 plain text 不会自动迁移）。

## JSONL 存储改造（2026-04-23）

> 目的：把 conversation / scheduler / inbox 这类 append-heavy 数据从「整 JSON 全量重写」改成「只追加一行 + 旁路 meta」，把 `save_message` 之类操作从 O(N) 降到 O(1)，并避免列表/统计要打开每个文件。

### 核心模块
- `app/core/jsonl_store.py` — 全部 JSONL 读写工具：`append_jsonl`（append-only 一行 = 一条记录）、`read_jsonl`、`read_jsonl_tail`（seek-from-end，查最近 N 条不读全文件）、`rewrite_jsonl`（atomic 重写）、`safe_load_json`（坏 JSON 回退 None）、`append_jsonl_many`（批量初始化 / 迁移用）。**所有新增的"流水"写入都走这里**。

### 文件布局（新）
```
users/{uid}/conversations/{conv_id}/
    meta.json          # {id,title,created_at,updated_at,message_count} — 每次 save 用 atomic 重写（小）
    messages.jsonl     # 每行一条消息 — append-only
    query_appendix/    # 用户上传附件（不变）
    .legacy.json       # 迁移备份（保留旧整 JSON）

users/{admin}/services/{svc}/conversations/{conv_id}/
    meta.json + messages.jsonl + .legacy.json   # 同上结构对称
    generated/, query_appendix/                  # 不变

users/{uid}/tasks/
    {task_id}.json                # 任务配置 + run summaries（runs[] 不再嵌入 steps[]）
    {task_id}.steps/{run_id}.jsonl # 每次 run 一个 jsonl，每行一个 step

users/{admin}/services/{svc}/tasks/
    {task_id}.json + {task_id}.steps/...        # 同上对称

users/{admin}/inbox/
    {msg_id}.json                  # 单条消息（不变）
    _index.json                    # 旁路摘要索引：{messages: {msg_id: {summary fields}}}
                                   # list_inbox / count_unread 只读这个，不打开每个 msg
```

### 在线懒迁移（lazy migration）
- 不写一次性脚本，**第一次访问**触发迁移：
  - `conversations.py::_migrate_if_needed(uid, conv_id)` — 旧 `{conv_id}.json` → 新目录 `meta.json + messages.jsonl + .legacy.json`
  - `published.py::_migrate_consumer_conv(...)` — 旧 `{conv_id}/messages.json` → 同目录拆出 `meta.json + messages.jsonl + .legacy.json`
  - `scheduler.py::_externalize_run_steps(...)` — 旧 task.runs[*].steps → `{task_id}.steps/{run_id}.jsonl`，下次 `_save_task` 自动外置
  - `inbox.py::_ensure_index(admin_id)` — 索引文件丢失 / 数量对不上时重建
- 全部幂等。失败保留旧文件不破坏数据。
- backup.py **不需要改**：所有新文件都落在已有的 module 路径（`conversations/`, `services/`, `tasks/`）下，walk 自动覆盖；restore 解压旧格式 ZIP 后，下次访问自动迁移。
- FilePanel **不暴露 conversations/**（保持现状）。

### API 形状不变
- `GET /api/conversations` → `[{id, title, created_at, updated_at, message_count}]`（之前就有 message_count，现在直接读 meta.json 不用计算）
- `GET /api/conversations/{id}` → `{id, title, ..., messages: [...]}`（一次性读 jsonl 全部还原）
- `GET /api/scheduler/{id}/runs` → 内部走 `_attach_run_steps` 把 jsonl 重新拼回 runs[*].steps，前端无感
- `GET /api/inbox` → `{messages, unread_count}`，`agent_response` 字段也在 `_INDEX_FIELDS` 里，前端不缺字段

### 短期记忆注入提速
- `memory_tools.load_recent_admin_messages` / `load_recent_consumer_messages` 改用 `get_recent_messages` / `get_consumer_recent_messages`，内部走 `read_jsonl_tail`，长会话注入只读尾部 N 行而不再 `json.load` 全文件。
- `memory_tools::list_service_conversations` 改成读 `meta.json` 优先，落 legacy 才打开 `messages.json`，并主动调 `_migrate_consumer_conv` 触发迁移。

### 已知边界
- **并发同 conv 写**：`append_jsonl` 用 `open("ab")` 在 Linux 小写入近似原子，Windows 不保证；与原方案（atomic_json_save 直接覆盖会丢一条）相比至少不会丢，但极端并发下可能 line 交错。如果将来出现这个问题，加 `asyncio.Lock` per conv_id。
- **不 fsync**：每次 append 不调 `os.fsync`，否则失去 JSONL 的速度优势。最近的未刷盘行在硬断电时可能丢，但 meta.json 走 atomic 不会损坏。
- **scheduler steps 不限流**（A 方案）：每次 run 结束一次性写 jsonl，run 内累积在内存 — 与改造前完全一致，不引入新风险。
- **inbox `_index.json`** 与文件数对不上时自动 `_rebuild_index`，所以手工删 / 加 `inbox/*.json` 不会破坏列表。

## 前端 Chat 渲染优化（2026-04-23）

> 痛点：30+ 条历史消息时，**输入框打字会卡顿**——每次 `setInputValue` 触发 `ChatPage` 全量 re-render，30 个 `MessageBubble` 全部重跑 `marked + hljs + DOMPurify`。长会话切换 / 滚动时 fps 也会随消息数线性下降。

### 三层优化（全部已落地）

**1. `React.memo(MessageBubble)`（治根：输入卡顿）**
- `frontend/src/pages/Chat/components/MessageBubble.tsx` 默认导出包了一层 `memo`。
- props 全部从 `messages` 数组里取（稳定引用），打字 / 顶层任何不相关 state 变化都会被浅比较挡住，**已渲染消息直接 bail out**，不进 reconcile。
- 调用点（`Chat/index.tsx` 的 `messages.map`）传的 `conversationId` 也是字符串，不会被打字干扰。

**2. `renderMarkdown` LRU 缓存（治根：重复渲染贵）**
- `frontend/src/pages/Chat/markdown.ts` 顶部加了 256 条上限的 `Map<string, string>` 缓存。
- key = `${_fileRevealEnabled ? '1' : '0'}|${text}`（admin / consumer 渲染规则不同，不能共享）。
- 跳过缓存条件：text 长度 ≤ 16 或 ≥ 200KB（极短无收益，极大占内存）。
- 命中 → `Map.delete + Map.set` 把 key 移到队尾（LRU）；满 → 弹掉队首。
- **副作用**：`StreamingMessage` / `SubagentCard` / `ScheduledTaskCard` / `FilePreview` 等所有复用 `renderMarkdown` 的组件都自动受益。流式文本因为每帧字符串都不同会 cache miss，旧条目被自然挤出，**不会污染历史的命中**。

**3. `react-virtuoso` 虚拟化（治本：长会话规模化）**
- 新增 `frontend/src/pages/Chat/components/MessageList.tsx`，封装 Virtuoso + smart-scroll 的所有逻辑。
- 关键设计：
  - **`customScrollParent={scrollParentEl}`**：复用外层 `.messagesContainer` 的 overflow + padding，CSS 一字未改，"回到底部"浮层按钮的 `position: absolute` 上下文也保留。
  - **「实时尾部」节点不要塞进 Virtuoso `Components.Footer`**：`StreamingMessage` / `PlanTracker` / `ApprovalCard` 直接作为 `MessageList` 的兄弟节点挂在 `messagesContainer` 下。详见下方「Virtuoso Footer 高频流式陷阱」。
  - **`followOutput="auto"`**：等价于旧 `useSmartScroll` 的「停在底部时跟随，上滑后停下」行为。
  - **`atBottomStateChange` → `atBottomRef`**：用 ref 不用 state，避免 follow tail 时频繁 re-render。
  - **`computeItemKey`**：`${timestamp}-${role}` 做指纹，新增消息时 Virtuoso 能复用旧 DOM 节点。
- ref 暴露 `scrollToBottom / resetScroll / isScrolledUp`，与旧 `useSmartScroll` API 一致；`Chat/index.tsx` 内部调用点全部无侵入迁移。
- 旧 `frontend/src/pages/Chat/useSmartScroll.ts` 已删除。
- bundle 增量：main.js +56KB / +20KB gzip（react-virtuoso 4.18.5）。

### 边界 / 注意

- **service-chat 没动**：`ServiceChatApp.tsx` 用的是 `StreamingMessage` 而非 `MessageBubble`，且消息流较短，本轮先不动；如果将来 consumer 端也出现长会话卡顿，按相同方法虚拟化即可（`renderMarkdown` 缓存已经全局生效）。
- **`streamBlocks` 仍走 React Context**：流式过程中 `ChatPage` 会被 `streamContext` 拉着重渲染，但 `MessageBubble` 已 memo + `MessageList` 只渲染可视区，所以重渲染只波及到末尾「实时尾部」节点（即正在流式的 `StreamingMessage`，挂在 `MessageList` 兄弟位置），符合预期。

### ⚠️ Virtuoso Footer 高频流式陷阱（2026-04-24 修复）

- **症状**：`write_file` / `edit_file` 流式中，IDE 风格代码卡片出来了，但内容区域一直显示「等待内容…」+ 旋转图标，直到 LLM 写完才一次性把全部代码渲染出来；历史回看正常。中间档的 `tool_call_chunk`（每秒几十次 `args_delta`）丢了。
- **根因**：早期版本把 `StreamingMessage` / `PlanTracker` / `ApprovalCard` 通过 `react-virtuoso` 的 `components.Footer` + `context` 注入。Virtuoso v4 的 Footer 是 `E.memo` 包的，内部 `useEmitterValue("context")` 走 `useSyncExternalStore` 订阅。`scheduleFlush` 每帧 `setStreamBlocks([...])` → `ChatPage` re-render → 新 `context` 对象 publish 到 emitter → 在 React 18 自动批处理 + Virtuoso emitter 内部去重的双重作用下，**高频中间帧大概率被合并/吞掉**，只有「最后一次稳定状态」（流结束、`block.done = true` 那次）才一定重渲染 Footer。其他低频卡片（`ToolIndicator` / `ScheduledTaskCard`）感觉不到，但每秒几十次的 `args_delta` 打字机直接报废。
- **修复**：把 `StreamingMessage` / `PlanTracker` / `ApprovalCard` 从 `MessageList::footerSlot` / Virtuoso Footer 中拿出来，作为 `MessageList` 的兄弟节点直接挂到 `messagesContainer` 下：
  - `frontend/src/pages/Chat/index.tsx` 553 行起改成 `<>{messages.length > 0 && <MessageList .../>} {showStreamBlocks && <StreamingMessage .../>} ...</>`。
  - `frontend/src/pages/Chat/components/MessageList.tsx`：删除 `footerSlot` prop、`FooterContext` interface、`Footer` memo 组件、`components` 对象、`context` useMemo、`<Virtuoso context=... components=...>`，把 `memo` / `useMemo` import 也删掉。
  - 滚动语义零变化：Virtuoso 用 `customScrollParent={scrollParentEl}`，`messagesContainer` 是真·滚动容器，`scrollTop = scrollHeight` 自然包含兄弟节点高度，`scrollFooterIntoView` 行为完全不变。
  - 新增 `messages.length === 0 && showStreamBlocks` 的场景兜底（首条消息流式中，messages 还是空数组）：`scrollToBottom` / `resetScroll` 在 `messageListRef.current` 为 null 时回退到 `scrollParentEl.scrollTop = scrollParentEl.scrollHeight`。
- **教训**：**任何高频（> 30Hz）更新的 React 子树都不要走 Virtuoso 的 `components.Footer/Header/EmptyPlaceholder` 走 `context` 注入**。Virtuoso 的内部 emitter / React 18 的 batching 不保证每个 publish 都被 commit，只保证最终一致。需要每帧都更新的实时 UI 应该走「直接 React state → 直接子树重渲染」的最短路径。Virtuoso 的 components 适合放「不怎么变」的辅助节点（如 loading spinner、版权说明、分页按钮）。
- **conversation 切换时滚到底部**：`loadMessages` 末尾的 `requestAnimationFrame(() => resetScroll())` 会在 setMessages 之后调用 Virtuoso 的 `scrollToIndex({ index: 'LAST' })`。Virtuoso 对未测量的 index 会先估算后落位，UX 与旧实现一致。
- **不要再回头给 `MessageBubble` 加 props**：所有新增 props 必须保证从 `messages` / 稳定引用来源传入，否则会破掉 `React.memo` 的浅比较。如果必须传函数，记得用 `useCallback`。
- **「滚到底部」必须两步走**：
  1. `virtuosoRef.current?.scrollToIndex({ index: 'LAST', align: 'end' })` — 让 Virtuoso 测量并把最后一条 data item 滚入视野（解决「切换对话刚 mount 时 scrollHeight 只反映已渲染头部、`scrollTop=scrollHeight` 落在中间」）。
  2. **下下帧**（`rAF` × 2）`scrollParent.scrollTop = scrollParent.scrollHeight` — 把 footer（StreamingMessage / PlanTracker / ApprovalCard，渲染在 data items 下方）也带进视野（解决「最后一条流式消息只露顶部」）。
  - 单独用任一步都会有 bug，两步缺一不可。封装在 `MessageList::scrollToAbsoluteBottom` 里。
- **「回到底部」按钮可见性必须用 React state 不是 ref**：MessageList 内部 `atBottomRef` 用于 follow-tail 判断（避免每次跨阈值都 re-render Virtuoso），但按钮 className 必须由 ChatPage 的 `isAtBottom` state 驱动，否则 className 不会随用户滚动重算。MessageList 通过 `onAtBottomChange?: (atBottom: boolean) => void` prop 把状态抛上去，ChatPage `setIsAtBottom`。两者并存：ref 给内部用，state 给 UI 用。
- **「回到底部」按钮显示规则放宽**：旧版只在 `isViewingStream && isStreaming && !isAtBottom` 才显示，长会话非流式状态上滑后没法一键到底，UX 偏弱。改为 `!isAtBottom && messages.length > 0` 即显示。
- **`selectedModel` 本地记忆**：`frontend/src/utils/lastSelectedModel.ts` 提供 `getLastSelectedModel/setLastSelectedModel`，localStorage key `last_selected_llm`。`Chat/index.tsx::loadModels` 优先恢复本地值（前提：仍出现在 `/api/models` 返回的 available 列表里，否则回退到 `data.default`）；下拉框 onChange 包了一层 `handleSelectModel` 同步写 localStorage。**只是本地记忆不动后端**——设置页里的 capability_defaults 才是真·全局默认，跨设备/跨浏览器仍走那个；当时刻意没做 B 方案（PUT defaults）以免覆盖用户在设置页的偏好。

### ⚠️ Virtuoso 不算 margin 的兄弟节点重叠陷阱（2026-04-26 修复）

- **症状**：streaming 时 `StreamingMessage` 的顶边框（assistant 气泡顶部 1px border）压在用户最后一条 query 气泡的最后一行行中心，重叠约半行 + 24px 间距，整个流式消息看起来"咬"在用户消息身上。
- **根因**：`react-virtuoso` v4 (`dist/index.mjs:303`) 用 `wo(m.children, e, "offsetHeight", r)` 测量每个 item 高度。`offsetHeight = height + padding + border`，**不含 margin**。`.messageBubble` / `.messageBubbleUser` 之前用 `margin-bottom: 24px` 做消息间距 → Virtuoso 内部记账漏算每条 24px → 它给 `customScrollParent` 模式下的 item-list 设的 `paddingBottom`（行 2451-2452）也短了对应高度 → 外层 `.messagesContainer` 的 scrollHeight 比真实视觉高度短 → 作为 MessageList **兄弟节点**渲染的 `StreamingMessage` / `PlanTracker` / `ApprovalCard` 从亏空位置开始，叠在最后一条历史消息上。
- **修复**：`chat.module.css` 把 `.messageBubble` / `.messageBubbleUser` 的 `margin-bottom: 24px` 改成 `padding-bottom: 24px`。padding 是 `offsetHeight` 的一部分 → Virtuoso 测对真实高度 → 兄弟节点回到正确位置。两个 wrapper 都没有 bg/border，padding 视觉上与 margin 100% 等价。
- **不要做的事**：
  - **不要回退到 `margin-bottom`** —— 重叠 bug 立即复现。
  - **不要在 `MessageList::itemContent` 包一层 padding 的 div + 关掉 bubble 自己的间距** —— 这种"两次重写"的方案改动面更大，且 `StreamingMessage` 当兄弟节点时还得另想办法保持间距。
  - **不要动 `MessageBubble.tsx` / `StreamingMessage.tsx` 给 wrapper 加 inline style** —— 两边都用同一个 `.messageBubble` class，CSS 改一次两边都生效。
- **类比预防**：今后任何虚拟化容器（不只 react-virtuoso，react-window / react-virtual 同理）+ 兄弟节点的布局，**永远不要用 margin 做 item 间距**——一律用 `padding-bottom`（或者父容器 `gap`，前提是父容器自己直接渲染 items，虚拟化场景一般不行）。

## Service「使用情况」面板（2026-04-23）

> 痛点：admin 看不到自己 published service 在被怎么用 —— 既没有 consumer 对话历史的 UI（数据其实在 `services/{svc}/conversations/`，但只有 memory_tools 能读），也没有「这个 key 调了多少次 / 谁失败了」的视图。

### 后端

**`app/services/usage_log.py`（新）** — 轻量请求级 access log：
- 按 service 分目录、按月轮转：`users/{admin_id}/services/{svc_id}/usage/usage-YYYY-MM.jsonl`
- `record_request(admin_id, svc_id, *, channel, key_id, conv_id, endpoint, status_code, latency_ms, ok)` 一行 JSONL，fire-and-forget（异常一律吞，绝不阻塞用户请求）。
- `list_records(..., limit, channel, max_months=6)` 按月份倒序合并 tail 读，只把命中的 channel 算入 limit。
- 字段是「最小集」——后续要 token / IP / model_used 时直接加字段，老记录靠 `rec.get(...)` 向后兼容。
- **不主动归档/清理**。典型 service 量小，单文件一年也就几 MB；爆了再说。

**`app/services/published.py`** —
- `create_consumer_conversation(..., source: str = "")` 多了 source 参数（"web" / "api" / "wechat"），写进 `meta.json`；老会话迁移过来 source 是空，UI 显示「未知」。
- 新增 `list_consumer_conversations(admin_id, svc_id)` —— 只读每个 conv 的 `meta.json`，不读 `messages.jsonl`，O(1) per conv。
- 新增 `delete_consumer_conversation(admin_id, svc_id, conv_id)` —— 整目录 rmtree，附件也一起清。

**接入点**：
- `app/routes/consumer.py`：3 个端点都包了计时 + `record_request`。`/chat` source="web"，`/chat/completions` source="api"，`/conversations` source="api"。SSE 用 `_record_stream(gen, ...)` 包外层 generator，在 finally 阶段记一条（捕获完整流时长 + 异常），客户端中断（GeneratorExit）算 ok。
- `app/channels/wechat/session_manager.py`：`create_consumer_conversation(..., source="wechat")`。
- `app/channels/wechat/bridge.py::handle_wechat_message`：try/finally 计时，channel="wechat", endpoint="wechat:on_message"，key_id 留空（微信没 key 概念）。

**新增 admin routes**（`app/routes/services.py`）：
- `GET /api/services/{sid}/conversations` — list 摘要
- `GET /api/services/{sid}/conversations/{cid}` — 整段对话（含 messages）
- `DELETE /api/services/{sid}/conversations/{cid}` — 硬删
- `GET /api/services/{sid}/usage?limit=&channel=` — 调用记录

### 前端

`frontend/src/services/api.ts` 加 `ServiceConvSummary / ServiceConvDetail / ServiceUsageRecord` 三个 type + 4 个 API 函数。

`frontend/src/pages/AdminServices/index.tsx`：在 service 详情右侧加一个 **"使用情况" `ModuleCard`**，header 用 `Segmented` 切「会话 (N) / 调用 (M)」，刷新按钮重读对应数据。
- 会话视图：Table 列 [来源 Tag / 标题 / 消息数 / 最近活跃 / 查看·删除]，「查看」打开 `Drawer` 用纯文本+pre-wrap 渲染消息（**没复用 admin 的 MessageBubble** —— 避免拖进 markdown / hljs / Virtuoso 一大套，admin 只是要看一眼，能看清就行）。
- 调用视图：Table 列 [时间 / 来源 / Endpoint / Key / 会话 / 状态 / 耗时]，会话列点了直接弹 Drawer 看那条会话；上方多一个 channel 过滤 `Segmented`（全部 / 网页 / API / 微信）。
- 进 service 时 `selectService` 自动并行加载 convs + usage。
- 来源色：web=blue, api=purple, wechat=green。

### 边界 / 注意

- **「会话」的 source 在 conv 创建时就定了**，是 conv 自己的属性，不是消息的属性。同一个 conv 可以被 web / api / wechat 任意一个**首次**触发创建，之后被多个渠道复用也不会改 source。如果以后要按消息粒度区分，得在 `save_consumer_message` 里加 `source` 字段。
- **request log 是 fire-and-forget**：`record_request` 内部 try/except 全吞 —— 哪怕 usage_log 文件被锁、磁盘满，也绝不能让 consumer API 返 500。代价是丢日志比丢请求体面。
- **WeChat 没有 key 概念**：消息从微信扫码 session 走，没经过 sk-svc 鉴权链，`key_id=""`。前端表格 Key 列空就显示 `-`。
- **不要给 record_request 加 await 或 IO 重试**：现在它是同步的、单 `open(path,"ab") + write` 的 syscall，POSIX 保证 < PIPE_BUF 写入原子，多 worker 并发也不会撕行。改成异步 / 加重试反而引入复杂度和新的丢日志路径。
- **Drawer 里直接拼字符串**：`m.tool_calls` 用 `JSON.stringify(...).slice(0, 300)` 截断，避免特别大的 tool result 把 Drawer 撑爆。如果以后要展开看完整内容，再加个「展开」按钮。
- **前端 `usageView` 切「调用」时不会自动重新拉 usage**：进 service 时已经一起拉了一遍，切 tab 只切视图。要刷新得点头部「刷新」按钮或者切 channel 过滤器。这个语义和 react state 流向一致；如果嫌不直观，把 `usageView` 切到 records 时触发一次 `loadSvcUsage` 即可。

---

## MCP / 外部工具

### huozi-mcp（项目级安装，2026-04-23）

- 配置文件：`.cursor/mcp.json`（**项目级**，仅本仓库 Cursor 会加载；`.cursor/` 已在 `.gitignore` 第 72 行，Key 不会被提交）。
- 端点：`https://cloud.huozi.app/mcp`，HTTP 传输 + `Authorization: Bearer <api_key>`。
- workspace：`liushinan1998`（key_id `k_f76b3a174a94bf6b`）。
- Key 是**长期凭据**，被泄露时去 https://huozi.app/workspace 吊销并用 `npx huozi-mcp --client cursor` 或 device-code 流程重新申请。
- 自部署时把 `HUOZI_CLOUD_URL` 指向自己的实例即可，CLI 流程一致。

## 前端移动端适配（2026-04-27）

> 范围：`/chat`（admin）、`/s/{id}`（consumer service chat）、FilePanel、FilePreview、`/settings/*`（含 `/services` / `/scheduler` / `/wechat` —— 它们实际是 settings 的 sub-route 共用 `SettingsLayout`）。断点统一 **≤ 767px**。

### 基础设施
- `frontend/src/hooks/useMediaQuery.ts` —— 新增。SSR 安全；`useMediaQuery(query)` + 导出常量 `MOBILE_BREAKPOINT = '(max-width: 767px)'` + shortcut `useIsMobile()`。内部用 `mql.addEventListener('change', ...)`，Safari < 14 走 `addListener` 兼容分支。
- `frontend/index.html` / `frontend/service-chat.html` / `app/routes/consumer_ui.py::_dev_fallback_html` —— 三处 viewport 都加了 `viewport-fit=cover`（顶到刘海/底部安全区）+ `<meta name="theme-color" content="#1c1c27">`（iOS Safari / Android Chrome 把地址栏涂成品牌色）。**新增 HTML 入口必须同步改**。
- `frontend/src/styles/global.css` 末尾追加了一段 `@media (max-width: 767px)` baseline：`input/textarea/select/[contenteditable]` 强制 `font-size: 16px !important`（阻止 iOS Safari focus 时放大页面）、滚动条 `width/height: 2px`、`.ant-modal` 窄屏铺满、`.ant-drawer-content-wrapper { max-width: 100vw }`、`body { touch-action: manipulation }` 禁双击缩放。**baseline 只放「全站都需要的」规则**，页面微调写各自 `.module.css`。

### AppLayout 抽屉化（`frontend/src/layouts/AppLayout.tsx`）
- 桌面：保留原 `<Sider>` + 内嵌 `FilePreview` + 右侧 `FilePanel` 的三栏布局，零改动。
- 移动：
  - **导航 Sider → `Drawer` (placement=left, width=min(85vw, 320px), zIndex 1050)**，在 `<Content>` 左上 `position: absolute; top:6; left:6; zIndex:20` 放了个 Phosphor `List`（汉堡）按钮作触发器。
  - **必须 `forceRender`** —— Chat 的 conversation 列表是 `createPortal(sidebarContent, document.getElementById('sider-slot'))`；抽屉关闭时如果不强制保留 DOM，`#sider-slot` 就没了，portal 报错。
  - 抽取了 `renderSidebarContents(isCollapsed)` helper，桌面 `Sider` 和移动 `Drawer` **共用同一份 JSX**（包含 `<div id="sider-slot"/>`），保证 portal target 在两种模式下都活着。
  - `useEffect` 监听 `location.pathname`，**切路由自动关掉 nav drawer**（移动侧栏点菜单后立刻收回）。
  - **FilePreview → 全屏右抽屉 (placement=right, width=100vw, zIndex 1040, `destroyOnClose={false}`)**。`onClose={() => closeFile()}` 同步清 `editingFile`，否则抽屉关了 state 还认为文件在编辑，下次打开会闪旧文件。`closeFile` 本身带 dirty-check 的 modal confirm（见 `fileWorkspaceContext`），ESC / 遮罩点击都会触发确认，不会悄悄丢改动。
  - 桌面的**可拖拽分隔条**（chat/preview 间的 `col-resize` div）在 `!isMobile` 判断下直接不渲染。
  - **flex 计算**要补 `isMobile` 分支：`flex: isSettings ? 1 : (showChat ? (showPreview && !isMobile ? splitRatio : 1) : 0)` —— 移动端不再因 `showPreview=true` 就把 chat 列挤到 splitRatio 宽度。
- `HeaderControls.tsx` 也加了 `isMobile` 判断，**分屏切换按钮 `SplitToggle` 及其分隔线在移动端整个不渲染**（移动已全屏 Drawer，分屏无意义）。文件浏览器按钮保留。

### FilePanel 抽屉化（`frontend/src/components/FilePanel.tsx`）
- 把原函数末尾的 `return (<div ref={panelRef}>...)` 抽成 `const panelBody = (...)`，然后末尾分支：`isMobile → <Drawer placement=right width=100vw title="文件" closable>{panelBody}</Drawer>`；桌面保留 `if (!fileBrowserOpen) return null; return panelBody;`。
- `if (!isMobile && !fileBrowserOpen) return null` —— **移动端即便关闭也要 render**（交给 Drawer 处理动画），桌面端可以早退省资源。
- 内部 `<div ref={panelRef}>` 在移动模式下 `width: '100%'`、`borderLeft: 'none'`；**左侧 `col-resize` 拖动把手在 `!isMobile` 下不渲染**（移动端没有拖动宽度的概念）。
- `panelRef` 语义不变（drag-leave 判定、键盘快捷键焦点），因为 ref 挂在内部 div 上，Drawer 是包装层不影响 DOM 树中 ref 指向的元素。
- Drawer `zIndex: 1045`，落在 nav drawer (1050) 和 FilePreview drawer (1040) 之间；如果两个文件相关 drawer 同时开着，FilePanel 会盖在 FilePreview 上——符合「选文件 > 看文件」的交互直觉。

### Chat (/chat) 移动样式（`frontend/src/pages/Chat/chat.module.css` 末尾 `@media (max-width: 767px)`）
- `.chatHeader { padding: 0 12px 0 52px; height: 48px }` —— **左侧留 52px 给浮动汉堡**（AppLayout 里的 List 按钮 36×36 + 留白）；少于 52px 会和 `.chatTitle` 重叠。
- `.chatTitle { font-size: 13px }`、`.messagesContainer { padding: 14px 14px 24px }`、`.messageBody { max-width: 94% }`、`.messageBubble/.messageBubbleUser { gap: 10px; padding-bottom: 18px }`、`.messageAvatar { 28×28 }`。
- `.inputArea { padding: 10px 12px 12px }`、`.inputToolbar { gap: 2px; padding: 0 2px 6px }`、`.capBtn { 32×32 }`（触控友好）、`.inputWrapper { padding: 4px 6px 4px 12px; gap: 6px }`。
- `.emptyState` / `.suggestionChips` / `.streamElsewhereBanner` 各自缩字号和内边距，banner 允许 `flex-wrap` 并 `flex: 1 1 100%`（按钮换行到下一行）。
- `.scrollBottomBtn` 向内收 `right/bottom: 12px`。
- `.modelSelect { width: 180px }`（桌面），移动覆写 `width: 128px`。**注意 `Select` 原来是 `style={{ width: 180 }}`**，改成 `className={styles.modelSelect}` 才能被 media query 覆盖。以后再给 `Select` 加 inline width 就会破坏这个覆写。
- **Chat 主 render 逻辑本身没动**，包括 Virtuoso + 浮动 StreamingMessage/PlanTracker/ApprovalCard 的结构、`scrollToAbsoluteBottom` 两步滚动、YOLO footer tag 等——零侵入。

### service-chat 移动样式（`frontend/src/service-chat/serviceChat.module.css` 末尾 `@media (max-width: 767px)`）
- `.header` padding `10px 14px`、logo 28×28、title 14px、**`.headerDesc { display: none }`**（描述过长会把标题挤出）。
- `.authBox` 宽度 `max-width: calc(100vw - 32px)`，iPhone SE 也不会溢出。
- `.messages { padding: 14px 12px 18px }`、`.userMsg { max-width: 90% }`。
- `.welcomeScreen` 标题 22px、chip `max-width: 100%`（桌面是 360px）。
- `.inputArea { padding: 8px 10px 10px }`、`.inputForm { gap: 6px }`、`.btnSend { padding: 0 14px; font-size: 13px; height: 40px }`、`.imgThumb img { 48×48 }`。
- ServiceChatApp.tsx 本身**没动**，纯 CSS 适配。

### 关键注意事项 / 未来改动须知
- **不要删除 Drawer 的 `forceRender`**：Chat 的侧栏 createPortal 依赖它。
- **给 `Select` / `Input` 设宽度时用 className 而非 inline style**，否则 media query 无法覆写。
- **移动端新增汉堡 / 悬浮按钮要避让 `chatHeader`**：目前占用 `top: 6; left: 6; zIndex: 20`，`chatHeader` 已预留 `padding-left: 52px`。如果再加一个右侧悬浮按钮（比如「新对话」快捷），需要同步预留 `padding-right`。
- **给 drawer 子内容新增「关闭」按钮时复用 `closeFile()`**，不要直接 `setEditingFile(null)` —— 会绕过 dirty-check。
- **新增全 HTML 入口**（比如未来的 `admin-chat.html`）记得加 viewport-fit=cover + theme-color 两个 meta。
- **SettingsLayout 侧栏自动随 AppLayout drawer 一起打开**：`SettingsLayout` 用 `createPortal` 把自己的 `Menu` 注入 AppLayout 的 `#sider-slot`，所以移动端汉堡菜单里**同时**展示「一级导航 + 当前 settings 子页选中项」。**不要**再额外给 Settings 搞独立侧栏，两个侧栏并存会让用户困惑。
- **Settings 管理页通用套路**（2026-04-27 扩展范围）：
  - `global.css` 追加了一段 Ant Design 组件兜底（`@media (max-width: 767px)` 内）：`.ant-tabs-nav-wrap { overflow-x: auto }`、`.ant-segmented { max-width: 100%; overflow-x: auto }`、`.ant-pagination { white-space: nowrap; overflow-x: auto }`、`.ant-tag { font-size: 12px }`、`.ant-drawer-body .ant-form-item .ant-form-item-label { padding-bottom: 4px }`、`.ant-modal-footer/.ant-drawer-footer { display: flex; gap: 8px }` + 子 `.ant-btn { flex: 1 }`、`.ant-table-content { overflow-x: auto }`、`.ant-select-dropdown { max-height: 50vh }`。以及两个工具类 `.jf-mobile-page-top`（`padding-left: 44px`）和 `.jf-mobile-page-header`（`padding-left: 52px; padding-right: 16px`）—— 供页面顶部自助避让汉堡按钮。**新加全站级 Ant Design fix 放这里，不要塞进单个页面 module.css。**
  - 每个 settings 子页面标准套路：导 `useIsMobile()` → 最外层 padding 改成 `isMobile ? '16px 12px 24px' : '24px 32px'` + **强制 `paddingLeft: 52`**（给汉堡按钮让位）→ 顶部头部 `flexWrap: 'wrap'` + `gap: 8~12` → 卡片 padding 从 `24px` 缩到 `16px 14px` → 内部 `grid-template-columns` 收敛（比如 `80px 1fr` / `repeat(2, 1fr)`）→ 过长并列元素改 `flexDirection: 'column'` 堆叠。已适配：`WeChatPage`、`InboxPage`、`PackagesPage`、`BackupPage`、`PromptPage` 及其 3 个 inline 编辑器（`UserProfileEditor` / `SystemPromptEditor` / `SoulSettings`）、`GeneralPage` 及 `ApiKeysCard`、`SubagentManager`（`inline` + `Modal` 双模式）。
  - **Table 横滚**：admin 管理页的 Table 统一加 `scroll={isMobile ? { x: 'max-content' } : undefined}` + `pagination={{ ..., simple: isMobile }}`。**简单列表页** PackagesPage 采用同样做法。
  - **Modal → 全屏**：体积大的编辑/创建 Modal（AdminServices 的 Service 编辑、Scheduler 的任务编辑、SubagentManager 的编辑 Modal、AdminServices 的 API Key Modal、WeChat History Modal）走 `width={isMobile ? '100%' : N}` + `style={isMobile ? { top: 0, paddingBottom: 0, maxWidth: '100%', margin: 0 } : undefined}`，获得全屏效果。**不**用 Drawer 代替 —— Modal 已经能满足大表单的垂直滚动需要，避免重写。
- **AdminServices / Scheduler 的 Stack 导航**（列表↔详情切换，替代双栏）：
  - 桌面：保留左侧 30% 列表 + 右侧 70% 详情的双栏。
  - 移动：用 `currentSvc` / `currentTask` state 决定渲染哪一栏：没选 → 列表铺满 100%；选中 → 详情铺满 100%，顶部多一个 Phosphor `ArrowLeft` 按钮调 `setCurrentSvc(null)` / `setCurrentTask(null)` 回到列表（原生 iOS/Android 的 stack 导航直觉）。
  - 代码层面：**不额外加 `view` state**，直接复用已有的 `currentSvc` / `currentTask`；`display` 属性切换（`isMobile && !showListOnMobile ? 'none' : 'flex'`），**不**做条件渲染（保持 state 不丢、返回列表后滚动位置还在）。
  - Detail header 里的按钮在移动端：图标按钮（`size="small"`）+ 空字符串作文本，复用同一 `<Button>` JSX 不写两套。返回按钮放在 header 最左侧，`order: isMobile ? ... : 0` 调 flex 顺序把标题挤下一行。
  - **`AdminServices` 的 `createPortal` 不改**：里面没用到 sider-slot（它的导航走 SettingsLayout 的 Tabs portal，自动跟着 AppLayout drawer 走）。
- **踩坑**：
  - 给 Modal 传 `style={{ top: 0 }}` 一定要配 `paddingBottom: 0, maxWidth: '100%', margin: 0`，否则 Ant Design 默认的 `top: 100px` 和 `padding-bottom: 24px` 会让 Modal 还差一截。
  - 不要在 settings inline 编辑器里用固定 `width: 280` 给侧栏，移动端必须 `isMobile ? '100%' : 280`，不然会横向溢出。
  - 通用兜底中 `.ant-modal-footer > .ant-btn { flex: 1 }` 会让所有 Modal 的按钮变等宽——视觉上更适合触屏，但**如果某个 Modal 故意只有一个按钮，也会被拉成全宽**。接受这个 trade-off，不再单独 override。

## 一次性封面页

### 活字印刷术

- `cover/movable-type.html` —— 活字印刷术封面页 demo，单文件、CDN 依赖（Google Fonts + ECharts SVG 渲染）、纯 CSS 纸纹与朱砂印章、ECharts 横向时间线（毕昇→古腾堡→王选 8 个节点，上下交错避让）。如果以后要做"项目封面/PPT 首页"风格的静态页，可以直接抄这套色板与版式：米色三段渐变 `#fbf3df → #f1e3c0 → #e6d2a3`、朱砂主色 `#9b2a2a`、衬线字体 `Noto Serif SC` + `Cormorant Garamond`、四角 1px 边框 + inset 6px 同色描边的"花边"装饰。

## 文档读取工具与深链跳转（2026-04-24）

### 后端：`app/services/document_tools.py`
- 两个新工具，均经 `safe_join` + `_is_allowed` 校验，禁止越权：
  - **`read_document(path)`** — 结构化文本提取
    - `.pdf` → pypdfium2 全文，每页前注 `[Page N]`
    - `.docx` → python-docx，按段落保留标题层级
    - `.xlsx` → openpyxl 转每个 sheet 为 markdown 表格
    - 纯文本 `.md/.txt/.csv/.json/...` → 提示用 `read_file`（不重复造轮子）
    - 输出超 `_MAX_TEXT_BYTES=200KB` 自动截断（约 50-70K tokens），提示改用 vision 工具或写脚本
  - **`view_pdf_page_or_image(path, page=1)`** — 多模态视觉弥补
    - PDF → pypdfium2 渲染单页为 PNG (`_PDF_RENDER_SCALE=1.5`，约 108 dpi) → base64
    - 图片（png/jpg/jpeg/webp/gif/bmp）→ base64
    - **限单页粒度**：PDF 必须传 page 参数（1-based），不允许批量
    - **运行时 throttle**：`_MAX_VIEW_PER_PATH=5`，同 path 单 thread 超限拒绝（用 `contextvars.ContextVar` 隔离，每请求独立计数；不同 path 计数独立）
    - 返回 `list[dict]`（`text` + `image_url` block），LangChain 0.3+ ToolNode 自动包成 multimodal `ToolMessage`
  - 旧版 Office `.doc/.xls/.pptx` 一律返回明确错误，引导用户另存或写脚本
- **注册策略**：
  - `agent.py::create_user_agent` — 两个工具**始终注入**（read-only + 路径限制 + throttle，零滥用风险）；`documents` 能力 prompt 无条件追加
  - `agent.py::create_batch_agent` — 仅注入 `read_document`（视觉工具按行调用太慢/太贵；单行需要 vision 让用户自己写 pypdfium2 脚本）
  - `consumer_agent.py::_create_consumer_read_tools` — 在 consumer 上下文中重新包一层 `@tool`（path 相对 `docs_dir`、`_is_allowed` 过滤、禁止 `generated/`），复用 `document_tools.py` 的纯函数 `_extract_pdf_text/_extract_docx_text/_extract_xlsx_text/_render_pdf_page_to_png_b64/_read_image_to_b64/_bump_and_check_view_count` + 共享常量 `_PLAIN_TEXT_EXTS/_IMAGE_EXTS`；prompt 同步追加
- **依赖**：`requirements.txt` 加 `pypdfium2>=4.30.0`（Apache-2.0、纯 wheel、无系统依赖、PDF 文本提取 + 渲染一体）、`python-docx>=1.1.0`、`pillow>=10.0.0`
- **不进沙箱**：两个工具直接在主进程跑，不走 `script_runner` 子进程沙箱（无需 numpy/matplotlib 类资源限制；本身就是 read-only + path-limited）

### 前端：`<<FILE:/path#anchor>>` 深链跳转
- **统一语法**：`<<FILE:/docs/x.md#标题>>` / `<<FILE:/docs/x.pdf#page=3>>`，标题不存在静默落到文件顶部（用户明确选择，不报错）
- **markdown.ts 改造**：
  - 导出 `slugifyHeading(raw)` 让 FilePreview 能匹配 agent 写的标题文本
  - `splitPathAndAnchor(raw)` 把 `<<FILE:>>` 体按首个 `#` 拆 `[path, anchor?]`
  - `filePathToHtml(path, anchor?)`：PDF 直接把 `#anchor` 拼到 iframe `src` —— 浏览器原生 PDF viewer（Chrome/Edge/Firefox）按 Adobe Open Parameters 规范处理 `#page=N`/`#zoom=`/`#search=`，无需 JS plumbing
  - `nonMediaFileToHtml(path, anchor?)` / `buildRevealAction(path, anchor?)` / `postProcessInlinePaths` 全部传 `data-jf-anchor` 透传给 reveal 委托
  - 非媒体文件 pill 显示 `📄 name #anchor`，方便用户预判跳转目标（CSS 类 `.jf-file-link-anchor`，accent 色 + monospace）
  - `INLINE_PATH_RE` 也支持可选 `#anchor`（保守：要含 `/` 且有扩展名）
  - `SANITIZE_OPTS.ADD_ATTR` 显式列出 `data-jf-file/data-jf-anchor`（DOMPurify 默认放行 data-*，列出仅是文档化）
- **fileWorkspaceContext.tsx**：
  - `revealInBrowser(path, anchor?)` 第二参可选；`pendingAnchor: string|null` + `consumePendingAnchor()` 一次性消费协议
  - 全局 `[data-jf-file]` 点击委托同时读 `data-jf-anchor` → `revealInBrowser(path, anchor)`
  - **anchor set 顺序**：先 `setPendingAnchor` 再 `await openFile`，保证 MarkdownPreview mount effect 能同步拿到（content 已缓存场景）
- **FilePreview.tsx**：
  - `MediaView` (kind='pdf')：`useEffect` 注入 `#${pendingAnchor}` 到 iframe src + 调 `consumePendingAnchor()`；`baseUrl` 永远是裸 `mediaUrl(path)`，避免重复 fragment
  - `MarkdownPreview`：`useEffect([pendingAnchor, html])` 用 `requestAnimationFrame` 延一帧再做 3 步 fallback 解析：
    1. `getElementById(anchor)` 直接命中（agent 写了 slug 或 CJK 直传）
    2. `getElementById(slugifyHeading(anchor))`（agent 写的是原标题文本）
    3. 遍历 h1-h6 做大小写不敏感的 `textContent.includes(needle)` 子串匹配
  - 找不到时**静默 no-op**，不弹 toast，符合「找不到打开文件顶部」设计
  - 一定 `consumePendingAnchor()`（无论是否命中）—— 防止用户切换文件后旧 anchor 重放

### 不要做的事
- **不要给 read_document 加 `.pptx` 支持**：python-pptx 抽文本噪声大、速度慢；用户写脚本更可控
- **不要让 view_pdf_page_or_image 一次返回多页**：会让 base64 体积失控 + agent 滥用读长文档；想全文用 read_document
- **不要尝试解析 PDF 的 PDF outline (bookmark) 跳到标题**：需要 pypdfium2 outline API + 复杂的 page index 转换；当前 `#page=N` 已覆盖 90% 场景
- **不要给 MarkdownPreview 的 anchor 解析加 toast 报错**：找不到默认行为是「打开文件顶部」，弹错反而打断用户阅读
- **不要在 chat-inline 的 markdown 渲染里做 anchor 滚动**：聊天气泡里点 `<<FILE:/x.md#标题>>` 应该跳到 FilePreview，不应该在聊天里"原地"滚（聊天里没有完整 markdown 渲染上下文）—— 当前实现就是走 reveal 路径，符合预期

## 聊天框 @ 文件引用（MentionPicker, 2026-04-24）

> 痛点：admin 想让 agent 看自己 docs/scripts/generated 里的某个文件时，必须手动写完整路径（甚至记不住），跟 markdown 渲染层早就支持的 `<<FILE:>>` clickable 形成不闭环。改造目标：在 chat 输入框打 `@<query>` 唤出 fuzzy 候选，回车插入，进 agent 时自动展开成 `<<FILE:>>` —— 与 `data-jf-file` 跳转面板形成 **「输入引用 ↔ 渲染跳转」** 完整闭环。

### 数据流（关键 invariant）

- **前端输入**：用户在 textarea 里看到的是 `[[FILE:/docs/x.md]]`（人类可读、可手动删除/编辑）
- **后端落库**：`app/routes/chat.py::api_chat` 在调 `save_message` / `stamp_message` 之前先调 `app.services.prompt.expand_file_mentions(canonical_message)`，把 `[[FILE:/path]]` 全部 rewrite 成 `<<FILE:/path>>`
- **持久化的对话历史 = agent 看到的内容 = markdown.ts 渲染时识别的内容 = `<<FILE:/path>>`**，三者完全一致，没有任何渠道会看到 `[[FILE:>>` 字面字符串
- `expand_file_mentions` 是幂等的（第二次调用 no-op）；`stamp_message` 也是 idempotent 的，两者前后顺序都安全

### 文件清单

- `app/services/prompt.py::expand_file_mentions(content)` —— 单一来源的 rewrite。接受 str / multimodal-list 两种 content 形态（与 `stamp_message` 完全对称，只处理 `type=="text"` 的 block）。**未来如果 consumer / 微信 bridge 也想支持 @ 引用，直接在它们的入口 import 同一个函数即可，绝不要复制粘贴正则**。
- `app/routes/files.py::api_list_file_index(root, include_dirs)` —— 新增 `GET /api/files/index`，BFS 走 `storage.list_dir`（local/S3 兼容），返回扁平 entries 列表。**深度上限 12、条目上限 10000、跳过 `__pycache__/node_modules/.git/venv/...` 和 `.DS_Store` 等垃圾**；点开头 (`startswith('.')`) 一律不进索引，避免 `.cursor/.config` 之类污染候选
- `frontend/src/utils/recentFiles.ts` —— `localStorage[jf-recent-files]` 容量 30 的 most-recent-first 列表；`pushRecentFile` 在 `fileWorkspaceContext.openFile` 成功打开后调用（含 media/binary 分支），用作 fuzzy 排序的 boost 信号
- `frontend/src/utils/fuzzyMatch.ts` —— subsequence 评分 + 多档 type bonus（exact 50 / startsWith 30 / contains 20 / path-contains 10）+ recent boost（`max(0, 200 - 7*idx)`）。**空 query 时返回 recents 在前的兜底列表**，让"@ + 立即按方向键"也能选最近用过的文件
- `frontend/src/pages/Chat/components/MentionPicker.tsx` —— 纯展示组件，下拉位于 `inputWrapper` 上方（`position: absolute; bottom: calc(100% + 6px)`，要求 wrapper `position: relative`）。MAX_CANDIDATES=8，匹配字符高亮用 `highlightMatches` 拆段。**键盘事件不在 picker 里处理**，由 chat 页的 `handleKeyDown` 在 textarea 上拦截，避免"focus 跑到 picker 后键盘 routing 复杂"
- `frontend/src/pages/Chat/index.tsx` —— 集成点：`mention` state（`{active, triggerStart, query, activeIndex}`）+ `detectMentionTrigger(value, cursor)` 词边界检测 + `handleInputChange` 替换 `setInputValue` + `handleKeyDown` 优先消化 `↑↓/Enter/Tab/Esc`（mention active 时）+ `insertMention(item)` 替换 `@<query>` 为 `[[FILE:/path]] `（带尾随空格）+ `requestAnimationFrame` 把光标精准放到 token 末尾
- `frontend/src/pages/Chat/chat.module.css` 末尾新增 `.mention*` 系列（picker 浮层 + header + list + item active 高亮 + name 匹配字符 primary 色加粗 + dir hint 右对齐 ellipsis）

### 触发规则（不要破坏）

- `@` 必须**前面是行首 / 空格 / `\n` / `\t`**（避免 email / `obj.@prop` 误触发）—— 在 `detectMentionTrigger` 内做检查
- query（`@` 与光标之间的文本）含任何空白字符立即关闭 picker（用户开始打下一个词，不再属于 mention 上下文）
- `handleInputChange` 是**唯一**的检测入口；点击改变光标位置不会重新触发 picker，这是有意为之（避免误触）；用户想用 `@` 必须主动打字
- 发送消息时 `setMention({active: false, ...})` 必须显式重置，否则 streaming 完成后下一条消息可能继承上一次的 mention 状态

### 性能 / 容量

- 文件索引 lazy-load：第一次 `@` 触发时拉 `/api/files/index`，缓存到 `fileIndex` state；同会话期间不再重复请求。**未来若需要刷新（用户在 FilePanel 新建文件后）**：在 `fileWorkspaceContext` 暴露一个 `bumpFileIndexEpoch` 让 chat 页订阅重拉
- in-memory fuzzy：典型 admin 用户几百到几千文件，subsequence 算法 O(n*m) 实测每次按键 < 5ms，无需 web worker
- BFS 上限：10000 条目 / depth 12，对正常项目永远不会触顶；触顶时 `truncated: true` 返回，前端目前未渲染该提示（可后续加）

### onMouseDown vs onClick（picker 选择陷阱）

- MentionPicker 内的候选项用 `onMouseDown(e) { e.preventDefault(); onSelect(...) }` —— **不能用 `onClick`**。原因：click 事件 fire 时 textarea 的 blur 已经触发，textarea 的 `onBlur` 会 setTimeout 关闭 picker（120ms 防抖让 mousedown 优先），但即便不关，焦点丢失也会让 `requestAnimationFrame(focus textareaRef)` 行为不可控。`mousedown.preventDefault()` 直接阻止 textarea blur，整个交互保持在 textarea focus 上下文里
- textarea blur 必须延迟 120ms 关 picker，**不能立即关**：mousedown → blur → click 的事件顺序中，立即关会让 picker DOM 在 click 之前消失，onSelect 永远不触发

### antd TextArea ref 的特殊取法

- antd v5 `Input.TextArea` 的 ref 是 `TextAreaRef`（`{ resizableTextArea: { textArea: HTMLTextAreaElement }, focus, blur, ... }`），**不是直接拿到 `HTMLTextAreaElement`**。`insertMention` 里要操作 `setSelectionRange`，必须从 `el.resizableTextArea.textArea` 取真实 dom：
  ```ts
  ref={(el) => {
    const inner = (el as unknown as { resizableTextArea?: { textArea: HTMLTextAreaElement } } | null)?.resizableTextArea?.textArea ?? null;
    textareaRef.current = inner;
  }}
  ```
- 不要 `import { TextAreaRef } from 'antd/es/input/TextArea'`（路径不稳定）；inline cast 更轻

### FileTokenInput（contenteditable 芯片输入框，2026-04-24 升级）

- **为什么换掉 antd TextArea**：`textarea` 是纯文本控件，无法在文本内嵌入可点击/可删除的富文本芯片（chip）。contenteditable div 可以把 `[[FILE:]]` token 渲染成交互式小卡片
- **文件**：`frontend/src/pages/Chat/components/FileTokenInput.tsx`
- **DOM 结构**（保持平坦，避免浏览器在 Enter 时插入 `<div>` ）：
  ```
  <div contenteditable="true">
    普通文字
    <span class="jf-token-chip" data-mention-path="/docs/x.pdf" contenteditable="false">
      <button class="jf-token-chip-x">×</button>  ← hover 时从宽度 0 展开
      <span class="jf-token-chip-icon">📄</span>
      <span class="jf-token-chip-name">x.pdf</span>
    </span>
    更多文字
  </div>
  ```
- **序列化**：`serializeDom(root)` 逐节点拼字符串，chip 输出 `[[FILE:/path]]`，BR 输出 `\n`，其他节点忽略
- **反序列化**（仅外部 value 变化时）：`hydrateFromText(root, value, onDelete)` 用正则把 `[[FILE:...]]` 换成 chip DOM 节点；以 `internalValueRef` 跟踪"自己的 onChange"，避免 re-hydrate 死循环
- **Backspace 芯片删除**：`getChipBeforeCaret` 检测"光标紧贴 chip 后方"时，拦截 Backspace 整体移除 chip span
- **× 按钮**：用 `mousedown`（不用 `click`）阻止 focus 丢失；CSS 默认 `width:0`，hover 时展开到 16px
- **文件夹芯片**：chip 上带 `data-jf-is-dir="true"` / `data-jf-file="/path"`，全局 `[data-jf-file]` 点击委托（fileWorkspaceContext）直接识别并导航文件夹
- **光标偏移**：`getCaretOffset` 用 `Range.cloneContents` 克隆 root→caret 片段再 serialize，优雅处理 chip 和文本混合；`setCaretOffset` 遍历 childNodes 匹配目标偏移
- **mention 触发**：检测逻辑与原 `detectMentionTrigger` 完全相同；在组件内部的 `syncToParent` 里运行，通过 `onMentionTrigger(null|{triggerStart,query})` 上报，父组件不再需要在 onChange 里自己检测
- **键盘路由**：`mentionPickerActive=true` 时 ↑↓/Enter/Tab/Esc 经过 `onMentionNavUp/Down/Confirm/Dismiss` 回调上报给父；Enter（无 Shift）→ `onSend`；Shift+Enter → 手动插入 `<br>`；浏览器自动插入的 `<div>` 被 `flattenBrowserWrappers` 展平

### 文件夹点击导航（2026-04-24）

- **问题**：点击 `<<FILE:/some/folder/>>` 或从 `[[FILE:]]` 中引用了一个目录路径时，`revealInBrowser` 调用 `openFile(path)` → `api.readFile(path)` → 报错
- **修复**（`stores/fileWorkspaceContext.tsx`）：
  - `revealInBrowser` 新增 `isDir?: boolean` 第三参数
  - 全局点击委托读取 `el.getAttribute('data-jf-is-dir')`，并同时 fallback 检查 `path.endsWith('/')`
  - 命中目录逻辑：`setBrowserPath(dirPath); setFileBrowserOpen(true); return` — 不调 `openFile`
- **chip 的目录检测**：chip span 上设置 `data-jf-is-dir="true"` + `data-jf-file="/path/"`

### 不要做 / 已否决

- **不在前端做 `[[FILE:>>` → `<<FILE:>>` 的 rewrite**：会让发到 agent 的 message 和 textarea 显示的不一致，且 SSE 重放历史时也会麻烦。统一在后端 `chat.py` 一处做 rewrite，单一真相源
- **不在 service-chat 接入 @-mention**：consumer agent 的文件视图本来就是 `allowed_docs` 限定子集，UX 会和 admin 不一致；且 consumer 大多在微信里用，textarea 输入比例小。如果以后要加，复用 fuzzyMatch / MentionPicker / `expand_file_mentions` 三件套即可，但 `/api/files/index` 需要新增 consumer 版本（按 `allowed_docs` 过滤）
- **不主动持久化 fileIndex**：每次 chat 页 mount 重拉一次足够新鲜（cost: 一次轻量 BFS）；持久化反而要处理失效

### 端到端验证

- `python -c "from app.services.prompt import expand_file_mentions; ..."` 用例覆盖：纯文本 / 多模态 list / 无 mention / 幂等场景，全部通过
- fuzzyMatch JS port 单元测试：`wec` + recent boost 让 wechat_send.py 排第一（249 vs 49）；`rep` → report.pdf 50+19；`docs` → 目录 exact 匹配 74 vs 内容文件 10；空 query 推 recents 1000/999；`nope` → 空数组
- ReadLints 全部 9 个改动文件：0 errors
- `npx tsc --noEmit`：除已有的 `react-virtuoso` 缺类型声明（环境问题，不在本次改动范围）外，0 errors

## FilePanel 上传进度（2026-04-25）

> 痛点：选/拖一堆文件或一个 200 文件的文件夹，UI 没任何反馈，用户不知道在跑还是卡了；超大文件夹甚至会让 fetch 静默挂十几秒。

- **`api.ts::uploadFilesWithProgress(path, files, keepStructure, onProgress)`**：用 `XMLHttpRequest` 替代 fetch，挂 `xhr.upload.onprogress` 拿字节级进度（fetch 只有 `ReadableStream` 实验性方案，浏览器兼容性差）。自动按 `MAX_BATCH_BYTES=50MB` / `MAX_BATCH_FILES=50` 切批，避免单 request 体积过大触发后端 body-size 限制；`cumulativeLoaded/cumulativeTotal` 跨批次累加，UI 看到的是「整体进度」而不是「当前批次进度」。401 / 网络错误 / 业务失败都映射到 reject 并把后端 `detail` 透出来。
- **`UploadProgressEvent` 字段**：`{loaded, total, batchIndex, batchCount, cumulativeLoaded, cumulativeTotal, currentFileName, fileCount}` —— `currentFileName` 是当前批次第一个文件的 heuristic（onprogress 只能拿到 XHR 的整体进度，没法对应到具体文件）。
- **`FilePanel.tsx::runUploadTo(targetDir, files, keepStructure)`** 公共 helper 收口三个旧调用点：button 选择 / 面板根拖拽 / 文件夹行拖拽（`onItemDrop`）。原本三处各自 `await api.uploadFiles + message.success + loadFiles` 重复代码，现在统一走 runUploadTo。
- **rAF 节流 onProgress 回调**：localhost 上 xhr.upload.onprogress 一秒能触发几十上百次，直接 setState 会让 React 反复 re-render 整个 FilePanel。改用 `pendingTick` + `requestAnimationFrame(flushTick)` 模式，每帧最多 setState 一次。
- **Sticky 进度横幅** 放在 toolbar 下方、剪贴板提示之上、文件列表之上：旋转 LoadingOutlined + `上传中…/上传文件夹中…` + 文件总数 + 多批时 `第 N/M 批` + `已传字节 / 总字节` + antd `Progress` 条（品牌粉色 `--jf-primary`）+ 当前文件名（截断 + title 提示）。完成后 setTimeout 400ms 移除，让用户看到 100% 瞬间。
- **不在切批之间 `loadFiles` 刷新**：只在最终成功后刷一次，避免列表抖动；失败时同样不刷新（保持错误前状态）。
- **Future**：如果将来要做「单文件级进度」（哪个文件正在传），需要把每个文件改成独立 XHR 请求并自己合并进度，权衡是请求数量上去了；当前批量 multipart 是后端友好的方案。

## 文件夹地址点击导航修复（2026-04-25）

- `fileWorkspaceContext.tsx::revealInBrowser(path, anchor?, isDir?)` 不能只依赖 `data-jf-is-dir` 或尾斜杠判断目录。原因：`@文件夹` 输入 chip 会带 `data-jf-is-dir="true"`，但聊天中 agent/markdown 渲染出的普通 `<<FILE:/docs>>` / `<<FILE:/generated/images>>` pill 没有文件系统元数据，而且多数 agent 不会给目录路径补尾斜杠，结果会继续走 `openFile('/docs')` → `api.readFile` → 报「打开文件失败」。
- 当前修复：在 explicit flag / trailing slash 之外，`revealInBrowser` 会通过**父目录元数据**判断：`api.listFiles(parentDir(path))` 后找到同名 `FileItem`，只有 `item.is_dir === true` 才 `setBrowserPath(path)` + `setFileBrowserOpen(true)` + `setPendingAnchor(null)` 并 return；确认是文件或找不到时才 fall through 到 `openFile(path)`。
- **不要用 `api.listFiles(path)` 直接探测目录**：某些后端/路径组合对「文件路径」可能返回空列表而不是抛错，前端会误判为“空文件夹”，导致 `<<FILE:/docs/a.md>>` 打不开文件、FilePanel 显示空目录。目录探测必须以父目录 listing 的 `is_dir` 为准。
- 取舍：普通文件点击会多一次 `GET /api/files?path=<parent>` 元数据请求，再打开文件；但不会弹 toast，且父目录列表通常很小。若未来 FilePanel/markdown 渲染能直接携带 `data-jf-is-dir`，可跳过这个探测。

## 移动端 safe-area + FilePanel↔Preview 互切（2026-04-27）

> 痛点：(1) iOS Safari / Android Chrome 的地址栏和底部工具栏会**盖住页面底部**（输入框+功能按钮被截掉一截），用户截图反馈「对话框和功能选择栏被浏览器自身按钮遮住」。(2) 移动端点开文件后 FilePanel drawer（`zIndex: 1045`）仍盖在 FilePreview drawer（`zIndex: 1040`）上方，用户得**手动关掉**文件面板才能看到预览；关掉预览回来又得重开面板。

### 修复 1：动态视口 + safe-area 内边距

- `frontend/src/styles/global.css` 窄屏块新增：
  ```css
  html, body, #root {
    height: 100vh;        /* 老浏览器 fallback */
    height: 100dvh;       /* iOS Safari / Android Chrome 响应 URL bar 收缩 */
    min-height: 100vh;
    min-height: 100dvh;
  }
  ```
  `100dvh` 不是 `100vh` —— 后者是「初始视口高度」，地址栏收起后页面反而被下方留白；`dvh` 会随浏览器 UI 动态调整，输入栏永远挂在可见区域底部。
- `pages/Chat/chat.module.css` 窄屏块：
  - `.chatHeader` `padding` 改为 `env(safe-area-inset-top, 0px) 12px 0 calc(52px + env(safe-area-inset-left, 0px))`，`height` 改为 `calc(48px + env(safe-area-inset-top, 0px))` —— 刘海屏顶部不遮住 chatTitle，且汉堡菜单也不会被压住（菜单按钮自身的 top 已 offset）。
  - `.inputArea` `padding` 四向全部用 `env(safe-area-inset-*, 0px)` —— iPhone 小黑条（home indicator）区域不再占用输入框点击面积。
- `service-chat/serviceChat.module.css` 窄屏块：`.header` 和 `.inputArea` 同样用 `env(safe-area-inset-*)`。
- `layouts/AppLayout.tsx` 汉堡按钮：`top`/`left` 改为 `calc(6px + env(safe-area-inset-*, 0px))`。
- **前提：`index.html` / `service-chat.html` 必须有 `viewport-fit=cover`**（否则 env 都返回 0）。已在历史 commit 里加过，别回滚。

### 修复 2：FilePanel ↔ FilePreview 互切（仅移动端）

- 现状：两个组件都是独立的全屏 Drawer，z-index 叠在一起，用户体验割裂。
- 方案：在 `AppLayout.tsx` 加一个 effect 监听 `editingFile` 变化，**只在 isMobile=true 时启用**：
  - `prevEditingRef: null → path`（打开文件）+ `fileBrowserOpen=true` → `setFileBrowserOpen(false)` 并用 `reopenPanelRef.current=true` 打标记；
  - `prevEditingRef: path → null`（关闭预览）+ `reopenPanelRef.current=true` → 自动 `setFileBrowserOpen(true)` 并清标记。
- 桌面端：effect 在 `!isMobile` 分支 early return，行为完全不变（两个面板同屏 split 显示）。
- 关键点：
  1. `prevEditingRef.current = editingFile` 在 effect 开头立刻更新，避免同一次转换被重复触发；
  2. deps 里必须含 `fileBrowserOpen` —— 否则用户在 `<<FILE:/folder/>>` 链接（`revealInBrowser` 先 `setFileBrowserOpen(true)` 再决定是否 `openFile`）场景下会漏触发；
  3. `reopenPanelRef` 用 `useRef` 不用 state —— 不需要触发 re-render，只是个跨调用的持久标记；
  4. 从 desktop → mobile 切屏时，effect 会把 reopenPanelRef 重置为 false，避免误触发。
- 不改 FilePanel/FilePreview 内部 —— 逻辑集中在 AppLayout 这个唯一同时持有两个 drawer 的地方，避免两个组件互相引用。

### 不要做的事

- **不要把 FilePreview drawer 的 zIndex 改成比 FilePanel 高** 就想靠盖住解决 —— 用户还是看到两层 drawer 叠加的动效，下层 drawer 的遮罩会让上层输入失焦，体验更差。
- **不要用 `100svh`**（small viewport height）：它是地址栏**展开**状态的高度，输入框会悬空在屏幕中间，桌子上留白。要的是动态响应，用 `dvh`。
- **不要动 `position: fixed` 布局**：桌面端 FilePanel 本来是 inline 侧栏（`position: relative`），只在 isMobile 时才是 Drawer。强制 fixed 会破坏桌面 split layout。

## 黄金 ETF 定时分析任务 Demo 设想（2026-04-28）

- 用户想在另一个 demo page 做一个“定时分析任务/黄金 ETF 智能交易系统”演示，样式需贴近现有定时任务页面，而不是微信双手机聊天组件。
- 演示重点：6 个定时任务时间轴（09:00 数据采集、14:00 决策前采集、14:10 新闻扫描、14:25 预测推送、15:30 验证、周日 22:00 周度演化）、4 因子决策打分、风控仓位、预测-验证-演化闭环、两条微信推送样例。
- 视觉方向：深色海洋主题、sea token 风格、玻璃卡片、任务时间轴、因子权重条、手机推送卡片、状态徽标；可用单个 HTML 原型先验证信息层级。

## 个性规则历史与 service Markdown 表格（2026-04-29）

- 个性规则历史复用 `PromptVersion` 结构，后端入口在 `app/services/prompt.py` 的 profile versions 区域，路由在 `app/routes/settings_routes.py` 的 `/api/user-profile/versions/*`。若继续扩展历史记录元数据，应保持和 system prompt versions 的接口形状一致。
- service-chat 直接复用 admin 的 `StreamingMessage` / `chat.module.css`，因此 service 根节点必须提供 `--jf-border`、`--jf-bg-raised`、`--jf-text` 等 `chat.module.css` 依赖的 token；否则 Markdown 表格边框等样式会因 CSS 变量缺失而失效。

## 个人 Prompt 拆分：用户规则 + Agent 记忆（2026-04-29）

> 痛点：原 `user_profile.custom_notes` 既要承载用户手写的"我的规则"，又想顺便让 agent 在对话中记下学到的偏好，两边职责打架；agent 不能直接写，用户也没办法一眼看出"哪些是我写的、哪些是 agent 自己学到的"。

**数据层**
- `user_profile.json` 在原有 `custom_notes` 之外加两个字段：`agent_notes: str`（agent 维护的自由文本，覆盖式）+ `agent_notes_locked: bool`（用户开关，锁定后 agent 写入会被拒绝）。
- `app/services/prompt.py` 新增 `get_agent_notes / set_agent_notes / is_agent_notes_locked`；`set_agent_notes` 内部以 `auto_version=False` 调 `set_user_profile`，保证不污染 `profile_versions` 历史。
- **重要 dedup**：顺手把 `set_user_profile` 改成只在 `custom_notes` **真的变化**时才 `save_profile_version`。否则只要保存任何一个字段都会写一条新版本，UI 会被噪音淹没。这是行为改动，记得别再回滚。

**Prompt 注入**
- `build_user_profile_prompt` 拆成两段输出：先 `## 个性规则`（用户手写部分），再 `## Agent 记忆`（agent 维护部分）。即使 agent_notes 为空也会注入 `## Agent 记忆` 占位段，目的是**主动告诉 agent 该工具存在 + 何时该调用**，否则 agent 不会主动去 read 一个空字段然后写。
- 锁定状态会在注入文本里明示 `⚠️ 用户已锁定此区域`，让 agent 不要一直试探。

**Agent Tool**
- `app/services/tools.py::create_update_personal_memory_tool(user_id)` 暴露 `update_personal_memory(content: str)`：覆盖式（agent 自己负责合并新旧），上限 8000 字符，锁定时返回 `[LOCKED] ...` 让 agent 立刻识别拒绝原因。
- 注册路径：`agent.py::create_user_agent` tools 列表 + `subagents.py::SHARED_TOOL_NAMES` + `build_subagent_tools` —— 主 agent 默认有，subagent 也可声明使用。
- 写入会通过 `set_agent_notes` → `set_user_profile`（auto_version=False）→ `clear_agent_cache(user_id)`，下一次会话立刻看到新的 system prompt。注意 **当前会话不会即时刷新 system prompt**（agent 实例已缓存），下一次 conversation 才生效；如要即时性需进一步改造。

**API**
- 新增 `GET /api/user-profile/agent-notes` 和 `PUT /api/user-profile/agent-notes`（body `{content, locked}`），与 `PUT /api/user-profile` 解耦：UI 上保存 Agent 记忆走独立 endpoint，不会触发 user-rules 的版本化或副作用。
- 现有 `PUT /api/user-profile` 仍然接受 `agent_notes/agent_notes_locked` 字段（`UserProfileRequest` 已扩展），为以后批量保存留口子，但 UI 默认不走这条路。

**前端**
- `UserProfileEditor.tsx` 顶部新增"Agent 记忆"面板（cyan 主题色 `rgba(95,201,230,0.06)`）：包含字符数、未保存 Tag、清空按钮、Lock Switch（lock/unlock icon），独立 `保存` 按钮。下方保留现有"我的个性规则"块（含版本历史 / 重命名 / 回滚 / 删除）。
- Modal 标题改为"个性规则与 Agent 记忆"，避免误导用户以为只有规则。
- Agent 记忆区**没有版本历史**（按设计），但有"清空"按钮（要二次确认 + 还得手动点保存）。

**踩过的坑 / 不要做的事**
- ❌ 不要让 agent 通过现有 memory subagent 的 `soul_write` 去写这个字段：soul 系统是一堆散文件、需要用户开 `memory_subagent_enabled / soul_edit_enabled`，权限模型完全不一样。Agent 记忆是单一文本块、主 agent 默认就有权写，两套并行。
- ❌ 不要把 agent_notes 也加进 `profile_versions`：用户决定不要历史；如果将来要加，需要单独 `agent_notes_versions.json`，不要混在 profile versions 里。
- ❌ 不要把工具改成 append 式（`append_personal_memory`）：会让 agent 难以删除/纠正旧记忆，覆盖式更清晰也更逼着 agent 想清楚保留什么。
- ❌ consumer agent **不**注入这个工具：consumer 是面向 service 消费者的，不该写入 admin 的 user profile（如果将来 consumer 也要 memory，应该走独立的 conversation-scoped memory，不复用这个字段）。

## 中英 i18n 架构（2026-04-29）

> 痛点：项目原本所有 UI 文案都是硬编码中文（59 个 .tsx/.ts 文件、~1600 行含中文），既不便于面向海外，也无法按用户偏好切换语言。这套方案以 i18next 为骨架，做到"localStorage 即用 + 后端 preferences 跨设备同步 + Accept-Language 统一前后端语言"。

**库选型：react-i18next**
- 业界标准，生态完善，bundle ~30KB gz；自带的 `useTranslation` / `Trans` / `react-i18next/initReactI18next` 已经够用。
- 配套 `i18next-browser-languagedetector`：先看 `localStorage['jf-lang']`，再看 `navigator.language`（命中 `en*` → en，否则 zh）。

**前端目录**
- `frontend/src/i18n/index.ts` —— init i18next、定义 `setLanguage / currentLang / SUPPORTED_LANGS`，写入 `localStorage` 同时 `document.documentElement.lang`。
- `frontend/src/i18n/locales/zh.json` & `en.json` —— **单文件**字典（按 `namespace.key` 命名，无 react-i18next namespace 拆分），方便 grep / 一次性翻译。namespace：`common / nav / login / settings / general / profile / prompt / header / chat / service / preferences / capabilities`。
- `frontend/src/main.tsx` 与 `frontend/src/service-chat/main.tsx` 双入口都 `import '../i18n'`，service-chat 也能切语言。
- `tsconfig.json` 加 `resolveJsonModule: true`（之前没显式开启，依赖 `moduleResolution: bundler` 默认行为；现在保险加上）。

**Antd locale 联动**
- `App.tsx` 内 `useTranslation()` 监听 `i18n.languageChanged`，根据当前语言把 `ConfigProvider locale` 切到 `zhCN` 或 `enUS`，DatePicker / Pagination / Popconfirm 默认按钮跟随。
- 没有走 i18n.ts 内一次性 `init` 的方案，目的是在 dev 热更时可以无刷新切换。

**Accept-Language 同步**
- `frontend/src/services/api.ts::request()` 自动注入 `Accept-Language: <jf-lang>`，所有 fetch 都带（包括 SSE 之外的 REST）。后端用这个 header 决定返回的 message 语言。
- 前端永远先读 `localStorage`，没设置时 fallback `navigator.language`，最后 fallback `zh`。这套规则与 i18n.ts 内 detector 顺序一致。

**后端 i18n**
- `app/core/i18n.py` —— `MESSAGES` 字典直接内联 zh/en 两份；`resolve_lang(request)` 看 Accept-Language 第一个 tag；`t(key, lang, **kwargs)` 翻译并支持 `str.format`。
- 范围**仅限 HTTP 路由的 success/error message**（如 `user_profile.updated`）。**不**翻译 agent system prompt / tool description / capability prompts —— 那会改变 agent 的输出语言，是另一个更大的设计决定，留给 Phase 2。
- 已收口：`app/routes/settings_routes.py` 内的 `system_prompt.empty / system_prompt.updated / user_profile.updated / version.not_found` 全走 i18n。其余路由（auth / chat / batch...）后续按需替换；任何路由要 i18n message，加一个 `Request` 依赖 + `t("key", resolve_lang(request))` 即可。

**用户偏好持久化**
- `app/services/preferences.py::_DEFAULTS` 加 `"language": ""`（空串 = auto，由 Accept-Language 兜底）。`update_preferences` 校验 `language` 必须 `"" | "zh" | "en"`。`get_language_preference(user_id)` 读取。
- 前端 `services/api.ts::UserPreferences` 同步加可选 `language?: '' | 'zh' | 'en'`。
- `AppLayout.tsx` 在 `user` 变化时 `getPreferences()`：如果后端有值就 `setLanguage()` 同步到 i18n；如果没有就 `updatePreferences({ language: currentLang() })` 把当前选择推上去（首次登录 = 把客户端检测结果固化到云端）。

**切换器入口（按用户要求两处）**
1. **Chat sidebar 用户栏**：`AppLayout.tsx` 的 user-row，`<LanguageSwitcher />` 放在 Settings 齿轮按钮**左侧**；`syncBackend=true`（默认）—— 切换会写后端 preferences。
2. **GeneralPage**：第一张卡片即"界面语言"，`<LanguageSwitcher variant="compact">` + 描述文本。
3. **Login 页**：右上角浮动 `<LanguageSwitcher syncBackend={false} />`，因为登录前没 token、不能 PUT preferences。
4. **service-chat header**：`<LanguageSwitcher syncBackend={false} />` 放在 header 右侧；consumer 没有 admin token 调 /api/preferences，只走 localStorage。

**LanguageSwitcher 实现细节**
- `frontend/src/components/LanguageSwitcher.tsx`，两个 variant：`icon`（地球图标按钮，默认）/ `compact`（带文本按钮，用于设置页）。
- 用 antd 的**静态** `message`（不通过 `App.useApp()`），因为 service-chat 没 `<AntdApp>` 包裹会拿不到 message context；静态 message 在 admin 里有 console warning 但功能正常，trade-off 接受。
- `setLanguage()` 改完才 `updatePreferences`（错了不阻塞 UI），失败仅 console.warn。

**已翻译范围（Phase 1）**
- AppLayout（菜单 / 退出 / 主题切换 tooltip / 移动端 hamburger）
- Settings/index.tsx 菜单全部 9 项
- Settings/GeneralPage.tsx：标题、5 张卡片、ApiKeysCard 全部按钮和说明
- Login.tsx：tagline、Tab、placeholder、必填提示、按钮
- Modals: UserProfileEditor / SystemPromptEditor 全部按钮、tooltip、Modal title、placeholder、message.success/error 文案、capability 标签（CAP_LABEL_KEYS 通过 i18n 解析）
- Chat: header（新对话 / 暂无对话 / 标题 fallback / welcome / 输入 placeholder / model placeholder / 上传 / Plan Mode / YOLO / 4 个 suggestion chip / 各 banner 与按钮 / message.success/error）
- service-chat: header / 鉴权框 / welcome / empty hint / 输入区 placeholder / 上传 / 发送 / 移除图片 / authError 文案 + 顶部加切换器

**Phase 2（待办、按需补全）**
- AdminServices (`pages/AdminServices/index.tsx` ~1500 行)
- Scheduler / WeChat 页
- Settings 子页：PromptPage / SubagentManager / PackagesPage / BackupPage / InboxPage
- BatchRunner / SoulSettings / FilePanel / FilePreview / EditDiffViewer / ApprovalCard / PlanTracker / ToolIndicator / SubagentCard / VoiceInput / ImageAttachment 等组件细节
- service-chat: ServiceToolBadge 状态文案 + serviceApi.ts 错误信息
- 后端：把其余路由（auth / chat / batch / scripts / files...）的 success/error message 也走 i18n（按需 PR，不强求）
- agent system prompt / tool description / capability prompt 是否要 i18n —— 单独评估

**踩过的坑**
- ❌ `LanguageSwitcher` 一开始用 `App.useApp().message` —— service-chat 没 antd ConfigProvider/AntdApp，会触发 warning 且消息不显示。改用静态 `message`。
- ❌ JSON 文件不能有 trailing comma（en.json 的 `chat.suggestionServiceMsg` 后面不能多一个 `,`）—— 加 key 时容易写错，改完一定要 build 确认。
- ❌ react-i18next `<Trans>` 组件用 `i18nKey` + 子元素插值，注意子元素的 index 从 1 开始（`<1>Tab</1>` 对应第一个子节点），不是 0。
- ❌ 后端 `resolve_lang(request)` 必须要把 `Request` 加进路由签名（FastAPI 不会自动注入），加完别忘了 `from fastapi import Request`。
- ❌ Capability prompt 的 i18n key 与 raw key 共存（`CAP_LABEL_KEYS` 映射），`t(labelKey)` 找不到时 fallback 到 raw key（i18next 默认行为），不会渲染空白。

## API key 与助手协作（安全）
- 用户若在对话中粘贴完整的第三方 API Key，应视为已泄露：建议用户在服务商控制台立刻撤销该 Key 并新建，勿在聊天记录、工单、截图中保留明文 Key。
- 助手不应使用「对话里出现的明文 Key」代为发起真实 API 调用（避免 Key 写入终端日志、工单回显等二次泄露）；可信做法是用户把 Key 仅存于本地 `.env`（已 gitignore）、由用户在专属环境自行 `curl`/脚本验证。
- OpenAI 图像能力以官方文档为准（型号名随产品迭代变化，对话里说的「gpt image2」请对照 [Images API](https://platform.openai.com/docs/guides/images) 当前支持的 `model` 字段）；无权访问该 Key 所属的账户与计费状态时，无法在对话内代为判定「是否能调用」某模型。
- 仓库 `config/model_catalog.json` 的 `image` 能力默认列出 `openai:gpt-image-1` 与 `openai:gpt-image-2`（可与 `users/{uid}/model_catalog.json` 深合并覆盖）；`app/services/providers/openai_image.py` 通过 `POST {base}/images/generations` 传 `model` 字段调用。

## 已知运维风险：scheduler 启动期 OOM（t3.medium / 4 GB 复现）

**症状**：dmesg 报 `python invoked oom-killer`，docker 容器内单一 python 进程 anon-rss ≈ 3.5 GB，每 30 秒循环 OOM-kill→docker 重启→再 OOM。t3.medium 默认 **无 swap**，更易触发。

**根因**：`app/services/scheduler.py::HeapScheduler` 启动时 `_reload_from_disk()` 把所有用户 `_meta.json` 中 `enabled=True` 的任务一次性灌入 heap；如果服务停机期间错过了多次 cron/interval 触发，重启瞬间 `_dispatch_due()` 会 **一次性 `_fire` 所有过期任务**（**无并发上限**），每个 fire 都会 `create_user_agent` → 一个完整 LangGraph + deepagents + provider client，单 agent ≈ 100–300 MB。少量 enabled cron + 几小时停机就足以 OOM。

**确认嫌疑**：进容器后 `find users -name '_meta.json' -path '*/tasks/*' | xargs grep -l '"enabled": true' | wc -l`，若 ≥ 几个且类型有 cron/interval，且 `next_run_at` 都早于「停机前」就是这条路径。

**应急止血**（按文档顺序）：
1. 把所有 tasks 的 `enabled` 改为 False 或重命名 `tasks/` 目录，让 fastapi 起来；
2. 加 swap（`dd if=/dev/zero of=/swapfile bs=1M count=4096 && mkswap /swapfile && swapon /swapfile`）；
3. 升 t3.medium → t3.large（4 GB → 8 GB）。

**治本（已实现，2026-05）**：
- `HeapScheduler.start()` 创建 `asyncio.Semaphore(scheduler_concurrency_slots())`；每条到期任务仍会 `_fire`，但 `_run_*_cleanup` 内 **`async with` 信号量** 后才执行 `_execute_*`，从而在进程内限定 **同时为「运行中」的任务数**。槽位数由 **`SCHEDULER_MAX_CONCURRENT`** 控制（默认 **4**，范围 1～64，`app/services/scheduler.py::scheduler_concurrency_slots()`）。
- `main.py::startup` 支持 **`DISABLE_SCHEDULER`**（`1`/`true`/`yes`/`on`）：不调用 `get_scheduler().start()`，shutdown 时也跳过 stop；仍可保留 HTTP 与其它功能，用于紧急救火。
- 启动日志会打印：`HeapScheduler capacity: at most N task(s)...`。
- （可选后续）单次任务内 overdue coalescing、agent LRU 仍为增强项。

**不要做**：指望靠 `gc.collect()` 扛住十几路并发 Agent —— 先降 `SCHEDULER_MAX_CONCURRENT` / 停用部分任务。

**启动期 CPU 爆满（另一常见病）**：`venv_manager.restore_all_venvs()` 曾对每个含 `users/{uid}/venv/requirements.txt` 的用户 **并行** `pip install -r`（`asyncio.gather` 无上限）。用户多时启动瞬间 CPU/磁盘同时打满。现已改为 **`RESTORE_VENVS_CONCURRENCY`（默认 1，有界并发）**，并可 **`RESTORE_VENVS_ON_STARTUP=0`** 完全跳过。`UVICORN_WORKERS` 须保持 **1**，否则每进程各跑一套 scheduler + venv 扫描。救援时可设 **`SAFE_STARTUP=1`**（`main.py::_apply_safe_startup`）一次强制 `DISABLE_SCHEDULER=1` + `RESTORE_VENVS_ON_STARTUP=0`。

## 定时任务 UI 决策：列表只列 root，spawn 子任务走谱系图（2026-05）

**背景**：v2 spawn-tree 上线后，`list_tasks` / `list_service_tasks` 默认用 `list_all_tasks_flat` 平铺 root + 全部 descendants 返回。当某条 root 任务 spawn 出几十个孙子（典型场景：循环型 agent 自驱），sidebar 会被瞬间挤爆，每条都是一整张卡，加载 + 渲染都拉胯。

**决策**：sidebar **只显示 root 任务**，spawn 子任务通过详情页的"谱系图"（reactflow + dagre）查看。

**实现**（`app/services/scheduler.py`、`app/routes/scheduler.py`、`frontend/src/pages/Scheduler/index.tsx`）：

- `scheduler.py::list_tasks(user_id, *, roots_only=True)`：默认只走 `st.list_root_tasks`（仅扫 scope_root 第一层），传 `roots_only=False` 才退回老的 `list_all_tasks_flat`（root + descendants 平铺）。
- `scheduler.py::list_service_tasks` / `list_all_service_tasks` 同款 `roots_only` 参数（默认 True）。
- `routes/scheduler.py::api_list_tasks` / `api_list_service_tasks` / `api_list_all_service_tasks` 暴露 query 参数 `?roots_only=false`，诊断脚本或老调用方需要平铺时显式带上。
- 调度堆 `HeapScheduler._reload_from_disk` 仍用 `list_all_tasks_flat`（spawn 子任务必须进堆才能按时执行，**这里不能改成 roots_only**，否则 spawn 出来的子定时任务再也不会触发）。

**配套前端调整**（同一提交）：

- `index.tsx::_resetViewIfFlat` / Segmented 切换条的 `hasFamily` 判断加入 `descendants_count` 信号：避免 v1→v2 迁移后 `children_count` 字段没更新时，谱系图按钮直接消失。
- `index.tsx` GraphView/TimelineView 的 `rootTaskId` prop 改为「root 任务直接用 `currentTask.id`；spawn 子任务才用 `root_task_id || spawn_chain[0] || id`」。`root_task_id` 字段在某些旧 `_meta.json` 里可能陈旧/缺失，root 直接走 self 才是最稳的「这棵树」入口。

**用户访问 spawn 子任务的入口**：

1. 从 sidebar 选 root 任务 → 详情页切到「谱系图」→ 点击想看的子任务节点 → 切回详情视图。
2. 谱系图节点之间会自动 `selectTask(id)`，spawn 子任务详情依然加载 `/scheduler/{id}` 全量数据，没有信息损失。
3. **不能从 sidebar 直接看到 spawn 子任务**（这是有意为之，spawn 子任务通常是 agent 自己派生的瞬时任务）。

**调试 spawn-tree 异常**：

- `scripts/diag_list_tasks.py` —— 看 `list_tasks` 实际返回（root only）和 `list_all_tasks_flat`（root + descendants）的差异。
- `scripts/diag_one_task.py` —— 给定任务 id，打印它的 spawn lineage / 在磁盘上的路径 / 在 list 里能不能看到。
- 谱系图只显示孤立节点的常见原因：(a) root 确实没 spawn 任何子任务；(b) v1 迁移时 child 目录没正确建到 parent 内；(c) `_meta.json` 里 `parent_task_id` 指错。前两者用 `find users/*/tasks -name '_meta.json'` + 查目录结构定位。

## AWS Bedrock 集成（2026-05-27）

**背景**：项目新增 AWS Bedrock 作为 LLM provider，通过 Bearer Token (ABSK-prefixed API Key) 调用 bedrock-runtime 的 InvokeModel API。

**认证方式**：Bearer Token（非 IAM SigV4），环境变量 `BEDROCK_API_KEY` + `BEDROCK_REGION`。

**调用端点**：`POST https://bedrock-runtime.{region}.amazonaws.com/model/{inference_profile_id}/invoke`

**已验证可用的 Anthropic 模型及其 Inference Profile ID**：

| 短名称 | Inference Profile ID | 说明 |
|--------|---------------------|------|
| claude-opus-4-7 | us.anthropic.claude-opus-4-7 | 旗舰（不支持 temperature） |
| claude-opus-4-6 | us.anthropic.claude-opus-4-6-v1 | |
| claude-opus-4-5 | us.anthropic.claude-opus-4-5-20251101-v1:0 | |
| claude-opus-4-1 | us.anthropic.claude-opus-4-1-20250805-v1:0 | |
| claude-sonnet-4-6 | us.anthropic.claude-sonnet-4-6 | 性价比最优 |
| claude-sonnet-4-5 | us.anthropic.claude-sonnet-4-5-20250929-v1:0 | |
| claude-haiku-4-5 | us.anthropic.claude-haiku-4-5-20251001-v1:0 | 快速/低成本 |

**关键实现文件**：
- `app/services/bedrock.py` — ChatBedrockInvoke LangChain wrapper
- `app/services/agent.py` — `_resolve_model()` 中 `bedrock:` 前缀分支 + THINKING_MODEL_CONFIG
- `app/core/api_config.py` — `get_provider_credentials("bedrock")`
- `app/core/user_api_keys.py` — `bedrock_api_key` / `bedrock_region` 字段
- `config/model_catalog.json` — bedrock provider 的 LLM 模型条目

**model_id 格式**：`bedrock:claude-sonnet-4-6`（前端选择器显示 "Bedrock Sonnet 4.6"）

**注意事项**：
- Opus 4.7 不支持 temperature/top_p/top_k 参数，只支持 adaptive thinking
- 所有模型需要 `us.` geo 前缀的 inference profile ID 才能调用（bare model ID 返回 400）
- 请求体使用 Anthropic Messages API 格式（`anthropic_version: "bedrock-2023-05-31"`）
- bedrock-mantle 端点（`/anthropic/v1/messages`）仅支持 Opus 4.7 + Haiku 4.5；其余模型必须走 bedrock-runtime
- 同一 Bearer Token 还可用于 bedrock-mantle 上的 OpenAI GPT-OSS 模型（`/v1/chat/completions` 路径），暂未集成
- `GET https://bedrock.us-east-1.amazonaws.com/foundation-models` 可列出所有可用模型（125个）

### Fine-grained tool streaming（2026-06-02）

**问题现象**：
1. 写/改文件时前端卡片只显示"正在写入…/等待内容…"，不流式出正在修改的内容。
2. 文件写入耗时过长就断连，已生成内容也不保存。

**根因**：Claude 默认会**缓冲整个工具入参 JSON 做校验后才一次性吐出**（`input_json_delta` 全憋到最后）。导致：
- 前端 `StreamingFilePreview` 整个过程拿不到增量 `partial_json` → 卡片无内容。
- 缓冲期间 SSE 无任何事件流出 → Cloudflare（origin 前置）100s 静默超时切断连接 → 纯工具调用时 `full_response` 为空，`chat.py` finally 兜底保存被跳过 → 内容全丢。

**修复**（`app/services/bedrock.py` `_build_body`）：当请求带 tools 时开启 fine-grained tool streaming：
- body 加 `"anthropic_beta": ["fine-grained-tool-streaming-2025-05-14"]`
- 每个 tool dict 加 `"eager_input_streaming": true`
- 效果：工具入参不缓冲、逐段 `partial_json` 实时流出，卡片实时显示 + SSE 持续有数据避免 CF 超时。
- 注意：fine-grained 模式下流中途 JSON 可能不完整（尤其触顶 `max_tokens` 时），前端 `extractStreamingField` 已能解析部分 JSON；LangChain 累加 tool_call_chunks 末尾再 parse。

**配套**：
- `app/services/agent.py`：非 thinking 的 Bedrock 模型 `max_tokens` 默认 8192 → **16000**，给大文件写入留余量（避免入参中途截断）。
- `app/routes/chat.py` finally 兜底保存：除 `full_response` 外，`blocks` 有流式内容（content/args/result）时也保存，纯工具调用中断不再丢已生成内容。
- 文档：AWS Bedrock `model-parameters-anthropic-claude-messages-tool-use`；Claude API `agents-and-tools/tool-use/fine-grained-tool-streaming`。

## Service Agent 资源范围注入（2026-06-02）

**问题**：发布 Service 时用 `allowed_docs` / `allowed_scripts` 限定了文件/脚本范围，但 Consumer agent 的 system prompt 里完全没告知，导致 agent "迷茫不知道去哪看东西"。

**修复**（`app/services/consumer_agent.py` `_build_consumer_system_prompt`）：在 `consumer_notice` 前追加 `## 可用资源范围` 段，直接写白名单 pattern（不扫描真实文件，轻量）：
- 文档 `allowed_docs`：含 `*` → 告知"可访问全部 /docs/，自行 ls 探索"；非空具体列表 → 逐条列 `/docs/{pattern}`；空 → "未开放任何文档"。
- 脚本 `allowed_scripts`：空（默认）→ 明确"本服务不提供脚本执行能力，请勿尝试运行脚本"；含 `*` → "可执行全部 /scripts/ 脚本"；具体列表 → 逐条列 `/scripts/{pattern}`。
- `service_config` 已携带这两个字段，无需改函数签名。
- 发布/编辑 Service（`app/routes/services.py` POST/PUT）会调 `clear_consumer_cache()`，范围改动后 prompt 自动重建。
- 设计取舍（用户确认）：只写 pattern 不列真实文件；`*` 时不逐一列出让 agent 自己 ls；脚本只列路径不带描述；禁脚本时明确告知。

### Consumer 路径前缀归一化（2026-06-03）

**问题**：发布的 service 访问不到白名单文档/脚本，`ls /docs/salarybenchmark` → "目录不存在"。

**根因**：Consumer 的 `ls`/`read_file` 等读工具根目录**已经是** `filesystem/docs`，`run_script` 的 script_path **相对** `filesystem/scripts`，二者 path 都不该带 `/docs/`、`/scripts/` 前缀；白名单 pattern（FileTreePicker rootPath=`/docs`、`/scripts`）也是相对各自根保存的（不含前缀）。但 agent 按 `/docs/xxx` 心智模型传路径（scope prompt 也这么写），`safe_join(docs_dir, "docs/xxx")` 拼成 `docs_dir/docs/xxx` → 目录不存在；脚本同理拼成 `scripts_dir/scripts/xxx` 且过不了白名单校验。

**修复**（`app/services/consumer_agent.py`）：
- `_create_consumer_read_tools` 加 `_norm_docs_path()`：剥掉开头多余 `docs/` 前缀；5 处 `clean = path.lstrip(...)` 全改用它。
- `_is_allowed` 改写：精确命中或在白名单目录下放行；**额外**让"通往白名单条目的祖先目录"也可列出（`pat.startswith(norm+"/")`），这样 `ls /docs` 能看到白名单文件夹本身、可逐级 ls 进入（旧逻辑用 `norm.startswith(pattern)`，带斜杠 pattern 会把文件夹本身过滤掉）。
- `_create_consumer_script_tools` 加 `_norm_script_path()`：剥掉开头 `scripts/` 前缀；`_script_allowed` 和 `run_script` 实际执行（`consumer_script_execution` + `_run_script_impl`）都用归一化后的路径。
- 容忍式设计：带不带前缀都能用，不破坏既有省略前缀的调用。

**加固（2026-06-03，字面优先、剥前缀兜底）**：处理"docs 根下真有字面叫 docs 的子目录"（或 scripts 根下有 scripts/）的歧义：
- `_norm_docs_path`：路径以 `docs/` 开头时，先看 `docs_dir/<原样路径>` 是否真实存在（即嵌套 docs/ 目录），存在则按字面解析，否则才剥掉开头多余的 docs/ 前缀；`/docs` 命名空间根 token 始终视为 docs 根（返回 ""）。
- `_norm_script_path`：脚本以 `scripts/` 开头时，先看 `scripts_dir/<原样路径>` 是否真实存在且未越界（isfile + realpath 前缀校验），存在则按字面，否则剥前缀。scripts_dir 在工厂层用 `get_user_filesystem_dir(admin_id)/scripts` 取（与 consumer_script_execution 一致）。
- 注意：docs/ 与 scripts/ 是两套**独立根目录**，各自只剥自己的前缀，两边有同名子项不会互相串（互不干涉）。
- 已用真实临时目录单测验证：普通路径剥前缀、嵌套同名目录走字面，两种都解析到真实存在的路径。

**访问模式显式声明（2026-06-03）**：scope prompt 在列白名单路径时必须讲清访问模式：
- 文档：标注"（只读）"，明确"仅可用 read_file / read_document 读取，不可修改/写入/删除"。
- 脚本：标注"（仅执行）"，明确"只能执行，不能读取源码、不能修改/写入"，并给出调用方式 `run_script(script_path=..., 可选 script_args/input_data/timeout)`。
- 原因：脚本能力**没有专门的 capability prompt**（不像 documents/web）。agent 怎么用 run_script 只来自两处：① `run_script` 工具的 docstring（@tool 自动转 schema 绑定给模型）② 这段资源范围列表。所以必须在范围列表里把 execute-only 语义说清楚，否则 agent 可能尝试 read/改脚本源码或写 docs。
- system prompt 拼装顺序（create_consumer_agent 672-692）：_build_consumer_system_prompt（base + 资源范围 + 重要约束）→ 追加 documents capability prompt → 按能力追加 web/scheduler/humanchat。

**改为脚本可只读（2026-06-04）**：纯执行对 prompt 太不友好（agent 不知道脚本怎么用、也看不见），改为允许 service agent **只读** allowed_scripts 白名单内的脚本源码（实际执行仍走 run_script，不可改/写）。
- 实现（用户选"接入现有 read_file/ls"而非新增 read_script）：`_create_consumer_read_tools` 加 `allowed_scripts` 参数 + scripts 命名空间路由。新增 `_norm_scripts_path`（字面优先剥 scripts/ 前缀）、`_is_script_allowed`（allowed_scripts 白名单，空=不可读、支持祖先目录列出）、`_is_scripts_ns`、统一 `_resolve(path)→(full,rel,allowed,ns,err)` 与 `_entry_allowed(rel,ns)`。`ls`/`read_file`/`list_files_sorted`/`read_document`/`view_pdf_page_or_image` 全部改走 `_resolve` 路由：路径以 `scripts/` 开头 → scripts_dir + allowed_scripts；否则 → docs_dir + allowed_docs（默认）。docs/scripts 两套独立根 + 独立白名单，互不影响。
- `read_file` docstring 已注明"也可读 /scripts/ 下白名单脚本源码"，发现性由工具描述承担。
- scope prompt（用户选 keep_min）：仅去掉"不能读取其源码"那句，保留"（仅执行）"标签与 run_script 调用说明（脚本不可修改/写入）。
- 调用点 create_consumer_agent 传 allowed_scripts；allowed_scripts 参数可选（默认 []）不破坏其它调用。
- 已用真实临时目录单测验证：白名单内可 ls+read 源码、白名单外/空白名单拒绝、* 全可读、docs 照常。

## 消费者查看/下载会话 generated 文件（2026-06-04）
service 消费者（终端用户，非 admin）可在 service 聊天页查看并下载本会话 `generated/` 下的文件。
- **决策**：聊天内预览/下载 + 独立文件面板都做；URL 用**短期签名 token**（不暴露 sk-svc- 主 key）；文件端点**校验 conv_id 存在**（404）。
- **后端签名 token**：`app/services/consumer_media_token.py` 用标准库 hmac-sha256 自实现（无新依赖）。token=`base64url(payload).base64url(sig)`，payload 绑定 `(admin_id, service_id, conv_id, exp)`，默认 TTL 6h。密钥优先 env `CONSUMER_MEDIA_SECRET`，否则持久化随机密钥到 `data/.consumer_media_secret`（`data/` 已在 .gitignore）。`create_media_token` / `verify_media_token`（校验签名+过期+绑定）。
- **消费者端点**（`app/routes/consumer.py`）：
  - 新增 `GET /api/v1/conversations/{conv_id}/media-token`（header 鉴权 + conv 存在校验）→ `{token, expires_in}`。
  - `GET .../files`（列表）加 `_assert_consumer_conv_exists` → 404。
  - `GET .../files/{path}`（下载/内联）改为**双鉴权**：`?token=`（供 `<img>`/`<a download>`/`<iframe>` 等无法带 header 的场景，token 必须绑定当前 conv_id 否则 401）**或** `Authorization: Bearer sk-svc-`（API fetch）；加 `download=1`（attachment）；conv 不存在/路径穿越/文件不存在统一 404。
  - 鉴权辅助 `_assert_consumer_conv_exists` 用 `published.get_consumer_conversation`（无 meta.json → None）。
- **存储层**：`base.py::consumer_file_response` 加 `download` 参数，本地分支用 `mimetypes.guess_type` 猜 MIME + `Content-Disposition: inline|attachment; filename*=UTF-8''…`（非 ASCII 文件名）；越界 safe_join 抛 PermissionError/ValueError/FileNotFoundError 由路由转 404。S3 仍 presigned redirect（download 对 S3 无效，cross-origin）。
- **前端**（`frontend/src/service-chat/`）：
  - `serviceApi.ts`：`buildConsumerMediaUrl(token, convId, path, {download})` 改用 `?token=`（不再用 `?key=`）；新增 `getMediaToken(apiKey, convId)`、`listGeneratedFiles(apiKey, convId)`、`GeneratedFile` 类型。
  - `markdown.ts`：新增 `setFileDownloadMode(enabled)`（与 admin 的 `_fileRevealEnabled` 区分）。consumer 开启后：非媒体 `<<FILE:>>` 渲染成 `<a href=mediaUrl download>` 直接下载链接；媒体 caption 的小按钮渲染成「⬇ 下载」而非「📁 定位」。`download` 加入 DOMPurify ADD_ATTR。
  - `ServiceChatApp.tsx`：建会话后调 `getMediaToken` 存 `mediaToken`（ref 给 builder 闭包）；`setMediaUrlBuilder` 用 token；`setFileRevealEnabled(false)` + `setFileDownloadMode(true)`；header 加「📁 文件」按钮开 `GeneratedFilesPanel`；auth 失败重置 token/面板。
  - `GeneratedFilesPanel.tsx`（新）：右侧抽屉，调 list API 列出本会话生成文件，图片显缩略图、其它显类型图标，每条带 `<a download>`（URL 带 token + download=1）。
  - i18n：`zh/en.json` 的 `service.*` 加 `filesBtn/filesTitle/filesRefresh/filesClose/filesLoading/filesEmpty/filesDownload`。
- **测试**：token 单测（签发/校验/篡改/过期/换密钥）+ 端到端单测（直接调 async 路由：媒体 token、列表、404、token inline/attachment 下载、header-key 下载、坏 token/跨会话 token→401、路径穿越/不存在→404、无凭证→401）全过；`npx tsc -b` 0 错。
- **注意**：直接调 async 路由函数测试时，FastAPI 不注入 Query/Header 默认值（默认值是 `Query(None)` 哨兵对象），需显式传 `token=`/`authorization=`。

## 超长 streaming 卡顿优化（增量渲染，2026-06-04）
admin 主页与 service 共用 `StreamingMessage` + `markdown.ts`，流式更新已用 `requestAnimationFrame` 节流（streamContext.tsx / streamHandler.ts，~60fps）。**卡顿根因不是更新频率，而是每帧对那个不断增长的文本块跑完整 `renderMarkdown`（marked + hljs + DOMPurify），内容越长帧数越多 → O(n²)。** 最贵依次：hljs 高亮（尤其 highlightAuto）、DOMPurify、marked。
- **修复（用户选「增量渲染」、admin+service 都修）**：`markdown.ts` 新增 `renderStreamingMarkdown(text)`，把流式文本拆「已稳定前缀 + 未完成尾部」：
  - `_splitStreaming(text)`：扫描行，跟踪代码围栏 ``` / ~~~ 状态；在**不在围栏内**时把最后一个空行（`\n\n`）作为顶层段落边界。返回 `{stable, tail, openFenceLang, fenceCode}`。若当前在未闭合围栏内 → stable 截到围栏起始，tail = 整个未闭合围栏。
  - 稳定前缀走带缓存的 `renderMarkdown`（命中缓存；完整高亮**只在每个块完成那一刻跑一次**）。
  - 未完成尾部 `_renderTail`：若是未闭合代码围栏 → 直接 `_escapeForCode` 输出 `<pre><code>`，跳过 marked/hljs/DOMPurify（内容已转义，安全），让「超长单代码块」也保持线性；否则用 `_renderCore` 渲染（体量小）但关高亮。
  - 高亮开关：模块级 `_highlightEnabled`，`highlightCode` 在 false 时只转义不跑 hljs；`_renderTail` 渲染尾部时临时置 false（try/finally 复位）。
  - `renderMarkdown` 重构出 `_renderCore`（marked+postProcess+DOMPurify，无缓存）共用。
- **StreamingMessage.tsx**：`text` 块仅当 `isLast && isStreaming`（流式尾部）走 `renderStreamingMarkdown`，否则（历史/非最后块/流结束）走 `renderMarkdown`（完整高亮）。流结束 isStreaming=false → 自动对最后块跑完整高质量渲染。
- **StreamingFilePreview.tsx（FilePreviewBody）**：原来每帧对全文件内容跑 `hljs.highlight`（写大文件 O(n²)）。改为 `status === 'streaming'` 时只 `escapeHtml`（不跑 hljs），done/pending/error 时再完整高亮一次。
- `ThinkingBlock` 渲染纯文本（React text node），无 markdown 解析，本就不卡，未改。
- **验证**：`_splitStreaming` 纯函数 6 项单测（段落边界 / 未闭合围栏 / 已闭合围栏归入 stable / 无边界 / 围栏内空行不算边界 / 多段落取最后边界）全过；`npx tsc -b` 0 错。
- **已知取舍**：流式中尾部段落、代码块暂不上色（完成即恢复）；跨段落的 loose list 在流式中可能短暂渲成相邻两个 `<ul>`（完成时整体重渲恢复），均为观感、不影响最终结果。

## service 多会话本地持久化（2026-06-04）
service 消费侧原来每次加载都新建空会话、刷新即丢（还在后端堆空会话）。改造：**本浏览器维度**记住会话列表（localStorage），可管理多个会话、刷新不丢、首次发消息才懒创建。
- **为什么存本地**：service key 共享、后端不按消费者隔离（`list_consumer_conversations` 是 admin 用、会列出该 service 全部消费者会话），不能给消费者列服务端会话，否则串台。会话**内容（消息）仍存后端**，切换时按 conv_id 拉 `GET /api/v1/conversations/{conv_id}`（返回 `{...,messages:[{role,content,tool_calls?,blocks?}]}`，blocks 与流式同结构可直接渲染）。
- **决策（用户选）**：抽屉式 UI（header ☰）、删除仅本地移除（服务端数据保留）、只自动标题（首条用户消息截断 30 字）。
- **serviceApi.ts**：新增 localStorage 存储 `svc_convs_{service_id}` = `{items:ConvMeta[], activeId}`（`loadConvStore`/`saveConvStore`，ConvMeta={id,title,updatedAt}）；新增 `getConversation(apiKey,convId)`（404 返回 null 由前端清理本地记录）+ `ConsumerMessage`/`ConsumerConversation` 类型。
- **ServiceChatApp.tsx**：
  - 新增 `convList`（useState 初值 `loadConvStore().items`）、`drawerOpen` 状态。
  - 消息转换：`backendMsgToEntry`（user→text；assistant→blocks，无 blocks 时从 content+tool_calls 合成）+ `normalizeBlocks`（补默认字段、历史 thinking 默认折叠）+ `makeTitle`。
  - `openConversation(convId,key)`：拉历史→转 entries→setMessages/setConversationId + 刷新 media token；404 从本地列表清掉。
  - `createAndRegister(key,firstText)`：懒创建（首次发消息）→建会话+登记到本地列表（带标题）+刷新 token；handleSend 改调它（原 initConversation 删除）。
  - 抽屉操作：`handleSelectConversation`（abort+reset 当前流→open）、`handleNewConversation`（清空、welcome 复现、conv 置 null 等首次发消息再建）、`handleDeleteConversation`（仅本地 filter）。
  - onDone：本轮结束把当前会话顶到列表最前 + 更新 updatedAt。
  - **持久化 effect 的关键坑**：persist effect 声明在 restore effect 之前，挂载时会先用 conversationId=null 写盘，把 localStorage 里的 activeId 清掉导致刷新无法恢复。解法：`didInitRef` 守卫——首屏恢复完成前 persist 不写盘；restore effect 读 store→标记 didInit→开 activeId 会话。handleAuthFail 重置 didInitRef=false 以便重登恢复（会话列表本身保留）。
- **ConversationDrawer.tsx（新）**：左侧滑出抽屉，列出会话（标题+时间+删除×）、「＋新建对话」、点击切换；CSS 加 `.menuBtn/.convOverlay/.convDrawer/.convRow*` 等。i18n：zh/en 加 `service.conv*`。
- **验证**：`npx tsc -b` 0 错、locale JSON 合法、lint 干净。

## LiveKit 实时语音插件（额外模块，2026-06-09 起）
把实时语音做成 JellyfishBot 之上的**解耦插件**（「上层对话框架」）。锁定选型：①桥接走**路1 远程 SSE 解耦**（Worker 不内嵌 agent，经 Core HTTP/SSE 远程驱动任务引擎，任务跑在 Core 进程→「对话/任务分进程互不干扰」）；②一期只接 admin 主界面；③SFU 自托管 LiveKit Server；④前台用**独立快速 LLM**（闲聊直答 vs 委派路由）+ 调试/Prompt 调音台。选 LiveKit 而非 Pipecat：原生多人 room（场景=边打电话边改文档的 peer-coding、多人会议室 copilot 何时沉默都靠它的 per-participant 音轨 + diarization）。官方有 `langchain_deepagent` 示例（deepagents 是本项目底座）佐证可行。
- **进程拓扑**：浏览器 ─WebRTC→ LiveKit Server(SFU) ←音轨─ 语音 Worker(插件，独立进程) ─桥接令牌→ Core `/api/voice/live/{session,delegate}` ─→ JellyfishBot agent(`/api/chat` 同源任务引擎)。多 worker/多区域时加 Redis（已在 compose 预埋 `profile: scale`，默认不启）。
- **桥接令牌（Core↔Worker 窄契约）**：`app/voice/live_token.py`，纯标准库（不引入 livekit-api，保持 Core 解耦）。① `mint_livekit_token` 签 LiveKit 接入 JWT(HS256)，**把桥接令牌塞进参与者 `metadata` 声明**；② `create_bridge_token/verify_bridge_token`（HMAC-SHA256，绑定 admin_id/conv_id/model/caps，6h TTL）。Worker 加入 room 后从参与者 metadata 读出桥接令牌回调 Core——**Worker 永不接触管理员真实登录 token**，改 admin_id/conv_id 会因验签失败被拒。**桥接令牌由 Core 签发+校验，Worker 只透传不签不验，故 `VOICE_BRIDGE_SECRET` 仅 Core 需要**（或落盘 `data/.voice_bridge_secret`）。另注意：`LIVEKIT_API_KEY/SECRET` 是 LiveKit 自有鉴权，需 Core/Worker/SFU 三方一致（dev 模式 devkey/secret），与桥接密钥是两套独立系统。
- **Core 路由** `app/routes/voice_live.py`（已注册进 main.py）：
  - 管理员侧（`get_current_user`）：`GET /status`（LiveKit 是否配置+公网 url）、`POST /token`（room=`jf-<admin_id>-<conv_id>`，签接入令牌）、`GET/PUT/DELETE /config`（前台配置 CRUD）。
  - Worker 侧（依赖 `get_voice_worker_session` 校验 `X-Bridge-Token`）：`GET /session`（一次拿全：标识+前台配置+供应商凭据）、`POST /delegate`（**复用 chat.py 的 `_create_user_agent_bounded`+`_stream_agent`，`yolo=True` 不弹审批，thread_id=`{admin_id}-{conv_id}` 与文字对话共享上下文，任务轮次持久化进同一 conversation**）。供应商凭据 `_provider_creds` 走 `api_config`（STT/TTS 用 capability 优先逻辑，LLM 用 openai 通用配置）。
- **前台配置存储** `app/services/voice_agent_config.py`：`users/<id>/voice_agent_config.json`，字段 enabled/greeting/system_prompt/routing_policy/fillers{delegating,tool_running,long_task}/interruption{allow_interruptions,min_interruption_words}/providers{stt,stt_model,llm_model,tts,tts_voice}。深合并默认值；改完下一通通话即时生效（Worker 启动时拉取，无需重部署）。
- **Worker 插件包** `plugins/voice/`（独立 pyproject/Dockerfile，依赖 livekit-agents 1.x + plugins-openai/silero/turn-detector）：`agent.py`（AgentServer 入口：connect→`wait_for_participant` 读 metadata→`fetch_session`→装配 STT/LLM/TTS→`AgentSession.start`→开场白）、`config.py`（`fetch_session` 引导）、`bridge.py`（`stream_delegate` 消费 Core SSE 事件词汇 token/tool_call/subagent_*/done/error；网络中断转成 error 事件）、`copilot.py`（前台 Agent + `delegate_to_jellyfish` function_tool 闭包：先说承接填充语→流式委派→关键节点节流播报进度→最终答案 return 给前台 LLM 朗读总结）、`orchestration.py`（**刻意不依赖 LiveKit、可移植**：填充语随机不重复、进度播报冷却节流、打断记忆 last_instruction/partial_answer、多人沉默 `should_agent_speak` 雏形）。
- **前端**：`services/api.ts` 加 voice-live 客户端（getVoiceLiveStatus/Token、get/update/resetVoiceAgentConfig + 类型）；`components/VoiceCallModal.tsx`（`livekit-client`：取 token→`Room.connect`→开麦→`TrackSubscribed` 挂 `<audio>` 播 agent 音、`ActiveSpeakersChanged` 显示「助手讲话中」、静音/挂断）；`pages/Settings/VoicePage.tsx`（调音台：配置编辑+连接状态+「试通话」）；路由 `/settings/voice` + 设置侧栏 `Microphone` 项 + i18n `settings.voice`。依赖加 `livekit-client@^2.6`。
- **部署**：`docker-compose.voice.yml`（叠加层，`-f docker-compose.yml -f docker-compose.voice.yml up`）含 livekit-server（dev 模式 devkey/secret）、voice-worker（build plugins/voice）、redis（profile scale）。`.env.example` 加 `LIVEKIT_URL/API_KEY/API_SECRET/VOICE_BRIDGE_SECRET`。
- **为未来准备的伏笔**：room 名编码身份（多租户/将来 SIP 电话进线复用）、委派接口将来可参数化 target=admin|service（语音版客服=第4分发渠道）、orchestration 独立模块（换框架不重写大脑）、STT 选型预留 diarization（多人 room）、语音轮次进同一 conversation（语音/文字无缝切换）。
- **验证**：`npx tsc -b` 0 错；新后端模块导入 OK、桥接令牌 roundtrip OK、7 个 voice-live 路由注册正确。（注：本机 `import app.main` 因缺 `langgraph.checkpoint.sqlite` 失败，是环境缺依赖、与本改动无关。）
- **待办/下一步**：Chat 主页加发起通话入口（目前仅调音台「试通话」）；真实 LiveKit 环境端到端联调（需起 livekit-server + worker）；后续多人 room/diarization、SIP、service 端语音。

## LiveKit EC2/NAT 连通 + Fish Audio TTS（2026-06-10）
- **`ws://localhost:7880` 连接失败根因**：`LIVEKIT_URL` 是发给**浏览器**去连 SFU 的地址，浏览器在本地，`localhost` 指本地电脑而非 EC2。修法：`.env` 里 `LIVEKIT_URL` 必须是浏览器可达的公网地址——http 访问用 `ws://<EC2公网IP>:7880`；https 域名访问必须用 `wss://<域名>/livekit`（经 nginx 反代到 7880，否则浏览器拦混合内容）。Core 经 `env_file: ./.env` 读取，改完 `docker compose up -d` 重启 Core 即生效（token 里嵌的 url 来自该值）。
- **EC2/NAT 媒体连不通根因**：`--dev` 模式会广播私网 IP（172.31.x.x）作 ICE candidate，浏览器直连不到→信令连上也没声音。修法：新增 `livekit/livekit.yaml` 设 `rtc.use_external_ip: true`（经 STUN 自动发现公网 IP），`docker-compose.voice.yml` 的 livekit-server 改用 `--config /etc/livekit.yaml`（挂载该文件），不再用 `--dev`。**安全组入站必放行**：TCP 7880/7881、UDP 50000-50100；出站需能到公网 UDP 3478 做 STUN 自检（自检失败可改 `node_ip: <公网IP>` 显式指定）。keys 仍 devkey/secret，需与 Core/Worker 一致。
- **Fish Audio TTS 集成**（官方插件 `livekit-plugins-fishaudio`，参照 LiUshin/ttsRealTalk）：
  - 选型确认：LiveKit 有官方 fishaudio 插件 → `fishaudio.TTS(model="s1"|"s2-pro", reference_id="音色ID", api_key=..., latency_mode="balanced")`，env `FISH_API_KEY`，音色用 `reference_id`（不是 voice）。
  - `plugins/voice/pyproject.toml` 加依赖；`agent.py` 抽 `_build_tts(providers,pcfg)` 按 `pcfg["tts"]` 选 `openai|fishaudio`，fish 初始化失败兜底回退 openai。`_openai_tts` 补 `model` 参数（默认 gpt-4o-mini-tts）；`_openai_stt` 默认模型改 `gpt-4o-mini-transcribe`（流式更省）。
  - `voice_agent_config.py` providers 加字段：`tts`(openai|fishaudio)、`tts_model`、`fish_reference_id`（深合并对老配置可见）。
  - Core `_provider_creds` 加 `out["fish"]`：从**全局环境变量** `FISH_API_KEY`(/`FISH_AUDIO_API_KEY`)、`FISH_REFERENCE_ID`(/`FISH_AUDIO_VOICE_ID`)、`FISH_MODEL` 读取（用户明确要求放 env，非 per-user UI key）；未设则 `None`。worker 拿 creds 里的 api_key 传给插件，docker-compose.voice.yml 也给 worker 加了 FISH_* env 兜底。
  - 前端 `VoicePage.tsx` 调音台加：TTS 供应商下拉(OpenAI/Fish Audio)、TTS 模型输入、选 fishaudio 时显示「Fish 音色 ID」输入并提示需配 `FISH_API_KEY`；`api.ts` 类型补 `tts_model?`/`fish_reference_id?`。
  - `.env.example` 加 Fish 段 + 改写 LIVEKIT_URL 注释强调勿用 localhost。

## 语音前台多供应商:STT(Fish/阿里) + LLM(Bedrock) 可选化(2026-06-13)
- **EC2 部署关键坑回顾**(已验证跑通):① `LIVEKIT_URL` 是发给浏览器的地址,localhost=浏览器本机,域名 https 必须 `wss://<域名>/livekit` 经 nginx 反代;② `--dev` 在 NAT 后广播私网 IP,改用 `livekit/livekit.yaml` 的 `rtc.use_external_ip:true`(STUN 自查公网 IP,日志 `found external IP via STUN`),安全组放行 UDP 50000-50100 + TCP 7881(媒体直连,绕过 Cloudflare,因 CF 只代理 HTTP/WS 不代理 UDP);③ 信令可走域名(nginx `/livekit/` 块 + `map $http_upgrade $connection_upgrade`)。④ 多机部署差异:橙云那台容器 nginx 占 `0.0.0.0:80`;有宿主机系统 nginx 自管 TLS 的那台,容器 nginx 要映射 `127.0.0.1:8080`(否则撞 80)。
- **三类 reasoning 通道**(排查 Bedrock thinking 用):chat.py 里 ①结构化 `{"type":"thinking"}` block→`thinking` 事件(Anthropic/我们的 Bedrock wrapper,需开 `-thinking` 变体);②`additional_kwargs.reasoning_content`→`thinking` 事件(OpenAI o 系/DeepSeek);③普通文本→`type:token`。看到 `<thinking>` 裹在 token 里 = 没开扩展思考,模型把推理当正文吐(provider 无关)。Bedrock wrapper `_anthropic_event_to_chunk` 已把 `thinking_delta` 映射成结构化 block。
- **供应商选型结论**(面向中国大陆+支付宝):Fish Audio STT(`transcribe-1`,$0.36/h)是**批量**(`POST /v1/asr`,multipart 字段 `audio`,无流式 WS),延迟高但够用;阿里 Paraformer 是**流式**且端点在国内,更优。LiveKit 有现成国内插件:`livekit-plugins-aliyun`(Paraformer/CosyVoice/Qwen,`DASHSCOPE_API_KEY`)、`volcengine`、`tencent`、`xunfei`。
- **STT 可选化**:worker `agent.py` 的 `_build_stt(providers,pcfg,vad)` 按 `pcfg["stt"]` 选 `openai|fishaudio|aliyun`:
  - fishaudio → 自写 `fish_stt.py`(`stt.STT` 非流式 `_recognize_impl`,`rtc.combine_audio_frames(buffer).to_wav_bytes()` → POST /v1/asr),外面包 `stt.StreamAdapter(stt=base, vad=vad)` 用 VAD 断句。
  - aliyun → `aliyun.STT(model="paraformer-realtime-v2")`,流式,直接用;key 经 `os.environ.setdefault("DASHSCOPE_API_KEY", ...)` 注入(插件读 env)。
- **前台 LLM 可选化**:`_build_llm(providers,pcfg)` 按 `pcfg["llm"]` 选 `openai|bedrock`。**Bedrock 复用现有 Bearer API key 走 OpenAI 兼容端点**(无需自写 LLM 适配器):`openai.LLM(base_url=f"https://bedrock-runtime.{region}.amazonaws.com/openai/v1", api_key=<bedrock key>, model=<profile id>)`。短名→profile 用 `_bedrock_model_id`(claude-sonnet-4-6 → us.anthropic.claude-sonnet-4-6)。AWS 文档 model-parameters-openai 确认 bedrock-runtime `/openai/v1/chat/completions` 接受 `Authorization: Bearer $AWS_BEARER_TOKEN_BEDROCK`。
- **Core `_provider_creds`** 新增 `out["bedrock"]={api_key,region}`(`get_provider_credentials("bedrock",user_id)`,复用主 agent 的 key)、`out["dashscope"]={api_key}`(全局 env `DASHSCOPE_API_KEY`)、fish 加 `base_url`。
- **配置/前端**:`voice_agent_config` providers 加 `llm`(openai|bedrock)、`aliyun_vocabulary_id`;`VoicePage` 调音台加「前台 LLM 供应商」「LLM 模型」下拉/输入、STT 下拉加「阿里云 Paraformer」、按选择显示 env 提示;`api.ts` 类型补 `llm?`/`aliyun_vocabulary_id?`。`pyproject` 加 `livekit-plugins-aliyun`。`.env.example` 加 `DASHSCOPE_API_KEY` + Bedrock 说明。
- **改完即时生效**:都是 worker 启动时从 `/session` 拉配置装配,改调音台→下一通通话生效,无需重部署 worker(但新增依赖 `livekit-plugins-aliyun` 需重建 voice-worker 镜像)。
- **阿里云 CosyVoice TTS**(2026-06-13 追加):`_build_tts` 加 `aliyun` 分支 → `aliyun.TTS(model="cosyvoice-v2", voice=cfg.aliyun_tts_voice or "longcheng_v2")`,与 Paraformer 共用 `DASHSCOPE_API_KEY`(env 注入)。config providers 加 `aliyun_tts_voice`(默认 longcheng_v2);VoicePage TTS 供应商下拉加「阿里云 CosyVoice」,选中显示「CosyVoice 音色」输入;`api.ts` 加 `aliyun_tts_voice?`。至此语音前台 STT=openai|fishaudio|aliyun、LLM=openai|bedrock、TTS=openai|fishaudio|aliyun 全可在调音台切换。
- **Fish 音色库 + CosyVoice 下拉**(2026-06-14):
  - **关键设计:worker 零改动**。worker 仍只读单个 `fish_reference_id`(即「当前音色」),音色库纯属 Core 配置/前端层。`fish_voices: [{id,label}]` 是收藏夹,选其一把它的 id 写进 `fish_reference_id`。
  - config providers 加 `fish_voices`(默认 `[]`)。注意 `_deep_merge` 对**列表是整体替换**(非 dict 才 `out[k]=v`),前端每次提交完整 `providers`(含完整数组)故正确持久化。
  - `VoicePage`:选 fishaudio 时「当前音色」改 `Select`(options 来自 `fish_voices`,allowClear;当前 id 不在库里则置顶显示);下方「Fish 音色库」管理区——已存音色渲染成 `Tag`(蓝色✓=当前,点击设为当前,× 删除),新增表单(标签 Input + reference_id Input + 添加按钮,本地 state `newFishLabel/newFishId`,重复 id/空 id 拦截)。删当前音色会清空 `fish_reference_id`。
  - CosyVoice 音色 `Input` 改 `AutoComplete`(可下拉建议 + 可手填任意音色参数):`COSYVOICE_VOICES` 收录 10 个常用 v2 音色(longxiaochun_v2/longcheng_v2/longwan_v2 等,带中文名标签)。CosyVoice v2 实际有几十个音色(longxxx_v2 中文 + loongxxx_v2 多语),全列表见 help.aliyun.com/zh/model-studio/cosyvoice-voice-list,故用可编辑下拉而非固定枚举。
  - `api.ts` providers 加 `fish_voices?: Array<{id,label}>`。改完下一通通话生效(worker 拉 `/session` 配置),无需重建镜像(无新依赖)。

## 语音前台 LLM 下拉化 + 两处崩溃修复(2026-06-14)
- **`model identifier is invalid`(trace 落在 `livekit/agents/inference/llm.py`)根因**:`livekit.plugins.openai.LLM` 在 **api_key 为空时会自动改走 LiveKit Cloud 推理网关**,把你的裸模型名当网关模型 → 报「模型标识无效」。用户 OpenAI key 坏了+语音前台 LLM=openai → 必现。**铁律:openai.LLM 必须有非空 api_key,否则别构造**。
- **LLM 下拉化(沿用主 Chat 模型表)**:语音前台实时 LLM 用的是 OpenAI 兼容的 `openai.LLM`(不是主 agent 的 `_resolve_model`/`ChatBedrockInvoke`),所以**只能跑 OpenAI 兼容的 provider**:`openai`/`kimi`/`bedrock`(走 Bedrock 的 `/openai/v1` 端点);anthropic 原生、minimax(Anthropic 兼容)worker 暂不支持(要装 livekit-plugins-anthropic)。
  - 前端 `VoicePage`:删掉「LLM 供应商 Select + 自由文本 Input」,改成单个 `Select`(showSearch),选项来自 `GET /api/models`(`api.getModels()`)过滤 `provider∈{openai,kimi,bedrock}`,**存完整 catalog id(`provider:model`,如 `bedrock:claude-sonnet-4-6`)到 `llm_model`**,同时把 `llm` 设为前缀(向后兼容)。当前 id 不在列表则置顶显示;空列表给 `notFoundContent` 提示去配凭据。`/api/models` 只返回有凭据且未隐藏的模型——没 key 的 provider 整组不出现。
  - worker `_build_llm`:`llm_model` **含冒号 → 按前缀路由**(`_LLM_BUILDERS={openai,kimi,bedrock}`);**无冒号 → 旧格式**(`llm` 供应商字段 + 裸名)向后兼容。各 builder(`_openai_llm/_kimi_llm/_bedrock_llm`)签名改为 `(providers, model)`,**缺 key 直接 raise**(不再静默回退 openai 空 key)。`entrypoint` 包 try:LLM 装配失败时打清晰日志后 return(不抛诡异网关错)。
  - Core `_provider_creds` 加 `out["kimi"]={api_key,base_url}`(`get_provider_credentials("kimi")`,base 默认 `https://api.moonshot.cn/v1`)。bedrock/openai 已有。
- **STT `end_time > 0` 报 `NoneType` 崩溃**:LiveKit `_on_stt_event` 对 `ev.alternatives[0].end_time` 做 `>0` 比较,某些 STT 插件(如 aliyun Paraformer)发出 `start_time/end_time=None` 的 `SpeechData` → `TypeError` 崩溃整个 session。**provider 无关的兜底**:copilot 改成 `_CopilotAgent(Agent)` 子类,覆盖官方钩子 `stt_node`(`async for ev in Agent.default.stt_node(self, audio, model_settings)` 里把 None 时间戳改 0.0 再 yield——这是 LiveKit 文档 docs.livekit.io/agents/build/nodes 的标准覆盖法,`Agent.default.stt_node` 是 staticmethod async gen)。另把自写 `fish_stt.py` 的 SpeechData 显式给 `start_time=0.0/end_time=duration`。
- **生效方式**:LLM 路由/STT 兜底都在 worker 代码里 → **需重建 voice-worker 镜像**(`docker compose -f docker-compose.yml -f docker-compose.voice.yml build voice-worker && up -d`);前端改动需重建前端。配置(选哪个模型)改完下一通通话即生效。
- **下拉与 Chat 完全一致(2026-06-14 追加)**:用户要求语音前台 LLM 下拉 = Chat 全量。catalog 的 5 个 LLM provider:openai/anthropic/kimi/minimax/bedrock,全部接进 worker。
  - `pyproject` 加 `livekit-plugins-anthropic>=1.2,<2.0`(自带 `anthropic` SDK)。`anthropic.LLM(model,api_key,base_url,...)` **支持 base_url** → minimax(Anthropic 兼容端点 `https://api.minimax.io/anthropic`,LLM 只需 api_key,group_id 仅 TTS/Video 用)复用该插件。
  - worker 新增 `_anthropic_llm`(原生,base_url 仅用户配自定义网关时传——anthropic 插件 base_url 是 NotGivenOr,别传 None)、`_minimax_llm`(固定 minimax anthropic base_url);`_LLM_BUILDERS` 扩成 5 个。
  - **thinking 变体处理**:该 anthropic 插件版本**不暴露 extended thinking 参数**,且实时语音要低延迟 → `-thinking` 一律取 base 模型(`_THINKING_BASE` 照抄主 agent THINKING_MODEL_CONFIG 的 base,注意 haiku/sonnet-4-5 的 base 带日期后缀如 claude-haiku-4-5-20251001);bedrock 路径先 `_resolve_thinking` 再 `_bedrock_model_id` 套 profile。
  - anthropic/bedrock 的裸模型名直接沿用 catalog 冒号后的名字(与主 agent init_chat_model/ChatBedrockInvoke 传的一致),保证行为同步。
  - Core `_provider_creds` 再加 `out["anthropic"]={api_key,base_url}`、`out["minimax"]={api_key, base_url=固定 anthropic 端点}`。
  - 前端去掉 provider 过滤,`llmModels = modelsRes.models`(全量);提示文案改「与 Chat 完全一致(OpenAI/Anthropic/Kimi/MiniMax/Bedrock)+ thinking 型号按基础模型跑」。
  - 仍需重建 voice-worker 镜像(新增 anthropic 插件依赖)+ 重建前端。
- **⚠️ 纠错:堆栈出现 `livekit/agents/inference/llm.py` 不代表跑旧代码!** `livekit-plugins-openai` 的 `LLMStream` **继承自** `livekit.agents.inference.llm.LLMStream`(openai 插件 llm.py 顶部 `from livekit.agents.inference.llm import LLMStream as _LLMStream`,真正 `_run`/抛 APIStatusError 在基类),所以 `openai.LLM`(含我们 bedrock/kimi 走的 openai 兼容路径)**正常就会**出现 `inference/llm.py` 堆栈。anthropic.LLM 则走 `livekit/plugins/anthropic/llm.py`。
- **404 `model_not_found`(`type:not_found_error`)= 真实错误:所选 model id 被对应端点拒绝**(模型名不存在/区域/账号没启用),不是网关问题、也不是旧代码。常见:① OpenAI 选了账号没有的型号(catalog 里 gpt-5.x 等未来名);② Bedrock inference profile 前缀/区域不匹配——`_BEDROCK_LLM_PROFILE` 硬编码 `us.` 前缀,若 bedrock 区域是 `ap-southeast-1`(新加坡,语音机公网 IP 13.212.84.160 即 ap-southeast-1)则应是 `apac.anthropic.xxx`,否则 404。排查靠 `_build_llm`/各 builder 已加的 `logger.info("前台 LLM=... model=... base_url=...")`,看日志确认实际发出的 model+base_url+region。
- 验证容器内是否新代码:`docker exec jellyfishbot-voice-worker python -c "import jellyfish_voice.agent as a; print(list(a._LLM_BUILDERS))"` 应输出 5 个 provider;改 worker 源码后必须重建镜像(`build` 非挂载卷):`build --no-cache voice-worker` + `up -d --force-recreate voice-worker`。

## 语音前台 LLM:Core OAI 网关重构(2026-06-15,取代上面 worker 内 per-provider 路由)
- **动机**:worker 内各自对接 openai/anthropic/bedrock 太脆(404/区域 profile/插件 thinking 不一致,且 bedrock 兼容端点行为与主 Chat 的 ChatBedrockInvoke 不同源)。用户明确:**用 jellyfishbot 自己的兼容 OAI 端点,给 LiveKit 发一个专门的 service**。结论=**Core 发布一个 OpenAI 兼容 `/chat/completions` 网关,worker 只用一个普通 `openai.LLM` 指过去**。模型选择/凭据/Bedrock Invoke 全在 Core 一处复用 `_resolve_model`,与主 Chat 100% 同源;worker 仍是原生 `openai.LLM` → LiveKit **原生 function_tool / 填充语 / 打断全保住**(绕开 LangChain `LLMAdapter` 的工具/旁白坑)。
- **Core 网关** `app/routes/voice_live.py`(C 段,挂在现有 router 上,路径 `POST /api/voice/live/llm/v1/chat/completions`):
  - 鉴权 `get_voice_llm_user`:读 `Authorization: Bearer <桥接令牌>`(openai 客户端固定把 api_key 放 Bearer),`verify_bridge_token` 校验 → admin_id。**不是 `X-Bridge-Token`**(那是 /session、/delegate 用的)。
  - 解析:`_resolve_model(model_id, user_id=admin_id)`(model_id 即 worker 发来的 catalog id,如 `bedrock:claude-sonnet-4-6`);返回 str(无显式 key 的默认路径)则 `init_chat_model(model=str)` 包一层;有 `tools`(OAI 格式)→ `model.bind_tools(tools)`(**ChatBedrockInvoke.bind_tools 与 init_chat_model 模型都直接吃 OAI tool dict**,无需转换)。
  - thinking 变体:实时语音不开扩展思考,`model_id` 命中 `THINKING_MODEL_CONFIG` 则用其 `base_model` 解析(同一张表,base 含 provider 前缀)。⚠️ `openai:gpt-5.4` 的 base 仍是自己(带 reasoning params),用户选它就会有 reasoning,属其显式选择。
  - 流式:`async for ch in model.astream(lc_messages)` → OpenAI `chat.completion.chunk` SSE。`_extract_text` 取纯文本(content 可能是 str 或 anthropic/bedrock 的 list block,忽略 thinking/tool_use 块);`ch.tool_call_chunks`(`{name,args,id,index}`,首帧带 name+id 后续只带 args)→ OAI `delta.tool_calls[{index,id,type,function:{name,arguments}}]`。首帧发 `{role:assistant}`,末帧发空 delta + `finish_reason=tool_calls|stop`,再 `data: [DONE]`。
  - 消息转换 `_oai_to_lc_messages`:system/user/assistant(含 tool_calls,arguments JSON 字符串 → args dict)/tool(tool_call_id)→ LangChain SystemMessage/HumanMessage/AIMessage/ToolMessage。续轮(工具执行后)LiveKit 会带 assistant.tool_calls + tool 结果消息回来,故必须支持这两类。
- **worker** `agent.py`:`_build_llm(provider_cfg, bridge_token)` **塌缩成单一**:`openai.LLM(model=<llm_model catalog id>, api_key=<桥接令牌>, base_url=f"{api_base()}/api/voice/live/llm/v1")`。**删掉**全部 `_openai_llm/_kimi_llm/_anthropic_llm/_minimax_llm/_bedrock_llm/_LLM_BUILDERS/_THINKING_BASE/_resolve_thinking/_BEDROCK_LLM_PROFILE`。`entrypoint` 改 `_build_llm(pcfg, sess.bridge_token)`。`config.py` 已有 `api_base()` 与 `VoiceSession.bridge_token`。
- **依赖**:`pyproject.toml` **删 `livekit-plugins-anthropic`**(LLM 不再在 worker 分供应商;STT/TTS 仍用 openai 系)。Core `_provider_creds` 的 llm/kimi/anthropic/minimax/bedrock 凭据**保留**(worker 已不用于 LLM,但 `_openai_stt/_openai_tts` 仍回退 `providers["llm"]` 取 OpenAI key)。
- **生效**:Core 改了路由 → 重启 Core(`up -d jellyfishbot`);worker 改了 `_build_llm`+删依赖 → 重建 voice-worker 镜像。前端 `VoicePage` LLM 下拉**无需改**(仍存 catalog id 到 `llm_model`,网关原样吃)。
- **前提**:worker→Core 走内网 `JELLYFISH_API_BASE`(如 `http://jellyfishbot:8000`),网关端点是内网调用,不经公网/nginx。openai.LLM 同时给了 api_key+base_url → 走标准 OAI client,**不会**回退 LiveKit 推理网关。
- **环境核对(2026-06-15)**:`docker-compose.voice.yml` voice-worker 已配 `JELLYFISH_API_BASE=http://jellyfishbot:8000`;Core 服务名=`jellyfishbot`、FastAPI 监听 8000(healthcheck `localhost:8000/docs`);主/voice compose 均未定义自定义网络 → 同项目默认网络,服务名可解析。本地裸跑用 `plugins/voice/.env.example` 的 `JELLYFISH_API_BASE=http://localhost:8000`。

## 语音前台 TTS/STT 模型名跨供应商串味 → OpenAI 404(2026-06-15)
- **现象**:`openai/tts.py _run` 抛 `404 The model 's2-pro' does not exist`(s2-pro 是 Fish 模型,却发给了 OpenAI TTS)。
- **根因**:`providers.tts_model` / `stt_model` 是**跨供应商共享的自由输入字段**,前端切换供应商时**不重置**。两条路径都会把外来模型名喂给 OpenAI:① provider 仍是 openai 但残留 fish/aliyun 模型名;② provider=fishaudio/aliyun 但初始化失败(如 **FISH_API_KEY 未配**)→ `_build_tts` 回退 `_openai_tts` 时把 `s2-pro` 一并带过去。
- **修复**:
  - worker `_openai_tts`/`_openai_stt` 加**白名单防御**:`_OPENAI_TTS_MODELS={gpt-4o-mini-tts,tts-1,tts-1-hd}`、`_OPENAI_STT_MODELS={gpt-4o-mini-transcribe,gpt-4o-transcribe,whisper-1}`;收到非白名单模型名 → 打 warning 并改用各自默认。直接消除 404 崩溃(无论哪条路径)。
  - 前端 `VoicePage` TTS/STT 供应商 `Select` 的 onChange 改成**切换时重置对应 `tts_model`/`stt_model` 到该供应商默认**(openai→gpt-4o-mini-tts/gpt-4o-mini-transcribe、fishaudio→s1/空、aliyun→cosyvoice-v2/paraformer-realtime-v2)。根治串味。
- **⚠️ 注意**:若实际意图是 Fish/阿里云 TTS 却落到 OpenAI,说明该供应商**初始化失败回退**了 —— 必看 worker 日志里紧邻的 `"Fish Audio TTS 初始化失败,回退 OpenAI TTS: <reason>"`,最常见是 **`FISH_API_KEY` 未在服务端 .env / docker-compose.voice.yml 配置**。防御只防崩溃,不替代配齐凭据。
- **生效**:worker 改 `_openai_tts/_openai_stt` 需重建 voice-worker 镜像;前端 reset 逻辑需重建前端。已存的坏配置无需手动改(worker 防御会兜底成默认模型)。

## ⚠️ fishaudio 插件 `api_key=None` defeats 环境变量回退(2026-06-15,Fish TTS 真正回退根因)
- **现象**:TTS provider 明明选了 fishaudio,却仍报 OpenAI 404 → 说明 `_fish_tts` 构造抛异常被 `_build_tts` 回退到 OpenAI。
- **根因**:`livekit.plugins.fishaudio.TTS` 签名 `api_key: NotGivenOr[str] = NOT_GIVEN`——**没传**才会回退读 `FISH_API_KEY` 环境变量。我们旧代码 `kwargs["api_key"] = creds.get("api_key") or None` 在 Core 没下发 fish 凭据(`out["fish"]=None`)时**显式传了 `api_key=None`**;LiveKit `is_given(None)` 为真 → 插件认为"已显式给定 None" → **不再读环境变量** → 抛 "api_key required" → 回退 OpenAI。即便 worker 容器本身有 `FISH_API_KEY` env(docker-compose.voice.yml 传了)也被这条挡掉。
- **关键规律**:**LiveKit 插件凡是 `NotGivenOr` 参数,缺值时一律"省略"而非传 None**。`model="s2-pro"` 不会报错(Python 不在运行时校验 Literal 注解,且 s2-pro 是 Fish 合法 backend)。
- **修复**:`_fish_tts` 只在 `creds.get("api_key")`/`base_url` 真有值时才放进 kwargs,否则省略 → 插件自己读 `FISH_API_KEY`/`FISH_BASE_URL` 环境变量。这样 Core 下发凭据 **或** worker 容器 env 任一有 key 即可工作。(自写的 `fish_stt.py` 无此问题——它 api_key=None 时主动读 env。)
- **配置前提**:`FISH_API_KEY` 在 `.env.example` 默认是**注释掉的**(`# FISH_API_KEY=`)。用户必须在实际 `.env` 取消注释并填值——docker-compose.voice.yml 的 worker `FISH_API_KEY=${FISH_API_KEY:-}` 和 Core `env_file: ./.env` 都从这里取。没配则 Fish 无论如何起不来(只能用 OpenAI/阿里)。
- **生效**:改 `_fish_tts` 需重建 voice-worker 镜像。
- **真实根因(2026-06-15,日志实锤)**:`Fish Audio TTS 初始化失败: TTS.__init__() got an unexpected keyword argument 'reference_id'`。即 FISH_API_KEY 已读到,但**安装的 fishaudio 插件版本(`>=1.2,<2.0` 解析到的某 1.2.x)的 TTS 构造签名不含 `reference_id`**(官方最新文档有,该旧版没有或叫别的)。修复:`_fish_tts` 用 `inspect.signature(fishaudio.TTS.__init__).parameters` 取实际接受的参数,desired kwargs(model/latency_mode/api_key/base_url + 音色键按 reference_id→voice→voice_id 择一)过滤后再构造,并 `logger.info` 打印实际签名。**通用规律:对第三方插件构造,按 inspect 签名过滤 kwargs 比硬编码稳。**
- **次要非致命**:`turn_detector` 报 `Could not find file languages.json`——Dockerfile 的 `download-files || true` 把下载失败吞了,模型没进镜像。运行时回退 VAD,功能正常只刷 ERROR。后续可去掉 `|| true` 或运行时补下载。
- **后续(2026-06-15):取消显式供应商的静默回退**。现象升级:堆栈仍落 `openai/tts.py` 但变 429 `insufficient_quota`——说明 Fish 仍在回退 OpenAI,而 OpenAI key 余额耗尽。**静默回退一直在掩盖 Fish 真实原因**,用户连续看到误导性 OpenAI 报错。改法:`_build_stt`/`_build_tts` 对**显式选择**的 fishaudio/aliyun 初始化失败时 `logger.exception` 打完整堆栈 + `raise RuntimeError("...初始化失败: <真实原因>(检查 FISH_API_KEY/DASHSCOPE_API_KEY)")`,**不再回退 OpenAI**;只有 vendor=openai(默认)才用 OpenAI。entrypoint 把 STT/LLM/TTS 装配统一包进一个 try,失败打清晰日志后 return(优雅退出,不抛裸 traceback)。这样下次失败时**第一条错误就是 Fish 的真实原因**,不再被 OpenAI 的 404/429 带偏。

## ⭐ 甩掉 livekit-plugins-aliyun:自迁移 Paraformer STT + CosyVoice TTS(2026-06-15)
- **根因(依赖死锁)**:`livekit-plugins-aliyun==1.3.0` 在 metadata 里**精确锁死** `livekit-agents==1.2.9`(用 `==` 而非范围),它一旦在依赖图里,pip 就把整个 `livekit-agents` 拉回 1.2.9,连带把 `livekit-plugins-fishaudio` 也压到老版(`>=1.2,<2.0` 解析到不含 `reference_id` 的 1.2.x)→ Fish 音色参数报 `unexpected keyword argument 'reference_id'`。aliyun 插件已停更,不会修这个 pin。
- **解法(用户拍板)**:**不用 livekit-plugins-aliyun,照着它的实现自己迁移一份适配**,全栈升 `livekit-agents` 1.6.x。验证过 aliyun 插件 stt.py/tts.py **只用 livekit.agents 稳定 API**(`stt.STT`/`stt.SpeechStream`/`tts.TTS`/`tts.SynthesizeStream`/`tts.AudioEmitter`/`utils.ConnectionPool`/`utils.audio.AudioByteStream`)+ 裸 `aiohttp` WebSocket 连 DashScope,与 1.6 完全兼容(它"锁老"纯属 metadata,代码不依赖老 API)。
- **新增模块**(套路同自写 `fish_stt.py`):
  - `plugins/voice/jellyfish_voice/aliyun_stt.py`:Paraformer 实时流式 STT。协议 `wss://dashscope.aliyuncs.com/api-ws/v1/inference`,run-task→推 100ms 音频帧→finish-task;`result-generated` 事件回传 sentence(interim/final by `sentence_end`)。**begin_time/end_time 缺失兜底成 0.0**(防下游 `end_time>0` 比较踩 NoneType)。鉴权 `DASHSCOPE_API_KEY` env 或构造 `api_key`。
  - `plugins/voice/jellyfish_voice/aliyun_tts.py`:CosyVoice 流式 TTS。run-task→continue-task(text)→finish-task;BINARY 帧即 PCM,`emitter.push(data=...)`;TEXT 帧看 `header.event`(task-finished/task-failed)。**关键改动:原插件用 `osc_data.TextStreamSentencizer` 分句,但 osc-data 会拖进 librosa/kaldifst/av 等重依赖 → 用自写轻量 `_Sentencizer` 替代**(按句末标点 `。！？!?；;…\n` 切句 + 去 emoji + 80 字强制断句),worker 镜像保持精简。
- **核验过(下载 wheel 实查,免无谓重建)**:`AudioEmitter` 在 1.6.0 的签名与插件用法**完全一致**——`initialize(*,request_id,sample_rate,num_channels,mime_type,frame_size_ms=200,stream=False)`、`start_segment(*,segment_id)`、`end_segment()`、`push(data:bytes)`。
- **`agent.py`**:`_aliyun_stt`/`_aliyun_tts` 改 `from . import aliyun_stt/aliyun_tts`,不再 `from livekit.plugins import aliyun`;`api_key` 取 `providers["dashscope"]["api_key"]`,没有则传 None(本地模块 None 时自读 `DASHSCOPE_API_KEY` env,无 NotGivenOr 坑)。
- **`pyproject.toml`**:**删 `livekit-plugins-aliyun`**;`livekit-agents`/openai/fishaudio/silero/turn-detector 全锁 `>=1.6,<1.7`;补 `aiohttp>=3.9`(本地 DashScope 适配需要,虽 agents 已间接带)。
- **生效**:重建 voice-worker 镜像(`build --no-cache voice-worker` + `up -d --force-recreate voice-worker`)。重建后 fishaudio 是真 1.6,`reference_id` 可用;阿里云 STT/TTS 走本地模块。**通用规律:某依赖用 `==` 精确锁核心库且已停更时,与其迁就老栈,不如把它那点只用稳定 API 的代码迁移进来自己维护。**
- **实测验证(2026-06-15 升级后首拨)**:日志 `阿里云 STT WebSocket 已连接`✓、`Fish TTS 构造 kwargs=[...voice_id...] 插件签名接受=[...voice_id, base_url, chunk_length...]`✓(确认装的是真 1.6 fishaudio,签名含 voice_id 等)。`turn_detector` 仍报 `languages.json` 缺失 → 回退 VAD(老的非致命,Dockerfile `download-files || true` 吞了下载失败)。另有 1.6 新弃用警告 `allow_interruptions/min_interruption_words/turn_detection deprecated, use turn_handling=TurnHandlingOptions(...)`(v2.0 前不影响,后续再迁)。

## ⚠️ Core 应用代码烤进镜像:改路由/网关必须 rebuild 而非 restart(2026-06-15)
- **现象**:voice-worker 升级 1.6 后,STT/TTS 都起来了,但 LLM 报 `APIStatusError 404 {'detail': 'Not Found'}`(堆栈落 `livekit/agents/inference/llm.py`,base_url=`http://jellyfishbot:8000/api/voice/live/llm/v1`)。
- **判读**:`{'detail': 'Not Found'}` 是 **Starlette「路由没匹配」专属响应**(不是 LiveKit 推理网关、不是鉴权失败——鉴权失败是 401/403)。说明请求**确实到了 Core**(jellyfishbot:8000),但 `/api/voice/live/llm/v1/chat/completions` 没被命中。代码侧已确认:`voice_live.py:390 @router.post("/llm/v1/chat/completions")` + router `prefix="/api/voice/live"` + `main.py` 已 `include_router(voice_live_router)`,路径完全对得上;且同 router 的 `/session` 能用(worker「会话引导完成」)→ 纯运行态问题。
- **根因**:`docker-compose.yml` 的 `jellyfishbot` 服务是 `build:`(Dockerfile 把 `./app` **烤进镜像**),`volumes:` 只挂 `./data`、`./data/users`,**不挂 `./app`**。所以新加的网关端点没进运行中的镜像。
- **铁律**:**改 Core Python 代码(路由/服务/任何 app/ 下文件)只 `restart` 无效,必须 `build`**:`docker compose build jellyfishbot && docker compose up -d --force-recreate jellyfishbot`。容器名=`jellyfishbot`。验证路由已注册:`docker exec jellyfishbot curl -s -o /dev/null -w "%{http_code}" -X POST http://localhost:8000/api/voice/live/llm/v1/chat/completions` → 非 404(401/422 均说明端点在)即可。(对比:voice-worker 也是 build 非挂卷,同理。只有挂了卷的 `./data` 类才热更。)
- **✅ 重建后实测通了(2026-06-16 03:04)**:`会话引导完成`→`前台 LLM 网关 model=bedrock:claude-sonnet-4-6`→`Fish TTS 构造`→`阿里云 STT 已连接`,**无 404 / Connection error**。整条链路(自迁移阿里云 STT/TTS + Fish 1.6 + Core LLM 网关 + 全栈 livekit-agents 1.6)打通。

## 语音 worker 两个非致命噪音清理(2026-06-16)
- **turn_detector 缺 `languages.json`(每次拨号 ERROR 一行,回退 VAD)的真因**:`download-files` 只给**已注册**的插件下模型。`agent.py` 顶部只 `from livekit.plugins import openai, silero`(worker 启动日志 preload 也只有这俩),turn_detector 是在 `_turn_detection()` 里**懒加载**的——构建时跑 `python -m jellyfish_voice.agent download-files` 时它**没注册** → 被跳过 → EOU 模型(含 languages.json)没进镜像 → 运行时加载失败回退 VAD。
  - **修复**:`agent.py` 顶部加 `try: from livekit.plugins import turn_detector except Exception: turn_detector=None`。该包 `__init__` 一导入就 `Plugin.register_plugin(EOUPlugin(_EUORunnerEn/Multilingual))`,而 `EOUPlugin`(base.py)有 `download_files` → 构建时即下模型进镜像。导入失败不致命(可选依赖,运行期仍回退 VAD)。Dockerfile 的 `download-files || true` 保留(构建容错)。**通用规律:livekit 插件都靠"导入即注册",`download-files` 只遍历已注册插件;凡是想在构建时预下模型的插件,必须在被 `download-files` 加载的模块里顶部导入。**
- **1.6 弃用警告 `allow_interruptions/min_interruption_words/turn_detection deprecated → use turn_handling=TurnHandlingOptions(...)`**:1.6 的 `AgentSession.__init__` 用 `@deprecate_params` 把这三个(及 min/max_endpointing_delay、false_interruption_timeout 等)标弃用,v2.0 移除。
  - **新结构 `turn_handling: TurnHandlingOptions`(普通 dict 即可,见 `livekit/agents/voice/turn.py`)**:`{"turn_detection": <mode|检测器|None>, "endpointing": {min_delay,max_delay,...}, "interruption": {"enabled":bool,"min_words":int,"min_duration",...}, "preemptive_generation": {...}, "user_turn_limit": {...}}`。映射:`turn_detection`→`turn_handling["turn_detection"]`;`allow_interruptions`→`interruption.enabled`;`min_interruption_words`→`interruption.min_words`。
  - **修复**:`agent.py` 装 `turn_handling={"interruption":{"enabled":..,"min_words":..}}`,有 EOU 模型才加 `turn_handling["turn_detection"]=_td`(否则省略 → 会话 auto-select 回退已传入的 VAD)。删掉 AgentSession 上那三个旧 kwargs,警告消除。
- **生效**:两处都在 voice-worker,重建镜像生效(`build --no-cache voice-worker` + `up -d --force-recreate voice-worker`)。turn_detector 修复需 `--no-cache`(或确保 download-files 层重跑)才会真正下模型。
- **其余非致命**(留着即可):`silero: inference is slower than realtime delay=0.3`(冷启动/CPU 抖动,偶发)、`_SegmentSynchronizerImpl.on_playback_started called after start_fut is set`(TTS 分段时序边界,无害)。

## 前端会话列表运行状态指示器改造(2026-06-22)
- **需求**:会话列表(`frontend/src/pages/Chat/index.tsx` 的 `conversations.map`)原来用 `.streamingDots`(跳动三点 `...`)表运行中、`.hitlBadge`(黄圈 `?`)表待审批,且都在**标题之后**。改为:运行中=点阵涟漪动画(bloom ripple),待审批=竖排 3 点琥珀黄(中深两头浅、静止),图标移到**最左**(标题之前)。用户参考了一个 Next.js 的 DotmSquare11 点阵组件,但其依赖(dotmatrix-core/hooks、@/components/ui)本仓库没有且是 Next 风格——**没整套搬,改为自写轻量自包含组件**(经用户确认走 lightweight 方案)。
- **新组件** `frontend/src/pages/Chat/components/RunIndicator.tsx` + `RunIndicator.module.css`:`<RunIndicator state="running"|"approval" label?/>`。running=3×3 inline-grid 点阵,每点按到中心的曼哈顿距离 `--dmx-ring`(常量数组 `[2,1,2,1,0,1,2,1,2]`)错开 `animation-delay`,`@keyframes dmxRipple` 由内向外缩放+透明度涟漪;approval=竖排 inline-flex 3 点,`var(--jf-warning,#faad14)`,中间 `nth-child(2)` opacity:1、两头 opacity:.4,静止。尊重 `prefers-reduced-motion`(涟漪停成静态淡显)。自定义 CSS 变量在 TSX 里用 `{ '--dmx-ring': ring } as CSSProperties`。
- **接入**:index.tsx 列表项把 RunIndicator 放到 `.convTitle` **之前**(`isConvStreaming`→running、`isConvHitl`→approval,二者互斥);顶部「在别处运行/待审批」横幅(`streamElsewhereBanner` 的 `streamElsewhereText`)把原 `⏸/⏳` emoji 换成同款 `<RunIndicator>`。组件根 class 自带 `margin-right:6px` + `vertical-align:middle`,列表/横幅复用同一间距。
- **清理**:`chat.module.css` 删掉死代码 `.streamingDots`/`.hitlBadge`/`@keyframes dotBounce`/`@keyframes hitlPulse`(只留一行注释指向新组件)。i18n key `chat.streamingTitle`/`chat.hitlBadge`/`chat.runStateRunning`/`chat.runStateAwaiting` 仍作 title/aria-label 文案保留。
- **校验**:`npx tsc --noEmit` 通过、无 lint。**生效**:前端构建即可(纯 React/CSS,无后端改动)。

### 用户 query 左侧导航列(QueryNavMarker, 2026-06-22 初版 → 内联 → 固定列 → **左侧居中悬浮**, 2026-06-24 终版)
- **需求(最终澄清)**:导航 bar **悬浮在页面左侧垂直居中**(不随消息滚动消失);**bar 数 = 用户 query 数**;滚动到对应 QA 时对应 bar 高亮(active);点击 bar 跳到对应 query。默认短横线,hover 略放大并显示 query 前 100 字。
- **三次踩坑教训(关键,别再走回头路)**:
  1. **内联进每条 item**(`messageRow=queryNavCell+messageCell`):跟随消息滚动、无 active、布局侵入。废弃。
  2. **rail 放进滚动容器 `.messagesContainer` 内 `position:absolute; top:0; bottom:0`**:本以为复用 `.scrollBottomBtn` 的 pinned 技巧能固定,实测**用 `top:0;bottom:0` 撑满会让 rail 高度=滚动内容全高**(absolute 在 scroll container 内的 containing block 高度是 content 高而非可视高),bars 全堆在内容顶部、滚动即消失。且 active 变化时对 rail item 调 `scrollIntoView({block:'nearest'})`,当 rail 不溢出时会**向上冒泡滚动消息容器 → 把用户「拉回」无法下拉到底**(这就是「页面被不断拉回」的根因)。废弃。
  3. **终版(正确)**:rail **挂到不滚动的祖先**——admin 是 `.chatArea`、service 是 `.page`(两者都加 `position:relative`),`position:absolute; left:4px; top:50%; transform:translateY(-50%); max-height:70vh; overflow-y:auto`。脱离滚动容器后**绝不随内容滚走**;垂直居中=「左侧中部悬浮」。bars 多到超过 70vh 时 rail 自身滚动。
- **active 自动滚入 rail 的防拉回处理(必须)**:active 变化时**不要**用 `scrollIntoView`(会冒泡)。改为:`if (rail.scrollHeight <= rail.clientHeight+1) return;`(不溢出直接不滚),溢出时**手动算 `rail.scrollTop`** 把 active item 带进 rail 可视区。只动 rail.scrollTop,永不触碰消息容器。
- **align 注意**:rail 用 `align-items:center` 但**不要 `justify-content:center`**——flex column + `justify-content:center` + `overflow-y:auto` 会裁掉顶部且无法滚到。rail 整体居中靠 `top:50%/translateY(-50%)` 实现,内部内容 `flex-start` 自然排布。
- **admin 架构(虚拟化)**:rail 渲染在 `index.tsx` 的 `.chatArea`(不在 `MessageList`/Virtuoso 内)。`MessageList` 仅:① 每条 item 外包 `<div data-jf-msg-index data-jf-msg-role>`;② 通过 handle 暴露 `scrollToMessage(index)`(走 `virtuosoRef.scrollToIndex({index,align:'start',behavior:'smooth'})`,**虚拟化下目标行可能未挂载,必须用 scrollToIndex 不能 DOM scrollIntoView**)。`index.tsx`:`userMarkers=messages.filter(role==='user')`;active 检测在 `scrollParentEl` 挂 passive scroll(rAF 节流),DOM rect 法取「基准线(容器顶+96px)之上最靠近」的用户消息下标;点击 → `messageListRef.current.scrollToMessage(index)`。
- **service-chat 架构(不虚拟化)**:rail 渲染为 `.page` 直接子节点(`!showWelcome && userMarkers.length>0`)。`scrollToMessage` 直接 `messagesContainerRef.querySelector('[data-jf-msg-index]').scrollIntoView({block:'start',behavior:'smooth'})`(节点恒在)。active 检测同款 DOM rect 法,监听 `.messages` 的 scroll。用户消息 `.userMsg` div 打 `data-jf-msg-index/role`。
- **文件**:`components/QueryNavMarker.tsx`(`active` prop → `markerActive` + `aria-current`)、`queryNav.module.css`(`.markerActive .dash` 主色+发光,**两端共享**)、`utils/userQueryPreview.ts`、`MessageList.tsx`(只剩 `scrollToMessage` + data 属性,rail/active 已移走)、`index.tsx`(rail + userMarkers + active effect + guarded rail-scroll)、`chat.module.css`(`.chatArea position:relative`、`.queryNavRail` 悬浮居中、messagesContainer 左 padding 38/移动 30)、`ServiceChatApp.tsx` + `serviceChat.module.css`(`.page position:relative`、`.queryNavRail` 同款、`.messages` 左 padding 38/移动 30)。
- **不要做的事**:① rail 绝不放滚动容器内(用 top/bottom 撑满会高度错算 + 滚走);② active 自动滚入 rail 用手动 scrollTop + 溢出守卫,**绝不用 scrollIntoView**(冒泡拉回);③ rail 不要 `justify-content:center`(裁顶);④ admin 点击跳转必须 Virtuoso `scrollToIndex`(虚拟化),service 才能直接 DOM scrollIntoView;⑤ 不要用 margin 给 messageItem 间距(Virtuoso offsetHeight 不算 margin,见上文 margin 陷阱)。

## `ValueError: No generations found in stream` 空流崩溃兜底(2026-06-22)
- **现象**:微信 bridge(`_run_agent_and_reply` 的 `agent.astream`)偶发整轮 abort,traceback 末尾 `langchain_core/.../chat_models.py:_agenerate_with_cache → generate_from_stream(iter(chunks)) → raise ValueError("No generations found in stream.")`。即模型这次**一个 chunk 都没产出**(空补全:只有 message_start/stop,无任何 content block),LangChain 流式聚合拿到空迭代器就抛。
- **定位**:报错服务 `svc_08f23ac2` 模型是 `bedrock:claude-opus-4-7-thinking`(thinking 变体,走我们自有 `app/services/bedrock.py:ChatBedrockInvoke._astream`,**不是**直连 Anthropic)。**排除「两个微信服务串模型」**的猜想:① 模型按服务隔离——`consumer_agent.py` cache_key 含 `service_id::conv_id`,且 `_resolve_model(svc_config["model"])` 各服务各自解析,admin 微信是另一 service_id→另一 agent+模型,不共享实例;② 回复人设「我是 Income Benchmarking Agent」正是本服务自身 system prompt,没串成 admin 的 4.5。**真因**:thinking 模型 + 长上下文经 summarization 截断,某次 agent-loop 调用(常在 tool_result 之后)返回空 turn,而我们的 `_astream`/`_stream` 对「零 chunk」**没兜底**→崩。两次失败都在同一 `ws_66d954cd` session(该会话历史很长)→不是随机干扰,是这条会话状态触发。
- **summarization 阈值复盘(不动)**:deepagents 默认 `SummarizationMiddleware` trigger=`("fraction",0.85)`(≈模型上限 85%,opus-4-7≈170k tokens)、keep=`("messages",20)`——**并不激进**,这次截断是会话**确实很长**所致。调小阈值只会更激进、更糟,故**保持默认**,靠空流兜底解决症状。
- **修复 A — `ChatBedrockInvoke`(`app/services/bedrock.py`)**:把单次 HTTP 流式抽成 `_stream_once(body,state)`/`_astream_once(body,state)`;`_stream`/`_astream` 统计 yielded,**零 chunk 自动重试 1 次**,仍空则 `yield ChatGenerationChunk(message=AIMessageChunk(content=""))` 让 agent 优雅收尾(不崩、不抛 traceback),并 `log.warning` 出 model/stop_reason 便于排查。重试只针对「成功但空流」,非 200 仍抛 RuntimeError(真错不掩盖)。
- **修复 B — `_convert_messages`(同文件)**:空 assistant turn(无 text 无 tool_use,如兜底产出的空 turn)由原来塞 `{"type":"text","text":""}` 改为**直接 `continue` 跳过**——空字符串 text block 会让下一轮 Anthropic 400(text block 不可为空)。Anthropic 容忍连续同角色消息,跳过安全。
- **修复 C — 直连 Anthropic(`app/services/agent.py`)**:`init_chat_model` 返回的 `ChatAnthropic` 没法直接改其库内 `_astream`;用 `_apply_empty_stream_guard(model)` 把实例 `__class__` **热替换**为 `_GuardedChatAnthropic(ChatAnthropic)`(惰性构建+缓存,仅覆写 `_astream`/`_stream` 同款重试+兜底,**不加字段**→pydantic v2 安全,`isinstance` 仍真)。已在 `_resolve_model` 的 minimax、thinking、generic 三个 init_chat_model 返回处套上;原「直接返回 model_id 字符串」也改为本地构建+套兜底(覆盖 api_key 走环境变量的 anthropic 路径)。
  - **为何 `__class__` 热替换安全**:deepagents `graph.py` 的 `AnthropicPromptCachingMiddleware(unsupported_model_behavior="ignore")` 是**无条件添加**(非 Anthropic 自动 no-op),且按 `isinstance` 识别 → 子类仍命中,prompt caching 不受影响。`bind_tools` 返回 `RunnableBinding`(`bound=` 该实例)→ 工具绑定后调用仍走 guarded `_astream`。已本地实测(class-swap 后 isinstance/字段/bind_tools 均正常;Bedrock 空流重试→兜底链路单测通过:EMPTY→2 次调用+1 空 chunk,GOOD→1 次不重试)。
- **生效**:纯 Core 改动,需重建 jellyfishbot 镜像(代码 baked 进镜像,非挂载):`docker compose build jellyfishbot` + `up -d --force-recreate jellyfishbot`。

## 微信 service 对话在 admin 端「只见用户消息、无助手回复」修复(2026-06-22)
- **现象**:微信侧 service 对话,admin 界面聊天记录只有用户发的,没有助手回复。Web 端 admin 显示正常。
- **两个根因**(`app/channels/wechat/bridge.py:_run_agent_and_reply`):
  - **① 崩溃在保存之前**:助手消息的 `save_consumer_message(..., "assistant", ...)` 原本在 streaming + HITL 循环**全部跑完之后**才执行(裸在 `async with` 体里、无 try)。任何异常(空流报错即其一)从 `agent.astream` 冒泡 → 这行被跳过 → 助手消息从没落库。用户消息在 agent 跑之前(`handle_wechat_message` line 81)就存了,故只剩用户消息。
  - **② send_message 投递→存空内容(主因)**:微信渠道注入了 `humanchat` 的 `send_message` 工具,回复文案走工具投递(`delivery.deliver_tool_message` 解析 ToolMessage 里的 `payload["text"]`),而 bridge 累积的 `full_response` 只收**直接流式 AIMessage 文本**——所以 `full_response` 往往是空的。即便不崩,`save_consumer_message(assistant, full_response="")` 也只存了空内容 → admin 渲染不出回复。Web 端(`channel="web"`)**不注入** send_message,`full_response` 直接拿到流式文本,所以 web 正常。
- **修复**:① 在 ToolMessage 的 `send_message` 分支里解析 `json.loads(content)` + `extract_media_tags` 取出 cleaned 文案,累积进新列表 `delivered_texts`;② 把整个流式循环包进 `try/finally`,助手保存移到 `finally`——内容优先 `full_response.strip()`,为空则用 `"\n\n".join(delivered_texts)`,有内容或有 tool_records 才存。这样崩溃也能把已产出内容落库,且工具投递的回复也能被 admin 看到。
- **关联空流报错**:②(存空内容)是主因、与空流无关;①(崩溃跳过保存)与空流报错相关——空流兜底修复后崩溃大减,finally 再兜一层把残留异常也保住记录。
- **scheduler 路径无此问题**:`scheduler.py` 的 `save_consumer_message` 用显式 `output_text` + blocks,内容非空。
- **生效**:同上,重建 jellyfishbot 镜像。

## 定时任务可指定执行模型(2026-06-24)
- **需求**:agent 创建定时任务时无法指定执行用的 LLM 模型,任务执行永远走系统默认(anthropic claude sonnet 4.5)。改为:agent 可在创建/派生任务时指定模型,**留空默认 = 当前 agent 正在用的模型**。
- **关键背景**:scheduler 的 `_run_agent_task` **本来就读** `config.get("model")`,为空才回退 `_get_default_model()`。缺口只在**工具层没暴露 model 参数** → task_config 永远没 model → 永远走默认。所以 admin 路径只需让工具写 `task_config["model"]` 即可,scheduler 零改动。
- **三个入口全做**(用户选 admin+spawn+service、reject_list 校验、docstring 动态列出可用模型):
  - **共享 helper(`app/services/tools.py`)**:`_available_llm_model_ids(user_id)`(走 `model_catalog.list_models("llm", only_available=True)`,失败/无凭据返回 `[]` → 放行不误杀)、`_model_list_for_docstring(user_id)`、`_validate_task_model(model, default_model, user_id) -> (resolved, error_or_None)`(留空→default;有效→原样;无效**且** ids 非空→返回错误字符串含可用列表,调用方直接 return)。
  - **Admin `create_schedule_tool(user_id, default_model="")`**:新增 `default_model` 工厂参数 + `model` 工具参数;agent 任务分支校验后写 `task_config["model"]`(script 任务忽略 model);docstring 含 `{MODEL_HINT}`/`{DEFAULT_MODEL}` 占位,**`return` 前 mutate `schedule_task.description`** 动态替换(try/except 兜底)。`agent.py::create_user_agent` 在 line 426 传 `default_model=model`(此处 `model` 已是解析后的当前模型 id)。
  - **`create_spawn_child_task_tool()`**:新增 `model` 工具参数;默认继承父任务模型——`TaskContext` 加 `model: Optional[str]` 字段,`_build_task_context_from_meta` 从 `cfg.get("model")` 填充;spawn 时 `_validate_task_model(model, ctx.model or "", ctx.uid)` → 写 `task_config["model"]`(`create_child_task` 把 data.task_config 透传持久化,无需改)。spawn 工厂无 user_id,docstring 不动态枚举,仅说明"留空继承父任务"。
  - **Service `create_service_schedule_tool(..., default_model="")`**:同 admin 模式;`consumer_agent.py::create_consumer_agent` 新增 `model_override` 参数 + cache_key 加 `::m={override}` 后缀,把 `model_id`(= `model_override or svc_config["model"]`)提前到 cache_key 前计算并传 `default_model=model_id` 给工具;`scheduler.py::_run_service_agent_task` 读 `config.get("model")` 传 `model_override=` 给 `create_consumer_agent`(**注意:service 任务的模型此前由 svc_config 决定,不读 task config——必须经 model_override 注入才生效**)。
- **能力提示词**:`CAPABILITY_PROMPTS` 的 `scheduler`/`service_scheduler`/`spawn` 各加一段「指定执行模型(可选)」,强调"留空=当前模型,仅用户明确要求才填,无效报错列可用列表"。
- **model id 格式**:catalog id(`provider:model`,如 `anthropic:claude-opus-4-7`/`bedrock:claude-sonnet-4-6`),与 chat 主页选择器、`_resolve_model` 完全一致。
- **验证**:venv 实测——schedule/spawn/service 三工具都有 `model` arg、docstring 占位被替换、默认模型显示、`_validate_task_model` 空→default/无效→报错;TaskContext.model 字段 + create_consumer_agent.model_override 签名就位;lint 0 错。
- **生效**:Core 改动需重建 jellyfishbot 镜像;前端改动需重建前端。
- **补完(2026-06-24 第二轮)**:
  - **`manage_scheduled_tasks` update 改 model**:admin(`create_manage_scheduled_tasks_tool`)与 service(`create_service_manage_tasks_tool`)的 update 分支都加 `model: Optional[str]` 参数。关键:**prompt 与 model 必须合并进同一个 `cfg = dict(existing.task_config)`**(原来 prompt 单独 `updates["task_config"]=cfg`,若 model 再写一次会互相覆盖)→ 改成 `if agent_prompt is not None or model is not None:` 一个分支内先后合并。model 走 `_validate_task_model(model, "", uid)`:有效写 `cfg["model"]`、传**空字符串**→ `cfg.pop("model")` 清除指定回退默认、无效→返回错误列表。两个 manage 工具均不动态枚举 docstring(参数已足够清晰)。
  - **前端定时任务控制台 model 选择器**(`frontend/src/pages/Scheduler/index.tsx`):`useEffect` mount 时 `getModels()` 拉 `{models, default}` 存 state;agent 任务区(`taskType==='agent'`)在「执行指令」下方加 `<Select allowClear showSearch optionFilterProp="label">`(options=`models.map(id→name)`,extra 提示默认模型);`openEditModal` 回填 `model: cfg.model`;`handleSave` agent 分支 `if (values.model) tc.model = values.model`;详情视图 agent 段 `infoRow('模型', cfg.model || '默认模型')`。`TaskConfig` 类型加 `model?: string`。三处(创建/编辑/详情)都覆盖,留空=默认。
  - 验证:venv 实测两个 manage 工具 args 含 `model`;`npx tsc -b` 0 错;lint 0 错。

## 定时任务指定模型后两个坑(2026-06-24 第三轮)
> 用户反馈:① 手动把 admin 定时任务模型设成 `bedrock:claude-opus-4-7-thinking` 触发→`Agent 执行失败: Bedrock InvokeModel failed (503): {"message":"Bedrock is unable to process your request."}`(**每次必现**);② 任务不设 model 时仍走旧全局默认(sonnet 4.5),没跟随用户在设置页选的默认。
- **坑① 根因:thinking 模型 + 非流式 InvokeModel 端点 = Bedrock 503**。定时任务主循环是 `agent.astream`(流式),但 **deepagents 的 `SummarizationMiddleware` 在上下文超阈值时用 `ainvoke` 调模型做摘要**→走 `ChatBedrockInvoke._agenerate`(非流式 InvokeModel)。opus-4.7 的 `thinking:{type:adaptive}` 在**非流式** InvokeModel 上不被 Bedrock 支持 → 返回 503「unable to process」。报错串是 `InvokeModel failed`(非 `InvokeModelWithResponseStream failed`),据此即可判定是非流式路径。
  - **修复**(`app/services/bedrock.py`):`_generate`/`_agenerate` **不再 POST 非流式 InvokeModel 端点**,改为内部调 `_stream`/`_astream`(流式 InvokeModelWithResponseStream)再用 `langchain_core.language_models.chat_models.generate_from_stream`/`agenerate_from_stream` 聚合成 ChatResult。彻底绕开会失败的非流式端点,且复用 `_stream` 已有的空流重试/兜底。`generate_from_stream` 靠 AIMessageChunk `__add__` 合并 content/tool_calls/thinking,摘要等非流式消费方无感。
  - **通用规律**:Bedrock(乃至 Anthropic)thinking/大模型**优先走流式端点**;凡是自写 ChatModel wrapper,`_generate`/`_agenerate` 直接路由到流式聚合最稳,免去维护两套端点行为差异。
- **坑② 根因:`scheduler.py::_run_agent_task` 的 `_get_default_model()` 没传 user_id**。该函数解析顺序是「user_id 有→`capability_defaults.llm`(用户设置页选的默认) > agent_config.json > DEFAULT_MODEL」,不传 user_id 直接跳过用户偏好 → 退回全局旧默认。
  - **修复**:`model = _get_default_model(user_id)`(user_id 在该函数作用域内已有)。service 任务无此问题(其默认来自 svc_config 自身模型,不调 `_get_default_model`)。
- **生效**:均为 Core 改动,需重建 jellyfishbot 镜像。验证:`generate_from_stream`/`agenerate_from_stream` 在 venv 可导入、bedrock 模块导入 OK、两文件 lint 0 错。

## 超管启动器迭代 P0:Token 用量埋点(2026-06-29)
> 背景:Tauri 启动器本质是**本机文件级超管控制台**(不走 HTTP 鉴权,直接读写同机 `config/`/`users/`/`data/`/`logs/`)。规划了 P0~P4 五阶段超管增强(P0 token 埋点[后端] / P1 配置中心扩展 / P2 账户运维 / P3 用量面板 / P4 系统健康)。**P0 是唯一后端改动且 P3 依赖它,故先做**。分工铁律:**启动器读文件、后端写文件** —— 凡文件里已有的数据启动器加功能零成本(纯 Rust 读 JSON);凡运行时才算得出的(典型=token 用量)必须先后端插桩。token 落每用户文件 → Tauri 读 + EC2 SSH 看同一份,一份数据两头吃。
- **新模块 `app/services/token_usage.py`**:
  - 存储 `users/{admin_id}/llm_usage/usage-YYYY-MM.jsonl`(按月轮转、append-only、不归档),与 `usage_log`(per-service 请求级元数据)正交——那个记"请求多快/成不成",这个记"烧了多少 token"。
  - rich per-call 字段:`{ts, model, input_tokens, output_tokens, total_tokens, service_id, channel, conv_id}`,`channel∈{web,api,wechat,scheduler,batch,voice}`。
  - `record_llm_usage(admin_id, model, in, out, *, service_id, channel, conv_id)`:fire-and-forget,全程吞异常(用量统计绝不能拖垮用户请求),0 token 不落空记录。
  - `TokenUsageCallback(BaseCallbackHandler)`:`on_llm_end` 读 `usage_metadata` 落盘。`_extract_usage` 优先 `generations[*].message.usage_metadata`(LangChain 规范,Anthropic/OpenAI/我们的 Bedrock 都带),回退 `llm_output['token_usage']`(OpenAI 旧式)。`_extract_model` 优先 `llm_output.model_name/model` → generation `response_metadata` → 构造时的 `model_hint` 兜底。
  - `build_usage_callbacks(admin_id, *, service_id, channel, conv_id, model_hint)`:返回 `get_langfuse_callbacks() + [TokenUsageCallback]`,**各 astream/ainvoke 调用点用它替代裸 `get_langfuse_callbacks()`**。
  - `list_records(admin_id, *, limit, max_months)`:newest-first 合并多月 tail,主要给未来 HTTP 端点/Docker parity 用(Tauri 端 Rust 直接读 JSONL)。
- **Bedrock 发射 usage_metadata(`app/services/bedrock.py`)**:`_anthropic_event_to_chunk` 的 `message_delta` 分支在拿到 `output_tokens` 时,用 `message_start` 存的 `usage_in` 拼成 `AIMessageChunk(content="", usage_metadata={input/output/total_tokens})` 发出。LangChain `AIMessageChunk.__add__` 累加 → 回调 `on_llm_end` 读到 → 与原生 ChatAnthropic/ChatOpenAI 统一(否则自写 Bedrock wrapper 的 token 永远抓不到)。
- **接线点(7 处,全部用 `build_usage_callbacks` 替换裸 callbacks 或新增 callbacks)**:`chat.py`(admin web,api_chat + api_chat_resume,channel=web)、`consumer.py`(consumer web/api 两个 config dict,channel=web/api,非流式复用 api 的 config)、`scheduler.py`(`_run_agent_task` admin channel=scheduler + `_run_service_agent_task` service channel=scheduler)、`wechat/bridge.py`(consumer 微信 channel=wechat)、`wechat/admin_bridge.py`(admin 微信 channel=wechat)、`batch.py`(channel=batch)。`voice_live.py` 的 LLM 网关暂未接(语音线另算)。
- **model_hint 取值**:admin 取 `req.model`;consumer 取 `svc_config.model`;scheduler 取任务 `model`/`task_model`;batch 取 `model`。留空也无妨——回调会从 response 元数据兜底。
- **验证**:全模块导入 OK、lint 0 错;模拟 `on_llm_end`(duck-typed LLMResult 带 usage_metadata)→ token 正确落 `llm_usage/*.jsonl`、model 优先从 `llm_output` 取、`list_records` 读回正确。
- **生效**:Core 改动,需重建 jellyfishbot 镜像。
- **待办**:P1 配置中心(扩展 `.env` 编辑到全 provider key/url + 运维开关 UI + 全局时区)、P2 账户运维(封禁/改名/存储占用/已配 key 脱敏展示——决策:仅 masked 不显明文)、P3 用量面板(Rust 读 `usage_log` + `llm_usage` 聚合,按模型/日期/用户)、P4 系统健康(磁盘占用/日志 tail/db 清理)。部署:优先 Tauri 本地包,Docker 侧靠 SSH 管文件,不额外做 HTTP 超管 API。

## 超管启动器迭代 P1:配置中心扩展(2026-06-29)
> 把启动器原来只能填 6 个核心 key(Anthropic/OpenAI/Tavily/CloudsWay)的「核心服务密钥」卡片,升级为覆盖**全 provider key/url + 多模态 per-capability 覆盖 + 联网搜索 + 运维开关 + 全局默认时区**的「配置中心」。核心设计=**单一真相源 + schema 驱动渲染**,新增配置项只改两处(Rust 键名列表 + JS schema),load/save 逻辑零改动。
- **Rust 通用化(`tauri-launcher/src-tauri/src/lib.rs`)**:
  - 删掉硬编码的 `struct EnvConfig`(6 个 `Option<String>` 字段),改为 `type EnvMap = HashMap<String, String>` + 模块级常量 `KNOWN_ENV_KEYS: &[&str]`(配置中心管理的全部 .env 键,**单一真相源**)。
  - `load_env_config(state) -> EnvMap`:解析 .env,只返回 `KNOWN_ENV_KEYS` 里**存在且非空**的键。
  - `save_env_config(config: EnvMap, state)`:① 第一遍遍历现有 .env 行——注释/未知行原样保留;**已知且本次提交**的键 upsert(空值=删除该行);② 第二遍追加文件里原本没有但本次提交了非空值的已知键。**只动已知键**(防误删无关 env),注释/手填的非托管变量永远保留。
  - `test_api_key` 不变(仍只支持 anthropic/openai/tavily 三个 provider 联通测试),新 provider 暂无测试按钮。
  - **命令注册不用改**:invoke_handler 按函数名注册,改签名不影响注册。`cargo check` 通过(2m25s 冷编译)。
- **HTML schema 驱动(`tauri-launcher/dist/index.html`)**:
  - 静态 6 输入框 → `<div id="configGroups">` 空容器 + JS `CONFIG_SCHEMA`(5 组:大模型 Provider / 联网搜索 / 多模态能力覆盖 / 运维开关 / 全局默认时区)。
  - `renderConfigForm()` 按 schema 生成可折叠分组(`.cfg-group` 点击标题展开/收起,首组默认展开);字段 4 种 `type`:`password`/`text`(input.mono)、`toggle`(自定义 `.cfg-switch` 开关)、`select`(STORAGE_BACKEND)。`test:{provider,urlKey}` 的字段才渲染「测试连接」行。
  - `loadConfig()`:先 render 再填值;toggle 用 `_TRUTHY={1,true,yes,on}` 判定 checked,缺值回退 `f.defaultOn`(仅 RESTORE_VENVS_ON_STARTUP 默认 on)。
  - `saveConfig()`:遍历 schema 收集 → toggle 显式写 `"1"/"0"`(确定性,不依赖后端默认,否则 RESTORE_VENVS 取消勾选会因写空被当默认 on),其余 trim 后原值(空=删行)。`testKey(provider,keyFieldKey,urlFieldKey)` 改为按字段键读 `cfg_<KEY>` 输入。
  - **CSS**:新增 `.cfg-group`/`.cfg-group-head`/`.cfg-group-caret`(collapsed 时旋转 -90°)、`.cfg-switch`(品牌粉 toggle)、`.input-box select`。
- **全局时区落地(`app/services/preferences.py`)**:这是 P1 唯一后端改动。新增 `_global_default_tz()` 读 env `DEFAULT_TZ_OFFSET_HOURS`(call-time 读、默认 8、解析失败回退 8.0);`get_preferences` 把 `result["tz_offset_hours"]` 动态填为全局默认**再** `update(json.load)` —— 用户 `preferences.json` 里**显式存了** tz 就覆盖全局默认,**没存**才用全局。`get_tz_offset` 回退也用 `_global_default_tz()`。**局限(诚实记录)**:多数老用户 `preferences.json` 因 `_DEFAULTS` 曾被持久化已带 `tz_offset_hours=8`,全局默认只对「从没存过 tz 的用户」生效——要强制刷全体得走 P2 的批量改 preferences(破坏性,未做)。
- **KNOWN_ENV_KEYS / CONFIG_SCHEMA 键集**(两边必须一致):LLM=ANTHROPIC/OPENAI(key+base_url)、BEDROCK(key+region)、KIMI(key+base_url)、MINIMAX(key+group_id)、DOUBAO(access+secret+region);搜索=TAVILY、CLOUDSWAY(search_key+search_url+read_url);多模态=IMAGE/TTS/VIDEO/S2S/STT(各 key+base_url);运维=DISABLE_SCHEDULER/SAFE_STARTUP/RESTORE_VENVS_ON_STARTUP/SUPERADMIN_SCRIPT_UNRESTRICTED(toggle)+SCHEDULER_MAX_CONCURRENT/RESTORE_VENVS_CONCURRENCY/UVICORN_WORKERS/SCRIPT_CONCURRENCY/SCRIPT_QUEUE_TIMEOUT(text)+STORAGE_BACKEND(select);时区=DEFAULT_TZ_OFFSET_HOURS。
- **生效**:Rust/HTML 改动需 `npx tauri build` 重新打包启动器(HTML 编译进二进制不可热修);`preferences.py` 后端改动需重建 jellyfishbot 镜像。**注意**:配置中心保存的 .env 改动要重启后端服务才生效(启动器「保存配置」toast 已提示)。
- **不要做的事**:① 不要回退到硬编码 struct 字段(每加一个配置项要改 struct+load+save+UI 四处);② 新增配置项**必须**同时改 `KNOWN_ENV_KEYS`(Rust)和 `CONFIG_SCHEMA`(JS),漏一个则 load/save 静默忽略;③ toggle 一律写 `"1"/"0"` 不写空(空对默认 on 的开关有歧义)。
- **待办**:P2 账户运维 / P3 用量面板(读 P0 的 `llm_usage/*.jsonl`,这才是 token 用量"看得见"的地方) / P4 系统健康。

## 超管启动器迭代 P3:Token 用量面板(2026-06-29)
> P0 只把 token 用量埋点落到 `users/{admin_id}/llm_usage/usage-YYYY-MM.jsonl`,前端**看不见**。P3 在启动器新增「用量统计」页直读这些 jsonl 聚合展示——符合「启动器读文件、后端写文件」核心原则,Rust 直接 walk 文件、零新增后端 HTTP,Docker/EC2 也能靠 SSH 看同一批文件。
- **Rust 命令(`tauri-launcher/src-tauri/src/lib.rs`)**:`get_token_usage_stats(months: Option<u32>) -> TokenUsageStats`(months clamp 1..24,默认 6)。
  - 结构:`UsageBucket{calls,input_tokens,output_tokens,total_tokens}` + `.add(inp,out,tot)`;`NamedBucket`(带 name);`TokenUsageStats{total, by_user, by_model, by_channel, by_day, months_scanned}`;`UsageRecord`(`#[serde(default)]` 全字段容错:model/input_tokens/output_tokens/total_tokens/ts/channel,字段名与 `token_usage.py` 写入完全一致)。
  - 流程:① 读 `users.json` 建 `uid→username` 映射(面板友好显示);② walk `users/*/llm_usage/`,每个 admin 取 `usage-*.jsonl` **按文件名倒序**(最近月在前)truncate 到 months 个;③ 逐行 `serde_json::from_str`(坏行 skip),累加到 total + by_user(用 username)+ by_model(空→"(未知)")+ by_channel(空→"(其它)")+ by_day(ts 前 10 字符);④ `total_tokens<=0` 时回退 `input+output`。
  - `bucket_map_to_sorted_vec(map, by_name)`:by_day 按名(日期)排序,其余按 total_tokens 降序。
  - 注册:invoke_handler 末尾加 `get_token_usage_stats`。`cargo check` 通过(exit 0;PowerShell 的 "RemoteException" 是把 cargo 进度当 stderr 的误报)。
- **HTML(`tauri-launcher/dist/index.html`)**:导航加第 5 项「用量统计」(柱状图 svg,在 账户/关于 之间),`data-page="usage"`,showPage 切换时调 `loadUsageStats()`。
  - 页面:月份 `<select id="usageMonths">`(1/3/6/12,默认 3,onchange 重载)+ 刷新按钮(`btn-secondary btn-sm`,**注意没有 btn-ghost 类**);4 卡 stats-bar(输入/输出/总 Tokens/调用次数);`.usage-grid`(2 列,响应式 900px 降 1 列)4 张表(按用户/模型/渠道/日期)。
  - JS:`fmtNum`(K/M/B 缩写)、`renderUsageRows(tbodyId, rows)`、`loadUsageStats()` 调 `invoke('get_token_usage_stats',{months})` 填卡片+4 表;加载/错误态都有占位。
  - CSS:新增 `.usage-select`/`.usage-grid`/`.usage-card-title`/`.usage-empty`/`.usage-num`(tabular-nums 右对齐)。
- **生效**:纯启动器改动(Rust+HTML),需 `npx tauri build` 重新打包(HTML 编译进二进制不可热修)。后端无改动。
- **不要做的事**:① 不要在后端加 HTTP usage 端点给启动器(违背「读文件」原则,且 Tauri 启动器不带后端 token 鉴权);② 字段名严格对齐 `token_usage.py`(改一边必改另一边);③ 引用的 CSS 类(btn/badge 等)先 grep 确认存在,别臆造(btn-ghost 已踩坑→改 btn-secondary)。
- **待办**:P2 账户运维 / P4 系统健康。

## Admin Web 端 Token 用量面板(2026-06-30)
> 与启动器 P3(超管看全量)正交:**admin 在 web 端看自己的 token 用量**,按 模型/供应商/Service/API Key/渠道/日期 拆分。数据源同一份 `users/{admin_id}/llm_usage/usage-YYYY-MM.jsonl`,但走后端 HTTP API(admin 有登录 token,且只看自己——`get_current_user` 天然隔离)。
- **新增 `key_id` 字段(`token_usage.py`)**:`record_llm_usage(..., key_id="")` + `_make_token_usage_callback`/`build_usage_callbacks` 透传。`key_id` = consumer sk-svc API key 标识("key_xxxxxx"),**仅 consumer 侧传**(admin 主链路 web/wechat/scheduler/batch 没有 key→空)。注入点:`consumer.py` 两处 `build_usage_callbacks` 加 `key_id=ctx.get("key_id","")`(ctx 本就带 key_id)。微信 consumer 无 key,默认空。
- **provider 维度不存字段**:catalog id 是 `provider:model`,聚合时 `model.split(":",1)[0]` 派生 provider。admin 每个 provider 只配一个 key,故 **by provider = by provider key**,不必新增字段。
- **聚合函数 `aggregate_usage(admin_id, *, months=3)`(`token_usage.py`)**:读最近 N 月 jsonl(用 `read_jsonl` 全量,非 tail——聚合不能漏行),返回 `{total, months_scanned, by_model, by_service, by_key, by_provider, by_channel, by_day}`,每个 by_* 是 `[{name, calls, input_tokens, output_tokens, total_tokens}]`;by_day 按日期升序,其余按 total_tokens 降序。name 是原始 id(service_id/key_id),**友好名映射不在本模块做**(避免 token_usage 依赖 published)。
- **路由 `app/routes/usage.py`**:`GET /api/usage/summary?months=3`(`get_current_user`,`user["user_id"]`=admin_id)。调 `aggregate_usage` 后,`_label_rows` 把 by_service 的 id→service 名("主链路(Admin)"兜底空)、by_key 的 id→"服务名 · key名"(遍历 `list_services`+`list_service_keys`,"—(无 Key)"兜底空)。main.py 注册 `usage_router`。
- **前端**:`api.ts` 加 `getUsageSummary(months)` + `UsageSummary/UsageRow/UsageBucket` 类型;`pages/Settings/UsagePage.tsx`(月份 Select 1/3/6/12 + 刷新 + 4 卡概览 + 6 张 antd Table:模型/供应商/Service/Key/渠道/日期,响应式 2 列);`Settings/index.tsx` 菜单加「用量统计」(ChartBar 图标,在 voice 与 general 之间);`router/index.tsx` 加 `path="usage"`;i18n `settings.usage` + `usage.*`(zh/en)。
- **超管 vs admin 职责**:超管跨用户全量看**启动器 P3**(Rust 直读 jsonl,serde 默认忽略未知字段→新增 key_id 不影响,launcher 无需改);admin 自己看 **web 设置页**。两条路径读同一批文件,互不耦合。若以后要超管在 web 看跨用户,需新加超管专用端点(当前没做,符合「超管用启动器」决策)。
- **生效**:后端改动(token_usage/consumer/usage/main)需重建 jellyfishbot 镜像;前端需重建。`tsc -b` 0 错、locale JSON 合法、aggregate_usage 冒烟通过。
- **不要做的事**:① 聚合用 `read_jsonl` 全量,别用 `read_jsonl_tail`(会漏);② 友好名映射放路由层不放 token_usage(防循环依赖);③ admin 端点只看自己(`user["user_id"]`),别加 admin_id 入参(防越权)。

## 超管客户端 P2:账户运维增强(2026-06-30)
> 启动器(Tauri 超管客户端)账户管理页新增 4 个运维能力:封禁/解禁、改名、存储占用、已配 Key 脱敏展示。核心架构沿用「launcher 直读写文件,封禁这一项需后端配合」。
- **封禁/解禁(唯一跨组件项)**:
 - 启动器 `set_admin_disabled(userId, disabled)`(`lib.rs`)写 `users/users.json` 的 `disabled: bool` 字段;封禁时**同时清空 `token`** 强制现有会话立即失效。
 - 后端 `app/core/security.py` 配合:`login()` 在校验密码**之前**检查 `info.get("disabled")`→返回 `{"success":False,"error":"账户已被禁用,请联系管理员"}`;`verify_token()` 命中 token 后检查 disabled→返回 None(令牌失效)。**没有这两处后端检查,封禁不生效**。
- **改名 `rename_admin_user(userId, newUsername)`**:改 `users.json` 的 `username`,**User ID(目录名)不变**;遍历查重名(排除自己)冲突报错。username 仅用于显示/登录,不影响 `users/{uid}/` 目录,故安全。
- **存储占用 `get_admin_storage(userId)`**:按需命令(不放进 list,避免列表加载 walk 大目录卡顿)。`fs::read_dir` 列 `users/{uid}/` 顶层条目,目录用 `dir_size_bytes`(walkdir 累加文件 len)、文件用 metadata.len();返回 `{total_bytes, breakdown:[{name,bytes}]}` 按字节降序。前端弹窗展示总占用 + 子目录明细(venv/conversations/filesystem/llm_usage 等一目了然)。
- **已配 Key 脱敏 `get_admin_key_status(userId)`**:读 `users/{uid}/api_keys.json`,对 13 个密钥字段(openai/anthropic/tavily/cloudsway/image/tts/video/s2s/stt/kimi/minimax/doubao/bedrock)看**密文非空**即视为已配置→返回友好供应商名列表。**不解密**(master key 在 data/encryption.key,启动器不碰),只判断字段是否有值。前端「API Key」列点「● 已配置」弹窗展示供应商 chip 列表。
- **前端(`dist/index.html`)**:账户表加「状态」列(✓正常/⛔已禁用);操作列加 封禁/解禁(随状态切换文案颜色)、改名、占用、重置密码、删除;3 个新 modal(rename/storage/keyStatus);JS handler `toggleDisabled/showRenameUser/confirmRenameUser/showStorage/showKeyStatus` + `fmtBytes` 字节格式化;`mockInvoke` 加 4 命令 mock(浏览器预览不报错)。
- **验证**:`cargo check` 0 错、`security.py` 语法 OK。**生效**:lib.rs 改动需 `npx tauri build` 重打包;security.py 需重建/重启后端;dist/index.html 在 dev 模式 Ctrl+R 即时,生产需 rebuild。
- **不要做的事**:① 存储占用别放进 `list_admin_users`(每次列表都 walk 全目录会卡,admin≤20 也不值得);② key 脱敏只读密文存在性,绝不在启动器解密(架构隔离);③ 封禁必须改后端 login+verify_token 两处,只写 users.json 不改后端=封禁无效。

## 超管客户端 P4:系统健康面板(2026-06-30，纯只读)
> 用户明确「不做清理」——超管会上 EC2 直接处理破坏性操作,启动器只看关键统计。故 P4 砍掉原计划的 db 清理/VACUUM/删日志/删孤立文件,只保留只读统计。纯启动器侧,**无后端改动**。
- **3 个只读命令(`lib.rs`)**:
 - `get_system_health()` — 快照(不 walk 大目录,秒回):运行态 `backend_running`/`frontend_running`(复用 `is_port_open` + AppState 端口)、checkpoints `{exists, db_bytes, wal_bytes, shm_bytes}`(只读 `data/checkpoints.db` + `-wal`/`-shm` 文件 len,WAL 模式三件套)、`logs: [{name,bytes,modified}]`(列 `logs/` 目录,无则空)。
 - `get_disk_usage()` — 按需(单独「计算占用」按钮,因 walk venv/.git/users 较慢):`fs::read_dir(project_dir)` 顶层每项用 `dir_size_bytes`(P2 加的 walkdir 累加)算占用,降序,返回 `{total_bytes, breakdown:[{name,bytes}]}`。
 - `tail_log(name, lines)` — 只读末尾 N 行。**路径安全**:`name` 拒绝含 `/`、`\`、`..`;`fs::canonicalize` 后校验 `target.parent() == logs_dir`(防符号链接逃逸);`lines.clamp(1,5000)`。
- **复用**:`StorageBreakdown`(P2)、`dir_size_bytes`(P2)、`is_port_open`/AppState 端口(已有)。新 struct:`SystemHealth`/`CheckpointInfo`/`LogFileInfo`/`DiskUsage`。
- **前端(`dist/index.html`,纯中文单文件无 i18n)**:导航加「系统健康」(脉搏图标,usage 与 about 之间);`page-health` = 运行态 4 卡(后端/前端在线徽章·DB大小·日志数) + 磁盘占用卡(「计算占用」按钮触发,total + breakdown 表) + Checkpoints 卡(db·wal·shm 三行 + "清理请在服务器 VACUUM"提示) + 日志卡(文件下拉 + 行数 100/300/1000 + 「查看」→ `<pre>` 渲染);JS `loadSystemHealth`/`loadDiskUsage`/`loadLogTail`(复用 `fmtBytes`);click handler 加 `page==='health'`;`mockInvoke` 补 3 命令。
- **验证**:`cargo check` 0 错(增量 ~1m50s)。**生效**:lib.rs 需 `npx tauri build` 重打包;dist/index.html dev 模式 Ctrl+R 即时、生产需 rebuild。
- **不要做的事**:① 任何清理/删除/VACUUM 都不做(超管 EC2 处理,启动器纯只读);② 磁盘占用必走单独按钮别塞进 get_system_health(walk 慢会拖垮页面首屏);③ tail_log 必须 canonicalize + 校验父目录,别只用字符串前缀判断(符号链接可逃逸);④ 不在健康页做内容级重统计(对话/token 已由 P2 存储 + P3 用量面板覆盖)。

## Admin Web 用量页 UI 重设计 — V3 玻璃霓虹(2026-06-30)
> 先在 `tauri-launcher/design/` 出了 3 版科技风独立 HTML 样例(v1-hud 军工 HUD / v2-terminal 复古终端 / v3-glass 玻璃拟态霓虹,统一 mock 数据便于对比,不进 dist 打包)。用户选 **V3**,要求落到 admin web 端 `UsagePage.tsx`(管理员登录后看自己 + service 用量),并**适配双轴主题**(`data-color` 日夜 × `data-style` regular/terminal)。
- **核心纪律:全部走 `--jf-*` 主题变量,零硬编码颜色**。demo 里的霓虹深色全部换成 `var(--jf-accent/primary/secondary/success/warning/info)` + rgb 变体;玻璃卡背景用 `var(--jf-bg-panel)` 叠低透明白 overlay,圆角 `var(--jf-radius-lg)`(terminal 主题自动归零),阴影 `var(--jf-shadow-float)`。这样 dark/light/terminal 三态都自洽——**这是能适配主题的唯一正确做法**。
- **双轴主题来源**(`frontend/src/styles/themes.css`):`[data-color='dark'|'light']` 切调色板(日夜),`[data-style='terminal']` 把所有 `--jf-radius-*` 归零 + 提供 `--jf-glow`。组件只引用 `--jf-*`,切主题零改动。
- **`--mono` 在前端未定义**(旧 UsagePage 用 `var(--mono)` 实际 fallback 到默认 monospace),统一改用 `var(--jf-font-code)`。
- **复杂视觉必须走 CSS Module**(`UsagePage.module.css`):极光辉光(`.aurora b` 三个 blur blob,用品牌 rgb,light 下 `opacity:.22` 自动柔和)、玻璃卡顶部渐变描边(`.glass::before` accent→primary 渐变线)、hover/transition、媒体查询。内联 style 做不了 `::before`/keyframes,这是为何不沿用旧版纯 inline。
- **图表纯 SVG 手绘,无第三方库**:① 环形渠道占比图(`Donut`):pathLength=100 技巧(`r=15.9155` 周长≈100),`stroke-dasharray="p 100-p"` + `dashoffset=25-acc` 累加,top5 + 「其他」聚合,配色循环 `DONUT_COLORS`(全 var 引用)。② 每日趋势面积图(`AreaChart`):input/output 双 series,JS 算 polyline + area path,`<linearGradient>` 用 `rgba(var(--jf-accent-rgb),.45)→0` 渐变填充,`vectorEffect="non-scaling-stroke"` 保线宽,标签按 `labelStep` 稀疏防拥挤,单点/空数据有 guard。
- **数据维度**(沿用 `GET /api/usage/summary` 的 `UsageSummary`):4 个汇总 stat 卡(输入/输出/总 tokens/调用次数,各配色 cy/pk/gr/pu)+ 环形(渠道)+ 面积(每日)+ Service 高亮卡(`fillSvc` 绿青渐变,呼应"看自己和 service 用量")+ Provider/Key 双列条。**stat 卡子指标只用真实派生值**(input/output 占比 %、≈tok/次、months_scanned),**不造假涨跌**(后端聚合无上期对比数据,绝不编 ▲8.2%)。
- **i18n**:`zh/en.json` 的 `usage.*` 补 `selfSub/month1/month3/month6/share/perCall/callsShort/monthsScanned/trend/days/others/footer`;所有 `t()` 调用带默认值兜底,缺 key 也不空白。
- **移动端**:`.module.css` `@media(max-width:767px)` 把 statGrid 2 列、mainGrid/twoCol 单列、`padding-left:52px` 避让汉堡按钮(与其它 settings 页一致)。
- **验证**:`npx tsc -b` 0 错、ReadLints 0 错。**生效**:前端构建即可。
- **启动器同款 V3(已完成)**:`dist/index.html` 的 `page-usage`(超管全量)已翻译为同款 V3——无 CSS Module,改用隔离前缀 `.uz-*` 工具类 + 内联 `<style>` + `--jf-*`;`--jf-accent-rgb` 等 rgb 三元组启动器未定义,故极光辉光用从 `--jf-accent/primary/secondary` 算出的字面 `rgba()`;图表用 vanilla JS 函数(`uzBars/uzDonut/uzArea`)直接拼 SVG 字符串。
- **不要做的事**:① 别在组件里硬编码任何 hex 颜色(会破坏 light/terminal);② 别为省事把图表换 echarts/recharts(纯 SVG ~150 行够用,且天然吃 CSS 变量配色,第三方库要额外注入主题色);③ stat 卡别加后端没有的"环比/同比"假数据;④ 别再用 `var(--mono)`(未定义),用 `var(--jf-font-code)`。

### 共享 `UsageView` 组件 + service 页单服务用量(2026-06-30)
> 用户要求:让 service 详情页也能看「当前 service 的最近用量」。把 V3 可视化逻辑抽成可复用组件,admin 全量页与单 service 页共用。
- **抽离组件 `frontend/src/components/UsageView.tsx`**:从 `UsagePage.tsx` 提取 V3 渲染(StatCard / Donut / AreaChart / BarList / CardTitle + helpers `fmtNum/fmtTok/pct/DONUT_COLORS`),复用 `pages/Settings/UsagePage.module.css` 的 `.uz-*`/glass 样式。**props**:`data: UsageSummary` + 可选 `dims?: UsageDims`(`{channel,model,day,service,provider,key}` 布尔,未传默认全 true)。`dims` 控制条件渲染——channel+model 同真才并排 mainGrid,否则各自独占一行;provider+key 同理 twoCol。**DRY**:`UsagePage.tsx` 已重构为壳(header + Segmented 月份 + 调 `getUsageSummary` + `<UsageView data={...} />`,无 dims=全量)。
- **后端单 service 聚合**:`token_usage.aggregate_usage(admin_id, *, months, service_id=None)` 加 `service_id` 过滤(循环里 `str(rec.get("service_id",""))!=svc_filter` 跳过)。新端点 `GET /api/services/{id}/token-usage?months=`(`services.py`,**函数内 lazy import** `aggregate_usage` + `usage.py` 的 `_key_name_map/_label_rows` 做 key 友好名映射,避免顶层循环 import),返回与 `/api/usage/summary` 同结构 `UsageSummary`。client `api.ts::getServiceTokenUsage(serviceId, months=3)`。
- **AdminServices 集成**:「使用情况」`ModuleCard` 的 Segmented 加第三项 `tokens`(label「用量」);切到 tokens 且 `!svcTokenUsage` 时**懒加载** `loadSvcTokenUsage`;tokens 视图内嵌月份 Segmented(1/3/6 月)+ `<UsageView data={svcTokenUsage} dims={{channel,model,day,key, service:false, provider:false}} />`(单 service 不显示 by_service/by_provider——本来就只这一个 service)。`selectService` 切服务时 `setSvcTokenUsage(null)` + `setUsageView('convs')` 防陈旧串台。
- **验证**:`npx tsc -b` 0 错、ReadLints 0 错。**生效**:Core 改动(token_usage/services.py)需重建 jellyfishbot 镜像;前端改动需重建前端。

---
> Source: [LiUshin/OpenJellyfish](https://github.com/LiUshin/OpenJellyfish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
