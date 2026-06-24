## frida-hookers

> - 项目名：`Frida-Hookers`

# AGENTS.md

## 速览

- 项目名：`Frida-Hookers`
- 主要入口：`app_gui.py`
- 次入口：`hookers.py`
- 当前产品形态：**GUI-first**；CLI 仍可用，但最近的用户可见行为都以 GUI 为准
- 核心共享状态：`core/models.py` 中的 `HookerContext`
- 主要目录分工：
  - `core/`：设备、工作区、会话、RPC、APK 扫描等服务层
  - `ui/`：GUI 组装、控制器、错误展示、日志面板、线程切回与后台 worker
  - `tests/`：本地轻量回归测试
  - `workspaces/`：按包名隔离的运行时工作区，不是框架源码
  - `hookers/js/`：GUI 当前实际使用的内置 Frida 脚本、RPC 资产、参数化脚本模板、清理 warp；也包含 `bypass_frida_svc_detect.js`、`replace_dlsym_get_pthread_create.js` 这类无单独快捷按钮的专项内置脚本
  - `mobile-deploy/`：设备侧二进制与本地辅助工具

## 文档编码约定

- 项目文档默认统一使用 **UTF-8（无 BOM）**
- `.editorconfig` 已显式约定：
  - `charset = utf-8`
  - `end_of_line = lf`
- `*.md` 同样按 UTF-8 保存，不单独使用 GBK / ANSI / UTF-16
- 如果 shell 里看到中文乱码，不要先假设文件已损坏；优先用 Python 按 UTF-8 读取确认，再判断是否只是终端显示编码问题

## 只看少数文件时，优先看这些

1. `app_gui.py`
   - GUI 运行时总装配入口
   - 构建 `HookerContext` 与各个 service
2. `ui/main_window.py`
   - 主窗口 UI 构建与公共 UI helper
3. `ui/composition.py`
   - controller / presenter 的实例化与 signal wiring
4. `core/models.py`
   - 理解共享状态、当前 App、当前会话、工作区目录的最好入口
5. `ui/quick_hook_actions.py`
   - GUI 快捷脚本按钮的共享配置表入口
   - 普通快捷脚本主要看这里；参数化按钮还要联动 `ui/hook_runtime.py` 与 `ui/composition.py`
6. `core/device_service.py`
   - 设备准备、Frida server 生命周期、App 运行态 façade
7. `ui/terminal_console.py`
   - GUI 右侧内嵌 CLI 终端控制器
   - 命令解析、历史记录、CLI 模式切换、实时补全、Tab 补全应用、REPL transcript 风格回显、App 级 CLI 分发
8. `ui/cli_terminal_view.py`
   - 黑色终端本体输入控件
   - 负责活动输入行、prompt 保护、历史区只读保护、命令提交信号、Tab/上下键转发

如果再多看几个文件，优先看：

- `core/session_service.py`：attach/spawn、脚本加载、会话生命周期
- `core/workspace_service.py`：工作区初始化、本地 APK、内置脚本清理、运行时脚本落地
- `core/rpc_service.py`：RPC 检索与 Hook 脚本生成
- `ui/error_presenter.py`：GUI 统一错误展示策略
- `ui/app_workflow.py`：准备环境、选 App、初始化工作区
- `ui/hook_runtime.py`：开始注入、停止 Hook、停止 server、重启 App、参数化快捷脚本弹窗
- `ui/log_panel.py`：右侧日志渲染、过滤、搜索、落盘
- `ui/terminal_console.py`：右侧 CLI 终端命令层、补全、历史、CLI 模式切换与命令分发
- `ui/cli_terminal_view.py`：黑色终端本体、活动输入行、prompt 保护、历史区只读保护与输入事件转发
- `ui/ui_thread_dispatcher.py`：把 worker / Frida 后台线程回调安全切回 GUI 主线程
- `ui/workers/*.py`：GUI 后台动作执行器；区分设备准备、Hook 启动、工作区初始化与通用一次性动作

## 当前架构模型

### 共享状态

