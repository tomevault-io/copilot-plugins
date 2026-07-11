## deepseek-translation-studio

> - 项目根目录为 `deepseek_translation_studio/`。

# AGENTS.md

## 1. 项目结构
- 项目根目录为 `deepseek_translation_studio/`。
- 入口文件是 `main.py`，应用组装在 `src/app.py`。
- 通用对话捕获剥离版入口是 `conversation_main.py`，窗口在 `src/ui/conversation_window.py`。
- UI 位于 `src/ui/`，业务服务位于 `src/services/`，引擎位于 `src/engines/`，数据库访问位于 `src/database/`，数据模型位于 `src/models/`。
- 默认 Prompt 模板位于 `src/resources/default_prompts.json`。

## 2. 运行命令
- 安装依赖后使用 `python main.py` 启动桌面应用。
- 使用 `python conversation_main.py` 启动剥离版通用对话捕获器。
- API 模式通过环境变量 `DEEPSEEK_API_KEY`、`DEEPSEEK_API_BASE`、`DEEPSEEK_MODEL` 配置。
- Web 模式默认打开 `https://chat.deepseek.com/`，可用 `DEEPSEEK_WEB_URL` 覆盖。
- 剥离版默认数据目录使用 `%LOCALAPPDATA%\DeepSeekConversationCapture`，可用 `DCC_APP_DATA_DIR` 覆盖；不得默认复用主程序的 `DST_APP_DATA_DIR`。

## 3. 测试命令
- 使用 `pytest` 运行单元测试。
- 修改 Prompt 拼接、SQLite repository、Web 捕获版本选择、导出逻辑后必须运行 `pytest`。

## 4. 构建命令
- 使用 `python scripts\build_exe.py` 构建 Windows exe。
- 使用 `python scripts\build_conversation_exe.py` 构建剥离版 `DeepSeek Conversation Capture`。
- 使用 `powershell -NoProfile -ExecutionPolicy Bypass -File scripts\publish_github_release.ps1` 发布到 GitHub；该脚本必须只从 `GITHUB_TOKEN`、`GH_TOKEN`、Git Credential Manager 或 Codex GitHub 集成 helper 临时读取 token，不得把 token 写入文件、Git remote 或日志。
- 构建产物必须位于 `dist\DeepSeek Translation Studio\DeepSeek Translation Studio.exe`。
- 剥离版构建产物必须位于 `dist\DeepSeek Conversation Capture\DeepSeek Conversation Capture.exe`。
- Web 模式打包产物必须同时包含 `_internal\playwright\driver\package\.local-browsers\chromium-*`，否则 exe 可以打开但无法启动 Playwright 浏览器。
- 修改打包入口、资源文件路径或 Playwright 引入方式后，必须重新构建并确认 exe 文件存在。
- 修改 `src/database/schema.sql` 后必须确认 `src/database/db.py` 中存在旧库迁移逻辑。

## 5. 代码风格
- 当前未发现 lint / format 命令。
- 核心类和跨模块函数必须保留类型注解。
- 新增功能优先使用标准库；引入新依赖前必须更新 README、requirements、pyproject，并说明许可证。
- 当前新增打包依赖为 PyInstaller；仅用于生成 Windows exe，不得把它引入业务运行路径。
- MSI 使用 WiX Toolset 3.14 binaries 作为构建期工具，下载到 `.build_tools`；不得把 WiX 二进制提交为源码或运行时依赖。
- PySide6 UI 文案可以使用中文；业务层异常信息应清晰可展示给用户。

