## moblie

> 本项目是一个基于 React 18 + TypeScript 的现代化移动端 Web 应用，采用 Vite 作为构建工具，集成了丰富的前端技术栈，具备良好的可维护性和扩展性。

# 项目开发指南

## 1. 项目概述

本项目是一个基于 React 18 + TypeScript 的现代化移动端 Web 应用，采用 Vite 作为构建工具，集成了丰富的前端技术栈，具备良好的可维护性和扩展性。

## 2. 技术栈

### 核心技术
- **框架**: React 18.3.1 + TypeScript 5.5.3
- **构建工具**: Vite 6.0.5
- **状态管理**: MobX 6.13.5 + MobX React 9.1.1
- **路由**: React Router DOM 6.27.0 + react-activation
- **UI 组件库**: Ant Design Mobile 5.38.1
- **样式**: Sass 1.79.6 + PostCSS

### 功能技术
- **网络请求**: Axios 1.7.7
- **表单处理**: React Hook Form 7.54.2 + Zod 3.24.2
- **动画**: react-spring 9.7.5 + motion 12.0.1
- **工具库**: dayjs、crypto-js、qrcode、html2canvas 等
- **数据可视化**: ECharts 5.6.0

## 3. 项目结构

```
src/
├── api/              # API 接口定义
│   ├── config/       # API 配置
│   └── product/      # 业务模块 API
├── assets/           # 静态资源
│   ├── css/          # 全局样式
│   └── images/       # 图片资源
├── business/         # 业务逻辑
│   ├── channel/      # 渠道相关
│   ├── hooks/        # 业务 Hook
│   └── order/        # 订单相关
├── components/       # 组件
│   ├── business/     # 业务组件
│   ├── common/       # 通用组件
│   └── ui/           # UI 组件
├── core-tools/       # 核心工具
│   ├── http/         # HTTP 工具
│   ├── jsBridge/     # JSBridge
│   └── sdk/          # SDK 集成
├── pages/            # 页面组件
├── routes/           # 路由配置
├── types/            # TypeScript 类型定义
├── utils/            # 工具函数
├── App.tsx           # 应用入口组件
├── main.tsx          # 应用入口文件
└── styles.css        # 全局样式
```

## 4. 开发流程

### 4.1 环境配置

1. 安装依赖
```bash
npm install
```

2. 启动开发服务器
```bash
# 开发环境
npm run dev

# 测试环境
npm run test-dev

# 预发布环境
npm run pre-dev

# 生产环境
npm run prd-dev
```

3. 构建项目
```bash
# 开发环境构建
npm run dev-build

# 生产环境构建
npm run build
```

### 4.2 代码提交

项目使用 Husky 进行 Git 钩子管理，提交代码前会自动执行：
- ESLint 代码检查
- Stylelint 样式检查
- 提交信息规范检查

提交信息格式要求：
```
<type>(<scope>): <subject>

<body>

<footer>
```

常用 type：
- feat: 新功能
- fix: 修复 bug
- docs: 文档更新
- style: 代码格式调整
- refactor: 代码重构
- test: 测试相关
- chore: 构建或依赖更新

## 5. 代码规范

### 5.1 TypeScript 规范

1. **类型定义**
   - 为所有变量、函数参数和返回值添加类型
   - 使用接口（interface）定义对象类型
   - 使用类型别名（type）定义复杂类型组合

2. **命名规范**
   - 组件名：PascalCase（如 `UserProfile.tsx`）
   - 变量名：camelCase（如 `userInfo`）
   - 常量名：UPPER_CASE（如 `API_BASE_URL`）
   - 类型名：PascalCase（如 `UserType`）

3. **代码风格**
   - 使用 4 个空格缩进
   - 使用双引号
   - 每行不超过 120 个字符
   - 适当使用空行分隔代码块

### 5.2 React 组件规范

1. **组件类型**
   - 优先使用函数组件
   - 使用 React Hooks 管理组件状态和副作用

