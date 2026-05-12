## ultraworker

> This file is a complete guide for Claude Code to automatically orchestrate the Ultrawork system.

# CLAUDE.md - Ultrawork Orchestration Guide

This file is a complete guide for Claude Code to automatically orchestrate the Ultrawork system.

## System Overview

Ultrawork is a 4-stage approval workflow system that detects Slack mentions and automatically processes tasks.

```
Slack Mention → Context Exploration → TODO Creation → [Stage 1 Approval]
                                                            ↓
                        Tech Spec Writing ← ─ ─ ─ ─ ─ ─ ─ ─ ┘
                              ↓
                       [Stage 2 Approval] → Code Implementation → [Stage 3 Approval]
                                                                        ↓
                              Final Report ← ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
                                   ↓
                            [Stage 4 Approval] → Done
```

---

## Orchestration Rules

### Automatic Trigger Conditions

Claude should automatically invoke the appropriate skill/agent in the following situations:

| User Request | Skill to Execute | Example |
|--------------|------------------|---------|
| "Sync Slack" | `/sync-slack` | "Update channel list" |
| "Analyze this thread" | `/explore-context` | "Get context for C0123-1706500000" |
| "Create TODO" | `/create-todo` | "Create tasks from EXP-001" |
| "Write spec" | `/write-spec` | "Write tech spec for TASK-001" |
| "Approve" | `/approve` | "Approve TASK-001" |
| "Reject" | `/reject` | "Request revision for TASK-001" |
| "Create report" | `/report` | "Generate final report for TASK-001" |
| "Create image with Gemini and post to X" | `/gemini-x-image-post` | "Generate image with Gemini and post to X" |
| "Create/manage cron job" | `/manage-cronjob` | "Create a cronjob to check threads every morning" |

### Automatic Workflow Progression

After completing each stage, Claude should guide or automatically execute the next step:

```python
# Pseudocode
if current_stage == "exploration_complete":
    guide("Would you like to create a TODO? `/create-todo {exploration_id}`")

if current_stage == "todo_approved":
    if workflow_type == "full":
        guide("Writing tech spec: `/write-spec {task_id}`")
    else:
        guide("After completing work, generate report with `/report {task_id}`")

if current_stage == "tech_spec_approved":
    guide("Start code implementation. After completion, `/approve {task_id}`")

if current_stage == "code_work_approved":
    guide("Generating final report: `/report {task_id}`")

if current_stage == "final_report_approved":
    complete_processing()
```

### Execution Persistence and Progress Sharing (Important)

Regardless of model type (e.g., Opus 4.6), Claude should **never stop in an idle state after sending a progress message**.

#### Required Rules

1. **Progress Message = Start Signal**
   - Messages like "Proceeding", "Processing" are considered execution starts, not endings.
   - Continue immediately with the next task after such messages.

2. **Continue After Slack Sharing**
   - Even after posting intermediate results/reports to Slack, continue if there's remaining work.
   - Slack sharing is a checkpoint, not a termination condition.

3. **Maintain Progress Sharing Loop**
   - Briefly share current status and next action with the user after each major step.
   - After sharing, execute the next step immediately without waiting.

4. **Only Valid Termination Conditions**
   - All TODO/approval stages are complete
   - User explicitly requests to stop
   - External input/permission is absolutely required to proceed (in this case, clearly request the needed input)

#### Execution Loop Pseudocode

```python
send_progress_message()

while remaining_work_exists:
    execute_next_task()
    share_progress("current_status", "next_action")

    if slack_sharing_complete and remaining_work_exists:
        share_progress("Slack sharing complete", "continuing remaining work")
        continue

send_final_completion_report()
```

---

## Agent Session Tracking Rules

### Agent Roles

Agent roles transition according to workflow progression:

| Role | Description | Related Skills |
|------|-------------|----------------|
| `RESPONDER` | Initial response (immediately after mention detection) | - |
| `PLANNER` | Context exploration, TODO creation | `/explore-context`, `/create-todo` |
| `SPEC_WRITER` | Tech spec writing | `/write-spec` |
| `IMPLEMENTER` | Code implementation | Code writing/modification |
| `REPORTER` | Final reporting | `/report` |

### Role Transition Rules by Skill

| Skill | Start Role | End Role |
|-------|------------|----------|
| `/explore-context` | RESPONDER/Any | PLANNER |
| `/create-todo` | PLANNER | PLANNER |
| `/write-spec` | PLANNER | SPEC_WRITER |
| `/approve` (spec) | SPEC_WRITER | IMPLEMENTER |
| `/approve` (code) | IMPLEMENTER | REPORTER |
| `/report` | IMPLEMENTER | REPORTER |
| `/approve` (report) | REPORTER | (Session complete) |

### Session Creation Timing

Agent sessions are automatically created in the following situations:

