## byakko-writer

> > **重要提示**: 根据项目规则，Claude 不应主动执行以下操作：

# CLAUDE.md - 白喵写作 (Byakko Writer) 项目指南

> **重要提示**: 根据项目规则，Claude 不应主动执行以下操作：
> - 不要自动重启应用
> - 不要自动运行项目
> - 不要自动安装依赖
> - 如果需要执行命令，只提供命令语句让用户自己执行

## 项目概述

白喵写作 (Byakko Writer) 是一款基于 Tauri + Vue 3 的桌面端小说写作应用，专为小说创作者设计，集成 AI 辅助功能，帮助作者更高效地创作。

## 技术栈

- **前端**: Vue 3 + TypeScript + Element Plus + Pinia
- **后端**: Rust + Tauri
- **数据库**: SQLite (通过 sqlx 访问)
- **构建工具**: Vite

## 项目结构

```
byakko-writer/
├── src/                    # 前端源码
│   ├── components/         # Vue 组件
│   │   ├── Sidebar.vue     # 侧边栏（卷/章节树）
│   │   ├── WritingArea.vue # 编辑区域
│   │   ├── AIChat.vue      # AI 对话面板
│   │   ├── OutlineEditor.vue    # 大纲编辑器
│   │   ├── CharacterCardManager.vue  # 角色卡管理
│   │   ├── SnapshotManager.vue  # 版本快照管理
│   │   ├── QuickReferencePanel.vue   # 速查面板
│   │   └── SettingsDialog.vue   # 设置面板
│   ├── stores/             # Pinia 状态管理
│   │   ├── book.ts         # 书籍相关状态
│   │   └── config.ts       # 配置状态
│   ├── views/              # 页面视图
│   │   ├── Home.vue        # 首页（书籍列表）
│   │   └── Editor.vue      # 编辑器页面
│   ├── api/                # API 封装
│   │   ├── aiChat.ts       # AI 对话 API
│   │   └── rag.ts          # RAG 检索 API
│   └── types.ts            # TypeScript 类型定义
├── src-tauri/              # Tauri 后端
│   ├── prompts/            # AI 提示词文件
│   │   ├── system/         # 系统提示词
│   │   └── tools/          # 工具提示词
│   └── src/
│       ├── main.rs         # 应用入口
│       ├── db.rs           # 数据库初始化和连接
│       ├── models/         # 数据模型
│       │   ├── ai.rs       # AI 相关模型
│       │   ├── book.rs     # 书籍模型
│       │   ├── chat.rs     # 对话模型
│       │   ├── outline.rs  # 大纲模型
│       │   └── ...
│       └── services/       # 业务服务层
│           ├── chat_service.rs      # AI 对话服务
│           ├── tool_call_service.rs # 工具调用服务
│           ├── prompt_service.rs    # 提示词服务
│           ├── outline_service.rs   # 大纲服务
│           ├── style_service.rs     # 写作风格服务
│           ├── auto_style_service.rs # 自动风格分析
│           ├── character_service.rs # 角色卡服务
│           ├── snapshot_service.rs  # 快照服务
│           ├── summary_service.rs   # 摘要服务
│           └── config_service.rs    # 配置服务
├── package.json
└── README.md
```

## 数据库结构

SQLite 数据库文件位于 `%APPDATA%/byakko-writer/data.db`

### 表结构

- **books**: 书籍信息
- **volumes**: 卷信息
- **chapters**: 章节信息
- **outlines**: 大纲信息（粗纲+细纲）
- **chapter_snapshots**: 章节版本快照
- **chapter_summaries**: 章节摘要（短摘要+长摘要+标签）
- **chat_sessions**: AI 对话会话
- **chat_messages**: AI 对话消息
- **character_cards**: 角色卡信息
- **writing_styles**: 写作风格分析结果

## 常用命令

