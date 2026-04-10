## tutech

> Deep dive into how the two AI agents work.

# Agent Architecture & Logic Documentation

Deep dive into how the two AI agents work.

---

## 🥷 PRICE NINJA AGENT

### Purpose
Monitor competitor prices in real-time and alert administrators when prices fall significantly below ours (15%+ difference). This enables quick decision-making on pricing strategy.

### Location
`backend/price_ninja_agent.ts`

### How It Works

#### 1. **Data Collection**
```typescript
// Fetch all products from database
const products = await supabase
  .from('products')
  .select('id, name, category, price')

// Fetch competitor data (mock)
const competitors = await supabase
  .from('competitor_products')
  .select('*')
```

**Data Structure:**
- **Our Products:** Indexed by category + product name
- **Competitor Products:** Mock data stored in database with competitor name, category, price, URL

#### 2. **Price Comparison Logic**

```typescript
for (const product of products) {
  // Find similar items from competitors
  const competitorMatches = competitors.filter(
    cp => cp.category === product.category &&
          isSimilarProduct(product.name, cp.product_name)
  )

  // Compare prices
  for (const competitor of competitorMatches) {
    const priceDifference = product.price - competitor.price
    const percentageDifference = (priceDifference / product.price) * 100

    // Check threshold
    if (percentageDifference >= 15) {
      // Generate alert
      alerts.push({...})
    }
  }
}
```

**Key Function: `isSimilarProduct()`**
- Normalizes product names (lowercase, trim spaces)
- Checks if one name contains the other
- Counts common keywords (minimum 2)
- Handles variations like "T-Shirt" vs "Tshirt"

#### 3. **Alert Generation**

**Alert Object:**
```typescript
{
  product_id: "uuid",
  product_name: "Blue Denim Jeans",
  our_price: 129.99,           // in DT
  competitor_name: "StyleWear",
  competitor_price: 109.99,    // in DT
  price_difference: 20.00,     // absolute
  percentage_difference: 15.4  // percentage
}
```

#### 4. **Admin Notification**

**Email Format:**
- HTML table showing all price alerts
- Columns: Product | Our Price | Competitor | Competitor Price | Difference %
- Recipient: Admin email (from users table where role = 'admin')
- Subject: "🥷 Price Ninja Alert Report"

#### 5. **Logging**

**Log Entry:**
```typescript
{
  agent_name: "price_ninja",
  action_type: "price_alert" | "email_sent" | "comparison_run",
  user_id: null,
  product_id: "uuid",
  message: "price_ninja: email_sent",
  metadata: {
    competitor: "StyleWear",
    price_difference: 20.00,
    percentage_difference: 15.4
  },
  created_at: timestamp
}
```

### Configuration

**Dynamic Parameters (from agent_config table):**
```sql
SELECT * FROM agent_config WHERE agent_name = 'price_ninja';

-- price_difference_threshold: 15 (%)
-- Can be adjusted per category:
--   t-shirt: 15%
--   jeans: 15%
--   jacket: 20% (higher threshold)
```

### Execution Flow

```
START
  │
  ├─> Load all products
  ├─> Load competitor data
  │
  ├─> For each product:
  │   ├─> Find similar competitor products
  │   ├─> Compare prices
  │   └─> If difference >= threshold:
  │       └─> Add to alerts
  │
  ├─> Log "comparison_run"
  │
  ├─> If alerts > 0:
  │   ├─> Get admin email
  │   ├─> Generate email HTML
  │   ├─> Send email
  │   └─> Log "email_sent" (for each alert)
  │
  └─> Return results
    STATUS: success/error
    COUNT: alerts generated
END
```

### Example Execution

**Input:**
```
Products in DB: 6 items
Competitors in DB: 5 items
Threshold: 15%
```

**Processing:**
```
Product: "Classic White T-Shirt" (45.99 DT)
  ├─> FashionHub: "Classic White T-Shirt" (39.99 DT)
  │   └─> Difference: 13.0% ❌ Below threshold
  └─> No alert

Product: "Blue Denim Jeans" (129.99 DT)
  ├─> StyleWear: "Blue Denim Jeans" (109.99 DT)
  │   └─> Difference: 15.4% ✅ Alert generated!
  └─> Alert #1 created

Product: "Leather Jacket" (349.99 DT)
  ├─> UrbanClothing: "Leather Jacket" (289.99 DT)
  │   └─> Difference: 17.1% ✅ Alert generated!
  └─> Alert #2 created
```

**Output:**
```json
{
  "success": true,
  "alerts_count": 2,
  "alerts": [
    {
      "product_name": "Blue Denim Jeans",
      "our_price": 129.99,
      "competitor_name": "StyleWear",
      "competitor_price": 109.99,
      "percentage_difference": 15.4
    },
    {
      "product_name": "Leather Jacket",
      "our_price": 349.99,
      "competitor_name": "UrbanClothing",
      "competitor_price": 289.99,
      "percentage_difference": 17.1
    }
  ]
}
```

### Customization