1. **On Slack mention detection**: New thread mentions create new sessions
2. **On skill invocation**: Automatically created if no session exists
3. **Explicit creation**: Manual creation from dashboard

### Session States

| State | Description |
|-------|-------------|
| `active` | Currently working on task |
| `waiting_feedback` | Waiting for human-in-the-loop feedback |
| `completed` | All stages complete |
| `failed` | Stopped due to error |

### Human-in-the-Loop Feedback Types

| Type | Description | Usage |
|------|-------------|-------|
| `approval` | Yes/No approval | Stage approvals |
| `input` | Free-form text input | Additional information requests |
| `choice` | Selection from options | Decision making |
| `review` | Review and comment | Spec/code review |

### Dashboard API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/agent/sessions` | GET | List sessions |
| `/api/agent/sessions/{id}` | GET | Get session details |
| `/api/agent/sessions/{id}/timeline` | GET | Session event timeline |
| `/api/workflows/{session_id}/graph` | GET | Workflow graph |
| `/api/workflows/{session_id}/stream` | GET | Real-time workflow updates (SSE) |
| `/api/executions` | GET | List skill executions |
| `/api/executions/{id}` | GET | Skill execution details |
| `/api/feedback/pending` | GET | Pending feedback requests |
| `/api/feedback/{id}/respond` | POST | Submit feedback response |
| `/api/memory/{session_id}` | GET | Session context memory |

---

## Slack MCP Tool Usage

### Token Type Tool Availability

**Important**: Available MCP tools differ based on the configured token type.

| Token Type | Available Tools | Description |
|------------|-----------------|-------------|
| **Personal Token (xoxc)** | `mcp__slack__*` | Personal Slack account based, search/read capable |
| **Bot Token (xoxb)** | `mcp__slack-bot-mcp__*` | Bot app based, reactions/message sending capable |
| **Both configured** | Both sets | Primary → Fallback strategy available |

**Check current configuration**: Refer to `slack_token_type` in `.ultrawork-setup-state.json`

- `personal` → Only `mcp__slack__*` tools available
- `bot` → Only `mcp__slack-bot-mcp__*` tools available
- Both present → Fallback strategy applicable

### Required: Load Tools

Before Slack operations, always load tools with ToolSearch:

```
# For personal token setup
ToolSearch: "slack"

# For bot token setup
ToolSearch: "+slack-bot"
```

### Fallback Strategy (Only When Both Are Configured)

**Note**: This strategy only applies when **both personal and bot tokens are configured**.
If only one token is configured, only that token's MCP tools are available.

When `mcp__slack__*` tools fail, use `mcp__slack-bot-mcp__*` tools as fallback.

```
ToolSearch: "+slack-bot"
```

### Tool Mapping (Primary → Fallback)

| Function | Primary (mcp__slack__) | Fallback (mcp__slack-bot-mcp__) |
|----------|------------------------|----------------------------------|
| Channel list | `slack_list_conversations` | `slack_list_channels` |
| User list | `slack_list_users` | `slack_get_users` |
| Thread fetch | `slack_get_thread` | `slack_get_thread_replies` |
| Send message | `slack_send_message` | `slack_post_message` |
| Thread reply | `slack_send_message(thread_ts)` | `slack_reply_to_thread` |
| Channel history | `slack_conversations_history` | `slack_get_channel_history` |
| User info | `slack_users_info` | `slack_get_user_profile` |
| Add reaction | - | `slack_add_reaction` |
| File upload | Python SDK (`SlackUploader`) | Python SDK (`SlackUploader`) |

### Slack File Upload

MCP tools don't support file upload, so use the Python SDK directly.

**Module location**: `src/ultrawork/slack/uploader.py`

**Class**: `SlackUploader`

```python
from ultrawork.slack import SlackUploader, upload_to_slack

# Method 1: Use class
uploader = SlackUploader(token="xoxb-...")  # Or use SLACK_BOT_TOKEN env var

# Upload file
result = uploader.upload_file(
    file_path="/path/to/file.png",
    channel_id="C0123456789",
    thread_ts="1706500000.000000",  # Optional: upload to thread
    title="File title",
    initial_comment="File description",
)

# Upload text content as file
result = uploader.upload_content(
    content="print('Hello, World!')",
    channel_id="C0123456789",
    filename="example.py",
    filetype="python",  # Optional: for syntax highlighting
    thread_ts="1706500000.000000",
)

# Upload multiple files
results = uploader.upload_multiple(
    file_paths=["/path/to/file1.png", "/path/to/file2.pdf"],
    channel_id="C0123456789",
    thread_ts="1706500000.000000",
    initial_comment="Comment displayed with first file",
)

# Method 2: Use convenience function
result = upload_to_slack(
    file_path="/path/to/file.png",
    channel_id="C0123456789",
    thread_ts="1706500000.000000",
    title="File title",
    comment="File description",
)
```

