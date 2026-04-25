## flip-the-ai

> bun run lint         # biome check .

# Meme Starter Pack

## Commands

```bash
# Линтинг и форматирование (biome — monorepo-aware)
bun run lint         # biome check .
bun run lint:fix     # biome check --write .
bun run format       # biome format --write .
bun run knip         # Проверка unused deps/files

# Website (cd workers/website)
bun run dev          # Vite dev (localhost:5173)
bun run build        # Сборка
bun run typecheck    # Проверка типов
bun run deploy       # Деплой на CF Workers

# Crypto API (cd workers/crypto-api)
bun run dev          # Wrangler dev (localhost:8787)
bun run build        # Сборка
bun run typecheck    # Проверка типов
bun run deploy       # Деплой на CF Workers

# Database (cd database)
bun run db:generate  # Генерация миграций Drizzle
bun run db:push      # Применение миграций к D1

# Remotion (cd local/remotion)
bun run dev          # Remotion Studio
bun run build        # Рендер видео
```

## Projects

### workers/website — `@meme-starter/website`
React + Vite SPA на Cloudflare Workers. Лендинг мем-токена.

| Библиотека | Версия |
|---|---|
| React | ^19.2.4 |
| Vite | ^8.0.0-beta.13 |
| @cloudflare/vite-plugin | ^1.24.0 |
| Tailwind CSS | ^4.1.18 |
| @tailwindcss/vite | ^4.1.18 |
| vite-tsconfig-paths | ^6.1.0 |
| babel-plugin-react-compiler | ^1.0.0 |
| ky | ^1.14.3 |
| TypeScript | ~5.9.3 |

### workers/crypto-api — `@meme-starter/crypto-api`
Hono API на Cloudflare Workers. Прокси к DexScreener.

| Библиотека | Версия |
|---|---|
| Hono | ^4.11.9 |
| ky | ^1.14.3 |
| Zod | ^4.3.6 |
| Wrangler | ^4.64.0 |

#### Live Token Widget
- `GET /token/:chain/:address` — прокси к DexScreener API (бесплатно, без ключа, 60 req/min)
- Поддерживаемые chain: `solana`, `ethereum`, `bsc`
- Возвращает: price, volume, liquidity, marketCap, fdv, priceChange, pairUrl
- Работает с любым токеном на DexScreener (включая pump.fun после graduation)
- Настройка токена: `workers/website/src/sections/TokenLive.tsx` → `CHAIN`, `TOKEN_ADDRESS`
- Production API URL: там же → `API_URL`

### database — `@meme-starter/database`
Drizzle ORM + Cloudflare D1. Схема и типы. НЕ инстанциировать drizzle() в пакете — только в рантайме воркера.

| Библиотека | Версия |
|---|---|
| drizzle-orm | ^0.45.1 |
| drizzle-kit | ^0.31.9 |

### local/remotion — Remotion видео
Вне bun workspaces, запускается независимо. Zod строго 3.22.3.

| Библиотека | Версия |
|---|---|
| Remotion | 4.0.419 |
| Zod | 3.22.3 (строго!) |
| React | 19.2.3 |
| Tailwind CSS | 4.0.0 |
| @remotion/transitions | ^4.0.419 |
| @remotion/bundler | ^4.0.419 |
| @remotion/renderer | ^4.0.419 |
| @remotion/google-fonts | 4.0.419 |

## Project Structure

