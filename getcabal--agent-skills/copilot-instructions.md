## agent-skills

> Cabal's MCP server lets you search people and companies, discover warm introduction paths through your team's network, and manage connectors.

# Cabal MCP (Secure)

Cabal's MCP server lets you search people and companies, discover warm introduction paths through your team's network, and manage connectors.

- **Endpoint**: `POST /secure_mcp`
- **Protocol**: MCP `2025-03-26` (JSON-RPC 2.0)
- **Auth**: OAuth 2.0 Bearer token
- **Scopes**: `read` (all tools), `write` (required for `add_connectors`)
- **Server**: `cabal-secure-mcp` v0.1.1

---

## PII Minimization

This endpoint is designed for third-party AI clients (Claude Desktop, ChatGPT, etc.). Tool responses minimize personally identifiable information:

- **No LinkedIn URLs** — replaced with opaque tool IDs (e.g., "Person ID (for tool use)")
- **No Cabal profile URLs** — replaced with opaque tool IDs
- **No email addresses** in search results, connector listings, or message recipient lists
- **No per-recipient message telemetry** — only aggregate engagement counts and rates
- **No timestamps** on messages, imports, or sent dates
- **No inferred connection reasons** (e.g., "Worked together at Google")

Tool IDs (UUIDs) are returned so you can chain them between tools. They are opaque identifiers — do not present them to the user as links.

---

## Core Concept: The UUID Chain

Most workflows follow a **search-then-act** pattern. Search tools return UUIDs; connector and intro tools consume them. Never fabricate UUIDs -- always obtain them from a prior tool call.

| ID Type | Produced By | Consumed By |
|---------|------------|-------------|
| Company UUID | `search_companies` | `get_company_connectors`, `get_company_connections`, `get_connector_intros_at_company` |
| Person UUID | `search_people` | `get_person_connectors`, `add_connectors` |
| Connector UUID | `get_company_connectors`, `get_person_connectors` | `get_connector_intros_at_company` |
| Location name | `lookup_locations` | `search_companies`, `search_people` |
| Industry name | `search_companies_industries` | `search_companies` |
| Investor ID (integer) | `search_companies_investors` | `search_companies` |
| Company UUID (for people filters) | `search_people_companies` | `search_people` |
| People list UUID | `search_people_person_lists` | `search_people` |
| Company list UUID | `search_people_company_lists` | `search_people` |

Always use helper tools (`lookup_locations`, `search_companies_industries`, etc.) to resolve valid filter values rather than guessing.

---

## Tool Reference

### Identity

#### `whoami`

Returns the authenticated user's workspace context.

- **Params**: none
- **Returns**: JSON with name, email, title, team name, and team role

---

### Search

#### `search_companies`

Search the global company database by name, domain, industry, location, size, funding, and investors.

- **Params**:
  - `query` (string) -- company name or domain
  - `industries` (string[]) -- use `search_companies_industries` to find valid values
  - `location_names` (string[]) -- use `lookup_locations` to find valid values
  - `sizes` (string[]) -- employee count ranges (e.g. "11-50", "51-200")
  - `latest_funding_stages` (string[]) -- e.g. "Series A", "Series B"
  - `total_funding_raised` (string[]) -- funding amount ranges
  - `founded` (string[]) -- year founded (e.g. "2020")
  - `investors` (integer[]) -- investor IDs from `search_companies_investors`
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: companies with Company ID, domain, headline, industry, location, size, founded, funding
- **Chains to**: `get_company_connectors`, `get_company_connections`, `get_connector_intros_at_company`

#### `search_people`

Search the global people database by name, company, headline, LinkedIn URL, job function, seniority, and location.