**Return value**:
```python
# On success
{
    "ok": True,
    "file_id": "F0123456789",
    "file_url": "https://files.slack.com/...",
    "filename": "example.py",
}

# On failure
{
    "ok": False,
    "error": "Error message",
    "error_code": "file_not_found",
}
```

**Required environment variable**: `SLACK_BOT_TOKEN` (or pass token argument to constructor)

**Required Bot permissions**: `files:write`, `files:read`

### Slack File Download

Download and process files/images from Slack threads for Claude consumption.

**Module location**: `src/ultrawork/slack/downloader.py`

**Class**: `SlackFileDownloader`

```python
from ultrawork.slack import SlackFileDownloader, download_thread_files

# Method 1: Use class
downloader = SlackFileDownloader(
    token="xoxc-...",  # Or uses SLACK_TOKEN env var
    cookie="xoxd-...",  # Or uses SLACK_COOKIE env var (required for xoxc tokens)
    download_dir="/path/to/save/files",  # Optional, uses temp dir if not set
)

# Get all files from a thread
files = downloader.get_thread_files(channel_id="C0123456789", thread_ts="1706500000.000000")

# Download and process all files
processed = downloader.download_all(files)

# Format for Claude prompt context
formatted_text = downloader.format_for_claude(processed)

# Method 2: Convenience function
processed, formatted_text = download_thread_files(
    channel_id="C0123456789",
    thread_ts="1706500000.000000",
)
```

**Supported file types**:

| Category | Extensions | Processing |
|----------|-----------|------------|
| Images | PNG, JPG, GIF, WEBP, BMP | Saved locally, use Read tool to view |
| PDFs | PDF | Saved locally, use Read tool to read |
| Text | CSV, TXT, JSON, YAML, MD, code files | Content included inline |
| Archives | ZIP | Extracted, each file processed |
| Binary | Others | Metadata only |

**Integration**: Thread files are automatically downloaded during mention processing (SDK poller).
File content is included in the prompt context under `## Thread Attachments`.

### Available Primary Tools (mcp__slack__)

| Tool | Purpose | Call Example |
|------|---------|--------------|
| `mcp__slack__slack_health_check` | Connection check | `mcp__slack__slack_health_check()` |
| `mcp__slack__slack_list_conversations` | Channel list | `mcp__slack__slack_list_conversations(types: "public_channel,private_channel", limit: 200)` |
| `mcp__slack__slack_list_users` | User list | `mcp__slack__slack_list_users(limit: 200)` |
| `mcp__slack__slack_get_thread` | Thread fetch | `mcp__slack__slack_get_thread(channel_id: "C0123", thread_ts: "1706500000.000000")` |
| `mcp__slack__slack_search_messages` | Message search | `mcp__slack__slack_search_messages(query: "keyword", count: 20)` |
| `mcp__slack__slack_send_message` | Send message | `mcp__slack__slack_send_message(channel_id: "C0123", text: "message", thread_ts: "1706500000")` |
| `mcp__slack__slack_users_info` | User info | `mcp__slack__slack_users_info(user_id: "U0123")` |

### Fallback Tools (mcp__slack-bot-mcp__)

| Tool | Purpose | Call Example |
|------|---------|--------------|
| `mcp__slack-bot-mcp__slack_list_channels` | Channel list | `mcp__slack-bot-mcp__slack_list_channels()` |
| `mcp__slack-bot-mcp__slack_get_users` | User list | `mcp__slack-bot-mcp__slack_get_users()` |
| `mcp__slack-bot-mcp__slack_get_thread_replies` | Thread fetch | `mcp__slack-bot-mcp__slack_get_thread_replies(channel_id: "C0123", thread_ts: "1706500000.000000")` |
| `mcp__slack-bot-mcp__slack_get_channel_history` | Channel history | `mcp__slack-bot-mcp__slack_get_channel_history(channel_id: "C0123", limit: 50)` |
| `mcp__slack-bot-mcp__slack_post_message` | Send message | `mcp__slack-bot-mcp__slack_post_message(channel_id: "C0123", text: "message")` |
| `mcp__slack-bot-mcp__slack_reply_to_thread` | Thread reply | `mcp__slack-bot-mcp__slack_reply_to_thread(channel_id: "C0123", thread_ts: "1706500000", text: "reply")` |
| `mcp__slack-bot-mcp__slack_get_user_profile` | User info | `mcp__slack-bot-mcp__slack_get_user_profile(user_id: "U0123")` |
| `mcp__slack-bot-mcp__slack_add_reaction` | Add reaction | `mcp__slack-bot-mcp__slack_add_reaction(channel_id: "C0123", timestamp: "1706500000", emoji: "thumbsup")` |

