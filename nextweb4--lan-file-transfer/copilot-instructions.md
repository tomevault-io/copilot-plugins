## lan-file-transfer

> - `lan_transfer/api.py`：FastAPI 路由、游客/用户/管理员 session 鉴权、文件可见性过滤、文件搜索/排序、下载响应 header。

# AGENTS.md

## 1. 项目结构
- `lan_transfer/api.py`：FastAPI 路由、游客/用户/管理员 session 鉴权、文件可见性过滤、文件搜索/排序、下载响应 header。
- `lan_transfer/audit.py`：保存目录内 `.lan-transfer-audit.jsonl` 审计日志，记录登录、登出、上传、下载、删除、权限、账户和组变更；不得记录密码、session token、authorization、cookie、secret 类字段或文件内容。
- `lan_transfer/auth.py`：账户、批量创建账户、删除账户、用户组创建/删除、密码 PBKDF2 哈希、`.lan-transfer-auth.json`、默认 `admin / 12345678`、默认 `everyone` 组、内存 session、5 次错误后按 IP 锁定 3 小时。
- `lan_transfer/storage.py`：二进制流保存、`.part` 临时文件、大小校验、SHA-256、manifest、owner、allowed_groups、旧 `default` 组迁移、用户删除后的文件转主、zip entry mtime。
- `lan_transfer/security.py`：文件名、相对路径、manifest 已存保存路径和根目录控制文件保留名安全处理；新增保存路径逻辑必须经过这里。
- `lan_transfer/server.py`：本地 uvicorn 线程生命周期；启动超时和停止失败必须显式报错，不能静默留下旧服务线程。
- `lan_transfer/config.py`：保存目录、端口和 LAN URL；LAN IP 优先使用 UDP 路由选出的非 loopback / 非 link-local 私有地址。
- `lan_transfer/logging_config.py`：保存目录内日志 handler 切换；必须先创建新 handler 成功，再替换并关闭旧 handler。
- `lan_transfer/desktop.py`：Tkinter Windows 窗口、用户/管理员 URL、二维码、服务启停、保存目录选择。
- `lan_transfer/static/user.html` 与 `user.js`：默认中文游客页，内置登录、中英文切换、文件搜索/排序；登录用户可上传并删除自己上传的文件。
- `lan_transfer/static/admin.html` 与 `admin.js`：默认中文管理员页，右上角工具/管理菜单、批量创建账户、可搜索组选择器、文件可见组、审计日志、上传、删除、修改密码。
- `tests/`：pytest 覆盖账户权限、批量创建、用户组可见性、搜索/排序、上传完整性、mtime、下载 header、zip、路径安全、同名文件、windowed exe 日志和发布元数据一致性。
- `docs/OPEN_SOURCE_AUDIT.md`：外部方案审计、许可证和冲突检查。
- `LICENSE`：MIT License，作者固定为 HaoXiang Huang。
- `RELEASE_NOTES.md`：GitHub Release 双语说明，发布前必须同步版本号、文件用途和作者信息。
- `version_info.txt`：PyInstaller Windows exe 版本资源，必须保持公司名、产品名、版权、版本和主页正确。
- `scripts/build_exe.ps1` 与 `LANFileTransfer.spec`：Windows exe 打包入口；脚本会运行 pytest、PyInstaller，并生成 `release-assets/` 下的 exe、zip 和 `SHA256SUMS.txt`。

## 2. 运行命令
- 首次准备：`py -m venv .venv`
- 安装依赖：`.\.venv\Scripts\python.exe -m pip install -r requirements.txt`
- 本地运行：`.\.venv\Scripts\python.exe -m lan_transfer.desktop`

## 3. 测试命令
- 运行测试：`.\.venv\Scripts\python.exe -m pytest`
- 修改 `api.py`、`auth.py`、`storage.py`、`security.py`、`static/user.js` 或 `static/admin.js` 的上传/下载/权限协议后，必须运行 pytest。

## 4. 构建命令
- 打包 exe：`powershell -ExecutionPolicy Bypass -File .\scripts\build_exe.ps1`
- 产物路径：`dist\LANFileTransfer.exe`
- 发布产物路径：`release-assets\LANFileTransfer.exe`、`release-assets\lan-file-transfer-v1.0.0-windows.zip`、`release-assets\SHA256SUMS.txt`

