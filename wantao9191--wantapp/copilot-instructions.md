## performance

> 性能优化规范


# 性能优化规范

## Next.js 优化

### 构建优化

- 使用 `next build` 进行生产构建
- 启用 `removeConsole` 移除生产环境 console
- 使用 `standalone` 输出模式

### 开发优化

- 使用 `--turbo` 模式加速开发
- 配置内存模式避免磁盘 I/O
- 优化 Webpack 配置

## React 优化

### 组件优化

- 使用 `React.memo` 避免不必要的重渲染
- 使用 `useMemo` 和 `useCallback` 优化计算
- 合理使用 `useState` 和 `useEffect`

### 代码分割

- 使用动态导入进行代码分割
- 按路由分割页面组件
- 懒加载非关键组件

## 数据库优化

### 查询优化

- 使用索引优化查询性能
- 避免 N+1 查询问题
- 使用连接池管理数据库连接

### 缓存策略

- 实现适当的缓存策略
- 使用 Redis 进行数据缓存
- 合理设置缓存过期时间

## 资源优化

### 图片优化

- 使用 Next.js Image 组件
- 支持 WebP 和 AVIF 格式
- 实现响应式图片

### 字体优化

- 使用 Google Fonts 优化
- 预加载关键字体
- 使用 `font-display: swap`

## 监控和分析

### 性能监控

- 使用 Web Vitals 监控性能
- 实现错误边界捕获错误
- 记录性能指标

### 代码分析

- 使用 ESLint 进行代码检查
- 使用 TypeScript 进行类型检查
- 定期进行代码审查

---
> Source: [wantao9191/wantapp](https://github.com/wantao9191/wantapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
