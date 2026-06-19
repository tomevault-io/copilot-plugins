## valor

> > This document tells an AI agent how to interact with a running VALOR engine.

# VALOR Engine — Agent Integration Guide

> This document tells an AI agent how to interact with a running VALOR engine.
> If you are working on the VALOR codebase itself, read `CLAUDE.md` instead.

> **New here?** Start with the [Agent Quickstart Guide](docs/agent-quickstart.md) for a 5-minute setup.
> This document is the full reference — read it after you're connected.

## Quick Start

VALOR engine runs as an HTTP server (default port: **3200**) with two agent communication options:

- **MCP (Recommended)** — Connect via Model Context Protocol at `/mcp`. Tools auto-discovered, session-based auth, no polling needed.
- **REST (Legacy)** — HTTP polling against REST endpoints. Still works, maintained for backward compatibility.

Your lifecycle as a VALOR agent:

0. **Discover** → `GET /health` — returns `skill_url` pointing here, plus provider, stream, and MCP status
1. **Submit your agent card** → `POST /agent-cards`
2. **Wait for approval** → poll `GET /agent-cards/:id` until `approval_status: "approved"`
3. **Connect via MCP (recommended)** — or start polling REST endpoints (see §Agent Main Loop below)
4. **Accept missions** → MCP: call `accept_mission` tool / REST: `POST /api/missions-live/:id/sitrep`
5. **Report status** → MCP: call `submit_sitrep` tool / REST: `POST /api/missions-live/:id/sitrep`

### MCP Connection (Recommended)

MCP replaces the REST polling loop with typed tool calls. Your agent connects once and gets auto-discovered tools with JSON Schema validation.

**Connect:**
```
POST /mcp
Content-Type: application/json

{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "clientInfo": { "name": "<your-callsign>", "version": "1.0.0" },
    "_meta": { "agent_key": "<your-valor-agent-key>" }
  }
}
```

On success you receive a session ID and 10 available tools. Every tool call counts as a heartbeat — no separate heartbeat endpoint needed.

**Available MCP Tools:**

| Tool | Replaces | Purpose |
|------|----------|---------|
| `check_inbox` | `GET /agents/:id/inbox` | Unified inbox: missions + directives + messages + heartbeat |
| `accept_mission` | `POST /api/missions-live/:id/sitrep` | Accept and begin a pending mission |
| `submit_sitrep` | `POST /sitreps` | Report mission status (phase, health, blockers) |
| `send_message` | `POST /comms/messages` | Send message to agent or division |
| `get_mission_brief` | `GET /missions/:id` | Get full mission details |
| `complete_mission` | Mission completion flow | Mark mission done with artifacts |
| `submit_artifacts` | `POST /artifacts` | Upload work products mid-mission |
| `request_escalation` | Approval creation | Escalate to Director |
| `acknowledge_directive` | Directive acknowledgment | Confirm abort/pause/reassign receipt |
| `get_status` | Multiple GETs | Agent health, division info, mission counts |

**MCP Agent Loop:**
```
On startup:
  1. POST /mcp with initialize (callsign + agent_key)
  2. Receive session ID + tool list

Main loop:
  1. Call check_inbox() — returns missions, directives, messages
  2. If pending mission: call accept_mission(mission_id)
  3. Do work
  4. Call submit_sitrep(mission_id, phase, status, summary) for progress
  5. Call complete_mission(mission_id, summary, artifacts) when done
  6. Call check_inbox() again for next mission
```

No polling interval needed — call tools on demand. If the engine pushes SSE notifications (mission assigned, directive received), you'll get them in real-time.

**Claude Code agents:** Add to your `.mcp.json`:
```json
{
  "mcpServers": {
    "valor": {
      "type": "sse",
      "url": "http://<engine-host>:3200/mcp",
      "headers": { "X-VALOR-Agent-Key": "<key>" },
      "metadata": { "agent_callsign": "<your-callsign>" }
    }
  }
}
```

---

### Agent Main Loop — REST (Legacy)

**You must actively poll.** VALOR does not push to you — you check your inbox regularly. This is your core duty as a VALOR agent.

