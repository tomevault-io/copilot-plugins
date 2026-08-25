## ebay-mcp

> This document provides context for Claude Code when working with this project.

# eBay Listing Automation Plugin - Memory & Context

This document provides context for Claude Code when working with this project.

## Project Identity

**Name**: eBay Listing Automation Plugin
**Type**: Claude Code Plugin
**Purpose**: Automate eBay listing creation, pricing research, fraud detection, and order fulfillment
**Primary Use Case**: Listing clothing, sneakers, and accessories on eBay

## Tech Stack

- **Runtime**: Bun (TypeScript)
- **Integration**: Model Context Protocol (MCP)
- **APIs**: eBay Developer API (Inventory, Browse, Fulfillment, Feedback)
- **Authentication**: OAuth 2.0 with Tailscale Funnel for HTTPS
- **Deployment**: Claude Code Plugin (local)

## Project Structure

```
ebay-listing-automation/
├── .claude-plugin/plugin.json    # Plugin manifest
├── agents/                        # 5 specialized AI agents
├── commands/                      # 10 slash commands
├── hooks/                         # 4 automation hooks
├── scripts/                       # Hook scripts
└── src/                          # MCP server & libs
    ├── mcp-server/               # Main MCP server
    ├── lib/                      # API clients & utilities
    ├── scripts/                  # OAuth authentication
    └── types/                    # TypeScript definitions
```

## Core Components

### 1. MCP Server (21 Tools)
Located in `src/mcp-server/index.ts`

**Inventory Management**:
- `ebay_create_inventory_item` - Create/update inventory
- `ebay_create_offer` - Create draft listing
- `ebay_publish_offer` - Publish to eBay

**Search & Research**:
- `ebay_search_by_image` - Visual similarity search
- `ebay_search_by_keyword` - Text-based search

**Order Fulfillment**:
- `ebay_get_orders` - Fetch order list
- `ebay_get_order` - Get order details
- `ebay_create_shipping_fulfillment` - Add tracking

**Best Offer Management** (Trading API):
- `ebay_get_best_offers` - Get buyer offers on a listing (filter by status)
- `ebay_respond_to_best_offer` - Accept, decline, or counter a buyer Best Offer

**Promoted Listings (Marketing API)**:
- `ebay_create_promotion_campaign` - Create a cost-per-sale ad campaign
- `ebay_get_campaigns` - List campaigns (filter by status)
- `ebay_add_listings_to_campaign` - Add listings with individual bid rates
- `ebay_get_ad_recommendations` - Get eBay-suggested bid rates
- `ebay_pause_campaign` - Pause a running campaign
- `ebay_resume_campaign` - Resume a paused campaign
- `ebay_end_campaign` - Permanently end a campaign

**Fraud Detection**:
- `ebay_get_buyer_info` - Buyer account info
- `ebay_calculate_fraud_risk` - Risk scoring

**Utilities**:
- `ebay_get_policies` - Seller policies
- `ebay_leave_feedback` - Buyer feedback

### 2. AI Agents (5 Specialized)

**Listing Creator** (`agents/listing-creator.md`)
- End-to-end listing workflow from photos to publication
- Invoked when: "List these items", "Create eBay listing"

**Price Researcher** (`agents/price-researcher.md`)
- Market analysis and competitive pricing
- Invoked when: "What should I price this at?", "Research pricing"

**Image Analyzer** (`agents/image-analyzer.md`)
- Product identification and condition assessment
- Invoked when: "What product is this?", "Analyze these photos"

**Fulfillment Manager** (`agents/fulfillment-manager.md`)
- Order processing, shipping, feedback
- Invoked when: "Ship my orders", "Check orders"

**Fraud Analyst** (`agents/fraud-analyst.md`)
- Buyer risk assessment and fraud detection
- Invoked when: "Check this buyer", "Is this fraud?"

### 3. Slash Commands (10 Manual Controls)

Authentication & Setup:
- `/ebay-auth` - OAuth with Tailscale Funnel
- `/ebay-status` - Health check