2. **组件结构**
   ```typescript
   // 导入
   import React, { useState, useEffect } from "react";
   import { Button } from "antd-mobile";
   
   // 类型定义
   interface Props {
     title: string;
     onClick: () => void;
   }
   
   // 组件
   const CustomButton: React.FC<Props> = ({ title, onClick }) => {
     // 状态管理
     const [loading, setLoading] = useState(false);
     
     // 副作用
     useEffect(() => {
       // 初始化逻辑
     }, []);
     
     // 事件处理
     const handleClick = async () => {
       setLoading(true);
       try {
         await onClick();
       } finally {
         setLoading(false);
       }
     };
     
     // 渲染
     return (
       <Button loading={loading} onClick={handleClick}>
         {title}
       </Button>
     );
   };
   
   export default CustomButton;
   ```

3. **Hooks 使用规范**
   - 只在函数组件或自定义 Hook 中使用 Hooks
   - 遵循 Hooks 的调用顺序
   - 使用 ESLint 插件确保 Hooks 使用正确

### 5.3 MobX 状态管理规范

1. **Store 结构**
   ```typescript
   import { makeObservable, observable, action, computed } from "mobx";
   
   class UserStore {
     // 可观察状态
     userInfo = null;
     loading = false;
     
     constructor() {
       makeObservable(this, {
         userInfo: observable,
         loading: observable,
         isLoggedIn: computed,
         setUserInfo: action,
         setLoading: action,
         fetchUserInfo: action
       });
     }
     
     // 计算属性
     get isLoggedIn() {
       return !!this.userInfo;
     }
     
     // 动作
     setUserInfo(userInfo) {
       this.userInfo = userInfo;
     }
     
     setLoading(loading) {
       this.loading = loading;
     }
     
     // 异步动作
     async fetchUserInfo() {
       this.setLoading(true);
       try {
         const response = await api.getUserInfo();
         this.setUserInfo(response.data);
       } catch (error) {
         console.error("Failed to fetch user info:", error);
       } finally {
         this.setLoading(false);
       }
     }
   }
   
   export default new UserStore();
   ```

2. **组件中使用 Store**
   ```typescript
   import React, { useEffect } from "react";
   import { observer } from "mobx-react";
   import userStore from "../stores/userStore";
   
   const UserProfile: React.FC = () => {
     useEffect(() => {
       userStore.fetchUserInfo();
     }, []);
     
     if (userStore.loading) {
       return <div>Loading...</div>;
     }
     
     return (
       <div>
         <h1>{userStore.userInfo?.name}</h1>
         <p>{userStore.userInfo?.email}</p>
       </div>
     );
   };
   
   export default observer(UserProfile);
   ```

### 5.4 API 调用规范

1. **API 配置**
   - 在 `src/api/config/` 中配置 API 基础 URL 和拦截器
   - 根据环境自动加载对应的环境变量

2. **API 定义**
   ```typescript
   // src/api/product/queryProductRate.ts
   import request from "../http";
   import { QueryProductRateParams, QueryProductRateResponse } from "../../types";
   
   /**
    * 查询产品利率
    * @param params 查询参数
    * @returns 产品利率数据
    */
   export const queryProductRate = (
     params: QueryProductRateParams
   ): Promise<QueryProductRateResponse> => {
     return request({
       url: "/api/product/rate",
       method: "GET",
       params
     });
   };
   ```

3. **组件中调用 API**
   ```typescript
   import React, { useState, useEffect } from "react";
   import { queryProductRate } from "../api/product";
   
   const ProductRate: React.FC = () => {
     const [rate, setRate] = useState<number>(0);
     const [loading, setLoading] = useState<boolean>(false);
     
     useEffect(() => {
       const fetchRate = async () => {
         setLoading(true);
         try {
           const response = await queryProductRate({ productId: "123" });
           setRate(response.rate);
         } catch (error) {
           console.error("Failed to fetch product rate:", error);
         } finally {
           setLoading(false);
         }
       };
       
       fetchRate();
     }, []);
     
     return (
       <div>
         {loading ? 'Loading...' : `Current rate: ${rate}%`}
       </div>
     );
   };
   ```

### 5.5 样式规范