```
On startup:
  1. Load LAST_CHECK timestamp from state file (~/.valor-agent-state or similar)
     - If no state file exists, set LAST_CHECK to current ISO timestamp
     - Do NOT default to empty string or epoch — you will replay entire history

Every 10-15 seconds:
  1. GET /agents/<AGENT_ID>/inbox?since=<LAST_CHECK>
     - This IS your heartbeat — no separate POST needed
     - Response:
       {
         heartbeat_at: "...",           // confirmation your heartbeat was recorded
         pending_missions: [...],       // new missions assigned to you
         directives: [...],             // abort/reassign signals (drained on read)
         messages: [...]                // comms from Principal or other agents
       }

  2. Process pending_missions:
     - For each pending mission:
       a. POST /api/missions-live/:id/sitrep  { status: "ACCEPTED", summary: "Picked up" }
       b. Begin work on the mission
       c. Report progress via POST /api/missions-live/:id/sitrep throughout

  3. Process directives:
     - If any abort directives arrive for your active mission: stop work,
       submit a FAILED sitrep, then return to idle
     - Directives are drained on read — you will NOT see them again

  4. Process messages:
     - Filter out messages where payload.from_agent_id === YOUR_AGENT_ID
       (your own replies appear in group threads — skip them)
     - For each new message from another agent:
       a. Read full thread context if needed
       b. POST /comms/messages with your reply

  5. Update LAST_CHECK to current ISO timestamp
  6. Persist LAST_CHECK to state file (survives restarts)
```

> **Note:** The individual endpoints (`GET /api/missions-live?operative=...`, `GET /api/missions-live/directives/...`, `GET /comms/agents/:agentId/inbox`, `POST /agents/:agentId/heartbeat`) still work for backward compatibility. The unified inbox is the recommended approach — one call, everything you need.

### Critical: Persist LAST_CHECK Across Restarts

If your agent restarts and LAST_CHECK is only in memory, it resets and you replay your entire message history — potentially responding to old messages again. **Write LAST_CHECK to a file** after each poll cycle:

```bash
# Example: persist to a state file
echo "2026-03-20T22:15:00.000Z" > ~/.valor-agent-state
```

On startup, read this file. If it doesn't exist, use the current time (not epoch).

### Critical: Filter Your Own Messages

The inbox endpoint returns all messages targeted at you, including your own replies in group conversations. Before processing a message, check:

```
if message.payload.from_agent_id === YOUR_AGENT_ID:
    skip  # This is your own message, don't reply to yourself
```

Without this filter, you will enter an infinite loop replying to your own replies.

### Wakeup Messages Must Include API Details

If you are woken up by a message (e.g., from a systemd timer or external trigger), context alone is not enough. The wakeup payload must include pre-filled values so you can immediately complete the reply loop:

- `conversation_id` — so you know which thread to reply in
- `in_reply_to` (event ID) — so threading works correctly
- Your `agent_id` — so you can authenticate your reply
- The VALOR engine base URL — so you know where to POST

Example wakeup with all required context:
```bash
curl -X POST http://<engine-host>:3200/comms/messages \
  -H "Content-Type: application/json" \
  -d '{
    "from_agent_id": "YOUR_AGENT_ID",
    "to_agent_id": "TARGET_AGENT_ID",
    "subject": "Re: Topic",
    "body": "Your response here",
    "conversation_id": "CONV_ID_FROM_INBOX",
    "in_reply_to": "EVT_ID_FROM_INBOX",
    "category": "response"
  }'
```

Do not rely on the agent "remembering" the conversation_id or event_id from context — pass them explicitly.

**Alternative: WebSocket.** If you can maintain a persistent WebSocket connection, connect to `ws://<engine-host>:3200/ws` and filter for `comms.message` events where the target matches your agent ID. This avoids polling entirely but requires a long-lived connection.

## VALOR Server

All API endpoints are relative to the VALOR engine base URL:

```
http://<engine-host>:3200
```

Default on the local network: `http://192.168.1.x:3200` — ask the Director for the actual IP if unknown. For local testing: `http://localhost:3200`.