## 6. 模块边界
- `src/ui/` 只做界面、信号和状态展示，不直接拼 SQL，不直接操作 Playwright 页面。
- `src/services/translation_service.py` 负责翻译任务编排和历史保存。
- `src/services/capture_service.py` 负责异常提示检测、版本去重和最长有效文本选择。
- `src/models/translation_task.py` 的 `task_mode` 决定任务语义；通用对话捕获必须使用 `TASK_MODE_GENERAL_CAPTURE`，不得依赖源文本是否像 SRT 来误启用 SRT 清理。
- `src/ui/conversation_window.py` 是剥离版 UI，只能创建 `ENGINE_WEB` + `TASK_MODE_GENERAL_CAPTURE` 请求；不得加入 API 设置、SRT 导出、任务类型下拉框或字幕补译入口。
- `src/services/browser_profile_service.py` 负责 Web 账号切换时的浏览器 profile 轮换；只能移动 `settings.app_data_dir` 内的 profile，不能删除或移动任意路径。
- `src/services/subtitle_translation.py` 只负责 SRT 检测和模型输出中的 SRT 代码块/网页控件文本清理，不自动改写 Prompt 为 JSON。
- `src/services/subtitle_translation.py` 可以做 SRT 完整性检查、剩余片段渲染和多次输出拼接；续译时必须保留原编号和时间轴，不得把未翻译的源字幕当作完成译文。
- SRT 合并和完成判定必须拒绝与源字幕正文相同的无效候选译文；但长度不超过 4、且不含拉丁/日文/韩文字符的中文数字、短中文、标点或符号类文本可以保留，因为它们翻译后可能天然不变。
- 短符号、音乐符号和标点类字幕（例如 `♪`、`...`）如果翻译后与源文本一致，可以作为有效译文保留；英文、日文假名、韩文等非可保留源文复制仍必须拒绝。
- 判断源文复制时必须与合并逻辑使用同一身份规则：先按编号+时间轴匹配，编号不一致时再按时间轴匹配；错编号但同时间轴的英文/日文/韩文等源文复制不得计入覆盖数。
- `src/services/translation_service.py` 负责在 Web 模式 SRT 输出不完整时优先在当前 DeepSeek 对话续发缺失片段；当前对话达到续译上限后，必须先用最新合并结果复核实际覆盖条数，再打开新的 DeepSeek 页面/对话补译缺失片段，并把结果合并回同一条历史记录。
- Web 模式 SRT 续译期间，原始续译捕获可以入库，但 UI 结果区必须显示已合并 SRT，不得用第二段裸输出覆盖第一段结果。
- Web 通用对话捕获模式必须复用 `src/engines/deepseek_web.py` 的可见文本捕获、自动继续生成和历史版本写入；不得绕过网页 DOM 去读取隐藏接口响应。
- Web 通用对话捕获模式不得启用 SRT 专用清理、SRT 完成判定、SRT 续译 Prompt 或新窗口补译；即使输入内容符合 SRT 结构，也应按普通网页回答保存。
- Web 通用对话捕获模式右侧状态必须显示内容长度，停止时整理当前捕获内容，不显示“正在整理 SRT”。
- 通用对话捕获模式必须隐藏或阻止 SRT 导出，只允许 TXT/Markdown 导出当前捕获内容。
- 剥离版 portable 启动脚本必须设置 `DCC_APP_DATA_DIR=%~dp0data`，不能设置主程序的 `DST_APP_DATA_DIR`。
- Web 模式第二次及后续发送 Prompt 前必须记录当前页面已显示译文作为本轮捕获基线；发送后旧译文不能被当成本轮新输出、不能触发稳定收尾，也不能让自动续译快速耗尽尝试次数并关闭浏览器。
- SRT 续译缺口定位必须优先按源字幕编号/时间轴的最高已覆盖位置判断尾部截断点；早期内部缺口只能在尾段完成后按“只补列出的缺失字幕”单独处理，不能导致从早期编号重译整段。当前输出含错误提示时必须先保留当前 DeepSeek 对话继续缺失尾段；只有当前对话续译上限耗尽且复核后仍不完整，才允许新开页面/对话补译缺失 SRT。
- 如果当前 SRT 捕获是稀疏片段（例如只出现 682、925、1162 等后段编号，前缀覆盖密度低于 50%），不得当成普通尾部截断继续往后补；必须从连续前缀后的第一条重新补译，避免覆盖数长期卡住。
- 如果当前 SRT 已显示到最后编号但存在大块内部缺口（例如从中段起缺失数百条），不得当成“小内部缺口列表补译”；必须从该大块缺口第一条开始重新补译后续字幕，再由合并器覆盖缺口。
- Web 模式 SRT 捕获一旦覆盖全部源字幕且无内部缺口，必须自动结束捕获并进入合并流程；不得继续因为页面 footer、点赞按钮或其他 DOM 变化持续记录 mutation。
- Web 模式 SRT 状态必须显示有效字幕条数 `已覆盖/总数`，不要用字符长度替代用户关心的字幕进度。
- Web 模式 SRT 捕获到拒答/异常提示时，不得把拒答文本显示到右侧或写入最终字幕；必须先保留当前对话并继续合并剩余 SRT，当前对话耗尽后只能用缺失片段打开新页面补译，禁止重跑整份源字幕。
- DeepSeek 显示“服务器繁忙，请稍后再试，或使用快速模式”并出现“继续生成”时，`src/engines/deepseek_web.py` 必须进入 `busy_retry` 状态，默认约 180 秒重试点击“继续生成”；不得点击“快速模式”，不得把普通 `max_continue_clicks` 当作服务器繁忙重试上限。
- Web 模式执行 SRT 任务时不得为了规避思考文本而强制关闭 DeepSeek “深度思考”；如果捕获到“正在思考/用户要求/这些要求”等非 SRT 内容，不能显示到右侧或计入字幕进度，必须通过清理、合并和当前对话重发严格 SRT 输出 Prompt 处理。
- Web 模式流式捕获 SRT 时，未完成的下一条字幕碎片（例如孤立编号加半截时间轴）不得并入上一条字幕文本；同一编号/时间轴出现多次时，合并结果必须优先使用较新的完整捕获。
- 清理 DeepSeek 思考/需求分析噪声时必须区分 SRT 结构位置；`这里要`、`用户要求`、`这些要求` 等文本如果位于字幕正文行内，必须保留为译文，不得触发后续字幕截断。
- 同一 DeepSeek 页面包含多段回答时，第一段 SRT 后如果出现 `已思考/我们被要求/开始输出` 等说明，且后面还有新的 SRT 块，清理逻辑必须跳过说明并继续提取后续 SRT；不得停在第一次截断位置。
- SRT 块之间的普通说明文字（例如 `下面是后续翻译结果`）不得进入右侧结果、历史记录或导出。
- 流式清理必须识别半截完整箭头时间轴，例如 `265` 后跟 `00:11:33,400 --> 00:11:34,5`，不得把它拼入上一条字幕正文。
- Web 模式捕获页面文本时，若 DOM 中存在 SRT 片段，应优先收集所有可见 SRT-like 节点并按 DOM 顺序合并，避免只选择较长的旧节点导致右侧进度停在早期字幕。
- 捕获版本高频写入时，服务层不得在每次 `record_version` 后重新全量读取该任务全部版本；当前任务的 best/error 状态必须用内存缓存维护，最终选择再读取完整版本。
- 结果区收到新的 Web/SRT 文本后必须滚动到末尾，保证用户看到最新同步进度。
- `src/engines/deepseek_web.py` 只捕获页面 DOM 中已经显示的文本，不读取隐藏接口响应。
- `src/engines/deepseek_web.py` 自动发送前必须验证 Prompt 已写入可见输入控件；不能只依赖 Playwright `fill()` 调用成功。
- Web 模式大 Prompt 必须优先使用可见 textarea/contenteditable 的原生 setter 加 `input/change` 事件写入；不要用键盘逐字插入大文本。
- Web 模式捕获结果必须过滤当前用户 Prompt/输入框文本，不能把用户消息、页面标题或模式按钮文本当作译文。
- `src/engines/deepseek_web.py` 的自动继续生成只允许在页面进入“继续生成 / Continue”截断态后点击；普通 idle/unknown 状态只能等待短暂 grace window，禁止全页扫描“继续”文本并提前点击。
- 若预览区/输出区已经出现真实可点击“继续生成”控件，即使捕获文本仍在变化，也必须在截断态稳定达到检测延迟后点击，不能继续等待文本静默。
- 页面中存在多个普通文本“继续”时，自动继续生成必须跳过不可点击文本，只能点击可见、可用、上下文安全且有可点击祖先的最新“继续生成”控件。
- 输出安静后如果页面尚未出现“继续生成”，必须保留短暂 grace window 继续检查，避免 DeepSeek 延迟渲染按钮时直接收尾。
- `src/engines/deepseek_web.py` 的自动继续生成只允许点击文本匹配“继续生成 / 继续 / Continue / Continue generating”的可见可用按钮。
- `src/engines/deepseek_web.py` 点击“继续生成”时必须容忍 DeepSeek 页面把可见文字放在不稳定的 `span` 中；Playwright 普通点击超时不能让任务失败，应先验证标签和安全上下文，再降级点击最近的按钮/role 控件或当前元素。
- 主窗口“停止”按钮只负责请求 Web 任务停止输出；`src/engines/deepseek_web.py` 应尝试点击网页“停止生成 / 停止回答”，`src/services/translation_service.py` 随后必须用已捕获内容整理 SRT 并显示到右侧，不得把停止请求当作直接丢弃结果。
- Web 模式捕获结束不能只依赖短时间文本稳定；必须结合网页“停止生成 / 发送 / 继续生成”等可见控件状态。状态未知时使用较长兜底等待，避免长 SRT 中途停顿时误关 Chromium。
- 浏览器被用户关闭时必须按 cancelled 处理，不能把 Playwright 的 Target closed/Locator 错误直接展示为失败。
- API 模式的“获取模型”和“测试 API”必须使用 API worker 后台线程执行，不得阻塞 PySide6 主线程；测试 API 使用 `/models` 端点，不发起聊天补全请求。
- 主窗口必须包含可点击署名信息：HaoXiang Hwang、`mailto:didadida1688@gmail.com`、`https://Nextweb4.github.io/`。
- `src/database/repositories.py` 是 SQLite 读写入口。

