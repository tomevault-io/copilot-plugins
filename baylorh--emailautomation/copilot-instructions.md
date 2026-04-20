## emailautomation

> > **Purpose:** This file provides comprehensive guidance to Claude Code when working with this codebase. It serves as the authoritative reference for system architecture, AI behavior rules, and development workflows.

# CLAUDE.md

> **Purpose:** This file provides comprehensive guidance to Claude Code when working with this codebase. It serves as the authoritative reference for system architecture, AI behavior rules, and development workflows.

---

## ⚠️ CRITICAL: Testing Account Information

**DO NOT FORGET THIS. EVER.**

| Role | Account | Notes |
|------|---------|-------|
| **Sending Account (Outbox)** | `baylor.freelance@outlook.com` | This is the ONLY account used for sending emails during development/testing |
| **Test Broker 1** | `bp21harrison@gmail.com` | Simulates broker replies |
| **Test Broker 2** | `baylor@manifoldengineering.ai` | Simulates broker replies |

**The product is being built FOR Jill Ames (jill@mohrpartners.com), but ALL testing and development uses baylor.freelance@outlook.com as the sending account.**

**NEVER reference jill@mohrpartners.com when discussing test setup, mailbox clearing, MSAL caches, or any hands-on testing activities.**

---

## ⚠️ E2E Test File (ALWAYS USE THIS)

**Standard test file:** `test_pdfs/E2E_Real_World_Test.xlsx`

This file is pre-configured with:
- 7 test properties (699 Industrial, 150 Trade Center, 2017 St. Josephs, 9300 Lottsford, 1 Randolph, 1800 Broad, 2525 Center West)
- Custom columns for testing: `Rail Access`, `Office %`, `Sprinkler Type`, `Building Condition Notes`
- Script sheet with outreach template (asks about parking)
- Broker emails split between `bp21harrison@gmail.com` and `baylor@manifoldengineering.ai`

**Column configuration during setup:**
| Column | Mode | Tests |
|--------|------|-------|
| Rail Access | Ask (Required) | Blocks closing if missing |
| Office % | Ask (Optional) | AI asks but doesn't block |
| Sprinkler Type | Accept Only | Extract if mentioned, never ask |
| Building Condition Notes | Note | Auto-append, never ask |

**DO NOT create new test Excel files. Always use this one.**

---

## Project Overview

**What is this?** An AI-powered commercial real estate email automation system that:
1. Sends personalized outreach emails to property brokers on behalf of tenant rep clients
2. Automatically processes broker replies using GPT-4o to extract property data
3. Populates Google Sheets with extracted information (SF, rent, docks, power, etc.)
4. Detects events requiring human attention (tours, negotiations, identity questions)
5. Maintains conversation threads until all required information is gathered

**Who uses it?** Tenant representatives (commercial real estate professionals who help businesses find warehouse/industrial space). They upload a spreadsheet of properties, configure email templates, and the system handles broker communication automatically.

**Business value:** Automates the tedious process of gathering property specifications from dozens of brokers simultaneously, freeing tenant reps to focus on high-value activities.

---

## Repository Structure

| Repository | Purpose | Tech Stack |
|------------|---------|------------|
| **EmailAutomation** (this repo) | Backend - email processing, AI extraction, sheet updates | Python, OpenAI, Microsoft Graph, Google Sheets API |
| **email-admin-ui** | Frontend - user dashboard, client management, campaign launch | React, Firebase, Tailwind CSS |

```
├── email_automation/           # Core Python modules
│   ├── ai_processing.py       # OpenAI extraction & event detection (THE BRAIN)
│   ├── processing.py          # Main pipeline: inbox → AI → sheet → notification
│   ├── email.py               # Outbox processing, email sending via Graph API
│   ├── sheets.py              # Google Sheets read/write operations
│   ├── sheet_operations.py    # NON-VIABLE divider, row movement
│   ├── messaging.py           # Firestore thread/message indexing
│   ├── notifications.py       # User notification creation
│   ├── column_config.py       # Dynamic column mapping system
│   ├── followup.py            # Automatic follow-up scheduling
│   └── clients.py             # Firebase/Google API client initialization
├── tests/                      # Comprehensive test suite
│   ├── standalone_test.py     # 25 AI extraction scenarios
│   ├── e2e_test.py            # Full pipeline tests with Scrub file
│   ├── campaign_lifecycle_test.py  # Multi-property campaign tests
│   └── conversations/         # Broker reply fixtures
├── main.py                     # Entry point: processes all users
├── app.py                      # Flask server for OAuth & debug APIs
└── Scrub Augusta GA.xlsx       # Test data file
```

