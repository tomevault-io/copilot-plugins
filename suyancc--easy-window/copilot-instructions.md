## coding-standards

> // ✅ 组件文件: PascalCase

# 代码质量标准和开发规范

## 代码风格规范

### 命名规范
```typescript
// 文件命名
// ✅ 组件文件: PascalCase
ComponentName.vue
PropertyEditor.vue

// ✅ 工具函数: camelCase  
utilityFunction.ts
apiService.ts

// ✅ 配置文件: kebab-case
api-config.ts
build-config.ts

// ✅ 常量: SCREAMING_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com'
const MAX_RETRY_COUNT = 3

// ✅ 变量和函数: camelCase
const userName = 'admin'
const isLoading = false
function handleClick() {}

// ✅ 类型和接口: PascalCase
interface UserInfo {}
type ComponentProps = {}
```

### 导入导出规范
```typescript
// ✅ 推荐的导入顺序
// 1. Node.js 内置模块
import path from 'path'

// 2. 第三方库
import { ref, computed, watch } from 'vue'
import { defineStore } from 'pinia'
import { ElMessage } from 'element-plus'

// 3. 内部模块 - 按路径深度排序
import type { ApiResponse } from '@/types'
import { apiClient } from '@/services/base'
import { useAppStore } from '@/stores/app'
import ComponentEditor from '@/components/editors/ComponentEditor.vue'

// ✅ 导出规范
// 优先使用命名导出
export const utilFunction = () => {}
export { ApiService } from './api'

// 默认导出仅用于Vue组件和主要模块
export default defineComponent({
  name: 'ComponentName'
})
```

## 代码注释规范

### JSDoc 注释标准
```typescript
/**
 * 用户信息接口
 * @interface UserInfo
 */
interface UserInfo {
  /** 用户ID */
  id: string
  /** 用户名 */
  username: string
  /** 用户邮箱 */
  email: string
}

/**
 * 获取用户信息
 * @param {string} userId - 用户ID
 * @param {Object} options - 可选参数
 * @param {boolean} options.includeProfile - 是否包含详细信息
 * @returns {Promise<ApiResponse<UserInfo>>} 返回用户信息
 * @throws {Error} 当用户不存在时抛出错误
 * @example
 * ```typescript
 * const user = await getUserInfo('123', { includeProfile: true })
 * console.log(user.data.username)
 * ```
 */
async function getUserInfo(
  userId: string, 
  options: { includeProfile?: boolean } = {}
): Promise<ApiResponse<UserInfo>> {
  // 实现逻辑
}
```

### Vue组件注释
```vue
<template>
  <!-- 主容器 - 负责布局和样式控制 -->
  <div class="component-container">
    <!-- 标题区域 -->
    <header class="header">
      <h1>{{ title }}</h1>
    </header>
    
    <!-- 内容区域 - 支持插槽自定义 -->
    <main class="content">
      <slot name="content">
        <!-- 默认内容 -->
      </slot>
    </main>
  </div>
</template>

<script setup lang="ts">
/**
 * 通用容器组件
 * 提供标准的页面布局结构，支持标题和内容自定义
 * 
 * @component Container
 * @example
 * <Container title="页面标题">
 *   <template #content>
 *     自定义内容
 *   </template>
 * </Container>
 */

interface Props {
  /** 页面标题 */
  title: string
  /** 是否显示边框 */
  bordered?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  bordered: true
})

// 组件逻辑...
</script>
```

## 错误处理规范

### 统一错误处理
```typescript
// ✅ 错误类型定义
enum ErrorCode {
  NETWORK_ERROR = 'NETWORK_ERROR',
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  PERMISSION_DENIED = 'PERMISSION_DENIED',
  RESOURCE_NOT_FOUND = 'RESOURCE_NOT_FOUND'
}

interface AppError {
  code: ErrorCode
  message: string
  details?: any
  timestamp: number
}

// ✅ 错误处理工具
class ErrorHandler {
  /**
   * 创建标准化错误对象
   */
  static createError(code: ErrorCode, message: string, details?: any): AppError {
    return {
      code,
      message,
      details,
      timestamp: Date.now()
    }
  }

  /**
   * 处理API错误
   */
  static handleApiError(error: any): AppError {
    if (error.response) {
      const { status, data } = error.response
      switch (status) {
        case 404:
          return this.createError(
            ErrorCode.RESOURCE_NOT_FOUND,
            '请求的资源不存在',
            { originalError: error }
          )
        case 403:
          return this.createError(
            ErrorCode.PERMISSION_DENIED,
            '没有权限访问该资源',
            { originalError: error }
          )
        default:
          return this.createError(
            ErrorCode.NETWORK_ERROR,
            data?.message || '网络请求失败',
            { originalError: error }
          )
      }
    }
    
    return this.createError(
      ErrorCode.NETWORK_ERROR,
      '网络连接失败',
      { originalError: error }
    )
  }

  /**
   * 显示用户友好的错误消息
   */
  static showUserError(error: AppError) {
    ElMessage.error(error.message)
    
    // 开发环境下输出详细错误信息
    if (import.meta.env.DEV) {
      console.error('错误详情:', error)
    }
  }
}

// ✅ 使用示例
async function loadUserData(userId: string) {
  try {
    const response = await userApi.getUser(userId)
    return response.data
  } catch (error) {
    const appError = ErrorHandler.handleApiError(error)
    ErrorHandler.showUserError(appError)
    throw appError
  }
}
```