## 5. 代码风格
- Python 代码优先使用标准库；当前第三方依赖只用于 HTTP、ASGI、multipart、二维码、Pillow 图像显示和 PyInstaller 打包。
- 前端保持原生 HTML/CSS/JS，不引入 CDN、云 SDK 或运行期外部网络资源。
- 发布署名必须统一为 Author `HaoXiang Huang`、Email `didadida1688@gmail.com`、Homepage `https://nextweb4.github.io/`、GitHub `https://github.com/NextWeb4`；README、LICENSE、Release Notes、网页页脚、桌面 About/页脚和 exe metadata 都必须一致。
- Web UI 使用当前暗色工具型界面；移动端下载、删除、上传按钮必须在卡片或工具区内可见，不依赖横向滚动表格。
- 前端字号必须使用稳定尺寸或媒体查询断点，不使用 `vw`/`vh` 驱动 `font-size`。
- 前端背景保持低噪声工具界面，不使用装饰性 `radial-gradient`、orb 或 bokeh 背景。
- 当前未发现 lint / format 命令；新增 lint / format 后需同步到本文件和 README。

## 6. 模块边界
- `auth.py` 只处理账户、密码、用户组、session 和 IP 锁定，不处理文件内容。
- `auth.py` 中账户/组/密码变更写 `.lan-transfer-auth.json` 失败时，必须回滚内存态，不能让当前进程与磁盘配置分叉。
- `auth.py` 写 `.lan-transfer-auth.json` 必须使用保存目录内用户上传不可到达的隐藏随机临时文件，写入、序列化或替换失败时必须删除临时文件，不能退回固定 `.lan-transfer-auth.json.tmp`。
- `auth.py` 中登录必须先校验密码，再暴露账户禁用状态；禁用账户的错误密码仍要计入 IP 失败次数和锁定。
- `auth.py` 中强审计 auth 变更必须通过 `state_transaction()` 覆盖 snapshot、写入、审计和回滚窗口，避免审计失败回滚覆盖并发成功变更；同时需要碰 storage 的跨模块操作必须按 storage lock -> auth lock 顺序，避免和上传提交路径死锁。
- `auth.py` 中 `restore_state()` 只在 settings 真的变化时才需要写盘；写回快照失败时，必须把内存恢复到调用前状态，不能让进程内 auth 状态与磁盘 settings 分叉。
- `auth.py` 中普通用户改密、管理员重置用户密码、禁用用户和变更用户 `role` 都必须在写盘成功后失效该用户既有 session；如果后续审计失败，API 必须通过快照恢复旧密码和旧 session。
- `storage.py` 只处理磁盘、manifest、hash、mtime、zip，不读取 FastAPI `Request`。
- `storage.py` 中需要审计确认后才算完成的删除，必须使用 prepare / audit / commit 或失败 rollback，不能在审计前不可逆删除文件。
- `storage.py` 中上传在审计成功前必须保持 `audit_status="pending"` 并对列表、下载和打包不可见；发布为 `complete` 和写上传审计必须处于同一个 `manifest_transaction()`，锁释放前审计必须成功，发布失败不能写成功审计。
- `storage.py` 中文件权限变更必须通过 `manifest_transaction()` 覆盖 manifest 更新、审计和回滚窗口，避免审计成功前并发请求看到新可见性。
- `storage.py` 删除 rollback 中如果原文件已经从 tombstone 恢复，即使 manifest 写回失败，也必须保留内存 manifest 记录，避免当前进程丢失已恢复文件。
- `storage.py` 中 `.delete` tombstone 代表待提交或待回滚删除；同名新上传选择保存路径时必须把 tombstone 视为占用，不能复用原 `saved_relative_path`。
- `storage.py` 写 `manifest.json` 必须使用保存目录内用户上传不可到达的隐藏随机临时文件，不能使用固定 `manifest.json.tmp`。
- `storage.py` 从 manifest 读取 `saved_relative_path` 后必须重新经过 `security.py` 校验，禁止旧/坏 manifest 指向根部控制文件。
- `storage.py` 的保存路径占用判断必须按 Windows 文件系统语义大小写不敏感比较 manifest 路径和 `.delete` tombstone；zip entry 去重也必须避免大小写不同但 Windows 解压会冲突的名称。
- `api.py` 只编排 HTTP 参数、鉴权、文件可见性过滤、响应 header 和错误码，不自行拼接不安全文件路径。
- `api.py` 中登录、登出、密码、账户、组、文件删除等强审计变更，审计写入失败时必须回滚本次 session/auth/storage 变更。
- `api.py` 必须覆盖 FastAPI 请求校验错误响应，不能把无效请求体中的密码、session token 或内部结构通过默认 422 `input` 字段回显给客户端。
- `api.py` 的文件列表排序必须容忍旧/坏 manifest 中非数字的 `file_size` / `server_mtime`，按 0 参与排序，不能让 `/api/files` 变成 500。
- `api.py` 的用户页原生下载 scope 只能收窄权限：`scope=guest` 必须按游客处理，`scope=user` 遇到管理员 cookie 时也必须收敛到游客，不能扩大为管理员下载。
- `api.py` 的用户端身份接口 `/api/login`、`/api/session`、`/api/logout`、`/api/password` 只接受普通 `user` 角色或游客清理无效 session；管理员必须使用 `/api/admin/*` 对应接口，避免同源 cookie 让用户页继承管理员权限。
- `api.py` 的管理员身份接口必须拒绝已认证的非管理员；`/api/admin/logout` 可以让游客清理无效本地 cookie，但不能删除普通用户 session 或产生未审计登出。
- `api.py` 删除用户时必须在账户删除后再回扫一次文件 owner，避免并发上传窗口留下旧 owner；删除用户审计失败时必须先确认 auth 快照恢复成功，再把文件 owner 从 `admin` 恢复给被删用户；如果 auth 恢复失败，文件必须继续归 `admin`，避免未来同名账户继承旧文件。
- `api.py` 删除组时必须在 `auth.delete_group()` 之后再重新检查用户和文件引用；如果删除窗口里出现新引用，必须恢复 auth 快照并返回 409；如果快照恢复失败，必须返回 500，不能假装删除已回滚。
- `config.py` 中同一个 `AppConfig` 实例的 `lan_ip`、`base_url`、`user_url` 和 `admin_url` 必须使用同一次 LAN IP 解析结果，避免窗口或状态响应中展示不一致地址。
- `config.py` 探测可用端口时必须跳过已被实际绑定监听的端口，不能用 `SO_REUSEADDR` 制造可能启动失败的假阳性。
- `security.py` 是路径清理唯一入口；新增保存路径逻辑必须调用 `normalize_relative_parts` / `ensure_inside`。
- `security.py` 必须同时保护新上传路径和 manifest 已存路径；根部 `manifest.json`、`manifest.json.tmp`、`.lan-transfer-auth.json`、`.lan-transfer-audit.jsonl` 都属于保留名。
- `desktop.py` 只负责启动/停止服务和显示本地信息，不处理上传文件内容；服务启动、手动停止、切换目录前停止和关闭窗口停止失败时都必须显式提示并刷新状态。
- `desktop.py` 的状态栏必须把 uvicorn 线程仍存活但尚未 `started` 的状态视为 active，而不是 stopped；服务启动后必须周期刷新状态，避免后台线程退出后窗口仍显示 Running。
- `desktop.py` 切换保存目录时必须先让替换服务在新配置上启动成功，再替换 `self.config_data` / `self.storage` / `self.server`；如果回滚时旧服务重启失败，必须把失败也显示给用户。
- `desktop.py` 切换保存目录时，端口探测、storage 初始化、logging 切换和 server 创建必须在同一失败回滚路径内；旧服务已停止后任一环节失败，都必须尝试恢复旧服务。
- `server.py` 启动线程在 `started=True` 前退出时，必须清空 `_server` / `_thread` 并显式报错；超时停止失败时要保留 live thread 语义，不能留下不可运行的陈旧引用。
- `user.js` 可以处理游客登录、用户上传和删除自己的文件；不得提供用户组管理或任意文件删除入口。
- `user.js` 只能把 `role === "user"` 视为已登录；未确认普通用户身份时，请求用户页状态和文件列表必须避免发送同源 cookie，原生 Raw/zip 下载链接必须附带 `scope=user` 或 `scope=guest`，`Zip all` 必须提交当前可见文件 ID，不能用空参数让后端按管理员 cookie 扩大范围。
- `user.js` 刷新文件列表前如果页面认为已登录，必须先重新确认 `/api/status` 仍是普通用户；失效时先清空上传/账户 UI，再渲染游客范围文件。
- `admin.js` 可以管理用户、组、文件权限；上传仍必须发送原始 `File`、`relative_path`、`last_modified_ms` 和 `size`。
- `admin.js` 中并发刷新用户、组和审计数据时必须忽略过期响应；任何回到登录态的路径都必须清空旧状态行，避免显示已失效管理员信息。
- `admin.js` 刷新文件列表前必须先确认 `/api/admin/session` 仍是管理员；`loadStatus()` 返回非管理员时必须停止整个管理员刷新链路；失效时必须切回登录态并清空旧文件/用户/组/审计数据，不能用游客 `/api/files` 结果渲染管理员权限控件。
- `admin.js` 的“全部打包”必须提交当前 `state.files` 的文件 ID，不能用空参数下载管理员全部可见文件；内联组编辑器的 accessible name 必须包含文件名或用户名上下文。
- 前端状态、文件、用户、组和审计的并发刷新必须同时忽略过期成功响应和过期失败响应，不能让旧错误覆盖新状态。
- 搜索框、排序下拉框、组搜索框和动态组搜索输入必须有可随语言切换更新的 accessible name。
- 拖拽上传区域不能声明为 `role="button"` 或 `tabindex="0"` 后再包含真实按钮；键盘入口应放在真实按钮上。
- 登录、改密、创建账户、批量创建账户和创建组等前端表单必须在请求未完成期间阻止重复提交；失败时不得提前清空仍可能有效的 session，成功后再按接口结果更新本地状态。
- 前端删除文件、删除用户、重置密码、启停用户、删除组等非表单状态变更按钮必须在请求未完成期间阻止重复点击；不能只依赖后端第二次请求返回 404 / 409。
- 文件搜索输入可以在前端 debounce 降低请求抖动，但搜索和排序结果必须继续来自后端 `/api/files`，不能在前端自行扩大或缓存不可见文件集合。
- 中英文切换只在前端静态资源内实现，不引入联网翻译、CDN 字体或远程语言包。

