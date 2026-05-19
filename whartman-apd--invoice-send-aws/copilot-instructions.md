## invoice-send-aws

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Docker-based application with two automated functions for QuickBooks Online invoice automation:
1. **Send Invoices** - Sends today's invoices to clients daily and emails summary to bookkeeper
2. **Create Invoices** - Creates monthly invoices from recurring transactions (task minutes from ClickUp)

## Common Commands

All commands run from `send_qbo_invoices/` directory:

### Docker Operations

```bash
# Build the Docker image
docker-compose build

# Run Send Invoices function
docker-compose run --rm send-invoices

# Run Create Invoices function
docker-compose run --rm create-invoices

# View logs
docker-compose logs

# Clean up containers
docker-compose down

# Rebuild and run (after code changes)
docker-compose build && docker-compose run --rm send-invoices
```

### Local Development

```bash
# Install dependencies locally for development
pip install -r requirements.txt

# Run functions directly (requires .env file)
python entrypoint.py send-invoices
python entrypoint.py create-invoices
```

### Environment Configuration

```bash
# Copy environment template
cp .env.template .env

# Edit .env file with your credentials
# Required variables:
# - AWS_REGION
# - QBO_SECRET_NAME / MSGRAPH_SECRET_NAME (for AWS Secrets Manager)
# - Or individual credentials if not using Secrets Manager
```

## Project Structure

```
send_qbo_invoices/
├── Dockerfile                 # Docker container definition
├── docker-compose.yml         # Docker Compose configuration
├── entrypoint.py             # Main entry point for Docker container
├── requirements.txt          # Python dependencies
├── .env.template            # Environment variable template
├── .env                     # Local environment variables (git-ignored)
├── shared/                  # Shared modules
│   ├── process_and_send_qbo_invoices.py  # Send invoices orchestration
│   ├── task_minutes_to_clickup_and_qbo.py # Create invoices from ClickUp
│   ├── apd_quickbooksonline.py           # QBO API wrapper
│   ├── apd_msgraph_v2.py                 # Microsoft Graph API wrapper
│   └── apd_common.py                     # HTML template processor
└── assets/                   # Shared assets
    └── sent_invoices_email_template.html
```

## Architecture

### Docker Services

#### send-invoices
- **Purpose**: Queries QBO for today's invoices, sends them, emails summary to bookkeeper
- **Entry Point**: `entrypoint.py send-invoices`
- **Core Function**: `send_qbo_invoices()` from `process_and_send_qbo_invoices.py`

#### create-invoices
- **Purpose**: Creates invoices from recurring transactions (ClickUp task minutes)
- **Entry Point**: `entrypoint.py create-invoices`
- **Core Function**: `process_all_clients()` from `task_minutes_to_clickup_and_qbo.py`

### Core Modules (in `shared/`)

- **process_and_send_qbo_invoices.py** - Main orchestration: queries QBO for today's invoices, sends them, generates summary email
- **task_minutes_to_clickup_and_qbo.py** - Creates invoices from ClickUp task minutes for billing period
- **apd_quickbooksonline.py** - QuickBooks Online API wrapper with OAuth token refresh, retry logic, and custom exceptions (QBOError, QBOAuthError, etc.)
- **apd_msgraph_v2.py** - Microsoft Graph API wrapper for sending emails via Office 365
- **apd_common.py** - HTML template processor (`APD_Html_Template` class)

### External Dependencies

- **AWS Secrets Manager** - Stores credentials at paths defined by `QBO_SECRET_NAME` and `MSGRAPH_SECRET_NAME` environment variables
  - QBO credentials are automatically updated when refresh tokens are rotated
- **QuickBooks Online API** - Invoice queries and sending
- **Microsoft Graph API** - Email delivery
- **ClickUp API** - Task minutes data retrieval

### Configuration

Environment variables (defined in `.env` file):
- `FUNCTION_NAME`: Which function to run (`send-invoices` or `create-invoices`)
- `AWS_REGION`: AWS region for Secrets Manager (default: `us-west-2`)
- `QBO_SECRET_NAME`: AWS Secrets Manager path for QBO credentials
- `MSGRAPH_SECRET_NAME`: AWS Secrets Manager path for Microsoft Graph credentials
- `EXCLUDED_CUSTOMERS`: Comma-separated list of customer names to exclude from invoice sending
- `BOOKKEEPER_EMAIL`: Email address for daily summary reports
- `SENDER_EMAIL`: Email address used as sender for notifications

## Important Notes

- Docker container uses Python 3.11 slim base image
- Code changes require rebuilding the Docker image (`docker-compose build`)
- `.env` file contains sensitive credentials and is git-ignored
- QBO refresh tokens are automatically saved back to Secrets Manager after rotation
- Logs are written to `logs/` directory (git-ignored)

## Deployment

For production deployment to AWS ECS, see [README.ECS-DEPLOYMENT.md](README.ECS-DEPLOYMENT.md)

---
> Source: [whartman-APD/invoice_send_aws](https://github.com/whartman-APD/invoice_send_aws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
