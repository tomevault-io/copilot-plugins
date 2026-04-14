## pitchperfect

> - 使用 ES 模块 (`import`/`export`)

# PitchPerfect 项目规范

## 编码标准

### 1. 模块系统
- 使用 ES 模块 (`import`/`export`)
- 导入顺序：React → 第三方库 → 项目内部模块 → 类型定义

### 2. 函数声明
- 优先使用 `function` 关键字而非箭头函数（尤其适用于顶层函数和组件）
- 顶层函数应使用显式返回类型注解

### 3. 条件语句
- **禁止嵌套三元运算符** - 优先使用 `switch` 语句或 `if/else` 链
- 多个条件时使用 `switch` 提高可读性

```typescript
// ❌ 不推荐
const color = tone === "emerald" ? "stroke-emerald-500" : tone === "slate" ? "stroke-slate-400" : "stroke-blue-500";

// ✅ 推荐
function getToneColor(tone: "blue" | "emerald" | "slate"): string {
  switch (tone) {
    case "emerald":
      return "stroke-emerald-500";
    case "slate":
      return "stroke-slate-400";
    default:
      return "stroke-blue-500";
  }
}
```

### 4. 样式管理
- 使用 `cn()` 工具函数合并 Tailwind 类名
- 提取公共样式部分，避免重复代码

```typescript
// ❌ 不推荐 - 重复代码
const cardBase = "relative overflow-hidden rounded-3xl border border-white/70 bg-white/80...";
const cardSoft = "relative overflow-hidden rounded-2xl border border-white/60 bg-white/75...";

// ✅ 推荐 - 提取公共部分
const cardBaseStyles = "relative overflow-hidden border bg-white/80 ring-1...";
const cardBase = cn(cardBaseStyles, "rounded-3xl border-white/70...");
const cardSoft = cn(cardBaseStyles, "rounded-2xl border-white/60...");
```

### 5. React Hooks
- 提取重复的 `useEffect` 逻辑为自定义 Hook
- 使用 `useCallback` 缓存回调函数

```typescript
// ✅ 推荐 - 提取通用逻辑
function useTimeout(callback: () => void, delay: number, deps: React.DependencyList) {
  const callbackRef = useRef(callback);
  callbackRef.current = callback;

  useEffect(() => {
    const timer = setTimeout(() => callbackRef.current(), delay);
    return () => clearTimeout(timer);
  }, deps);
}
```

### 6. 错误处理
- 优先使用显式错误处理而非 `try/catch`（当可能时）
- 提供有意义的错误消息

### 7. 命名规范
- 组件：PascalCase
- 函数/变量：camelCase
- 常量：UPPER_SNAKE_CASE
- 类型/接口：PascalCase

### 8. 代码清晰度优先
- 选择显式代码而非紧凑代码
- 避免过度简化的单行长代码
- 保持合理的函数长度（<50 行）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Smartloe)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Smartloe)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