## 7. 禁止事项
- 禁止将上传文件读取为文本、转码、压缩图片/视频、改写 EXIF / metadata 或改变原始二进制内容。
- 禁止一次性把大文件完整读入内存；后端必须按 chunk 读取 `UploadFile`。
- 禁止上传中断或校验失败后留下目标文件；只能留下已完成校验并原子替换后的文件。
- 删除文件时禁止先删 manifest 再删磁盘文件；磁盘文件存在时必须先删除成功，再移除 manifest 记录。
- 删除磁盘文件或删除后的 manifest 写入失败时必须保留/恢复 manifest 和原文件，并返回 409，不能留下 manifest 指向缺失文件。
- 禁止同名文件直接覆盖；必须自动生成非冲突保存名。磁盘文件或 manifest 中已有 `saved_relative_path` 任一占用时，都不能复用该保存路径；在 Windows 语义下只有大小写差异的路径也视为同一路径；并发上传选择目标名时必须用独占占位避免两个请求选中同一路径。
- 禁止游客上传、删除或看到非 `public` 组文件。
- 禁止 `/api/status` 向游客返回 Windows 保存目录；保存目录只对已登录用户显示。
- 禁止 `/api/status` 和 `/api/admin/status` 返回默认密码；默认密码只能在文档和管理员创建用户响应中提示。
- 禁止普通用户删除别人上传的文件或把文件发布到自己不属于的组。
- 禁止管理员通过用户端登录、用户端 session 刷新、用户端登出或用户端改密接口被当作普通用户；用户页也不得因为浏览器保留管理员 cookie 而显示管理员文件、管理员保存目录或任意文件删除入口。
- 禁止普通用户通过 `/api/admin/logout` 或其他管理员身份接口删除自己的 session、清理管理员状态或绕过管理员审计边界。
- 禁止非管理员访问账户、用户组、文件权限管理 API；删除用户和删除组也必须走管理员 API。
- 禁止未登录调用 `/api/admin/setup` 抢改默认管理员密码；当前项目默认已配置，管理员改密必须登录后调用 `/api/admin/password`。
- 禁止删除内置 `public` / `everyone` 组；仍被用户或文件引用的组必须阻止删除，不能静默扩大或收窄文件可见范围。
- 禁止删除内置 `admin` 账户；删除其他账户后，其既有文件必须转给管理员，避免同名新用户继承旧文件删除权。
- 删除用户时如果账户删除失败，必须把本次已转给管理员的文件 owner 回滚到原用户；不能留下“用户仍存在但文件已转主”的半成功状态。
- 删除用户、创建/更新用户、批量创建用户、创建/删除组、登录、登出、改密和文件删除如审计写入失败，必须回滚本次内存与磁盘状态；前端登出失败时不得清空本地 session 状态。
- 禁止批量创建账户时部分成功部分失败；重复用户名或非法组必须让整批失败。
- 批量创建接口只能创建普通 `user` 账户；需要管理员角色时必须由已登录管理员通过单个用户创建或更新接口显式设置。
- 批量创建账户单次最多 200 个，避免本地桌面工具被巨大 JSON 请求拖垮。
- 禁止把组选择退回手写组名输入；必须从 `/api/admin/groups` 已有组数据渲染选择器并支持搜索。
- 禁止绕过 `allowed_groups` 可见性检查直接下载或打包隐藏文件。
- 禁止把缺失、空列表或类型错误的 `allowed_groups` 当作 `public`；这类旧/坏 manifest 必须默认收敛到 `everyone`，避免游客可见。
- 禁止信任浏览器提供的 `relative_path`；不允许 `../`、绝对路径、盘符路径逃逸保存目录。
- 禁止用户上传占用保存目录根部控制文件名：`manifest.json`、`manifest.json.tmp`、`.lan-transfer-auth.json`、`.lan-transfer-audit.jsonl` 必须被改名保存。
- 禁止信任 manifest 中的 `saved_relative_path`；下载、打包、删除和回滚前必须拒绝绝对路径、`..`、非法段和根部控制文件名。
- 禁止审计日志记录明文密码、session token 或文件内容；只允许记录操作者、IP、动作、目标和必要元数据。
- `audit.py` 必须递归脱敏 metadata 中的 password、token、session、authorization、cookie、secret 类字段；snake_case、kebab-case、camelCase、PascalCase 写法都必须识别，连 `Set-Cookie`、`sessionId`、`authCookie` 这类复合键也不能漏，`sha256` 等文件完整性哈希不得被误脱敏。
- 禁止未处理异常把内部异常文本、路径或堆栈回显给客户端；客户端只能收到固定 500 文案，细节写日志；请求校验失败只能返回固定 422 文案，不能回显原始请求体。
- 禁止引入云服务、远程中转、CDN 或运行期外部网络依赖。

