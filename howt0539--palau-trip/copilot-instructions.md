## palau-trip

> 帛琉旅行伴侶 PWA — 記帳、行程記錄、生物圖鑑、航班住宿、緊急資訊。

# palau-trip

帛琉旅行伴侶 PWA — 記帳、行程記錄、生物圖鑑、航班住宿、緊急資訊。

## Tech Stack

- 純 HTML/CSS/JS（單一 index.html）
- Cloudflare Pages 部署（palau-trip.pages.dev）
- Cloudflare Worker 代理 Notion API（worker/）
- PWA manifest + apple-mobile-web-app

## Deploy

- 前端：`bash deploy.sh`
- Worker：`cd worker && npx wrangler deploy`

## Design Context

### Users
- 目前是 Howt 和 Ashley 使用的帛琉旅行伴侶 app
- 未來可能做成通用旅行 app 模板給其他人用
- 使用情境：旅行前規劃、旅行中記帳/記錄、旅行後回顧
- Mobile-first（iPhone Safari），透過 howt-base proxy 存取

### Brand Personality
**療癒、像素風、探險感** — Dave the Diver 風格，輕鬆可愛但不幼稚

### Emotional Goals
**期待與興奮** — 旅行前的倒數感覺

### Anti-references
- 企業軟體風格（冷灰、密集表格、無趣 dashboard）
- 太童趣/卡通化的像素風

### Design Principles
1. **Deep Sea Immersion** — 強化「潛入深海」沉浸感
2. **Pixel Art Consistency** — 所有 icon 使用像素風格 SVG
3. **Glass Over Solid** — 半透明毛玻璃效果，讓深海背景透出來
4. **Tool First, Theme Second** — 核心功能可用性優先，主題不妨礙操作
5. **Mobile Thumb Zone** — 互動元素在拇指可及範圍，最小 44px 點擊區域

---
> Source: [howt0539/palau-trip](https://github.com/howt0539/palau-trip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
