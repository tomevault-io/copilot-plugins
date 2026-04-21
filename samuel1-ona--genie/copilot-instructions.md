## genie

> We’re integrating the Genie-Proposal-Summarizer React/Vite frontend with the live AO agent.


We’re integrating the Genie-Proposal-Summarizer React/Vite frontend with the live AO agent.

Principles:

- The frontend **never** calls Tally/Discord/Telegram directly; it talks to the AO agent via actions (ScrapeGovernance, AddSubscriber, BroadcastNotification, etc.).
- A single backend bridge `/api/ao` forwards messages to the AO network/relay.
- Admin-only actions are gated (server-side) to prevent abuse.
- All env creds live in `.env` and are **never** exposed to the browser.

High-level flow:
UI → /api/ao (bridge) → AO network → Genie agent handlers → (internal) Tally/Discord/Telegram.

We will:

1. Replace the mock AO bridge with a **real /api/ao** server route.
2. Add minimal **admin auth** to protect sensitive actions (scrape/clear/reset/balance).
3. Harden client with **retry, timeout, and optimistic UI**.
4. Add **observability** (toasts, logs) and **production configs**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Samuel1-ona) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
