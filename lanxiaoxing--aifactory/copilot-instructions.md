## aifactory

> - React 18 + TypeScript 5.2

# 项目：仿 CATL 风格企业官网

## 技术栈
- React 18 + TypeScript 5.2
- Vite 构建工具
- Leaflet 地图库（轻量级地图解决方案）
- CSS Modules 样式管理
- react-router-dom 路由管理

## 项目结构
- `src/components/`：可重用组件
  - `Header/`：顶部导航菜单
  - `Map/`：世界地图组件
  - `Footer/`：页脚内容
- `src/pages/`：页面组件
  - `Home/`：首页（包含地图）
- `src/styles/`：全局样式
- `src/types/`：TypeScript 类型定义
- `src/utils/`：工具函数

## 核心功能实现

### 1. 菜单导航
typescript

// src/components/Header/index.tsx

const menuItems = [

{

title: '关于我们',

subItems: ['公司简介', '发展历程', '企业文化'] // 子菜单仅结构，无需页面

},

{

title: '产品中心',

subItems: ['手机', '电池', '回收利用']

},

// 其他菜单项...

];

### 2. 世界地图实现
typescript

// src/components/Map/WorldMap.tsx

import { MapContainer, TileLayer, Marker, Popup } from 'react-leaflet';

const highlightedCountries = [

{ name: 'India', position: [20.5937, 78.9629] },

{ name: 'Indonesia', position: [-0.7893, 113.9213] },

{ name: 'Brazil', position: [-14.2350, -51.9253] }

];

const WorldMap = () => (

<MapContainer center={[20, 0]} zoom={2}>

<TileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" />

{highlightedCountries.map(country => (

<Marker position={country.position} key={country.name}>

<Popup>{country.name}</Popup>

</Marker>

))}

</MapContainer>

);

### 3. 样式规范
css

/* src/styles/global.css */

:root {

--catl-blue: #003366;

--catl-accent: #ff6600;

}

.header {

background-color: var(--catl-blue);

color: white;

height: 80px;

}

## 开发命令
- `npm run dev`：启动开发服务器（端口 3000）
- `npm run build`：构建生产版本
- `npm run lint`：运行 ESLint 检查
- `npm run typecheck`：运行 TypeScript 类型检查

## 代码规范
1. **组件设计**：
   - 使用函数组件和 Hooks
   - 所有组件使用 TypeScript 类型定义
   - 组件文件命名使用 PascalCase

2. **地图实现要求**：
   - 使用 Leaflet 的 React 封装
   - 高亮国家使用橙色标记（与 CATL 品牌色一致）
   - 添加适当的缩放控件

3. **响应式设计**：
   - 菜单在移动端显示汉堡菜单
   - 地图容器高度至少 500px

## 注意事项
1. 不需要实现后端或数据库连接
2. 所有菜单项只需实现基本路由，子菜单不需要实际页面
3. 地图数据使用 OpenStreetMap 的免费瓦片服务
4. 高亮国家的坐标数据已预先定义，如需调整请更新 `highlightedCountries` 数组

## 参考资源
- CATL 官网设计参考：https://www.catl.com/
- Leaflet 官方文档：https://leafletjs.com/
- react-leaflet 示例：https://react-leaflet.js.org/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lanxiaoxing) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
