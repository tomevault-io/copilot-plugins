## outlook

> interact w/ Microsoft Office (O365) Outlook Email inbox


# Outlook Emails

## Overview

The `outlook-email` command is installed globally.

It is a dual-mode email management system for Office 365 (Microsoft Outlook) with offline analysis capabilities. 

It provides:

1. **Online Mode**: Pull emails from Outlook to local Markdown storage
2. **Offline Mode**: Query and manage the stored email database without Outlook connectivity

The system is designed for AI agents to integrate email processing into workflows, supporting:
- Batch email ingestion with deduplication
- Email marking (read/unread) with offline metadata
- Pattern analysis on stored emails
- Integration with other tools and scripts

## Core Concepts

### Email IDs
- **Outlook ID**: Long base64-encoded identifier from Microsoft Graph API
- **SHA1 Hash**: 40-character hex hash of Outlook ID, used as filename
- **Short ID**: First 6 characters of hash (e.g., `6498ce`), used for Git-like partial matching

Example:
```
Outlook ID: AQMkAGJjZGY31MGViLTEYi1iYWQ2LTBjNDBjZjAzYmE3MgBGAA...
SHA1 Hash:  6498cec18d676f0328ff649bf933e7ec3c0adb2b
Short ID:   6498ce
```

### Storage Format
Each email is stored as a Markdown file in `storage/` with YAML front matter containing metadata, and the HTML body in a code block:

```markdown
---
id: 'AQMkAGJ...'
subject: 'Project Status Update'
from:
  emailAddress:
    name: 'Alice Smith'
    address: 'asmith@company.com'
toRecipients:
  - emailAddress:
      name: 'Team'
      address: 'team@company.com'
receivedDateTime: '2026-01-05T10:30:00Z'
isRead: false
webLink: 'https://outlook.office365.com/owa/?ItemID=...'
body:
  contentType: html
_stored_id: '6498cec18d676f08ff64932bf93e7ec33c0adb2b'
_stored_at: '2026-01-05T18:05:13.476Z'
offline:
  read: true              # Custom offline metadata
  readAt: '2026-01-05T18:06:00.000Z'
---

# Project Status Update

```html
<html>
<head>...</head>
<body>
<p>Email content here...</p>
</body>
</html>
```
```

### ID Matching (Git-style)
All CLI commands support partial IDs. The system matches the longest unique prefix:

```bash
# These all refer to the same email:
outlook-email view 6498cec18d676f08ff64932bf93e7ec33c0adb2b  # Full (40 chars)
outlook-email view 6498cec18d676f08                            # 16 chars
outlook-email view 6498ce                                      # 6 chars (short)
outlook-email view 6498                                        # 4 chars (if unique)
outlook-email view 6498cec18d676f08ff64932bf93e7ec33c0adb2b.md  # Filename format
```

Error on ambiguity:
```
Error: Ambiguous ID "62". Matches: 62e8e2d5adb20b15..., 62b19cb17ec4628a...
```

## Online Mode: Fetching Emails from Outlook

### Command: `outlook-email pull`

**Purpose**: Fetch unread emails from Outlook, store locally as Markdown files, mark as read/processed in Outlook.

> **IMPORTANT**: When instructed to fetch more emails, use this command. **Always fetch exactly one email at a time** (`--limit 1`) unless specifically directed to pull more. This ensures controlled processing and avoids overwhelming the local storage with unreviewed emails.

**Command**:
```bash
outlook-email pull --since <date> [--limit N]
```

**Parameters**:
- `--since <date>`: Required. Fetch emails received on/after this date
  - Formats: `YYYY-MM-DD`, `yesterday`, `"7 days ago"`, `"1 day ago"`
- `--limit <n>`: Optional. Stop after processing N emails (default: no limit)

**Behavior**:
1. Fetches all unread emails from inbox since date
2. Paginates through results (50 per request)
3. Stops when reaching emails older than date
4. Skips already-stored files (deduplication via SHA1)
5. Stores each new email as Markdown under `storage/<id>.md`
6. Marks processed emails as read in Outlook
7. Moves processed emails to "Processed" folder in Outlook
8. Updates `webLink` with the new permalink after move
9. Prints progress: `✓ Stored: <id>+<subject...>`

