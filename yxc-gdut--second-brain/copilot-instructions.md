## frontend

> Vue 3 前端项目特定规范


# 前端编码规范（Vue 3 + TypeScript）

## Vue 3 组件规范

### 文件结构
```
src/
├── api/           # API 请求封装（camelCase，如 notes.ts）
├── components/    # 通用组件（PascalCase.vue，如 NoteCard.vue）
├── composables/  # 组合式函数（useXxx.ts，如 useNotes.ts）
├── stores/       # Pinia store（useXxxStore.ts）
├── views/        # 页面组件（PascalCase.vue，如 Home.vue）
└── router/       # 路由配置
```

### script setup 语法
```typescript
// ✅ 正确
<script setup lang="ts">
const props = defineProps<{ id: string }>()
const emit = defineEmits<{ (e: 'update', value: string): void }>()
</script>

// ❌ 错误：直接解构 props（丢失响应式）
<script setup lang="ts">
const { id } = defineProps<{ id: string }>() // ❌ 不要这样做
</script>
```

### Props 定义
```typescript
// ✅ 正确：必选 props 用 defineProps
defineProps<{ noteId: string; title: string }>()

// 可选 props 提供默认值
const props = withDefaults(defineProps<{ maxHeight?: number }>(), {
  maxHeight: 300
})
```

### emits 声明
```typescript
// ✅ 正确：显式声明所有 emit
const emit = defineEmits<{
  (e: 'save', note: NoteItem): void
  (e: 'delete', id: string): void
}>()

// ❌ 错误：没有声明 emits
const emit = defineEmits(['save', 'delete'])
```

## TypeScript 规范

### API 返回类型定义
```typescript
// ✅ 正确：定义完整接口
interface NoteItem {
  id: string
  content: string
  tags: string[]
  category: 'work' | 'personal'
  source?: string
  createdAt: string
}

interface ApiResponse<T> {
  success: boolean
  data: T
  error?: string
}

// ❌ 错误：直接用 any
const response: any = await fetch('/api/notes')
```

### 禁止 any
```typescript
// ✅ 正确：不知道类型时用 unknown
function parseResponse(data: unknown): NoteItem {
  if (typeof data === 'object' && data !== null) {
    return data as NoteItem
  }
  throw new Error('Invalid data')
}

// ❌ 错误
const data: any = response
```

## 样式规范

### Tailwind CSS v4
- 使用 `@import "tailwindcss"` 而非 `@tailwind` 指令
- 优先使用 Tailwind 原子类，自定义样式写在 `<style scoped>` 中
- 避免 magic number，使用 CSS 变量

### CSS 变量

**必须使用 KDesign Token CSS 变量**，禁止魔法数字：

```css
/* ✅ 正确：使用 KDesign Token */
.card {
  border-radius: var(--kd-radius-lg, 8px);
  color: var(--kd-color-text-primary, #0D0D0D);
  font-size: var(--kd-font-size-base, 14px);
  padding: 16px;
  transition: box-shadow var(--kd-time-fast, 120ms) var(--kd-easing-ease);
}

/* ❌ 错误：魔法数字 */
.card {
  border-radius: 8px;
  color: #0D0D0D;
  font-size: 14px;
}
```

**常用 Token 前缀**：
- `--kd-color-*`：颜色（primary, secondary, tertiary, success, danger, warning, info, fill, background, line, text, icon）
- `--kd-font-*`：字体（size, weight, family, line-height）
- `--kd-radius-*`：圆角（sm, md, lg）
- `--kd-shadow-*`：阴影（sm, md, lg）
- `--kd-time-*`：动画时长（fast, normal, slow）
- `--kd-easing-*`：缓动函数
- `--kd-opacity-*`：透明度
- `--kd-space-*`：间距

## API 请求规范
```typescript
// ✅ 正确：统一封装在 api/ 目录
// api/notes.ts
export async function fetchNotes(params: {
  category?: 'work' | 'personal'
  page?: number
}): Promise<ApiResponse<NoteItem[]>> {
  const res = await fetch(`/api/notes?${new URLSearchParams(params as any)}`)
  return res.json()
}
```

## 组件拆分原则