- **Params**:
  - `query` (string) -- person name, company name, headline keywords, or LinkedIn URL
  - `functions` (string[]) -- e.g. "Engineering", "Sales", "Marketing", "Finance", "Human Resources", "Operations", "Product", "Design", "Legal"
  - `levels` (string[]) -- e.g. "C-Suite", "VP", "Director", "Manager", "Senior", "Entry"
  - `location_names` (string[]) -- use `lookup_locations` to find valid values
  - `current_company_uuids` (string[]) -- use `search_people_companies` to find valid UUIDs
  - `former_company_uuids` (string[]) -- use `search_people_companies` to find valid UUIDs
  - `profile_list_uuids` (string[]) -- use `search_people_person_lists` to find valid UUIDs
  - `company_list_uuids` (string[]) -- use `search_people_company_lists` to find valid UUIDs
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: people with Person ID, headline, company, location (no LinkedIn URLs)
- **Chains to**: `get_person_connectors`, `add_connectors`

---

### Filter Helpers

These tools resolve human-readable names into the exact values and IDs that search tools accept. Call them before applying filters.

#### `lookup_locations`

Find valid location values for the `location_names` filter on `search_companies` and `search_people`.

- **Params**: `query` (string, **required**) -- e.g. "San Francisco", "New York", "London"
- **Returns**: matching location strings ranked by relevance

#### `search_companies_industries`

Find valid industry values for the `industries` filter on `search_companies`.

- **Params**: `query` (string, **required**) -- e.g. "AI", "fintech", "healthcare"
- **Returns**: matching industry strings ranked by relevance

#### `search_companies_investors`

Find investor company IDs for the `investors` filter on `search_companies`.

- **Params**: `query` (string, **required**) -- e.g. "Sequoia", "Andreessen"
- **Returns**: `{label, value}` pairs where `value` is an **integer** ID (not a UUID)

#### `search_people_companies`

Find company UUIDs for the `current_company_uuids` and `former_company_uuids` filters on `search_people`.

- **Params**: `query` (string, **required**) -- e.g. "Google", "Stripe"
- **Returns**: `{label, value}` pairs where `value` is a company UUID

#### `search_people_person_lists`

List accessible people lists to get UUIDs for the `profile_list_uuids` filter on `search_people`.

- **Params**: `query` (string, optional) -- filter lists by name
- **Returns**: `{label, value}` pairs where `value` is a list UUID

#### `search_people_company_lists`

List accessible company lists to get UUIDs for the `company_list_uuids` filter on `search_people`.

- **Params**: `query` (string, optional) -- filter lists by name
- **Returns**: `{label, value}` pairs where `value` is a list UUID

---

### Connectors & Intros

#### `get_company_connectors`

Find team members who can make warm introductions to people at a specific company. Results are sorted by connection strength.

- **Params**:
  - `company_uuid` (string, **required**) -- from `search_companies`
  - `query` (string) -- filter connectors by name
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: connectors with Connector ID, headline, company, and connection type (Direct/Inferred). No LinkedIn URLs or inferred connection reasons.
- **Chains to**: `get_connector_intros_at_company`

#### `get_company_connections`

Find all reachable people at a company -- people your team can reach through warm intros, grouped with their connectors.

- **Params**:
  - `company_uuid` (string, **required**) -- from `search_companies`
  - `query` (string) -- filter connections by name
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: people at the company with Person ID, headline, each with a list of connectors (name, Connector ID, Direct/Inferred). No LinkedIn URLs.

#### `get_connector_intros_at_company`

Find which specific people at a company a given connector can introduce you to.

- **Params**:
  - `company_uuid` (string, **required**) -- from `search_companies`
  - `connector_uuid` (string, **required**) -- from `get_company_connectors`
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: people with Person ID, headline, and connection type. No LinkedIn URLs, shared company counts, or overlap years.

#### `get_person_connectors`

Find team members who can make a warm introduction to a specific person. Results are sorted by connection strength.

- **Params**:
  - `person_uuid` (string, **required**) -- from `search_people`
  - `query` (string) -- filter connectors by name
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: connectors with Connector ID, headline, company, and connection type (Direct/Inferred). No LinkedIn URLs or inferred connection reasons.

