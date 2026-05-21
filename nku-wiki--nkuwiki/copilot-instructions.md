## miniprogram-development

> ├── components/        # 自定义组件

# 微信小程序开发规范

## 项目结构规范

### 标准目录结构
```
services/app/
├── components/        # 自定义组件
│   ├── common/       # 通用组件
│   ├── business/     # 业务组件
│   └── ui/          # UI组件
├── pages/            # 页面文件
├── utils/           # 工具函数
├── api/             # API接口封装
├── styles/          # 公共样式
├── behaviors/       # 组件行为
├── app.js           # 小程序入口文件
├── app.json         # 全局配置
└── app.wxss         # 全局样式
```

### 文件命名规范
```
# ✅ 推荐的文件命名
search-bar/           # 组件目录：小写连字符
user-profile.js       # JS文件：小写连字符
api-client.js         # 工具文件：小写连字符
post-detail.wxml      # 页面文件：小写连字符

# ❌ 避免的命名
SearchBar/            # 避免大驼峰目录名
userProfile.js        # 避免小驼峰文件名
api_client.js         # 避免下划线（除特殊情况）
```

## JavaScript 编码规范

### 1. 基础语法规范
```javascript
// ✅ 推荐的代码风格
const userInfo = {
  openid: '',
  nickname: '',
  avatar: ''
}

// 使用const/let，避免var
const API_BASE_URL = 'https://api.nkuwiki.com'
let searchResults = []

// 函数命名：动词开头，小驼峰
function getUserInfo() {
  return wx.getStorageSync('userInfo')
}

// 异步函数优先使用async/await
async function searchKnowledge(query) {
  try {
    const result = await apiClient.post('/knowledge/search', { query })
    return result.data
  } catch (error) {
    console.error('搜索失败:', error)
    throw error
  }
}
```

### 2. 页面/组件生命周期
```javascript
// 页面生命周期标准模板
Page({
  data: {
    // 页面数据
    query: '',
    searchResults: [],
    loading: false,
    hasMore: true
  },
  
  // 页面加载
  onLoad(options) {
    console.log('页面加载:', options)
    this.initPage(options)
  },
  
  // 页面显示
  onShow() {
    this.refreshUserInfo()
  },
  
  // 页面卸载
  onUnload() {
    this.cleanup()
  },
  
  // 自定义方法
  async initPage(options) {
    const { query } = options
    if (query) {
      this.setData({ query })
      await this.performSearch(query)
    }
  },
  
  async performSearch(query) {
    if (!query.trim()) {
      wx.showToast({ title: '请输入搜索内容', icon: 'none' })
      return
    }
    
    this.setData({ loading: true })
    
    try {
      const results = await searchKnowledge(query)
      this.setData({ 
        searchResults: results,
        loading: false 
      })
    } catch (error) {
      this.setData({ loading: false })
      this.showError('搜索失败，请重试')
    }
  },
  
  // 错误处理
  showError(message) {
    wx.showToast({
      title: message,
      icon: 'none',
      duration: 2000
    })
  },
  
  // 清理资源
  cleanup() {
    // 清理定时器、取消请求等
  }
})
```

### 3. 组件定义规范
```javascript
// 自定义组件标准模板
Component({
  // 组件属性
  properties: {
    placeholder: {
      type: String,
      value: '请输入搜索内容'
    },
    disabled: {
      type: Boolean,
      value: false
    }
  },
  
  // 组件数据
  data: {
    inputValue: '',
    focused: false
  },
  
  // 组件生命周期
  lifetimes: {
    attached() {
      // 组件实例进入页面节点树时执行
      console.log('SearchBar组件已挂载')
    },
    
    detached() {
      // 组件实例被从页面节点树移除时执行
      this.cleanup()
    }
  },
  
  // 页面生命周期
  pageLifetimes: {
    show() {
      // 页面显示时执行
    },
    hide() {
      // 页面隐藏时执行
    }
  },
  
  // 组件方法
  methods: {
    onInput(e) {
      const value = e.detail.value
      this.setData({ inputValue: value })
      
      // 触发自定义事件
      this.triggerEvent('input', { value })
    },
    
    onConfirm(e) {
      const value = e.detail.value.trim()
      if (!value) {
        wx.showToast({ title: '请输入搜索内容', icon: 'none' })
        return
      }
      
      this.triggerEvent('search', { query: value })
    },
    
    onFocus() {
      this.setData({ focused: true })
      this.triggerEvent('focus')
    },
    
    onBlur() {
      this.setData({ focused: false })
      this.triggerEvent('blur')
    },
    
    // 清理方法
    cleanup() {
      // 清理定时器等资源
    }
  }
})
```

