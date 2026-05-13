## antd-usage

> Ant Design 组件使用规范


# Ant Design 使用规范

## 主题配置

项目使用自定义 Ant Design 主题，配置在 [src/lib/antd-config.ts](mdc:src/lib/antd-config.ts)：

- 主色调: `#1890ff`
- 圆角: `6px`
- 禁用线框模式

## 组件使用

- 所有 Ant Design 组件必须通过 `@ant-design/nextjs-registry` 包装
- 使用 `ConfigProvider` 应用主题
- 优先使用 Ant Design 的 Form 组件进行表单处理
- 使用 `App` 组件包装以支持全局配置

## 布局组件

项目包含自定义布局组件：

- `BasicLayout` - 基础布局容器
- `BasicHeader` - 顶部导航
- `BasicAside` - 侧边栏
- `BasicTabs` - 标签页管理

## 表单处理

- 使用 `Form.Item` 包装表单字段
- 使用 `FormItemRenderer` 进行动态表单渲染
- 表单验证使用 Zod 进行类型安全验证

## 图标使用

- 使用 `@ant-design/icons` 提供的图标
- 自定义图标放在 `AppIcons` 组件中
- 支持 UnoCSS 图标预设

---
> Source: [wantao9191/wantapp](https://github.com/wantao9191/wantapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
