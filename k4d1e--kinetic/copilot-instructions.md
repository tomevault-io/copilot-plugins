## kinetic

> - **Protocol Key**: `meta_surgeon_protocol`

# Meta Surgeon Protocol - Sprint Plan Action Card Definition

## Protocol Identity
- **Protocol Key**: `meta_surgeon_protocol`
- **Mission Title**: Meta Surgeon Protocol
- **Display Name**: Meta Surgeon Protocol
- **Protocol Type**: schema-based
- **Entity Label**: Entity Signal Strength
- **Total Steps**: 4
- **Sprint Circle Index**: 0

## Description
Establish entity signal through structured data implementation. Force search engines to recognize the business as a verified local entity by injecting 4 "Truth Layers" (Organization schema, GeoCircle definitions, Service/Product catalog, and Review aggregation).

## Page 1: Strategist Insight

### Insight Message
"Before we fight for keywords, we must introduce you to the algorithm. Right now, Google is guessing who you are. By injecting these 4 'Truth Layers' into your code, we force the search engines to recognize {{COMPANY_NAME}} as a verified local entity, not just another website."

### Dynamic Placeholders
- `{{COMPANY_NAME}}`: Replaced with `window.currentPropertyName` or property domain name
- `{{PROPERTY_NAME}}`: Same as COMPANY_NAME for this protocol

## Step 1: Global Identity

### Step Definition
```yaml
title: "Global Identity"
description: "First, we hard-code your brand's DNA (Logo, Phone, Social Profiles) into the pages of your site."
```

### Execution Instructions
```yaml
concept: "global brand identity elements"
action: "Add organization schema markup with logo, contact information, and social media profiles"
schemaType: "Organization schema (schema.org/Organization)"
implementation: "Inject structured data into all pages to establish brand identity in search engines"
deliverable: "global-identity-plan.md"
```

### Cursor Prompt Generation Template
```
You are implementing Global Identity as part of Meta Surgeon Protocol.

OBJECTIVE: Add organization schema markup with logo, contact information, and social media profiles

IMPLEMENTATION FOCUS:
Inject structured data into all pages to establish brand identity in search engines

SCHEMA TYPE: Organization schema (schema.org/Organization)

INSTRUCTIONS:
1. Analyze the current website structure in this workspace
   - Detect the technology stack (HTML, React, Vue, Next.js, static site, etc.)
   - Identify all pages/components where global brand identity elements should be implemented
   - Determine the best location for schema markup injection

2. Create a comprehensive implementation plan that includes:
   - Current state analysis: What structure currently exists?
   - Files that need modification: List specific files and their paths
   - Code additions required: Outline the schema markup structure
   - Implementation approach: How to integrate with existing code
   - Dependencies and order: What needs to be done first?
   - Testing strategy: How to verify the implementation works

3. Adapt to the detected technology:
   - For static HTML: Add JSON-LD script tags to <head> or before </body>
   - For React/Vue/Next: Create reusable schema components or use head management
   - For template engines: Inject schema through layout templates
   - For CMSs: Provide plugin recommendations or custom code injection

CONTEXT & REQUIREMENTS:
- Work with ANY file structure and technology stack
- Use schema.org vocabulary for maximum compatibility
- Ensure JSON-LD format for easy implementation
- Make schema dynamic (pull from site data, not hardcoded)
- Follow Google's Structured Data guidelines
- Include: Organization name, logo, contact info (phone, email), social media profiles
- Ensure mobile responsiveness and accessibility
- Validate schema markup can be tested with Google's Rich Results Test

DELIVERABLE:
Create a detailed implementation plan saved as: global-identity-plan.md

The plan should be actionable, technology-agnostic, and ready for immediate implementation regardless of the website's industry (e-commerce, local service, SaaS, restaurant, etc.).
```

## Step 2: Territory Claim

### Step Definition
```yaml
title: "Territory Claim"
description: "Next, we draw the digital borders. We will define the exact 'GEO Circle' for each city you service so Google knows exactly where your trucks go."
```

### Execution Instructions
```yaml
concept: "geographic service area definitions"
action: "Define service areas using geographic schema markup"
schemaType: "GeoCircle or ServiceArea schema (schema.org/GeoCircle)"
implementation: "Specify exact locations where products/services are offered using latitude, longitude, and radius"
deliverable: "territory-claim-plan.md"
```

### Key Implementation Details
- Use GeoCircle schema to define service radius
- Include latitude/longitude coordinates for service center
- Specify radius in kilometers or miles
- Can have multiple service areas for different locations
- Nest within Organization or LocalBusiness schema

## Step 3: Commercial Definition

### Step Definition
```yaml
title: "Commercial Definition"
description: "Now, we define the product. We explicitly tell the bot that 'Roofing' is a Service you sell, with a specific price range, turning your page from a 'brochure' to a 'catalog'."
```

### Execution Instructions
```yaml
concept: "structured product/service catalog with pricing"
action: "Transform content pages into catalog entries with Service/Product schema"
schemaType: "Service or Product schema (schema.org/Service, schema.org/Product)"
implementation: "Add structured data defining offerings with descriptions, pricing, and categories"
deliverable: "commercial-definition-plan.md"
```