- **单文件组件 < 300 行**（含 template + script + style）
- **超过 300 行必须拆分**：
  - 业务逻辑 → `composables/useXxx.ts`
  - 复杂模板 → 拆子组件
  - 长样式 → CSS 模块或抽象公共样式
- 组件只做一件事（Single Responsibility）
- props 超过 5 个时，考虑用对象 props 或拆分组件

## 路由规范

- **新页面必须在 `router/index.ts` 注册**
- 路由命名用 kebab-case：`/note/:id` → `name: 'note-detail'`
- 非首屏页面使用懒加载：

```typescript
// ✅ 正确：路由懒加载
{
  path: '/settings',
  name: 'settings',
  component: () => import('@/views/Settings.vue')
}

// ❌ 错误：首屏路由懒加载（增加首屏加载时间）
{
  path: '/',
  component: () => import('@/views/Home.vue')
}
```

## 响应式数据规范

```typescript
// ✅ 正确：用 toRef/toRefs 解构 props
const props = defineProps<{ id: string; title: string }>()
const { id, title } = toRefs(props)

// ✅ 正确：直接访问 props
const content = computed(() => props.title.toUpperCase())

// ❌ 错误：直接解构（丢失响应式）
const { id, title } = props  // 响应式丢失！

// ❌ 错误：defineProps 直接解构
const { id } = defineProps<{ id: string }>()  // 响应式丢失！
```

### Composable 中使用 toRef
```typescript
// composables/useNote.ts
export function useNote(noteId: Ref<string>) {
  const note = ref<Note | null>(null)

  watch(noteId, async (id) => {
    if (id) {
      note.value = await getNoteById(id)
    }
  }, { immediate: true })

  return { note }
}

// 组件中使用
const props = defineProps<{ id: string }>()
const { note } = useNote(toRef(props, 'id'))
```

## 提交前检查清单
- [ ] `npm run type-check` 通过
- [ ] `npm run lint` 无 error
- [ ] `npm run build` 构建成功
- [ ] 无 `any` 类型
- [ ] 无 `console.log`
- [ ] Props 有类型定义
- [ ] emits 有显式声明
- [ ] 组件文件 < 300 行（过长的拆分）

## E2E 测试规范

### 触发条件
当修改了以下内容时，必须补充/更新对应的 E2E 测试用例：
- `frontend/src/views/` 下的页面组件
- `frontend/src/components/` 下的通用组件（影响多个页面时）
- `frontend/src/api/` 下的 API 接口（影响数据流时）
- `frontend/src/router/index.ts` 路由变更

### E2E 测试目录
```
e2e/tests/           # 测试文件放在这里
  ├── home.spec.ts
  ├── chat.spec.ts
  └── ...
e2e/tests/helpers.ts # 共享工具函数
```

### 用例编写规范
- 使用 Playwright Test 框架
- 文件命名：`{feature}.spec.ts`
- 必须收集 console errors（使用 helpers.ts 中的 `collectConsoleErrors`）
- 断言方式：DOM 断言 + 路由断言 + 零 console error
- 不做截图对比（设计在迭代中）
- 每个测试用例要有清晰的 describe + test 描述

### 用例模板
```typescript
import { test, expect } from '@playwright/test'
import { collectConsoleErrors, expectNoConsoleErrors, waitForPageReady } from '../helpers'

test.describe('功能名称', () => {
  test('操作描述', async ({ page }) => {
    const errors = collectConsoleErrors(page)
    
    // 1. 前置条件（导航到页面）
    await page.goto('/path')
    await waitForPageReady(page)
    
    // 2. 执行操作
    await page.locator('selector').click()
    
    // 3. 断言结果
    await expect(page.locator('element')).toBeVisible()
    await expect(page).toHaveURL(/expected/)
    
    // 4. 零 console error
    expectNoConsoleErrors(errors)
  })
})
```

### 供开发者确认的用例描述格式
开发者在 Cursor Chat 中让 AI 生成 E2E 用例后，应确认以下内容：
1. 用例是否覆盖了本次需求改动的主要路径
2. 用例是否覆盖了异常场景（空状态、网络失败、边界值）
3. 用例是否与产品需求描述一致

---
> Source: [yxc-gdut/second-brain-](https://github.com/yxc-gdut/second-brain-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