1. **Sass 使用**
   - 使用 SCSS 语法
   - 利用变量定义颜色、字体等全局样式
   - 使用混合宏（mixin）实现样式复用
   - 使用函数（function）处理复杂样式逻辑

2. **样式文件组织**
   - 全局样式：`src/styles.css` 和 `src/assets/css/`
   - 组件样式：与组件文件同目录，使用 `.module.scss` 后缀
   - 页面样式：与页面组件同目录

3. **移动端适配**
   - 使用 `postcss-px-to-viewport-8-plugin` 进行 px 到 vw 的转换
   - 设计稿基准：375px 宽度
   - 确保在不同设备上的显示效果一致

## 6. 组件开发

### 6.1 组件分类

1. **UI 组件**：基础 UI 元素，如按钮、输入框等
2. **业务组件**：包含业务逻辑的组件，如订单列表、产品卡片等
3. **页面组件**：完整的页面，如首页、个人中心等

### 6.2 组件开发流程

1. **需求分析**：明确组件的功能和使用场景
2. **设计**：确定组件的 API、状态和样式
3. **实现**：编写组件代码
4. **测试**：编写单元测试和集成测试
5. **文档**：编写组件使用文档

### 6.3 组件 API 设计

1. **Props 设计**
   - 明确组件的输入参数
   - 使用 TypeScript 接口定义 Props 类型
   - 为可选参数提供默认值

2. **事件处理**
   - 使用回调函数处理组件内部事件
   - 事件命名使用 `on` 前缀，如 `onClick`、`onChange`

3. **样式定制**
   - 支持通过 className 自定义样式
   - 支持通过 style 属性覆盖内联样式

## 7. 路由管理

### 7.1 路由配置

1. **主路由**：`src/routes/index.tsx`
2. **子路由**：`src/routes/subRoutes/`
3. **标签页路由**：`src/routes/tabRoutes.tsx`

2. **路由配置示例**
   ```typescript
   import { Routes, Route } from "react-router-dom";
   import Home from "../pages/Tabs/Home";
   import Product from "../pages/Tabs/Product";
   import Investment from "../pages/Tabs/Investment";
   import My from "../pages/Tabs/My";
   
   const TabRoutes: React.FC = () => {
     return (
       <Routes>
         <Route path="/home" element={<Home />} />
         <Route path="/product" element={<Product />} />
         <Route path="/investment" element={<Investment />} />
         <Route path="/my" element={<My />} />
       </Routes>
     );
   };
   
   export default TabRoutes;
   ```

### 7.2 路由跳转

1. **组件内跳转**
   ```typescript
   import React from "react";
   import { useNavigate } from "react-router-dom";
   import { Button } from "antd-mobile";
   
   const Home: React.FC = () => {
     const navigate = useNavigate();
     
     const handleGoToProduct = () => {
       navigate("/product");
     };
     
     return (
       <Button onClick={handleGoToProduct}>
         查看产品
       </Button>
     );
   };
   ```

2. **编程式跳转**
   ```typescript
   import { useNavigate } from "react-router-dom";
   
   const navigate = useNavigate();
   navigate("/path", { state: { data: "value" } });
   ```

## 8. 测试规范

### 8.1 测试类型

1. **单元测试**：测试单个组件或函数
2. **集成测试**：测试组件之间的交互
3. **E2E 测试**：测试完整的用户流程

### 8.2 测试工具

项目使用 Jest 和 React Testing Library 进行测试。

### 8.3 测试文件命名

测试文件与被测试文件同目录，使用 `.test.tsx` 或 `.spec.tsx` 后缀。

### 8.4 测试示例

```typescript
   import React from "react";
   import { render, screen, fireEvent } from "@testing-library/react";
   import CustomButton from "./CustomButton";

describe("CustomButton", () => {
  it("should render correctly", () => {
    render(<CustomButton title="Click me" onClick={() => {}} />);
    expect(screen.getByText("Click me")).toBeInTheDocument();
  });
  
  it("should call onClick when clicked", () => {
    const mockOnClick = jest.fn();
    render(<CustomButton title="Click me" onClick={mockOnClick} />);
    
    fireEvent.click(screen.getByText("Click me"));
    expect(mockOnClick).toHaveBeenCalledTimes(1);
  });
  
  it("should show loading state", () => {
    render(<CustomButton title="Click me" onClick={() => {}} loading={true} />);
    expect(screen.getByRole("button")).toHaveAttribute("disabled");
  });
});
```