#### `get_second_degree_person_connectors`

Find second-degree introduction paths to a person — connectors who know an intermediary who knows the target.

- **Params**:
  - `person_uuid` (string, **required**) -- from `search_people`
  - `query` (string) -- filter connectors by name
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: paths with Connector ID, Intermediary ID, Target ID, names, and headlines. No profile URLs or path scores.

#### `get_second_degree_company_connectors`

Find second-degree introduction paths to people at a company.

- **Params**:
  - `company_uuid` (string, **required**) -- from `search_companies`
  - `query` (string) -- filter connectors by name
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: paths with Connector ID, Intermediary ID, Target ID, names, and headlines. No profile URLs or path scores.

#### `list_team_connectors`

List existing connectors on the team. Use this to see who is already a connector before adding new ones.

- **Params**:
  - `query` (string) -- filter connectors by name
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: connectors with Connector ID, type, headline, and company. No email, LinkedIn URL, origin, or relationship tags.

---

### Company Data

#### `get_company_investors`

Find investors in a specific company.

- **Params**:
  - `company_uuid` (string, **required**) -- from `search_companies`
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: investors with Investor ID, name, headline. No profile URLs.

#### `get_company_portfolio`

Find companies in an investor's portfolio.

- **Params**:
  - `company_uuid` (string, **required**) -- investor company UUID from `search_companies`
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: portfolio companies with Company ID, name, industry, headline. No profile URLs.

---

### Messaging

#### `search_messages`

Search messages for your team. Returns message IDs, subjects, status, and recipient counts. Use the returned message ID with `get_message_stats` for aggregate engagement summary, or with `update_draft_message` to modify a draft.

- **Params**:
  - `query` (string) -- search by recipient name/email
  - `status` (string) -- "draft", "sent", or "all" (default "sent")
  - `date_after` (string) -- ISO 8601 date filter
  - `date_before` (string) -- ISO 8601 date filter
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: messages with Message ID, subject, type (Draft/Sent), and recipient count. No recipient names, dates, or per-message engagement stats.

#### `get_message_stats`

Get aggregate engagement statistics for a specific sent message. Returns summary counts and rates without exposing recipient-level telemetry. Use `search_messages` first to find the message UUID.

- **Params**:
  - `message_uuid` (string, **required**) -- from `search_messages`
- **Returns**: Message ID, recipient count, and summary table (Sent/Opened/Clicked/Replied counts and rates). No sender email, sent-at timestamp, or per-recipient breakdown.

#### `create_draft_message`

Create a draft email message. Can be standalone or linked to an existing intro request. The draft will appear in the user's drafts tab for review before sending.

- **Params**:
  - `recipients` (array, **required**) -- each with `value` (email) and optional `label` (display name)
  - `subject` (string, **required**)
  - `body` (string, **required**) -- HTML supported
  - `cc` (array, optional) -- CC recipients (same format as recipients)
  - `bcc` (array, optional) -- BCC recipients (same format as recipients)
  - `intro_request_uuid` (string, optional) -- UUID of an existing intro request to link to
- **Returns**: subject, Draft UUID, recipient count, and status. No recipient list in response.

#### `update_draft_message`

Update an existing draft message's subject, body, or recipients.

- **Params**:
  - `message_uuid` (string, **required**) -- from `search_messages` or `create_draft_message`
  - `subject` (string, optional)
  - `body` (string, optional)
  - `recipients` (array, optional)
  - `cc` (array, optional)
  - `bcc` (array, optional)
- **Returns**: subject, Draft UUID, recipient count, and status. No recipient list in response.

#### `create_intro_request`

Create an intro request to get introduced to a target person or company through a facilitator (connector). Creates the intro request with a draft message ready for review.