## API 调用规范

### 1. API客户端封装
```javascript
// utils/api-client.js
class ApiClient {
  constructor() {
    this.baseURL = 'https://api.nkuwiki.com'
    this.timeout = 10000
  }
  
  // 通用请求方法
  async request(options) {
    const { url, method = 'GET', data, header = {} } = options
    
    // 添加通用请求头
    const defaultHeader = {
      'Content-Type': 'application/json',
      'Authorization': this.getAuthToken()
    }
    
    const requestHeader = { ...defaultHeader, ...header }
    
    return new Promise((resolve, reject) => {
      wx.request({
        url: `${this.baseURL}${url}`,
        method,
        data,
        header: requestHeader,
        timeout: this.timeout,
        success: (res) => {
          this.handleResponse(res, resolve, reject)
        },
        fail: (error) => {
          this.handleError(error, reject)
        }
      })
    })
  }
  
  // 响应处理
  handleResponse(res, resolve, reject) {
    const { statusCode, data } = res
    
    if (statusCode === 200) {
      if (data.code === 200) {
        resolve(data)
      } else {
        reject(new Error(data.message || '请求失败'))
      }
    } else {
      reject(new Error(`HTTP ${statusCode}: ${this.getErrorMessage(statusCode)}`))
    }
  }
  
  // 错误处理
  handleError(error, reject) {
    let message = '网络请求失败'
    
    if (error.errMsg) {
      if (error.errMsg.includes('timeout')) {
        message = '请求超时，请检查网络连接'
      } else if (error.errMsg.includes('fail')) {
        message = '网络连接失败'
      }
    }
    
    reject(new Error(message))
  }
  
  // 获取认证token
  getAuthToken() {
    const userInfo = wx.getStorageSync('userInfo')
    return userInfo?.token || ''
  }
  
  // HTTP状态码错误信息
  getErrorMessage(statusCode) {
    const messages = {
      400: '请求参数错误',
      401: '未授权访问',
      403: '权限不足',
      404: '资源不存在',
      500: '服务器内部错误',
      502: '网关错误',
      503: '服务不可用'
    }
    return messages[statusCode] || '未知错误'
  }
  
  // GET请求
  async get(url, params = {}) {
    const queryString = this.buildQueryString(params)
    const requestUrl = queryString ? `${url}?${queryString}` : url
    
    return this.request({
      url: requestUrl,
      method: 'GET'
    })
  }
  
  // POST请求
  async post(url, data = {}) {
    return this.request({
      url,
      method: 'POST',
      data
    })
  }
  
  // 构建查询字符串
  buildQueryString(params) {
    return Object.keys(params)
      .filter(key => params[key] !== undefined && params[key] !== null)
      .map(key => `${encodeURIComponent(key)}=${encodeURIComponent(params[key])}`)
      .join('&')
  }
}

// 导出单例
export default new ApiClient()
```

### 2. 业务API封装
```javascript
// api/knowledge.js
import apiClient from '../utils/api-client'

export const knowledgeApi = {
  // 知识库搜索
  async search(params) {
    const { query, page = 1, pageSize = 10, openid } = params
    
    if (!query || !query.trim()) {
      throw new Error('搜索关键词不能为空')
    }
    
    if (!openid) {
      throw new Error('用户标识不能为空')
    }
    
    return apiClient.post('/api/knowledge/advanced-search', {
      query: query.trim(),
      page,
      page_size: pageSize,
      openid
    })
  },
  
  // 获取搜索建议
  async getSuggestions(query) {
    if (!query || query.length < 2) {
      return { data: [] }
    }
    
    return apiClient.get('/api/knowledge/suggestions', { query })
  },
  
  // 获取热门搜索
  async getHotSearches() {
    return apiClient.get('/api/knowledge/hot-searches')
  },
  
  // 记录搜索历史
  async recordSearchHistory(openid, query) {
    return apiClient.post('/api/wxapp/search-history', {
      openid,
      query,
      timestamp: Date.now()
    })
  }
}
```

## 用户交互规范