Listing Workflow:
- `/ebay-analyze [folder]` - Analyze product photos
- `/ebay-draft [folder]` - Create draft listing
- `/ebay-publish [offer-id]` - Publish to eBay

Order Management:
- `/ebay-orders` - View orders
- `/ebay-ship [order-id]` - Process shipping
- `/ebay-feedback [item-id] [tx-id]` - Leave feedback

Utilities:
- `/ebay-policies [type]` - View seller policies
- `/ebay-fraud-analysis [username|order-id]` - Fraud check

## Workflow Patterns

### Complete Listing Workflow
```
1. User organizes photos in folder: items/nike-jordan-1/
2. User: "List these Nike shoes"
3. Listing Creator agent:
   - Analyzes photos (Image Analyzer capabilities)
   - Researches pricing (Price Researcher capabilities)
   - Creates draft listing
   - Asks for approval
4. User: "Publish it"
5. Agent publishes listing
6. Hook: Notifies user to package item
```

### Manual Workflow (Alternative)
```
/ebay-analyze items/nike-jordan-1
/ebay-draft items/nike-jordan-1
/ebay-publish [offer-id]
```

### Order Fulfillment Workflow
```
1. Item sells
2. User: "Ship my orders" or /ebay-orders
3. Fulfillment Manager shows pending orders
4. User provides tracking number
5. Agent marks as shipped via API
6. After delivery: leave feedback automatically
```

### Best Offer Workflow
```
1. Buyer sends Best Offer on listing
2. User: "Check offers on item 12345" or "Respond to the offer"
3. Agent fetches offers via ebay_get_best_offers
4. Agent analyzes offer vs. listing price, buyer feedback
5. Agent recommends: accept, counter (with suggested price), or decline
6. User confirms action
7. Agent responds via ebay_respond_to_best_offer
```

### Promoted Listings Workflow
```
1. User: "Promote my listings" or "Create an ad campaign"
2. Agent creates campaign via ebay_create_promotion_campaign
3. Agent gets bid recommendations via ebay_get_ad_recommendations
4. Agent adds listings with recommended rates via ebay_add_listings_to_campaign
5. User can pause/resume/end campaigns as needed
```

### Fraud Prevention Workflow
```
1. Buyer places high-value bid
2. User: "Check buyer john_doe_123"
3. Fraud Analyst agent:
   - Analyzes account age, feedback, history
   - Calculates risk score (0-100)
   - Provides recommendation
4. User decides: accept, require payment, or cancel
```

## Important Context

### Seller Profile
- **Selling Focus**: Sneakers, men's clothing, accessories
- **Shipping**: Free USPS Ground, fast shipping
- **Returns**: 30-day return policy

### Listing Standards
Always include in listings:
- All 80 characters in title for SEO
- Brand, model, colorway, size, condition
- Complete measurements
- Transparent flaw documentation
- Material/care instructions
- "From smoke-free, pet-free home"
- Free shipping policy
- Authenticity statement for Nike items

### Pricing Strategy
- Research active listings (sold listings not accessible via API)
- Start price: 90% of lowest similar active
- Buy It Now: 110% of highest similar active
- Leave 10-15% room for Best Offers
- Validate manually with "Sell one like this"

### Fraud Prevention Rules
**Block/Require Immediate Payment**:
- Zero feedback buyers
- Account age < 30 days
- Feedback percentage < 95%
- Recent negative feedback
- International high-risk countries (Nigeria, Ghana, Romania, etc.)
- Unpaid item strikes

**Risk Score Interpretation**:
- 0-25: Low risk - proceed normally
- 26-50: Medium risk - caution advised
- 51-75: High risk - require immediate payment
- 76-100: Critical risk - cancel/block

## API Limitations & Workarounds

### Known eBay API Constraints

**1. No Sold Listings Access**
- Finding API decommissioned (Feb 2025)
- Cannot programmatically access completed/sold listings
- **Workaround**: Use active listings for price research + manual validation

**2. Limited Buyer Information**
- No public buyer info API
- **Workaround**: Extract from order/transaction objects