- `HookerContext` 是唯一共享运行时上下文
- service 通过同一份 context 协作
- controller 尽量不直接互相 import 调用，而是通过最小能力注入协作

### `core/` 分层

- `device_service.py`
  - 当前已经是 **façade + 3 个内部职责组件**：
    - `_DeviceBridge`：ADB/su、远端文件、forward
    - `_FridaServerManager`：Frida device、server 生命周期、端口、probe、日志
    - `_AppRuntimeInspector`：App 元数据、前台状态、PID、应用列表、radar.dex
  - 外部世界仍只通过 `DeviceService` 调用
- `workspace_service.py`
  - 按包名创建/补齐工作区
  - 准备本地 APK
  - 管理工作区脚本、旧内置脚本清理、解密输出与参数化运行时脚本落地
- `session_service.py`
  - attach / spawn / script load / resume / stop / detached 处理
  - 注入宿主侧 `Hookers` 结构化日志桥
- `rpc_service.py`
  - 复用 `hookers/js/rpc.js` 做高频对象/页面/组件查询
  - 生成 Hook 脚本
- `apk_scan_service.py`
  - 本地 `ApkCheckPack.exe` APK 扫描
  - 只负责校验输入路径并执行外部 exe，返回 stdout/stderr/returncode
- `errors.py`
  - 结构化错误模型
  - 统一转成 GUI 可消费的 `UiErrorPayload`

### `ui/` 分层

- `main_window.py`
  - 负责 UI 构建
  - 持有公共 UI helper：`set_busy()`、`set_status_text()`、脚本列表相关 helper、`toggle_log_focus_mode()`、`closeEvent()`
  - 负责快捷按钮分组布局、专项按钮 tooltip、当前 Attach / Spawn 模式徽章
- `quick_hook_actions.py`
  - 快捷脚本按钮的配置驱动入口
  - 按钮文案、脚本名、日志模板、tooltip、功能分组等基础配置都在这里收口
  - 参数化按钮也会在这里登记，但仍需要 `hook_runtime.py` / `composition.py` 的专门分支
- `composition.py`
  - 负责 `MainWindow` 的 controller / presenter 装配
  - 负责 signal wiring
  - 不承载业务逻辑
- `error_presenter.py`
  - 统一消费 `UiErrorPayload`
  - 负责 warning / critical / 状态栏 / 日志 / 弹窗展示策略
- `app_workflow.py`
  - 准备环境并刷新 App
  - 前台 App 自动选中
  - 工作区初始化
- `hook_runtime.py`
  - attach / spawn 开始注入
  - 停止 Hook
  - 停止 Frida Server
  - 重启 App
  - detached 后状态恢复
  - 为特定快捷脚本提供参数弹窗与运行时脚本生成
- `rpc_tools.py`
  - RPC 工具动作、结果展示、结果弹窗
- `terminal_console.py`
  - 右侧 GUI 内嵌 CLI 终端命令层
  - 当前支持 CLI 模式切换、当前 App 命令模式、顶层 App 选择辅助命令、上下键历史、实时补全、Tab 补全应用与 REPL transcript 风格回显
  - 终端查询结果统一写入现有日志面板，不单独维护第二套输出缓冲
- `cli_terminal_view.py`
  - 黑色终端本体控件
  - 负责活动输入行、prompt 保护、历史区只读保护、命令提交信号、Tab/上下键转发与日志追加时的 prompt 保留
- `apk_scan.py`
  - 左侧 APK 扫描流程
- `log_panel.py`
  - 右侧日志缓冲、过滤、搜索、增量渲染、落盘
- `ui_messages.py`
  - 高频 GUI 文案
- `controller_types.py`
  - GUI 侧协议、typed payload、callback alias
  - 把 MainWindow / controller / service 之间的依赖边界显式类型化
- `ui_thread_dispatcher.py`
  - 用 Qt Signal 把任意 Python 回调安全切回 owner 所在线程
  - 避免 worker、Frida 回调线程直接操作 Qt 控件
- `workers/action_worker.py`
  - 通用一次性后台动作执行器
  - 被 RPC、APK 扫描、停止 Hook、重启 App 等流程复用