---

## Accessing Outlook Inbox (IMPORTANT - READ THIS)

**DO NOT reinvent Outlook access. Use the helper script:**

```bash
# List inbox messages
python tests/outlook_helper.py inbox

# List inbox for specific user
python tests/outlook_helper.py inbox NO7lVYVp6BaplKYEfMlWCgBnpdh2

# Download attachments from a message
python tests/outlook_helper.py attachments <msg_id>

# List available user IDs
python tests/outlook_helper.py users
```

**Default user:** `NO7lVYVp6BaplKYEfMlWCgBnpdh2` (baylor.freelance@outlook.com)

**How it works internally:**
1. `download_token()` from `firebase_helpers.py` downloads MSAL token from Firebase Storage
2. Token is stored at path: `msal_caches/{user_id}/msal_token_cache.bin`
3. MSAL `ConfidentialClientApplication` uses the cached token
4. `acquire_token_silent()` gets the access token
5. Microsoft Graph API calls use the access token

**NEVER do this manually. Always use `tests/outlook_helper.py`.**

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              USER FLOW                                   │
│                                                                          │
│  1. Upload Excel → 2. Configure columns → 3. Launch campaign → 4. Monitor│
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
┌──────────────────────────────┐    ┌──────────────────────────────────────┐
│      FRONTEND (React)        │    │         BACKEND (Python)             │
│                              │    │                                      │
│  • Upload Excel files        │    │  • Process outbox → send emails      │
│  • Configure campaigns       │    │  • Monitor inbox for replies         │
│  • View notifications        │    │  • AI extraction via OpenAI          │
│  • Compose user replies      │    │  • Update Google Sheets              │
│  • Review conversations      │    │  • Create notifications              │
│                              │    │  • Manage thread state               │
└──────────┬───────────────────┘    └──────────────────┬───────────────────┘
           │                                           │
           └─────────────► FIRESTORE ◄─────────────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
              Microsoft Graph      Google Sheets
              (Outlook email)      (Data storage)