```
meme_starter_pack/
├── package.json              # Root: workspaces ["workers/*", "database"]
├── tsconfig.base.json        # Shared TypeScript config
├── biome.json                # Linter/formatter
├── knip.json                 # Unused deps checker
├── bun.lock
├── .gitignore
│
├── workers/
│   ├── website/              # @meme-starter/website (React + Vite SPA)
│   │   ├── vite.config.ts
│   │   ├── wrangler.jsonc
│   │   ├── index.html
│   │   └── src/
│   │       ├── main.tsx
│   │       ├── App.tsx
│   │       ├── index.css
│   │       └── sections/     # Page sections (Hero, About, etc.)
│   │
│   └── crypto-api/           # @meme-starter/crypto-api
│       ├── wrangler.jsonc
│       └── src/
│           ├── index.ts
│           ├── types.ts
│           ├── routes/
│           │   └── token.ts      # GET /token/:chain/:address
│           └── services/
│               └── dexscreener.ts # DexScreener API client
│
├── database/                 # @meme-starter/database
│   ├── drizzle.config.ts
│   ├── drizzle/              # Миграции
│   └── src/
│       ├── index.ts
│       └── schema/
│
└── local/
    └── remotion/             # Вне workspaces, независимый
        ├── remotion.config.ts
        ├── render.ts
        ├── cli/
        ├── public/
        └── src/
            ├── Root.tsx
            ├── index.ts
            ├── index.css
            ├── components/
            ├── compositions/
            └── lib/
```

## SEO & Crawler Optimization (workers/website)

При генерации или модификации сайта, AI-агент ОБЯЗАН выполнить следующее:

### Обязательные файлы
- `index.html` — заполнить все плейсхолдеры `{{...}}` реальными данными проекта
- `public/robots.txt` — заменить `{{DOMAIN}}` на реальный домен
- `public/sitemap.xml` — заменить `{{DOMAIN}}` и `{{CURRENT_DATE}}`
- `public/images/og-preview.png` — OG-изображение 1200x630px (описать генерацию или добавить плейсхолдер)

### index.html чеклист
- [ ] `<html lang="en">` — язык страницы
- [ ] `<title>` — уникальный заголовок (до 60 символов)
- [ ] `<meta name="description">` — описание (до 160 символов)
- [ ] `<meta name="keywords">` — ключевые слова через запятую
- [ ] `<meta name="robots" content="index, follow">`
- [ ] `<meta name="theme-color">` — цвет браузерной панели
- [ ] `<link rel="canonical">` — абсолютный URL
- [ ] `<link rel="icon">` — favicon
- [ ] Open Graph теги: og:type, og:title, og:description, og:url, og:site_name, og:image (с width/height)
- [ ] Twitter Card теги: twitter:card, twitter:site, twitter:title, twitter:description, twitter:image
- [ ] JSON-LD `<script type="application/ld+json">` с WebSite + Organization
- [ ] `<noscript>` блок с полным текстовым содержанием всех секций сайта
- [ ] Font preconnect для Google Fonts (если используются)

### Семантика HTML (в React-компонентах)
- `<header>` для шапки, `<main>` для контента, `<footer>` для подвала
- Один `<h1>` на странице, далее `<h2>`, `<h3>` по иерархии
- `<section id="...">` для якорной навигации
- `<nav>` для навигационных блоков
- `alt` атрибуты на всех `<img>`
- `rel="noopener noreferrer"` на внешних ссылках с `target="_blank"`

### Noscript Fallback
Блок `<noscript>` в index.html должен содержать:
- Заголовок H1 с названием и тегпайном проекта
- Краткое описание каждой секции сайта (About, Features, etc.)
- Все внешние ссылки (соцсети, документация)
- Текстовую версию ключевых данных (фичи, roadmap steps, etc.)
Это критично для ботов Telegram, Discord, и legacy-краулеров, которые не исполняют JS.

### Изображения
- OG-изображение: строго 1200x630px, PNG или JPG, до 500KB
- Сжимать изображения перед коммитом (WebP для контента, PNG для OG)
- Добавлять `loading="lazy"` на изображения ниже fold
- НЕ добавлять lazy loading на hero-изображение (первый экран)

### Wrangler (Cloudflare Workers)
Текущая конфигурация SPA fallback (`not_found_handling: "single-page-application"`)
корректна для одностраничника. При добавлении маршрутов — учитывать 404 handling.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dimadem) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