## 8. 完成标准
- 新保存目录自动生成 `.lan-transfer-auth.json`，包含默认 `admin / 12345678`、`public` 和 `everyone` 组；旧保存目录中的 `default` 必须自动迁移为 `everyone`。
- 上传后 manifest 记录原文件名、保存文件名、大小、SHA-256、原始 `lastModified`、服务端 mtime、上传时间、相对路径、`owner_username` 和 `allowed_groups`。
- 上传响应前必须完成大小校验、SHA-256 和 mtime 设置尝试。
- `/api/files`、Raw 下载、单文件 zip、批量 zip 必须按当前 session 过滤可见文件。
- `/api/files` 的搜索/排序只能作用于当前 session 已可见文件；支持 `size`、`uploaded`、`mtime`、`name`、`owner`。
- 管理员批量创建普通账户必须经过 `/api/admin/users/batch`，默认密码仍为 `12345678`；删除账户必须经过 `DELETE /api/admin/users/{username}`。
- 删除组必须经过 `DELETE /api/admin/groups/{group_id}`，且只能删除非内置、未被用户或文件引用的组。
- Web UI 默认中文；切换语言后同一浏览器用 `localStorage` 记住语言。
- 普通用户只能删除自己的文件；管理员可以删除所有文件并修改文件可见组。
- 原始单文件下载必须设置 `Content-Disposition`、`Content-Length`、`Last-Modified`、`ETag`、`X-Original-Mtime` 和合理 `Content-Type`；`Content-Disposition` 的 ASCII fallback 必须清理路径分隔符、引号、分号和控制字符，`filename*` 必须百分号编码路径分隔符。
- 单文件 zip 和批量 zip 必须使用原始 mtime 写入 zip entry，文件内容从磁盘原样写入，并按大小写不敏感规则避免重复 zip entry 名；manifest 中的 `relative_path` 和 `saved_relative_path` 在写入 zip entry 前也必须重新走路径安全校验；旧/坏 manifest 中的非法 mtime 字段必须降级到可用时间或省略响应 `Last-Modified`，不能泄漏为 500。
- 如果被选中的 zip 文件全部缺失，必须返回错误，不能返回空 zip。
- zip 构建、响应 metadata 设置或审计失败都必须尝试删除临时 zip 文件；临时文件删除失败只能写日志，不能掩盖原始 HTTP 错误；manifest 中保存路径异常必须返回 400，不能泄漏为 500。
- 登录、登出、上传、下载、删除、权限修改、账户和组变更成功后必须写 `.lan-transfer-audit.jsonl` 审计事件；管理员可通过 `/api/admin/audit` 查看最近事件。
- 上传和文件权限修改写审计失败时必须回滚本次文件/权限变更，不能返回失败但留下未审计的新文件或新可见性；上传审计成功前的 pending 文件也不能被列表、下载或打包访问。
- 上传写审计失败且物理清理也失败时，残留 manifest 记录必须标记为不可见隔离状态，不能继续通过列表、下载或打包接口访问。
- 文件删除写审计失败时必须恢复 manifest 和原文件；删除 commit 失败时必须恢复 manifest 和原文件并写入补偿审计事件，避免只留下成功删除审计；密码、账户、组、登录和登出写审计失败时必须恢复 auth/session 状态。
- README 必须说明运行、打包、测试、账户/组权限、安全、mtime 限制和 zip 下载建议。