**Add New Competitor:**
```sql
INSERT INTO competitor_products 
  (competitor_name, product_name, category, price, url) 
VALUES 
  ('NewStore', 'Blue Jeans', 'jeans', 119.99, 'https://newstore.tn/...');
```

**Adjust Threshold:**
```sql
UPDATE agent_config 
SET config_value = '20' 
WHERE agent_name = 'price_ninja' 
AND config_key = 'price_difference_threshold';
```

---

## 🔄 CLIENT RECOVERY AGENT

### Purpose
Automatically identify customers who haven't engaged with the platform recently and send personalized, friendly re-engagement messages with discount offers to encourage their return.

### Location
`backend/client_recovery_agent.ts`

### How It Works

#### 1. **User Inactivity Detection**

**Definition of Inactive:**
- Last login > 30 days ago (configurable)
- OR no orders in last 30 days
- AND role = 'customer'

```typescript
// Query inactive users
const inactiveUsers = await findInactiveUsers(inactivityDays: 30)

// Returns users with:
// - id, email, full_name
// - last_login timestamp
// - days_inactive (calculated)
// - total_spent (lifetime value)
```

#### 2. **Message Personalization**

**Three Message Templates** (randomly selected):

**Template 1 - Friendly Casual:**
```
Hi [Name],

We noticed you haven't visited us in a while, and we miss you! 👋

It's been [X] days since we last saw you, and we have some 
amazing new arrivals in our clothing collection.

Use code: {{DISCOUNT_CODE}} for 10% OFF!

Come check out what's new!
```

**Template 2 - Upbeat:**
```
Hey [Name]! 

Where have you been? We haven't seen you in {{DAYS_INACTIVE}} days! 😢

We just restocked some of your favorite categories...

Use code: {{DISCOUNT_CODE}} for 10% OFF!
```

**Template 3 - Casual & Personal:**
```
[Name], it's been a minute! ⏰

We've been busy adding new styles...

Special gift for you: {{DISCOUNT_CODE}} for 10% OFF
```

**Key Features:**
- Personalized with customer name
- References inactivity period (creates urgency)
- Mentions categories customer browsed before
- Includes discount code
- Friendly, not corporate tone

#### 3. **Discount Code Assignment**

```typescript
// Get active discount code
const discountCode = await getDiscountCode(discountPercent: 10)

// Returns: { code: 'COMEBACK10', discount_percent: 10 }

// If no code exists, create one:
INSERT INTO discount_codes 
  (code, discount_percent, valid_from, valid_until, max_uses) 
VALUES 
  ('COMEBACK10', 10, NOW(), NOW() + INTERVAL '30 days', -1)
```

**Discount Code Table:**
```typescript
{
  id: uuid,
  code: 'COMEBACK10',
  discount_percent: 10,
  valid_from: timestamp,
  valid_until: timestamp,
  max_uses: -1 (unlimited),
  used_count: 0
}
```

#### 4. **Email Sending**

**Email Format:**
```html
<div style="font-family: Arial; max-width: 600px;">
  <h1>Welcome Back, [Name]! 🎉</h1>
  
  <div class="message">
    [Personalized message content]
  </div>
  
  <p style="text-align: center;">
    TUTECH - Your Favorite Clothing Store
  </p>
</div>
```

#### 5. **Logging**

**Log Entries Created:**

```typescript
// For each message sent:
{
  agent_name: "client_recovery",
  action_type: "message_sent",
  user_id: "customer-uuid",
  product_id: null,
  message: "client_recovery: message_sent",
  metadata: {
    email: "customer@example.com",
    discount_code: "COMEBACK10",
    days_inactive: 35,
    total_spent: 259.97  // Lifetime value
  },
  created_at: timestamp
}

// For overall scan completion:
{
  agent_name: "client_recovery",
  action_type: "scan_complete",
  user_id: null,
  product_id: null,
  message: "client_recovery: scan_complete",
  metadata: {
    inactive_users_found: 5,
    messages_sent: 5,
    discount_code: "COMEBACK10",
    inactivity_days: 30,
    timestamp: timestamp
  },
  created_at: timestamp
}
```

### Configuration

**Dynamic Parameters (from agent_config table):**
```sql
-- Inactivity threshold
SELECT * FROM agent_config 
WHERE agent_name = 'client_recovery' AND config_key = 'inactivity_days';
-- Returns: '30' (days)

-- Discount percentage
SELECT * FROM agent_config 
WHERE agent_name = 'client_recovery' AND config_key = 'discount_percent';
-- Returns: '10' (%)
```

### Execution Flow

```
START
  │
  ├─> Get inactivity threshold (30 days)
  ├─> Get discount percentage (10%)
  │
  ├─> Find inactive users:
  │   ├─> Query users with last_login < 30 days ago
  │   ├─> Join with orders to get spending info
  │   └─> Exclude admin users
  │
  ├─> If no inactive users:
  │   ├─> Log "scan_complete"
  │   └─> Return 0 messages sent
  │
  ├─> For each inactive user:
  │   ├─> Select random message template
  │   ├─> Personalize with name & days inactive
  │   ├─> Get discount code
  │   ├─> Format email HTML
  │   ├─> Send email
  │   └─> Log "message_sent"
  │
  ├─> Log overall "scan_complete"
  │
  └─> Return results
    STATUS: success/error
    COUNT: messages sent
END
```

