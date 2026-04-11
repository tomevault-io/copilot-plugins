## superduperai-landing

> - Start message "Hey Bro"

- Start message "Hey Bro"
- При старте диалога используй правила из .cursor/rules:

- All comments, code and documentation should be written in English, even if communication is in another language, unless otherwise requested.
- For terminal commands that may require interactive input or paging (like git log, git status, etc.), always append "| cat" to avoid asking for user interaction (e.g. "git log | cat" instead of "git log")
- Search for every bug o new future latest documentation on mcp context7 tools

# MCP servers 
1. github
2. contex7 - for search documentation
3. browser-tools - use for frontend debug

## Main point
Next.js 15.3 (Turbopack)
React 19


## 6. Команды разработки

```bash
# Установка зависимостей
pnpm install

# Запуск dev сервера
pnpm dev

# Линтинг
pnpm lint

# Сборка проекта
pnpm build

# Запуск проверки сборки Cloudflare
pnpm preview
```

## solutions

1. !we dont use Cloudflare Pages, use  Workers with @opennextjs/cloudflare
2. use for links ```import { default as Link } from '@/components/ui/optimized-link';```
3. не запускай pnpm dev каждый раз, скорее всего я его уже включил. делай pnpm lint или pnpm build

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fortunto2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