```

### Data Ownership

| Collection | Written By | Read By | Purpose |
|------------|------------|---------|---------|
| `users/{uid}/clients/` | Frontend | Backend | Client metadata, criteria |
| `users/{uid}/outbox/` | Frontend | Backend | Emails queued for sending |
| `users/{uid}/threads/` | Backend | Frontend | Conversation threads |
| `users/{uid}/msgIndex/` | Backend | Backend | O(1) message lookup |
| `users/{uid}/notifications/` | Backend | Frontend | Real-time updates |

### Email Processing Pipeline

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         EVERY 30 MINUTES                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. OUTBOX PROCESSING (email.py)                                        │
│     └─> Read users/{uid}/outbox/                                        │
│     └─> Send via Microsoft Graph API                                    │
│     └─> Index in threads/, msgIndex/, convIndex/                        │
│     └─> Delete outbox entry                                             │
│                                                                          │
│  2. INBOX PROCESSING (processing.py)                                    │
│     └─> Fetch new emails from Graph API                                 │
│     └─> Match to indexed threads via conversationId/inReplyTo           │
│     └─> Load sheet context (current column values)                      │
│     └─> Call AI extraction (ai_processing.py)                           │
│     └─> Write extracted data to Google Sheets                           │
│     └─> Create notifications for frontend                               │
│     └─> Send auto-reply OR pause for user input                         │
│                                                                          │
│  3. FOLLOW-UP PROCESSING (followup.py)                                  │
│     └─> Check threads with no response for X days                       │
│     └─> Send follow-up emails (up to 3 per thread)                      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## AI Processing Rules (CRITICAL)

> **These rules define what the AI assistant CAN and CANNOT do when processing broker emails. They are the core "personality" of the system.**

### Identity & Boundaries

The AI acts as a professional assistant for the tenant rep. It:
- Gathers property information efficiently and politely
- Never reveals the client's identity, company name, or budget
- Never makes commitments about tours, contracts, or negotiations
- Escalates any question it cannot confidently answer

### Field Extraction Rules

| Field | Can Extract? | Can Request? | Notes |
|-------|--------------|--------------|-------|
| Total SF | ✅ Yes | ✅ Yes | Primary required field |
| Ops Ex /SF | ✅ Yes | ✅ Yes | Operating expenses (NNN) |
| Drive Ins | ✅ Yes | ✅ Yes | Number of drive-in doors |
| Docks | ✅ Yes | ✅ Yes | Number of loading docks |
| Ceiling Ht | ✅ Yes | ✅ Yes | Clear height in feet |
| Power | ✅ Yes | ✅ Yes | Electrical capacity |
| Rent/SF /Yr | ✅ Yes | ❌ Never | Accept if offered, never ask |
| Gross Rent | ❌ Never | ❌ Never | Formula column: `=(H+I)*G/12` |
| Flyer / Link | ✅ Yes | ✅ Yes | Brochure URLs |
| Floorplan | ✅ Yes | ✅ Yes | Layout document links |
| Listing Brokers Comments | ✅ Append only | N/A | Contextual notes, not data |

### Read-Only Fields (NEVER Update)

These fields contain pre-existing client data that the AI must NEVER modify, even if someone different signs the email:

- `Property Address` - Property identifier
- `City` - Location
- `Property Name` - Building name
- `Leasing Company` - Broker's firm
- `Leasing Contact` - Original contact name
- `Email` - Original contact email

**Why:** If broker "Bob Smith" replies but the row shows "Jane Doe", we keep Jane Doe because she's the official contact on record. Bob might be covering for Jane temporarily.

### Event Detection

The AI detects these events from broker replies:

| Event | Trigger | AI Response |
|-------|---------|-------------|
| `property_unavailable` | "No longer available", "leased", "off market" | Ask for alternatives, move row below NON-VIABLE |
| `new_property` | Broker suggests different property | Create approval notification with suggested email |
| `call_requested` | Broker wants to discuss by phone | Escalate to user, pause auto-replies |
| `tour_requested` | Broker offers showing/tour | Create notification with pre-filled acceptance |
| `close_conversation` | Natural end (e.g., "glad to help") | Mark thread complete |
| `needs_user_input` | Question AI can't answer | Escalate to user with context |
| `contact_optout` | Broker refuses communication | Add to opt-out list |
| `wrong_contact` | Wrong person for this property | Escalate with redirect info |
| `property_issue` | Health/safety or major concern | Escalate by severity |

### Escalation Subreasons

When `needs_user_input` is detected, include a specific subreason:

| Subreason | Trigger |
|-----------|---------|
| `needs_user_input:confidential` | "Who is this for?", "What company?" |
| `needs_user_input:client_question` | "What size do you need?", "What's your budget?" |
| `needs_user_input:scheduling` | Scheduling requests requiring user calendar |
| `needs_user_input:negotiation` | Price counters, term negotiations |
| `needs_user_input:legal_contract` | LOI, lease, contract questions |
| `needs_user_input:unclear` | Ambiguous (fallback) |

### Response Quality Standards

When generating reply emails, the AI must:
- Be professional, concise, and friendly
- Thank the broker for information provided
- Only request fields that are: (a) still missing AND (b) in the required list
- Never request Rent/SF or Gross Rent
- Never repeat information the broker already provided
- Never ask the same question twice in a conversation

### Number Formatting

All numeric values must be written as plain decimals:
- ✅ `12500` not `12,500 SF` or `12,500 square feet`
- ✅ `24.50` not `$24.50/SF` or `$24.50 per square foot`
- ✅ `28` not `28'` or `28 feet`

---

## Development Workflow

### Quick Start

```bash
# Backend (this repo)
pip install -r requirements.txt
python main.py                  # Run email processing
python app.py                   # Run Flask server for OAuth

# Frontend (email-admin-ui)
cd ~/Documents/GitHub/email-admin-ui
npm install
npm start                       # Development server
```

### Making Changes

**Frontend Changes:**
```bash
cd ~/Documents/GitHub/email-admin-ui
# Make changes to src/, styles/, components/
CI=true npm run build           # MUST pass before commit
git add -A && git commit -m "description" && git push
# GitHub Actions auto-deploys to Firebase Hosting + Functions
```

