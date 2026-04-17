## oss-manager

> **OSS Manager** 是一套基于 Rust 语言开发的高性能对象存储（S3 兼容）文件传输解决方案。它具备断点续传、精确并发控制以及多云厂商支持等特性。

# OSS Manager 上下文记忆

## 项目概览

**OSS Manager** 是一套基于 Rust 语言开发的高性能对象存储（S3 兼容）文件传输解决方案。它具备断点续传、精确并发控制以及多云厂商支持等特性。

本项目采用 Rust Workspace 模式组织，包含以下成员：

*   **`oss-core`**: 包含核心业务逻辑的库。
    *   **S3 客户端工厂**: 针对 AWS, Cloudflare R2, Aliyun, Tencent 等进行了优化。
    *   **状态管理**: 使用 SQLite (`sqlx`) 持久化任务进度，以支持断点续传。
    *   **传输逻辑**: 实现了递归上传/下载、服务端复制/移动以及跨 Bucket 传输。
*   **`oss-cli`**: 基于 `oss-core` 构建的命令行接口。
    *   提供 `cp`, `mv`, `rm`, `ls`, `tree` 等命令。
    *   交互式管理配置文件的 Profile。
    *   **打包**:
        *   包含 `package.metadata.deb` 用于构建 Debian 软件包 (`.deb`).
        *   包含 `setup.iss` 用于使用 Inno Setup 构建 Windows 安装程序。
*   **`oss-desktop`**: 图形化应用程序。
    *   基于 Tauri 2, pnpm, 和 React 19 构建。
    *   **设计系统**: 实现了类 VSCode 的布局（活动栏、侧边栏、面板、状态栏）。
    *   **高级特性**: 多窗口支持（标签页拖出）、全局搜索、文件预览。

## 构建与运行指南

### 先决条件
*   Rust 工具链 (1.75+)
*   Node.js (v20+) 用于桌面端前端。
*   CMake (构建某些 C 依赖项如 `aws-lc-sys` 所需)。

### 核心命令

*   **构建工作区 (Release)**:
    ```bash
    cargo build --release
    ```
*   **运行 CLI (开发)**:
    ```bash
    cargo run -p oss-cli -- <ARGS>
    # 示例: cargo run -p oss-cli -- --help
    ```
*   **桌面端开发**:
    ```bash
    cd oss-desktop
    npm run tauri dev
    ```
*   **运行测试**:
    ```bash
    cargo test
    ```

## 代码库架构

### `oss-core` (库)

*   **`src/lib.rs`**: 入口点，导出模块和 `create_client` 工厂函数。
*   **`src/config.rs`**: 管理用户配置（JSON 序列化）和全局设置。
*   **`src/db.rs`**: 使用 `sqlx` 进行 SQLite 数据库交互。定义了用于持久化的 `Task` 和 `Part` 结构。
*   **`src/ops.rs`**: 低级 S3 操作封装（List, Copy, Delete, Head, Read）。处理不同提供商的差异。
*   **`src/transfer.rs`**: 高级编排。包含 `TransferManager`，处理 `upload`（上传）、`download`（下载）、`copy_cloud`（云端复制）、`move_cloud`（云端移动）、`remove`（删除）的递归和逻辑。
*   **`src/uploader.rs`**: 实现 `ResumableUploader`（带状态同步的分片上传）。
*   **`src/downloader.rs`**: 实现 `Downloader`（并发范围 GET 请求）。

### `oss-cli` (应用)

*   **`src/main.rs`**: CLI 的单一源文件。
    *   使用 `clap` 进行参数解析。
    *   使用 `inquire` 进行交互式提示。
    *   将命令分发给 `TransferManager` 或 `ConfigManager`。

### `oss-desktop` (GUI 应用)

**技术栈**
*   **前端**: React 19, TypeScript, Tailwind CSS v4.
*   **后端**: Rust (Tauri 2), 使用 `oss-core`.
*   **状态管理**: React Context (`TransferContext`, `I18nContext`, `NotificationContext`).

**关键特性与实现细节**
*   **布局**:
    *   **活动栏 (ActivityBar)**: 在文件、配置、设置之间切换。
    *   **侧边栏 (Sidebar)**: 上下文感知的侧面板（例如文件树、配置列表）。
    *   **主内容区**: 标签页接口。
    *   **状态栏 (Status Bar)**: 全局通知和任务状态。
*   **窗口管理**:
    *   **标签页拆分 (Tear-out)**: 将标签页拖出窗口会创建一个新的 Tauri 窗口 (`create_window` 命令)。
    *   **合并**: 通过 Tauri 事件支持在窗口之间拖拽合并标签页。
*   **文件浏览器**:
    *   **拖放**: 支持从操作系统上传文件以及在文件夹之间移动文件。
    *   **预览**: 集成 `FileDetails` 视图，用于查看图像和文本/代码（最大 5MB）。
    *   **搜索**: 标题栏搜索输入框过滤当前视图。
*   **通知**: 自定义通知中心，带有传输进度条。

## 开发规范

*   **Async/Await**: 项目完全基于异步，由 `tokio` 驱动。
*   **错误处理**: 使用 `anyhow` 进行结果传播。
*   **数据库**: 迁移由 `sqlx` 处理。代码在启动时运行迁移。
*   **日志**: 使用 `tracing` 进行日志记录。`tauri-plugin-log` 处理桌面端日志。
*   **国际化 (I18n)**: 前端使用自定义 `I18nContext` 进行中/英切换。

## 关键配置文件

*   **`~/.oss-manager/config.json`**: 存储用户配置和设置。
*   **`~/.oss-manager/oss.db`**: SQLite 数据库文件，用于跟踪活动和已完成的传输任务。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/u-00a0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
