## danmuai

> - **双窗架构**：`MainWindow`（设置/日志）+ `DanmuOverlay`（透明置顶弹幕），入口均在 `main.py`

# AGENTS.md — DanmuAI 桌面弹幕工具

## 架构

- **双窗架构**：`MainWindow`（设置/日志）+ `DanmuOverlay`（透明置顶弹幕），入口均在 `main.py`
- **单例 `DanmuApp`**（`main.py:137`）：管理全局信号，UI 不直接修改逻辑状态，通过 `ConfigStore` 事务性更新
- **并发模型**：截图在主线程（事件驱动 `_screenshot_loop`），压缩+AI 请求由 `QThreadPool.globalInstance()` + `AiRunnable`（`app/runnable.py`）调度，最大 3 in-flight（`MAX_IN_FLIGHT`）
- **图像流**：`compress_screenshot()` 用 PIL + `io.BytesIO` 内存压缩为 JPEG + Base64，无临时文件
- **双 API 模式**：`api_mode=doubao`（默认）走 `/responses` 端点 + 流式；`api_mode=openai` 走 `/chat/completions` + SSE 流式（`ai_client.py:38-43`）
- **弹幕消费**：`AIReplyFIFOBuffer` 缓冲 AI 回复，`reply_timer` 单次触发，根据右侧弹幕密度自适应延迟（200/500/1000ms）逐条送入引擎
- **管道节流**：AI 回复到达后延迟触发下一截图；`_screenshot_scheduled` 标志防重复调度
- **配置存储**：SQLite `%APPDATA%/DanmuAI/config.db`，`api_key` 用 Fernet 加密（`.key` 同目录），无加密降级为 base64；WAL 模式 + 写锁（`config_store.py:37-47`）

## 运行与测试

```bash
pip install -r requirements.txt
python main.py                                    # 启动
pip install pytest pytest-qt Pillow                # 测试额外依赖
python -m pytest tests/test_danmu_engine.py -v     # 单个测试文件
python -m pytest tests/ -v                         # 全部测试
```

## 关键陷阱

- **Overlay 窗口标志**：必须同时设置 `FramelessWindowHint | WindowStaysOnTopHint | Tool | BypassWindowManagerHint` + `WA_TranslucentBackground | WA_TransparentForMouseEvents | WA_ShowWithoutActivating`，加上 Win32 `SetWindowLongPtrW` 设置 `WS_EX_LAYERED | WS_EX_TRANSPARENT` 实现点击穿透（`overlay.py:30-38,52-58`）
- **仅支持主屏幕**：Overlay 和截图都硬编码 `screens[0]`（`snipper.py:15`、`overlay.py:147-156`）
- **截图在主线程**：`ScreenCapturer.grab()` → `screens[0].grabWindow(0, ...)` 直接截取主屏幕（`snipper.py:19`）
- **网络调用**：`AiWorker._request()` 是同步 `httpx.Client`（`http2=True`），运行在 QThreadPool 线程（`runnable.py:23-48`）；注意压缩也在该线程执行
- **快捷键线程安全**：`keyboard` 库在非 Qt 线程回调，信号经 `_ToggleBridge` 路由到主线程（`hotkey.py:5-6,16`）
- **SQLite 跨线程**：`check_same_thread=False`，WAL 模式 + `threading.Lock` 保护写操作（`config_store.py:37,47`）
- **高 DPI**：Overlay 用 `devicePixelRatio()` 缩放 pixmap（`overlay.py:69-71`），多屏/非 100% 缩放需校验截图选区坐标
- **加密锁死**：`%APPDATA%/DanmuAI/.key` 丢失 → 已加密的 API Key 不可恢复
- **程序终止**：主窗口 X 按钮 → `closeEvent.ignore()` + `hide()`（`main.py:128-131`），真正退出只能通过托盘"退出"调用 `DanmuApp.quit()`；`setQuitOnLastWindowClosed(False)`（`main.py:629`）
- **弹幕截断**：`DanmuEngine.add_text()` 硬截断超过 15 字的内容，加 `...` 后缀（`danmu_engine.py:112-115`）
- **去重机制**：内存 `deque(maxlen=30)` + Levenshtein 相似度，阈值通过 `dedup_threshold` 配置（默认 0.85）；启动时从 SQLite 加载最近 30 条到去重窗口（`danmu_engine.py:57,65-75`）
- **场景切换检测**：16×16 灰度指纹哈希，场景变化时清除 `reply_buffer` 中待显示弹幕并重置 round（`main.py:273-281`）
- **失败退避**：连续 5 次 AI 错误 → `_failure_backoff_paused=True` 暂停截图；401/403/402 等致命错误立即暂停（`main.py:467-504`）
- **过时弹幕丢弃**：`drop_stale` 配置 + `freshness`（loose/medium/strict）控制最大容忍 lag（`main.py:376-384`）
- **RTT 自适应冷却**：smart 模式下记录最近 20 次 API RTT，下次截图冷却 = P90(RTT) × 0.9，范围 1.5-30s；前 3 次默认 `interval × 1000`（`main.py:451-458`）
- **`STAGGER_INTERVAL`**：初始值 1.0s，但在 `start()` 和 `_on_config_changed()` 中被覆盖为 `float(interval)`（配置的截图间隔秒数）
- **httpx 线程本地**：每个 QThreadPool 线程通过 `threading.local()` 持有独立 httpx.Client（`ai_client.py:18-30`），连接错误时销毁重建

## 原型优先

修改或新建 Qt 组件页面前，必须先在 `prototype/` 中用 HTML/CSS 原型验证视觉和参数，通过审查后方可复刻到 `ui/`。

## 文档与源码冲突

`docs/` 下的文档部分内容与代码不同步（如声称异步模型、截图落盘、QBuffer 压缩），以 `main.py` 和 `app/` 源码为准。

---
> Source: [PEPETII/danmuai](https://github.com/PEPETII/danmuai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