- `workers/device_worker.py`
  - 后台执行 connect / start_frida_server / deploy_radar_dex / refresh_applications
- `workers/hook_worker.py`
  - 后台执行 attach / spawn 启动链路
  - 启动失败时负责兜底 stop_active_session 清理
- `workers/workspace_worker.py`
  - 后台执行 prepare_app_context + ensure_workspace
  - 不强制把目标 App 拉到前台

## 当前行为真相（非常重要）

- GUI 是当前主要产品表面；当代码与 README 不一致时，以代码为准
- Attach **只会** attach 到已经运行的 App，不会主动拉起目标 App
- 选中 App 不会自动初始化工作区
- 只有点击 `初始化工作目录并刷新列表` 才会真正：
  - 生成工作区目录
  - 准备本地 APK
- GUI 启动时默认脚本根目录是 `hookers/js`；选中 App 后会切到 `workspaces/<package>/js`
- GUI 快捷按钮默认从 `hookers/js` 读取内置脚本；但参数化脚本会先在 `workspaces/<package>/js` 生成运行时副本再启动
- 左侧脚本列表在未选 App 时指向 `hookers/js`；选中 App 后默认指向 `workspaces/<package>/js`，但用户仍可手动切换脚本根目录
- `workspaces/<package>/js` 中已有脚本在重复初始化工作目录时会被保留；当前初始化流程不会主动清理或删除原有脚本，只会补齐辅助资源、刷新本地 APK，并把 `hookers/js` 里的内置脚本复制到工作区，文件名前缀为 `内置-`
- `准备环境并刷新 App` 后，GUI 会尝试识别当前前台 App；如果该包名存在于刷新后的应用列表里，则自动选中
- 如果没有可识别的前台 App，则保持 App 选择为空
- 当前固定使用单一设备侧 server：
  - 本地文件：`mobile-deploy/rusda-server-16.2.1-android-arm64`
  - 远端名称：`rusda-16.2.1`
- `remote_frida_dir` 当前只允许：
  - `/data/local/tmp`
  - 或 `/data/local/tmp` 下的受管子目录
- 当受管 rusda 已运行且 probe 成功时，准备环境阶段会跳过重复清理/重启
- 如果远端 rusda 文件已存在，但需要重启服务，则复用已有文件，不重复上传
- GUI 退出不会自动删除远端 rusda 文件
- 左侧有显式 `停止 Frida Server` 按钮
- 左侧 APK 扫描与当前 App、工作区、会话无关，是独立本地文件工具
- APK 扫描调用：`mobile-deploy/ApkCheckPack.exe -f <apk>`
- GUI 当前已经完成 controller 化拆分，`MainWindow` 不再直接承载主要业务逻辑
- 快捷脚本按钮当前已配置化；基础配置统一收口于 `ui/quick_hook_actions.py`，参数化按钮再由 `ui/hook_runtime.py` / `ui/composition.py` 补专门交互与运行时脚本生成
- 快捷按钮区当前已经按功能分组显示，不再是单一平铺网格
- 右侧日志区当前使用黑色终端本体内嵌 REPL 输入，不是外部真实 shell，也不是嵌入 `PromptSession`
- `jni_method_trace`、`trace_init_proc`、`find_anti_frida_so`、`bypass_root_detect`、`bypass_vpn_detect` 当前带简短 tooltip，用于提示参数要求或专项用途
- Attach / Spawn 单选区下方当前有独立模式徽章，会实时显示 `当前模式：Attach` / `当前模式：Spawn`
- 需要点击 `进入 CLI 模式` 后，才能在黑色终端底部 prompt 后直接输入命令
- `CLI 模式` 按钮当前位于 `运行日志` 标题行右侧；退出 CLI 模式后终端恢复只读浏览，不清日志、不清历史、不影响当前 Hook / 异步任务
- 终端提示符当前显示在黑色终端本体内部：
  - 未选 App：`hooker >`
  - 已选 App：`<package> >`
- 终端命令回显当前采用 REPL transcript 风格：
  - `[CMD] hooker > help`
  - `[CMD] <package> > attach demo.js`
