## jxkh

> Controller → Service → Domain

# 企业 AI 管家系统 - Cursor Rules

## 架构规范

### 分层架构

```
Controller → Service → Domain
                  ↓
           Integration → Infrastructure
```

### 允许调用

- ✅ Controller → Service
- ✅ Service → Domain
- ✅ Service → Integration
- ✅ Service → Infra (接口)

### 禁止调用

- ❌ Controller 直接访问 Repository
- ❌ Service 直接调用 HTTP
- ❌ Domain 依赖 infra
- ❌ 跨层反向调用

### 复杂度限制

- 单文件 ≤ 500 行
- 单函数 ≤ 50 行
- 嵌套 ≤ 3 层
- 单次修改文件 ≤ 5

---

## UI 设计规范

### 色彩系统

- **主色**: `#2196F3`
- **成功**: `#52c41a`
- **警告**: `#fa8c16`
- **错误**: `#f5222d`
- **信息**: `#1677ff`

### 中性色

- `gray[900]`: `#000000` (主要文字)
- `gray[600]`: `#606266` (次要文字)
- `gray[400]`: `#A0AEC0` (占位符)
- `gray[200]`: `#EBEEF5` (边框)
- `gray[50]`: `#F5F7FA` (页面背景)

### 排版

- 字体大小必须使用 `rem` 单位
- 禁止使用 px 单位定义字体
- 字重：normal/medium/semibold/bold

### 间距

- 基于 4px 基数的间距系统
- 禁止使用非 4px 倍数的间距

### 动画

- 点击反馈：150ms
- 悬停效果：200ms
- 页面跳转：200-300ms

### 禁止项

- ❌ 直接使用 hex 颜色值（除调试外）
- ❌ 使用非 4px 倍数的间距
- ❌ 使用 px 单位定义 rem 可替代的值
- ❌ 混用多种蓝色系
- ❌ 使用非标准字号（13px, 15px）
- ❌ 点击反馈小于 100ms
- ❌ 不使用 aria-label 标注图标
- ❌ 对比度低于 4.5:1

---

## 代码规范

### Java (Spring Boot)

```java
// 必须使用统一异常处理
// 必须使用 lombok
// 禁止裸 except
// 必须使用 logging
```

### TypeScript

```typescript
// 禁止 any
// 必须定义类型
// 必须使用 ESLint
```

### Python (FastAPI)

```python
// 必须类型标注
// 禁止裸 except
// 必须使用 logging
```

---

## 测试规范

- 新增代码 → 必须有测试
- 修改代码 → 必须更新测试
- 覆盖率 ≥ 70%

---

## Git 规范

### 分支策略

```
main          # 生产
develop       # 开发
feature/*     # 功能
release/*     # 发布
hotfix/*      # 热修复
```

### Commit 格式

```
type: subject

body

footer
```

类型：feat, fix, docs, style, refactor, test, chore

---

## K8s 规范

- 必须配置 resources limits/requests
- 必须有 liveness/readiness probe
- 禁止 latest tag

---

## 文档规范

- 每次修改必须评估风险等级
- 新增代码必须有测试覆盖
- 单次修改文件不超过 5 个

---
> Source: [multifort/jxkh](https://github.com/multifort/jxkh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
