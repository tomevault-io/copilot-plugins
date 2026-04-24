## journeybox

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JourneyBox (旅迹盒子) - AI智能旅行规划助手。基于React的旅行规划应用,使用DeepWisdom API(gpt-4o模型)生成真实可行的旅行计划,集成Unsplash图片和高德地图服务。

**技术栈**: React 18.2.0, React Router 6, Ant Design 5.x, Framer Motion, Axios

## 开发命令

```bash
npm install          # 安装依赖
npm start            # 启动开发服务器 (http://localhost:3000)
npm run build        # 生产构建 (输出到 build/)
npm test             # 运行测试 (当前无测试文件)
```

**调试技巧**:
```bash
# 查看环境变量加载情况
# 检查浏览器控制台的 "【环境变量调试】" 输出 (aiService.js:13)

# 清理localStorage (解决状态混乱)
# 浏览器控制台: localStorage.clear()
```

## 环境变量配置

复制 `.env.example` 为 `.env` 后配置:

```env
# DeepWisdom AI服务 (必需) - https://newapi.deepwisdom.ai/
REACT_APP_DEEPWISDOM_API_KEY=your_key
REACT_APP_DEEPWISDOM_BASE_URL=https://newapi.deepwisdom.ai/v1
REACT_APP_DEEPWISDOM_MODEL=gpt-4o

# Unsplash图片服务 (可选) - https://unsplash.com/developers
REACT_APP_UNSPLASH_ACCESS_KEY=your_key

# 高德地图服务 (可选) - https://lbs.amap.com
REACT_APP_AMAP_API_KEY=your_key
REACT_APP_AMAP_SECURITY_KEY=your_key
REACT_APP_AMAP_WEB_KEY=your_key
```

## 核心架构

### 数据流

```
CreateTripEnhanced (4步向导)
    ↓
generateTravelPlan() [aiService.js]
    ↓
GeneratingTripPage (动画等待)
    ↓
ItineraryPage (展示结果)
```

### 路由结构 (src/App.js)

**主要路由**:

| 路由 | 页面 | 说明 |
|------|------|------|
| `/` | HomePage | 行程列表 |
| `/create-trip` | CreateTripEnhanced | 创建向导(主入口) |
| `/generating-trip` | GeneratingTripPage | AI生成动画 |
| `/itinerary/:id` | ItineraryPage | 详细日程(63KB,最复杂) |
| `/community` | CommunityPage | 社区分享 |
| `/community/post/:id` | PostDetailPage | 帖子详情 |
| `/community/create` | CreatePostPage | 创建帖子 |
| `/auth` | AuthPage | 登录注册 |
| `/discover` | DiscoverPage | 发现页面 |
| `/profile` | ProfilePage | 个人中心 |
| `/history` | HistoryPage | 历史记录 |


### 服务层 (src/api/)

| 文件 | 职责 | 数据源 |
|------|------|--------|
| `aiService.js` | AI生成/优化行程 | DeepWisdom API |
| `tripService.js` | 行程CRUD | Mock内存数据 |
| `userService.js` | 用户认证 | Mock内存数据 |
| `unsplashService.js` | 图片搜索 | Unsplash API |
| `mapService.js` | 地图服务 | 高德地图API |
| `weatherService.js` | 天气服务 | 心知天气API |
| `postService.js` | 社区帖子 | Mock内存数据 |
| `communityService.js` | 社交功能 | Mock内存数据 |

**重要**: 当前无真实后端,所有Mock数据存在内存中,刷新页面后丢失

## AI服务关键细节 (aiService.js)

### Token限制

```javascript
COMPLETION_TOKEN_LIMIT = 3500   // 生成行程
OPTIMIZATION_TOKEN_LIMIT = 3000 // 优化行程
// chat/completions需留缓冲避免截断
```

### 请求节流

```javascript
minRequestInterval = 5000  // 5秒最小间隔
// 违反会触发429错误
```

### 重试策略

- 默认重试1次
- 基础延迟10秒
- 429错误: 延迟×10 (100秒)
- 其他错误: 延迟×3 (30秒)

### 核心函数

```javascript
generateTravelPlan(tripData)  // 生成行程
// tripData: {destination, startDate, endDate, budget, interests, travelStyle, participants}
// 返回: {overview, tips, days[], accommodation, transportation} 或 {error, errorDetails, canRetry}

optimizeTripPlan(currentPlan, options)  // 优化行程
askTravelQuestion(question, tripContext)  // AI问答
```

### 使用限制与注意事项

**请求间隔**: 必须至少间隔5秒,否则触发429错误
```javascript
// aiService.js 自动节流
lastRequestTime + 5000 > now → 等待剩余时间
```

**Token限制**:
- 代码实际: 3500 (生成) / 3000 (优化)
- **影响**: 5天以上行程可能被截断

**JSON解析增强**: aiService包含自动修复中文标点/缺失引号的逻辑
```javascript
// fixJsonString() 自动处理:
"：" → ":"  // 中文冒号
"，" → ","  // 中文逗号
unquoted_key → "quoted_key"  // 属性名加引号
```

### 错误处理

| 状态码 | 含义 | canRetry | 处理策略 |
|--------|------|----------|---------|
| 429 | 频率限制 | ✓ | 延迟100秒后重试 |
| 400 | 请求格式错误 | ✗ | 检查tripData格式 |
| 401/403 | 认证失败 | ✗ | 检查API密钥 |
| 500+ | 服务器错误 | ✓ | 延迟30秒后重试 |

**常见失败原因**:
1. 请求间隔<5秒 → 429错误
2. 长行程(5天+) → JSON截断解析失败
3. API密钥未配置 → 使用硬编码后备密钥(不安全)