## 9. 性能优化

### 9.1 组件优化

1. **使用 React.memo 优化函数组件**
   ```typescript
   const MemoizedComponent = React.memo((props) => {
     // 组件逻辑
   });
   ```

2. **使用 useMemo 优化计算值**
   ```typescript
   const expensiveValue = useMemo(() => {
     return calculateExpensiveValue(a, b);
   }, [a, b]);
   ```

3. **使用 useCallback 优化回调函数**
   ```typescript
   const handleClick = useCallback(() => {
     console.log("Clicked");
   }, []);
   ```

### 9.2 渲染优化

1. **虚拟列表**：使用 react-virtualized 处理长列表
2. **懒加载**：使用 React.lazy 和 Suspense 实现组件懒加载
3. **图片优化**：使用适当尺寸的图片，考虑使用 WebP 格式

### 9.3 网络优化

1. **API 请求优化**
   - 使用缓存减少重复请求
   - 合并多个请求
   - 实现请求节流和防抖

2. **资源加载优化**
   - 使用 CDN 加速静态资源
   - 启用 Gzip 压缩
   - 实现资源预加载

## 10. 部署规范

### 10.1 构建配置

1. **环境配置**：根据不同环境使用不同的配置文件
2. **资源路径**：确保构建后的资源路径正确
3. **代码分割**：使用动态导入实现代码分割

### 10.2 部署流程

1. 构建生产版本
   ```bash
   npm run build
   ```

2. 上传构建产物到服务器

3. 配置 Nginx
   ```nginx
   server {
     listen 80;
     server_name example.com;
     
     root /path/to/build;
     index index.html;
     
     location / {
       try_files $uri $uri/ /index.html;
     }
     
     location /api {
       proxy_pass http://api.example.com;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
     }
   }
   ```

## 11. 常见问题与解决方案

### 11.1 依赖冲突

**问题**：安装新依赖时出现版本冲突

**解决方案**：
1. 使用 `npm ls <package-name>` 查看依赖树
2. 使用 `npm install <package-name>@<version>` 安装特定版本
3. 考虑使用 yarn 或 pnpm 管理依赖

### 11.2 构建失败

**问题**：执行 `npm run build` 失败

**解决方案**：
1. 检查 TypeScript 类型错误
2. 检查 ESLint 代码规范错误
3. 检查 Stylelint 样式规范错误
4. 查看构建日志，定位具体错误信息

### 11.3 运行时错误

**问题**：应用运行时出现错误

**解决方案**：
1. 查看浏览器控制台错误信息
2. 使用 React DevTools 调试组件状态
3. 使用 MobX DevTools 调试状态管理
4. 检查网络请求是否正常

## 12. 附录

### 12.1 常用命令

| 命令 | 描述 |
|------|------|
| npm run dev | 启动开发服务器 |
| npm run build | 构建生产版本 |
| npm run lint | 执行代码检查 |
| npm run preview | 预览生产构建 |
| npm run prepare | 安装 Husky 钩子 |

### 12.2 开发工具推荐

1. **IDE**：WebStorm、VS Code
2. **插件**：
   - ESLint
   - Prettier
   - Stylelint
   - React DevTools
   - MobX DevTools
   - TypeScript Hero

### 12.3 学习资源

1. **React 文档**：https://react.dev/
2. **TypeScript 文档**：https://www.typescriptlang.org/
3. **Vite 文档**：https://vitejs.dev/
4. **MobX 文档**：https://mobx.js.org/
5. **Ant Design Mobile 文档**：https://mobile.ant.design/

---

**最后更新时间**：2026-01-18
**版本**：1.0.0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShrekYan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
