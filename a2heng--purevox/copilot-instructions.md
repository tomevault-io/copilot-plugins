## purevox

> Windows / Linux 桌面应用 + Android 客户端：实时 AI 音频降噪 / 目标说话人提取 / 回声消除，支持本地麦克风和远程网络推流。

# PureVox — AI 麦克风降噪

Windows / Linux 桌面应用 + Android 客户端：实时 AI 音频降噪 / 目标说话人提取 / 回声消除，支持本地麦克风和远程网络推流。

**栈**: Python 3.8+（最低；Win7 需 3.8）+ PySide6 + 纯 C 共享库（gcc/mingw 编译，ctypes 绑定）+ ONNX Runtime（==1.11.1，模型 opset 13/14/15，均 ≤16）
**桌面入口**: `python run_pyside6.py`
**Android 入口**: `android/` — Kotlin + OkHttp + Opus JNI

---

## 维护准则（发版后管理，所有贡献者必读）

**功能最小化模型** —— 本项目的首要约束：

1. **一个功能只有一条规范实现路径**。禁止"功能 ABC 三种都能用"的设计——多套平行实现等于高维护成本。新增功能有多个可行做法时，只保留一种并写进文档，其余不进入代码。
2. **先扩展，再新建**。开新方法 / 新类 / 新文件之前，先搞清楚已有方法能否扩展：优先 改已有函数/类 → 加参数/加配置 → 复用既有抽象；确认确实无法扩展才允许新建，并在 CHANGELOG 说明为何不能扩展。
3. **被替代的实现不保留平行代码**。如 Linux 的 PortAudio/GStreamer/JACK、旧虚拟麦克风架构等已弃用方案，直接删除，不留"备选"。
4. **改动前先读对应模块，尊重既有设计意图**；删除功能需在 CHANGELOG 记录。

**本项目的单一实现路径（强制执行）**：

- Linux 音频采集/输出**只用原生 PipeWire**（`pvpipe`）
- 虚拟麦克风（Linux）= 单一生产者 + 双出口，全部健康，详见下方「Linux 音频架构」：
  ① 单声道 null-sink `purevox_out`（唯一写入口）；② 内置 monitor
  `purevox_out.monitor`（宽口径源）+ 非 monitor 真源 `purevox_mic`
  （`module-remap-source` 把 monitor 重映射而来，供 OBS 等"只列真源"软件）。
  不用 pw-loopback。**禁建第二路源用 `module-null-sink media.class=Audio/Source/Virtual`
  ——实测会把 pipewire-pulse 协议搞坏（pactl 报协议错误、plasma-pa context kaput、
  系统托盘清空，仅重启 pipewire-pulse 恢复）**，健康方案是 module-remap-source
- 音频格式一律 **F32 单声道 48kHz**（PipeWire 负责重采样与声道转换，模型永远拿 48k 单声道）
- 设备枚举只用 `pw-dump` 标准 introspection（`pvplatform.audio.pwpipe_client`）

---

## 运行 / 构建

**内嵌 Python 3.8（推荐，独立于系统环境）**：本项目可自带独立 Python 3.8，
与系统 Python（如 3.14）完全隔离。Windows 走 NuGet 下载预编译包；Linux 无预编译
3.8 可下，源码以 **git 子模块** `packages/cpython`（CPython@v3.8.20）锁定，
由引导脚本 out-of-tree 一次性编译。产物统一放 `packages/`。

