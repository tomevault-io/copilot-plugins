## surkaa-pad

> > 生成时间：2026-05-02 | Git: `6ee3b21`

# CLAUDE.md

> 生成时间：2026-05-02 | Git: `6ee3b21`

## 项目概述

SurKaa Pad 是一款基于 Tauri 2 的端到端加密日记应用。前端 Vue 3 + TypeScript + Quasar，后端 Rust。日记正文用 AES-256-GCM 加密，附件用 AES-256-CTR 流式加密，密钥通过 Argon2id 从主密码派生，加密后同步至阿里云 OSS。

## 常用命令

```bash
# 前端
pnpm install                    # 安装依赖
pnpm dev                        # Vite 开发服务器 (端口 5173)
pnpm build                      # 类型检查 + 构建
pnpm tauri:msi:dev              # Tauri 桌面开发模式
pnpm tauri:msi:build            # Tauri 桌面生产构建
pnpm tauri:android:dev          # Tauri Android 开发模式
pnpm tauri:android:build        # Tauri Android APK 构建

# 后端 (Rust, 在 src-tauri 目录下执行)
cargo build
cargo test                      # 运行全部 Rust 测试
cargo test -- <测试名>           # 运行单个测试
cargo clippy                    # Rust 代码检查

# 前端测试
pnpm vitest                     # Vitest 测试（监视模式）
pnpm vitest -- run              # 单次运行
pnpm tsc                        # 仅类型检查 (vue-tsc --noEmit)
```

## 架构

### 前端 (`src/`)

- **路由** (`src/router/index.ts`): Hash 模式，带 keep-alive 页面缓存管理。`/` (Unlock) → `/diary-list`, `/diary-detail/:id?`, `/diary-search`, `/settings`。后退导航会自动销毁离开页面的缓存。
- **Pinia Store**:
  - `config.ts` — 通过 Tauri Store 插件持久化到 `settings.json`。`useTauriConfig()` 返回一个与 Rust 后端双向自动同步的 Vue ref。配置项包含主题、生物识别开关、加密后的 OSS 配置、置顶日记 ID 等。
  - `data.ts` — 日记列表 ID、摘要缓存、当前编辑状态的内存存储。
- **Tauri 绑定** (`src/bindings.ts`): 由 tauri-specta 从 Rust 命令签名自动生成，**请勿手动编辑**。仅 Windows 调试构建时自动重新导出。
- **API 包装** (`src/utils/api.ts`): 解包 tauri-specta 的 `Result<T, E>` 类型——错误时 throw，成功时返回数据。
- **编辑器** (`src/components/editor/`): 基于 Tiptap/ProseMirror 的富文本编辑器（`TiptapEditor.vue`），通过自定义 Node 扩展（`tiptap-extensions/`）支持图片、视频、音频、文件等附件内联展示。HTML ↔ Markdown 双向转换由 `markdownConverter.ts` 处理，附件以 `[[TYPE:filename]]` 标记语法存储在 Markdown 中。
- **录音组件** (`src/components/CaptureAudioDrawer.vue`): 使用 `MediaRecorder` API 录音，录制完成组装完整文件后上传。
- **媒体操作** (`src/composables/useMediaAction.ts`): 统一管理附件上传、录音、拍照等操作的 composable。包含 `uploadAttachment`（文件选择）、`uploadMemoryAttachmentChunked`（内存数据分片上传）、附件操作（旋转/切换加密/保存解密/缓存）等。

### 后端 (`src-tauri/src/`)

每个领域模块目录包含 `mod.rs`、类型定义、命令、错误和测试：

