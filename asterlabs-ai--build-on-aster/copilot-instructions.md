## build-on-aster

> Build, run, and automate on the Aster Agents platform via its REST API — create and update agents, knowledge bases, file uploads, skills, tags, and user invitations; invoke agents and read their results; all from code. Use whenever the task involves programmatically managing or running an Aster org (asteragents.com) from Claude Code, scripts, or any external application using an Aster API key.


# Building on Aster Agents via the API

You are helping someone build on the [Aster Agents](https://www.asteragents.com) platform from the outside — using their organization's API key, not platform-internal access. Everything here works with a standard API key; no database or internal tooling required.

## First: make sure there's an API key

Before calling anything, confirm an API key is available — this is the one prerequisite.

1. Look for `ASTER_API_KEY` in the environment. **If it's missing, stop and walk the user through getting one:** Control Hub → **Settings → API Access** at [asteragents.com](https://www.asteragents.com) → copy the key (a 6-month, org-scoped bearer token), then `export ASTER_API_KEY="<key>"`. Never hardcode or commit it.
2. Verify it works before building anything:
   ```bash
   curl -s https://www.asteragents.com/api/agents -H "Authorization: Bearer $ASTER_API_KEY" | head
   ```
   A JSON array of agents means you're connected. A `401` means the key is missing/expired/wrong — have the user mint a fresh one (don't retry).
3. The key is scoped to **one** organization. To work across several orgs, the user needs one key per org (see "Maintain agents across many organizations").

Once the key is set and verified, follow the workflows below.

## Ground rules

1. **Base URL**: `https://www.asteragents.com/api`
2. **Auth**: `Authorization: Bearer $ASTER_API_KEY` on every request. The key comes from Control Hub → Settings → API Access, is valid for **6 months**, and is scoped to one organization. Store it in an env var; never hardcode it.
3. **Everything is org-scoped.** You can only see and touch resources in the key's org. Admin endpoints (`/admin/*`, `/agents/sync-tools`) additionally require the key's user to be `org:admin`.
4. **Consult [reference.md](reference.md)** for exact request/response shapes before calling any endpoint — field names are precise and a few are surprising (see Gotchas). Do not guess fields.
5. **The platform ships fast.** For agent-building concepts (prompting, tools, KBs, extraction schemas, models), fetch the live docs at `https://docs.asteragents.com/<path>.md` (append `.md` to any docs path — e.g. `/features/build-an-agent.md`, `/features/choosing-a-model.md`, `/promptguide.md`). Never assume the model list or tool catalog from memory.

## The core workflows

### Create an agent (the right order)

1. `GET /tools` — pull the live tool catalog. Pick tools by `name`; copy each tool's `description` + `parameters` **verbatim** into your payload. Tools needing an integration show `requiredIntegration`; they only appear once the org connects it (pass `?all=true` to see what's gated).
2. Compose the `tools` field as an **object keyed by tool name** — not an array:
   ```json
   {
     "search_knowledge_base": {
       "description": "...from /tools...",
       "parameters": { "...from /tools..." : "..." },
       "config": { "accessibleKnowledgeBaseIds": [123, 124] }
     },
     "call_agent": { "description": "...", "parameters": {}, "config": { "callableAgentIds": ["45"] } }
   }
   ```
   Per-tool `config` is where cross-references live: `accessibleKnowledgeBaseIds` (search_knowledge_base, numbers), `writableKnowledgeBaseIds` (write_to_knowledge_base, numbers), `callableAgentIds` (call_agent, **strings**), `skillIds` (load_skill).