- **Params**:
  - `target_global_person_uuid` (string) -- UUID from `search_people` (provide this OR target_global_company_uuid)
  - `target_global_company_uuid` (string) -- UUID from `search_companies`
  - `facilitator_uuid` (string, **required**) -- UUID from `get_company_connectors` or `get_person_connectors`
  - `intermediary_global_person_uuid` (string, optional) -- for second-degree intro requests
  - `request_reason` (string, optional) -- "sales_partnership", "fundraising", "networking", "job", "other_request_reason"
  - `context_message` (string, optional) -- additional context for the draft message body
  - `company_list_slug` (string, optional)
- **Returns**: target, facilitator, status, Intro Request UUID, and Draft Message UUID. No URL field.

---

### Lists

#### `manage_list`

Create, update, get, list, or delete people and company lists.

- **Params**: varies by action (create, update, get, list, delete)
- **Returns**: list details including UUID, name, type, and item count

#### `add_items_to_list`

Add people or companies to an existing list.

- **Params**:
  - `list_uuid` (string, **required**)
  - `items` (array, **required**) -- UUIDs of people or companies to add
- **Returns**: count of items added and any duplicates skipped

#### `remove_list_items`

Remove items from an existing list.

- **Params**:
  - `list_uuid` (string, **required**)
  - `items` (array, **required**) -- UUIDs of items to remove
- **Returns**: count of items removed

---

### Connection Management

#### `check_connection_status`

Returns the current status of the user's connections: Gmail connected (yes/no), total connections count, connectors count, and last connection import status.

- **Params**: none
- **Returns**: JSON with gmail connected status, connection totals, connectors on team, and last import status. No Gmail email address or import timestamps.

#### `list_connection_sources`

List available connection sources and their current status.

- **Params**: none
- **Returns**: available sources with connection status and descriptions. No upload timestamps.

#### `connect_gmail`

Initiates Gmail OAuth connection for the user. Returns an action block rendered by the frontend as an inline "Connect Gmail" button.

- **Params**: none
- **Returns**: instructions and action block, or confirmation that Gmail is already connected (no email shown)

#### `upload_connections`

Imports connections from a file. Admins can import on behalf of a team member using `on_behalf_of_user_uuid`.

- **Params**:
  - `source` (string) -- "linkedin", "google_contacts", or "other" (default "linkedin")
  - `upload_uuid` (string, optional) -- UUID of an already-attached file
  - `on_behalf_of_user_uuid` (string, optional) -- UUID of a team member (admin only, preferred)
  - `on_behalf_of` (string, optional) -- email or name of a team member (admin only, fallback)
- **Returns**: import confirmation without filenames or user identifiers

---

### Write

#### `add_connectors`

Add one or more connectors to the team. Requires the `write` OAuth scope.

- **Params**:
  - `connectors` (array, **required**, 1-25 items) -- each item must include exactly one of:
    - `global_person_uuid` (string) -- UUID from `search_people`
    - `linkedin_url` (string) -- LinkedIn profile URL
    - `email` (string) -- email address (pair with optional `first_name` and `last_name`)
  - `send_invite` (boolean, default false) -- send email invitation (email-based connectors only)
- **Returns**: results grouped into Added, Already Existed, and Failed. Each entry shows name and Connector ID (no emails). Failed entries show reference numbers without input values.

---

### Utility

#### `get_credit_balance`

Returns the team's current credit balance.

- **Params**: none
- **Returns**: credit balance details

---

## Common Workflows

### 1. Find who can intro to a company

```
search_companies(query: "Acme Corp")        --> Company ID
get_company_connectors(company_uuid: "...")  --> list of connectors with Connector IDs
get_connector_intros_at_company(            --> specific people the connector knows
  company_uuid: "...",
  connector_uuid: "..."
)
```

### 2. Find who can intro to a person

```
search_people(query: "Jane Smith")          --> Person ID
get_person_connectors(person_uuid: "...")    --> connectors who know Jane
```

