## preparade

> pnpm dev  # 启动开发服务器在 localhost:5173

# 开发工作流规则

## 开发服务器
```bash
pnpm dev  # 启动开发服务器在 localhost:5173
```

## 构建和预览
```bash
pnpm build    # TypeScript 编译 + Vite 构建
pnpm preview  # 预览生产构建
```

## Git 提交规范
每次功能完成后必须：
1. 谨慎使用 `git add` (不准使用 `git add .`)
2. 提供清晰的 commit message
3. 确保代码质量和功能完整性

### Commit Message 格式
```
feat: 添加新演示组件 InteractiveQueryDemo
fix: 修复 CSV 数据解析错误
style: 优化按钮样式和响应式布局
docs: 更新项目文档
refactor: 重构状态管理逻辑
```

## 添加新演示组件流程
1. 在 `src/components/` 创建新组件文件
2. 在 [src/reportData.ts](mdc:src/reportData.ts) 的 `ReportContentItem` 接口中添加新的 `demoComponent` 值
3. 在 [src/App.vue](mdc:src/App.vue) 中添加相应的 `v-if` 条件渲染
4. 在 `reportData` 数组中添加演示内容配置
5. 测试功能完整性
6. 提交代码

## CSV 数据处理流程
当处理新的 CSV 数据源时：
1. 将 CSV 文件放在 `src/components/` 目录
2. 使用 `import csvData from './filename.csv?raw'` 导入
3. 实现解析函数处理 CSV 格式差异
4. 创建相应的 TypeScript 接口定义数据结构
5. 在组件中集成数据解析逻辑

## 代码质量检查
- 确保所有 TypeScript 类型定义正确
- 验证 TailwindCSS 类名使用正确
- 检查组件响应式状态管理
- 确认演示功能的交互流畅性

## 特殊注意事项
- 项目完全中文化，包括变量名使用中文描述性注释
- 所有 API 调用都是模拟的，使用 `setTimeout` 模拟延迟
- 数据不需要持久化，刷新页面重置状态
- 专注于 UI 交互演示，不需要真实的后端集成
description:
globs:
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/betasecond) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