- 快捷按钮当前新增了两个“参数化脚本”入口：
  - `jni_method_trace.js`：弹窗输入目标 `so`，生成 `workspaces/<package>/js/jni_method_trace.runtime.js`
  - `trace_init_proc.js`：弹窗输入 `so/startAddr/endAddr`，生成 `workspaces/<package>/js/trace_init_proc.runtime.js`
- `高级 Frida 启动` 当前支持参数化模板脚本：
  - `jni_method_trace.js`
  - `trace_init_proc.js`
- 在高级启动器里添加这两个参数化模板时，会**立即弹参**、自动生成 runtime 脚本，再把 runtime 条目加入右侧启动顺序列表
- 高级启动器右侧已选列表里的参数化 runtime 项当前支持 **重新配置参数**
  - 重新配置会复用该项自己的 runtime 实例文件
  - 不同参数化项即使模板相同，也不会再互相覆盖
  - 右侧列表与启动顺序日志仍显示用户友好的固定名称，不暴露内部实例 key
- `detect_network_stack`、`hook_register_natives`、`find_anti_frida_so` 现在都**跟随当前 Attach / Spawn 选择**，不再固定 Spawn
- `detect_network_stack.js` 仍然是短时观察脚本，会自动请求停止 Hook；其余快捷脚本默认仍是手动停止
- GUI 终端当前支持的命令分两类：
  - 当前 App 命令：`help/h`、`ls`、`pid`、`uid`、`activitys/a`、`services/s`、`object/o`、`oe`、`view/v`、`gs`、`attach`、`spawn`、`restart`、`stop`
  - 顶层辅助命令：`apps`、`refresh`、`select <package>`
- 项目内命令始终优先；未命中项目内命令时，当前只对**包含 `frida` 的外部命令**放行
- 这些外部命令当前通过 `QProcess` 以**单次 PowerShell 命令**执行，不再维护完整 PowerShell 持久会话
- 不包含 `frida` 的外部命令当前会被明确拦截，不会再放开成通用 shell
- 终端里的 `activitys/services/object/oe/view/gs/refresh` 当前走独立异步 worker，不会把真实设备上的慢 RPC 直接阻塞在 GUI 主线程
- 终端里的 `attach/spawn/restart/stop` 不走 CLI 阻塞等待语义；启动后立即返回，由现有按钮、终端 `stop` 命令或 auto-stop 结束会话
- 终端当前支持：
  - CLI 模式按钮切换
  - 上下键命令历史
  - 输入时实时出现候选
  - `Tab` 展开/应用当前候选
  - `attach/spawn` 脚本名补全
  - `select` 包名补全
  - 仅对包含 `frida` 的外部命令做 PowerShell 单次执行
- 终端当前补全规则固定为：
  - 无空格阶段只补命令
  - `attach/spawn` 只补脚本
  - `select` 只补包名
- 终端多候选补全当前带分类前缀：
  - `命令: attach`
  - `脚本: alpha.js`
  - `包名: pkg.demo`
- 终端补全当前已经把显示文本与实际插入文本分离；候选列表展示分类前缀，但写回终端活动输入行时仍插入真实命令/脚本/包名
- `activitys/services/object/oe/view/gs/refresh` 的异步状态当前通过追加 `[TOOL] 命令执行中...` 日志表达，不再依赖旧的上方 context 标签
- 右侧日志已支持脚本侧结构化消息，宿主会把 `hookers_log` 统一格式化为 `[JS] / [JS:WARN] / [JS:ERROR]`
- GUI 中所有后台线程结果最终都通过 `UiThreadDispatcher` 或主窗口信号切回主线程后再更新控件
- `ActionWorker` 是 GUI 通用一次性任务执行器；RPC、APK 扫描、停止 Hook、停止 server、重启 App 都复用它

## 当前快捷按钮体系

先区分两类：

- **内置快捷脚本**：直接使用 `hookers/js/` 下的内置脚本启动
- **参数化快捷脚本**：先弹窗收集参数，再在 `workspaces/<package>/js/` 生成运行时脚本副本后启动