### 3. Search with filters (helper-first pattern)

```
lookup_locations(query: "San Francisco")    --> "San Francisco, California, US"
search_companies_industries(query: "AI")    --> "Artificial Intelligence"
search_companies(
  location_names: ["San Francisco, California, US"],
  industries: ["Artificial Intelligence"]
)
```

### 4. Find people at specific companies

```
search_people_companies(query: "Stripe")    --> company UUID
search_people(
  current_company_uuids: ["..."],
  levels: ["VP", "Director"]
)
```

### 5. See all reachable people at a company

```
search_companies(query: "Acme Corp")        --> Company ID
get_company_connections(company_uuid: "...")  --> people + their connectors
```

### 6. Add a new connector

```
list_team_connectors(query: "Jane")         --> check if already exists
search_people(query: "Jane Smith")          --> get Person ID
add_connectors(connectors: [
  { global_person_uuid: "..." }
])
```

---

## Best Practices

- **Use helper tools before filtering.** Don't guess location names, industry values, or investor IDs. Call `lookup_locations`, `search_companies_industries`, or `search_companies_investors` first.
- **Chain UUIDs between tools.** Every UUID you pass to a tool should come from a previous tool call. Never fabricate UUIDs.
- **Handle pagination.** Results are capped at 25 per page. Check the pagination header (total count, current page, total pages) and request additional pages when needed.
- **Start broad, narrow incrementally.** Begin with a simple query, then add filters to refine results.
- **Check before writing.** Call `list_team_connectors` before `add_connectors` to avoid duplicates.
- **Prefer `global_person_uuid` over email** when adding connectors. UUID-based adds link to rich profile data; email-based adds create minimal records.
- **Investor IDs are integers, not UUIDs.** The `investors` filter on `search_companies` takes integer IDs from `search_companies_investors`.
- **Tool IDs are opaque.** IDs labeled "(for tool use)" are UUIDs meant for chaining between tools. Do not present them to users as clickable links.
- **Responses are markdown-formatted text.** All tool responses are returned as markdown in the MCP `text` content block.

---

## Error Handling

**Tool errors** (`isError: true` in the MCP response) indicate a problem with your input or the operation:
- `Missing required parameter: <param>` -- a required argument was not provided
- `Invalid UUID format for <param>` -- the value is not a valid UUID
- `Company or connector not found.` -- the UUID doesn't match any record
- `You do not have permission to add connectors to this team.` -- the user lacks the required team role
- `Too many connectors (maximum 25).` -- the `connectors` array exceeds the batch limit

**Protocol errors** (JSON-RPC `error` object) indicate a problem with the request itself:
- `-32700` Parse error -- invalid JSON
- `-32600` Invalid request -- missing `jsonrpc: "2.0"`, bad request structure, or insufficient OAuth scope (e.g. `Insufficient scope: the '<tool>' tool requires the 'write' scope`)
- `-32601` Method not found -- unrecognized JSON-RPC method
- `-32602` Unknown tool -- the tool name doesn't exist
- `-32603` Internal error -- unexpected server error

**Empty results are not errors.** A search that returns zero results will return a success response with a "Found 0 results" message or a "No connectors found" message.

---

## L402 Lightning Payment (Pay-per-Request)

The Secure MCP endpoint supports pay-per-request access via the L402 protocol. No API key or account needed -- pay with Bitcoin over Lightning.

### Discovery

Any unauthenticated request to the Secure MCP endpoint (`POST /secure_mcp`) returns `401 Unauthorized` with a `WWW-Authenticate` header pointing to the OAuth protected resource metadata, which includes the L402 flow.

Alternatively, any request to the Agent API (`/api/agent_api/v1/*`) without an `Authorization` header returns `402 Payment Required` with full L402 flow instructions and a `WWW-Authenticate: L402` header.

### How It Works

L402 issues a one-time token (macaroon) paired with a Lightning invoice. After payment, the client proves ownership by presenting the payment preimage alongside the token.