### Example Execution

**Scenario:**
- Inactivity threshold: 30 days
- Current date: 2024-02-05

**Finding Inactive Users:**
```sql
SELECT 
  u.id, u.email, u.full_name,
  u.last_login,
  COUNT(o.id) as total_orders,
  SUM(o.total_price) as total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.role = 'customer'
  AND u.last_login < '2024-01-06'
GROUP BY u.id;

-- Results:
-- Email: customer1@example.com
--   Last login: 2023-12-01 (65 days ago) ✅ Inactive
--   Total orders: 2
--   Total spent: 259.97 DT
--
-- Email: customer2@example.com
--   Last login: 2024-01-20 (16 days ago) ❌ Active
```

**Message Generation for Inactive User:**
```
Template selected: Template #2 (random)

Message:
"Hey Ahmed Ben Ali! 

Where have you been? We haven't seen you in 65 days! 😢

We just restocked some of your favorite categories...

Use code: COMEBACK10 for 10% OFF your next purchase!

Valid for 30 days.

Your TUTECH Friends ✨"

Discount Code: COMEBACK10 (10% off, expires in 30 days)
```

**Output:**
```json
{
  "success": true,
  "message": "Sent recovery messages to 5 inactive users",
  "count": 5,
  "messages": [
    {
      "user_id": "uuid-1",
      "email": "customer1@example.com",
      "full_name": "Ahmed Ben Ali",
      "discount_code": "COMEBACK10",
      "discount_percent": 10
    },
    // ... 4 more
  ]
}
```

### Customization

**Adjust Inactivity Threshold:**
```sql
-- Change to 14 days instead of 30
UPDATE agent_config 
SET config_value = '14' 
WHERE agent_name = 'client_recovery' 
AND config_key = 'inactivity_days';
```

**Add New Message Template:**
```typescript
// In client_recovery_agent.ts, add to RECOVERY_MESSAGES array:
(name: string, daysInactive: number) => `
Your custom message template here...
Use code: {{DISCOUNT_CODE}} for {{DISCOUNT_PERCENT}}% off!
`
```

**Create Custom Discount Code:**
```sql
INSERT INTO discount_codes 
  (code, discount_percent, valid_from, valid_until, max_uses) 
VALUES 
  ('WELCOME20', 20, NOW(), NOW() + INTERVAL '60 days', 100);
```

---

## 📊 Comparison & Differences

| Aspect | Price Ninja | Client Recovery |
|--------|------------|-----------------|
| **Goal** | Price monitoring | Customer engagement |
| **Trigger** | Manual / Scheduled | Manual / Scheduled |
| **Data Source** | Products + Competitors | Users + Orders |
| **Output** | Email alerts to admin | Email offers to customers |
| **Decision Made** | Should we adjust prices? | Re-engage or not? |
| **Frequency** | Daily recommended | Weekly recommended |
| **Revenue Impact** | Defensive (protect margins) | Offensive (recover customers) |

---

## 🔌 Integration Points

### Triggering Agents from Frontend

**Method 1: Manual Button Click (Current)**
```typescript
// In AdminDashboard.tsx
async function triggerAgent(agentType: 'price_ninja' | 'client_recovery') {
  const response = await supabase.functions.invoke(`${agentType}-agent`, {
    method: 'POST'
  })
  return response
}
```

**Method 2: Scheduled Cron Job**
```bash
# GitHub Actions or cron-job.org
POST https://[project].supabase.co/functions/v1/price_ninja_agent
Headers: Authorization: Bearer [KEY]
```

**Method 3: Webhook Trigger**
```typescript
// From external system
fetch('https://[project].supabase.co/functions/v1/price_ninja_agent', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${ANON_KEY}` }
})
```

---

## 🚀 Future Enhancements

### Price Ninja v2
- [ ] AI-powered price optimization recommendations
- [ ] Competitor tracking with historical data
- [ ] Dynamic pricing based on demand
- [ ] Auto-adjust prices with admin approval

### Client Recovery v2
- [ ] ML segmentation (VIP vs. at-risk users)
- [ ] Personalized discount amounts based on lifetime value
- [ ] A/B testing different message templates
- [ ] Track conversion rates per message type

---

## 📈 Metrics & Analytics

### Track These KPIs

**Price Ninja:**
- Alerts generated per run
- Average price difference detected
- Avg response time to price changes
- Admin actions taken on alerts

**Client Recovery:**
- Inactive users identified
- Emails sent
- Click-through rate
- Conversion rate (users who return)
- Revenue recovered

---

**Document Version:** 1.0  
**Last Updated:** 2024-02-05  
**Status:** Production Ready

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ahmedalamoussa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ahmedalamoussa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
