## gp-auto-cd

> - 防止反复恢复代码又重新修改，浪费时间和 credits


# 开发工作流规则

## 1. 代码修改原则

**谨慎修改，避免浪费：**

- 代码量较多，修改时务必谨慎
- 避免语法错误或修改错代码块
- 防止反复恢复代码又重新修改，浪费时间和 credits
- 修改前确认理解上下文和依赖关系

## 2. Git Commit 消息

**任务完成后，自动生成英文 Git commit 消息：**

### 消息格式

```
v<version> <type>(<scope>): <subject>

<body>
```

### Type 类型

| Type | 说明 |
|------|------|
| `feat` | 新功能 |
| `fix` | Bug 修复 |
| `docs` | 文档更新 |
| `refactor` | 代码重构（无功能变化） |
| `chore` | 构建/配置变更 |

### 示例

```
v1.5.2 fix(iframe): select instrumentmanager iframe over modaliframe

- Changed _iframeReadyWindow single cache to _iframeReadyList array
- Added getBestReadyIframe() to prefer instrumentmanager URL
- Fixed issue where modaliframe (0 inputs) was selected
```

### 规则

1. **语言**: 必须使用英文
2. **Subject**: 首字母小写，不超过 50 字符，不以句号结尾（含版本号后总长不超过 72 字符）
3. **Body**: 使用列表说明具体修改内容
4. **Scope**: 使用修改的模块名（如 fill, save, ui, config）
5. **版本号**: 统一使用脚本头部 `// @version` 字段的值，确保 `SCRIPT_VERSION` 常量与 `@version` 保持一致

   **版本号更新规则（语义化版本）：**
   | 版本变化 | 触发条件 | 示例 |
   |---------|---------|------|
   | 主版本 (X.0.0) | 重大功能变更、破坏性修改 | 1.0.0 → 2.0.0 |
   | 次版本 (0.X.0) | 新增功能、新模块添加 | 1.0.0 → 1.1.0 |
   | 修订号 (0.0.X) | Bug 修复、小优化 | 1.0.0 → 1.0.1 |

### 文件变更清单格式

**在 Git commit 消息之后，必须提供以下格式的文件清单：**

```
Files changed:
- New: newfile.js
- Modified: auto-add-card.user.js
- Modified: README.md
```

## 3. 日志输出要求

**脚本必须将运行日志输出到本地文件，便于后续分析和优化：**

- 日志文件路径: `d:\code\addpaycard\logs\auto-card-log.txt`
- 每次运行追加写入，保留最近 20 次运行记录
- 日志内容包含: 时间戳、操作步骤、错误信息、页面元素检测结果
- 脚本启动时自动创建 logs 目录（如不存在）

## 4. 检查清单

每次任务完成前确认：

### 代码质量
- [ ] 代码修改谨慎，无语法错误
- [ ] 理解所有修改的代码上下文
- [ ] 手动测试脚本功能正常

### 日志输出
- [ ] 日志文件正确写入 `logs/auto-card-log.txt`
- [ ] 日志包含完整的元素检测信息

### 文档更新
- [ ] README.md 已同步（如功能变化）
- [ ] 脚本头部注释已更新（如版本号）
- [ ] CHANGELOG.md 已更新（如适用）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eudemonchan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