### Key Implementation Details
- Use Service schema for service-based businesses (roofing, plumbing, consulting)
- Use Product schema for e-commerce or physical products
- Include: name, description, provider, serviceType, areaServed
- Add pricing with PriceSpecification or offers
- Include category/serviceType for classification

## Step 4: Reputation Sync

### Step Definition
```yaml
title: "Reputation Sync"
description: "Finally, we aggregate your trust. Let's take your scattered 5-star reviews and format them into a 'CollectionPage' that search engines can count."
```

### Execution Instructions
```yaml
concept: "review aggregation and testimonial markup"
action: "Aggregate customer reviews into structured CollectionPage format"
schemaType: "Review and AggregateRating schema (schema.org/Review)"
implementation: "Format testimonials as structured review data that search engines can index and display"
deliverable: "reputation-sync-plan.md"
```

### Key Implementation Details
- Use Review schema for individual testimonials
- Use AggregateRating to show overall rating statistics
- Include: reviewRating (1-5 stars), author, datePublished, reviewBody
- For aggregate: ratingValue, reviewCount, bestRating (5), worstRating (1)
- Can pull from Google My Business, Yelp, or on-site testimonials

## Page 6: Completion Status

### Status Lines
```yaml
scanning: "Status: Scanning Source Code"
established: "Status: Entity Signal Established."
success: "Success: Your business is now a verified entity. 4 Schema Packs Active."
```

### Animation Sequence
1. Line 1 appears with blinking dots (scanning state)
2. After 2 seconds, Line 2 appears (established state)
3. After 2 more seconds, Line 3 appears (success state)
4. Trigger `sprintCardAnimationComplete` event
5. Unlock next sprint circle

## Button State Logic

### Execution Assist Button
- **Initial State**: Visible on all step pages (2-5)
- **Purpose**: Open modal with Cursor prompt
- **Action**: User clicks → Modal opens with generated prompt

### Next Step Button
- **Initial State**: Disabled on all step pages (2-5)
- **Enable Trigger**: User copies prompt from Execution Assist modal
- **Action**: Advance to next page when clicked

### Complete Button (Page 5 only)
- **Initial State**: Disabled
- **Enable Trigger**: User copies Step 4 prompt
- **Action**: Navigate to Page 6, save completion to database, unlock next circle

## Protocol Characteristics

### No E.V.O. Analysis Required
This protocol does NOT use E.V.O. dimension analysis. It is a manual implementation protocol where:
- User reads step description
- User opens Execution Assist to get Cursor prompt
- User copies prompt and executes in Cursor
- User marks step complete by advancing

### Analysis Button Visibility
```javascript
// In sprintPlan.js, Analysis buttons are hidden by default for this protocol
if (protocolType === 'meta_surgeon_protocol') {
  analysisButtons.forEach(btn => btn.style.display = 'none');
  executionAssistButtons.forEach(btn => btn.style.display = 'flex');
}
```

## Database Schema

### sprint_action_cards Table Entry
```sql
INSERT INTO sprint_action_cards (card_type, display_name, total_steps, description)
VALUES (
  'meta_surgeon_protocol',
  'Meta Surgeon Protocol',
  4,
  'Establish entity signal through structured data: Organization, GeoCircle, Service/Product, and Review schema'
);
```

### Completion Tracking
When user clicks Complete button, save to `sprint_card_completions`:
```javascript
{
  cardType: 'meta_surgeon_protocol',
  propertyId: currentPropertyId,
  sprintIndex: 0,
  startedAt: ISO_timestamp,
  completedAt: ISO_timestamp,
  duration: milliseconds,
  progressPercentage: 95,
  steps: [
    { stepNumber: 1, name: "Global Identity", completedAt: timestamp },
    { stepNumber: 2, name: "Territory Claim", completedAt: timestamp },
    { stepNumber: 3, name: "Commercial Definition", completedAt: timestamp },
    { stepNumber: 4, name: "Reputation Sync", completedAt: timestamp }
  ]
}
```

## Prompt Generation Logic

### Entry Point
```javascript
// executionAssist.js - generatePrompt()
const isSchemaProtocol = executionInstructions.schemaType !== undefined;

if (isSchemaProtocol) {
  return this.generateSchemaPrompt(context, fileName);
}
```

### Schema Prompt Template Variables
- `${mission}` - "Meta Surgeon Protocol"
- `${stepName}` - Current step title
- `${executionInstructions.action}` - Specific action
- `${executionInstructions.implementation}` - Implementation focus
- `${executionInstructions.schemaType}` - Schema.org type
- `${executionInstructions.concept}` - High-level concept
- `${fileName}` - Deliverable filename

## Future E.V.O. Integration

When E.V.O. generates this card procedurally, it should:

1. **Detect Trigger Condition**: 
   - No Organization schema detected on site
   - Missing local business markup
   - Low entity recognition signals

2. **Generate Dynamic Insight**:
   - Analyze current schema implementation state
   - Identify which "Truth Layers" are missing
   - Customize insight message with specific gaps

3. **Customize Steps Based on Tech Stack**:
   - Detect site technology (React, WordPress, static HTML)
   - Adjust deliverable instructions for detected stack
   - Provide tech-specific implementation guidance

4. **Validate Completion**:
   - Use Google Rich Results Test API to verify schema
   - Check for valid Organization schema on homepage
   - Confirm all 4 schema types are properly implemented

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/k4d1e) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