### 1. 加载状态处理
```javascript
// 页面方法示例
async loadData() {
  // 显示加载状态
  wx.showLoading({
    title: '加载中...',
    mask: true
  })
  
  try {
    const data = await this.fetchData()
    
    this.setData({ 
      list: data,
      loaded: true 
    })
    
    // 成功提示（可选）
    if (data.length === 0) {
      wx.showToast({
        title: '暂无数据',
        icon: 'none'
      })
    }
    
  } catch (error) {
    console.error('加载数据失败:', error)
    
    // 错误提示
    wx.showToast({
      title: error.message || '加载失败',
      icon: 'none',
      duration: 2000
    })
    
  } finally {
    // 隐藏加载状态
    wx.hideLoading()
  }
}

// 下拉刷新
async onPullDownRefresh() {
  try {
    await this.refreshData()
  } catch (error) {
    this.showError('刷新失败')
  } finally {
    wx.stopPullDownRefresh()
  }
}

// 上拉加载更多
async onReachBottom() {
  if (this.data.loading || !this.data.hasMore) {
    return
  }
  
  this.setData({ loading: true })
  
  try {
    const newData = await this.loadMoreData()
    
    this.setData({
      list: [...this.data.list, ...newData],
      hasMore: newData.length > 0,
      loading: false
    })
    
  } catch (error) {
    this.setData({ loading: false })
    this.showError('加载更多失败')
  }
}
```

### 2. 用户反馈规范
```javascript
// 成功反馈
showSuccess(message, duration = 1500) {
  wx.showToast({
    title: message,
    icon: 'success',
    duration
  })
}

// 错误反馈
showError(message, duration = 2000) {
  wx.showToast({
    title: message,
    icon: 'none',
    duration
  })
}

// 确认对话框
showConfirm(title, content) {
  return new Promise((resolve) => {
    wx.showModal({
      title,
      content,
      confirmText: '确定',
      cancelText: '取消',
      success: (res) => {
        resolve(res.confirm)
      },
      fail: () => {
        resolve(false)
      }
    })
  })
}

// 操作确认示例
async deletePost(postId) {
  const confirmed = await this.showConfirm(
    '删除确认',
    '确定要删除这条帖子吗？删除后无法恢复。'
  )
  
  if (!confirmed) {
    return
  }
  
  try {
    await postApi.delete(postId)
    this.showSuccess('删除成功')
    this.refreshList()
  } catch (error) {
    this.showError('删除失败')
  }
}
```

## 存储和缓存规范

### 1. 本地存储管理
```javascript
// utils/storage.js
export const storage = {
  // 设置用户信息
  setUserInfo(userInfo) {
    try {
      wx.setStorageSync('userInfo', userInfo)
      return true
    } catch (error) {
      console.error('保存用户信息失败:', error)
      return false
    }
  },
  
  // 获取用户信息
  getUserInfo() {
    try {
      return wx.getStorageSync('userInfo') || null
    } catch (error) {
      console.error('获取用户信息失败:', error)
      return null
    }
  },
  
  // 设置搜索历史
  setSearchHistory(history) {
    try {
      const maxLength = 20 // 最多保存20条历史
      const limitedHistory = history.slice(0, maxLength)
      wx.setStorageSync('searchHistory', limitedHistory)
      return true
    } catch (error) {
      console.error('保存搜索历史失败:', error)
      return false
    }
  },
  
  // 获取搜索历史
  getSearchHistory() {
    try {
      return wx.getStorageSync('searchHistory') || []
    } catch (error) {
      console.error('获取搜索历史失败:', error)
      return []
    }
  },
  
  // 添加搜索记录
  addSearchRecord(query) {
    if (!query || !query.trim()) {
      return
    }
    
    const history = this.getSearchHistory()
    const trimmedQuery = query.trim()
    
    // 移除重复项
    const newHistory = [trimmedQuery, ...history.filter(item => item !== trimmedQuery)]
    
    this.setSearchHistory(newHistory)
  },
  
  // 清除所有数据
  clear() {
    try {
      wx.clearStorageSync()
      return true
    } catch (error) {
      console.error('清除存储失败:', error)
      return false
    }
  },
  
  // 移除指定key
  remove(key) {
    try {
      wx.removeStorageSync(key)
      return true
    } catch (error) {
      console.error(`移除存储${key}失败:`, error)
      return false
    }
  }
}
```