## 9. Review 标准
- 优先检查文件内容是否会被改写、路径是否可穿越、manifest 旧路径是否可指向控制文件、游客/用户是否可绕过组权限、普通用户是否可删除他人文件、删除用户后同名新用户是否会继承旧文件删除权、删除组是否会破坏文件可见性、管理员 session 是否可绕过、同名文件大小写差异是否覆盖、控制文件名是否被用户上传占用、固定 manifest/auth 临时文件是否被上传占用或失败残留、`.part` 是否会伪装成完整文件、审计日志是否泄漏密码或 token。
- 修改批量创建、用户/组删除、搜索/排序或组选择器后，必须补充或更新 `tests/test_api.py` 中对应 API 行为测试，并至少运行 `node --check` 检查相关前端脚本。
- 修改 `requirements.txt` 前必须更新 `docs/OPEN_SOURCE_AUDIT.md`，说明许可证、维护状态、冲突点、采用范围和回滚方案。
- 修改 PyInstaller spec 后必须确认 `lan_transfer/static/` 被打包进 exe。
- 修改版本号、作者、发布文件名、Release Notes、PyInstaller spec 或 `scripts/build_exe.ps1` 后，必须重新运行打包脚本并确认 `release-assets/SHA256SUMS.txt` 覆盖所有发布资产。
- 修改前端上传逻辑后必须确认仍发送 `relative_path`、`last_modified_ms` 和 `size`，且不把文件读成文本。
- 修改日志配置或保存目录切换逻辑时，必须确认旧 logging handler 被关闭，避免 Windows 文件句柄残留。

