## deno-domain-checker

> The Discord integration handles all notifications for domain status changes.

# Discord Integration

The Discord integration handles all notifications for domain status changes.

## Implementation

The Discord functionality is implemented in @discord.ts and provides:

- Webhook-based notifications
- Rich message formatting
- Status change alerts
- Configurable mentions

## Configuration

Discord integration requires:
1. A webhook URL (set via `DISCORD_WEBHOOK_URL` in `.env`)
2. Optional mention configuration (set via `DISCORD_MENTION` in `.env`)

## Notification Types

The system sends different types of notifications:

### Domain Status Changes
- Domain becomes available
- Domain enters redemption period
- Domain enters pending delete status
- Domain status changes to active

### System Notifications
- Application startup
- Error conditions
- Rate limit warnings

## Message Format

Notifications include:
- Domain name
- Current status
- Previous status (for changes)
- Timestamp
- Relevant links
- Color-coded status indicators

## Example Configuration

```env
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/your-webhook-url
DISCORD_MENTION=@domain-alerts
```

## Error Handling

The Discord integration includes:
- Retry logic for failed notifications
- Rate limit handling
- Webhook validation
- Fallback notification methods

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yutakobayashidev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