```bash
# 开发模式
pnpm tauri dev

# 构建生产版本
pnpm tauri build

# 仅构建前端
pnpm build

# 检查 Rust 代码
cd src-tauri && cargo check

# 格式化 Rust 代码
cd src-tauri && cargo fmt
```

## 开发注意事项

### 添加新的 Tauri 命令

1. 在 `src-tauri/src/services/` 下创建或修改服务文件
2. 在 `src-tauri/src/main.rs` 中导入并注册命令
3. 前端通过 `invoke('command_name', args)` 调用

### 数据库操作

- 使用 `get_pool().await` 获取数据库连接池
- 使用 sqlx 的 query/query_as 执行 SQL
- 所有数据库操作都是异步的
- 数据库迁移在 `src-tauri/src/db.rs` 中管理

### 状态管理

- 使用 Pinia 管理状态
- `book.ts` 管理当前书籍、章节、内容、快照
- `config.ts` 管理应用配置（AI 配置、主题等）

### 样式规范

- 使用 Element Plus 组件
- 主题色通过 CSS 变量 `--el-color-primary` 控制
- 支持深色/浅色主题切换
- 侧边栏和 AI 面板支持拖拽调整宽度

### AI 功能架构

- **工具调用方案**: AI 通过工具主动查询信息
  - `list_all_chapters` - 获取所有章节列表
  - `query_chapter_summary` - 查询章节详细摘要
  - `query_chapter_content` - 查询章节完整内容
  - `get_outline` - 获取大纲
  - `save_outline` - 保存大纲
  - `list_character_cards` - 列出角色卡
  - `create_character_card` - 创建角色卡
  - `analyze_writing_style` - 分析写作风格
- **流式输出**: AI 响应采用 SSE 流式输出
- **提示词管理**: 所有提示词存储在 `src-tauri/prompts/` 目录

## 核心功能实现

### AI 润色功能
1. 用户选中文本，点击润色按钮
2. 发送选中文本 + 润色提示词到 AI
3. AI 返回润色后的文本
4. 显示对比界面，用户选择是否应用

### 大纲生成功能
1. 用户请求生成大纲
2. AI 查询相关章节内容
3. 生成粗纲和细纲
4. 显示生成结果，提供单独保存按钮
5. 保存后触发事件刷新大纲编辑器

### 写作风格学习
1. 后台服务定期分析用户作品
2. 选取代表性章节（开头、中间、结尾）
3. AI 分析语言风格、叙事方式等
4. 存储风格特征，后续创作时参考

## 常见问题

### 数据库连接失败
- 检查 `%APPDATA%/byakko-writer` 目录是否存在且有写入权限
- 数据库文件会在首次启动时自动创建

### 前端调用后端命令报错
- 检查命令名是否正确
- 检查参数类型是否匹配
- 查看 Rust 编译错误信息

### AI 流式输出中断
- 检查网络连接
- 查看 AI 配置是否正确
- 检查控制台错误日志

## 代码规范

### Rust
- 使用 anyhow 处理错误
- 异步函数使用 `async/await`
- 数据库操作使用 sqlx
- 服务层按功能模块化

### Vue
- 使用 Composition API + `<script setup>`
- 组件名使用 PascalCase
- Props 使用 TypeScript 类型定义
- 事件使用 camelCase

### 类型
- 前后端共享类型定义，保持同步
- 使用 TypeScript 严格模式
- Rust 模型使用 serde 序列化

## 待优化项

- [ ] 章节标题编辑有时不保存
- [ ] 性能优化（大数据量时卡顿）
- [ ] 导出功能（Word/PDF/Markdown）
- [ ] 拼写检查
- [ ] 写作数据统计
- [ ] 云同步

## 更新日志

### v0.0.5-beta
- 基础写作功能
- AI 对话与润色
- 大纲生成（粗纲/细纲）
- 角色卡管理
- 版本快照与对比
- 写作风格学习
- 深色/浅色主题

---
> Source: [xiaofei114/Byakko-Writer](https://github.com/xiaofei114/Byakko-Writer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
