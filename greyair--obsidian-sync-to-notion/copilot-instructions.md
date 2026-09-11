## obsidian-sync-to-notion

> 本文件用于 AI/自动化代理快速理解本项目并安全执行任务。

# AGENTS.md

本文件用于 AI/自动化代理快速理解本项目并安全执行任务。

## 1) 项目简介

- 项目：Obsidian 插件 `notion-encrypted-sync`
- 目标：加密笔记并与 Notion 双向同步（含占位符补水、冲突处理、附件同步）
- 语言：TypeScript
- 构建：esbuild

## 2) 常用命令

```bash
npm install
npm run typecheck
npm run build
npm run release:check
```

## 3) 目录结构（核心）

- `src/main.ts`：插件入口、命令注册、调度器、服务装配
- `src/settings.ts`：配置类型与默认值
- `src/crypto/crypto-service.ts`：加解密实现（可选算法，兼容旧密文）
- `src/sync/sync-engine.ts`：双向同步主流程（下行占位符、上行更新/新建、冲突）
- `src/sync/hydration-service.ts`：占位符打开自动补水
- `src/providers/notion-provider.ts`：Notion API 适配层
- `src/providers/s3-provider.ts`：S3 加密附件后端
- `src/ui/settings-tab.ts`：插件设置 UI
- `src/ui/conflict-view.ts`：冲突三栏合并 UI

## 4) 关键产品约束

1. `syncEncryptionMode=encrypted` 时，Notion 正文同步密文。
2. 支持算法配置：
   - `xchacha20poly1305-argon2id`（推荐）
   - `aes256gcm-pbkdf2`（兼容）
3. 会话锁：一次解锁可编辑，空闲自动锁，锁后需再次输入密码。
4. 占位符策略：下行先占位，打开时自动补水。
5. 冲突策略：保留双份，不覆盖丢失。

## 5) 代理修改规则

- 优先小步增量修改，避免大面积重构。
- 修改后必须执行：
  - `npm run typecheck`
  - `npm run build`
- 不输出敏感信息到日志（token/密钥/明文）。
- 保持 Notion 字段兼容：`Title`、`Tags`、`Status`、`UpdatedAt`。
- 附件后端行为：
  - `s3`：加密后上传
  - `notion`：以加密 blob 方式写入页面块

## 6) 发布相关

- 插件元数据：`manifest.json`
- 版本映射：`versions.json`
- 变更记录：`CHANGELOG.md`
- CI：`.github/workflows/ci.yml`
- Release 打包：`.github/workflows/release.yml`

## 7) 常见任务建议

- 若改加密逻辑：确认旧密文可解密（向后兼容）
- 若改同步逻辑：至少验证创建/更新/冲突三类路径
- 若改设置项：同步更新 `DEFAULT_SETTINGS` 与设置页展示

---
> Source: [greyair/Obsidian-Sync-to-Notion](https://github.com/greyair/Obsidian-Sync-to-Notion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
