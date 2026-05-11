## typescript-development

> 类型定义文件统一放在 `src/types/` 目录下，按功能模块组织：

# TypeScript 开发规范

## 类型定义规范

### 项目类型结构
类型定义文件统一放在 `src/types/` 目录下，按功能模块组织：

```
src/types/
├── index.ts           # 导出所有类型
├── components.ts      # 组件相关类型
├── api.ts            # API接口类型
├── store.ts          # 状态管理类型
├── router.ts         # 路由类型
├── utils.ts          # 工具函数类型
└── global.d.ts       # 全局类型声明
```

### 基础类型定义

#### 组件Props类型
```typescript
// src/types/components.ts
export interface BaseComponentProps {
  id: string
  className?: string
  style?: CSSProperties
  disabled?: boolean
  visible?: boolean
}

export interface ButtonProps extends BaseComponentProps {
  type?: 'primary' | 'success' | 'warning' | 'danger' | 'info'
  size?: 'large' | 'default' | 'small'
  loading?: boolean
  icon?: string
  onClick?: (event: MouseEvent) => void
}

export interface InputProps extends BaseComponentProps {
  modelValue?: string | number
  placeholder?: string
  maxlength?: number
  showWordLimit?: boolean
  clearable?: boolean
  'onUpdate:modelValue'?: (value: string | number) => void
}
```

#### API响应类型
```typescript
// src/types/api.ts
export interface ApiResponse<T = any> {
  code: number
  message: string
  data: T
  success: boolean
  timestamp: number
}

export interface PaginationParams {
  page: number
  pageSize: number
  total?: number
}

export interface PaginatedResponse<T> extends ApiResponse<T[]> {
  pagination: PaginationParams
}

// 项目相关API类型
export interface ProjectInfo {
  id: string
  name: string
  description?: string
  version: string
  createTime: string
  updateTime: string
  components: ComponentConfig[]
}

export interface ComponentConfig {
  id: string
  type: string
  props: Record<string, any>
  children?: ComponentConfig[]
  position: {
    x: number
    y: number
    width: number
    height: number
  }
}
```

#### Store状态类型
```typescript
// src/types/store.ts
export interface AppState {
  theme: 'light' | 'dark'
  language: 'zh-CN' | 'en-US'
  sidebarCollapsed: boolean
  loading: boolean
}

export interface ProjectState {
  currentProject: ProjectInfo | null
  projects: ProjectInfo[]
  selectedComponent: ComponentConfig | null
  draggedComponent: ComponentConfig | null
}

export interface UserState {
  userInfo: UserInfo | null
  isLoggedIn: boolean
  permissions: string[]
}

export interface UserInfo {
  id: string
  username: string
  email: string
  avatar?: string
  role: string
}
```

### Vue 3 + TypeScript 最佳实践

#### 组件Props定义
```typescript
// 推荐的Props定义方式
<script setup lang="ts">
interface Props {
  title: string
  content?: string
  type?: 'info' | 'warning' | 'error'
  closable?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  type: 'info',
  closable: true
})

// 计算属性类型推断
const computedClass = computed(() => ({
  [`alert-${props.type}`]: true,
  'is-closable': props.closable
}))
</script>
```

#### 响应式数据类型
```typescript
import { ref, reactive } from 'vue'
import type { ComponentConfig, ProjectInfo } from '@/types'

// ref 类型声明
const loading = ref<boolean>(false)
const projectList = ref<ProjectInfo[]>([])
const selectedComponent = ref<ComponentConfig | null>(null)

// reactive 类型声明
interface FormData {
  name: string
  email: string
  age: number
}

const formData = reactive<FormData>({
  name: '',
  email: '',
  age: 0
})
```

#### 事件处理器类型
```typescript
<script setup lang="ts">
interface Emits {
  change: [value: string]
  click: [event: MouseEvent]
  submit: [data: FormData]
}

const emit = defineEmits<Emits>()

// 事件处理函数
const handleClick = (event: MouseEvent) => {
  emit('click', event)
}

const handleSubmit = (data: FormData) => {
  emit('submit', data)
}
</script>
```

## 工具函数类型定义

### 通用工具类型
```typescript
// src/types/utils.ts

// 深度只读类型
export type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends Record<string, any>
    ? DeepReadonly<T[P]>
    : T[P]
}

// 可选属性类型
export type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>

// 必需属性类型
export type RequiredBy<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>>

// 联合转交集类型
export type UnionToIntersection<U> = 
  (U extends any ? (k: U) => void : never) extends ((k: infer I) => void) ? I : never

// 函数参数类型提取
export type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never

// 函数返回值类型提取
export type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any
```

