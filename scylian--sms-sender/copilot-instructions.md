## sms-sender

> This file provides guidance to WARP (warp.dev) when working with code in this repository.

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

Node.js CLI application for sending SMS messages via the Twilio API.

## Commands

```bash
# Install dependencies
npm install

# Send an SMS (requires .env configured)
node index.js <phone_number> "<message>"

# Or using npm script
npm start -- <phone_number> "<message>"
```

## Architecture

Single-file application (`index.js`) that:
- Loads Twilio credentials from environment variables via `dotenv`
- Exports `sendSms(to, message)` for programmatic use
- Provides CLI interface when run directly

## Environment Setup

Copy `.env.example` to `.env` and populate with Twilio credentials:
- `TWILIO_ACCOUNT_SID`
- `TWILIO_AUTH_TOKEN`
- `TWILIO_PHONE_NUMBER`

## Conventions

- Phone numbers use E.164 format (e.g., `+1234567890`)
- The `sendSms` function is async and returns the Twilio message SID on success

---
> Source: [scylian/sms-sender](https://github.com/scylian/sms-sender) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
