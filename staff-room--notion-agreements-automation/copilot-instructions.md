## notion-agreements-automation

> **Core Project**: n8n workflow to send Notion agreements via Documenso for e-signature.

# CLAUDE.md

**Core Project**: n8n workflow to send Notion agreements via Documenso for e-signature.

## What This Does

Takes a Notion agreement → Creates Documenso envelope → Sends for signature → Updates Notion status.

## Required Setup

```bash
# n8n Environment Variables
NOTION_TOKEN=secret_xxx
DOCUMENSO_API_URL=https://your-instance.com  
DOCUMENSO_TOKEN=xxx
DOCUMENSO_TEMPLATE_ID=xxx
```

## Key Files

- `workflows/documenso-send.json` - Main n8n workflow
- Notion database ID: `c0923656ee1d41de882310cab0ddbd3f`

## Core Workflow

1. **Webhook Trigger**: `/webhook/agreements/send`
2. **Fetch Agreement**: Get data from Notion API
3. **Create Envelope**: Send to Documenso API
4. **Update Notion**: Store envelope ID + signing URL

## Testing

```bash
# Test the workflow
curl -X POST http://localhost:5678/webhook/agreements/send \
  -H "Content-Type: application/json" \
  -d '{"agreement_url": "https://notion.so/xxx"}'
```

## Common Issues

- **"Agreement not found"**: Check Notion token permissions
- **"Documenso error"**: Verify API token and template ID
- **Webhook fails**: Check n8n logs for API errors

## Development

- Use ngrok for local webhook testing
- Test with Documenso sandbox environment  
- Keep credentials in n8n environment variables only

The workflow JSON file contains the actual implementation.

---
> Source: [Staff-Room/notion-agreements-automation](https://github.com/Staff-Room/notion-agreements-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