### Flow

#### 1. Request an invoice

```
POST /api/l402/request
Content-Type: application/json

{
  "team_id": 123,
  "credits_amount": 5
}
```

**Parameters:**
- `team_id` (integer, **required**) -- the team purchasing credits
- `credits_amount` (integer, **required**) -- credits to purchase (1 -- 10,000)

**Response (200):**
```json
{
  "macaroon": "<base64-encoded token>",
  "invoice": "lnbc500n1p...",
  "payment_hash": "0a1b2c3d...",
  "amount_sats": 500,
  "credits_amount": 5,
  "sats_per_credit": 100,
  "expires_at": "2026-03-25T17:30:00Z",
  "check_url": "https://getcabal.com/api/l402/check/0a1b2c3d..."
}
```

**Errors:**
- `400` -- `credits_amount` out of range (1 -- 10,000)
- `404` -- team not found or L402 not enabled for the team

#### 2. Pay the Lightning invoice

Pay the BOLT11 invoice (`invoice` field) using any Lightning wallet. The `amount_sats` field shows the exact cost.

**Pricing:** `amount_sats = credits_amount × sats_per_credit`. The current `sats_per_credit` rate is returned in the invoice response (default: 100).

#### 3. Poll for payment confirmation

```
GET /api/l402/check/{payment_hash}
```

No authentication required. Returns the current payment status.

**Response -- pending:**
```json
{
  "status": "pending",
  "expires_at": "2026-03-25T17:30:00Z"
}
```

**Response -- paid:**
```json
{
  "status": "paid",
  "paid_at": "2026-03-25T16:45:00Z",
  "amount_sats": 500,
  "credits_amount": 5,
  "preimage": "abcd1234..."
}
```

**Response -- expired:**
```json
{
  "status": "expired",
  "expires_at": "2026-03-25T17:30:00Z"
}
```

The endpoint actively checks the Lightning node when status is pending, so polling immediately after payment will usually return `paid`.

#### 4. Authenticate with the L402 token

Use the macaroon and preimage from the paid invoice to authenticate your request:

```
POST /secure_mcp
Authorization: L402 <macaroon>:<preimage>
Content-Type: application/json

{ "jsonrpc": "2.0", "id": 1, "method": "tools/call", "params": { ... } }
```

The `Authorization` header format is: `L402 <base64_macaroon>:<preimage_hex>`

#### 5. Verify a token (optional)

```
GET /api/l402/verify
Authorization: L402 <macaroon>:<preimage>
```

**Response (200):**
```json
{
  "valid": true,
  "team_id": 123,
  "credits_amount": 5,
  "status": "paid",
  "expires_at": "2026-03-25T17:30:00Z"
}
```

### Token Lifecycle

| Status | Meaning |
|--------|---------|
| `pending` | Invoice created, awaiting Lightning payment |
| `paid` | Payment received and preimage verified |
| `used` | Token consumed on first successful API request |
| `expired` | Invoice expired before payment was received |

### Important Notes

- **Single-use.** Each L402 token is consumed on the first successful request. After use, the token cannot be replayed.
- **Tokens expire.** Default expiry is 1 hour. Check `expires_at` in the invoice response.
- **Credits are issued immediately on payment.** When the Lightning payment settles, credits are added to the team's top-up balance automatically. The token then authorizes spending those credits.
- **`credits_amount` range:** 1 -- 10,000 per invoice.
- **`sats_per_credit`** is configurable server-side and returned in every invoice response so clients can display accurate pricing.
- **Idempotent payment processing.** Duplicate webhook deliveries and concurrent polling are handled safely -- credits are never double-issued.
- **Preimage verification.** The server verifies `SHA256(preimage) == payment_hash` before accepting any payment, preventing forged settlement data.

---
> Source: [getcabal/agent-skills](https://github.com/getcabal/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
