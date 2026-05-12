## aircoder

> 透過 Tailscale VPN，從任何裝置遠端存取 Mac 上的完整終端機。前端為 React SPA，後端為 FastAPI，終端機透過 ttyd + tmux 提供。特別為 Claude Code 等 AI 程式開發工具的使用體驗做了最佳化。

# AirCoder

## 專案簡介

透過 Tailscale VPN，從任何裝置遠端存取 Mac 上的完整終端機。前端為 React SPA，後端為 FastAPI，終端機透過 ttyd + tmux 提供。特別為 Claude Code 等 AI 程式開發工具的使用體驗做了最佳化。

## 架構

```
iPhone (Browser) → Tailscale VPN → Mac :8080 (FastAPI)
                                    ├── Web UI (React SPA)
                                    ├── API (/api/*)
                                    └── /ttyd/* (反向代理) → 127.0.0.1:7681 (ttyd)
                                                              └── tmux → Claude Code
```

## Tech Stack

| 層級 | 技術 |
|------|------|
| 前端 | React 19 + TypeScript + Vite + Tailwind CSS 4 + Zustand + shadcn/ui |
| 後端 | FastAPI + Uvicorn |
| 終端 | ttyd + tmux |
| 網路 | Tailscale（僅綁定 Tailscale IP，不綁 `0.0.0.0`） |
| Python 套件管理 | [uv](https://docs.astral.sh/uv/)（使用 `uv sync` 安裝、`uv add` 新增套件） |
| JS 套件管理 | npm |

## 開發指令

```bash
# 前端開發（Vite dev server，自動 proxy /api 到 Tailscale IP）
cd frontend && npm run dev

# 前端 build（輸出到 scripts/server/static/dist/）
cd frontend && npm run build

# 前端 lint
cd frontend && npm run lint

# Python 套件同步
uv sync

# 新增 Python 套件
uv add <package-name>

# 前端測試（Vitest，純函式單元測試）
cd frontend && npm run test:run

# 後端測試（pytest，解析邏輯單元測試）
uv run --group dev python -m pytest -v

# 啟動完整服務（ttyd + FastAPI + caffeinate）
./scripts/start-aircoder.sh

# 停止服務（保留 tmux session）
./scripts/stop-aircoder.sh
```

## 檔案結構

```
frontend/src/
├── api/            # API client 模組（apiFetch 基底在 client.ts）
├── components/
│   ├── layout/     # App shell（TopBar、BottomBar、MobileShell）
│   ├── terminal/   # 終端 UI（TerminalView、InputBar、QuickKeys 等）
│   ├── files/      # 檔案瀏覽器
│   ├── git/        # Git 面板
│   └── ui/         # shadcn 基底元件
├── hooks/          # use-sse.ts、use-push.ts
├── stores/         # Zustand stores（app、sessions、file-tree、git）
├── types/          # TypeScript 型別定義
└── lib/            # 工具函式

scripts/server/
├── main.py         # FastAPI 入口
├── config.py       # 環境設定（Tailscale IP、tmux 路徑、port）
├── routers/        # API 路由（tmux、sessions、files、git、push、watch、upload、ttyd_proxy）
├── services/       # 商業邏輯（tmux_service、file_service、git_service）
└── static/         # 前端 build 輸出 + PWA 資源
```

## 程式碼慣例

- **語言**：註解、docstring、log 訊息一律使用英文
- **UI 字串空格規則**：前端 zh-TW locale 的 UI 字串中，中文與英文之間、中文與數字之間加空格
- **前端命名**：元件 PascalCase（`TerminalView.tsx`）、hooks kebab-case（`use-sse.ts`）、stores kebab-case（`app-store.ts`）
- **後端命名**：模組 snake_case（`tmux_service.py`）、函式 snake_case、類別 PascalCase
- **TypeScript**：strict mode 啟用，所有函式需有型別標註
- **Python**：所有函式加 type hints，request/response 用 Pydantic `BaseModel`
- **路徑別名**：前端使用 `@/*` 對應 `./src/*`

## 重要注意事項

- **TCC 限制**：launchd agents 無法存取 `~/Documents/`、`~/Desktop/`、`~/Downloads/`（除非開啟 Full Disk Access）。自動啟動的 scripts 必須放在 `~/.local/bin/aircoder/` 之類的位置。
- **tmux 環境變數**：`tmux-attach.sh` 會在建立 tmux session 前清除 Claude Code 的環境變數，避免巢狀 session 衝突。
- **ttyd 反向代理**：ttyd 僅監聽 `127.0.0.1:7681`，搭配 `--base-path /ttyd`。所有存取透過 FastAPI 的反向代理 `/ttyd/`（port 8080），確保同源，解決 Safari PWA 的跨域 iframe 限制。
- **ttyd WebSocket subprotocol**：ttyd 要求 WebSocket 連線使用 `tty` subprotocol，反向代理需在兩端（browser↔proxy 和 proxy↔ttyd）都處理。
- **Apple Silicon vs Intel**：`tmux-attach.sh` 用 `which tmux` 自動偵測路徑。Apple Silicon 用 `/opt/homebrew/bin/tmux`，Intel 用 `/usr/local/bin/tmux`。
- **Tailscale-only binding**：FastAPI 僅綁定 Tailscale IP，Tailscale 沒啟動就會失敗，這是刻意設計——絕對不要綁 `0.0.0.0`。

## 已知陷阱

### iframe-bridge `_iframe` 註冊時機（iOS PWA 白畫面）

`iframe-bridge.ts` 的所有函式（`isTerminalReady`、`injectCSS`、`isWebSocketAlive`、`reloadIframe`、`sendToTerminal`、`scrollTerminal`）都依賴模組級變數 `_iframe`。

**陷阱**：`TerminalView` 在 `ttydUrl` 為空時 return null（不渲染 iframe）。如果用 `useEffect([], [])` 註冊，首次掛載時 iframe 不存在，`_iframe` 永遠是 null，bridge 函式全部靜默失效。必須用 **callback ref** 確保 iframe 渲染後立即註冊：

```tsx
// ✅ 正確
const iframeCallbackRef = useCallback((el: HTMLIFrameElement | null) => {
    iframeRef.current = el
    registerIframe(el)
}, [])

// ❌ 錯誤：首次掛載時 iframe 不存在，_iframe 永遠是 null
useEffect(() => { registerIframe(iframeRef.current) }, [])
```

**原則**：`TerminalView` 內部的 iframe 操作（`handleReload`、`forceRepaint`）一律用 `iframeRef.current` 直接操作，不要經過 bridge 的 `_iframe`。Bridge 函式只給外部元件（如 `InputBar`）使用。這樣即使 bridge 註冊有問題，TerminalView 的核心流程（尤其是 iOS PWA 冷啟動重繪）仍然可靠。

## 安裝與設定

詳見 [docs/setup-guide.md](docs/setup-guide.md)。

---
> Source: [FuMiaoKang/AirCoder](https://github.com/FuMiaoKang/AirCoder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
