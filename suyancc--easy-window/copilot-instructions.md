## state-management

> ├── index.ts          # 导出所有stores

# 状态管理与API服务规范

## Pinia 状态管理架构

### Store 文件组织
```
src/stores/
├── index.ts          # 导出所有stores
├── app.ts           # 应用全局状态
├── project.ts       # 项目相关状态
├── component.ts     # 组件管理状态
├── user.ts          # 用户状态
├── cache.ts         # 缓存管理
└── settings.ts      # 用户设置
```

### Store 定义规范

#### 基础Store结构
```typescript
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { ProjectInfo, ComponentConfig } from '@/types'

export const useProjectStore = defineStore('project', () => {
  // State - 使用 ref 定义响应式状态
  const currentProject = ref<ProjectInfo | null>(null)
  const projects = ref<ProjectInfo[]>([])
  const selectedComponent = ref<ComponentConfig | null>(null)
  const draggedComponent = ref<ComponentConfig | null>(null)
  const loading = ref(false)

  // Getters - 使用 computed 定义计算属性
  const hasCurrentProject = computed(() => currentProject.value !== null)
  const projectCount = computed(() => projects.value.length)
  const selectedComponentId = computed(() => selectedComponent.value?.id)

  // Actions - 定义修改状态的方法
  const setCurrentProject = (project: ProjectInfo | null) => {
    currentProject.value = project
  }

  const addProject = (project: ProjectInfo) => {
    projects.value.push(project)
  }

  const updateProject = (id: string, updates: Partial<ProjectInfo>) => {
    const index = projects.value.findIndex(p => p.id === id)
    if (index !== -1) {
      projects.value[index] = { ...projects.value[index], ...updates }
    }
  }

  const removeProject = (id: string) => {
    const index = projects.value.findIndex(p => p.id === id)
    if (index !== -1) {
      projects.value.splice(index, 1)
    }
  }

  const selectComponent = (component: ComponentConfig | null) => {
    selectedComponent.value = component
  }

  // 异步操作
  const loadProjects = async () => {
    loading.value = true
    try {
      const response = await projectApi.getProjects()
      projects.value = response.data
    } catch (error) {
      console.error('加载项目失败:', error)
      throw error
    } finally {
      loading.value = false
    }
  }

  const saveProject = async (project: ProjectInfo) => {
    loading.value = true
    try {
      if (project.id) {
        await projectApi.updateProject(project.id, project)
        updateProject(project.id, project)
      } else {
        const response = await projectApi.createProject(project)
        addProject(response.data)
      }
    } catch (error) {
      console.error('保存项目失败:', error)
      throw error
    } finally {
      loading.value = false
    }
  }

  // 清理方法
  const reset = () => {
    currentProject.value = null
    projects.value = []
    selectedComponent.value = null
    draggedComponent.value = null
    loading.value = false
  }

  return {
    // State
    currentProject,
    projects,
    selectedComponent,
    draggedComponent,
    loading,
    // Getters
    hasCurrentProject,
    projectCount,
    selectedComponentId,
    // Actions
    setCurrentProject,
    addProject,
    updateProject,
    removeProject,
    selectComponent,
    loadProjects,
    saveProject,
    reset
  }
})
```

