## testing

> 测试开发规范


# 测试开发规范

## 测试框架

项目使用 Vitest + Testing Library 进行测试：

- 单元测试: 测试工具函数和 Hooks
- 组件测试: 测试 React 组件
- API 测试: 测试 API 路由
- 集成测试: 测试完整功能流程

## 测试文件组织

```
tests/
├── api/           # API 路由测试
├── components/    # 组件测试
├── hooks/         # Hook 测试
├── lib/           # 工具函数测试
├── integration/   # 集成测试
└── setup/         # 测试配置
```

## 测试命名

- 测试文件: `*.test.{ts,tsx}`
- 测试描述: 使用中文描述测试场景
- 测试用例: 使用 `describe` 和 `it` 组织

## 测试脚本

项目提供多种测试命令：

- `pnpm test` - 运行所有测试
- `pnpm test:watch` - 监听模式
- `pnpm test:coverage` - 生成覆盖率报告
- `pnpm test:api` - 仅运行 API 测试
- `pnpm test:ui` - 仅运行 UI 测试

## 测试数据

- 使用 `@testing-library/jest-dom` 提供 DOM 断言
- 使用 `jsdom` 环境模拟浏览器
- 测试数据使用工厂函数生成

## 最佳实践

- 每个测试用例只测试一个功能点
- 使用 `beforeEach` 和 `afterEach` 进行清理
- 模拟外部依赖（API、数据库等）
- 测试边界条件和错误情况

---
> Source: [wantao9191/wantapp](https://github.com/wantao9191/wantapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
