## second-brain

> - Vue 3 + Composition API + TypeScript

# 第二大脑 — 前端代码规范

## 技术栈
- Vue 3 + Composition API + TypeScript
- Vite 7 + PWA
- Tailwind CSS v4（使用 `@import "tailwindcss"` 而非 `@tailwind` 指令）
- Pinia 状态管理

## 代码组织
```
src/
├── api/           # API 请求封装
├── assets/        # 静态资源
├── components/    # 通用组件（大驼峰，如 NoteCard.vue）
├── composables/   # 组合式函数（useXxx.ts）
├── router/        # 路由配置
├── stores/        # Pinia store
├── views/         # 页面组件（大驼峰，如 Home.vue）
└── App.vue
```

## 命名规范
| 类型 | 规范 | 示例 |
|------|------|------|
| 组件文件 | PascalCase.vue | `NoteCard.vue` |
| 组合式函数 | useCamelCase.ts | `useNotes.ts` |
| Store | useXxxStore.ts | `useAppStore.ts` |
| API 文件 | camelCase.ts | `notes.ts` |
| 变量/函数 | camelCase | `fetchNotes` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_TIMES` |
| CSS 变量 | kebab-case | `--btn-primary-color` |

## TypeScript 规范
- **禁止使用 `any`**（除非明确标记 `// eslint-disable-next-line @typescript-eslint/no-explicit-any`）
- 组件 props 必须定义类型
- API 返回值必须定义接口
- 使用 `interface` 而非 `type` 来定义对象结构（更推荐）

```typescript
// ✅ 正确
interface NoteItem {
  id: string
  content: string
  category: 'work' | 'personal'
}

// ❌ 错误
const note: any = {}
```

## Vue 3 规范
- 使用 `<script setup lang="ts">` 语法
- 组件 emit 必须声明 emits 选项
- 避免在 `setup()` 中直接解构 `props`（会丢失响应式），用 `toRefs()` 或直接访问

```typescript
// ✅ 正确
const props = defineProps<{ id: string }>()
console.log(props.id)

// ❌ 错误（会丢失响应式）
const { id } = defineProps<{ id: string }>()
```

## 样式规范
- 使用 Tailwind CSS 原子类
- 自定义样式写在 `<style scoped>` 中
- 避免全局样式污染

## Git Commit 规范
```
feat: 新功能
fix: 修复 bug
docs: 文档更新
style: 代码格式（不影响功能）
refactor: 重构
test: 测试相关
chore: 构建/工具变更
```

示例：
```bash
git commit -m "feat: 新增笔记详情页"
git commit -m "fix: 修复 OCR 预览页标签显示错误"
```

## 组件开发模板

标准 Vue 3 组件结构（`<script setup lang="ts">`）：

```vue
<template>
  <div class="component-name">
    <!-- 模板内容 -->
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useRoute } from 'vue-router'

// ========== Props & Emits ==========
interface Props {
  id: string
  title?: string
}

const props = withDefaults(defineProps<Props>(), {
  title: '默认标题',
})

const emit = defineEmits<{
  (e: 'update', value: string): void
  (e: 'delete', id: string): void
}>()

// ========== Composables ==========
const { loading, data, error, execute } = useAsync(() => fetchData(props.id))

// ========== Reactive State ==========
const localState = ref<string>('')
const computedValue = computed(() => localState.value.toUpperCase())

// ========== Methods ==========
function handleClick() {
  emit('update', localState.value)
}

// ========== Lifecycle ==========
onMounted(() => {
  execute()
})
</script>

<style scoped>
.component-name {
  /* 样式 */
}
</style>
```

### 组件内逻辑抽取原则
- 超过 50 行的逻辑（非模板/样式）应抽取为 composable（`useXxx.ts`）
- 可复用逻辑（多组件共享）必须抽取为 composable
- 网络请求逻辑统一放 `api/` 目录，组件只调用 composable