### 组件级错误边界
```vue
<template>
  <div class="error-boundary">
    <template v-if="!hasError">
      <slot />
    </template>
    <template v-else>
      <div class="error-display">
        <el-alert
          title="组件加载失败"
          :description="errorMessage"
          type="error"
          show-icon
        >
          <template #default>
            <el-button @click="retry">重试</el-button>
          </template>
        </el-alert>
      </div>
    </template>
  </div>
</template>

<script setup lang="ts">
/**
 * 错误边界组件
 * 捕获子组件的错误并提供恢复机制
 */
import { ref, onErrorCaptured } from 'vue'

const hasError = ref(false)
const errorMessage = ref('')

// 捕获子组件错误
onErrorCaptured((error: Error) => {
  hasError.value = true
  errorMessage.value = error.message
  
  // 记录错误到日志系统
  console.error('组件错误:', error)
  
  // 阻止错误向上传播
  return false
})

const retry = () => {
  hasError.value = false
  errorMessage.value = ''
}
</script>
```

## 性能优化规范

### 响应式数据优化
```typescript
// ✅ 合理使用 ref 和 reactive
import { ref, reactive, computed, readonly } from 'vue'

// 基础类型使用 ref
const count = ref(0)
const loading = ref(false)

// 复杂对象使用 reactive
const userForm = reactive({
  name: '',
  email: '',
  age: 0
})

// 只读数据使用 readonly
const config = readonly({
  apiUrl: 'https://api.example.com',
  timeout: 5000
})

// ✅ 计算属性优化
const expensiveComputed = computed(() => {
  // 仅在依赖变化时重新计算
  return heavyCalculation(someReactiveData.value)
})

// ✅ 大列表优化
const virtualList = computed(() => {
  // 虚拟滚动 - 只渲染可见项
  const startIndex = Math.floor(scrollTop.value / itemHeight)
  const endIndex = Math.min(startIndex + visibleCount, items.value.length)
  return items.value.slice(startIndex, endIndex)
})
```

### 组件性能优化
```vue
<template>
  <!-- ✅ 使用 v-memo 优化大列表 -->
  <div
    v-for="item in largeList"
    :key="item.id"
    v-memo="[item.name, item.status]"
    class="list-item"
  >
    {{ item.name }} - {{ item.status }}
  </div>

  <!-- ✅ 条件渲染优化 -->
  <template v-if="showExpensiveComponent">
    <ExpensiveComponent :data="componentData" />
  </template>

  <!-- ✅ 异步组件 -->
  <Suspense>
    <template #default>
      <AsyncComponent />
    </template>
    <template #fallback>
      <div class="loading">加载中...</div>
    </template>
  </Suspense>
</template>

<script setup lang="ts">
import { defineAsyncComponent } from 'vue'

// ✅ 异步加载组件
const AsyncComponent = defineAsyncComponent(() =>
  import('@/components/HeavyComponent.vue')
)

// ✅ 防抖优化
import { debounce } from 'lodash-es'

const handleSearch = debounce((query: string) => {
  // 搜索逻辑
}, 300)

// ✅ 内存泄漏防护
import { onUnmounted } from 'vue'

let timer: NodeJS.Timeout | null = null

onUnmounted(() => {
  if (timer) {
    clearInterval(timer)
    timer = null
  }
})
</script>
```

## 测试规范

### 单元测试标准
```typescript
// tests/utils/format.spec.ts
import { describe, it, expect } from 'vitest'
import { formatDate, formatFileSize } from '@/utils/format'

describe('格式化工具函数', () => {
  describe('formatDate', () => {
    it('应该正确格式化日期', () => {
      const date = new Date('2024-01-01T10:30:00')
      const result = formatDate(date, 'YYYY-MM-DD')
      expect(result).toBe('2024-01-01')
    })

    it('应该处理无效日期', () => {
      const result = formatDate(null, 'YYYY-MM-DD')
      expect(result).toBe('')
    })
  })

  describe('formatFileSize', () => {
    it('应该正确格式化文件大小', () => {
      expect(formatFileSize(1024)).toBe('1.00 KB')
      expect(formatFileSize(1048576)).toBe('1.00 MB')
      expect(formatFileSize(0)).toBe('0 B')
    })
  })
})
```