### Search Fallback Strategy

When `mcp__slack__slack_search_messages` fails, fall back to channel history:

```
# Primary: Search
mcp__slack__slack_search_messages(query: "keyword", count: 20)

# Fallback: Fetch channel history and filter
mcp__slack-bot-mcp__slack_get_channel_history(channel_id: "C0123", limit: 100)
→ Filter results by keyword matching
```

### Search Query Syntax (Primary Only)

```
from:@user        - Messages from specific user
in:#channel       - Messages in specific channel
to:@bot           - Mentions to bot
"exact phrase"    - Exact phrase match
keyword1 keyword2 - AND search
```

---

## Long-Term Memory (LTM) System

### Overview

The agent manages two types of long-term memory records:
- **RequestRecord (req-*)**: Captures who/when/where/what/why/how of a request (5W1H)
- **WorkRecord (work-*)**: Captures what was done, where, and why (actions + artifacts)

Records are connected through:
- **FacetKey** implicit edges (`k/<facet>/<value>` format)
- **ShallowLink** explicit edges between records
- **CausalLink** causal chain tracking (caused_by, leads_to, blocks, supersedes)

### Directory Structure

```
data/memory/
├── requests/              # RequestRecord files (req-YYYYMMDD-NNNN.md)
├── works/                 # WorkRecord files (work-YYYYMMDD-req-...-NN.md)
├── index/
│   └── facet_index.yaml   # Machine-readable inverted index
├── indexes/               # Agent-readable QMD indexes
│   ├── who.md             # Person -> records mapping
│   ├── what.md            # Topic/entity -> records mapping
│   ├── where.md           # Location (channel/file) -> records mapping
│   ├── links.md           # Intent-work connections + causal chains
│   └── timeline.md        # Chronological record listing
└── channel_history/       # Quarterly channel summaries (existing)
```

### When to Save (4-Signal Gate)

Evaluate 4 boolean signals. Threshold varies by record type.

| Signal | Question |
|---|---|
| novelty | Is this new information not already in memory? |
| actionability | Did this lead to a concrete action or decision? |
| persistence | Is this useful beyond the current session? |
| connectedness | Can this connect to existing records? |

- **RequestRecord**: 2/4 signals required to save
- **WorkRecord**: 1/4 signals required to save (lower bar because actions are inherently valuable)

**Save triggers (REQUIRED - you MUST save at these points):**
1. After `/explore-context` completion -> you MUST save a RequestRecord using `/remember`
2. After code changes or architectural decisions -> you MUST save a WorkRecord using `/remember`
3. After file modifications or test writing -> you MUST save a WorkRecord using `/remember`
4. When discovering connections to past records -> you MUST update causal links using `/remember link`

**Do NOT save:** Simple file existence checks, re-confirming already-recorded info, official doc lookups, empty search results.

### How to Save (`/remember`)

```bash
# Save a request record
/remember req --who U0123 --where eng-common --what "Auth refactoring" \
  --topics authentication,middleware --why "SRP violation" \
  --intents "JWT separation;Chain improvement;Test writing"

# Save a work record
/remember work --request-ref req-20260225-0001 \
  --step-ref "req-20260225-0001#step1" \
  --purpose "Extract JWT verifier module" \
  --action extract_module --topics jwt,authentication \
  --files-modified src/auth/middleware.py \
  --files-created src/auth/jwt_verifier.py

# Add a causal link between records
/remember link --source req-20260225-0001 --target req-20260220-0003 \
  --relation caused_by --reason "Perf improvement revealed structural issues"

# Update an existing record
/remember update --id req-20260225-0001 --intent-status step1=done
```

### How to Search (`/recall`)

```bash
# Search by topic
/recall --what authentication

# Search by person
/recall --who U0123456789

# Search by file/location
/recall --where src/auth/middleware.py

# Combined search (multiple dimensions = more precise)
/recall --what authentication --who U0123 --when 2026-Q1

# Trace causal chain (root cause)
/recall --trace-cause work-20260225-0001

# Trace effect (blast radius)
/recall --trace-effect req-20260225-0001
```

### How to Use Memory

**Automatic usage (REQUIRED at these points):**
1. Before `/explore-context`: you MUST `/recall` related past requests/work
2. Before `/write-spec`: you MUST `/recall` related file locations and architecture
3. Before code implementation: you MUST `/recall` past modifications to the same files
4. On new requests: you MUST `/recall` same requester's history for patterns

**Structural inference rules:**
1. **Time proximity**: Same date prefix (req-20260225-*) records are likely from the same session
2. **ID-based search**: Found REQ? Glob for WRK-same-date-* to find related work
3. **Intent chains**: Use links.md to find all work records for a request's intents
4. **File-based backtracking**: Use where.md to find all records touching a file
5. **Topic clustering**: Use what.md to find related topic clusters