当前 GUI 里还会把快捷按钮按功能分成 5 组：

- **网络与抓包**
- **UI 观察**
- **Native / JNI**
- **加密分析**
- **对抗与绕过**

| 按钮文案 | key | 对应脚本 | 类型 | 启动前参数 | 模式策略 | 实际启动脚本位置 | 特殊行为 |
|---|---|---|---|---|---|---|---|
| 探测网络栈 | `detect_network_stack` | `detect_network_stack.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/detect_network_stack.js` | 短时观察脚本；观察窗口结束后会自动请求停止 Hook |
| 查看 OkHttp 拦截器 | `print_okhttp_interceptors` | `print_okhttp_interceptors.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/print_okhttp_interceptors.js` | 手动停止 |
| 抓取 OkHttp 请求 | `okhttp_capture` | `okhttp.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/okhttp.js` | 手动停止 |
| 监控 JNI 注册 | `hook_register_natives` | `hook_register_natives.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/hook_register_natives.js` | 手动停止 |
| 定位反 Frida SO | `find_anti_frida_so` | `find_anit_frida_so.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/find_anit_frida_so.js` | 手动停止 |
| 监听点击事件 | `click_trace` | `click.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/click.js` | 手动停止 |
| 监听输入框 | `edit_text_trace` | `edit_text.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/edit_text.js` | 手动停止 |
| 监听文本视图 | `text_view_trace` | `text_view.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/text_view.js` | 手动停止 |
| 监听 URL | `url_trace` | `url.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/url.js` | 手动停止 |
| 监听页面跳转 | `activity_events_trace` | `activity_events.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/activity_events.js` | 手动停止 |
| 监听加密调用 | `hook_encryption_algo` | `hook_encryption_algo.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/hook_encryption_algo.js` | 手动停止 |
| 监听摘要/HMAC | `hook_encryption_algo2` | `hook_encryption_algo2.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/hook_encryption_algo2.js` | 手动停止 |
| 跟踪 JNI 调用 | `jni_method_trace` | `jni_method_trace.js` | 参数化 | 目标 `so` 文件名 | 跟随当前 Attach / Spawn 选择 | `workspaces/<package>/js/jni_method_trace.runtime.js` | 点击后弹窗输入 `so`；本次 GUI 会话内记住最近一次输入 |
| 跟踪 init_proc（需地址） | `trace_init_proc` | `trace_init_proc.js` | 参数化 | 目标 `so`、`startAddr`、`endAddr` | 跟随当前 Attach / Spawn 选择 | `workspaces/<package>/js/trace_init_proc.runtime.js` | 点击后弹窗输入参数；`startAddr/endAddr` 只接受十六进制；本次 GUI 会话内记住最近一次输入 |
| 绕过常见 Root 检测 | `bypass_root_detect` | `bypass_root_detect.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/bypass_root_detect.js` | 手动停止 |
| 绕过常见 VPN 检测 | `bypass_vpn_detect` | `bypass_vpn_detect.js` | 内置 | 无 | 跟随当前 Attach / Spawn 选择 | `hookers/js/bypass_vpn_detect.js` | 手动停止 |

### 接入新按钮时的真实分界

- 普通内置脚本：优先改 `ui/quick_hook_actions.py` + `ui/ui_messages.py`
- 需要额外参数的脚本：除了配置表，还要看 `ui/hook_runtime.py` 和 `ui/composition.py`
- 如果按钮需要写入运行时脚本，优先沿用当前 JNI / init_proc 的模式，不要另起一套参数系统
- 如果按钮属于专项/参数化能力，优先补 tooltip，避免用户只看按钮文案误判用途

## 什么时候改哪里

- 设备准备、root 检查、远端 server、前台 App、App 枚举：
  - `core/device_service.py`
- 工作区初始化、本地 APK、脚本副本、输出文件、运行时脚本落地：
  - `core/workspace_service.py`
- attach/spawn、脚本加载、会话 stop/detach/restart：
  - `core/session_service.py`
- 快捷脚本按钮配置、按钮文案、网格位置：
  - `ui/quick_hook_actions.py`
