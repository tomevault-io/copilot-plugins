## llm-router

> Documentation update requirements for breaking changes and architectural changes


# 文档同步更新规则

## Breaking Changes

当开发过程中引入 **Breaking Changes**（破坏性变更）时，必须立即更新以下文档：

| 变更类型 | 必须更新的文档 |
|---------|--------------|
| API 接口变更（路径、参数、响应格式） | `docs/API.md` |
| 环境变量变更 | `docs/API.md`、`README.md` |
| 配置文件格式变更 | `docs/DEVELOPMENT.md`、`README.md` |
| 数据库 Schema 变更 | `docs/ARCHITECTURE.md` |
| 移除或重命名公共方法/类 | `docs/API.md`、`docs/ARCHITECTURE.md` |

### Breaking Changes 检查清单

- [ ] 更新了 API 文档（请求/响应格式、错误码）
- [ ] 更新了配置说明（环境变量、配置文件）
- [ ] 更新了迁移指南（如有必要）
- [ ] 更新了 CHANGELOG（记录变更内容）

---

## 架构重大改动

当项目架构发生重大改动时，必须立即更新以下文档：

| 改动范围 | 必须更新的文档 |
|---------|--------------|
| 新增/删除模块或目录 | `docs/ARCHITECTURE.md` |
| Provider 架构变更 | `docs/ARCHITECTURE.md`、`.cursor/rules/project-architecture.mdc` |
| Pipeline 处理逻辑变更 | `docs/ARCHITECTURE.md`、`.cursor/rules/project-architecture.mdc` |
| 存储层变更 | `docs/ARCHITECTURE.md` |
| 核心类/接口变更 | `docs/ARCHITECTURE.md`、`.cursor/rules/project-architecture.mdc` |

### 架构改动检查清单

- [ ] 更新了架构文档（目录结构、模块关系）
- [ ] 更新了架构规则文件（`.cursor/rules/project-architecture.mdc`）
- [ ] 更新了开发指南（如流程发生变化）
- [ ] 更新了示例代码（如使用方式变化）

---

## 文档优先级

变更完成后，按以下优先级更新文档：

1. **P0 - 立即更新**
   - `docs/API.md` - API 变更
   - `docs/ARCHITECTURE.md` - 架构变更
   - `.cursor/rules/*.mdc` - 开发规则变更

2. **P1 - 同一天内更新**
   - `README.md` - 使用说明变更
   - `docs/DEVELOPMENT.md` - 开发流程变更

3. **P2 - 本周内更新**
   - 其他辅助文档
   - 代码注释中的文档链接

---

## 示例场景

### 场景 1：修改 API 响应格式

```python
# 修改前
{"code": 0, "data": {...}}

# 修改后（Breaking Change）
{"status": "success", "result": {...}}
```

**必须更新：**
- `docs/API.md` - 更新响应示例
- `docs/API.md` - 更新错误码说明
- `README.md` - 如有示例代码需同步更新

### 场景 2：新增 Provider

**必须更新：**
- `.cursor/rules/project-architecture.mdc` - 更新目录结构、添加 Provider 步骤
- `docs/ARCHITECTURE.md` - 更新架构图和说明
- `docs/DEVELOPMENT.md` - 更新配置说明

---

## 提醒

> **文档是代码的契约** - 保持文档与代码同步，确保其他开发者能够准确理解系统。
>
> 如不确定某个变更是否属于 Breaking Change 或架构改动，请**宁可更新，不要遗漏**。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/crimson-and-clover)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/crimson-and-clover)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
