## phone-record-manager

> - `phone_record_manager/` 是主应用包。

# AGENTS.md

## 1. 项目结构
- `phone_record_manager/` 是主应用包。
- `phone_record_manager/app.py` 负责应用启动入口。
- `phone_record_manager/ui/` 放 Qt 界面层。
- `phone_record_manager/services/` 放业务编排层。
- `phone_record_manager/db/` 放 SQLite 表结构、仓储和持久化操作。
- `phone_record_manager/security/` 放主密码哈希和敏感字段处理。
- `tests/` 放单元测试。
- `scripts/` 放打包脚本。
- `.github/workflows/release.yml` 在 tag 推送时构建并发布 GitHub Release。
- `phone_record_manager.spec` 是 PyInstaller 配置。

## 2. 运行命令
- 本地源码运行：
  `python -m venv .venv`
  `.venv\Scripts\python -m pip install -r requirements.txt`
  `.venv\Scripts\python -m phone_record_manager`
- 数据目录默认写入 `%LOCALAPPDATA%\PhoneRecordManager\phone_records.sqlite3`。
- 可用 `PHONE_RECORD_MANAGER_HOME` 覆盖数据目录。

## 3. 测试命令
- 当前可用测试命令：`python -m unittest discover -s tests`
- 目前仓库未发现 lint / format 命令。

## 4. 构建命令
- 当前发布打包命令：`.\scripts\build_release.ps1`
- 仅构建 PyInstaller 目录包：`.\scripts\build_exe.ps1`
- 单独构建 MSI：`.\scripts\build_msi.ps1 -AppSource <PyInstaller输出目录>`
- 当前 PyInstaller 产物是 Windows `.exe` 文件夹包和压缩发布包。
- MSI 依赖 WiX Toolset CLI；脚本不会自动安装 WiX，未安装时只能验证 portable ZIP。

## 5. 代码风格
- Python 代码默认按现有写法：`from __future__ import annotations`、类型注解、显式仓储/服务分层。
- 数据库写入优先走 `with connection:`，避免跨调用漏事务。
- UI 层只做交互和展示，不直接写 SQL。
- 业务校验优先放 `services/`，持久化细节放 `db/`。

## 6. 模块边界
- `db/` 只处理 SQLite 表、查询、写入和快照。
- `services/` 只做业务编排、日志和跨仓储动作。
- `ui/` 只处理界面状态、表单输入和用户操作。
- `security/` 只处理密码哈希、敏感字段加解密与判定。
- 导入导出、备份恢复必须经过服务层，不允许 UI 直接拼装数据库快照。

## 7. 禁止事项
- 不允许绕过 `PHONE_RECORD_MANAGER_HOME` 和 `LOCALAPPDATA` 直接改写固定绝对路径。
- 不允许在 UI 中直接访问表结构或拼 SQL。
- 不允许新增网络依赖来实现本地数据功能。
- 不允许把主密码、数据库内容、备份内容、`.wixpdb`、`version_info.txt` 之外的隐私信息打进发布包。
- 不允许在未验证测试结果前声称修复完成。

## 8. 完成标准
- 修改后必须至少跑一遍 `python -m unittest discover -s tests`。
- 涉及打包时必须验证 `.\scripts\build_exe.ps1` 能生成可运行产物。
- 涉及发布打包时必须验证 `.\scripts\build_release.ps1 -SkipMsi` 至少能生成 portable ZIP 和 `SHA256SUMS.txt`。
- 涉及备份、导入、恢复、注册记录、换号流程时，必须补回归测试。
- 涉及启动入口或打包入口时，必须确认 `python -m phone_record_manager` 与 PyInstaller 入口一致。

## 9. Review 标准
- 先看是否破坏数据完整性、主密码安全、备份恢复和软删除语义。
- 先看是否引入隐私泄露、明文导出或多余联网。
- 先看是否把业务逻辑塞进 UI。
- 先看测试是否覆盖异常路径和边界输入。
- 先看打包是否包含不该随发布的本地配置、缓存或个人数据。

## 10. 常见风险
- 备份/导入会影响全库数据，必须谨慎验证恢复逻辑。
- PyInstaller 依赖收集不全会导致 Qt 或 `cryptography` 运行时报错。
- SQLite 约束、软删除和外键级联最容易在批量修改时出问题。
- 新增安装器或签名流程前，必须确认不会把开发机上的个人证书、`.env`、缓存和本地数据库带进包里。
- MSI 只能打包 `dist/` 或 `release-assets/` 中已经过隐私扫描的 PyInstaller 产物。

---
> Source: [NextWeb4/phone-record-manager](https://github.com/NextWeb4/phone-record-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