### 2. 缓存策略
```javascript
// utils/cache.js
class CacheManager {
  constructor() {
    this.cachePrefix = 'nkuwiki_cache_'
    this.defaultTTL = 5 * 60 * 1000 // 5分钟
  }
  
  // 设置缓存
  set(key, data, ttl = this.defaultTTL) {
    const cacheKey = this.cachePrefix + key
    const cacheData = {
      data,
      timestamp: Date.now(),
      ttl
    }
    
    try {
      wx.setStorageSync(cacheKey, cacheData)
      return true
    } catch (error) {
      console.error('设置缓存失败:', error)
      return false
    }
  }
  
  // 获取缓存
  get(key) {
    const cacheKey = this.cachePrefix + key
    
    try {
      const cacheData = wx.getStorageSync(cacheKey)
      
      if (!cacheData) {
        return null
      }
      
      // 检查是否过期
      const { data, timestamp, ttl } = cacheData
      if (Date.now() - timestamp > ttl) {
        this.remove(key)
        return null
      }
      
      return data
    } catch (error) {
      console.error('获取缓存失败:', error)
      return null
    }
  }
  
  // 移除缓存
  remove(key) {
    const cacheKey = this.cachePrefix + key
    
    try {
      wx.removeStorageSync(cacheKey)
      return true
    } catch (error) {
      console.error('移除缓存失败:', error)
      return false
    }
  }
  
  // 清除所有缓存
  clearAll() {
    try {
      const { keys } = wx.getStorageInfoSync()
      
      keys.forEach(key => {
        if (key.startsWith(this.cachePrefix)) {
          wx.removeStorageSync(key)
        }
      })
      
      return true
    } catch (error) {
      console.error('清除缓存失败:', error)
      return false
    }
  }
}

export default new CacheManager()
```

## 性能优化规范

### 1. 图片处理
```javascript
// 图片懒加载组件使用
// wxml
<image 
  src="{{item.imageUrl}}" 
  lazy-load="{{true}}"
  mode="aspectFill"
  loading="eager"
  bindload="onImageLoad"
  binderror="onImageError"
/>

// js
onImageLoad(e) {
  console.log('图片加载成功')
},

onImageError(e) {
  console.error('图片加载失败:', e.detail.errMsg)
  // 可以设置默认图片
  // this.setData({ 'list[index].imageUrl': '/images/default-avatar.png' })
}

// 图片压缩工具
export const imageUtils = {
  // 压缩图片
  compressImage(filePath, quality = 80) {
    return new Promise((resolve, reject) => {
      wx.compressImage({
        src: filePath,
        quality,
        success: resolve,
        fail: reject
      })
    })
  },
  
  // 获取图片信息
  getImageInfo(src) {
    return new Promise((resolve, reject) => {
      wx.getImageInfo({
        src,
        success: resolve,
        fail: reject
      })
    })
  }
}
```

### 2. 数据优化
```javascript
// 数据分页加载
async loadPage(page = 1, pageSize = 10) {
  const cacheKey = `page_${page}_${pageSize}`
  
  // 尝试从缓存获取
  let data = cache.get(cacheKey)
  
  if (!data) {
    // 从API获取
    data = await this.fetchPageData(page, pageSize)
    
    // 缓存数据
    cache.set(cacheKey, data, 2 * 60 * 1000) // 2分钟缓存
  }
  
  return data
}

// 防抖搜索
debounceSearch: null,

onSearchInput(e) {
  const query = e.detail.value
  
  // 清除之前的定时器
  if (this.debounceSearch) {
    clearTimeout(this.debounceSearch)
  }
  
  // 设置新的定时器
  this.debounceSearch = setTimeout(() => {
    this.performSearch(query)
  }, 500) // 500ms防抖
},

// 节流滚动
throttleScroll: null,
scrollTop: 0,

onPageScroll(e) {
  if (this.throttleScroll) {
    return
  }
  
  this.throttleScroll = setTimeout(() => {
    this.scrollTop = e.scrollTop
    this.handleScroll()
    this.throttleScroll = null
  }, 100) // 100ms节流
}
```

## 组件开发规范

### 1. 可复用组件设计
```javascript
// components/post-item/index.js
Component({
  properties: {
    post: {
      type: Object,
      required: true
    },
    showActions: {
      type: Boolean,
      value: true
    },
    compact: {
      type: Boolean,
      value: false
    }
  },
  
  data: {
    liked: false,
    likeCount: 0
  },
  
  lifetimes: {
    attached() {
      this.initComponent()
    }
  },
  
  methods: {
    initComponent() {
      const { post } = this.data
      if (post) {
        this.setData({
          liked: post.liked || false,
          likeCount: post.likeCount || 0
        })
      }
    },
    
    // 点击帖子
    onTapPost() {
      const { post } = this.data
      this.triggerEvent('tap', { post })
    },
    
    // 点赞操作
    async onLike() {
      const { post, liked, likeCount } = this.data
      
      try {
        // 乐观更新UI
        this.setData({
          liked: !liked,
          likeCount: liked ? likeCount - 1 : likeCount + 1
        })
        
        // 调用API
        await postApi.toggleLike(post.id, !liked)
        
        // 触发事件通知父组件
        this.triggerEvent('like', {
          postId: post.id,
          liked: !liked
        })
        
      } catch (error) {
        // 回滚UI
        this.setData({
          liked,
          likeCount
        })
        
        wx.showToast({
          title: '操作失败',
          icon: 'none'
        })
      }
    },
    
    // 分享操作
    onShare() {
      const { post } = this.data
      this.triggerEvent('share', { post })
    }
  }
})
```