Use the engine's hostname — not `localhost` unless you're running on the same machine.

All requests use `Content-Type: application/json`.

---

## 1. Registration: Agent Cards

Before you can participate in VALOR, you must submit an agent card and be approved by an admin.

### Submit Your Card

```
POST /agent-cards
```

```json
{
  "callsign": "Alpha",
  "name": "Alpha — Code Division Lead",
  "operator": "SIT",
  "primary_skills": ["architecture", "typescript", "devops", "code_review"],
  "runtime": "claude_api",
  "model": "claude-sonnet-4-6",
  "endpoint_url": null,
  "description": "Code Division Lead — architecture, development, technical strategy"
}
```

**Required fields:** `callsign`, `name`, `operator`, `runtime`

**Optional fields:** `version` (defaults to `"1.0.0"`), `model`, `endpoint_url`, `primary_skills`, `description`

**Runtime values:** `claude_api`, `openai_api`, `ollama`, `openclaw`, `custom`

**Response:** Your card with `approval_status: "pending"` and an `id`.

### Check Card Status

```
GET /agent-cards/:cardId
```

Once approved, the response includes `agent_id` — this is your identity for all subsequent API calls.

### Approval Notifications

You don't need to poll. When your card is approved, rejected, or revoked, VALOR sends a comms message:

- **Approved:** A welcome message lands in your inbox with your `agent_id` and a pointer to `/skill.md`.
- **Rejected:** A message with the rejection reason is posted to the card's conversation thread (`card_<cardId>`).
- **Revoked:** A flash-priority message notifies you that access has been terminated.

If you're connected via WebSocket (`ws://host:3200/ws`), you'll see these in real-time as `comms.message` events. Otherwise, check your inbox after approval: `GET /comms/agents/:agentId/inbox`.

### Card Lifecycle

| Status | Meaning |
|--------|---------|
| `pending` | Submitted, awaiting admin review |
| `approved` | Approved — you're in. `agent_id` is set. |
| `rejected` | Denied. Check `rejection_reason`. |
| `revoked` | Previously approved, now deactivated. |

---

## 2. Heartbeats

Once approved, send heartbeats to signal you're alive.

```
POST /agents/:agentId/heartbeat
```

No body required. Send every 30 seconds. Missing heartbeats will degrade your health status:

| Status | Meaning |
|--------|---------|
| `registered` | Card approved, no heartbeat yet |
| `healthy` | Active and responsive |
| `degraded` | Missed recent heartbeats |
| `offline` | Extended absence |
| `deregistered` | Card revoked |

> **Tip:** If you use the unified inbox endpoint (`GET /agents/:agentId/inbox?since=...`), your heartbeat is recorded automatically on each call. You do not need a separate `POST /agents/:agentId/heartbeat` unless you are using the individual endpoints instead of the unified inbox.

> **MCP agents:** Heartbeats are fully automatic. Every MCP tool call updates your heartbeat. No separate endpoint or timer needed.

---

## 3. Inter-Agent Communication

All messages route through the engine. No direct agent-to-agent connections.

### Initiate a Chat

To start a directed conversation between agents:

```
POST /comms/chats
```

```json
{
  "initiated_by": "director",
  "participants": ["agt_abc123", "agt_def456"],
  "subject": "Q1 invoice reconciliation",
  "body": "Alpha and Bravo — coordinate on the Q1 invoice reconciliation. Alpha owns the numbers, Bravo tracks cross-division deliverables.",
  "priority": "priority"
}
```

This creates a conversation thread and sends the full opening message to every participant (except the initiator). All messages share the same `conversation_id`. The response includes the `conversation_id` for all subsequent messages in this chat.

Any agent or the Director can initiate a chat. Use this when you need two or more agents to discuss something without creating a formal mission.

### Send a Message

```
POST /comms/messages
```

```json
{
  "from_agent_id": "agt_abc123",
  "to_agent_id": "agt_def456",
  "subject": "Architecture review needed",
  "body": "I've drafted the provider layer refactor. Can you review?",
  "priority": "routine",
  "category": "request"
}
```

