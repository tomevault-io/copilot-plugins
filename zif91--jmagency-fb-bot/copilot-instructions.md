## jmagency-fb-bot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Telegram bot for monitoring Facebook advertising accounts, designed to be deployed on Google Cloud Functions. The bot provides automated daily reports, hourly critical status checks, and on-demand account information via Telegram inline buttons.

## Development Commands

This project is intended to be deployed on Google Cloud Functions. Based on the technical specification, the following commands would be used:

### Deployment Commands
```bash
# Deploy daily report function
gcloud functions deploy daily-report \
  --runtime python39 \
  --trigger-topic daily-report-topic \
  --entry-point daily_report

# Deploy hourly check function  
gcloud functions deploy hourly-check \
  --runtime python39 \
  --trigger-topic hourly-check-topic \
  --entry-point hourly_check

# Deploy telegram webhook function
gcloud functions deploy telegram-webhook \
  --runtime python39 \
  --trigger-http \
  --entry-point telegram_webhook
```

### Cloud Scheduler Setup
```bash
# Daily report at 10:00 UTC+6 (04:00 UTC)
gcloud scheduler jobs create pubsub daily-report-job \
  --schedule="0 4 * * *" \
  --topic=daily-report-topic \
  --message-body="{}"

# Hourly critical checks
gcloud scheduler jobs create pubsub hourly-check-job \
  --schedule="0 * * * *" \
  --topic=hourly-check-topic \
  --message-body="{}"
```

## Architecture

### Core Components
The system consists of three main Cloud Functions:

1. **daily-report-function** - Scheduled daily reports at 10:00 UTC+6
2. **hourly-check-function** - Hourly critical status monitoring  
3. **telegram-webhook-function** - Handles Telegram bot interactions and inline button callbacks

### File Structure (Planned)
```
├── main.py                 # Main Cloud Functions entry points
├── facebook_client.py      # Facebook Marketing API client
├── telegram_client.py      # Telegram Bot API client  
├── requirements.txt        # Python dependencies
├── deploy.sh              # Deployment script
└── README.md              # Documentation
```

### Monitored Facebook Accounts
```python
FB_ACCOUNTS = [
    '1295277935230804',  # KZ Astana
    '160299730351768',   # KZ Almaty
    '1779179689511929',  # JM_Tanuki_Almaty
    '349938931436634',   # kaspiika uz
    '564758959363164',   # bella ciao rest
    '848382079950987'    # Tanuki UZ
]
```

## Integration Details

### Facebook Marketing API
- **Account Status**: `/{account_id}?fields=account_status,balance,spend_cap,amount_spent`
- **Spending Data**: `/{account_id}/insights?time_range={...}&fields=spend`
- **Activities/Errors**: `/{account_id}/activities?fields=event_type,extra_data`

### Telegram Bot Interface
Main menu structure with inline keyboards:
- 📈 Account Status - Overview of all account statuses
- 💰 Balances - Current balances and debts
- 📊 Spending - Submenu for different time periods (yesterday, week, month)
- ⚠️ Errors - Current moderation errors and warnings
- 🔄 Refresh - Force update all data

### Environment Variables
```bash
FB_ACCESS_TOKEN=<Facebook access token>
TELEGRAM_BOT_TOKEN=<Telegram bot token>
TELEGRAM_CHAT_ID=<Target chat ID for notifications>
```

## Key Features

### Daily Reports (10:00 UTC+6)
Automated comprehensive status report including:
- Account status (active/restricted/blocked)
- Current balance or debt
- Previous day spending
- Spending limits
- Moderation status

### Hourly Critical Monitoring
Real-time alerts for:
- Account blocks
- Spending limit violations
- Payment issues
- Critical moderation errors
- Facebook warnings

### On-Demand Queries
Interactive Telegram interface with callback handlers for immediate data access.

## Security & Performance Notes

- All tokens stored in Google Secret Manager
- Rate limiting implementation for Facebook API calls
- Parallel API requests for performance
- 5-minute data caching
- Request timeouts and retry mechanisms
- IAM roles with minimal required permissions
- Chat ID whitelist for bot access control

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zif91)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zif91)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
