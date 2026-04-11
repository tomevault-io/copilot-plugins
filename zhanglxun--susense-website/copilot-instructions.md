## susense-website

> 这是一个为“深圳市速睿森视科技有限公司”构建的企业官网。项目主要用于展示公司的数字化转型、软件开发及 AI 应用等业务。

# Gemini 项目分析报告

## 1. 项目概览
这是一个为“深圳市速睿森视科技有限公司”构建的企业官网。项目主要用于展示公司的数字化转型、软件开发及 AI 应用等业务。
网站采用纯静态 HTML 结构，设计风格偏向现代企业官网，包含多个业务板块的介绍页面。

## 2. 技术栈
- **核心**: 静态 HTML5, CSS3, JavaScript (ES5/ES6)
- **框架**: Bootstrap 4.3.1
- **库/插件**:
  - jQuery (DOM 操作)
  - Flickity (轮播图)
  - AOS (滚动动画)
  - Jarallax (视差滚动)
  - Mapbox GL (地图展示)
  - Feather Icons (图标)
- **构建工具**: 无 (纯静态，无 Node.js 构建流程)

## 3. 项目结构
```
/
├── *.html              # 各个业务页面 (index, about, news, case-*, solution-*)
├── assets/
│   ├── css/            # 样式文件 (theme.min.css)
│   ├── js/             # 脚本文件 (layout.js, theme.min.js)
│   ├── img/            # 图片资源
│   └── libs/           # 第三方库
├── scripts/            # 可能包含一些辅助脚本
└── README.md / CLAUDE.md # 项目文档
```

## 4. 关键机制
- **公共组件注入**: 头部 (`header`) 和底部 (`footer`) 并非服务端渲染或构建时生成，而是通过 `assets/js/layout.js` 在客户端动态注入。这简化了开发但可能影响首屏加载和 SEO。
- **资源引用**: 存在混合引用情况。部分资源引用本地 `assets/libs`，但也有大量 CSS/JS 引用远程地址 `http://image.susense.cn/`。

## 5. 现状与问题 (基于扫描)
1.  **断链与缺失**: 文档中提到存在指向不存在页面 (`overview.html`) 或 CSS 文件 (`theme.min2.css`) 的链接。
2.  **外部依赖风险**: 强依赖 `image.susense.cn`，如果该服务器不稳定，网站样式和功能将受损。建议本地化。
3.  **SEO 不足**: 页面缺少 `meta description`，且客户端注入导航对搜索引擎不够友好。
4.  **维护性**: `layout.js` 中硬编码了 HTML 字符串，修改导航或页脚比较繁琐且容易出错。

## 6. 初步建议
- **资源本地化**: 将所有远程依赖下载到本地 `assets/libs`，确保网站独立运行。
- **修复断链**: 清理或修复所有无效链接。
- **SEO 优化**: 为每个页面添加独立的 Title 和 Meta Description。
- **工程化 (可选)**: 如果后续更新频繁，考虑引入简单的静态站点生成器 (如 Vite, Gulp 或 简单的 Node 脚本) 来处理公共头尾的组装，替代运行时注入。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhanglxun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zhanglxun)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