### Causal Chain Types

| Type | Meaning | Example |
|---|---|---|
| caused_by | This record was triggered by another | Performance issue revealed structural problem |
| leads_to | This record's result triggered another | JWT extraction led to chain improvement |
| blocks | Must complete before another can start | Module split required before testing |
| supersedes | This record replaces an older one | Expanded scope replaces original request |

---

## Skill Detailed Execution Guide

### 1. /sync-slack - Slack Synchronization

**Trigger condition**: When user requests Slack information sync

**Execution sequence**:
```
1. ToolSearch: "slack" (load tools)
2. mcp__slack__slack_health_check() (verify connection)
3. mcp__slack__slack_list_conversations(types: "public_channel,private_channel", limit: 200)
   → On failure: mcp__slack-bot-mcp__slack_list_channels()
4. mcp__slack__slack_list_users(limit: 200)
   → On failure: mcp__slack-bot-mcp__slack_get_users()
5. Write: data/registry/channels.yaml (save channels)
6. Write: data/registry/users.yaml (save users)
7. Output summary
```

**Output format**:
```yaml
# data/registry/channels.yaml
updated_at: "2026-01-29T10:00:00"
channels:
  C0123456789:
    name: "engineering"
    type: "public"
    purpose: "Engineering team discussions"
    is_monitored: false
    member_count: 15
```

---

### 2. /explore-context - Context Exploration

**Trigger conditions**:
- User requests thread/keyword analysis
- Automatic exploration after mention detection

**Execution sequence**:
```
1. ToolSearch: "slack" (load tools)
2. Trigger analysis:
   - If thread ID: mcp__slack__slack_get_thread()
     → On failure: mcp__slack-bot-mcp__slack_get_thread_replies()
   - If keyword: mcp__slack__slack_search_messages()
     → On failure: mcp__slack-bot-mcp__slack_get_channel_history() + keyword filtering
3. Extract keywords (technical terms, project names from messages)
4. Search related conversations: mcp__slack__slack_search_messages(query: "extracted keywords")
   → On failure: mcp__slack-bot-mcp__slack_get_channel_history() + filtering for each monitored channel
5. Fetch highly relevant thread details: mcp__slack__slack_get_thread()
   → On failure: mcp__slack-bot-mcp__slack_get_thread_replies()
6. Save exploration results: Write data/explorations/EXP-{YYYY-MMDD}-{NNN}.md
7. Guide next steps
```

**Exploration ID generation rule**:
```
EXP-{year}-{monthday}-{sequence}
Example: EXP-2026-0129-001
```

**Relevance score calculation**:
```
+0.3: Exact keyword match
+0.2: Same participant
+0.2: Within last 7 days
+0.1: Same channel
+0.1: 5+ replies
```

**Exploration termination conditions**:
- Relevance < 0.3
- Depth > 5
- No new information for 2 consecutive rounds

---

### 3. /create-todo - TODO Creation

**Trigger condition**: User requests TODO creation after exploration complete

**Execution sequence**:
```
1. Read: data/explorations/EXP-{id}.md (exploration results)
2. Determine workflow type:
   - Code changes needed → full
   - Documentation/config only → simple
3. Generate TODO items (recommendations → specific items)
4. Write: data/tasks/TASK-{YYYY-MMDD}-{NNN}.md
5. ToolSearch: "slack" (load tools)
6. mcp__slack__slack_send_message() (approval request)
   → On failure: mcp__slack-bot-mcp__slack_post_message() or slack_reply_to_thread()
7. Output result
```

**Task ID generation rule**:
```
TASK-{year}-{monthday}-{sequence}
Example: TASK-2026-0129-001
```

**Workflow type stages**:

| Type | Stages |
|------|--------|
| full | todo → tech_spec → code_work → final_report → done |
| simple | todo → final_report → done |

---

### 4. /write-spec - Tech Spec Writing

**Trigger condition**: After TODO approval (full workflow only)

**Prerequisite check**:
```
workflow.current_stage == "todo"
workflow.stages.todo.status == "approved"
```

**Execution sequence**:
```
1. Read: data/tasks/TASK-{id}.md (check prerequisites)
2. Read: data/explorations/EXP-{id}.md (context)
3. Codebase analysis:
   - Glob: **/*.py
   - Grep: relevant keywords
4. Write: data/specs/TASK-{id}_spec.md (tech spec)
5. Edit: data/tasks/TASK-{id}.md (update stage)
6. ToolSearch: "slack"
7. mcp__slack__slack_send_message() (Stage 2 approval request)
   → On failure: mcp__slack-bot-mcp__slack_post_message() or slack_reply_to_thread()
```

**Required spec sections**:
1. Problem Statement
2. Proposed Solution
3. Implementation Details (with code)
4. Files Summary
5. Testing Strategy
6. Rollout Plan
7. Risks and Mitigations