## 关键页面

### CreateTripEnhanced (4步向导)

1. **目的地**: 12城市快选(北京/上海/杭州/成都/西安/三亚/厦门/丽江/重庆/青岛/大理/苏州) + 自定义
2. **日期**: 范围选择,自动计算天数
3. **偏好**: 8种兴趣标签 + 5档预算等级
4. **风格**: 轻松悠闲/平衡适中/紧凑充实

### ItineraryPage (最复杂页面)

**功能**:
- 时间轴展示每日行程
- 景点卡片含时间/地点/描述/费用/图片
- Unsplash图片集成(每个景点单独请求)
- AI优化建议抽屉
- 实时编辑功能
- 地图集成

**已知问题**:
- 文件过大(1479行,63KB),20+个useState
- 大量Unsplash API调用无缓存
- 职责混杂(展示/生成/地图/预订)
- 需要拆分重构

## 约定与模式

### 图片路径

- 公共图片: `/image/filename.jpg` (无`/public`前缀)
- 城市封面: `/image/beijing.jpg`
- Unsplash: 使用UnsplashImage组件自动处理归属

### Mock数据模式

```javascript
// service文件顶部定义mock数据
const mockTrips = [{...}];

// 函数返回Promise + setTimeout模拟延迟
export const getTrips = () => {
  return new Promise(resolve => {
    setTimeout(() => resolve(mockTrips), 500);
  });
};
```

### 错误处理模式

```javascript
// 服务层: 抛出用户友好错误
catch (error) {
  console.error('描述性错误信息', error);
  throw new Error('用户友好的错误消息');
}

// 组件层: 显示Ant Design消息
catch (error) {
  message.error(error.message);
}
```

### 认证状态 (AuthContext)

```javascript
const { user, isAuthenticated, loading, login, logout } = useAuth();
```

### localStorage使用(需要重构)

**当前散乱使用情况**:

| 位置 | Key | 用途 | 问题 |
|------|-----|------|------|
| `userService.js:5` | `users` | 用户列表 | 无前缀,可能冲突 |
| `userService.js:20` | `currentUser` | 当前登录用户 | 密码明文存储 |
| `unsplashService.js:82` | `forceOnlineImages` | 强制在线图片 | 与useLocalImages冲突 |
| `unsplashService.js:127` | `useLocalImages` | 使用本地图片 | 逻辑互斥但可能同时true |
| `ItineraryPage.js:211` | `startAiGeneration` | AI生成启动标记 | 临时标记,易遗留 |
| `ItineraryPage.js:139` | `forceOfflineMode` | 离线模式 | 命名不一致 |

**建议**: 实现统一的storage抽象层

**调试命令**:
```javascript
// 查看所有localStorage
Object.keys(localStorage).forEach(k =>
  console.log(k, localStorage.getItem(k))
);

// 清理所有数据
localStorage.clear();
```

## 技术债务

**P0 - 安全问题(立即修复)**:
1. API密钥硬编码 (`aiService.js:5`) - 已泄露至代码库
2. 密码明文存储 (`userService.js:93-97`) - localStorage无加密

**P1 - 用户体验影响**:
3. ItineraryPage拆分 - 63KB过大,20+个useState
4. Unsplash调用优化 - 每景点单独请求,无缓存
5. localStorage统一管理 - 6处散乱使用,无命名空间

**P2 - 技术债务**:
6. Mock数据分离 - `tripService.js`中1258行硬编码数据
7. 代码分割 - App.js同步导入所有页面
8. 错误处理统一 - 3种不同错误返回模式

**P3 - 长期规划**:
9. 后端实现 - 所有service需真实API替换mock
10. TypeScript迁移 - 提升类型安全
11. 测试覆盖 - 缺少单元测试和集成测试

## 会话工作流(长期代理模式)

每次会话开始时:
```bash
pwd                                    # 确认目录
cat claude-progress.txt 2>/dev/null   # 读取进度
git log --oneline -10                  # 查看提交历史
git status                             # 检查工作状态
```

会话结束前:
```bash
# 1. 提交更改
git add -A
git commit -m "type(scope): description"

# 2. 更新进度文件
cat > claude-progress.txt << 'EOF'
## Session: $(date +%Y-%m-%d\ %H:%M)

### 完成的工作
- [具体描述]

### 下一步
- [明确的行动项]

### 已知问题
- [未解决的问题,如有]
EOF
```

**原则**:
- 一次只处理一个功能
- 保持代码在"清洁状态"
- 小步迭代,避免大规模重写

## 部署

**关键配置**:
```bash
# Vercel环境变量必须配置
REACT_APP_DEEPWISDOM_API_KEY=your_key  # 必需
REACT_APP_AMAP_API_KEY=your_key         # 可选(地图功能)
REACT_APP_UNSPLASH_ACCESS_KEY=your_key  # 可选(图片功能)
```

**注意事项**:
- Vercel环境变量不要选"Add Secret",直接粘贴值
- 需要为Production/Preview/Development都配置
- 部署后需要"Redeploy"才能生效

## 项目特点

**优势**:
- AI驱动,生成真实可行的旅行计划
- 4步向导,用户体验流畅
- Unsplash高质量图片集成
- 精美的生成动画效果

**局限**:
- 无真实后端,数据刷新后丢失
- Mock数据占用大量代码空间
- 部分页面过大(ItineraryPage 63KB)
- 缺少单元测试

**最佳实践建议**:
1. 优先修复P0安全问题(API密钥泄露)
2. 考虑拆分ItineraryPage(63KB过大)
3. 遵循现有Mock数据模式保持一致性

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/calderbuild) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