### 2. 组件样式规范
```css
/* components/post-item/index.wxss */

/* 组件根容器 */
.post-item {
  background: #fff;
  border-radius: 12rpx;
  margin: 20rpx;
  padding: 24rpx;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.1);
}

/* 紧凑模式 */
.post-item.compact {
  margin: 10rpx;
  padding: 16rpx;
}

/* 标题样式 */
.post-title {
  font-size: 32rpx;
  font-weight: 600;
  color: #333;
  line-height: 1.4;
  margin-bottom: 16rpx;
  
  /* 文本截断 */
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}

/* 内容样式 */
.post-content {
  font-size: 28rpx;
  color: #666;
  line-height: 1.5;
  margin-bottom: 20rpx;
  
  /* 文本截断 */
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;
  overflow: hidden;
}

/* 操作栏 */
.post-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 16rpx;
  border-top: 1rpx solid #f0f0f0;
}

/* 点赞按钮 */
.like-btn {
  display: flex;
  align-items: center;
  padding: 8rpx 16rpx;
  border-radius: 20rpx;
  background: #f8f8f8;
  transition: all 0.3s ease;
}

.like-btn.liked {
  background: #ffe6e6;
  color: #ff4757;
}

.like-icon {
  width: 32rpx;
  height: 32rpx;
  margin-right: 8rpx;
}

.like-count {
  font-size: 24rpx;
}

/* 响应式设计 */
@media (max-width: 375px) {
  .post-title {
    font-size: 30rpx;
  }
  
  .post-content {
    font-size: 26rpx;
  }
}
```

## 错误处理和调试

### 1. 全局错误处理
```javascript
// app.js
App({
  onLaunch() {
    this.initErrorHandler()
  },
  
  // 初始化错误处理
  initErrorHandler() {
    // 监听小程序错误
    wx.onError((error) => {
      console.error('小程序错误:', error)
      this.reportError('miniprogram_error', error)
    })
    
    // 监听未处理的Promise拒绝
    wx.onUnhandledRejection((event) => {
      console.error('未处理的Promise拒绝:', event)
      this.reportError('unhandled_rejection', event.reason)
    })
  },
  
  // 错误上报
  reportError(type, error) {
    // 只在生产环境上报
    if (this.globalData.env !== 'production') {
      return
    }
    
    const errorInfo = {
      type,
      message: error.message || error,
      stack: error.stack,
      timestamp: Date.now(),
      userAgent: wx.getSystemInfoSync(),
      userId: this.globalData.userInfo?.openid
    }
    
    // 上报到错误监控服务
    wx.request({
      url: 'https://api.nkuwiki.com/api/error-report',
      method: 'POST',
      data: errorInfo,
      success: () => {
        console.log('错误上报成功')
      },
      fail: (err) => {
        console.error('错误上报失败:', err)
      }
    })
  }
})
```

### 2. 调试工具
```javascript
// utils/debug.js
class DebugHelper {
  constructor() {
    this.isDebug = wx.getAccountInfoSync().miniProgram.envVersion !== 'release'
  }
  
  // 调试日志
  log(...args) {
    if (this.isDebug) {
      console.log('[DEBUG]', ...args)
    }
  }
  
  // 性能监控
  time(label) {
    if (this.isDebug) {
      console.time(label)
    }
  }
  
  timeEnd(label) {
    if (this.isDebug) {
      console.timeEnd(label)
    }
  }
  
  // 显示调试信息
  showDebugInfo(title, data) {
    if (!this.isDebug) {
      return
    }
    
    wx.showModal({
      title: `调试信息: ${title}`,
      content: JSON.stringify(data, null, 2),
      showCancel: false
    })
  }
  
  // 网络请求日志
  logRequest(url, options, response) {
    if (!this.isDebug) {
      return
    }
    
    console.group('🌐 API请求')
    console.log('URL:', url)
    console.log('请求参数:', options)
    console.log('响应数据:', response)
    console.groupEnd()
  }
}

export default new DebugHelper()
```

---
> Source: [NKU-WIKI/nkuwiki](https://github.com/NKU-WIKI/nkuwiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