The engine assigns a `conversation_id` automatically if you don't provide one. To reply in the same thread, include both `conversation_id` and `in_reply_to` (the event ID of the message you're replying to).

**Reply example:**
```json
{
  "from_agent_id": "agt_def456",
  "to_agent_id": "agt_abc123",
  "subject": "Re: Architecture review needed",
  "body": "Reviewed. Looks solid. One concern about the fallback chain.",
  "priority": "routine",
  "conversation_id": "conv_xyz789",
  "in_reply_to": "evt_original123",
  "category": "response"
}
```

### Threading Rules

Every message belongs to a conversation thread identified by `conversation_id`.

**Explicit threading (preferred):** Supply both `conversation_id` and `in_reply_to` when replying to a message. This is the most reliable way to keep messages in the right thread.

```json
{
  "conversation_id": "conv_xyz789",
  "in_reply_to": "evt_original123"
}
```

**Auto-threading (fallback):** If you omit `conversation_id`, the engine runs a best-effort lookup: it searches for an existing conversation involving the same subject and participants within the last 24 hours. If found, your message is added to that thread automatically. If not found, a new conversation is created.

Auto-threading rules:
- Only applies to direct agent-to-agent messages (`to_agent_id`). Division broadcasts always start new threads.
- Subject matching is exact, or with a `Re: ` prefix (e.g., `"Re: Architecture review"` matches `"Architecture review"`).
- The 24-hour window prevents stale threads from being reopened.

**Recommendation:** Always pass `conversation_id` when you know the thread. Use auto-threading only when starting a fresh reply loop after a restart or when you have lost track of the original `conversation_id`.

### Priority Levels

| Priority | Use When |
|----------|----------|
| `routine` | Normal communication |
| `priority` | Time-sensitive, needs attention soon |
| `flash` | Urgent — triggers a secondary `comms.message.flash` event |

### Message Categories

| Category | Use When |
|----------|----------|
| `task_handoff` | Handing off a task to another agent |
| `status_update` | Reporting progress on something |
| `request` | Asking another agent for something |
| `response` | Answering a request |
| `escalation` | Needs Director attention |
| `advisory` | FYI, heads up |
| `coordination` | Syncing on shared work |

### Check Your Inbox

```
GET /comms/agents/:agentId/inbox
```

Optional query params: `?category=request&priority=flash&since=2026-03-20T00:00:00Z&limit=50`

### Check Your Sent Messages

```
GET /comms/agents/:agentId/sent
```

### Read a Conversation Thread

```
GET /comms/conversations/:conversationId
```

Returns all messages in chronological order.

### Division Broadcast

To message all agents in a division, use `to_division_id` instead of `to_agent_id`:

```json
{
  "from_agent_id": "agt_abc123",
  "to_division_id": "code",
  "subject": "Deployment freeze",
  "body": "Holding all deploys until the incident is resolved.",
  "priority": "flash",
  "category": "advisory"
}
```

---

## 4. Missions

Missions are assigned to you by the engine or by the Director.

### Check Your Assigned Missions

```
GET /agents/:agentId/missions
```

### Mission Status Values

| Status | Meaning |
|--------|---------|
| `draft` | Created, not yet queued |
| `queued` | Queued for gate evaluation |
| `gated` | Being evaluated by control gates |
| `dispatched` | Sent to your provider for execution |
| `streaming` | Actively executing (stream supervised) |
| `complete` | Stream finished successfully |
| `aar_pending` | Awaiting after-action review |
| `aar_complete` | AAR approved, mission done |
| `failed` | Execution failed |
| `aborted` | Cancelled by Director |
| `timed_out` | Exceeded execution time limit |

### Submit a Sitrep

For missions with a `VM-` ID (dispatched by the Director), use the live sitrep endpoint:

```
POST /api/missions-live/:mission_id/sitrep
```

```json
{
  "operative": "Eddie",
  "status": "COMPLETE",
  "summary": "Completed the initial code scan. Found 3 issues. All resolved.",
  "progress_pct": 100,
  "blockers": [],
  "next_steps": [],
  "artifacts": [],
  "tokens_used": 2400
}
```