**3. Marketplace Insights API**
- Provides sold listings data
- Requires eBay business approval
- **Future**: Apply for access

**4. Policy Management**
- ✅ Can create/read policies via Account API (`sell.account` scope)
- ❌ **Sandbox limitation**: Business Policies opt-in not available in sandbox
- **Solution**: Must use Production environment for policy creation
- **API Support**: `createFulfillmentPolicy`, `createPaymentPolicy`, `createReturnPolicy`

## Authentication Setup

### OAuth Flow
1. **Environment**: Sandbox (testing) or Production (live)
2. **Method**: OAuth 2.0 Authorization Code Grant
3. **Redirect URI**: Uses Tailscale Funnel for HTTPS
   - Set via `EBAY_REDIRECT_URI` environment variable
   - Port: 5732 (local server, configurable via `EBAY_AUTH_PORT`)
4. **Token Lifespan**:
   - Access token: 2 hours (auto-refresh)
   - Refresh token: 18 months

### Configuration Files
- `.env.local` - Sandbox credentials
- `.env.production.local` - Production credentials
- `.ebay-tokens.json` - Cached access tokens (gitignored)

### Required eBay Setup
1. Create app at developer.ebay.com
2. Add redirect URI in app settings
3. Enable `sell.account` OAuth scope in app settings
4. Run `/ebay-auth` to get refresh token

### Production Environment Setup

**⚠️ Important**: Business policy creation requires **Production environment** due to sandbox limitations.

**Steps to enable production**:

1. **Enable Production Keyset**
   - Go to https://developer.ebay.com/my/keys
   - Find your Production app
   - Click "apply for an exemption" (required for enabled status)
   - Fill out exemption form and submit
   - Wait for approval (1-2 business days)

2. **Configure Production Credentials**
   - Get App ID, Cert ID, and Dev ID from production app
   - Add credentials to `.env.production.local`
   - Add production redirect URI in app settings

3. **Authenticate with Production**
   ```bash
   EBAY_ENV=production bun run auth
   ```

4. **Create Business Policies (Production Only)**
   ```bash
   EBAY_ENV=production bun run scripts/create-default-policies.ts
   ```

**See PRODUCTION_SETUP.md for detailed instructions.**

## Development Commands

```bash
# Install dependencies
bun install

# Authenticate with eBay
bun run auth

# Test API connection
bun run test-connection.ts

# Start MCP server (manual testing)
bun run src/mcp-server/index.ts

# Install as plugin
mv ebay-mcp ~/.claude/plugins/ebay-listing-automation
```

## SEO Optimization Guidelines

### Title Format (80 chars max)
```
[Brand] [Model] [Colorway] [Size] [Condition] [Key Features]
```

Example:
```
Nike Air Jordan 1 High OG Chicago 2015 Size 10 Pre-owned Excellent Red White
```

### Keywords to Include
- Brand name
- Model/style name
- Colorway (if applicable)
- Size
- Condition grade
- Year/edition (OG, Retro, etc.)
- Color descriptors
- Special features (Limited, Deadstock, etc.)

### Item Specifics (Always Complete)
- Brand
- Type
- US Shoe Size / Size
- Model
- Colorway
- Condition
- Style
- Product Line
- Release Year
- Department (Men's, Women's, etc.)

## Error Handling

### Common Issues & Solutions

**"Invalid refresh token"**
- Token expired or wrong environment
- Solution: Run `/ebay-auth` to get new token

**"Missing seller policies"**
- Policies not configured
- Solution: Create policies at ebay.com/sh/policies

**"Category ID required"**
- No category specified
- Solution: Ask Claude to suggest based on item type

**"401 Unauthorized"**
- Access token expired
- Solution: Auto-refresh (should happen automatically)

**"403 Forbidden"**
- Missing scopes or permissions
- Solution: Re-authenticate with correct scopes

## Best Practices

### When Listing Items
1. Take 8-12 clear photos per item
2. Include measurements with ruler/tape
3. Document all flaws transparently
4. Use natural lighting
5. Organize photos by item in folders
6. Name folders descriptively: `brand-model-colorway`