| 模块 | 职责 |
|---|---|
| `cryptos` | AES-256-GCM 加解密、AES-256-CTR 流式加解密、Argon2id 密钥派生。`Crypto` 是 `Arc<RwLock<Option<DerivedKey>>>` 的可克隆句柄。`create_ctr_cipher()` 返回独立的 `Aes256Ctr` 实例，用于分片上传场景下跨 chunk 顺序加密。 |
| `diaries` | 加密日记清单的 CRUD。每篇日记在 OSS 存储为 `{id}/manifest.enc`。标题取自正文首行。含迁移系统（V1→V2 为附件添加 etag 字段）。 |
| `attachments` | 附件管理（添加、删除、旋转、切换加密状态、分片上传）。包含自定义 `attachment://` URI Scheme 协议，用于在界面中直接内联展示解密后的媒体内容。分片上传三阶段：start → upload chunks → finish/abort，状态存储在 `AppState.chunked_uploads`。附件 ID 分配使用 MEX 算法，分配器存储在 `AppState.attachment_allocators`。 |
| `object` | 对 `s3` crate 的封装，提供对 OSS 的流式上传/下载/删除操作，以及分片上传（initiate/upload_part/complete/abort）。未加密附件使用预签名 URL 直接访问。通用 MIME 类型 fallback 常量 `STREAM_MIME_TYPE` 定义在 `object_types.rs`。 |
| `caches` | 两层缓存：`DiaryMemoryCache`（内存 DashMap，按日记 ID 索引）和 `LocalFileCache`（磁盘缓存，用 MD5 记录 etag 便于缓存校验）。`LocalFileCache` 支持 `SaveHandle`（流式保存）和 `ChunkedSaveHandle`（分片增量写入），均使用临时文件 + 原子重命名确保崩溃安全。 |
| `tasks` | `TaskPool` 管理可取消的异步任务，向前端返回取消令牌。 |
| `stream` | `ByteStream` 类型别名及相关工具：CTR 流加密适配器、数据收集、文件转流。 |
| `state` | `AppState`——中心化管理状态，持有 Crypto、OssClient（通过 OnceLock 延迟初始化）、缓存层、任务池、分片上传状态（`chunked_uploads: DashMap`）、附件 ID 分配器（`attachment_allocators: DashMap`）。 |
| `storages` | OSS 路径工具：`remote_manifest_key(id)` → `"{id}/manifest.enc"`，`remote_attachments_key(id, filename)` → `"{id}/{filename}"`。 |
| `utils` | 文件工具、基于时间戳的降序 ID 生成、用于通过 Tauri Channel 发送类型化事件的 MessageSender trait。 |

### 加密流程

1. 用户输入主密码 → Argon2id 从密码 + salt 派生 256 位 DEK。
2. DEK 存储在 `Crypto` 的 `Arc<RwLock<Option<DerivedKey>>>` 中，会话期间有效。
3. 日记文本：JSON manifest 用 AES-256-GCM 加密（nonce 前置在密文前）。
4. 附件：AES-256-CTR 流式加密（nonce 存储在 `AttachmentMeta` 中），支持通过计数器偏移实现 Range 请求解密。
5. 生物识别解锁：将 DEK 的 hex 字符串存储在平台密钥库中，通过指纹/面容验证后取出。

### 缓存策略

- `DiaryMemoryCache`: `DashMap<String, Arc<(DiaryManifest, String)>>`，按日记 ID 索引，值为 (manifest, 远程 etag)。每次读取优先检查。
- `LocalFileCache`: 磁盘缓存目录 `{app_cache_dir}/lfc/`。每个 key 对应 `{key}.data` + `{key}.md5`。MD5 存储的是远程 etag，用于与 OSS 比对决定是否重新下载。

### 关键设计

