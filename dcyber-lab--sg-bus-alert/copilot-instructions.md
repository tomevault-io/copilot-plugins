## sg-bus-alert

> Project: `sg-bus-alert`

# Gemini Project Memory

Project: `sg-bus-alert`

## Purpose

Send Telegram reminders for Singapore commute buses and answer Telegram status queries.

## Core runtime facts

- Language: Node.js
- Entry: `index.mjs`
- Scheduler: `systemd --user`
- Poll interval: 10 seconds
- State file: `state.json`
- Config file: `.env`

## External services

- `arrivelah2.busrouter.sg` for live arrivals
- `open-meteo.com` for weather
- Telegram Bot API for chat polling and message sending

## Functional behavior

- Proactive weekday morning reminders
- On-demand `状态` / route query replies
- Same-day mute with `上车了` or `暂停`
- Resume with `恢复`

## If modifying

- Keep docs aligned with code
- Keep user-facing output in Chinese unless requested otherwise
- Preserve the 3-arrival display format unless the user asks to redesign it

---
> Source: [dcyber-lab/sg-bus-alert](https://github.com/dcyber-lab/sg-bus-alert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
