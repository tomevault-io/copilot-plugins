## classlan

> 目的：帮助自动化/AI 代码助手快速理解此仓库的结构、构建方式、重要位置与修改约束。

# AGENTS.md — AI 代码助手指南

目的：帮助自动化/AI 代码助手快速理解此仓库的结构、构建方式、重要位置与修改约束。

## 快速概览
- **语言/框架**：C++ (Qt 6)、CMake 构建系统
- **主要组件**：`Student/`（客户端）、`Teacher/`（服务/管理端）、`llm-memory/`（协议与设计文档）、`script/`（构建脚本）

## 构建与运行
- **推荐工具链**：Qt 6.11.x (mingw_64)、MinGW-w64 (64-bit)、CMake、Windows PowerShell
- **自动构建**：`script/build.ps1`（优先使用，处理 Qt/MinGW 路径、CMake 配置与打包）
- **调试构建**：`script/debug_build.ps1`
- **发布构建**：`script/release_build.ps1`

## 重要文件与链接
| 类型 | 文件路径 |
|------|---------|
| 构建脚本 | `script/build.ps1`、`script/debug_build.ps1`、`script/release_build.ps1` |
| 协议文档 | `llm-memory/comm_stat_machine.md`、`llm-memory/two_step_auth.md`、`llm-memory/chat_protocol.md` |
| 学生端通讯 | `Student/inc/client/class_client.h`、`Student/src/client/class_client.cpp` |
| 教师端通讯 | `Teacher/inc/server/class_server.h`、`Teacher/src/server/class_server.cpp` |
| 学生管理服务 | `Teacher/inc/service/student_mgmt_service.h` |
| 项目常量 | `Student/inc/main.h`、`Teacher/inc/main.h` |

## 核心协议
- **格式**：JSON 紧凑格式 + `\n` 换行符分隔
- **传输**：Qt QSslSocket 加密通讯（TLS 1.2+）
- **消息模板**：`{"type": "TYPE", "timestamp": <ms>, "data": {...}}`

## 核心消息类型
| 类型 | 方向 | 功能 |
|------|------|------|
| AUTH_REQUEST/RESPONSE | 双向 | 口令验证（两步认证第一步） |
| SIGN_IN_REQUEST/RESPONSE | 双向 | 学生身份签到（第二步） |
| HEARTBEAT | C→S | 心跳保活 + 举手状态 |
| CHAT_MESSAGE | 双向 | 聊天消息（服务器广播） |
| SET_HAND | S→C | 服务器指令：清除学生举手 |
| DISCONNECT | 双向 | 连接中断通知 |

## 代码架构
```
Student/
├── inc/     # 头文件
│   ├── main.h        # 全局常量、工具函数
│   ├── client/       # 通讯类
│   └── widget/       # UI 组件
└── src/     # 源文件（与 inc/ 镜像）

Teacher/
├── inc/
│   ├── main.h
│   ├── server/       # SSL 服务器
│   ├── service/      # 业务服务
│   └── widget/       # UI 组件
└── src/
```

## 命名规范
- **枚举/状态**：`enum class ClientState { CS0_DISCONNECTED, ... };`
- **类名**：`PascalCase`（`ClassClient`、`MainWindow`）
- **成员变量**：`m_` 前缀（`m_socket`、`m_password`）
- **方法名**：`camelCase`（`connectToServer()`）
- **常量**：`k` 前缀（`kConnectTimeout`）

## 依赖项
| 依赖 | 版本 | 用途 |
|-----|------|------|
| Qt | 6.11.x | GUI + 网络框架 |
| OpenSSL | 3.6.2 | SSL/TLS + 证书生成 |
| CMake | 3.21+ | 构建系统 |
| MinGW-w64 | 64-bit | C++ 编译器 |
| libvncserver | 预编译 | VNC 屏幕广播 |

## 近期修改记录（2026-05-12）
1. **移除Unignore按钮**：Kick按钮改为toggle行为，再次点击可取消忽略
2. **Screen窗口显示修复**：Student显示学生姓名，ID显示学号
3. **自适应卡片布局**：View视图卡片从左到右、从上到下排列，列数根据窗口宽度自动调整
4. **移除Status显示**：Screen窗口不再显示"Waiting for screen"状态

## UI 实现规范
- **静态UI必须使用ui文件**：用户界面上的静态实现必须通过Qt Designer创建的.ui文件实现，禁止纯代码硬编码UI布局

## 提交命名规范
- **格式**：`[Type] Description`
- **类型说明**：
  - `[New]`：新增功能或模块
  - `[Fix]`：修复bug或问题
  - `[Update]`：更新、改进现有功能
  - `[Security]`：安全相关修改
  - `[Release]`：版本发布提交

## 版本发布规则
- **标签命名**：使用 `vX.Y.Z` 格式标识发布版本（如 `v0.0.1`）
- **分支策略**：发布前创建 `vX.Y.Z` 分支进行最终测试
- **发布流程**：
  1. 创建发布分支：`git checkout -b vX.Y.Z`
  2. 提交最终修改：`git commit -m "Release vX.Y.Z"`
  3. 推送到远程：`git push origin vX.Y.Z`
  4. 创建标签：`git tag vX.Y.Z`
  5. 推送标签：`git push origin vX.Y.Z`
- **更新源**：通过 Gitee Releases 获取最新版本

## AI 行为准则
- **链接而非复制**：引用 `llm-memory/` 下的协议文档，不复制内容
- **协议变更需审查**：涉及通信协议、状态机或认证逻辑的修改需人工审查
- **代码风格**：遵循项目命名规范，保持 C++17 + Qt 风格一致性

## 已知限制
1. **无测试框架**：项目缺乏自动化测试
2. **Qt版本固定**：当前工具链为 Qt 6.11.0 (mingw_64)
3. **Windows专用**：脚本为 PowerShell，仅支持 Windows

——
该文件为精简版说明，旨在让 AI 助手快速上手。详细协议与设计文档请查看 `llm-memory/` 目录。

---
> Source: [NotchBrown/ClassLAN](https://github.com/NotchBrown/ClassLAN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