#### 应用全局状态
```typescript
// src/stores/app.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'

export const useAppStore = defineStore('app', () => {
  // 主题设置
  const theme = ref<'light' | 'dark'>('light')
  const language = ref<'zh-CN' | 'en-US'>('zh-CN')
  
  // 界面状态
  const sidebarCollapsed = ref(false)
  const loading = ref(false)
  const fullscreen = ref(false)
  
  // 通知和消息
  const notifications = ref<Array<{
    id: string
    type: 'info' | 'success' | 'warning' | 'error'
    title: string
    message: string
    timestamp: number
  }>>([])

  // Actions
  const toggleTheme = () => {
    theme.value = theme.value === 'light' ? 'dark' : 'light'
    // 保存到本地存储
    localStorage.setItem('theme', theme.value)
  }

  const setLanguage = (lang: 'zh-CN' | 'en-US') => {
    language.value = lang
    localStorage.setItem('language', lang)
  }

  const toggleSidebar = () => {
    sidebarCollapsed.value = !sidebarCollapsed.value
  }

  const setLoading = (state: boolean) => {
    loading.value = state
  }

  const addNotification = (notification: Omit<typeof notifications.value[0], 'id' | 'timestamp'>) => {
    const newNotification = {
      ...notification,
      id: Date.now().toString(),
      timestamp: Date.now()
    }
    notifications.value.unshift(newNotification)
    
    // 自动清理旧通知（保留最新50条）
    if (notifications.value.length > 50) {
      notifications.value = notifications.value.slice(0, 50)
    }
  }

  const removeNotification = (id: string) => {
    const index = notifications.value.findIndex(n => n.id === id)
    if (index !== -1) {
      notifications.value.splice(index, 1)
    }
  }

  // 初始化方法
  const initializeApp = () => {
    // 从本地存储恢复设置
    const savedTheme = localStorage.getItem('theme') as 'light' | 'dark'
    if (savedTheme) {
      theme.value = savedTheme
    }
    
    const savedLanguage = localStorage.getItem('language') as 'zh-CN' | 'en-US'
    if (savedLanguage) {
      language.value = savedLanguage
    }
  }

  return {
    theme,
    language,
    sidebarCollapsed,
    loading,
    fullscreen,
    notifications,
    toggleTheme,
    setLanguage,
    toggleSidebar,
    setLoading,
    addNotification,
    removeNotification,
    initializeApp
  }
}, {
  persist: {
    key: 'app-settings',
    storage: localStorage,
    paths: ['theme', 'language', 'sidebarCollapsed']
  }
})
```

## API 服务架构

### API 服务组织
```
src/services/
├── index.ts         # 导出所有API服务
├── base.ts          # 基础HTTP客户端
├── project.ts       # 项目相关API
├── component.ts     # 组件相关API
├── user.ts          # 用户相关API
├── upload.ts        # 文件上传API
└── cache.ts         # 缓存服务
```

### 基础HTTP客户端
```typescript
// src/services/base.ts
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios'
import type { ApiResponse } from '@/types/api'
import { useAppStore } from '@/stores/app'
import { ElMessage } from 'element-plus'

class HttpClient {
  private instance: AxiosInstance

  constructor(baseURL: string) {
    this.instance = axios.create({
      baseURL,
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json'
      }
    })

    this.setupInterceptors()
  }

  private setupInterceptors() {
    // 请求拦截器
    this.instance.interceptors.request.use(
      (config) => {
        const appStore = useAppStore()
        appStore.setLoading(true)
        
        // 添加认证token
        const token = localStorage.getItem('token')
        if (token) {
          config.headers.Authorization = `Bearer ${token}`
        }
        
        return config
      },
      (error) => {
        const appStore = useAppStore()
        appStore.setLoading(false)
        return Promise.reject(error)
      }
    )

    // 响应拦截器
    this.instance.interceptors.response.use(
      (response: AxiosResponse<ApiResponse>) => {
        const appStore = useAppStore()
        appStore.setLoading(false)
        
        // 检查业务状态码
        if (!response.data.success) {
          ElMessage.error(response.data.message || '请求失败')
          return Promise.reject(new Error(response.data.message))
        }
        
        return response
      },
      (error) => {
        const appStore = useAppStore()
        appStore.setLoading(false)
        
        // 处理HTTP错误
        if (error.response) {
          const { status, data } = error.response
          switch (status) {
            case 401:
              ElMessage.error('登录已过期，请重新登录')
              // 跳转到登录页
              break
            case 403:
              ElMessage.error('没有权限访问该资源')
              break
            case 404:
              ElMessage.error('请求的资源不存在')
              break
            case 500:
              ElMessage.error('服务器内部错误')
              break
            default:
              ElMessage.error(data?.message || '网络错误')
          }
        } else {
          ElMessage.error('网络连接失败')
        }
        
        return Promise.reject(error)
      }
    )
  }

  async get<T>(url: string, params?: any): Promise<ApiResponse<T>> {
    const response = await this.instance.get<ApiResponse<T>>(url, { params })
    return response.data
  }

  async post<T>(url: string, data?: any): Promise<ApiResponse<T>> {
    const response = await this.instance.post<ApiResponse<T>>(url, data)
    return response.data
  }

  async put<T>(url: string, data?: any): Promise<ApiResponse<T>> {
    const response = await this.instance.put<ApiResponse<T>>(url, data)
    return response.data
  }

  async delete<T>(url: string): Promise<ApiResponse<T>> {
    const response = await this.instance.delete<ApiResponse<T>>(url)
    return response.data
  }

  async upload<T>(url: string, file: File, onProgress?: (progress: number) => void): Promise<ApiResponse<T>> {
    const formData = new FormData()
    formData.append('file', file)
    
    const response = await this.instance.post<ApiResponse<T>>(url, formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      },
      onUploadProgress: (progressEvent) => {
        if (onProgress && progressEvent.total) {
          const progress = Math.round((progressEvent.loaded * 100) / progressEvent.total)
          onProgress(progress)
        }
      }
    })
    
    return response.data
  }
}

// 创建API客户端实例
export const apiClient = new HttpClient(import.meta.env.VITE_API_BASE_URL || 'http://localhost:3000/api')
```