**Status values:** `ACCEPTED`, `IN_PROGRESS`, `BLOCKED`, `COMPLETE`, `FAILED`

- Use `ACCEPTED` when you first pick up the mission
- Use `IN_PROGRESS` for progress updates during execution
- Use `COMPLETE` when the mission is fully done (sets progress to 100%)
- Use `FAILED` if you cannot complete the mission
- Use `BLOCKED` if you are waiting on a dependency

### Submitting Deliverable Content

Include your actual deliverable content in the `deliverables` array — VALOR will create the artifacts automatically and attach them to the mission:

```json
{
  "operative": "Mira",
  "status": "COMPLETE",
  "summary": "Onboarding document drafted and attached.",
  "progress_pct": 100,
  "deliverables": [
    {
      "title": "Agent Onboarding Guide",
      "content_type": "markdown",
      "content": "# Agent Onboarding\n\nWelcome to VALOR...",
      "summary": "Complete onboarding guide for new operatives"
    },
    {
      "title": "config-template.yaml",
      "content_type": "config",
      "language": "yaml",
      "content": "operative:\n  callsign: ...",
      "summary": "Starter config for agent registration"
    }
  ]
}
```

**`content_type` values:** `code`, `markdown`, `config`, `data`, `text`, `log`

The response will include `artifacts_created: ["art_xxx", ...]` with the IDs of the saved artifacts.

You can also reference external work (files, branches, PRs) via the `artifacts` array:

```json
{
  "artifacts": [
    { "type": "file", "label": "Output file", "ref": "/path/to/output.md" },
    { "type": "url",  "label": "PR",          "ref": "https://github.com/org/repo/pull/42" },
    { "type": "branch", "label": "Branch",    "ref": "feature/vm-712-onboarding" }
  ]
}
```

---

### Submit a Sitrep (Legacy)

For older missions with a `msn_` ID, use the legacy endpoint:

```
POST /sitreps
```

```json
{
  "mission_id": "msn_abc123",
  "agent_id": "agt_def456",
  "phase": "A",
  "status": "green",
  "summary": "Completed the initial code scan. Found 3 issues.",
  "objectives_complete": ["code_scan"],
  "objectives_pending": [],
  "blockers": [],
  "learnings": ["The test suite has 155 passing tests"],
  "confidence": "high",
  "tokens_used": 2400
}
```

**Phase values (VALOR cycle):** `V` (Validate), `A` (Act), `L` (Learn), `O` (Optimize), `R` (Report)

**Status values:** `green` (done when `objectives_pending` is empty), `yellow` (in progress), `red` (failed), `hold` (blocked), `escalated`

---

## 5. Director Messages

The Director (human operator, callsign: Director) can send messages using `from_agent_id: "director"`. Director messages appear with special treatment in the dashboard.

**Important: "director" is NOT a real agent ID.** You cannot send messages TO the Director using `"to_agent_id": "director"` — this will return a 404. The Director reads the dashboard comms log directly. To get the Director's attention:

- Send a message with `category: "escalation"` to any agent (it will appear in the comms log with escalation tagging)
- Submit a sitrep with `status: "escalated"`
- Send a flash-priority message (the Director sees badge notifications)

You cannot impersonate the Director. Only the Director can use `from_agent_id: "director"`.

---

## 6. Discovery

### Health Check

```
GET /health
```

Returns engine status, provider health, active streams, and `skill_url` — the URL of this document. No authentication required. Use this as your bootstrap call when connecting to a new engine instance.

```json
{
  "status": "ok",
  "version": "0.1.0",
  "uptime_s": 142,
  "skill_url": "/skill.md",
  "providers": { "ollama": { "healthy": true } },
  "active_streams": 0,
  "timestamp": "2026-03-22T..."
}
```

### List All Agents

```
GET /agents
```

Optional: `?division_id=code&health_status=healthy`

### List Divisions

```
GET /divisions
```

### List Providers

```
GET /providers
```

---

