## lab-todo

> TodoWeb 專案結構和架構指南


# TodoWeb 專案結構指南

## 專案用途

提供 Todo List Web 給予使用者進行待辦事項管理。

## 技術堆疊

- Vue 3
- Vite
- TailwindCSS
- PrimeVue UI
- Vue Router
- Pinia
- Axios
- PrimeIcons

## 資料夾結構

```
todo-web # Vue 3 前端專案
├── public
│   └── favicon.ico
├── src
│   ├── assets # 靜態資源
│   │   ├── layout # Layout 樣式
│   │   │   ├── _core.scss
│   │   │   ├── _footer.scss
│   │   │   ├── _main.scss
│   │   │   ├── _menu.scss
│   │   │   ├── _mixins.scss
│   │   │   ├── _preloading.scss
│   │   │   ├── _responsive.scss
│   │   │   ├── _topbar.scss
│   │   │   ├── _typography.scss
│   │   │   ├── _utils.scss
│   │   │   ├── layout.scss
│   │   │   └── variables # SCSS 變數
│   │   │       ├── _common.scss
│   │   │       ├── _dark.scss
│   │   │       └── _light.scss
│   │   ├── layout.scss # Layout 樣式入口
│   │   └── main.css # 全域樣式
│   ├── layout # Layout 元件
│   │   ├── AppFooter.vue
│   │   ├── AppLayout.vue # 主 Layout
│   │   ├── AppMenu.vue
│   │   ├── AppMenuItem.vue
│   │   ├── AppSidebar.vue
│   │   ├── AppTopbar.vue
│   │   └── composables
│   │       └── layout.js # Layout composable
│   ├── router
│   │   └── index.js # Vue Router 設定
│   ├── services # API 服務層
│   │   ├── auth-api.js # 驗證 API
│   │   └── axios-instance.js # Axios 實例
│   ├── utils # 工具函式
│   │   └── validators.js # 驗證工具函式
│   ├── views # 頁面元件
│   │   ├── Access.vue # 403 頁面
│   │   ├── Error.vue # 錯誤頁面
│   │   ├── HomeView.vue # 首頁
│   │   ├── Login.vue # 登入頁
│   │   ├── NotFound.vue # 404 頁面
│   │   └── todo # Todo 模組
│   │       └── Todo.vue # Todo 頁面
│   ├── App.vue # 根元件
│   └── main.js # 應用程式入口
├── .editorconfig
├── .env.development # 開發環境變數
├── .oxlintrc.json # OxLint 設定
├── .prettierrc.json # Prettier 設定
├── eslint.config.js # ESLint 設定
├── index.html # HTML 入口
├── jsconfig.json
├── package.json
├── README.md
└── vite.config.js # Vite 設定
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/a35506322)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/a35506322)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