### 组件测试示例
```typescript
// tests/components/Button.spec.ts
import { mount } from '@vue/test-utils'
import { describe, it, expect } from 'vitest'
import Button from '@/components/Button.vue'

describe('Button 组件', () => {
  it('应该渲染正确的文本', () => {
    const wrapper = mount(Button, {
      props: {
        type: 'primary'
      },
      slots: {
        default: '点击按钮'
      }
    })

    expect(wrapper.text()).toBe('点击按钮')
    expect(wrapper.classes()).toContain('el-button--primary')
  })

  it('应该触发点击事件', async () => {
    const wrapper = mount(Button)
    
    await wrapper.trigger('click')
    
    expect(wrapper.emitted()).toHaveProperty('click')
    expect(wrapper.emitted('click')).toHaveLength(1)
  })

  it('禁用状态下不应触发事件', async () => {
    const wrapper = mount(Button, {
      props: {
        disabled: true
      }
    })

    await wrapper.trigger('click')

    expect(wrapper.emitted('click')).toBeFalsy()
  })
})
```

## 代码审查清单

### 提交前检查项
```bash
# ✅ 代码质量检查
npm run lint          # ESLint 检查
npm run type-check     # TypeScript 类型检查
npm run test           # 运行测试
npm run build          # 构建检查

# ✅ 手动检查项目
# 1. 是否有 console.log 等调试代码
# 2. 是否有硬编码的配置值
# 3. 是否有未使用的导入和变量
# 4. 是否有TODO注释需要处理
# 5. 是否添加了必要的错误处理
# 6. 是否更新了相关文档
```

### 代码审查标准
```typescript
// ❌ 不推荐的写法
function badFunction(data: any) {
  console.log(data) // 调试代码未清理
  
  // 硬编码配置
  const apiUrl = 'http://localhost:3000/api'
  
  // 缺少错误处理
  const result = data.items.map(item => item.name)
  
  return result
}

// ✅ 推荐的写法
/**
 * 处理数据列表，提取名称字段
 * @param data 包含items数组的数据对象
 * @returns 名称数组
 * @throws 当数据格式不正确时抛出错误
 */
function goodFunction(data: { items: Array<{ name: string }> }): string[] {
  // 输入验证
  if (!data || !Array.isArray(data.items)) {
    throw new Error('数据格式不正确：期望包含items数组的对象')
  }

  try {
    return data.items.map(item => {
      if (typeof item.name !== 'string') {
        throw new Error('项目名称必须是字符串类型')
      }
      return item.name
    })
  } catch (error) {
    console.error('处理数据时发生错误:', error)
    throw error
  }
}
```

## 环境配置规范

### 开发环境配置
```typescript
// .env.development
VITE_APP_NAME=Easy Window
VITE_APP_VERSION=1.0.0
VITE_API_BASE_URL=http://localhost:3000/api
VITE_LOG_LEVEL=debug

// .env.production  
VITE_APP_NAME=Easy Window
VITE_APP_VERSION=1.0.0
VITE_API_BASE_URL=https://api.easywindow.com
VITE_LOG_LEVEL=error
```

### 配置管理
```typescript
// src/config/env.ts
interface EnvConfig {
  appName: string
  appVersion: string
  apiBaseUrl: string
  logLevel: 'debug' | 'info' | 'warn' | 'error'
  isDevelopment: boolean
  isProduction: boolean
}

/**
 * 环境配置管理器
 */
export const envConfig: EnvConfig = {
  appName: import.meta.env.VITE_APP_NAME || 'Easy Window',
  appVersion: import.meta.env.VITE_APP_VERSION || '1.0.0',
  apiBaseUrl: import.meta.env.VITE_API_BASE_URL || 'http://localhost:3000/api',
  logLevel: (import.meta.env.VITE_LOG_LEVEL as EnvConfig['logLevel']) || 'info',
  isDevelopment: import.meta.env.DEV,
  isProduction: import.meta.env.PROD
}

// 配置验证
if (!envConfig.apiBaseUrl) {
  throw new Error('API_BASE_URL 环境变量未设置')
}
```

---
> Source: [suyancc/easy_window](https://github.com/suyancc/easy_window) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
