## init-vue

> 初始化 Vue3 项目（Router + Pinia + ESLint + Prettier + Element Plus）。支持 JS/TS，可选包管理器。--ts 启用 TypeScript，--no-axios 跳过 Axios 封装。


# 初始化 Vue3 项目

## 前置条件

- **Node.js** >= 18
- **包管理器**：已安装 pnpm（默认）/ npm / yarn 之一

## 参数说明

| 参数 | 说明 |
|------|------|
| `<项目名>` | 必填，项目目录名 |
| `--ts` | 可选，启用 TypeScript |
| `--no-axios` | 可选，跳过 Axios 相关安装与配置 |
| `--pnpm` | 可选，使用 pnpm（默认） |
| `--npm` | 可选，使用 npm |
| `--yarn` | 可选，使用 yarn |

示例：`init-vue my-app --ts --no-axios --npm`

## 执行步骤

### 1. 脚手架

解析 `$ARGUMENTS`：
- 提取第一个不含 `--` 的部分作为项目名
- 检测 `--ts` → `MODE=ts`（默认 `js`）
- 检测 `--no-axios` → `NO_AXIOS=true`
- 检测 `--pnpm` / `--npm` / `--yarn` → `PM=pnpm|npm|yarn`（默认 `pnpm`）

```bash
# 使用 npx 统一创建（跨包管理器兼容）
npx create-vue@latest <项目名> [--ts] --router --pinia --eslint --prettier --bare
```

### 2. 依赖安装 + 项目配置

```bash
bash ~/.claude/skills/init-vue/scripts/init.sh <项目名> <MODE> <PM> [--no-axios]
```

脚本完成以下操作：
- 进入项目目录
- 使用指定包管理器安装 Element Plus 等依赖
- 覆盖 `vite.config.<ext>`（含 Element Plus 自动导入 + SCSS）
- 覆盖 `.prettierrc.json`
- 覆盖 `eslint.config.<ext>`（通用 Vue 配置，不含 oxlint）
- 创建 `src/` 目录结构
- 生成 `src/stores/index.<ext>`（Pinia + 持久化），将 counter 移到 modules/
- 生成 `src/utils/request.<ext>`（除非 `--no-axios`）
- 生成 `src/styles/var.scss`
- 更新入口文件 `main.<ext>`

### 3. 报告

根据 MODE 列出完成项。提醒后续手动配置：
- `router/index.<ext>` 添加路由
- `stores/modules/` 添加状态模块
- 若未跳过 axios，提醒配置 `utils/request.<ext>` 中的 `baseURL`

---
> Source: [ipcer-firegod/init-vue](https://github.com/ipcer-firegod/init-vue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