**Examples**:

```bash
# RECOMMENDED: Pull exactly 1 new email (safest, most controlled)
outlook-email pull --since yesterday --limit 1

# Pull from specific date, one at a time
outlook-email pull --since 2026-01-01 --limit 1

# Pull from last week, one at a time
outlook-email pull --since "7 days ago" --limit 1

# Pull multiple emails (only when specifically instructed)
outlook-email pull --since yesterday --limit 5

# Pull all unread from past week (use with caution)
outlook-email pull --since "7 days ago"
```

**Output Example**:
```
Fetching unread emails since: 2026-01-05
Processing limit: 1
Found 14 unread emails.
✓ Stored: (71c95a7e429ff98a+NOTICE: LDAP Password Expiration)
  → Marking as read...
  → Moving to Processed folder...
  ✓ Updated in Outlook (webLink updated)

Reached processing limit of 1. Stopping.

Summary:
  Available:  14
  Processed:  1
  Written:    1
  Skipped:    0
```

**Best Practices**:
- Always use `--limit 1` unless you need to bulk import
- Use `--since yesterday` or `--since "1 day ago"` for recent emails
- After pulling, use `outlook-email list` to see the new email
- The `webLink` field in the stored file opens the email directly in Outlook

## Offline Mode: Analysis & Metadata

### Local Query & Management

All `outlook-email` commands work purely offline, reading/writing YAML files in `storage/`.

### Command: `outlook-email list [OPTIONS]`

**Purpose**: List emails with filtering and display

**Command**:
```bash
outlook-email list [--limit N] [--since DATE] [--folder NAME]
```

**Options**:
- `-l, --limit <n>`: Max results (default: 10)
- `--since <date>`: Filter emails after date (same formats as pull)
- `--folder <name>`: Only show emails from this source folder (e.g. `Processed`, `Inbox`)

**Output Format**:
```
📧 <total> emails (showing <n>):

   1.  <short_id>  <relative_date>  <sender_name>  <subject>
   2.  ...
```

**Output Example**:
```
📧 3 emails:

   1.  659520  01/05      Jane Smith                  Re: [eng/deploy-tools] Update config.yaml (PR #42)
   2.  2d6309  01/05      Bob Johnson                 Re: Cleanup unused staging environment
   3.  e20ba3  01/05      Carol Williams              Cleanup unused staging environment
```

**Use Cases**:
- Scan unread emails
- Find emails from specific period
- Show all (including read) for review
- Pipeline output to other tools

### Command: `outlook-email view <id>`

**Purpose**: Display full YAML content of an email

**Command**:
```bash
outlook-email view <id>
```

**Supports**: Partial IDs, full IDs, or filename format

**Output**: Complete YAML (can be ~2KB per email)

**Use Cases**:
- Extract email data for analysis
- Pipe to `jq` or other YAML parsers for processing
- Archive or backup emails
- Debug email structure

**Example**:
```bash
# View full email
outlook-email view 6498ce

# Parse specific field with yq
outlook-email view 6498ce | yq '.subject'

# Extract body and save
outlook-email view 6498ce | yq '.body.content' > email_body.html
```

### Command: `outlook-email read <id>`

**Purpose**: Mark an email as read (offline only)

**Command**:
```bash
outlook-email read <id>
```

**Effect**: 
- Adds/updates `offline.read: true` to YAML
- Adds `offline.readAt` timestamp
- Does NOT sync back to Outlook (offline only)

**Output**:
```
✓ Marked as read: 6498cec18d676f08ff64932bf93e7ec33c0adb2b
  Christmas Retreat
```

**Use Cases**:
- Mark emails as processed in analysis workflows
- Track which emails have been reviewed
- Pipeline processing: pull → process → mark read

### Command: `outlook-email unread <id>`

**Purpose**: Mark an email as unread (offline only, reverses read state)

**Command**:
```bash
outlook-email unread <id>
```

**Effect**:
- Removes `offline.read` and `offline.readAt` from YAML
- Reverts email to unread status in local database

**Output**:
```
✓ Marked as unread: 6498cec18d676f08ff64932bf93e7ec33c0adb2b
  Christmas Retreat
```