### 项目API服务
```typescript
// src/services/project.ts
import { apiClient } from './base'
import type { ProjectInfo, ComponentConfig, ApiResponse, PaginatedResponse } from '@/types'

export class ProjectService {
  // 获取项目列表
  async getProjects(params?: {
    page?: number
    pageSize?: number
    keyword?: string
  }): Promise<PaginatedResponse<ProjectInfo>> {
    return apiClient.get<ProjectInfo[]>('/projects', params)
  }

  // 获取单个项目
  async getProject(id: string): Promise<ApiResponse<ProjectInfo>> {
    return apiClient.get<ProjectInfo>(`/projects/${id}`)
  }

  // 创建项目
  async createProject(project: Omit<ProjectInfo, 'id' | 'createTime' | 'updateTime'>): Promise<ApiResponse<ProjectInfo>> {
    return apiClient.post<ProjectInfo>('/projects', project)
  }

  // 更新项目
  async updateProject(id: string, project: Partial<ProjectInfo>): Promise<ApiResponse<ProjectInfo>> {
    return apiClient.put<ProjectInfo>(`/projects/${id}`, project)
  }

  // 删除项目
  async deleteProject(id: string): Promise<ApiResponse<void>> {
    return apiClient.delete<void>(`/projects/${id}`)
  }

  // 复制项目
  async cloneProject(id: string, name: string): Promise<ApiResponse<ProjectInfo>> {
    return apiClient.post<ProjectInfo>(`/projects/${id}/clone`, { name })
  }

  // 导出项目
  async exportProject(id: string, format: 'vue' | 'html' | 'json'): Promise<ApiResponse<Blob>> {
    return apiClient.get<Blob>(`/projects/${id}/export`, { format })
  }

  // 导入项目
  async importProject(file: File): Promise<ApiResponse<ProjectInfo>> {
    return apiClient.upload<ProjectInfo>('/projects/import', file)
  }
}

export const projectApi = new ProjectService()
```

## 可组合函数开发