**Backend Changes:**
```bash
cd ~/Documents/GitHub/EmailAutomation
# Make changes to email_automation/
python3 -m py_compile email_automation/<file>.py   # Syntax check
python tests/standalone_test.py                     # MUST pass (25/25)
python tests/e2e_test.py                           # MUST pass
git add -A && git commit -m "description" && git push
# Render auto-deploys on push
```

### Testing Requirements

> **⚠️ MANDATORY: Run tests after ANY code change. Tests hit production code paths.**

| Test Suite | Command | Pass Criteria |
|------------|---------|---------------|
| AI Extraction | `python tests/standalone_test.py` | 25/25 scenarios pass |
| E2E Pipeline | `python tests/e2e_test.py` | All properties process correctly |
| Campaign Lifecycle | `python tests/campaign_lifecycle_test.py` | 11/11 scenarios pass |

### Pre-Commit Checklist

- [ ] `CI=true npm run build` passes (frontend)
- [ ] `python tests/standalone_test.py` passes (backend)
- [ ] `python tests/e2e_test.py` passes (backend)
- [ ] Commit message is descriptive
- [ ] No secrets/credentials in code

---

## Test Scenarios Reference

### Core Scenarios (25)

| Category | Scenario | Expected Behavior |
|----------|----------|-------------------|
| **Data Extraction** | `complete_info` | All fields → send closing email |
| | `partial_info` | Some fields → request remaining |
| | `vague_response` | No data → re-request specifics |
| | `multi_turn_conversation` | Accumulate data across messages |
| **Events** | `property_unavailable` | Move to NON-VIABLE, ask for alternatives |
| | `new_property_suggestion` | Create approval notification |
| | `call_requested_with_phone` | Escalate (don't ask for phone again) |
| | `tour_requested` | Create notification with suggested response |
| **Escalation** | `identity_question` | `needs_user_input:confidential` |
| | `budget_question` | `needs_user_input:client_question` |
| | `negotiation_attempt` | `needs_user_input:negotiation` |
| | `legal_contract_question` | `needs_user_input:legal_contract` |
| **Contact Issues** | `contact_optout_not_interested` | Add to opt-out list |
| | `wrong_contact_redirected` | Note forwarding, keep original contact |
| | `different_person_replies` | Keep original Leasing Contact |
| **Property Issues** | `property_issue_major` | Escalate with severity |
| | `property_issue_critical` | Urgent escalation |

### E2E Campaign Test (Verified 2026-03-02)

| Property | Scenario | Result |
|----------|----------|--------|
| 699 Industrial Park Dr | `complete_info` | ✅ Row completed, closing email sent |
| 1 Kuhlke Dr | `partial_info` + `complete_remaining` | ✅ Multi-turn completion |
| 2058 Gordon Hwy | `wrong_contact:forwarded` | ✅ Escalated correctly |
| 1 Randolph Ct | `property_issue:major` | ✅ Issue flagged |
| 135 Trade Center Court | `unavailable` + `new_property` | ✅ New property approval flow |
| 555 Commerce Blvd | `needs_user_input:confidential` | ✅ Paused, awaiting user |
| 200 Logistics Lane | `close_conversation` | ✅ Natural termination |

---

## Notification System

### Notification Types

| Kind | Priority | Trigger | Frontend Action |
|------|----------|---------|-----------------|
| `sheet_update` | normal | AI extracts a field | Show in sidebar |
| `row_completed` | normal | All required fields filled | Mark property complete |
| `property_unavailable` | normal | Property off market | Show unavailable badge |
| `action_needed` | important | User input required | Show action modal |
| `conversation_closed` | normal | Thread naturally ended | Mark thread done |

### `action_needed` Reasons

| Reason | User Action Required |
|--------|----------------------|
| `call_requested` | Schedule call or decline |
| `tour_requested` | Accept/modify tour response |
| `new_property_pending_approval` | Approve and send outreach email |
| `needs_user_input:*` | Compose reply to broker question |
| `contact_optout:*` | Acknowledge, possibly remove from list |
| `wrong_contact:*` | Update contact info or acknowledge |
| `property_issue:*` | Review issue details |

---

## Conversation Display (Frontend)

The `ConversationsModal` component displays email threads grouped by property:

- **Grouping:** Threads with same property address (extracted from subject, ignoring "RE:", "FW:") are merged into a single conversation view
- **Sorting:** Messages sorted chronologically within each property group
- **Thread count:** Shows "N threads" badge when multiple threads exist for same property
- **Deduplication:** Multiple outbox entries to same property appear as single conversation

This provides a clean "campaign summary" view showing the complete communication history per property.

---

## Sheet Operations

### NON-VIABLE Divider

A special row in the Google Sheet separating viable properties (above) from non-viable (below):
- Properties marked `property_unavailable` are moved below this divider
- New suggested properties are inserted above this divider
- `sheet_operations.py` handles row movement

### Column Mapping

The system dynamically maps user column headers to canonical fields:

```python
# Example: User's "Square Footage" maps to canonical "total_sf"
column_config = {
    "mappings": {"Square Footage": "total_sf", ...},
    "extractionFields": ["total_sf", "ops_ex_sf", ...],
    "requiredFields": ["total_sf", "ops_ex_sf", "drive_ins", "docks", "ceiling_ht", "power"],
    "formulaFields": ["gross_rent"],  # Never write
    "neverRequest": ["rent_sf_yr"]    # Accept but don't ask
}
```

### Required Fields for Completion

A row is "complete" when these fields have values:
- Total SF
- Ops Ex /SF
- Drive Ins
- Docks
- Ceiling Ht
- Power

---

## API Reference

### Flask Endpoints (app.py)

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/auth/login` | GET | Initiate Microsoft OAuth |
| `/auth/callback` | GET | OAuth callback handler |
| `/api/status` | GET | Check token validity |
| `/api/trigger-scheduler` | POST | Manual processing trigger |
| `/api/accept-new-property` | POST | Create row for suggested property |
| `/api/decline-property` | POST | Remove suggested property row |
| `/api/list-optouts` | POST | Get opted-out contacts |
| `/api/clear-optout` | POST | Remove from opt-out list |

### Firebase Functions (email-admin-ui/functions)

| Function | Purpose |
|----------|---------|
| `api` | Create Google Sheet from uploaded Excel |
| `deleteSheet` | Delete Google Sheet when client removed |
| `analyzeSheetColumns` | AI maps Excel columns to canonical fields |
| `chatWithPropertyContext` | AI chat for composing replies |

---

## Environment Variables

```bash
# Azure (Microsoft Graph)
AZURE_API_APP_ID=
AZURE_API_CLIENT_SECRET=
AZURE_TENANT_ID=

# Firebase
FIREBASE_API_KEY=
FIREBASE_SA_KEY=

# Google
GOOGLE_OAUTH_CLIENT_ID=
GOOGLE_OAUTH_CLIENT_SECRET=
GOOGLE_REFRESH_TOKEN=

# OpenAI
OPENAI_API_KEY=
OPENAI_ASSISTANT_MODEL=gpt-4o
```

---

## Deployment

| Component | Platform | Deploy Trigger |
|-----------|----------|----------------|
| Frontend | Firebase Hosting | Push to main → GitHub Actions |
| Functions | Firebase Functions | Push to main → GitHub Actions |
| Backend | Render.com | Push to main → Auto-deploy |
| Scheduler | GitHub Actions | Cron every 30 minutes |

---

## Troubleshooting

### Common Issues

**"Token expired" errors:**
- Run `python app.py` and re-authenticate via `/auth/login`
- Token cache is in `msal_caches/*.bin`

**Emails not sending:**
- Check `users/{uid}/outbox/` in Firestore console
- Verify Microsoft Graph token is valid: `GET /api/status`

**AI not extracting data:**
- Check OpenAI API key is valid
- Review `ai_processing.py` prompts
- Run `python tests/standalone_test.py -s <scenario>` to debug

**Thread not matching:**
- Check `msgIndex/` and `convIndex/` collections
- Verify `conversationId` or `inReplyTo` headers present

**Notifications not appearing:**
- Check `users/{uid}/clients/{clientId}/notifications/` in Firestore
- Verify frontend `onSnapshot` listener is active

---

## Testing & Debugging Commands

### Accessing Outlook Emails (CRITICAL - DO NOT FORGET)

> **⚠️ This is the ONLY correct way to access Outlook. Do NOT try other methods.**

The backend uses MSAL tokens stored in Firebase Storage. To fetch Outlook conversations:

```bash
cd /Users/baylorharrison/Documents/GitHub/EmailAutomation
export $(cat .env | grep -v '^#' | xargs)
python3 tests/e2e_helpers.py outlook
```

**How it works (same as main.py):**
1. Downloads token cache from Firebase Storage via `firebase_helpers.download_token()`
2. Creates MSAL `ConfidentialClientApplication` with the cache
3. Calls `acquire_token_silent()` to get access token
4. Uses Microsoft Graph API to fetch SentItems and Inbox

**Key files:**
- `tests/e2e_helpers.py` → `fetch_outlook_conversations()` - Fetches and displays all conversations
- `firebase_helpers.py` → `download_token()` - Downloads MSAL cache from Firebase Storage
- `main.py` (lines 50-120) - Reference implementation

**DO NOT:**
- Try to use local cache files directly (they may be stale)
- Use `requests` with hardcoded tokens
- Use the Flask app's `/api/status` endpoint for this

### E2E Test Helper Commands

```bash
# Check Firestore status (threads, outbox, notifications)
python3 tests/e2e_helpers.py status

# Check specific collections
python3 tests/e2e_helpers.py threads
python3 tests/e2e_helpers.py outbox
python3 tests/e2e_helpers.py notifications

# Trigger GitHub Actions workflow
python3 tests/e2e_helpers.py trigger

# Check workflow status
python3 tests/e2e_helpers.py workflow

# Clear all test data (DESTRUCTIVE)
python3 tests/e2e_helpers.py clear
```

---

## ⚠️ E2E Testing Philosophy (CRITICAL - READ THIS)

**The purpose of E2E tests is to verify NEW features work correctly, not just re-test existing functionality.**

### When Setting Up E2E Tests:
1. **Always include at least one NEW/untested variable** - If we just added support for a new custom field mode, the E2E test MUST include that mode
2. **Test the feature you just built** - The whole point is verification. Don't test the same 6 standard fields every time.
3. **Test ALL custom field modes** - ask_required, ask_optional, accept_only, note

### Example:
- We add support for "accept_only" mode (extract but never ask)
- E2E test MUST: Include a custom field set to "Accept Only"
- E2E test verifies: AI extracts the value when mentioned, never asks for it, doesn't block closing

**DO NOT just run the same test with the same standard fields over and over. E2E tests exist to verify NEW code paths.**

---

## E2E Test Cleanup Procedures

### What Claude Does (Automated)
- Clear `baylor.freelance@outlook.com` inbox/sent (via Graph API)
- Delete campaign from Firebase (client, threads, indexes, notifications)

### What User Does (Manual)
- **Clear `bp21harrison@gmail.com`** - Delete test campaign emails from inbox/sent
- **Clear `baylor@manifoldengineering.ai`** - Delete test campaign emails from inbox/sent

**ALWAYS remind the user to clear their broker test accounts before starting a new E2E test.**

---

### 1. Clear Outlook Mailbox (baylor.freelance@outlook.com)

```python
# Run from EmailAutomation directory
export $(cat .env | grep -v '^#' | xargs) && export GOOGLE_APPLICATION_CREDENTIALS=service-account.json && python3 << 'EOF'
import requests
from tests.outlook_helper import get_access_token

token = get_access_token()
headers = {"Authorization": f"Bearer {token}"}

# Keywords to match campaign emails
campaign_keywords = [
    "industrial", "trade center", "st. josephs", "lottsford",
    "randolph", "broad st", "center west", "commerce way",
    "warehouse", "distribution", "logistics", "storage",
    "augusta", "evans", "kuhlke", "gordon hwy"
]

folders = ['Inbox', 'SentItems']
total_deleted = 0

for _ in range(5):  # Multiple passes to clear all
    for folder in folders:
        resp = requests.get(
            f"https://graph.microsoft.com/v1.0/me/mailFolders/{folder}/messages",
            headers=headers,
            params={"$top": "100", "$select": "id,subject,from"}
        )
        for msg in resp.json().get("value", []):
            subject = msg.get("subject", "").lower()
            if any(kw in subject for kw in campaign_keywords):
                requests.delete(f"https://graph.microsoft.com/v1.0/me/messages/{msg['id']}", headers=headers)
                total_deleted += 1

print(f"✅ Deleted {total_deleted} campaign emails")
EOF
```

### 2. Delete Campaign from Firebase

```python
export $(cat .env | grep -v '^#' | xargs) && export GOOGLE_APPLICATION_CREDENTIALS=service-account.json && python3 << 'EOF'
from google.cloud import firestore

db = firestore.Client()
user_id = "NO7lVYVp6BaplKYEfMlWCgBnpdh2"  # baylor.freelance@outlook.com

# Find and delete campaign
clients = list(db.collection('users').document(user_id).collection('clients').stream())
for client in clients:
    client_id = client.id
    name = client.to_dict().get('name', 'Unknown')
    print(f"Deleting campaign: {name}")

    # Delete threads
    threads = db.collection('users').document(user_id).collection('threads').where('clientId', '==', client_id).stream()
    for t in threads:
        # Delete messages subcollection
        msgs = db.collection('users').document(user_id).collection('threads').document(t.id).collection('messages').stream()
        for m in msgs:
            m.reference.delete()
        t.reference.delete()

    # Delete notifications
    notifs = db.collection('users').document(user_id).collection('clients').document(client_id).collection('notifications').stream()
    for n in notifs:
        n.reference.delete()

    # Delete outbox items
    outbox = db.collection('users').document(user_id).collection('outbox').where('clientId', '==', client_id).stream()
    for o in outbox:
        o.reference.delete()

    # Delete msgIndex entries
    msg_index = db.collection('users').document(user_id).collection('msgIndex').stream()
    for m in msg_index:
        m.reference.delete()

    # Delete convIndex entries
    conv_index = db.collection('users').document(user_id).collection('convIndex').stream()
    for c in conv_index:
        c.reference.delete()

    # Delete the client
    client.reference.delete()
    print(f"✅ Deleted campaign and all related data")
EOF
```

### 3. Verify Clean State

```bash
# Check inbox is clean
python3 tests/outlook_helper.py inbox

# Check Firebase has no test data
export $(cat .env | grep -v '^#' | xargs) && export GOOGLE_APPLICATION_CREDENTIALS=service-account.json && python3 -c "
from google.cloud import firestore
db = firestore.Client()
user_id = 'NO7lVYVp6BaplKYEfMlWCgBnpdh2'
clients = list(db.collection('users').document(user_id).collection('clients').stream())
threads = list(db.collection('users').document(user_id).collection('threads').stream())
print(f'Clients: {len(clients)}, Threads: {len(threads)}')
"
```

---

## Key Learnings from E2E Testing

1. **Thread grouping matters:** Users expect one conversation per property, not per email thread. Frontend groups by property address.

2. **Duplicate prevention is critical:** New Property modal needed ref-based guards to prevent multiple submissions.

3. **Optimistic UI improves UX:** Dismiss buttons immediately on click, don't wait for Firestore confirmation.

4. **Read-only fields must be protected:** Even when a different person replies, never update Leasing Contact or Email.

5. **Escalation pauses auto-reply:** When `needs_user_input` is detected, the system waits for the user to compose a reply—no auto-reply is sent.

6. **Multi-turn conversations work:** The system correctly accumulates data across multiple broker replies until all required fields are gathered.

---

## E2E Test Results (2026-03-04)

### Conversation Quality Scores

| Property | Score | Notes |
|----------|-------|-------|
| 100 Commerce Way | A (5/5) | Clean conversation, proper flow |
| 200 Industrial Blvd | A (5/5) | Multi-turn handled correctly |
| 300 Warehouse Dr | C (3/5) | ⚠️ Double greeting bug in initial email |
| 400 Distribution Ave | A (5/5) | Correctly escalated identity question |
| 500 Logistics Ln | A (5/5) | Correctly escalated tour request |
| 600 Storage Ct | D (2/5) | ⚠️ Double greeting + duplicate follow-ups |

**Overall: 25/30 (83%)**

### Bugs Found & Fixed

**Bug 1: Duplicate Follow-ups (FIXED)**
- **Symptom:** 600 Storage Ct received two follow-up emails
- **Cause:** Race condition - multiple workflow runs could send same follow-up
- **Fix:** Added claim mechanism in `followup.py` using transactions

**Bug 2: Double Greeting in Multi-Property Emails (TODO)**
- **Symptom:** Emails show "Hi," followed by "Hi [Name],"
- **Affected:** 300 Warehouse Dr, 600 Storage Ct (both 2nd-contact emails)
- **Likely cause:** Frontend generates scripts with "Hi," that don't account for name insertion
- **Location to fix:** Frontend LLM prompt for multi-property email generation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BaylorH) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