## API 调用规范

### 目录结构
```
src/api/
├── notes.ts      # 笔记相关 API
├── tags.ts       # 标签相关 API
└── types.ts      # 共享类型定义
```

### 调用模式
```typescript
// ✅ 正确：组件通过 composable 调用 API
// composables/useNotes.ts
export function useNotes() {
  const notes = ref<Note[]>([])
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function fetchNotes() {
    loading.value = true
    error.value = null
    try {
      notes.value = await getNotes()
    } catch (err) {
      error.value = err instanceof Error ? err.message : '加载失败'
      // 可以在这里统一做用户提示
    } finally {
      loading.value = false
    }
  }

  return { notes, loading, error, fetchNotes }
}

// ❌ 错误：组件直接写 fetch + try-catch
const notes = ref([])
const response = await fetch('/api/notes')
notes.value = await response.json()
```

### 错误处理三件套
1. **try-catch 包裹**：所有 async 函数必须有 try-catch
2. **错误状态记录**：error ref 记录错误信息
3. **用户提示**：在 UI 上展示错误（toast / inline message），不能静默失败

## 状态管理规范

### 优先级：ref/reactive → Pinia Store

```typescript
// 1. 组件内部状态 → ref/reactive
const loading = ref(false)
const formData = reactive({ title: '', content: '' })

// 2. 跨组件共享状态 → Pinia Store
// stores/useNotesStore.ts
export const useNotesStore = defineStore('notes', () => {
  const notes = ref<Note[]>([])
  const activeCategory = ref<'work' | 'personal'>('work')

  async function fetchNotes() { /* ... */ }

  return { notes, activeCategory, fetchNotes }
})
```

### 何时用 Pinia
- 多个页面/组件需要读写同一份数据（如笔记列表、用户设置）
- 需要跨组件持久化状态
- 状态逻辑复杂，需要 getter/action 封装

### 何时不用 Pinia
- 仅在单个组件内使用的状态（如表单输入、弹窗开关）
- 父子组件传递用 props/emits 即可

## 错误处理规范

### API 错误统一处理模式
```typescript
// ✅ 正确模式
async function handleSave() {
  loading.value = true
  try {
    await createNote(formData)
    showToast('保存成功')
    router.push('/')
  } catch (err) {
    // 1. 记录错误（生产环境可接入监控）
    console.error('保存笔记失败:', err)
    // 2. 展示用户可理解的错误信息
    const message = err instanceof Error ? err.message : '保存失败，请重试'
    showToast(message, 'error')
  } finally {
    loading.value = false
  }
}

// ❌ 错误：吞掉错误
async function handleSave() {
  try {
    await createNote(formData)
  } catch {
    // 什么都不做，用户不知道出错了
  }
}

// ❌ 错误：用空 catch
async function handleSave() {
  try {
    await createNote(formData)
  } catch (err) {
    // eslint-disable-next-line no-empty
  }
}
```

### 错误分级
| 级别 | 处理方式 | 示例 |
|------|----------|------|
| 网络错误 | 提示「网络异常，请检查连接」 | fetch 超时、DNS 失败 |
| 业务错误 | 展示后端返回的 error 字段 | 创建笔记时标题重复 |
| 未知错误 | 提示通用错误 + 建议重试 | 未知异常 |

## 提交前必查清单
- [ ] `npm run type-check` 通过
- [ ] `npm run lint` 无 error（warn 可接受）
- [ ] `npm run build` 构建成功
- [ ] 功能本地验证通过
- [ ] 无 console.log / debugger

## AI 辅助开发提示

用 Cursor / AI 写代码时，可以在对话开头加这段：

> "你是一个 Vue 3 + TypeScript 开发者，参考项目根目录的 CLAUDE.md 规范来写代码。"

这样 AI 生成代码会自动遵循以上规范。

---
> Source: [yxc-gdut/second-brain-](https://github.com/yxc-gdut/second-brain-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