- 参数化快捷脚本弹窗、输入校验、运行时脚本生成、快捷按钮分发：
  - `ui/hook_runtime.py`
  - `ui/composition.py`
- GUI 内嵌终端命令、历史、补全、命令分发：
  - `ui/terminal_console.py`
- 终端相关提示符、help 文案、补全文案：
  - `ui/ui_messages.py`
- RPC 查询与 Hook 脚本生成：
  - `core/rpc_service.py`
- 本地 APK 扫描：
  - `core/apk_scan_service.py`
- GUI 装配或 signal 连接：
  - `ui/composition.py`
- 主窗口控件布局、splitter、focus mode、公共 UI helper：
  - `ui/main_window.py`
- 统一错误展示：
  - `ui/error_presenter.py`
- 设备准备 / App / 工作区 GUI 流程：
  - `ui/app_workflow.py`
- RPC 工具 GUI 流程：
  - `ui/rpc_tools.py`
- APK 扫描 GUI 流程：
  - `ui/apk_scan.py`
- 日志过滤 / 搜索 / 渲染：
  - `ui/log_panel.py`
- 文案统一与状态文本：
  - `ui/ui_messages.py`
- GUI 协议与 payload：
  - `ui/controller_types.py`
- 主线程切回 / 跨线程 GUI 安全调度：
  - `ui/ui_thread_dispatcher.py`
- 一次性后台动作封装：
  - `ui/workers/action_worker.py`
- 设备准备后台任务：
  - `ui/workers/device_worker.py`
- Hook 启动后台任务：
  - `ui/workers/hook_worker.py`
- 工作区初始化后台任务：
  - `ui/workers/workspace_worker.py`

## GUI 主流程

1. 运行 `python app_gui.py`
2. 构建 `HookerContext`
3. 创建 `DeviceService` / `WorkspaceService` / `SessionService` / `RpcService` / `ApkScanService`
4. 打开主窗口
5. 点击 `准备环境并刷新 App`
6. 选择目标 App（或由前台自动选中）
7. 按需点击 `初始化工作目录并刷新列表`
8. 选择脚本或生成脚本
9. 选择 Attach 或 Spawn
10. 点击 `开始注入`
11. 通过 RPC 工具、日志面板继续分析

## GUI 终端主流程

1. 运行 `python app_gui.py`
2. 点击右侧 `进入 CLI 模式`，让黑色终端本体进入可输入状态
3. `ui/terminal_console.py` 负责：
   - 切换 CLI 模式
   - 解析命令
   - 维护历史
   - 维护 REPL transcript 风格回显
   - 维护实时补全、Tab 补全应用与候选分类前缀
   - 分发到现有 app/rpc/hook 能力
   - 对包含 `frida` 的外部命令做单次 PowerShell fallback
4. `ui/cli_terminal_view.py` 负责：
   - 维护底部活动输入行
   - 保护 prompt 与历史输出区
   - 转发 Enter / Tab / 上下键
5. 当前 App 查询类命令结果统一写入同一个黑色终端本体
6. `attach/spawn/restart/stop` 仍复用现有 GUI 会话链，不会进入 CLI 的阻塞等待循环
7. `refresh/select/apps` 用于把 GUI 终端进一步拉近 CLI 工作流，但最终仍以 GUI 上下文状态为准

## GUI 快捷脚本主流程

### 普通快捷脚本

1. 运行 `python app_gui.py`
2. 点击 `准备环境并刷新 App`
3. 选择目标 App
4. 在中部对应功能分组里直接点击快捷按钮
5. `ui/quick_hook_actions.py` 决定：
   - 使用哪个内置脚本
   - 所属分组、busy 文案、tooltip 和工具日志模板
6. `ui/hook_runtime.py` 统一走内置脚本启动链路
7. 脚本输出通过 `SessionService` 注入的 `Hookers` 日志桥回流到右侧日志面板

### 参数化快捷脚本

