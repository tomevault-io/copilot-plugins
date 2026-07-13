## offline-utility-suite

> - 当前仓库用于恢复并发布用户提供的离线工具成品：三个 PyInstaller/Tkinter Windows 程序和一个单文件 SRT 编辑器网页。

# 项目协作规则

## 1. 项目结构
- 当前仓库用于恢复并发布用户提供的离线工具成品：三个 PyInstaller/Tkinter Windows 程序和一个单文件 SRT 编辑器网页。
- 恢复后的 Python 源码放在 `src/` 的独立子目录；网页源码放在 `web/`；测试放在 `tests/`；构建脚本放在 `scripts/`；发布产物只放在被 Git 忽略的 `release-assets/`。
- 原始 EXE 仅作为恢复证据，不复制进 Git 跟踪范围。

## 2. 运行命令
- 桌面源码运行前设置 `$env:PYTHONPATH='src'`；Folder Locker 用 `python -m folder_locker.app`，SRT 去重用 `python -m srt_dedupe.app`。
- 网页项目必须保持可直接双击 HTML 离线运行，不得要求服务端。

## 3. 测试命令
- 自动测试：`$env:PYTHONPATH='src'; python -m unittest discover -s tests -v`。
- 构建后 GUI/ZIP/元数据烟雾测试：`.\tests\run_ui_smoke.ps1`；CI 无交互桌面时只允许使用 `-SkipLaunch`，本地发布前必须实际启动。
- 当前未发现独立 lint / format 命令；提交前至少运行 `python -m compileall -q src scripts tests`，后续增加 lint 时再补充准确命令。

## 4. 构建命令
- 构建命令：`powershell -ExecutionPolicy Bypass -File scripts/build.ps1`；指定隔离环境时传 `-Python .\.build-venv\Scripts\python.exe`。
- 构建脚本必须先运行测试，再生成两个 PyInstaller 单文件 EXE、静态 HTML、便携 ZIP 和 SHA256；不得把本机绝对路径写入配置。

## 5. 代码风格
- Python 使用 UTF-8、4 空格缩进、类型提示和职责清晰的小函数；UI 文案集中管理，避免散落硬编码。
- HTML/CSS/JavaScript 保持无构建、无外部 CDN 的离线单文件能力；中英文文案集中映射，语言首选项存入 `localStorage`。

## 6. 模块边界
- `src/folder_locker/core.py` 负责容器格式、加解密、安全路径和 ACL；`app.py` 不实现密码学或文件格式。
- `src/srt_dedupe/core.py` 负责解析、去重、时间轴与序列化；`app.py` 只负责批处理编排与 UI 状态。
- `src/utility_suite/` 只放共享署名和本机语言设置；`web/srt-editor.html` 必须自包含，不引用 Python 模块。

## 7. 禁止事项
- 不得将 `.env`、令牌、密钥、凭据、本机路径、缓存、构建目录、旧 `.git` 历史或原始恢复中间物提交到仓库。
- 不得声称能从 PyInstaller 成品恢复原始注释、变量名或完全一致的源码；任何语义重建必须有行为测试支撑。
- 不得伪造数字签名；无证书时只设置作者、版权、产品名、公司名和版本元数据。
- 文件夹保护功能不得采用不可逆加密或可能导致数据丢失的删除/覆盖流程。

## 8. 完成标准
- 每个工具都有可维护源码、双语 UI/说明、作者信息、许可证、可复现构建脚本和对应测试。
- Windows 产物能启动，ZIP 解压后能运行，核心功能通过测试，Release 资产具有 SHA256。

## 9. Review 标准
- Review 必须核对：作者与版本元数据、离线边界、路径与编码边界、SRT 换行/时间格式、去重不变量、文件锁定可逆性、Git 跟踪文件清单和发布哈希。
- 修改核心逻辑必须增加或更新回归测试；仅凭 GUI 外观不得判定功能正确。

## 10. 常见风险
- Python 3.12/3.13 字节码反编译支持不完整，恢复结果可能需要依据运行行为重建。
- 文件夹“锁定”通常只是 Windows 属性/重命名伪装，不等同加密；文档和 UI 不得夸大安全性。
- SRT 文件可能含 BOM、CRLF、逗号/点毫秒、空字幕块、重复时间轴和非 UTF-8 编码，解析与导出必须覆盖这些边界。
- Windows Defender 可能对未签名 PyInstaller 单文件包误报；发布说明必须明确未签名状态和 SHA256 校验方式。

---
> Source: [NextWeb4/offline-utility-suite](https://github.com/NextWeb4/offline-utility-suite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