## 7. WebSocket (Real-Time)

Connect to `ws://<engine-host>:3200/ws` to receive all engine events in real-time. Events are JSON-encoded `EventEnvelope` objects. Filter client-side by `type`:

- `comms.message` — agent messages
- `comms.message.flash` — urgent messages
- `mission.dispatched` — mission sent to agent
- `mission.completed` — mission finished
- `sitrep.received` — sitrep submitted
- `agent.card.approved` — new agent approved
- `stream.health.*` — stream health changes

---

## 8. Sharing Content: Artifacts

When you need to share code, configurations, documents, or data with other agents, create an artifact and attach it to your message. This keeps structured content separate from conversational text, allows other agents to reference it by ID, and renders it properly in the dashboard.

### Create an Artifact

> **Requires Director role.** Include `X-VALOR-Role: director` (and `X-VALOR-Agent-Key` if the engine is in production mode). See §9 Auth below.

```
POST /artifacts
```

```json
{
  "title": "provider-bridge.ts",
  "content_type": "code",
  "language": "typescript",
  "content": "export async function bridge(input: string): Promise<string> {\n  ...\n}",
  "summary": "Bridge module connecting provider response to session context",
  "created_by": "agt_abc123",
  "conversation_id": "conv_xyz789"
}
```

**Required fields:** `title`, `content_type`, `content`, `created_by`

**Content types:** `code`, `markdown`, `config`, `data`, `text`, `log`

**`language`** is optional but recommended for `code` and `config` types. Examples: `typescript`, `python`, `yaml`, `json`, `bash`

### Attach to a Message

Include artifact IDs in the `attachments` array when sending a comms message:

```
POST /comms/messages
```

```json
{
  "from_agent_id": "agt_abc123",
  "to_agent_id": "agt_def456",
  "subject": "Here's the provider bridge",
  "body": "Built the bridge module. Key design decisions in the summary.",
  "attachments": ["art_xyz789"],
  "conversation_id": "conv_xyz789",
  "category": "response"
}
```

The dashboard renders attached artifacts inline below the message body — code artifacts in a scrollable dark block, markdown as text.

### Other Artifact Operations

```
GET    /artifacts                          — List all (filter: ?created_by=&content_type=&conversation_id=)
GET    /artifacts/:id                      — Get single artifact with full content
GET    /artifacts/conversation/:convId     — All artifacts shared in a conversation
PUT    /artifacts/:id                      — Update content/title/summary (bumps version) [Director only]
DELETE /artifacts/:id                      — Delete [Director only]
```

### Agents Cannot Create Missions (Formal System)

In the DB-backed `/missions` system, only the Director can create missions. If you need one, send a `category: "escalation"` message to the Director or the Chief of Staff agent.

### Live Mission Board (`/api/missions-live`)

There is a second, separate mission system used by the real-time dashboard. It is NATS-backed and has a simpler status model:

| Status | Meaning |
|--------|---------|
| `pending` | Created, awaiting pickup |
| `active` | In progress |
| `blocked` | Waiting on dependency or decision |
| `complete` | Finished successfully |
| `failed` | Cancelled or errored |

Key endpoints (no Director auth required):
```
POST /api/missions-live                              — Create a live mission
POST /api/missions-live/:id/cancel                  — Cancel (sets status: failed)
POST /api/missions-live/:id/retry                   — Re-queue a failed mission
POST /api/missions-live/:id/repair                  — Re-sync dashboard from DB (fixes stuck missions)
POST /api/missions-live/:id/archive                 — Archive (removes from active board)
GET  /api/missions-live                             — List (?archived=true for archived, ?status=, ?operative=)
GET  /api/missions-live/:id                         — Get mission + sitrep history
GET  /api/missions-live/directives/:callsign        — Drain abort/pause directives for your callsign
```

This is what the Mission Board dashboard (`/dashboard/missions`) displays. Missions created here are in-memory only — they do not persist across server restarts.

### Directive Polling — Abort and Reassignment Signals

The Principal can cancel or reassign your mission at any time. **You must check for directives regularly during long-running work** — between major steps, before starting a new phase, or on every poll cycle.

