## nexus

> Master outbound workflow for ALL NEXUS-generated documents — emails, capability statements, RFQs, quotes, proposals. EVERYTHING goes through this pipeline. No exceptions.


# NEXUS OUTBOUND WORKFLOW — THE MASTER PIPELINE

**This is the single source of truth for how NEXUS generates ANY outbound document. Every email, every capability statement, every RFQ, every quote, every proposal goes through this pipeline. No skipping steps. No shortcuts. No asking Dee to explain it again.**

---

## THE RULE

**Before ANY document leaves NEXUS — email, cap statement, RFQ, quote, or proposal — it MUST pass through this pipeline:**

1. **CLASSIFY** the document type and recipient
2. **BUILD** the document with the correct framework
3. **APPLY** ProposalBio to the content
4. **VERIFY** against protection rules
5. **PLACE** in the correct SEND_TO folder
6. **PRESENT** to Dee as a complete package

**If you skip a step, you're wrong. Go back and do it right.**

---

## STEP 1: CLASSIFY — What Are We Sending and To Whom?

### Document Types

| Type | Recipient | Framework | Protection Rules |
|------|-----------|-----------|-----------------|
| **Quote Response** | Government CO / Procurement Officer | `QUOTE_RESPONSE_TEMPLATE.html` + ProposalBio + Sector Colors | All 10 Biohacks, company-info-verification |
| **Buyer Email** | Government CO / Procurement Officer | ProposalBio + Human Touch | Name Recognition (#7), Familiarity (#6) |
| **Capability Statement** | Government CO / Procurement Officer | ProposalBio + Sector Colors + NEXUS Template | All 10 Biohacks |
| **RFQ to Supplier** | Vendor / Manufacturer | Supplier Protection | NEVER reveal buyer (see `never-reveal-buyer-to-supplier.mdc`) |
| **Quote to Buyer** | Government CO / Procurement Officer | ProposalBio + Pricing | Familiarity (#6), Name Recognition (#7) |
| **Proposal** | Government CO / Procurement Officer | Full ProposalBio (all 10) | Must pass gate (composite >= 75) |
| **Subcontractor Outreach** | Subcontractor | Supplier-style Protection | NEVER reveal buyer |
| **Partnership Email** | Potential Partner (Lakota, Kojak, etc.) | Human Touch | DDI capabilities focus |
| **Follow-up Email** | Any previous contact | Human Touch + context | Match original tone |

### Recipient Classification

| Recipient Type | Tone | ProposalBio Required? | Protection Level |
|---------------|------|----------------------|-----------------|
| **Buyer / CO / Procurement** | Human Touch + Professional | YES — all applicable biohacks | Maximize Name Recognition (#7) |
| **Supplier / Vendor** | Business-only, transactional | NO | Maximum — never reveal buyer |
| **Subcontractor** | Business, moderate warmth | NO | High — never reveal buyer |
| **Partner** | Human Touch, relationship | Lite (tone matching) | Low — DDI info is fine |

---

## STEP 2: BUILD — Construct the Document

### For BUYER EMAILS

**Required elements (every single time):**

1. **Subject line** — Include "EDWOSB" + service type + solicitation number if known
   - Format: `Michigan-Based EDWOSB — [Service], [Solicitation Reference]`
   - Or for presolicitation: `EDWOSB Interest — [Service] ([Solicitation Number])`

2. **Opening** — Personal greeting if CO name is known, otherwise "Hi,"

3. **First paragraph** — State the solicitation/opportunity + DDI's EDWOSB status + Michigan-based
   - Mirror the buyer's language from the RFP/solicitation (Biohack #6)
   - Use the agency name 3+ times across the email (Biohack #7)

4. **Capabilities list** — Bulleted, mirrors RFP scope language exactly
   - Each bullet = a "yes we can do that" (Biohack #5 Yes Stacking)
   - Use the buyer's exact terminology, not DDI's

5. **Smart questions** — 3-4 questions showing you did your homework (Biohack #4 Reciprocity)

6. **Closing** — Mention attached capability statement + warm sign-off

7. **Signature block** — ALWAYS:
   ```
   Dee Davis
   President & CEO
   Dee Davis Inc
   755 W. Big Beaver Rd., Suite 2020, Troy, MI 48084
   (248) 376-4550 | info@deedavis.biz
   EDWOSB | WOSB | WBENC | MBE | SBE | SWFT Authorized
   ```

### For CAPABILITY STATEMENTS

**Required elements (every single time):**

1. **Color scheme** — Match the sector per `cap-statement-colors.mdc`:
   - Medical / Healthcare = Blue + Red
   - Defense / DLA = Navy + Gold
   - Facilities / Grounds = Forest Green + Slate
   - Industrial / Construction = Charcoal + Orange
   - Transportation / Logistics = Deep Purple + Teal
   - Reseller / Wholesale = Steel Blue + Emerald
   - Personal Items = Rose/Pink

2. **Header** — Company name, CAGE (8UMX3), UEI (HJB4KNYJVGZ1), DUNS (002636755), SAM Active, all cert badges. **LOGO MUST be the actual base64-embedded PNG** — extract from any existing working cap statement in `BIDS:RESOURCES/` (e.g., VA DRY ICE DELIVERY). NEVER use a text placeholder like "DDI" in a div. The logo is always an `<img>` tag with `src="data:image/png;base64,..."` and `class="logo-img"`.

3. **Title bar** — "CAPABILITY STATEMENT — [Service Description Mirroring Buyer Language]"
   - Subtitle: "Prepared for [Agency Name] — [Division] | [Solicitation Number] | [Month Year]"

4. **EDWOSB advantage box** — Always present, always prominent

5. **Company overview** — Tailored to THIS specific opportunity
   - Use buyer's mission/vision if known (Biohack #7)
   - Reference their specific needs from the solicitation (Biohack #6)
   - Midwest regional language if Michigan (Biohack #1)

6. **Core competencies** — 4 boxes, each mirrors a scope area from the solicitation

7. **Service capabilities** — Bulleted list, mirrors RFP language

8. **Why Choose DDI** — Tailored differentiators for this specific buyer

9. **NAICS codes** — Relevant to THIS solicitation only

10. **Certifications** — Full list: EDWOSB, WOSB, WBENC, MBE, SBE, E-Verify, SWFT

11. **Contact grid** — Per `company-info-verification.mdc` (248.376.4550, info@deedavis.biz, Troy MI 48084)

12. **Footer** — Certs + "Prepared for [Agency] — [Solicitation] | [Date]"

### For SUPPLIER RFQs

**Required elements:**
1. DDI-YYYY-### number (NEVER buyer's solicitation number)
2. Generic title (NEVER buyer name in title)
3. Generic client reference ("Michigan municipal client")
4. Generic location (county level max, never specific address)
5. Supplier deadline = government deadline MINUS 3-5 days
6. Per `quote-generator-supplier-protection.mdc` and `rfq-buyer-protection-checklist.mdc`

### For SUBCONTRACTOR OUTREACH

**Required elements:**
1. Same buyer protection as suppliers — never reveal the end client
2. Scope of work description (generic)
3. DDI's role as prime contractor
4. Service area (geographic, generic)

---

## STEP 3: APPLY PROPOSALBIO — Score the Content

**Every buyer-facing document gets scored against the 10 ProposalBio biohacks.**

### The 10 Biohacks (from `proposalbio_module.py`)

| # | Biohack | What It Does | How to Apply |
|---|---------|-------------|-------------|
| 1 | **Mirror Neuron** | Match buyer's cultural tone | Federal = formal. State = moderate. Local = conversational. Midwest = "reliable, proven, track record" |
| 2 | **Cognitive Ease** | Simple, clear writing | 6-8th grade reading level. <=12 words/sentence. <=10 sentences/paragraph. 40%+ white space |
| 3 | **Story Arc** | Challenge → Solution → Result | Include at least 1 story arc per page. DDI as guide, not hero |
| 4 | **Reciprocity** | Give value before asking | Smart questions, insights, benchmarks. Show you did homework |
| 5 | **Yes Stacking** | Build agreement before the ask | Capabilities list mirrors RFP scope = mental "yes" on each bullet |
| 6 | **Familiarity** | Echo their language back | Use THEIR terminology from the RFP, not ours. Mirror 50%+ of key terms |
| 7 | **Name Recognition** | Say their name often | Agency name 3+ times. Reference their mission/vision. No "your organization" |
| 8 | **Sensory Language** | Concrete, not vague | Specific numbers, locations, regulations. No "quality," "excellent," "proven" without evidence |
| 9 | **Rhythm** | Sentence variety | Mix short (6 words) and long (20 words). Cadence breaks. No monotone sections |
| 10 | **Eye Tracking** | Scannable layout | Headers, bullets, white space, visual hierarchy. Scannable in 30 seconds |

### ProposalBio Application by Document Type

| Document | Required Biohacks | Score Gate |
|----------|------------------|-----------|
| Buyer Email | #1, #2, #4, #5, #6, #7, #8, #9 | Document in SEND_TO_BUYER email file with biohack table |
| Capability Statement | All 10 | Full HTML generation with buyer language embedded |
| Proposal | All 10 | Composite >= 75, no critical failures, gate UNLOCKED |
| Supplier RFQ | None (protection rules only) | Pass supplier protection checklist |
| Subcontractor Outreach | None (protection rules only) | Pass buyer protection checklist |

### ProposalBio Documentation Requirement

**Every buyer email MUST include a ProposalBio framework table at the top of the SEND_TO_BUYER_EMAIL_READY.md file:**

```markdown
## PROPOSALBIO FRAMEWORK

| Biohack | Application in This Email |
|---------|--------------------------|
| #1 Mirror Neuron | [How tone was matched to this buyer's culture] |
| #2 Cognitive Ease | [Sentence length, reading level, structure] |
| #3 Story Arc | [Challenge → Solution → Result arc used] |
| #4 Reciprocity | [Value given before asking] |
| #5 Yes Stacking | [How capabilities mirror RFP requirements] |
| #6 Familiarity | [Specific buyer terms mirrored — list them] |
| #7 Name Recognition | [Agency name count, mission/vision reference] |
| #8 Sensory Language | [Concrete details: numbers, locations, regs] |
| #9 Rhythm | [Sentence variety, cadence breaks] |
| #10 Eye Tracking | [Structure, scannability, visual hierarchy] |

**BUYER LANGUAGE SOURCE:** [Where buyer language was sourced from]
```

**This is NOT optional. This is how we track that ProposalBio was actually applied, not just claimed.**

---

## STEP 4: VERIFY — Check Against Protection Rules

### For ALL Documents

Run these checks from `company-info-verification.mdc`:
- [ ] Phone: 248.376.4550 (NOT 248.247.5020)
- [ ] Address: Troy, Michigan 48084 (NOT 48083)
- [ ] Email: info@deedavis.biz (NOT bids@deedavisinc.com)
- [ ] CAGE: 8UMX3
- [ ] UEI: HJB4KNYJVGZ1

### For Buyer-Facing Documents

Run these checks:
- [ ] Agency name used 3+ times (Biohack #7)
- [ ] Solicitation number referenced
- [ ] Buyer language mirrored (not DDI generic language)
- [ ] EDWOSB mentioned in first paragraph
- [ ] Correct sector color scheme on cap statement
- [ ] Smart questions included (not generic)
- [ ] Signature block complete with all certs

### For Supplier-Facing Documents

Run these checks from `quote-generator-supplier-protection.mdc` and `rfq-buyer-protection-checklist.mdc`:
- [ ] NO agency/client names anywhere
- [ ] NO solicitation numbers
- [ ] NO specific addresses
- [ ] NO procurement officer names
- [ ] RFQ number is DDI-YYYY-### format
- [ ] Client reference is generic ("Michigan municipal client")
- [ ] Location is county-level max
- [ ] Supplier deadline is 3-5 days BEFORE government deadline

### Diversity Scan (from `diversity-inclusion-scanning.mdc`)

For EVERY solicitation, include:
```
## DIVERSITY ADVANTAGE
- [ ] EDWOSB set-aside: YES/NO
- [ ] WOSB preference: YES/NO
- [ ] Small business set-aside: YES/NO
- [ ] Local vendor preference: YES/NO
- [ ] Evaluation points for diversity: YES/NO
- [ ] DEI plan required: YES/NO
- [ ] Best value evaluation: YES/NO
```

---

## STEP 5: PLACE — Put Documents in the Right Folder

### Folder Structure (from `bid-folder-organization.mdc`)

```
BIDS:RESOURCES/[CLIENT] [BID TYPE]/
├── SEND_TO_BUYER/
│   ├── [SolNumber]_[Type]_Capability_Statement.html
│   ├── SEND_TO_BUYER_EMAIL_READY.md
│   └── [Any other buyer-facing docs]
├── SEND_TO_SUPPLIER/
│   ├── DDI-YYYY-###_RFQ_[Product].pdf
│   └── [Safe-to-share specs/drawings]
├── SEND_TO_SUBCONTRACTOR/
│   ├── SUBCONTRACTOR_OUTREACH_EMAIL.md
│   └── [Safe-to-share scope docs]
├── WORKFLOW_CHECKLIST.md
└── [Internal docs: strategy, analysis, scripts]
```

### Placement Rules

| Document | Goes In | Filename Convention |
|----------|---------|-------------------|
| Buyer email | SEND_TO_BUYER/ | `SEND_TO_BUYER_EMAIL_READY.md` |
| Capability statement | SEND_TO_BUYER/ | `[SolNumber]_[Type]_Capability_Statement.html` |
| Supplier RFQ | SEND_TO_SUPPLIER/ | `DDI-YYYY-###_RFQ_[Product].pdf` or `.html` |
| Sub outreach email | SEND_TO_SUBCONTRACTOR/ | `SUBCONTRACTOR_OUTREACH_EMAIL.md` |
| Workflow checklist | Root folder | `WORKFLOW_CHECKLIST.md` |
| Strategy/analysis | Root folder | Descriptive name |
| Original solicitation | Root folder | Original filename |

---

## STEP 6: PRESENT — Give Dee the Complete Package

**When presenting a completed outreach package to Dee, ALWAYS include:**

1. **Folder location** — "Everything is in `BIDS:RESOURCES/[FOLDER NAME]/SEND_TO_BUYER/`"
2. **What's in the folder** — List every file
3. **ProposalBio summary** — Which biohacks were applied and how
4. **Key buyer language used** — The specific terms mirrored from the RFP
5. **Action needed** — "Convert cap statement to PDF, attach, copy email, send to [CO email]"
6. **Next email** — "Ready for the next one?"

### Example Presentation

```
PACKAGE READY — [AGENCY NAME] [SERVICE]

Folder: BIDS:RESOURCES/[FOLDER NAME]/SEND_TO_BUYER/
Files:
  1. [SolNumber]_[Type]_Capability_Statement.html
  2. SEND_TO_BUYER_EMAIL_READY.md

ProposalBio Applied:
  - #6 Familiarity: Mirrored "[term 1]", "[term 2]", "[term 3]" from RFP
  - #7 Name Recognition: "[Agency]" appears X times
  - #1 Mirror Neuron: [State/Federal] tone, [Region] language

Buyer Language Source: [URL/document name]

ACTION: Convert cap statement to PDF → Attach → Copy email → Send to [CO email]
```

---

## DOCUMENT TYPE QUICK REFERENCE

### When Dee provides a solicitation number, PDF, or says "generate a response" / "bid on this" / "respond to this":
→ This is a **QUOTE RESPONSE** — a multi-page document, NOT a cap statement
→ STEP 1: Parse the solicitation — extract agency, SOW, CLINs, requirements, NAICS, set-aside, evaluation method, CO name/email, deadlines
→ STEP 2: Identify service lane → look up colors in `cap-statement-colors.mdc`
→ STEP 3: Copy `GENERATED_RESPONSES/QUOTE_RESPONSE_TEMPLATE.html`
→ STEP 4: Replace ALL `{{PLACEHOLDER}}` values with solicitation-specific content
→ STEP 5: Extract base64 logo from any existing cap statement in `BIDS:RESOURCES/`
→ STEP 6: Apply ProposalBio to Technical Capability section (mirror buyer language, use agency name 3+ times, sensory language, story arc)
→ STEP 7: Build Compliance Matrix — one row per solicitation requirement, all showing COMPLIANT
→ STEP 8: Build Pricing table — use exact CLINs from solicitation
→ STEP 9: Verify company info per `company-info-verification.mdc` (248.376.4550, info@deedavis.biz, 48084)
→ STEP 10: Save as `[SolNumber]_[Type]_Quote_Response.html` in `SEND_TO_BUYER/`
→ STEP 11: Tell Dee: "Quote response ready. Open in Chrome → Cmd+P → Save as PDF. [X] pages."

**The quote response ALWAYS uses the template. ALWAYS has a Table of Contents. ALWAYS has each section on its own page. ALWAYS matches the cap statement colors. No exceptions.**

### When Dee says "send an email" or "buyer outreach":
→ STEP 1: Classify as Buyer Email
→ STEP 2: Build email + cap statement together (ALWAYS paired)
→ STEP 3: Apply ProposalBio, document in biohack table
→ STEP 4: Verify company info + buyer language
→ STEP 5: Place both in SEND_TO_BUYER/
→ STEP 6: Present as complete package

### When Dee says "create a cap statement":
→ STEP 1: Classify sector for color scheme
→ STEP 2: Build with NEXUS HTML template, buyer language, all sections
→ STEP 3: Apply all 10 ProposalBio biohacks
→ STEP 4: Verify company info + agency name frequency
→ STEP 5: Place in SEND_TO_BUYER/
→ STEP 6: Present with biohack summary

### When Dee says "generate an RFQ" or "quote request for suppliers":
→ STEP 1: Classify as Supplier RFQ
→ STEP 2: Build with DDI-YYYY-### number, generic language
→ STEP 3: Skip ProposalBio (not buyer-facing)
→ STEP 4: Run supplier protection checklist — ZERO buyer info
→ STEP 5: Place in SEND_TO_SUPPLIER/
→ STEP 6: Present with protection confirmation

### When Dee says "presolicitation" or "sources sought":
→ Follow `presolicitation-auto-response.mdc` FIRST
→ Then run this pipeline for the cap statement + email
→ Cap statement + email generated in SAME session
→ Both placed in SEND_TO_BUYER/ immediately

### When Dee says "subcontractor outreach":
→ STEP 1: Classify as Subcontractor document
→ STEP 2: Build with generic scope, no buyer info
→ STEP 3: Skip ProposalBio
→ STEP 4: Run buyer protection checklist
→ STEP 5: Place in SEND_TO_SUBCONTRACTOR/
→ STEP 6: Present with protection confirmation

---

## METADATA COLLECTION

**Before generating any buyer-facing document, collect this metadata (used by ProposalBio analyzer):**

| Field | Source | Example |
|-------|--------|---------|
| `client_name` | Solicitation | "DTMB", "USACE", "VA" |
| `agency` | Solicitation | "Michigan Department of Technology, Management & Budget" |
| `agency_type` | Classification | "Federal", "State", "Local", "Cooperative" |
| `region` | Geography | "Midwest", "Northeast", "Southeast", "Southwest", "West_Coast", "Mid_Atlantic" |
| `rfp_number` | Solicitation | "RFP-171", "W912DR25QA005" |
| `rfp_text` | Solicitation scope | Key paragraphs from the RFP describing requirements |
| `service_type` | Classification | "Drug Testing", "Fingerprinting", "Grounds Maintenance" |
| `sector` | Color scheme | "Medical", "Defense", "Facilities", "Industrial", "Transportation", "Reseller" |
| `co_name` | Solicitation | Contracting officer name (for personalized greeting) |
| `co_email` | Solicitation | Contracting officer email (for TO field) |

---

## CRITICAL REMINDERS

1. **EVERYTHING goes through NEXUS.** No generating emails without cap statements. No cap statements without ProposalBio. No documents without the right folder.

2. **Email + Cap Statement are ALWAYS a pair for buyer outreach.** Never send an email to a buyer without a tailored cap statement. Never generate a cap statement without the accompanying email. They are one package.

3. **ProposalBio is NOT optional for buyer-facing docs.** The biohack table in SEND_TO_BUYER_EMAIL_READY.md proves the work was done. No table = the work wasn't done.

4. **Supplier documents get ZERO buyer information.** This is a business survival rule, not a preference.

5. **Company info comes from `company-info-verification.mdc`.** Phone is 248.376.4550. Email is info@deedavis.biz. ZIP is 48084. Period.

6. **Cap statement colors come from `cap-statement-colors.mdc`.** Match the sector. Every time.

7. **Folder structure comes from `bid-folder-organization.mdc`.** Create the full structure. Every time.

8. **Don't ask Dee to explain this again.** It's all here. Read it. Follow it. Every time.

---

## RULES THIS WORKFLOW REFERENCES

- `company-info-verification.mdc` — Contact info source of truth
- `cap-statement-colors.mdc` — Sector color schemes
- `bid-folder-organization.mdc` — Folder structure + SEND_TO rules
- `presolicitation-auto-response.mdc` — Presolicitation workflow
- `never-reveal-buyer-to-supplier.mdc` — Supplier protection
- `never-reveal-end-buyer.mdc` — End buyer protection
- `quote-generator-supplier-protection.mdc` — RFQ formatting
- `rfq-buyer-protection-checklist.mdc` — RFQ verification
- `diversity-inclusion-scanning.mdc` — EDWOSB advantage scan
- `human-touch-correspondence.mdc` — Buyer vs. supplier tone
- `dee-working-style.mdc` — Direct, honest, action-oriented

## BUSINESS MODEL RULE

**DDI is a Contract Management Firm (Prime/Sub model). This applies to EVERY document generated.**

- DDI primes all contracts — subs/partners execute the work
- All prices are DDI agency rates (what the client pays DDI)
- Never expose sub costs, worker pay, or platform fees in client-facing docs
- DDI's value = opportunity identification + bid prep + contract management + compliance + QA + billing
- Reference `DDI_BUSINESS_MODEL.md` for margin targets and SBA subcontracting limits
- Reference `DEE_DAVIS_INC_COMPLETE_SERVICE_CATALOG.md` for current agency rates
- Reference `ddi-business-model.mdc` for the full pricing and model rule

## CODE THIS WORKFLOW USES

- `proposalbio_module.py` — ProposalBioAnalyzer class, 10 biohack scoring engine
- `capability_statement_generator.py` — CapabilityStatementGenerator class, Airtable integration
- `capability_statement_template.html` — HTML template for cap statements
- `nexus_backend.py` — AirtableClient for logging to Airtable

---

*This workflow exists because Dee shouldn't have to explain it twice. Read the rule. Follow the steps. Generate the package. Present it complete. Every single time.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DEEDAVISINC)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DEEDAVISINC)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
