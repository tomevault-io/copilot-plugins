## antdv-next-pro

> 基于 [Antdv Next](https://www.antdv-next.com) 的中后台管理系统模板，对标 Ant Design Pro。

# Antdv Next Pro

基于 [Antdv Next](https://www.antdv-next.com) 的中后台管理系统模板，对标 Ant Design Pro。

## Tech Stack

- **Framework**: Vue 3.5+ with TypeScript
- **UI Library**: antdv-next (Ant Design Vue Next)
- **Build Tool**: Vite 6
- **State Management**: Pinia 3 with persisted state
- **Router**: Vue Router 4
- **HTTP Client**: Axios with interceptors
- **Mock**: MSW (Mock Service Worker)
- **I18n**: vue-i18n
- **Icons**: @antdv-next/icons
- **Package Manager**: pnpm

## Project Structure

```
src/
├── api/            # API modules (user, dashboard, table)
├── components/     # Global components (SiderMenu, TabBar, HeaderSearch, NoticeIcon, SettingDrawer, Authority)
├── composables/    # Composables (usePermission)
├── config/         # App config (defaultSettings)
├── directives/     # Custom directives (v-auth, v-role)
├── layouts/        # Layouts (BasicLayout, UserLayout, BlankLayout)
├── locales/        # I18n language packs (zh-CN, en-US)
├── mock/           # MSW mock handlers
├── router/         # Routes + guards
├── stores/         # Pinia stores (app, user, tab)
├── styles/         # Global CSS
├── types/          # TypeScript declarations
├── utils/          # Utilities (request, helpers)
└── views/          # Page views (20+ pages)
```

## Commands

- `pnpm dev` — Start dev server (port 3100)
- `pnpm build` — Production build
- `pnpm build:staging` — Staging build
- `pnpm preview` — Preview production build
- `pnpm type-check` — TypeScript check

## Key Patterns

- **Auto Import**: Components and composables are auto-imported via unplugin
- **Permission**: Route-level (meta.authority), menu-level (filtered), button-level (v-auth directive, Authority component)
- **Theme**: ConfigProvider with runtime algorithm switching (dark/compact), SettingDrawer for visual config
- **Layout**: Three modes (side/top/mix), responsive drawer on mobile
- **Mock**: MSW handlers in src/mock/, enabled when VITE_USE_MOCK=true

## Antdv Next Knowledge Base

Full component API documentation is available at `.agents/llm-full.txt` (54k lines).
Use the antdv-next skill for component reference.

---
> Source: [NatriumGroup/Antdv-next-pro](https://github.com/NatriumGroup/Antdv-next-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