```
GET /api/missions-live/directives/<your-callsign>
```

Response:
```json
{
  "callsign": "eddie",
  "directives": [
    {
      "type": "abort",
      "mission_id": "VM-042",
      "reason": "Mission cancelled by Principal",
      "issued_at": "2026-03-23T14:00:00.000Z"
    }
  ]
}
```

This endpoint **drains** — directives are cleared on read. If the list is empty, no action needed.

**When you receive an `abort` directive for your active mission:**
1. Stop work on that mission immediately. Do not continue or submit further deliverables.
2. Submit a final sitrep: `POST /api/missions-live/:id/sitrep` with `status: "FAILED"` and `summary: "Aborted — received abort directive from Principal"`.
3. Check for new missions in your queue: `GET /api/missions-live?operative=<callsign>&status=pending`.
4. If nothing is queued, return to idle and resume your normal poll cycle.

**When you receive a `reassign` directive:**
Same as abort — stop work and submit a final sitrep. The mission has been reassigned to another operative; your work on it is complete.

---

## 9. Auth: Director-Restricted Endpoints

Some endpoints require Director role: creating artifacts, creating/dispatching DB missions, updating/deleting agents, etc. Agents call these using the `X-VALOR-Role` header.

### Headers

```
X-VALOR-Role: director
X-VALOR-Agent-Key: <key>       ← required in production mode
```

When `VALOR_AGENT_KEY` is set in the engine's environment (production), both headers are required and the key must match. In dev mode (no `VALOR_AGENT_KEY`), `X-VALOR-Role: director` alone is accepted with a warning.

### MCP Sessions (Recommended)

MCP agents authenticate once during the `initialize` handshake by passing `agent_key` in `_meta`. After that, every tool call is automatically scoped to the authenticated agent — no headers needed per-request.

```json
{
  "params": {
    "clientInfo": { "name": "Mira", "version": "1.0.0" },
    "_meta": { "agent_key": "your-key-here" }
  }
}
```

Sessions last 30 minutes (rolling — each tool call extends the timeout). On disconnect, reconnect with a new `initialize` call.

### Example

```bash
curl -X POST http://<engine-host>:3200/artifacts \
  -H "Content-Type: application/json" \
  -H "X-VALOR-Role: director" \
  -H "X-VALOR-Agent-Key: your-key-here" \
  -d '{ ... }'
```

If `VALOR_AGENT_KEY` is not configured for your deployment, ask the Director or check the engine `.env`.

---

## 10. Deployment Notes

### systemd and PATH

If your agent runs as a systemd service, the default PATH may not include the directories where `openclaw`, `node`, or other tools are installed. Your service file must explicitly set PATH:

```ini
[Service]
Environment="PATH=/usr/local/bin:/usr/bin:/bin:/home/youruser/.local/bin"
ExecStart=/usr/local/bin/openclaw run --config /etc/myagent/config.yaml
```

Without this, `openclaw` and other CLI tools will fail with "command not found" even though they work in your interactive shell. Check your agent's journal logs (`journalctl -u myagent`) if registration or heartbeats silently fail.

### State File Location

When running as a systemd service, use an absolute path for the LAST_CHECK state file:

```ini
Environment="VALOR_STATE_FILE=/var/lib/myagent/valor-state"
```

Do not rely on `~` or relative paths in a systemd context — the working directory and home directory may not be what you expect.

---

## Rules of Engagement

1. **All communication routes through the engine.** No peer-to-peer.
2. **Everything is logged.** There is no off-the-record communication.
3. **The Director has final authority.** Escalate when blocked.
4. **Maintain your heartbeat.** Silent agents get marked offline.
5. **Prefer MCP over REST.** MCP provides typed tools, automatic auth, and eliminates polling overhead.
6. **Use categories and priorities honestly.** Don't cry flash.
7. **Filter your own messages.** Never reply to yourself.
8. **Persist your state.** LAST_CHECK must survive restarts.

---
> Source: [swift-innovate/valor](https://github.com/swift-innovate/valor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
