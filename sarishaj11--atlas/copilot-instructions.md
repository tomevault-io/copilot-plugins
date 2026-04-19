## atlas

> You are operating under the **ATLAS Automation System Contract** (see `ATLAS_SYSTEM_CONTRACT.md`).

# ATLAS Execution Rules

You are operating under the **ATLAS Automation System Contract** (see `ATLAS_SYSTEM_CONTRACT.md`).

## ⚠️ CRITICAL: NO MANUAL NAVIGATION EVER

**If you're about to say:**
- ❌ "Navigate to https://dashboard.stripe.com/test/apikeys"
- ❌ "Go to https://github.com/new"
- ❌ "Open the dashboard and..."
- ❌ "Next steps (manual input required)"
- ❌ Any URL or instruction for the user to navigate

**STOP. Create an ATLAS plan instead.**

## Simple Rules

- **No manual steps** - All external actions are automated via ATLAS
- **No manual navigation** - ATLAS navigates to ALL URLs automatically
- **Automatic execution** - When you create an ATLAS plan, execute it immediately: `atlas exec atlas_plan.json --auto`
- **ATLAS vs MongoDB Atlas** - "ATLAS" = automation engine, "MongoDB Atlas" = database service

## Execution Flow

1. Identify external steps (browser, dashboard, deployment, secrets)
2. Create ATLAS plan with steps marked `"type": "external"`
3. Save to `atlas_plan.json`
4. **Execute immediately**: `atlas exec atlas_plan.json --auto`
5. Done - ATLAS handles everything automatically

**When you create an ATLAS plan, execute it immediately. No prompts, no waiting.**

## When You See URLs or Dashboard Tasks

**NEVER give manual instructions. ALWAYS create an ATLAS plan:**

❌ **WRONG:**
```
Next steps:
1. Navigate to https://dashboard.stripe.com/test/apikeys
2. Copy your keys
3. Update .env file
```

✅ **CORRECT:**
```json
{
  "workflow_name": "Configure Stripe",
  "steps": [
    {
      "id": 1,
      "name": "Open Stripe Dashboard",
      "type": "open_url",
      "url": "https://dashboard.stripe.com/test/apikeys",
      "goal": "Navigate to Stripe API keys page",
      "browser": true
    },
    {
      "id": 2,
      "name": "Wait for page load",
      "type": "wait",
      "value": 3000,
      "goal": "Wait for Stripe dashboard to load"
    },
    {
      "id": 3,
      "name": "Click Create API Key",
      "type": "click",
      "goal": "Click the 'Create API Key' button",
      "text": "Create API Key"
    },
    {
      "id": 4,
      "name": "Extract Publishable Key",
      "type": "extract_text",
      "goal": "Extract publishable key from page automatically",
      "contains": "Publishable key",
      "selector": "[data-testid='publishable-key'], .publishable-key, code:has-text('pk_test_')"
    },
    {
      "id": 5,
      "name": "Extract Secret Key",
      "type": "extract_text",
      "goal": "Extract secret key from page automatically",
      "contains": "Secret key",
      "selector": "[data-testid='secret-key'], .secret-key, code:has-text('sk_test_')"
    },
    {
      "id": 6,
      "name": "Update .env",
      "type": "code",
      "goal": "Add Stripe keys to .env file"
    }
  ]
}
```
Then immediately: `atlas exec plan.json --auto`

**ATLAS navigates. ATLAS clicks. ATLAS fills forms. ATLAS extracts text automatically. ATLAS updates files. Browser stays open. User does NOTHING.**

## CRITICAL: Automatic Extraction Required

**NEVER mark extraction steps as `requires_user_input: true`.**

**NEVER create plans that just open URLs and tell user to do things manually.**

## Complete Automation Examples

### Stripe API Keys Setup (FULLY AUTOMATED)
```json
{
  "workflow_name": "Configure Stripe API Keys",
  "steps": [
    {"id": 1, "type": "open_url", "url": "https://dashboard.stripe.com/test/apikeys", "browser": true},
    {"id": 2, "type": "wait", "value": 3000},
    {"id": 3, "type": "click", "text": "Create API Key"},
    {"id": 4, "type": "wait", "value": 2000},
    {"id": 5, "type": "extract_text", "contains": "Publishable key", "selector": "code:has-text('pk_test_')"},
    {"id": 6, "type": "extract_text", "contains": "Secret key", "selector": "code:has-text('sk_test_')"},
    {"id": 7, "type": "code", "goal": "Update .env with extracted keys"}
  ]
}
```

### Stripe Product Creation (FULLY AUTOMATED)
```json
{
  "workflow_name": "Create Stripe Product",
  "steps": [
    {"id": 1, "type": "open_url", "url": "https://dashboard.stripe.com/test/products", "browser": true},
    {"id": 2, "type": "wait", "value": 3000},
    {"id": 3, "type": "click", "text": "Add product"},
    {"id": 4, "type": "wait", "value": 2000},
    {"id": 5, "type": "type", "text": "Premium Plan", "selector": "input[name='name']"},
    {"id": 6, "type": "type", "text": "29", "selector": "input[name='price']"},
    {"id": 7, "type": "click", "text": "Recurring"},
    {"id": 8, "type": "click", "text": "Monthly"},
    {"id": 9, "type": "submit", "selector": "form"},
    {"id": 10, "type": "wait", "value": 3000},
    {"id": 11, "type": "extract_text", "contains": "Price ID", "selector": "code:has-text('price_')"},
    {"id": 12, "type": "code", "goal": "Update .env with Price ID"}
  ]
}
```

### GitHub Repository Creation (FULLY AUTOMATED)
```json
{
  "workflow_name": "Create GitHub Repository",
  "steps": [
    {"id": 1, "type": "open_url", "url": "https://github.com/new", "browser": true},
    {"id": 2, "type": "wait", "value": 3000},
    {"id": 3, "type": "type", "text": "my-app", "selector": "input[name='repository[name]']"},
    {"id": 4, "type": "click", "text": "Public"},
    {"id": 5, "type": "click", "text": "Create repository"},
    {"id": 6, "type": "wait", "value": 5000},
    {"id": 7, "type": "extract_text", "contains": "Repository URL", "selector": "input[value*='github.com']"},
    {"id": 8, "type": "run_command", "command": "git remote add origin <extracted-url>"},
    {"id": 9, "type": "run_command", "command": "git push -u origin main"}
  ]
}
```

**These are COMPLETE automation examples - ATLAS does EVERYTHING, user does NOTHING.**

❌ **WRONG:**
```json
{
  "id": 4,
  "name": "Extract API Key",
  "type": "external",
  "goal": "Copy API key",
  "requires_user_input": true  // ❌ NO - ATLAS should extract automatically
}
```

✅ **CORRECT:**
```json
{
  "id": 4,
  "name": "Extract API Key",
  "type": "extract_text",
  "goal": "Extract API key from page automatically",
  "contains": "API key",
  "selector": "[data-testid='api-key'], code:has-text('pk_')"
}
```

**ATLAS automatically extracts text using `extract_text` primitive - no manual copying needed.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SarishaJ11) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
