## albion-killbot

> Server layered architecture and file placement


# Server Architecture

Follow the existing layered layout. Do not skip layers or put business logic in interfaces.

```
interfaces/   Entry points (api, bot, crawler) — HTTP routes, Discord commands, cron controllers
services/     Business logic — orchestration, domain rules
ports/        External abstractions — database, queue, albion, discord, stripe
ports/adapters/   Concrete clients (mongoDbClient, albionApiClient, amqpClient, …)
helpers/      Shared utilities (logger, embeds, locale, cache, markers)
config/       node-config files (default.js, production.js)
```

## Patterns

**API**: `routes/` → `controllers/` → `services/` → `ports/`. Routes export `{ path, router }` and include `@openapi` JSDoc.

**Bot**: `commands/` and `controllers/` call `services/` only — never adapters directly.

**Ports**: Facade over adapters. Adapters hold HTTP/SDK calls; ports add retries, caching, and domain shaping.

```javascript
// ✅ Controller delegates to service
const eventsService = require("../../../services/events");
async function getEvent(req, res) {
  const event = await eventsService.getEvent(req.params.eventId, { server: req.query.server });
  if (!event) return res.sendStatus(404);
  return res.send(event);
}

// ❌ Controller calling albion adapter directly
const albionApiClient = require("../../../ports/adapters/albionApiClient");
```

New external integrations: add adapter under `ports/adapters/`, expose via a `ports/*.js` module, consume from services.

## Bot guild registry

Admin `/admin/servers` reads from MongoDB collection `botGuilds` via `services/botGuilds.js` (paginated `findBotGuilds`) — not Discord's paginated guild API. The bot process keeps the collection in sync:

- **Startup**: each shard upserts its cached guilds once (`interfaces/bot/controllers/servers.js` `preinit`)
- **Realtime**: `GuildCreate` upserts, `GuildDelete` removes
- **Periodic cache refresh** updates limits/settings/track only; it does not re-sync all guilds

Indexes: unique `{ id: 1 }`, `{ name: 1, id: 1 }` (see migration `bot-guilds-indexes`).

## Event image rendering

Killboard canvas code lives under `services/images/` (import as `require("…/services/images")`):

```
services/images/
  index.js              # public API: generateEventImage, generateInventoryImage
  layouts/              # image composition
    eventImage.js
    inventoryImage.js
  theme/                # visual tokens
    colors.js
    textStyles.js
  canvas/               # font registration, assetsPath, drawImage
    index.js
  draw/                 # drawing primitives
    background.js
    inventory.js
    items.js
    player.js
    stats.js
    participantBars.js
```

Post-processing (`optimizeImage` via sharp) stays in `helpers/images.js`.

## Discord monospace tables

Monospace table formatting for bot message `content` lives under `helpers/` (ported from albion-nexus):

- `table.js` — `formatTableLines`, `formatPanelMessageContent` (title `##` + code-block table + footer subtext `-#`)
- `string.js` — `monospaceDisplayWidth`, `padMonospace`, `truncatePlainTextEllipsis`
- `discordLimits.js` — Discord text caps and `truncateDiscordText`

Rankings (`embedRanking` in `helpers/embeds.js`) use message `content` only — no embed thumbnail or fields. Footer markers live in `helpers/markers.js` (`Markers.buildFooter.ranking`, `Markers.isRankingMessage`). Before posting, callers use `interfaces/bot/helpers/channels.js` (`resolveChannel`) and `messages.js` (`deleteMatchingMessages` + `Markers.isRankingMessage`) to remove prior bot posts of the same ranking type (cron + `/ranking`).

---
> Source: [agnjunio/albion-killbot](https://github.com/agnjunio/albion-killbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