## 7. 禁止事项
- 禁止绕过网站限制、破解访问控制、抓取未展示数据、绕过验证码或绕过风控。
- 禁止把剪贴板作为 Web 捕获的核心存储方式。
- 禁止在检测到异常提示后覆盖此前已捕获的有效译文。
- 禁止自动点击登录、注册、验证、充值、订阅、删除、清空相关按钮。
- 禁止将 API Key、Authorization header、cookie 写入日志或测试夹具。
- 禁止在 PySide6 主线程执行 API 请求、Playwright 自动化或大文件导出。

## 8. 完成标准
- `python main.py` 可以启动主窗口。
- Prompt 模板内容可以在主窗口中编辑并保存到 SQLite。
- 主窗口默认引擎必须是 `DeepSeek Web`，默认状态下 API Key、Base URL 和 API 模型输入框应隐藏。
- 主窗口必须提供任务类型选择；默认保留“翻译 / SRT 自动”，切换“通用对话捕获”后应自动选中同名 Prompt 模板。
- 剥离版主窗口必须只显示通用对话捕获界面：开始捕获、停止、切换 Web 账号、清空、导出 TXT、导出 Markdown、模板编辑、固定前后缀、自动继续生成设置、提示词输入、Prompt 预览、捕获结果和日志。
- 剥离版启动后必须默认选中“通用对话捕获”模板，窗口标题包含 `DeepSeek Conversation Capture`。
- 主窗口提供深色模式开关。
- 主窗口 Web 模式必须提供“停止”按钮；任务运行时启用，按下后禁用并显示“正在整理 SRT”状态。
- `python scripts\build_exe.py` 可以生成可直接双击打开的 Windows exe。
- `python scripts\build_exe.py` 还必须生成 `dist\packages\DeepSeek Translation Studio-0.1.0.msi` 和 `dist\packages\DeepSeek Translation Studio-0.1.0-portable.zip`。
- `python scripts\build_conversation_exe.py` 可以生成剥离版 exe、MSI 和 portable zip，文件名以 `DeepSeek Conversation Capture-0.1.0` 开头。
- 打包后的 `--playwright-smoke-test` 必须使用随包 `chromium-*` 主浏览器；发布包不包含 `chromium_headless_shell-*`。
- exe 版本信息、MSI 属性、portable 包内 `ABOUT.html` 和主窗口都必须包含 HaoXiang Hwang、邮箱和个人网址。
- portable 包必须包含 `Start Portable.bat`，并把 `DST_APP_DATA_DIR` 指向 portable 目录下的 `data`。
- 首次启动会初始化 SQLite 和默认 Prompt 模板。
- API 模式必须提供“获取模型”和“测试 API”按钮；获取模型成功后模型下拉框可选择返回的模型 ID。
- Web 模式捕获到的新文本会写入 `web_capture_versions`。
- 自动继续生成事件必须写入 `web_capture_versions.event_type`，点击次数必须写入 `history.continue_click_count`；只有实际点击成功后才能增加计数，点击超时或降级点击失败不能污染 UI/历史计数。
- Web 模式结束后会把最长有效文本写入 `history`。
- Web 通用对话捕获结束后必须把最长有效网页文本写入 `history`，并实时显示到右侧结果区；不得触发 SRT 清理或补译。
- Web 通用对话捕获即使输入为 SRT-like 文本，也必须保留网页回答里的普通说明、思考标记和页脚文本；这些只在 SRT 翻译模式中按 SRT 规则清理。
- Web 模式 SRT 任务结束时，最终 SRT 的条数必须对应源字幕条数；若仍有缺口，只能以 warning 收尾并明确提示缺失。
- Web 模式 SRT 当前对话自动续译耗尽后，必须用最新合并文本复核 `covered_count/total_count`，再打开新 DeepSeek 页面补译缺失片段；最终 warning 必须使用复核后的实际覆盖条数，不得沿用旧日志里的中间值。
- Web 模式 SRT 捕获出现低密度稀疏片段时，续译 Prompt 必须从 `contiguous_completed_count + 1` 对应的字幕开始，而不是从最高已出现编号之后开始。
- Web 模式 SRT 捕获已到最后编号但存在大块内部缺口时，续译 Prompt 必须从该大块缺口第一条开始，并使用后续字幕 tail 模式，而不是把数十/数百条缺失字幕塞进 listed 模式。
- Web 模式 SRT 捕获如果先有连续前缀、随后跳到后段稀疏片段且尾部仍缺失，续译 Prompt 必须从中间大块缺口第一条开始；不得因为最高出现编号被抬高而直接从最后编号之后继续。
- Web 模式 SRT 补译捕获到的新字幕必须实时合并并显示到右侧结果；后段稀疏片段不能提前显示到右侧末尾，必须等中间大块缺口逐步补齐后再显露后段。
- DeepSeek 服务器繁忙提示出现时，Web 捕获不能按普通 idle 收尾或失败；必须保持浏览器任务运行、按间隔只点击“继续生成”，并继续保留深度思考/专家模式质量设置。
- Web 模式 SRT 续译补充如果连续返回两条及以上非可保留源文复制，最终结果不得显示这些源文；必须继续请求缺失片段或以 warning 收尾。
- Web 模式 SRT 补译中，连续两条及以上非可保留源文复制必须继续视为无效补译；短中文数字、名字、标点或符号类补译与源文完全相同时，可以作为未改写译文插入，避免覆盖数停住并循环补译。
- Web 账号切换必须在无翻译任务运行时进行；切换时把当前 `browser_profile` 移动到 `browser_profile_backups` 后创建新的空 profile，不得直接删除旧登录态。
- TXT 和 Markdown 导出必须包含原文、译文、Prompt、引擎、状态、创建时间和备注；Markdown Prompt 代码块必须使用足够长的动态围栏，避免 Prompt 内部 ``` 截断导出。
- SRT 导出必须只写入 `translated_text`，使用 UTF-8 BOM，避免字幕播放器或旧 Windows 工具显示乱码。
- 主窗口清空操作必须只清当前输入、结果和捕获状态，不得清除模板、API 设置和固定前后缀，便于连续复用同一配置。
- 主窗口运行翻译任务时收到关闭事件必须先请求 worker 停止并忽略本次关闭，任务收尾后才允许关闭，避免销毁运行中的 QThread 或 Playwright context。

## 9. Review 标准
- Review 捕获逻辑时，必须检查 MutationObserver、300ms 快照、SQLite 版本表、异常提示检测和最长有效文本选择是否同时存在。
- Review Web 输入逻辑时，必须检查专家模式、深度思考模式、Prompt 写入验证和发送按钮触发顺序。
- Review SRT 翻译逻辑时，必须检查 Prompt 是否保留用户模板原意、捕获结果是否保留换行、最终 SRT 是否去掉代码块工具栏文本。
- Review 通用对话捕获时，必须检查 `task_mode` 是否传到服务层、`next_prompt_callback` 和 `capture_complete_callback` 是否为 `None`、最终结果是否来自 `capture_service.final_text`。
- Review 通用对话捕获 UI 时，必须检查任务类型切换、同名模板选择、结果标签、内容长度状态和 SRT 导出隐藏。
- Review 剥离版 UI 时，必须检查没有 `engine_combo`、`task_mode_combo`、`api_key_input` 和 `export_srt_button`，且请求固定为 Web 通用捕获。
- Review 剥离版打包时，必须检查独立 exe、portable zip、`Start Portable.bat` 中的 `DCC_APP_DATA_DIR` 和随包 Chromium。
- Review SRT 自动续译时，必须检查缺口识别、续译 Prompt、合并输出、未完成 warning 和历史记录写入；不要用原文 SRT 复制内容冒充译文。
- Review SRT 稀疏捕获时，必须覆盖“首轮只捕获后段编号、前面大面积缺失”的场景，确认续译从第一个连续缺口开始。
- Review SRT 续译补充时，必须用测试覆盖“补充输出连续复制多条非可保留源字幕”的场景，确认原文不会进入右侧结果、历史记录或 SRT 导出。
- Review SRT 补译合并时，必须同时覆盖“连续多条非可保留源文复制被拒绝”和“短中文数字/符号类未改写字幕可插入”的场景，避免补译页面已输出但覆盖数不前进导致循环补译。
- Review SRT 补译合并时，还必须覆盖短符号/标点未改写字幕可插入的场景，例如 `♪`、`...`。
- Review SRT 源文复制保护时，必须覆盖“模型改错编号但保留源时间轴和源正文”的场景，防止按时间轴合并时把源文复制误判为译文。
- Review Web 账号切换时，必须确认 profile 路径在 `settings.app_data_dir` 内、任务运行时不能切换、旧 profile 只备份不删除。
- Review SRT 完成判定时，必须检查 `covered_count/total_count`、内部缺口和尾部缺口；不能只看文本长度或最后一个编号。
- Review SRT 捕获清理时，必须确认 DeepSeek 的“本回答由 AI 生成，内容仅供参考...”页脚不会写入最后一条字幕。
- Review SRT 流式格式时，必须确认类似 `3\n00:00` 的未完成块不会进入上一条字幕正文，最终显示保持“编号、时间轴、译文、空行”的 SRT 结构。
- Review Web/SRT 同步显示时，必须确认右侧结果区能看到最新捕获末尾，且停止后不会显示 DeepSeek 思考/需求分析等非 SRT 对话文本。
- Review Web/SRT 卡住问题时，必须用测试覆盖“第一轮只输出思考/需求分析且没有任何 SRT”的场景，证明状态进度不误报、右侧不污染、当前对话会继续请求 SRT。
- Review 捕获性能时，必须确认 Playwright 轮询没有重复执行同一轮 DOM 全量文本收集，SQLite best version 不在每个捕获版本后全表扫描。
- Review SRT 拒答处理时，必须确认“你好，这个问题我暂时无法回答...”不会覆盖右侧合并结果；当前对话未耗尽时不得创建新窗口，耗尽后新窗口只能补译复核得到的缺失片段。
- Review Web 自动续译时，必须确认发现当前页“继续生成”按钮会优先点击；只有没有可继续按钮且 SRT 仍缺失时，才在同一对话里发送剩余字幕 Prompt。
- Review Web 自动继续生成时，必须确认“继续生成”只在截断态后检查和点击，不能在 `停止生成` 控件仍可见或页面只是普通 idle 时点击。
- Review 停止按钮时，必须确认网页停止输出、服务层整理 partial SRT、右侧显示整理结果和历史记录 warning 状态同时成立。
- Review API 逻辑时，必须检查 API Key 未进入日志和历史。
- Review API 模型列表/测试功能时，必须检查 `/models` 请求的 URL、Authorization header、UI 后台线程和模型下拉框更新。
- Review UI 逻辑时，必须检查耗时任务是否通过 QThread 执行。
- Review UI 视觉更新时，必须检查主窗口图标、品牌栏、可点击署名链接和按钮图标。
- Review UI 生命周期时，必须检查运行中关闭窗口不会直接销毁 worker 线程，且会复用停止请求路径整理当前结果。
- Review 引擎切换时，必须检查 Web 模式隐藏 API 输入框、API 模式隐藏 Web 继续生成控件。
- Review schema 变更时，必须检查测试和数据兼容性。
- Review 打包逻辑时，必须验证 exe smoke、Playwright smoke、portable zip 关键文件、MSI 行政解包；MSI 不得包含 `chromium_headless_shell-*` 导致路径过长。

## 10. 常见风险
- DeepSeek 网页 DOM 改动会导致输入框或发送按钮选择器失效。
- DeepSeek 网页输入框可能是 contenteditable 或嵌套编辑器，`locator.fill()` 返回成功不代表可见输入框已有内容。
- 用户原文可能超过 100k 字符；如果 Web 输入流程没有大文本快速写入路径，会表现为专家模式/深度思考已开启但输入框长期为空。
- 如果网页捕获没有排除用户 Prompt，任务会在模型真正回答前误判 success，并把原文或 Prompt 当作译文保存。
- 如果通用对话捕获没有显式 `task_mode` 分支，SRT-like 输入会被误走字幕清理/补译，导致日常对话结果被截断或卡住。
- Playwright persistent context 首次需要用户手动登录，自动登录不属于本项目能力。
- PySide6 和 Playwright 增加打包体积，后续打包需单独审计。
- 不能只复制 `DeepSeek Translation Studio.exe` 单文件；Web 模式依赖同目录 `_internal` 中的 Playwright driver 和 Chromium。
- MSI/portable 打包如果包含 `chromium_headless_shell-*`，Windows Installer 或用户解压可能因路径过长失败；发布包只要求包含 `chromium-*` 主浏览器。
- SQLite 多线程写入需保持 repository 的短连接模式，避免长期持有连接。

---
> Source: [NextWeb4/deepseek-translation-studio](https://github.com/NextWeb4/deepseek-translation-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