### 组件桥接系统
```typescript
// src/composables/useComponentBridge.ts
import { ref, computed, watch } from 'vue'
import type { ComponentConfig } from '@/types'

interface ComponentBridgeOptions {
  // 组件特定配置
  defaultProps?: Record<string, any>
  // 事件映射
  eventMap?: Record<string, string>
  // 属性验证
  propValidators?: Record<string, (value: any) => boolean>
}

export function useComponentBridge(
  componentType: string, 
  options: ComponentBridgeOptions = {}
) {
  const { defaultProps = {}, eventMap = {}, propValidators = {} } = options

  // 组件配置
  const config = ref<ComponentConfig>({
    id: '',
    type: componentType,
    props: { ...defaultProps },
    children: [],
    position: { x: 0, y: 0, width: 100, height: 50 }
  })

  // 计算属性
  const computedProps = computed(() => ({
    ...config.value.props,
    'data-component-id': config.value.id,
    'data-component-type': config.value.type
  }))

  // 事件处理
  const createEventHandler = (eventName: string) => {
    return (event: Event) => {
      const mappedEvent = eventMap[eventName] || eventName
      
      // 触发全局事件
      window.dispatchEvent(new CustomEvent(`component:${mappedEvent}`, {
        detail: {
          componentId: config.value.id,
          componentType: config.value.type,
          event,
          data: event
        }
      }))
    }
  }

  // 属性更新
  const updateProps = (newProps: Record<string, any>) => {
    // 验证属性
    for (const [key, value] of Object.entries(newProps)) {
      if (propValidators[key] && !propValidators[key](value)) {
        console.warn(`属性 ${key} 验证失败:`, value)
        continue
      }
    }
    
    config.value.props = { ...config.value.props, ...newProps }
  }

  // 位置更新
  const updatePosition = (position: Partial<ComponentConfig['position']>) => {
    config.value.position = { ...config.value.position, ...position }
  }

  // 初始化组件
  const initialize = (initialConfig: Partial<ComponentConfig>) => {
    config.value = {
      ...config.value,
      ...initialConfig,
      props: { ...defaultProps, ...initialConfig.props }
    }
  }

  // 导出API到全局（供外部语言调用）
  const exportToGlobal = () => {
    if (!window.easyWindowAPI) {
      window.easyWindowAPI = {}
    }
    
    window.easyWindowAPI[config.value.id] = {
      updateProps,
      updatePosition,
      getConfig: () => config.value,
      trigger: (eventName: string, data?: any) => {
        createEventHandler(eventName)(new CustomEvent(eventName, { detail: data }))
      }
    }
  }

  // 监听配置变化
  watch(config, (newConfig) => {
    exportToGlobal()
  }, { deep: true })

  return {
    config,
    computedProps,
    updateProps,
    updatePosition,
    initialize,
    createEventHandler,
    exportToGlobal
  }
}
```

### 缓存管理
```typescript
// src/composables/useCache.ts
import { ref, computed } from 'vue'

interface CacheItem<T> {
  data: T
  timestamp: number
  expiry?: number
}

export function useCache<T>(key: string, ttl: number = 5 * 60 * 1000) {
  const cache = ref<Map<string, CacheItem<T>>>(new Map())

  const set = (itemKey: string, data: T, customTtl?: number) => {
    const expiry = customTtl ? Date.now() + customTtl : Date.now() + ttl
    cache.value.set(itemKey, {
      data,
      timestamp: Date.now(),
      expiry
    })
  }

  const get = (itemKey: string): T | null => {
    const item = cache.value.get(itemKey)
    if (!item) return null

    if (item.expiry && Date.now() > item.expiry) {
      cache.value.delete(itemKey)
      return null
    }

    return item.data
  }

  const has = (itemKey: string): boolean => {
    return get(itemKey) !== null
  }

  const remove = (itemKey: string) => {
    cache.value.delete(itemKey)
  }

  const clear = () => {
    cache.value.clear()
  }

  const size = computed(() => cache.value.size)

  // 清理过期缓存
  const cleanExpired = () => {
    const now = Date.now()
    for (const [key, item] of cache.value.entries()) {
      if (item.expiry && now > item.expiry) {
        cache.value.delete(key)
      }
    }
  }

  // 定期清理
  setInterval(cleanExpired, 60000) // 每分钟清理一次

  return {
    set,
    get,
    has,
    remove,
    clear,
    size,
    cleanExpired
  }
}
```

## 状态持久化

### Pinia持久化配置
```typescript
// src/stores/index.ts
import { createPinia } from 'pinia'
import { createPersistedState } from 'pinia-plugin-persistedstate'

const pinia = createPinia()

// 配置持久化插件
pinia.use(createPersistedState({
  key: (id) => `easy-window-${id}`,
  storage: localStorage,
  serializer: {
    serialize: JSON.stringify,
    deserialize: JSON.parse,
  },
}))

export { pinia }

// 导出所有stores
export { useAppStore } from './app'
export { useProjectStore } from './project'
export { useComponentStore } from './component'
export { useUserStore } from './user'
```

### 选择性持久化
```typescript
// 在store中配置持久化选项
export const useProjectStore = defineStore('project', () => {
  // ... store逻辑
}, {
  persist: {
    key: 'project-store',
    storage: localStorage,
    paths: ['currentProject', 'recentProjects'], // 只持久化指定字段
    beforeRestore: (ctx) => {
      console.log('恢复store状态:', ctx.store.$id)
    },
    afterRestore: (ctx) => {
      console.log('store状态恢复完成:', ctx.store.$id)
    }
  }
})
```

---
> Source: [suyancc/easy_window](https://github.com/suyancc/easy_window) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
