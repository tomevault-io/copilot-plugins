## bwg-monitor-skill

> Monitor BandwagonHost (搬瓦工) VPS stock and notify when plans become available. Supports manual check and cron-based auto monitoring.


# BWG Stock Monitor

Monitor BandwagonHost (搬瓦工) CN2 GIA-E VPS stock. Notify via Telegram when plans come back in stock.

## Usage

### Check stock now (one-shot)

```bash
python3 {baseDir}/scripts/check_stock.py
```

Returns JSON:
```json
{
  "timestamp": "2026-02-11 23:00:00",
  "products": [
    {"pid": 94, "name": "CN2 GIA-E 10G", "price": "49.99", "in_stock": false, "url": "..."},
    ...
  ],
  "in_stock_count": 0,
  "text": "..."
}
```

### Output

- `text` — Human-readable stock summary for messaging
- `products` — Full product list with stock status
- `in_stock_count` — Number of products currently in stock
- `changed` — List of products whose stock status changed since last check

When `in_stock_count > 0`, alert the user immediately with the buy link.
When `changed` contains items, notify the user of status changes.

## Configuration

Edit `{baseDir}/config.json` to add/remove monitored plans:

```json
{
  "products": [
    {"pid": 94, "name": "CN2 GIA-E 10G", "price": "49.99", "desc": "10G SSD / 0.5G RAM / 1Gbps"},
    ...
  ]
}
```

## Cron Setup

For automatic monitoring, set up an OpenClaw cron job:
- Schedule: every 3 minutes (`*/3 * * * *`)
- Payload: `agentTurn` with message "Run BWG stock check and notify me if anything changed"

## Migration

Copy the entire `skills/bwg-monitor/` folder to another OpenClaw instance's `skills/` directory.

---
> Source: [ylongw/bwg-monitor-skill](https://github.com/ylongw/bwg-monitor-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
