## lint-config

> 此项目是一个 monorepo，包含多个与代码规范相关的配置包，主要分为 ESLint 配置和 Stylelint 配置两大类。

# @flypeng/lint-config 项目概述

此项目是一个 monorepo，包含多个与代码规范相关的配置包，主要分为 ESLint 配置和 Stylelint 配置两大类。

## 项目结构

- [package.json](mdc:package.json) - 主项目配置
- [packages/](mdc:packages) - 所有配置包的目录
  - `eslint-config` - 主要 ESLint 配置包，整合了所有其他 ESLint 配置
  - `eslint-config-basic` - 基础 ESLint 配置
  - `eslint-config-javascript` - JavaScript 专用 ESLint 配置
  - `eslint-config-typescript` - TypeScript 专用 ESLint 配置
  - `eslint-config-vue` - Vue 专用 ESLint 配置
  - `eslint-config-react` - React 专用 ESLint 配置
  - `eslint-config-other` - 其他文件类型的 ESLint 配置
  - `stylelint-config` - Stylelint 配置包
- [eslint.config.js](mdc:eslint.config.js) - 项目的 ESLint 配置
- [.stylelintrc](mdc:.stylelintrc) - 项目的 Stylelint 配置
- [.prettierrc](mdc:.prettierrc) - 项目的 Prettier 配置

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/flingyp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
