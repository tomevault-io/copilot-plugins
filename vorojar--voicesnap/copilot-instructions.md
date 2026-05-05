## voicesnap

> VoiceSnap/                          ← Git 仓库根目录

# VoiceSnap 项目指南

## 项目结构

```
VoiceSnap/                          ← Git 仓库根目录
├── VoiceSnapGo/                    ← 主版本 (Win10/11 + macOS), Go + Wails v3 + Svelte
├── VoiceSnapWin7/                  ← Win7 精简版, 纯 Go + Win32 API, 无 Wails
├── README.md
├── CHANGELOG.md
└── .gitignore                      ← 白名单模式, *.exe/*.dll/*.zip/*.onnx 等已忽略
```

## Git 仓库

- 地址: https://github.com/vorojar/VoiceSnap
- 分支: main
- .gitignore 是白名单模式 (默认忽略所有, 只放行源码目录)
- 二进制文件 (exe/dll/zip/onnx/sfx/syso) 全部不入库

---

## VoiceSnapGo (主版本)

### 环境

- Go 1.26+ (系统默认, PATH 里的 go)
- Wails v3 CLI alpha.71 (`wails3` in PATH)
- Node.js v20.11.1, npm 10.2.4
- LLVM MinGW UCRT 工具链 (CGO 编译器)
- CGO_ENABLED=1 必须开启 (sherpa-onnx 是 C 库)

### 构建命令

```bash
export PATH="/usr/bin:$PATH:/c/Program Files/Go/bin:$HOME/go/bin:/c/Users/voroj/AppData/Local/Microsoft/WinGet/Packages/MartinStorsjo.LLVM-MinGW.UCRT_Microsoft.Winget.Source_8wekyb3d8bbwe/llvm-mingw-20251216-ucrt-x86_64/bin"
export CGO_ENABLED=1
export GOPROXY=https://goproxy.cn,direct
cd D:/mycode/funasrnano/VoiceSnap/VoiceSnapGo
windres voicesnap.rc -o voicesnap.syso
go build -ldflags "-H windowsgui -s -w" -o voicesnap.exe .
```

### 运行时 DLL (放在 exe 旁边)

- `onnxruntime.dll` (~15MB) — 来自 sherpa-onnx-go-windows v1.12.24
- `sherpa-onnx-c-api.dll` (~4MB)
- `sherpa-onnx-cxx-api.dll` (~248KB)
- 来源: `~/go/pkg/mod/github.com/k2-fsa/sherpa-onnx-go-windows@v1.12.24/lib/x86_64-pc-windows-gnu/`

### 关键设计

- 托盘图标: macOS 用 `SetTemplateIcon(trayIconMac)` 单色模板; Windows 用 `SetIcon(trayIconWin)` 彩色 ICO + `SetTooltip`
- 首次启动显示设置窗口, 之后启动隐藏到托盘 (config.FirstLaunchDone 标记)
- 历史记录过滤: 去掉标点后只有 1 个词的不记录 (如 "The.", "Okay.")

---

## VoiceSnapWin7 (Win7 专用版)

### 核心限制

- **必须用 Go 1.20** — Go 1.21+ 的运行时用了 Win8+ API (CreateFile2, WaitOnAddress)
- **必须用 onnxruntime-for-win7** — 官方 onnxruntime v1.23 用了 Win8+ API
- **CPU 推理** — DirectML 需要 Win10, Win7 不支持
- **无 Wails/WebView2** — WebView2 不支持 Win7, 所以用纯 Win32 系统托盘

### 环境

- Go 1.20.14 安装在: `D:/go1.20/go`
- 同样使用 LLVM MinGW UCRT 工具链
- sherpa-onnx-go v1.12.20 + sherpa-onnx-go-windows v1.12.20 (降级版, ORT API v17)

### 构建命令

```bash
cd "D:/mycode/funasrnano/VoiceSnap/VoiceSnapWin7"
windres voicesnap.rc -o voicesnap.syso
GOROOT="D:/go1.20/go" CGO_ENABLED=1 GOPROXY=https://goproxy.cn,direct \
  PATH="/c/Users/voroj/AppData/Local/Microsoft/WinGet/Packages/MartinStorsjo.LLVM-MinGW.UCRT_Microsoft.Winget.Source_8wekyb3d8bbwe/llvm-mingw-20251216-ucrt-x86_64/bin:/usr/bin" \
  "D:/go1.20/go/bin/go.exe" build -ldflags "-H windowsgui -s -w" -o voicesnap_win7.exe .
```

### SFX 打包

```bash
cd "D:/mycode/funasrnano/VoiceSnap/VoiceSnapWin7"
bash build_sfx.sh portable
```
- 输出: `VoiceSnap_Setup.exe` (~6.7MB, 不含模型)
- 包含: voicesnap_win7.exe + 91 个 DLL (含 74 个 api-ms-win-* Win7 shim)
- 用户双击解压到同级 VoiceSnap/ 目录并自动启动
- 首次启动自动下载模型 (~152MB)

### 运行时 DLL (91 个, 全部打包进 SFX)

来自两个来源:
1. **sherpa-onnx-go-windows v1.12.20** — `sherpa-onnx-c-api.dll`, `sherpa-onnx-cxx-api.dll`
2. **onnxruntime-for-win7** (GitHub: yycmagic/onnxruntime-for-win7, ORT v1.20)
   - `onnxruntime.dll` (wrapper, 不含 CreateFile2)
   - `onnxrt64.dll` (实际 ORT)
   - `ucrtbase.dll` + VC 运行时 (vcruntime140.dll, msvcp140.dll 等)
   - 74 个 `api-ms-win-core-*` / `api-ms-win-crt-*` shim DLL
   - 下载位置曾存放在: `/tmp/ort-win7/x64/`

### go.mod 注意事项

- `go 1.20` — 不能升级, Go 1.20 不认识 `go 1.24.0` 等
- `go mod tidy` 会尝试升级 go 版本 — 必须手动改回 `go 1.20`
- darwin/linux engine stub 文件不能 import sherpa-onnx, 否则 mod tidy 会拉平台包

### Win7 版特有功能

- 原生 Win32 系统托盘 (Shell_NotifyIcon)
- 托盘菜单: VoiceSnap (win7) / 设置热键 / 提示音 / 开机自启动 / GitHub / 退出
- 热键对话框: Win32 窗口按键捕获
- 模型自动下载: 后台 goroutine 下载 + Win32 进度条窗口 + 主线程消息泵
- startup 包: 直接用 advapi32 注册表 API (不依赖 golang.org/x/sys/windows/registry, 那个要 Go 1.24)

---

## 通用注意事项

- GOPROXY=https://goproxy.cn,direct (中国网络)
- `.syso` 文件不入 git, clone 后需 `windres voicesnap.rc -o voicesnap.syso`
- Resource icon ID 必须是 3 (Wails 自动检测)
- `getAsyncKeyState()` 返回 `uint16` (不是 int16), 0x8000 会溢出 int16
- Git Bash 下 taskkill 需要 `//F //IM` (双斜杠转义)
- 模型文件: `models/sensevoice/model.int8.onnx` + `tokens.txt` (~152MB)
- 模型下载地址: `http://www.maikami.com/voicesnap/sensevoice.zip` (主) / ModelScope (备)

---
> Source: [vorojar/VoiceSnap](https://github.com/vorojar/VoiceSnap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