### API工具类型
```typescript
// API请求配置类型
export interface RequestConfig {
  url: string
  method: 'GET' | 'POST' | 'PUT' | 'DELETE'
  params?: Record<string, any>
  data?: any
  headers?: Record<string, string>
  timeout?: number
}

// API服务类型
export type ApiService<T extends Record<string, any>> = {
  [K in keyof T]: T[K] extends (...args: any[]) => any
    ? (...args: Parameters<T[K]>) => Promise<ApiResponse<ReturnType<T[K]>>>
    : never
}
```

## 全局类型声明

### Vue全局属性扩展
```typescript
// src/types/global.d.ts
import 'vue'

declare module 'vue' {
  interface ComponentCustomProperties {
    $message: typeof ElMessage
    $confirm: typeof ElMessageBox.confirm
    $loading: (options?: any) => any
  }
}

// 全局变量类型
declare global {
  interface Window {
    __TAURI__: any
    electronAPI?: any
  }
  
  // 环境变量类型
  interface ImportMetaEnv {
    readonly VITE_APP_NAME: string
    readonly VITE_APP_VERSION: string
    readonly VITE_API_BASE_URL: string
    readonly VITE_AUTO_UPDATE_ENABLED: string
  }
}
```

### Element Plus 组件类型扩展
```typescript
// 扩展Element Plus组件类型
import type { ElForm, ElTable, ElDialog } from 'element-plus'

export type FormInstance = InstanceType<typeof ElForm>
export type TableInstance = InstanceType<typeof ElTable>
export type DialogInstance = InstanceType<typeof ElDialog>

// 表单验证规则类型
export interface FormRules {
  [key: string]: Array<{
    required?: boolean
    message?: string
    trigger?: string | string[]
    validator?: (rule: any, value: any, callback: any) => void
  }>
}
```

## 类型安全最佳实践

### 1. 严格类型检查
```typescript
// tsconfig.json 配置
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

### 2. 类型守卫使用
```typescript
// 类型守卫函数
export function isString(value: unknown): value is string {
  return typeof value === 'string'
}

export function isArray<T>(value: unknown): value is T[] {
  return Array.isArray(value)
}

export function isValidComponentConfig(config: any): config is ComponentConfig {
  return config && 
         typeof config.id === 'string' && 
         typeof config.type === 'string' &&
         config.props !== undefined
}

// 使用类型守卫
function processData(data: unknown) {
  if (isValidComponentConfig(data)) {
    // 这里data被正确推断为ComponentConfig类型
    console.log(data.id, data.type)
  }
}
```

### 3. 泛型约束
```typescript
// 泛型约束示例
interface Identifiable {
  id: string
}

function updateItem<T extends Identifiable>(items: T[], id: string, updates: Partial<T>): T[] {
  return items.map(item => 
    item.id === id ? { ...item, ...updates } : item
  )
}

// 使用示例
const updatedComponents = updateItem(components, 'comp-1', { 
  props: { visible: false } 
})
```

### 4. 条件类型使用
```typescript
// 条件类型示例
type ApiMethod<T> = T extends 'get' 
  ? (url: string, params?: any) => Promise<any>
  : T extends 'post'
  ? (url: string, data?: any) => Promise<any>
  : never

// 映射类型
type ComponentEvents<T> = {
  [K in keyof T as `on${Capitalize<string & K>}`]: T[K] extends Function 
    ? T[K] 
    : never
}
```

## 错误处理和类型安全

### 异步操作类型安全
```typescript
import type { ApiResponse } from '@/types/api'

// 封装API调用的类型安全函数
async function safeApiCall<T>(
  apiCall: () => Promise<ApiResponse<T>>
): Promise<[T | null, string | null]> {
  try {
    const response = await apiCall()
    if (response.success) {
      return [response.data, null]
    } else {
      return [null, response.message]
    }
  } catch (error) {
    return [null, error instanceof Error ? error.message : '未知错误']
  }
}

// 使用示例
const [projectData, error] = await safeApiCall(() => fetchProject(id))
if (error) {
  console.error('获取项目失败:', error)
} else if (projectData) {
  // projectData 已经是正确的类型
  console.log('项目名称:', projectData.name)
}
```

---
> Source: [suyancc/easy_window](https://github.com/suyancc/easy_window) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