**Use Cases**:
- Recover from accidental marks
- Re-queue emails for processing
- Testing workflows

### Command: `outlook-email send`

**Purpose**: Send an email via SMTP (fast, headless, no browser).

**Command**:
```bash
outlook-email send --to <email> --subject <subject> [options] <file>
```

**Arguments**:
- `<file>`: Path to a file containing the email body. Use `-` to read from stdin.

**Options**:
- `--to <email>`: Recipient address(es), comma-separated (required)
- `--subject <text>`: Email subject (required)
- `--cc <email>`: CC address(es), comma-separated
- `--bcc <email>`: BCC address(es), comma-separated
- `--from <email>`: Override the From address (default: derived from the cached Graph token)
- `--html`: Treat body as raw HTML (sent as a `text/html` MIME part)
- `--smtp-host <host>`: SMTP host (default: `$SMTP_HOST`, else the sender domain's MX)
- `--smtp-port <port>`: SMTP port (default: `$SMTP_PORT`, else `25`)
- `-v, --verbose`: Show progress / SMTP debug output

**Output** (default, quiet):
```
✓ Sent: My Subject → recipient@example.com
```

**Behavior**:
1. Reads the body from `<file>` (or stdin) and derives the From identity (address + display name) from the cached Graph token's JWT claims
2. Builds a MIME message — `text/plain` by default, or multipart `text/html` with a plain-text fallback when `--html` is set
3. Resolves the SMTP host at runtime — `--smtp-host` flag, else `$SMTP_HOST`, else the MX record of the sender's email domain (no host is baked into the source)
4. Connects to that host (many internal relays accept mail with **no authentication** on the local network), opportunistically upgrading to STARTTLS if offered, then submits the message
5. Prints a one-line confirmation

> **Note**: When relying on the sender domain's MX, the resolved host may require corporate network/VPN access and may restrict delivery to internal recipients depending on the relay's policy. Override with `--smtp-host` / `$SMTP_HOST` as needed.

**Plain text example**:
```bash
echo "Hi Alice, let's sync tomorrow." > body.txt
outlook-email send --to alice@example.com --subject "Quick sync" body.txt
```

**HTML example**:
```bash
cat > email.html << 'EOF'
<p>Hi team,</p>
<ul>
  <li><b>Item one</b></li>
  <li><i>Item two</i></li>
</ul>
<p>See <a href="https://example.com">this link</a> for details.</p>
EOF
outlook-email send --to team@example.com --subject "Update" --html email.html
```

**Stdin example**:
```bash
echo "Quick note from a script." | outlook-email send --to alice@example.com --subject "Note" -
```

**HTML formatting support**: Full HTML is delivered as-is in a `text/html` MIME part, so any markup your email client renders works — `<b>`, `<i>`, `<u>`, `<ul>/<ol><li>`, `<a href>`, `<blockquote>`, `<code>`, `<table>`, inline styles, etc.

**Dependencies**:
- `python3` — must be on PATH (uses standard library only: `smtplib`, `email`)
- `dig` or `nslookup` — used to resolve the sender domain's MX when no host is given
- Network access to the resolved SMTP host (often corporate network/VPN)

## Workflow Scenarios

### Scenario 1: Daily Email Digest Processing

**Goal**: Pull emails daily, categorize by sender, mark processed

**Steps**:
```bash
# 1. Pull emails from past 24 hours
outlook-email pull --since "1 day ago"
# Output: Stored 12 new emails, marked as read/processed in Outlook

# 2. List new emails for review
outlook-email list --since yesterday --limit 20
# Output: Shows today's emails with sender/subject

# 3. Extract emails for batch analysis
outlook-email list --since yesterday --limit 100 > /tmp/emails.txt
# Can then pipe to custom analysis scripts
```

### Scenario 2: Alert Processing & Categorization

**Goal**: Auto-categorize and process system alerts

**Steps**:
```bash
# 1. Pull new alerts
outlook-email pull --since "1 hour ago" --limit 50

# 2. List and find critical emails
outlook-email list --limit 100 | grep -i "critical\|alert"

# 3. For each critical email, extract and analyze
for id in f86bca 8fb5bf a4ae87; do
  outlook-email view $id | yq '.subject, .from.emailAddress.address'
done

# 4. Mark processed
outlook-email read f86bca
outlook-email read 8fb5bf
outlook-email read a4ae87

# 5. Verify
outlook-email list --limit 10  # Confirm read count reduced
```

### Scenario 3: Time-based Filtering & Archive

**Goal**: Archive old processed emails, focus on recent

**Steps**:
```bash
# 1. List only recent unread (past 3 days)
outlook-email list --since "3 days ago" --limit 50

# 2. Export all emails from specific date for archival
outlook-email list --since 2026-01-01 --all --limit 1000 > archive_2026_01_01.txt

# 3. Extract emails for backup
for file in storage/*.yml; do
  id=$(basename "$file" .yml)
  outlook-email view "$id" >> backup_all_emails.yaml
done
```

### Scenario 4: Email Content Analysis Pipeline

**Goal**: Extract text from email bodies for NLP/analysis

**Steps**:
```bash
# 1. Pull recent emails
outlook-email pull --since yesterday

# 2. Extract HTML bodies for analysis (strip HTML tags since lynx may not be available)
outlook-email list --limit 20 | awk '{print $3}' | while read id; do
  echo "=== Email: $id ==="
  outlook-email view "$id" | yq '.body.content' | sed 's/<[^>]*>//g' | sed '/^[[:space:]]*$/d'
done

# 3. Or extract sender domains for analysis
outlook-email list --all --limit 100 | while read line; do
  outlook-email view "$line" | yq '.from.emailAddress.address' | awk -F@ '{print $2}'
done | sort | uniq -c
```

### Scenario 5: Test Email Pull with Limit

**Goal**: Safely test the pull process before full run

**Steps**:
```bash
# 1. Test with limit=1, check output
outlook-email pull --since yesterday --limit 1
# Shows: Found N emails, Processed 1

# 2. Check stored file exists
ls -lh storage/*.yml | tail -1

# 3. View the stored email
outlook-email list --limit 1

# 4. If good, run full pull without limit
outlook-email pull --since yesterday
```

### Scenario 6: Integration with External Tools

**Goal**: Feed email data into other systems (database, analytics, etc.)

**Steps**:
```bash
# 1. Export as JSON for database insertion
outlook-email list --all --limit 100 | while read line; do
  outlook-email view "$line"
done | yq -r -s 'map({id: ._stored_id, subject, from: .from.emailAddress.address, date: .receivedDateTime}) | .[] | @json' > emails.jsonl

# 2. Push to database
cat emails.jsonl | while read json; do
  psql -d email_db -c "INSERT INTO emails (id, subject, sender, date) VALUES ($(echo $json | jq -r '.id, .subject, .from, .date'))"
done

# 3. Or export CSV
outlook-email list --all | while read line; do
  id=$(echo "$line" | awk '{print $1}')
  outlook-email view "$id" | yq -r '[.from.emailAddress.address, .subject, .receivedDateTime] | @csv'
done > emails.csv
```

## Integration Patterns for AI Agents

### Pattern 1: Fetch & Analyze

```bash
# Typical agent workflow
1. outlook-email pull --since yesterday       # Get new emails
2. outlook-email list --limit 50             # Scan subjects
3. outlook-email view <id>                   # Get full content
4. <agent analyzes>
5. outlook-email read <id>                   # Mark processed
```

### Pattern 2: Incremental Processing

```bash
# Process in batches to avoid overwhelming
1. outlook-email pull --since yesterday --limit 20    # Get batch
2. for each email:
   a. outlook-email view <id> | extract content
   b. Send to AI for analysis
   c. Store result in database
   d. outlook-email read <id>  # Mark done
3. Repeat if more emails available
```

### Pattern 3: Error Recovery

```bash
# Handle processing failures gracefully
1. Pull emails
2. Try to process each
3. If error:
   - Log error + email ID
   - Do NOT mark as read (leave unread for retry)
4. outlook-email unread <id>  # Explicitly reset if needed
5. Retry later
```

### Pattern 4: Filtering by Content

```bash
# Find and process specific email types
1. outlook-email pull --since "7 days ago"
2. outlook-email list --all | grep "critical\|urgent"
3. For each match:
   - outlook-email view <id> > /tmp/email.yaml
   - Extract subject/sender/body via yq
   - Process with priority logic
   - outlook-email read <id>
```

## Data Access Examples

### Extract Sender Name
```bash
outlook-email view 6498ce | yq '.from.emailAddress.name'
# Output: "Server Administration"
```

### Extract All Recipients
```bash
outlook-email view 6498ce | yq '.toRecipients[].emailAddress.address'
# Output: 
# team@company.com
# otherteam@company.com
```

### Extract Email Subject
```bash
outlook-email view 6498ce | yq '.subject'
# Output: "[Admin] - LDAP Password Reset"
```

### Extract Received Date
```bash
outlook-email view 6498ce | yq '.receivedDateTime'
# Output: "2026-01-05T17:02:54Z"
```

### Check Read Status (Offline)
```bash
outlook-email view 6498ce | yq '.offline.read'
# Output: true (if read), null (if unread)
```

### Extract Body Text
```bash
# Strip HTML tags to get readable plain text (lynx may not be installed)
outlook-email view 6498ce | yq '.body.content' | sed 's/<[^>]*>//g' | sed '/^[[:space:]]*$/d'
```

## Common Tasks

### Task: List all emails from specific sender
```bash
outlook-email list --all --limit 1000 | grep "alice@company.com"
```

### Task: Count emails by sender
```bash
for id in $(ls storage/*.yml | xargs -I{} basename {} .yml | head -100); do
  outlook-email view "$id" | yq '.from.emailAddress.address'
done | sort | uniq -c | sort -rn
```

### Task: Find emails with specific keywords
```bash
outlook-email list --all | grep -E "urgent|critical|alert" | awk '{print $1}' | while read id; do
  outlook-email view "$id" | yq '.subject'
done
```

### Task: Export email thread data
```bash
outlook-email view 6498ce | yq '{subject, from: .from.emailAddress, sent: .receivedDateTime, recipients: .toRecipients[].emailAddress.address}'
```

### Task: Bulk mark emails as read
```bash
outlook-email list --limit 50 | awk '{print $1}' | while read id; do
  outlook-email read "$id"
done
```

## Error Handling

### Ambiguous ID
```bash
$ outlook-email view 62
Error: Ambiguous ID "62". Matches: 62e8e2d5adb20b15..., 62b19cb17ec4628a...
```
**Resolution**: Use more specific prefix (e.g., `62e8e2`)

### Email Not Found
```bash
$ outlook-email view abcdef
Email not found: abcdef
```
**Resolution**: Check ID is correct, or list emails to find ID

### Already Read/Unread
```bash
$ outlook-email read 6498ce
⊘ Email already marked as read: 6498cec18d676f08ff64932bf93e7ec33c0adb2b
```
**Resolution**: Normal - idempotent operation, safe to retry

## Tips for AI Agents

1. **Always check what's available first**: `outlook-email list --limit 10` to understand data volume 
2. **Use partial IDs**: Shorter `6498ce` instead of full `6498cec18d676f08...` for commands
3. **Pipe to tools**: `outlook-email view <id> | yq '.field'` to extract structured data
4. **Test with limit**: Use `--limit 5` or `--limit 1` when prototyping workflows
5. **Check for duplicates**: `--since` with pull handles deduplication automatically
6. **Mark processed**: Always `read` after processing to track state
7. **Use relative dates**: `yesterday`, `"7 days ago"` are clearer than exact dates
8. **Batch process carefully**: Large batches can be slow; consider `--limit` on long runs
9. **Parse YAML carefully**: Use `yq` with proper selectors; emails have nested objects
10. **Log errors**: Save failed email IDs for retry/debugging

## Architecture Notes

- **Storage**: All files in `storage/` are YAML (Git-friendly, human-readable)
- **No database**: Direct filesystem access via Node.js file API
- **Stateless**: Each command is independent; safe to run in parallel on different date ranges
- **Deduplication**: SHA1 hashing ensures same Outlook email = same storage file
- **Offline metadata**: `offline.*` fields are never overwritten by pull (pull only reads Outlook data)
- **Single direction**: `pull` syncs Outlook → storage; `read`/`unread` mark locally only (don't sync back)

---
> Source: [mikesmullin/outlook](https://github.com/mikesmullin/outlook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