---

### 5. /approve - Approval Processing

**Trigger condition**: User requests approval for specific stage

**Execution sequence**:
```
1. Read: data/tasks/TASK-{id}.md
2. Check current stage
3. Record approval:
   - status: "approved"
   - approved_at: current time
   - approved_by: user ID
4. Transition to next stage
5. Edit: data/tasks/TASK-{id}.md
6. Guide next steps (or complete processing)
```

**Next actions by stage**:

| Approved Stage | Next Action |
|----------------|-------------|
| todo (full) | Guide `/write-spec` |
| todo (simple) | Guide `/report` after work |
| tech_spec | Guide code implementation (claude -p / codex -p) |
| code_work | Guide `/report` |
| final_report | Complete processing + Slack notification |

---

### 6. /reject - Rejection Processing

**Trigger condition**: User requests revision

**Execution sequence**:
```
1. Read: data/tasks/TASK-{id}.md
2. Record rejection:
   - status: "rejected"
   - rejected_at: current time
   - rejection_reason: reason
   - revision_count: +1
3. Edit: data/tasks/TASK-{id}.md
4. ToolSearch: "slack"
5. mcp__slack__slack_send_message() (revision request notification)
   → On failure: mcp__slack-bot-mcp__slack_post_message() or slack_reply_to_thread()
6. Guide revision process
```

**Maximum revision count**: 3 (escalate if exceeded)

---

### 7. /report - Final Report

**Trigger conditions**:
- full: After code_work completion
- simple: After todo approval and work completion

**Execution sequence**:
```
1. Read: data/tasks/TASK-{id}.md (check prerequisites)
2. Read: data/explorations/EXP-{id}.md
3. Read: data/specs/TASK-{id}_spec.md (full only)
4. Collect work evidence (git log, test results, etc.)
5. Edit: data/tasks/TASK-{id}.md (add report section)
6. ToolSearch: "slack"
7. mcp__slack__slack_send_message() (final approval request)
   → On failure: mcp__slack-bot-mcp__slack_post_message() or slack_reply_to_thread()
```

**Required report sections**:
- Summary
- Problem Solved (Before/After)
- Changes Made
- Test Results
- Verification Checklist

---

### 8. /gemini-x-image-post - Gemini Image Generation and X Post

**Trigger conditions**:
- User requests to create image with Gemini and post to X(Twitter)

**Execution sequence**:
```
1. Collect request requirements (goal, style, ratio, text, hashtags)
2. Write 20+ line prompt
3. Generate image with Gemini (gemini-3-pro-image-preview, fallback if needed)
4. Save image + generate ALT text
5. Upload media to X API and post
6. Share result summary and link/ID
```

---

## File Structure and Formats

### Directory Structure

```
data/
├── registry/           # Slack registry
│   ├── channels.yaml   # Channel list
│   └── users.yaml      # User list
├── threads/            # Thread records
│   └── {channel_id}/
│       └── {thread_ts}.md
├── explorations/       # Exploration results
│   └── EXP-YYYY-MMDD-NNN.md
├── tasks/              # Task records
│   └── TASK-YYYY-MMDD-NNN.md
├── specs/              # Tech specs
│   └── TASK-YYYY-MMDD-NNN_spec.md
├── cronjobs/           # Cron job definitions
│   ├── CRON-YYYY-MMDD-NNN.yaml
│   └── logs/           # Execution logs
└── logs/               # Logs
```

### File Format: YAML Frontmatter + Markdown

```yaml
---
# YAML metadata
task_id: "TASK-2026-0129-001"
title: "Task title"
workflow:
  type: "full"
  current_stage: "todo"
  stages:
    todo:
      status: "pending"
---

# Markdown body

## TODO
- [ ] Item 1
- [ ] Item 2
```

---

## State Value Definitions

### Workflow Stages (current_stage)

| Value | Description |
|-------|-------------|
| `todo` | TODO stage |
| `tech_spec` | Tech spec stage |
| `code_work` | Code work stage |
| `final_report` | Final report stage |
| `done` | Complete |

### Stage Status (stages.*.status)

| Value | Description |
|-------|-------------|
| `pending` | Waiting |
| `in_progress` | In progress |
| `approved` | Approved |
| `rejected` | Rejected |
| `completed` | Completed |
| `skipped` | Skipped |

### Severity

| Value | Description |
|-------|-------------|
| `critical` | Critical |
| `high` | High |
| `medium` | Medium |
| `low` | Low |

---

## Slack Message Templates (Block Kit)

All Slack messages use **Block Kit** format for interactive, rich messages. The `BlockKitBuilder` class in `src/ultrawork/slack/block_kit.py` provides builders for all message types.

### How to Send Block Kit Messages