1. 运行 `python app_gui.py`
2. 点击 `准备环境并刷新 App`
3. 选择目标 App
4. 点击参数化快捷按钮（如 JNI / init_proc）
5. `ui/hook_runtime.py` 先弹窗收集参数
6. 宿主把模板脚本写成工作区运行时脚本
7. 再按当前 Attach / Spawn 选择启动
8. 脚本输出同样通过 `Hookers` 日志桥回流

## 测试基线

- 当前本地测试基线：`python -m pytest tests -q`
- 当前通过数量：**215 passed**
- 已覆盖的重点包括：
  - `ui_messages`
  - `log_panel`
  - `app_workflow`
  - `hook_runtime`
  - `terminal_console`
  - `cli_terminal_view`
  - `rpc_tools`
  - `apk_scan`
  - `main_window` 的部分公共行为
  - `composition`
  - `quick_hook_actions`
  - `error_presenter`
  - `workspace_service`
  - `models`
  - `apk_scan_service`
  - `device_service` 的纯逻辑与 façade 行为
  - `session_service` 的结构化异常路径与结构化日志桥
  - 高级 Frida 启动器
  - 高级启动器参数化模板添加 / 重新配置 / 多实例不互相覆盖
- 这些测试默认不依赖：
  - 真机
  - 真 ADB 连接
  - 真 Frida 会话

## 开发与维护时最容易误判的点

- 不要把 `workspaces/<package>/` 当成框架源码
- 不要把 `mobile-deploy/` 二进制当成业务实现说明
- 不要默认 README 一定和当前代码同步；当前代码才是事实来源
- 不要默认“无快捷按钮脚本”就不属于内置集合；优先确认 `hookers/js/`
- 不要假设 Attach 会自动把 App 拉到前台
- 不要假设选中 App 会自动初始化工作区
- 当前初始化工作目录会把 `hookers/js` 的内置脚本复制到工作区，文件名加 `内置-` 前缀；已有同名副本会跳过，不会覆盖
- 不要假设 APK 扫描依赖当前 App 或当前会话
- 不要因为 `device_service.py` 涉及 ADB/Frida 就认为它不可测试；其中已有不少纯解析/纯本地测试
- 不要把 `composition.py` 当作业务层；它只做装配与接线
- 不要在新增快捷按钮时同时修改多处散落逻辑；先看 `ui/quick_hook_actions.py`
- 不要忽略“普通快捷脚本”和“参数化快捷脚本”的差异；后者必须同步检查 `ui/hook_runtime.py` 与 `ui/composition.py`
- 不要忘记同步检查快捷按钮所属分组、tooltip 和模式提示是否仍然准确
- 不要把 GUI 终端误当成真实 shell；它只是 GUI 内嵌 CLI 控制器，命令语义参考 `hookers.py`，但不会复用 `PromptSession` 或 CLI 阻塞等待模型
- 不要忘记当前终端不是完整系统 shell；项目内命令永远优先，只有包含 `frida` 的外部命令才会被放行到 PowerShell
- 不要在终端里直接再实现一套独立会话管理；`attach/spawn/restart/stop` 仍应复用现有 `hook_runtime.py` / `session_service.py`
- 不要忘记终端补全现在有明确分层：无空格只补命令，`attach/spawn` 只补脚本，`select` 只补包名；候选展示带 `命令:` / `脚本:` / `包名:` 前缀
- 不要把终端候选列表显示文本当成真实输入文本；当前显示文本和实际插入文本已经分离
- 不要再按旧模型理解“上方独立输入框 + 下方日志框”；当前是黑色终端本体同时承担显示 + 输入 + 历史 + 补全
- 不要让 controller 再重新互相强耦合回 `MainWindow`
- 不要在 worker 或 Frida 回调线程里直接碰 Qt 控件；统一走 `ui/ui_thread_dispatcher.py` 或主窗口信号
- 不要把 `controller_types.py` 当成无意义类型壳；它实际上定义了 GUI 依赖边界和 payload 契约
- 不要把 `apk_scan_service.py` 当成业务分析层；它本质是对 `ApkCheckPack.exe` 的本地执行封装

---
> Source: [Mengz3/Frida-Hookers](https://github.com/Mengz3/Frida-Hookers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