- **Tiptap 编辑器** (`TiptapEditor.vue`): 基于 Tiptap/ProseMirror，使用 `StarterKit` + 自定义 Node 扩展。附件 URL 直接存储为节点 `src` 属性（而非通过外部 Storage 间接查找），`updateSrc()` 通过 `tr.setNodeMarkup` 精确更新单个节点属性。编辑内容通过 `onUpdate` → `htmlToMarkdown` → `emit('update:modelValue')` 同步到父组件。附件 map 变化不再触发全文重渲染。
- **Markdown 转换** (`markdownConverter.ts`): HTML 端到 Markdown 存储端双向转换。附件以 `[[TYPE:filename|config]]` 语法嵌入 Markdown。转换逻辑有 50 个单元测试覆盖。
- **URI Scheme 协议** (`attachment_protocol.rs`): 前端请求 `http://attachment.localhost/{diary_id}/{filename}`，协议处理器从 OSS/缓存获取数据并流式解密后返回。支持 HTTP Range 请求（最大单次 1MB），适用于视频拖动播放。
- **etag 缓存校验**: 读取日记时先 HEAD 请求获取远程 etag，与内存缓存和本地文件缓存比对，命中则跳过下载。详见 `diaries/diary.rs:get_diary()`。
- **迁移系统**: `DiaryManifest` 含 `version` 字段，`MigrationRegistry` 按版本顺序链式注册迁移步骤（当前 V2）。
- **前端配置持久化**: `useTauriConfig()` 创建 Vue `customRef`，初始化时自动读取、修改时自动保存，通过 `onKeyChange` 响应跨窗口变更。
- **分片上传**: 粘贴等内存数据场景使用三阶段分片上传（`cmd_start_chunked_upload` → `cmd_upload_chunk` × N → `cmd_finish_chunked_upload`），5MB 分片。S3 multipart ETag 格式为 `"{md5}-{part_count}"`，存入 `AttachmentMeta.etag` 作为不透明字符串。进度通过每个 chunk 完成后返回的 `uploadedBytes/totalBytes` 更新。
- **Rust 日志**: 使用 `tauri_plugin_log::log`（`use tauri_plugin_log::log;`），不要单独引入 `log` crate。

## 工作规范

### 测试要求

- **能写测试就写测试**。任何涉及逻辑判断、边界条件、数据处理、加解密、缓存校验的代码都应该有对应的测试用例。
- 前端用 Vitest + happy-dom（DOM 环境），测试文件放在 `__test__/` 目录下。需 DOM 的测试在文件顶部加 `// @vitest-environment happy-dom` 指令。
- 后端 Rust 测试放在各模块的 `*_tests.rs` 文件中，或通过 `#[cfg(test)]` 内嵌在模块内。
- 写完测试后务必跑一遍确认通过：前端 `pnpm vitest -- run`，后端 `cargo test`。
- 部分 Rust 测试需要阿里云 OSS 凭证，需在 `src-tauri/` 下配置 `.env` 文件（`ALIYUN_KEY`、`ALIYUN_SECRET`、`ALIYUN_BUCKET_NAME`、`ALIYUN_ENDPOINT`、`ALIYUN_REGION`），并使用 `serial_test` 控制互斥执行。

### Rust 代码检查

- 所有 Rust 修改完成后，运行 `cargo clippy` 检查并修复警告。
- 不要在代码中残留 `#[allow(dead_code)]` 或 `#[allow(unused)]` 标注，除非有明确的阶段性保留理由。

### 分支与提交

- **大修改开新分支**。任何涉及多文件、新功能、重构的修改都应在独立 feature 分支上进行，不要在 `master` 上直接大量修改。
- **小步提交**，不要让未提交的修改大量堆积。每完成一个可独立工作的逻辑单元就提交一次。
- 提交信息遵循已有风格：`<type>: <描述>`，如 `feat:`、`fix:`、`refactor:`、`perf:`、`docs:`、`test:`。

### 不要做的事

- **严禁手动编辑 `src/bindings.ts`**。该文件由 tauri-specta 在 Windows 调试构建时自动生成，手动修改会在下次构建时被覆盖。新增或修改命令签名应通过 Rust 端的 `#[tauri::command]` 和 `#[specta]` 导出。
- 不要随意引入新的重量级依赖，优先复用现有 crate/package。
- 不要在 PR/提交中包含 `.env`、`node_modules`、`target` 等敏感或构建产物文件。

---
> Source: [surkaa/surkaa-pad](https://github.com/surkaa/surkaa-pad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