When sending via MCP tools, pass both `text` (fallback) and `blocks` (JSON string):

```python
mcp__slack__slack_send_message(
  channel_id: "C0123",
  thread_ts: "1706500000",
  text: "Fallback text for notifications",
  blocks: '[{"type": "header", ...}, {"type": "section", ...}, {"type": "actions", ...}]'
)
```

When sending via Python SDK:

```python
from ultrawork.slack.block_kit import BlockKitBuilder, send_block_message

message = BlockKitBuilder.build_todo_approval(task_id=..., title=..., ...)
send_block_message(client, channel_id, message, thread_ts=thread_ts)
```

### Stage 1 Approval Request (TODO)

Uses `BlockKitBuilder.build_todo_approval()`. Includes:
- Header: `:clipboard: TODO List Review Request`
- Fields: Task, Type, Task ID, Based on
- TODO items list
- Already Decided section (if applicable)
- Estimated Effort
- **Reaction guide**: :+1: = Approve / :-1: = Request Changes
- Context footer with task ID and stage number

### Stage 2 Approval Request (Tech Spec)

Uses `BlockKitBuilder.build_spec_approval()`. Includes:
- Header: `:page_facing_up: Tech Spec Review Request`
- Fields: Task, Task ID, Spec File, Stage
- Key Changes list
- Test Plan
- **Reaction guide**: :+1: = Approve / :-1: = Request Changes
- Context footer

### Final Approval Request

Uses `BlockKitBuilder.build_final_report_approval()`. Includes:
- Header: `:clipboard: Final Report`
- Fields: Task, Task ID, Stage
- Summary, Key Results, Changes, Verification sections
- **Reaction guide**: :+1: = Final Approval / :-1: = Request Changes
- Context footer

### Completion Notification

Uses `BlockKitBuilder.build_completion_notification()`. Includes:
- Header: `:checkered_flag: Task Complete`
- Task name and ID
- Duration and Approval count fields
- Context footer with timestamp

### Approval via Reactions (No Interactivity URL Required)

Approval and rejection are handled entirely via Slack emoji reactions.
No Slack app Interactivity Request URL configuration is needed.

The SDK poller daemon (`src/ultrawork/slack/reaction_approval.py`) automatically:
- Monitors pending approval messages for reactions every poll cycle
- :+1: / :white_check_mark: reactions trigger approval
- :-1: / :x: reactions trigger rejection
- Sends Block Kit notification after processing
- Triggers next workflow stage automatically

---

## Error Handling

### Slack MCP Connection Failure → Fallback to Slack Bot MCP

```
⚠️ Slack MCP connection failed - Attempting fallback

Cause: {error_message}

Automatic fallback:
1. ToolSearch: "+slack-bot" (load Slack Bot MCP tools)
2. Retry with alternative tools:
   - slack_list_conversations → slack_list_channels
   - slack_list_users → slack_get_users
   - slack_get_thread → slack_get_thread_replies
   - slack_send_message → slack_post_message / slack_reply_to_thread
   - slack_search_messages → slack_get_channel_history + filtering
   - slack_users_info → slack_get_user_profile
```

### Complete Connection Failure

```
❌ Slack connection failed (both Primary and Fallback failed)

Cause: {error_message}

Resolution:
1. Primary: Verify Slack token
2. Fallback: Verify Slack Bot token
3. Check network connection
4. Re-run `mcp__slack__slack_health_check()`
```

### Prerequisite Not Met

```
❌ Prerequisite not met

Current stage: {current_stage}
Required status: {required_status}
Actual status: {actual_status}

{action_guidance}
```

### File Not Found

```
❌ File not found

File: {file_path}

Check:
1. Verify file path
2. Verify previous stage completion
```

---

## Quick Reference

### Full Workflow Example

```bash
# 1. Sync Slack
/sync-slack

# 2. Explore thread
/explore-context C0123456789-1706500000.000000

# 3. Create TODO
/create-todo EXP-2026-0129-001

# 4. After TODO approval, write spec
/approve TASK-2026-0129-001
/write-spec TASK-2026-0129-001

# 5. After spec approval, implement code
/approve TASK-2026-0129-001
claude -p "Implement according to data/specs/TASK-2026-0129-001_spec.md"

# 6. After code completion, approve
/approve TASK-2026-0129-001

# 7. Generate final report
/report TASK-2026-0129-001

# 8. Final approval
/approve TASK-2026-0129-001

# 9. (Optional) Set up follow-up cron job
/manage-cronjob create "Thread Monitor" --schedule weekday --at 09:00 --action check_thread_reactions
```

### Cron Job Examples

