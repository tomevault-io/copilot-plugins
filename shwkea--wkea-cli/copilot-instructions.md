## wkea-cli

> > AI 操作手册。WKEA 后台管理命令行工具 (TypeScript + Commander.js v12)。

# wkea-cli Agent Guide

> AI 操作手册。WKEA 后台管理命令行工具 (TypeScript + Commander.js v12)。

## 1. 项目概述

npm 包 `wkea-manage-cli`。TypeScript + Commander.js v12 + axios，esbuild 打包为 `dist/index.js`，target Node.js 20。后端三套接口：`/api/manage/passport/`（登录）、`/api/manageV2/<module>/`（业务CRUD）、`/api/ec/`（枚举+COS凭证）。配置存 `~/.wkea/config.json`。

## 2. 环境搭建

```bash
npm install
npm run build                           # esbuild 打包 + 复制 docs/
node dist/index.js init                 # 配置 API + 登录
node dist/index.js whoami               # 验证
```

## 3. 构建/测试

```bash
npm run build                           # esbuild → dist/index.js + zip
npx tsc --noEmit                        # 类型检查 (必须零错误)
node dist/index.js <cmd> --help         # 命令可用性验证
```

## 4. 代码约定

- **命令注册**: `src/commands/<module>/` 下 `index.ts` 聚合子命令，`crud.ts` 实现 CRUD
- **API 调用**: `new ApiClient(getApiUrl())` → API 函数 → `checkResponse()` → `success()/error()`
- **输出**: `formatJsonWithFields(data, FIELDS)` 输出 JSON + 字段说明表格
- **大结果**: `--save-json` 写 `/tmp/wkea-cli-json/` 供 AI 完整读取
- TypeScript strict, esbuild target node20

## 5. Git 工作流

```bash
npx tsc --noEmit && npm run build       # 提交前必做
```
Commit 纯中文。

## 6. 安全边界

- **绝对禁止**: 捏造产品价格 (必须来自供应商报价)；猜测数据；硬编码密码
- **需要确认**: 修改定价逻辑
- **配置文件**: `~/.wkea/config.json` 存明文密码，勿分享

## 命令速查

| 模块 | 子命令 |
|------|--------|
| `brand` | crud/list, bind-vendors/categories, urls |
| `vendor` | crud/list, brands/categories, contact/bank/invoice/address, merge |
| `product` | spu/sku crud, supply, spec/attribute bind, quick-create |
| `demand` | crud/list, parse, items, quote-to-vendor, save-price |
| `customer` | crud/list, address/invoice/bank/contact |
| `stock` | crud/list, switch-unit, warehouse |

> 📖 架构文档 → `docs/project-overview.md`

---
> Source: [shwkea/wkea-cli](https://github.com/shwkea/wkea-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