## 10. 常见风险
- 默认密码 `12345678` 方便初始化但很弱；README 必须提示首次运行后修改管理员密码和新建用户密码。
- session cookie 用于让大文件下载保持浏览器原生流式下载；不要改成前端 `fetch` blob 下载。
- `/api/status`、`/api/session` 和 `/api/admin/session` 在 header token 有效时必须刷新 same-origin session cookie，避免列表 API 可用但原生下载失效；刷新响应不得在 JSON 中回显 session token。
- 用户页存在管理员同源 cookie 时，游客状态的列表、状态和“全部打包”仍必须收敛到游客/普通用户可见范围，不能让原生导航下载按管理员权限扩大文件集合。
- 文件搜索和排序如果在前端自行过滤，容易泄漏已经被后端过滤掉的文件；必须以后端 `/api/files` 返回结果为准。
- iOS / Android / 浏览器下载目录是否保留 mtime 不由服务端完全控制；README 必须继续保留 zip 建议。
- Raw 下载只能提供 `Last-Modified` 等 HTTP header，不能强制浏览器把落盘 mtime 写成原始时间。
- PyInstaller 许可证是 GPL with bootloader exception；分发 exe 前需保留相关许可证说明。
- `0.0.0.0` 监听会暴露到可达网络；游客虽然只能看 `public` 文件，也会暴露该组文件名和下载能力。

---
> Source: [NextWeb4/lan-file-transfer](https://github.com/NextWeb4/lan-file-transfer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