```bash
# Create a weekday morning thread checker
ultrawork cron:create "Morning Check" --schedule weekday --at 09:00 \
  --action check_thread_reactions --notify-user U06CLS6E694

# Create an hourly mention scanner
ultrawork cron:create "Mention Scan" --schedule interval --hours 2 \
  --action scan_mentions --notify-channel D06CVUV964C

# Create a daily pending tasks reminder
ultrawork cron:create "Daily Reminder" --schedule daily --at 10:00 \
  --action dm_pending_tasks --notify-channel D06CVUV964C

# Manage jobs
ultrawork cron:list            # List all active jobs
ultrawork cron:pause CRON-ID   # Pause a job
ultrawork cron:run CRON-ID     # Manually trigger
ultrawork cron:logs CRON-ID    # View execution history
```

### CLI Commands

```bash
ultrawork task:list              # Task list
ultrawork task:show TASK-ID      # Task details
ultrawork slack:status           # Slack status
ultrawork slack:channels         # Channel list
ultrawork slack:users            # User list
ultrawork index:pending          # Pending approval list
ultrawork cron:list              # Cron job list
ultrawork cron:show CRON-ID      # Cron job details
ultrawork cron:create "Name"     # Create cron job
ultrawork cron:pause CRON-ID     # Pause cron job
ultrawork cron:resume CRON-ID    # Resume cron job
ultrawork cron:delete CRON-ID    # Delete cron job
ultrawork cron:run CRON-ID       # Manually trigger cron job
ultrawork cron:logs CRON-ID      # View execution logs
```

---

## Development Environment

```bash
# Install dependencies
make dev

# Formatting
make format

# Lint + type check
make check

# Tests
make test

# Run CLI
make run
# Or: uv run ultrawork
```

---

## Channel History Memory System

### Overview

Channel history memory stores quarterly conversation summaries from key Slack channels, enabling Claude to understand context and search for unfamiliar terms/projects.

### Directory Structure

```
data/memory/channel_history/
├── README.md              # Usage guide
├── schema.yaml            # Schema definition
├── {channel_name}/        # Per-channel directory
│   ├── 2026-Q1.yaml       # Quarterly summary
│   ├── 2026-Q2.yaml
│   └── ...
```

### Channel Memory Search Rules

**Automatic search triggers**: Automatically search channel memory in these situations:

1. **Unknown term encountered**:
   - Internal project names (e.g., "Stargate", "Pylon", "STORM")
   - Abbreviations (e.g., "RAG", "POC", "SG")
   - Customer/project names

2. **Context understanding needed**:
   - Reference to previous discussions needed
   - Decision verification needed
   - Owner/participant identification needed

### Search Methods

```bash
# Search for term
grep -r "term" data/memory/channel_history/

# View specific channel's recent quarter
cat data/memory/channel_history/eng-common/2026-Q1.yaml

# Search in terminology dictionary
grep -A3 "term: \"searchterm\"" data/memory/channel_history/*/2026-Q1.yaml
```

### Search Example

```yaml
# Search result example
terminology:
  - term: "Stargate"
    definition: "Sionic AI's search engine service"
    aliases: ["SG"]
    context: "Primarily mentioned in search quality and indexing discussions"
```

### Sync Skill

```
/sync-channel-history                # Sync all main channels
/sync-channel-history eng-common     # Sync specific channel only
```

### Main Channel List

| Category | Channel | Description |
|----------|---------|-------------|
| Engineering | eng-common | Engineering team discussions |
| Product | product-storm | STORM product related |
| Business | business-common | Business discussions |
| Research | research-common | Research discussions |
| Other | qna, standup | Q&A, standups |

### Memory Usage Scenarios

**Scenario 1**: Mention references "Stargate indexing issue"
```
1. Search "Stargate" term in channel memory
2. Check definition in terminology section
3. Understand previous discussion context from key_discussions
4. Generate appropriate response
```

**Scenario 2**: Need to reference "previous K8s migration decision"
```
1. Search "K8s" or "migration" in eng-common channel memory
2. Find related thread in key_discussions
3. Check decision details in decisions field
```

### Memory Update Frequency

- **Automatic sync**: Quarterly (recommended)
- **Manual sync**: Use `/sync-channel-history` skill
- **Trigger**: At new quarter start or after major project completion


## Language Configuration

System language is set to **한국어** (`ko`).

**IMPORTANT**: All the following MUST be in 한국어:
- All Slack messages and responses
- All thinking and reasoning output
- All skill execution output (TODO items, specs, reports, approval messages)
- All error messages and status updates sent to Slack
- All user-facing CLI output
- All exploration summaries and context analysis

Technical terms, code identifiers, file paths, and command names should remain in their original form.


## Token Configuration Note

Only personal token is configured in this installation.
- `mcp__slack-bot-mcp__` tools are not available
- Only `mcp__slack__` tools are available
- Bot-only features like `slack_add_reaction` are not available

---
> Source: [DolbonIn/ultraworker](https://github.com/DolbonIn/ultraworker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
