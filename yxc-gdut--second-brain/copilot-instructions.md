## general

> 适用于所有代码文件的通用编码规范


# 通用编码规范

## 技术栈背景
- 前端：Vue 3 + Composition API + TypeScript + Vite + Tailwind CSS v4
- 后端：Koa.js + Node.js + TypeScript
- 存储：Markdown 文件（work.md / personal.md）

## 强制规范

### TypeScript
- **禁止使用 `any`**，用 `unknown` 替代或定义具体类型
- 所有函数参数必须有类型注解
- 接口用 `interface` 而非 `type`（更推荐）
- 禁止 `var`，只允许 `const` 和 `let`

```typescript
// ✅ 正确
interface NoteItem {
  id: string
  content: string
  category: 'work' | 'personal'
}

// ❌ 错误
const note: any = {}
function foo(a) { return a }
```

### Git Commit 规范
```
feat: 新功能
fix: 修复 bug
docs: 文档更新
style: 代码格式（不影响功能）
refactor: 重构
test: 测试相关
chore: 构建/工具变更
```

### 代码风格
- 使用 ESLint + Prettier 自动格式化，不要手动调整
- 使用 `async/await` 而非 `.then().catch()`
- 优先使用 `const`，只有变量会变时用 `let`
- 禁止 `console.log`（调试完成后删除）

### 安全红线
- **禁止硬编码 token/secret/password**：所有敏感信息必须通过环境变量（`import.meta.env.VITE_*`）获取
- **禁止 `eval()`、`new Function()`**：安全审计零容忍
- **`v-html` / `dangerouslySetInnerHTML` 审查**：使用前必须确认内容来源可信，或经过 DOMPurify 等库消毒
- **禁止内联事件处理器**：如 `onclick="..."`，必须用 Vue 事件绑定
- 用户输入必须校验/过滤，防止 XSS/注入
- API key、JWT 等不得出现在前端代码或 commit 中

### 性能规范
- **列表渲染必须用 `:key`**：优先使用唯一 ID，避免用 index
- **大组件懒加载**：超过 300 行的组件或路由级组件必须 `defineAsyncComponent` 或路由懒加载
- **避免不必要 watcher**：能用 computed 解决的不用 watch，watch 必须指定 `{ immediate: false }` 除非确实需要
- **避免深层响应式**：大列表数据用 `shallowRef` 代替 `ref`
- **图片懒加载**：使用 `loading="lazy"` 属性
- **避免模板中的复杂计算**：移入 computed 或方法中

### 禁止模式清单
- ❌ `any` 类型（用 `unknown` 或具体类型替代）
- ❌ `var` 声明（用 `const`/`let`）
- ❌ `console.log`（调试完成后必须删除，用 `console.error` 记录错误）
- ❌ `debugger` 语句
- ❌ 直接操作 DOM（`document.querySelector` / `innerHTML` 等），用 Vue ref
- ❌ `// @ts-ignore`（用 `@ts-expect-error` 并说明原因）
- ❌ 魔法数字（未命名的硬编码数字，如 `if (len > 120)`，应提取为常量）
- ❌ 未处理的 Promise（async 函数必须有 try-catch）

---
> Source: [yxc-gdut/second-brain-](https://github.com/yxc-gdut/second-brain-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