### When Researching Prices
1. Search visually similar items first
2. Filter by condition
3. Check multiple sources
4. Account for seasonality
5. Leave negotiation room
6. Validate with manual "Sell one like this"

### When Shipping
1. Package immediately after sale
2. Label box with item number
3. Don't seal until label purchased
4. Ship within 1 business day
5. Always use tracking
6. Signature required for $750+

### When Detecting Fraud
1. Check all new buyers for high-value items
2. Block zero-feedback international buyers
3. Require immediate payment for risky transactions
4. Document all fraud concerns
5. Trust your instincts
6. Follow eBay's cancellation policies

## Plugin Integration

### MCP Server Configuration
Automatically configured in `plugin.json`:
```json
{
  "mcpServers": {
    "ebay": {
      "command": "bun",
      "args": ["run", "${CLAUDE_PLUGIN_ROOT}/src/mcp-server/index.ts"],
      "env": {
        "EBAY_ENV": "${EBAY_ENV:-sandbox}"
      }
    }
  }
}
```

### Hook Events
- **SessionStart**: Check connection on startup
- **PostToolUse**: Log activity, notify on publish
- **UserPromptSubmit**: Suggest workflows

### Agent Invocation
Agents invoked automatically by Claude based on context:
- Natural language triggers
- Slash command execution
- Manual selection via `/agents`

## Testing Checklist

Before going live:
- [ ] Sandbox authentication successful
- [ ] Test API connection verified
- [ ] Create test listing in sandbox
- [ ] Test image analysis
- [ ] Test pricing research
- [ ] Test fraud detection
- [ ] Create seller policies
- [ ] Get production credentials
- [ ] Authenticate production environment
- [ ] Test production listing

## Important Reminders

### Always Remember
- 🔒 Never commit credentials to git
- 📝 Document all flaws transparently in listings
- 📦 Package items immediately after sale
- 🏷️ Label boxes with item numbers
- ⚖️ Use all 80 characters in titles for SEO
- 🔍 Check high-risk buyers before accepting bids
- 📸 Take quality photos with good lighting
- 💰 Leave room for negotiation in pricing
- ⭐ Leave feedback after delivery confirmation

### Never Do
- ❌ List items without photos
- ❌ Lie about condition or flaws
- ❌ Ship without tracking
- ❌ Accept high-value bids from zero-feedback buyers
- ❌ Skip fraud analysis on suspicious accounts
- ❌ Seal boxes before purchasing labels
- ❌ Use all caps in titles (eBay penalty)
- ❌ Ignore buyer messages

## Future Enhancements

### Planned Features
1. Marketplace Insights API integration (pending approval)
2. Cloudflare R2 for image hosting
3. Bulk image upload to eBay Picture Services
4. Automated repricing based on market changes
5. Sales analytics dashboard
6. Template library for common items
7. AI-powered condition grading
8. Automated Best Offer management

### Community Contributions Welcome
- Additional agents for specialized tasks
- More automation hooks
- Enhanced SEO optimization algorithms
- Cloudflare Workers for webhooks
- Mobile app integration

## Support & Resources

### Documentation
- README.md - Complete user guide
- QUICKSTART.md - 5-minute setup
- PLUGIN.md - Architecture details
- IMPLEMENTATION_STATUS.md - Build status

### External Links
- eBay Developer: https://developer.ebay.com
- eBay Seller Hub: https://www.ebay.com/sh
- Seller Policies: https://www.ebay.com/sh/policies
- MCP Protocol: https://modelcontextprotocol.io
- Tailscale Funnel: https://tailscale.com/kb/1223/funnel

### Community
- GitHub: https://github.com/jbwashington/ebay-mcp
- Issues: https://github.com/jbwashington/ebay-mcp/issues

> **Note**: Contributors should update personal details (username, redirect URIs) via environment variables, not in source code.

---

**Last Updated**: January 11, 2025
**Version**: 1.0.0
**Status**: Production Ready (pending user authentication setup)

---
> Source: [jbwashington/ebay-mcp](https://github.com/jbwashington/ebay-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