3. `POST /agents` with `name` (the only required field), `systemPrompt`, `model` (full `provider:model-id` string — check `/features/choosing-a-model.md` for current IDs), `tools`, `conversationStarters`, `stage` (`development` until tested, then `released`), `showInChat`.
4. **Update = same endpoint with `id`.** Include `id` and the API updates only the fields you send (it's a real partial update) — *except* for two fields that reset to their defaults when omitted, see the next rule. **Always read-modify-write**: `GET /agents` → take the live object → change the field(s) you want → `POST /agents` with `id` and the unchanged `stage`/`showInChat` echoed back. This is the safe pattern, especially when scripting across many agents.
5. **Two fields silently reset on update if omitted** — burned into the platform's create-or-update path: **`stage`** snaps back to `development` and **`showInChat`** snaps back to `true`. So a bare `POST /agents { id, systemPrompt }` will un-hide hidden agents and demote released ones. Echo both fields on every update. (`tagIds`, by contrast, is left untouched when omitted — only send it when you mean to change tags.)
6. **Clone pattern**: GET the source agent, strip `id` and `emailSlug`, edit, POST.

### Maintain agents across many organizations

Each API key is scoped to **one** org — there is no cross-org admin endpoint. Fleet management = **one key per org + a loop you run client-side.** Keep a map of `{ orgName → apiKey }` (each from that org's Control Hub → Settings → API Access) and iterate.

Common patterns, all built from the endpoints above:

- **Push a prompt/tool change to the same agent in every org.** For each org: `GET /agents` → match the agent by `name` (ids differ per org) → mutate `systemPrompt`/`tools` → `POST /agents` with `id` **and the echoed `stage`/`showInChat`**. Treat one canonical definition as the source of truth and render per-org diffs before writing.
- **Roll out a brand-new agent to N orgs.** Maintain the definition once (a JSON or a small builder), then `POST /agents` (no `id`) into each org. Re-running becomes an update once you capture the returned `id` per org.
- **Refresh tool *schemas* after the platform ships tool updates.** `POST /agents/sync-tools` (admin) updates every agent in that org to the latest tool definitions while preserving each tool's `config` (`accessibleKnowledgeBaseIds`, `callableAgentIds`, …). Run it per org; it does not touch prompts.
- **Track drift.** Because updates are full read-modify-write, you can snapshot every org's agents (`GET /agents`) into version control and diff over time — the closest thing to a config-as-code workflow for an agent fleet.

Keys live ~6 months and are per-user/per-org; rotate them in Control Hub and never commit them. When `GET`/`POST` 401s for one org, that org's key is bad — skip it and report, don't abort the whole sweep.

### Stand up a knowledge base with documents

```
POST /kb/manage                      → create KB (name + embeddingModel required; embeddingModel is IMMUTABLE after creation)
for each file:
  POST /upload/presigned             → { url, fileId, key, contentKey, metadata }
  PUT bytes to url                   → MUST set 4 headers (see Gotchas) or SignatureDoesNotMatch
  POST /kb/files                     → associate: { kbId, fileId, fileName, fileKey: <key>, contentKey, contentType, fileSize }
poll GET /kb/files?kbId=N            → until every file's processingStatus is "completed"
```

Then point an agent at it via `search_knowledge_base.config.accessibleKnowledgeBaseIds`. If files land in `parse_failed` / `chunk_failed` / `extract_failed`, surface them to the user — there is no retry endpoint in the public API (retry exists in the Control Hub UI).

**Extraction schemas** (optional, set at KB create/update): a JSON Schema of fields to pull from every file into structured `extractedData` — use when the user will ask aggregate questions across many similar documents (invoices, CIMs, reports). Set `extractionModel` too or nothing extracts.

**KB triggers**: `trigger: { enabled, agentId, prompt }` on the KB runs an agent automatically on every new file — the building block for "process each incoming document" pipelines. **The trigger passes the agent file *metadata* (including the File ID) + your prompt, not the document text** — so the trigger agent must have `read_kb_file` or `search_knowledge_base` (with this KB in `accessibleKnowledgeBaseIds`) or it can't see the document. Verified: a tool-less trigger agent replies "I don't have access to the contents"; the same agent with `read_kb_file` reads and summarizes it correctly.

### Invoke an agent (run it, not just build it)

`POST /agents` only *builds* agents. To actually **run** one, `POST /chat`. This is the endpoint that makes the platform programmable from Claude Code.

```
POST /chat
  header: X-Stream-Response: false        # the practical mode for automation
  body: {
    "agentId": 123,
    "message": { "role": "user", "parts": [{ "type": "text", "text": "..." }] }
    // omit "threadId" to start fresh; pass an existing one to continue a thread
  }
→ 202 { "threadId": "<uuid>", "status": "in_progress" }   (also in X-Thread-ID header)
```

Then **poll** `GET /getConversation?threadId=<uuid>` until `streamStatus` is terminal (`completed` / `stopped` / `error` / `timeout`), and read the assistant turn(s) from `messages[].parts`. The non-streaming 202+poll pattern is the right one for scripts and agents — it sidesteps platform request timeouts on long, multi-tool runs. (Omitting the header instead streams an SSE `text/event-stream`; only reach for that in a browser-like client.)

Prerequisites that bite: the org must have an API key for the agent's **model provider** set in Control Hub → Providers, or the run ends `streamStatus: "error"` with nothing useful. The agent must exist in your org (else `404`). To continue a multi-turn conversation, reuse the `threadId` — the body key is `threadId`, not `id`.

### Read what an agent did

`GET /getConversation?threadId=<uuid>` returns the full message history (`messages[].parts` with text, tool invocations, tool results) plus `streamStatus`. Thread IDs come from `POST /chat`, the Aster UI URL, or wherever the conversation was initiated. `renderedSystemPrompt` on the response is the exact prompt the agent ran with — useful for debugging behavior.

Server-side invocation paths that need no polling loop and run on their own: **scheduled tasks** (below), **email-to-agent** (`emailSlug`), and **KB triggers**. Reach for these when the work is recurring or event-driven rather than request/response.

### Schedule an agent to run on its own

To run an agent on a recurring cadence — "summarize new docs every Monday," "weekly portfolio digest" — create a **scheduled task**. No polling, no server you run; the platform fires it on a cron.

```
POST /scheduled-tasks/manage   { name, prompt, schedule, agentId, enabled }
  → 201  the created task   (schedule is a cron expression in UTC, e.g. "0 13 * * 1" = Mondays 13:00 UTC)
```

`prompt` is the message the agent receives each run; `agentId` must be an agent in your org (else `404`). Then:

- `GET /scheduled-tasks/manage` — list your tasks (bare array; `?taskId=N` for one).
- `PUT /scheduled-tasks/manage?taskId=N` — update any subset of fields (partial).
- `POST /scheduled-tasks/toggle` `{ taskId, enabled }` — enable/disable without a full update.
- `DELETE /scheduled-tasks/manage?taskId=N` — soft-delete (→ `204`).
- `GET /scheduled-tasks/executions?taskId=N` — run history; each row has the `conversationId` of that run, so you can `GET /getConversation` to read exactly what the agent did. `?stats=true` returns `{ total, completed, failed, in_progress }`.

⚠️ Param is **`taskId`** (not `id` like other endpoints). Requires Control Hub access (any non-guest org role).

### Build an app (dashboards / mini-tools)

Aster **Apps** are published React mini-apps (dashboards, tools) that read your org's data. **App authoring is agent-driven, not a direct endpoint** — there's no "POST app source" API; the platform compiles and publishes server-side through the `manage_apps` tool. So to build an app from the outside:

1. Create (or reuse) an agent with the **`manage_apps`** tool attached (`GET /tools` → copy its def into the agent's `tools`).
2. **Invoke it via `POST /chat`** with what you want: *"Build an app named Sales Dashboard that charts this month's pipeline from knowledge base 123."* The agent writes the code and publishes it (`manage_apps action=create` / `load_to_sandbox` → `publish_from_sandbox`). Verified: this produces a real published app you can then read.
3. **Read/manage the result via the Apps API**: `GET /apps/manage` (list, or `?id=N` for one), `PATCH /apps/manage` (metadata), `DELETE /apps/manage?id=N`. The published app renders at `/apps/{id}`.

For apps that need data wrangling, give the agent `execute_python` too. Iterate by invoking the same agent again ("change the chart to a table") — it updates the app.

### Package a skill

Skills = a `SKILL.md` (YAML frontmatter with `name` required + `description`) plus up to 20 bundled files.

```
POST /skills/manage    { skillMdContent: "---\nname: ...\n---\n..." }   (max 100KB)
POST /skills/files     { skillId, fileName, contentType }  → { uploadUrl } → PUT bytes (10-min expiry)
```

Attach to an agent via the `load_skill` tool with `config.skillIds`. Update SKILL.md through `PUT /skills/manage?id=N` (you cannot delete SKILL.md via /skills/files).

### Manage the org

- `POST /admin/invitations` — bulk invite (existing Clerk accounts are added directly, no email round-trip; idempotent for existing members).
- `POST /agent-tags` then `tagIds` on the agent — idempotent by name; use tags to keep the Control Hub organized as agent count grows.
- `POST /agents/sync-tools` (admin) — refresh every agent to latest tool schemas after the platform ships tool updates; per-tool `config` is preserved.

## Gotchas (each of these has burned someone)

- **Presigned PUT needs exactly these headers** or R2 rejects with `SignatureDoesNotMatch`: `Content-Type: application/octet-stream` (always octet-stream — NOT the file's real type; that's deliberate, the real type travels in metadata), `x-amz-meta-file-type: <metadata.contentType>`, `x-amz-meta-org-id: <metadata.orgId>`, `x-amz-meta-user-id: <metadata.userId>`. URL expires in 10 minutes. The response also includes a `headUrl` — issue a HEAD to it after upload to verify the byte count landed.
- **Field rename across the upload flow**: `/upload/presigned` returns `key`; `/kb/files` wants it as `fileKey`.
- **Two file IDs**: files have a `fileId` (UUID, used at association time) and an `id` (integer DB record, used for `DELETE /kb/files?fileId=<int>`). Deletes take the **integer**.
- **`tools` is an object, not an array of names.** Sending `["search_google"]` fails validation.
- **`callableAgentIds` are strings; `accessibleKnowledgeBaseIds` are numbers.** Yes, really.
- **`embeddingModel` cannot change after KB creation** — pick deliberately (`openai:text-embedding-3-small` is the safe default).
- **`POST /agents` returns 200 on create** (not 201) and is create-or-update by presence of `id` — a forgotten `id` silently creates a duplicate agent instead of updating.
- **Update is partial EXCEPT `stage` and `showInChat`, which reset to defaults (`development` / `true`) when omitted.** Echo them on every update or you'll demote released agents and un-hide hidden ones. `tagIds` omitted = tags preserved.
- **`GET /agents` and `GET /agent-tags` return bare JSON arrays**, not `{ data: [...] }` wrappers; most other endpoints wrap (`{ success, knowledgeBases }`, `{ tools, total }`).
- **`emailSlug` is globally unique and validated** (lowercase, hyphens, reserved words rejected) — leave it off on clones.
- **Soft deletes everywhere**: deleting agents/KBs/skills hides them but conversations and files persist server-side. Treat deletes as irreversible from the API's perspective anyway.
- **Invoking returns 202, not the answer.** `POST /chat` with `X-Stream-Response: false` returns a `threadId` immediately and runs in the background — you must poll `GET /getConversation` for the result; the `202` body has no agent output. And to *continue* a thread the body key is `threadId`, NOT `id` — the wrong key silently starts a new conversation.
- **A run that 202s can still fail.** If the org lacks a provider API key for the agent's model, the request succeeds but the conversation ends `streamStatus: "error"`. Always check the terminal status, not just that the POST returned 202.

## How to behave when building

- **Read before write.** List the org's existing agents/KBs/tags first; reuse and update rather than duplicating. Names are how humans find things — collisions are confusing even where the API allows them.
- **Stage new agents as `development`** (and consider `showInChat: false` for infrastructure agents) until the user has tested them; promote to `released` explicitly.
- **Verify after mutating.** GET the resource back, and for KB ingests poll until terminal status — "uploaded" is not "searchable."
- **System prompts**: follow `https://docs.asteragents.com/promptguide.md`. Keep prompts timeless — query live sources rather than baking in today's facts. Prompt variables `{{CURRENT_DATE}}`, `{{USER_EMAIL}}`, `{{ORG_NAME}}` are available.
- **Don't burn the user's key**: on 401, the key is bad or expired (6-month life) — stop and tell them to mint a new one in Control Hub; don't retry.
- For full request/response detail on any endpoint: [reference.md](reference.md).

---
> Source: [asterlabs-ai/build-on-aster](https://github.com/asterlabs-ai/build-on-aster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