- 克隆后先 `git submodule update --init --depth 1 packages/cpython` 拉子模块
- `./bootstrap_python38.sh`（Linux，幂等）→ 生成自包含 `packages/python38/` + 装依赖
- `./bootstrap_python38.ps1`（Windows）→ 生成 `packages\python38w\`（NuGet 完整版，含头文件/链接库）
- 内嵌解释器与系统 Python 互相独立；`packages/python38*`、`.py38-src/` 不进版本库（gitignore）

### Windows (PowerShell)

```powershell
chcp 65001
# 方式一（内嵌 3.8，推荐）：
powershell -ExecutionPolicy Bypass -File bootstrap_python38.ps1
# 方式二（系统 Python）：pip install -r requirements.txt -r requirements-win.txt
python run_pyside6.py
powershell -ExecutionPolicy Bypass -File build_win.ps1   # 打包产物目录 dist/PureVox/（自动用 packages\python38w\python.exe）
# 注：Windows 侧 aimic.dll 用 mingw gcc 编译（setup.py 走 CC 或 PATH 上的 gcc，
# 链接捆绑的 onnxruntime-win-x64-1.11.1）
```

### Windows 7 兼容性（实测结论，勿回退）

纯 PySide6 6.1.3 包无法直接跑 Win7——Qt 6.2+ 官方仅 Win10+，6.6.x import 即报
`DLL load failed ... 找不到指定的程序`；**PySide6==6.1.3 是最后一个支持 Win7 的版本**
（requirements.txt 已锁死并注释原因）。另外两个 Win7 缺失项必须在打包时补：

- **API-Set 转发 DLL**：捆绑的 onnxruntime.dll 还导入 `api-ms-win-core-libraryloader-l1-2-0.dll`
  和 `api-ms-win-core-processtopology-obsolete-l1-1-0.dll`（Win8+ 的 API-Set 由内核虚拟解析，
  Win7 与其构建机 System32 均无物理文件）。仓库在
  `packages/onnxruntime-win-x64-1.11.1/lib/` 固化两个 **x64 转发 stub**（导出符号转发到
  KERNEL32；生成材料见其下 `apiset/*.def`，用 mingw `x86_64-w64-mingw32-gcc -shared`
  复现，例如 `x86_64-w64-mingw32-gcc -shared stub.c apiset/libloader.def -o
  api-ms-win-core-libraryloader-l1-2-0.dll`）。`build_win.ps1` 打包时从仓库拷这两个
  stub 进 `_internal`，勿改回"从构建机 System32 拷"。
- **MSVC 运行库**：onnxruntime 依赖 MSVCP140/VCRUNTIME140 等，`build_win.ps1` 会把构建机
  System32 的 VC runtime 拷进包，避免 Win7 需单独装 VC++ redist。
- **打包瘦身勿删 `Qt6Qml.dll`/`Qt6Quick.dll`**：PySide6 核心库 `pyside6.abi3.dll`（所有
  Qt*.pyd 都链接它）**硬依赖 `Qt6Qml.dll`**，`build_win.ps1` 第 4 步若删除它，EXE 在 Win7
  启动即报 `DLL load failed while importing QtWidgets: 找不到指定的模块`（2026-08-09 实机
  pefile 分析确认；与 pyinstaller/hooks-contrib 版本无关）。瘦身只允许删 import 闭包外的
  `Qt6Pdf.dll`/`Qt6DataVisualization.dll`。

注意：四件套 wheel 名含 `abi3`；在线安装 PySide6==6.1.3 时会自动带对版本。

**Qt 6.5+ API 禁忌（Win7 实测 2026-08-09）**：`QStyleHints::colorScheme()` 与
`Qt.ColorScheme` 是 Qt 6.5+ 才有，Win7 锁定的 PySide6 6.1.3 上没有——启动即抛
`AttributeError: 'QStyleHints' object has no attribute 'colorScheme'`。深色判定一律用
**调色板亮度**（`app.palette().window().color().lightness() < 128`，Qt 6.1 即可用），
已固化在 `theme_colors.is_dark_current()`（2014 版），UI 主题同步 `_sync_theme_ui`
也走它。新增代码禁写 `styleHints().colorScheme()`。

**`.ps1` 脚本必须纯 ASCII（英文）**：`build_win.ps1` / `bootstrap_python38.ps1`
不含中文/非 ASCII/BOM。Windows PowerShell 5.1 对无 BOM 的 UTF-8 脚本按 ANSI
(cp1252/GBK) 误读导致语法错误（`chcp 65001` 只在本机掩盖）；中文文件名
（如 `用户手册.html`）在脚本里用通配符（`*.html`）引用，不写字面量。

### Linux

依赖因发行版而异（Ubuntu / Fedora / AOSC 包名不同，参考 `.github/workflows/ci.yml` 与 README）。AOSC 示例：

```bash
sudo oma install -y gcc pkgconf pipewire libpipewire-0.3-devel
# 内嵌 3.8（推荐）：
./bootstrap_python38.sh
./py38 setup.py build_ext --inplace --force   # 产出 libaimic.so + libpvpipe.so
./py38 run_pyside6.py
bash pack_deb.sh                              # deb → dist/PureVox-Linux-x64-<date>-release.deb
bash pack_rpm.sh                              # rpm → dist/PureVox-Linux-x64-<date>-release.rpm
bash pack_appimage.sh                         # AppImage → dist/PureVox-Linux-x64-<date>-release.AppImage
```

deb 布局：`/opt/purevox/` 放全部源码+libaimic.so/libpvpipe.so+模型+html+捆绑的 `libonnxruntime.so*`（1.11.1）；
`/usr/bin/purevox` 启动脚本（先导出 `LD_LIBRARY_PATH=/opt/purevox` 再 exec）。
`/usr/share/applications/purevox.desktop` + hicolor 图标。Depends 按 AOSC 包名（无 onnxruntime，
PT 已捆绑）。`server/opus.dll`
是 Windows 的，不入 deb。`.so` 为固定名 `libaimic.so`/`libpvpipe.so`，不用 `sysconfig.EXT_SUFFIX` 定位。
Linux 输入/输出/设备枚举/AEC 全原生 PipeWire；opuslib 缺失时 `pip install --user`（写进 Recommends）。

### Android

```powershell
$env:ANDROID_HOME = "D:\Android\Sdk"; $env:ANDROID_SDK_ROOT = "D:\Android\Sdk"
$env:ANDROID_NDK_HOME = "D:\Android\Sdk\ndk\27.0.12077973"
cd android
.\gradlew.bat assembleDebug    # 输出 android/app/build/outputs/apk/debug/
.\gradlew.bat installDebug     # 安装到设备
```

要求：JDK 17、SDK platform 34、NDK 27、CMake 3.22.1。首次编译需 Opus 源码放到
`android/opus-src/`（gitignore，JNI CMake 引用该路径）。

### CI（`.github/workflows/ci.yml`，精简为通用包 deb/rpm/appimage + 产物目录 + apk）

- **触发方式：只有 push tag（`v*`）才跑 CI**，分支 push 不触发（保持日常快速提交零成本）；需验证分支时可 `workflow_dispatch` 手动跑。release 由 tag 触发后自动附产物（见下 `release` job）
- `linux` job：容器矩阵只留 3 项，产出通用安装包——
  - `ubuntu-22.04`：gcc 编纯 C 库 + import 冒烟 + `pack_deb.sh` 出 deb + `pack_appimage.sh` 出 AppImage（best-effort，捆绑内嵌 python38）
  - `fedora`：编库 + 冒烟 + `pack_rpm.sh` 出 rpm
  - `python3.8`：官方 `python:3.8-bullseye`，验证纯 C 库在最低 Python 3.8 可编译、可 ctypes 装载
  - onnxruntime 用仓库内捆绑的预编译 1.11.1 SDK，**不 pip 装 onnxruntime**
- `windows` job：windows-latest + Python 3.8 + msys2/mingw gcc 编 `aimic.dll` + 语法/导入冒烟；`build_win.ps1`（PyInstaller one-folder）出 `dist/PureVox/`，CI 上传该目录（`actions/upload-artifact` 会自动压缩为 zip，命名 `PureVox-Windows-x64-<yyyy-MM-dd-HHmm>-release`）
- `android` job：ubuntu-latest 编 debug APK（JDK17 + SDK 34 + NDK r27）；下载 opus 源码到 `android/opus-src/`，产物改名 `PureVox-Android-arm64-<yyyy-MM-dd-HHmm>-debug.apk`
- `release` job：`needs` 三构建 job + `if: startsWith(github.ref,'refs/tags/')`，tag push 时下载全部产物，Windows 目录重打成 zip（`zip -9`），`gh release create` 把 deb / rpm / AppImage / Windows zip / APK 全部 attach
- **产物命名统一**：`PureVox-<平台>-<架构>-<yyyy-MM-dd-HHmm>-<release|debug>.<ext>`（Windows 上传目录由 CI 自动压缩 / Linux deb / rpm / AppImage 一律 release，Android 为 debug）。文件名时间戳 `yyyy-MM-dd-HHmm`；产物体内版本字段 = `yyyy.MM.dd.HHmm`（如 `2026.08.10.1517`，deb control / rpm / setup.py 一致，**由 tag 名 `v<yyyy.MM.dd.HHmm>` 推导**，避免并发 job 各自 `date` 导致产物版本不一）。
- **onnxruntime 预编译 SDK（双平台统一 1.11.1）**：Windows 用捆绑 `packages/onnxruntime-win-x64-1.11.1`；Linux/macOS 默认捆绑 `packages/onnxruntime-linux-x64-1.11.1`（`include/`+`lib/`）。setup.py 仍支持 `ORT_INCLUDE_DIR` / `ORT_LIB_DIR` 环境变量覆盖（CI/pip 场景，wheel 内 .so 带版本号后缀，需先建 `libonnxruntime.so` 软链接再 `-lonnxruntime`，运行时 `LD_LIBRARY_PATH` 指向 capi 目录）
- **Linux job 按发行版分开是刻意设计，勿合并成一个 job**（2026-08-10 决策）：deb 在
  Ubuntu、rpm 在 Fedora 产出，是因为 rpm 打包须依赖 `rpmbuild` 与真实 Fedora 包名解析
  （`pipewire-devel` 等），移到 Ubuntu 上构建可靠性下降；分开还有并行收益与故障隔离
  （一个发行版坏不掉其他产物）。AppImage 在 ubuntu job 内（best-effort，
  `continue-on-error: true`），捆绑内嵌 python38——该 job 需先装 `libssl-dev`
  （否则子模块编译出的 CPython 无 ssl 模块，pip 无网络，`bootstrap_python38.sh` 失败）
  与 `file`（appimagetool 打包必需），并确保 `PyAudio` 不在 Linux 依赖里
  （已移到 `requirements-win.txt`，否则编译缺 `portaudio.h` 让 AppImage 静默失败）。
- **CI 踩坑（实测细节补充，避免重踩）**：
  - 容器 job 在 checkout 前先装系统依赖（含 `git`）——REST API 下载不支持 submodules；
    AppImage job 才拉 `packages/cpython` 子模块
  - appimagetool 容器无 FUSE → 用 `--appimage-extract-and-run`；`.desktop` 要在
    AppDir 根目录放一份；图标用 `audio_icon_base_on_1024.png` 直接生成 256/512 png
  - `pipewire_client.c` 勿 include `<spa/param/audio/raw-utils.h>`（老 spa/bullseye
    没有）；`PW_KEY_TARGET_OBJECT` 需 0.3.64+，老版本回退 `node.target`
  - `pack_deb.sh` 末尾 `| head` 会 SIGPIPE(141) 使 `sh -e` 退出 → 补 `|| true`
  - Ubuntu 容器 pip 装 pillow 遇到匹配版本时用 `--break-system-packages` 兜底
    （`||` 回退普通安装），不再 `pip install --upgrade`；`python3-setuptools`
    由 apt/dnf 装机（sysdeps 里）供 setup.py
  - Android JNI `CMakeLists.txt` 注释必须用 `#`（CMake 不认 `//`）；`gradlew`
    无执行位，构建前 `chmod +x`
  - Windows pwsh 无 `\` 行继续符；`aimic.dll` 编译只走 `build_win.ps1` 一条路

---

## 架构

### 桌面端 (Python/C)

| 模块 | 职责 |
|---|---|
| `run_pyside6.py` | 单实例锁、启动入口，导入 `ui_pyside6.run_app` |
| `ui_pyside6.py` | 主 UI（PySide6）——面板布局、设备选择、模式切换、48kHz 检测弹框 |
| `audio_processor.py` | 核心音频引擎 —— `AudioThread`(全双工流)、`SpeakerCapture`(AEC loopback)、`RingBuffer`、设备枚举、TSE 参考录音工具(`_recorder`/`load_tse_reference`/`_wsola_time_stretch`) |
| `aimic.c` → `libaimic.so`（+ `aimic.py` ctypes 绑定） | C 音频核心（`audio_processor_new`/`denoise_new`/`tse_new`/`aec_new`/STFT/频谱/RingBuffer，ONNX Runtime C API，无任何 C++） |
| `pipewire_client.c` → `libpvpipe.so`（+ `pvpipe.py` ctypes 绑定） | 原生 PipeWire 桥（Linux，纯 C）—— PwBridge，F32 单声道 48kHz 协商 |
| `aimic.py` / `pvpipe.py` | ctypes 绑定层 —— 加载 libaimic.so / libpvpipe.so，Python 类/方法名与旧 pybind11 绑定完全一致（音频热路径仅做 list↔float 数组搬运） |
| `pvplatform/` | 平台抽象层 —— `audio/`(SpeakerCapture 三端、device_api、pwpipe_client)、`system/`(单实例/自启动/防火墙/虚拟麦克风，win+posix) |
| `server/` | 远程麦克风 HTTPS/WSS 服务器 —— `https_server.py`、`audio_bridge.py`(RemoteAudioSource)、`opus_codec.py`、`mdns_publisher.py`、`tls_manager.py` |
| `config_manager.py` | JSON 配置读写，启动时迁移旧 key；api_type/output_device 平台感知默认值 |
| `model_config.py` | ONNX 模型文件名常量 |
| `dialog_about.py` / `dialog_eq.py` / `dialog_tse_reference.py` | 关于 / 均衡器 / TSE 参考录音弹框（统一 `dialog_` 前缀） |
| `html/` | 浏览器端远程推流页面 —— `index.html`、`app.js`、`audio-capture.js`、`ws-client.js`、Opus WASM 编码器 |
| `dialog_vbcable_check.py` | VB-CABLE 虚拟麦克风检测弹框（仅 Windows；只检测不自动安装，给下载/教程指引，复选框默认勾选检测、取消即跳过不再提示） |
| `dialog_virtual_mic_linux.py` | Linux 虚拟声卡状态面板 —— 指示灯 + 双出口说明 + 手动「创建/清理」（启动不自动创建，`ensure_virtual_mic`/`remove_virtual_mic` 全幂等） |
| `build_win.ps1` / `pack_deb.sh` / `pack_rpm.sh` / `pack_appimage.sh` / `setup.py` | Windows 产物目录打包（PyInstaller，CI 上传自动压缩）/ Linux deb / rpm / AppImage 打包 / 纯 C 共享库构建（gcc，`build_ext --inplace` 产出 libaimic.so + libpvpipe.so） |

### Linux 音频架构（原生 PipeWire，强制）

数据流（本地）：麦克风源节点 → `PureVox-input` 流 → 降噪 → `PureVox-output` 流 → `purevox_out`（虚拟麦克风 sink）
监听：独立输出流 `PureVox-monitor` → 扬声器节点（同一路降噪音频）
AEC far-end：独立输入流 `PureVox-far`（`stream.capture.sink=tap 扬声器 sink 输出`，会话内创建/销毁，
采样率恒 48kHz 单声道）

- 格式协商 **F32 单声道 48000Hz**：PipeWire 内置重采样 + 声道转换，模型永远拿 48k 单声道，输出自动上混到目标设备声道数，不存在"一个通道一个模型 / 通道不匹配 / 采样率不齐"
- 虚拟麦克风（Linux 虚拟声卡）= **单一生产者 + 双出口**，实现见 `pvplatform/system/_posix.py`：
  - 生产者：单声道 null-sink `purevox_out`（`pw-cli create-node`，唯一写入口，
    `media.class=Audio/Sink`、`audio.position=[MONO]`、`object.linger=true`，
    node.description=「PureVox out」）。PureVox 降噪输出流 `PureVox-output` 只写入它。
  - 出口 1 `purevox_out.monitor`（monitor 源，宽口径）：系统录音/麦克风列表的一个
    PureVox 项，Audacity / 浏览器 / pavucontrol 等"列出全部源"的软件直接选用。
  - 出口 2 `purevox_mic`（真源，供 OBS 等**只列真源**的软件）：`module-remap-source`
    把 monitor 重映射而来（`media.class=Audio/Source`、无 monitor_of、
    `channel_map=mono`），自动取数、无第二生产者；node.description=「PureVox mic」。
  - 生命周期全幂等（检测-重置，有则不动）：`virtual_mic_ready()`（探测 sink 存在）→
    `ensure_virtual_mic()`（建 sink + 真源，先 `_kill_stray_loopbacks` 清旧架构残留
    pw-loopback）→ `remove_virtual_mic()`（卸载模块 + destroy 节点）。
  - **启动不再自动创建**（手动模型）：菜单「虚拟声卡」→ `dialog_virtual_mic_linux.py`
    状态面板（指示灯 + 双出口说明 + 「创建/清理」按钮）；ui 只做分发，
    与 Windows `dialog_vbcable_check.py` 同一原则。
- **禁用/踩坑**（违反任一即弄坏系统托盘/协议，见 `_posix.py` 头注释）：
  - `pw-loopback`：旧虚拟麦克风架构，已弃用，仅防御性 `pkill` 清残留。
  - `module-null-sink media.class=Audio/Source/Virtual` 建第二路真源：实测把
    **pipewire-pulse 协议状态弄坏**（pactl 报"连接失败：协议错误"、plasma-pa
    libpulse context kaput、系统托盘清空，仅重启 pipewire-pulse 恢复）。真源必须用
    `module-remap-source`。
  - **PortAudio 直接打开 null-sink**（须经 monitor 引用）：触发 PipeWire ALSA 插件
    堆崩溃（`free(): corrupted unsorted chunks`）。
  - **重启 pipewire-pulse "修托盘"**：plasma-pa 的 libpulse context 变 kaput、托盘
    清空，KDE 不自动重连。
  - remap-source 会强制覆盖 node.description（显示 "Remapped ... source"），set-param 改不掉。
- **pw_stream 线程约束**：所有 pw_stream 操作必须经 `_run_on_loop` 在 PipeWire 主循环线程
  执行（`pw_loop_invoke` + 条件变量同步；block 参数不可靠会竞态）。
- **进程回调（数据线程）禁止加锁/分配**——pvpipe 用无锁 SPSC 环形缓冲（输入环满丢最旧、
  输出环满丢新），Python 线程读→降噪→写（2s 缓冲吸收调度抖动），回调只搬数据。
- 设备列表（pw-dump）：PureVox 自身输入 = Audio/Source 物理麦克风（排除 PureVox-*、purevox
  源[那是对外输出，选它当输入会回授]、幻影路由 `api.alsa.path` 未指定具体设备如 `hw:sofhdadsp`
  无 `,N`、error 节点）；输出 = Audio/Sink 节点（扬声器 + `purevox_out`）
- VU 电平显示**降噪输出峰值**（`_pw_loop` 里取 `out`，勿改成输入 `data`）
- UI 下拉框直接显示节点名（node.name），真实节点名存 userData，读下拉框一律走 `_combo_value()`

### Android 端 (Kotlin)

| 模块 | 职责 |
|---|---|
| `MainActivity.kt` | 主界面 —— 服务器发现、连接、推流控制、VU 显示、调试信息、RTT 追踪 |
| `audio/AudioCapture.kt` | AudioRecord 采集 48kHz/16bit，帧大小 960 (20ms) |
| `audio/OpusEncoder.kt` | JNI 调用 native opus 编码 |
| `network/WsClient.kt` | OkHttp WebSocket 客户端，base64 Opus 推流，ack RTT 追踪 |
| `network/TlsHelper.kt` | 自签名证书信任 |
| `discovery/MdnsDiscovery.kt` / `SubnetScanner.kt` | mDNS 发现 + 子网扫描备用 |
| `service/StreamService.kt` | 前台服务保活 + WakeLock |
| `VuMeterView.kt` | 自定义 VU 表绘制 |

### 网络推流协议

```
浏览器/Android → WSS → Python 服务器 → audio_processor pipeline → 扬声器

客户端 JSON: {"type":"audio","data":"<base64 opus>","seq":N,"timestamp":T}
服务器 ACK:  {"type":"ack","seq":N}
服务器 API:  GET /api/status → {"sample_rate":48000, "active_clients":N, ...}
```

帧大小 960 samples (20ms @48kHz) —— Opus 编码器 (JS WASM / Android JNI) 与 Python 解码器对齐。

---

## 命名规范

### 品牌名
**品牌名统一为 `PureVox`**（P、V 大写）。所有用户可见文本——窗口标题、UI 文案、日志、关于对话框、README、用户手册、CHANGELOG、菜单——一律用 `PureVox`，禁止 `Purevox` / `purevox` / `PUREVOX` 等变体。

### 代码内部标识符
- 含品牌名的 Python 类/标识符统一 `PureVox...`（如 `PureVoxServer`）
- 其它标识符遵循工程约定第 4 条（Python snake_case、C++ PascalCase、Kotlin camelCase）

### 平台强制小写（非品牌变体，勿改）
各处用户可见/系统标识中的 `Purevox` 变体已统一为 `PureVox`（注册表 Run 键、防火墙规则名、单实例 Mutex 名、发行产物 `PureVox.exe`/`PureVoxMic.apk`、WakeLock `PureVoxMic:AudioWakeLock`、settings.gradle rootProject、README/手册/CSS 注释）。以下标识属**平台/协议强制小写**，改小写会破坏功能或违背平台惯例：
- Android 包名 `com.purevox.mic`（Java 包名惯例 + JNI 函数名 `Java_com_purevox_mic_*` 必须与包名逐字符匹配，含 `namespace`/`applicationId`/布局类引用）
- mDNS 服务类型 `_purevox._tcp.local.`（DNS SRV 按 RFC 小写约定）
- 用户数据目录 `~/.purevox/`、日志名 `purevox_*.log`、CA 证书 `purevox-ca.crt`（POSIX 小写路径惯例）
- 模型代号 `purevox9`（内部模型代号）
- 浏览器 localStorage key `purevox_mic_id` / `purevox_theme`
- JNI/CMake 内部名（`purevox_opus_jni`、opus_jni.c 的 native 函数，随包名）

---

## 工程约定

1. **所有设备强制 48kHz** — 启动前逐设备检测，失败弹框阻止，不做重采样或半双工回退。
2. **模型规格 48kHz / 2048 NFFT / 1024 hop** — 任何缓冲区/块大小与此冲突的以此为准。
3. **配置 key 无 API 前缀** — 用 `input_device` / `output_device` / `monitor_device`，不用 `WASAPI_` 前缀。
4. **命名** — Python: snake_case 方法和变量；C++: snake_case 方法和 PascalCase 类；Kotlin: camelCase。
5. **错误处理** — 内部用 `try/except` + `_module_log()` 记录，不冒泡到 UI 线程；UI 用 `QMessageBox` / `QDialog` 提示。
6. **日志** — 统一 `logger.py` 的 `Logger` 类，层级 `dev`/`msg`/`warn`/`err`。
7. **主程序禁止依赖 numpy/torch** — 所有频谱/FFT 在 C++ 端完成；纯 Python 仅做 Qt GUI 和数据中转（`List[float]`）。
8. **Android 主题跟随系统** — `Theme.MaterialComponents.DayNight.NoActionBar`，亮色/深色自动切换。
9. **品牌拼写规约** — 品牌名一律 `PureVox`；`purevox` 全小写仅限平台/协议强制标识（见命名规范），改大小写视为破坏行为。
10. **许可证头** — 每个源码文件顶部必须带 GPL-3.0 版权头 + 模型声明 + `SPDX-License-Identifier: GPL-3.0-or-later`（照抄 `audio_processor.py` 顶部，按 `#`/`//` 注释风格替换）；新增文件也必须带。
11. **README 双语约定** — 默认中文 `README.md`，英文单独 `README_EN.md`；改文件名/平台结构/打包命令时两处必须同步，不得改名或删除。
12. **C 源码禁止中文（纯 ASCII）** — `*.c`/`*.h` 的注释与字符串一律英文 ASCII，禁止任何非 ASCII 字符（含中文注释）。`aimic.c` 由 Windows 的 mingw gcc 编译，中文注释/非 UTF-8 编码会破坏 Windows CI；`pipewire_client.c` 虽仅 Linux 编译也应遵守。新增/修改 C 代码时不得加入中文注释（历史中文注释需逐步清理）。
13. **弹框/检测面板文件统一 `dialog_` 前缀** — 独立弹框一律 `dialog_*.py`（如 `dialog_about.py`、`dialog_eq.py`、`dialog_tse_reference.py`、`dialog_vbcable_check.py`）；新增弹框模块必须遵循此前缀，不得用 `*_check.py` / `*_dialog.py` 等变体。

---

## 注意事项

- **AEC SpeakerCapture**: Linux 端 AEC far 走 pvpipe `set_far(sink_name, True)`（`stream.capture.sink` tap 扬声器 sink 输出，恒 48k 单声道免重采样，会话内创建/销毁）。Windows 用 `GetMixFormat` 获取设备原生格式（WASAPI loopback 共享模式**必须用引擎 MixFormat**）；音频引擎 `audio_processor_set_aec_far_sample_rate()` 将 far-end 重采样到 48kHz。
- **网络模式缓冲**（未做低延迟优化，目标以稳为主，不追求最小延迟）:
  - `_output_buffer`: `RingBuffer(48000)` + 预填充 `1024*3` (64ms)
  - `_network_loop TARGET_ACC`: `1024*5` (107ms), `MAX_ACC`: `1024*8` (171ms)
  - 速率补偿: 输出缓冲 >128ms 时主动丢弃多余帧
- **本地（Linux）低延迟是唯一做过优化的一条**（非网络）：
  - `create_stream` 显式发 `SPA_PARAM_Buffers`（4096B=1024 样本）→ 输出流缓冲
    从 12288 样本（256ms）降到 1024 样本（~21ms）/hop，见 CHANGELOG 2026-08-10
  - `RING_CAPACITY`（pipewire_client.c）从 2s(96000) 收到 4 hop(4096/85ms) 封顶，
    输入环稳态保持 ~0；本地全链路延迟即 ~1 帧 + hop
  - 网络模式不受此优化影响：仍走热路径外缓冲，该条数值不适用于本地
- **旧配置兼容**: `WASAPI_input_device` → `input_device` 等迁移在 `ConfigManager.load_config()` 中。

### 长时间运行稳定性观察（2026-08-10，代码走查记录，尚未实测）

**结论**：2 小时连续运行**声音本身不会劣化**（无累积延迟/爆音/数值漂移），但存在一个内存型长期隐患和一个事件型弱点：

- **无数值溢出/回绕（安全）**：C/Python 环形缓冲读写位置全用 `size_t`/无界 int（单调递增不回绕成负数），满时丢最旧/丢新有界。无 float 位置累加（2^24 精度丢失点不存在）。最接近的 `int` 计数器是 TSE 调试帧号（~530 天回绕，与音频无关）。AEC/EQ 状态皆为有界信号值，libsamplerate 用 double 长程稳定。
- **无延迟累积（安全）**：网络模式 `acc` 硬顶 171ms（`MAX_ACC=1024*8`）、输出缓冲 >128ms 即丢（`TARGET_OBUF`）、本地 SPSC 环 85ms 有界。水位被阈值夹牢，不会单向漂移。
- **无内存泄漏（正常路径，安全）**：ONNX session 只创建一次（无周期性重建），OrtValue 每帧创建/释放配对；AEC far 流创建/销毁对称并释放 `far_ctx_`；进程回调无 malloc/free。
- **⚠️ 高危内存隐患（已确认，待修复）**：`_network_loop` 每帧走 `process_pipeline`（audio_processor.py:952）→ C 侧无条件向 `viz_in_48k_`/`viz_out_48k_` 追加（aimic.c:2143/2146/2152/2155），而唯一排空点被 `_viz_enabled` 门控（audio_processor.py:978-985），窗口隐藏到托盘即置 False（ui_pyside6.py:2209）。FVec 只扩不缩 → **约 1.4GB/小时**增长，2h 可达 GB 级 OOM。范围**仅**网络模式；本地 `_pw_loop`/全双工走 `process`/`process_eq_only`，不经 viz append，不受影响。修复方向：drain 与 `_viz_enabled` 解耦，或 C 侧给 viz 缓冲设上限超限丢旧。
- **⚠️ 事件型弱点（非时间累积）**：pipewire_client.c 无 `pw_core_add_listener`/core error/lost 监听，无自动重连。运行中 USB 拔插/PipeWire 重启 → 流静默冻结且 `pvb_active()` 仍返回 true，无恢复。优先级低于 viz 泄漏。
- **低危**：网络补偿不对称——欠载时重复上一帧（audio_processor.py:739-746）+ 睡眠单向减速，偶发 21.3ms 冻结/丢片伪影（次/数十分钟级）；`_network_loop` 每帧 `acc = acc[HOP:]` list 切片分配抖动；输入 SPSC 环"满时生产者也会写 `r_`"（pipewire_client.c:92-93）违反纯 SPSC 契约，但仅过载瞬时自愈。

---

## 许可证

- 源码 **GPL-3.0**（SPDX: `GPL-3.0-or-later`），见 `LICENSE`
- 内置 AI 模型（`*.onnx`）**不随 GPL 授权**，归 a2heng 所有，禁止提取用于其他项目，仅随 PureVox 经授权使用 → 见 `MODEL-LICENSE.md`
- 作者另有 MIT 模型仓库可自由使用：`lightweight-denoise-48k` / `lightweight-aec-48k`（README 已写）

---
> Source: [a2heng/PureVox](https://github.com/a2heng/PureVox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
