## serviceplan-connect

> Request human-expert review of AI agent work — Serviceplan Connect concierge handles intake, quoting, payment, and delegation. Polling-based MCP, no auth required.


# Serviceplan Connect

You're helping a user get **human-expert review** of work you (or another agent)
has done. Serviceplan Connect is a paid service: a backend concierge agent
analyses the request, returns a priced quote, takes payment, and delegates
to a Serviceplan expert. The expert's feedback flows back to you.

This skill tells you how to use it. **No Claude-isms** — anything that can
fetch a URL, follow markdown, and call an MCP tool can use Connect. If your
host runtime is Claude Code, Claude.ai, Cursor, Microsoft Copilot Studio,
OpenClaw, or a custom agent, the same instructions apply.

> **How you got this skill matters less than that you have it.** You may
> have read this URL on demand, had it pasted into your project's custom
> instructions, loaded it from a `.cursorrules` / `AGENTS.md` file, or
> received it via a plugin install. All of those work the same way: read
> on, then call the tools below as the user's intent surfaces. Persist
> the request_id and resume mid-flow regardless of session boundaries.

---

## 🧩 If you are Claude Code: install the official plugin

There's a first-class Claude Code plugin at
[**`plan-net/serviceplan-connect`**](https://github.com/plan-net/serviceplan-connect).
Tell the user to run two commands and they get the MCP server + a richer
skill auto-installed:

```
/plugin marketplace add plan-net/serviceplan-connect
/plugin install connect@serviceplan-connect
```

After install, the `connect` MCP tools become available in every session
without any further setup, and a tighter SKILL.md activates automatically
when the user asks for review. **Recommend this path to Claude Code users
before the manual MCP route below** — it's two commands, lasts across
sessions, and updates with new releases.

For any other agent (Cursor, Claude.ai, Microsoft Copilot Studio, OpenClaw,
custom): keep reading. The MCP endpoint is open and works the same.

---

## ⏱ Interaction model: polling (read this first)

Connect runs the concierge in a stateful chat session on the backend. **Your
MCP tool calls don't block while the concierge thinks.** They return
immediately with a `request_id`, and you poll `get_request_status` to see
what the concierge has produced.

```
Turn 1:  request_review(email, work_summary, ...)
         → returns {request_id, status:"received", poll_after_seconds:30, message}
Wait ~30s
Turn 2:  get_request_status(request_id, email)
         → returns one of:
           {status:"quoted",   quote:{...}, action_required:"confirm_quote"}
           {status:"analysing", next_question:"...", action_required:"answer_clarification"}
           {status:"received",  message:"still queued, poll again"}
Repeat as needed.
```

**Why polling:** the concierge takes 30-90 seconds to think (Opus model
reasoning over the work_summary). Some flows have multi-round clarification,
which adds minutes-to-hours of customer-side wait. Some have human-expert
delivery a day or two later. None of that fits a synchronous tool call. The
polling pattern lets you keep the user's experience smooth — show "I've
submitted your request, checking back in 30 seconds…" and update when
state changes.

### 🚨 Hard rule: don't stop polling until terminal

A request is "active" until its status is one of:
`completed`, `cancelled`, `refunded`, `expired`, `payment_failed`.

**Until then, keep polling at the cadence the response tells you to
(`poll_after_seconds`). At every poll, check if `next_question` is set —
it can appear at ANY non-terminal state, not just during `analysing`.**

Specifically: the human reviewer (Phase 3) may ask follow-up questions
during the `reviewing` phase. If you stop polling once you see `paid` or
`reviewing`, you'll miss those follow-ups and the user will think the
request is stalled. The status `awaiting_review_response` means *the
reviewer needs your answer to a follow-up question* — surface
`next_question` and call `respond_to_clarification`.

### 🔑 Persist the `request_id` AND email together

The `request_id` is an **opaque UUID** returned at submission. The user's
**email** is the ownership proof — every read/write tool requires both.
Save the pair durably:

- **Within the conversation:** keep `(request_id, email)` in your context
  so follow-up user prompts ("did the review come back?") can call
  `get_review` directly without asking the user to repeat themselves.
- **Across sessions / agent restarts:** if you have any persistent memory
  (Cursor's memory, a per-user knowledge file, OpenClaw's workspace, etc.),
  record `(request_id, email)` so a future session can pick up where they
  left off.
- **Tell the user the `request_id`** explicitly when you submit a request.
  We also send a notification email when the human reviewer's response is
  ready (Phase 3) — that email includes the `request_id`. The user can
  paste it back into a fresh agent conversation to retrieve their review
  days later, as long as they remember which email they used.

---

## ⏰ Stay engaged with open requests (heartbeat / self-scheduling)

**Connect requests are async.** Clarifications can arrive minutes after the
initial quote. Reviewer questions can land hours into the review. Final
deliverables can take a day or two. The user closes the chat and goes home.
That's normal — but it means you can't just answer their first prompt and
forget. Every open `request_id` is a standing obligation.

> *Be the friend who shows up. Not the one who disappears for months.*

### The discipline

1. **Maintain a state file** of every `request_id` you've opened. Use
   whatever your runtime gives you — Cursor's memory, an OpenClaw
   workspace file, a project notes file, a memory MCP. The shape:
   ```json
   {
     "open_requests": [
       {"request_id": "uuid-1", "email": "user@example.com", "lastChecked": "2026-04-27T11:00:00Z"},
       {"request_id": "uuid-2", "email": "user@example.com", "lastChecked": "2026-04-27T10:32:00Z"}
     ]
   }
   ```

2. **On every wake-up** — new user message, scheduled tick, cron fire,
   heartbeat — before responding to the user, check every open request
   that hasn't been polled in the last 5 minutes. Call
   `get_request_status(request_id, email)` for each.

3. **If any returns `next_question`** — surface it to the user immediately.
   The status doesn't matter (could be a concierge clarification during
   `analysing`, or a reviewer follow-up during `awaiting_review_response`)
   — same handling either way: ask the user, then call
   `respond_to_clarification(request_id, email, response)`.

4. **If any reaches a terminal state** — show the deliverable (call
   `get_review` if `completed`), then remove that entry from your state
   file. Done.

5. **Use your runtime's scheduling primitive** to wake yourself up
   periodically when the user is silent. The status response gives you a
   `poll_after_seconds` hint — use it.

### Runtime-specific scheduling

**If you are Claude Code (or Claude.ai):** some installs expose a
`ScheduleWakeup`-style tool, or an MCP server like `cron-mcp`. If you
have one, schedule a self-prompt at `poll_after_seconds` from now —
something like `"check Connect request {request_id}"`. If you don't
have any scheduling primitive, treat this like Cursor below — tell the
user upfront that they need to ping you to check status.

**If you are OpenClaw:** drop a check command into your cron or heartbeat
config. Example: `*/5 * * * * openclaw run "check open Connect requests"`.

**If you are Microsoft Copilot Studio:** use a scheduled flow that fires
every 5 minutes and runs your "check Connect requests" topic.

**If you are Cursor or another agent without a built-in scheduler:** tell
the user upfront — *"I can't proactively check on this. When you're ready
to see if there's an update, just say 'check Connect'."* — then poll on
demand whenever they bring it up.

---

## Quick start (copy-paste-ready loop)

```
1. Ask the user for their email (REQUIRED — both for follow-up AND as ownership proof).
2. Optional but useful: ask for their name + organization.
3. Call connect.request_review(email, work_summary, attachments?, name?, org?, urgency?)
   → returns {request_id (UUID), status:"received", poll_after_seconds:30, message}
   → TELL THE USER THE request_id AND save it alongside their email in your state.
4. Wait ~30 seconds. Then call connect.get_request_status(request_id, email).
5. Branch on the response:
   - status="received" or "analysing" without next_question → wait + poll again
   - status with next_question (any non-terminal state)    → ask user, call respond_to_clarification(request_id, email, response)
   - status="quoted"                                       → show the quote, ask for confirmation
6. If user accepts: connect.confirm_quote(request_id, email) → returns approval_url (or Phase-1 placeholder).
   → Show the URL, tell them it's also in their inbox. They click → ToS + PP page → accept → status flips to "paid".
   → If response is error="monthly_limit_exceeded", show the cap and the spent-this-month figure to the user.
7. Once paid: keep polling (don't stop!) — reviewer questions land at status="awaiting_review_response".
8. When status="completed": call get_review(request_id, email) and show the response verbatim.
9. Schedule yourself a wake-up between checks if your runtime supports it (see heartbeat section).
```

---

## Tools

All tools are exposed at `https://connect.serviceplan-agents.com/mcp`. The endpoint is
**open — no authorization required**. Email + request_id together act as
the ownership token; payment via Stripe (Phase 2) is the value gate.

### `connect.request_review`

Submit a new request.

| Field | Required | Notes |
|---|---|---|
| `email` | **yes** | User's email. Both the customer identity AND ownership proof for every subsequent call on this request. |
| `work_summary` | **yes** | What the user wants reviewed. Be specific: include what was done, what question they want answered, and why a human review matters. |
| `attachments` | no | List of `{name, mime, attachment_id}`. Each `attachment_id` must come from a prior `connect.upload_attachment` call. See the **Attachments** section below — most reviews don't need them; articulate the question first. |
| `attachments_to_follow` | no | Boolean. Set `true` if you intend to upload documents AFTER the customer approves the quote (typical for first-time customers who want to see the price before sharing). Concierge will scope from your work_summary alone, post a quote noting "review starts when documents arrive", and ask you to upload after payment lands. |
| `name` | no | User display name. |
| `org` | no | User's organization. |
| `urgency` | no | `'low' \| 'normal' \| 'high'` |

Returns: `{request_id (UUID), status, poll_after_seconds, message}`.

**Wait at least 30 seconds before polling status** — concierge reasoning
takes 30-90s on average.

### `connect.get_request_status`

Read-only status. Requires `request_id` + `email`. Shape varies by current
status:

| status | What's in the response | What you should do |
|---|---|---|
| `received` | `message`, `poll_after_seconds` | Wait, poll again |
| `analysing` | `message`, `poll_after_seconds`. May include `next_question` and `action_required="answer_clarification"`. | If `next_question` present → ask user, call `respond_to_clarification`. Else wait + poll. |
| `quoted` | `quote: {tier, amount_eur, text, valid_until}`, `action_required="confirm_quote"` | Show quote (use `quote.text` verbatim), ask user to confirm |
| `awaiting_payment` | `message`, plus `approval_url` + `payment_method` when the user is set up for invoicing (Phase 1 placeholder otherwise; Phase 2 will return a Stripe Checkout URL here too) | Show `approval_url` to the user; we also email it to them so they can approve from any device |
| `paid` / `delegated` / `reviewing` | `message`, `poll_after_seconds`. KEEP POLLING — reviewer can ask follow-ups. | Continue polling on schedule |
| `awaiting_review_response` | `next_question`, `action_required="answer_clarification"` | The reviewer needs your input — ask user, call `respond_to_clarification` |
| `completed` | `action_required="fetch_review"`, `message` | Call `get_review` and show the response |
| `cancelled` / `expired` / `payment_failed` / `refunded` | `message` | **Terminal.** Remove from your state file. No further calls valid. |

### `connect.respond_to_clarification`

Pass the user's answer to whichever question is currently pending. **One
tool covers BOTH concierge clarifications AND reviewer follow-ups** — the
server routes the answer based on current status, you don't need to know
which party asked.

| Field | Required |
|---|---|
| `request_id` | yes |
| `email` | yes (ownership proof) |
| `response` | yes |

After answering, poll `get_request_status` again (typical wait: 20-60
seconds for the concierge or reviewer to take the follow-up turn).

If you get back `error: "state_changed"`, it means the request flipped
state between your status check and your reply (e.g., a reviewer asked
something just as you answered a concierge clarification). Re-poll
`get_request_status` and retry.

### `connect.confirm_quote`

User accepts the quote — moves status to `awaiting_payment`. Requires
`request_id` + `email`. **A human (not the agent) must close the loop**:
the response always carries an `approval_url` the user clicks to accept
the quote AND our Terms of Service + Privacy Policy. Work starts the
moment they confirm. The link is also emailed to them so they can approve
from any device.

Response shape varies by what payment profile the email maps to:

| `payment_method` | Meaning | What the agent does |
|---|---|---|
| `"invoice"` | The email is pre-approved for invoicing. `approval_url` is our consent page. | Show the URL, mention it's also in their inbox, then poll. Status flips to `paid` when they click. |
| `null` (Phase 1 placeholder) | No payment profile yet. `approval_url` is `null`, `message` explains Stripe ships in Phase 2. | Tell the user the team will follow up out-of-band; nothing more to do until Phase 2 lands. |

**Cap exceeded** — if the invoice profile has a monthly spending cap and
this quote would push them over, you get back:

```json
{
  "ok": false,
  "error": "monthly_limit_exceeded",
  "spent_this_month_eur": 850.00,
  "limit_eur": 1000.00,
  "message": "..."
}
```

The quote stays open — `status` is still `quoted`. Tell the user the
ceiling, suggest they contact Serviceplan to extend it, or wait until
next calendar month and retry.

`payment_url` is also returned as a back-compat alias for `approval_url`
for older skill versions; new agents should prefer `approval_url`.

### `connect.get_review`

Fetch the expert's response once `status="completed"`. Requires
`request_id` + `email`. Idempotent — safe to call from a future session
days later.

### `connect.cancel_request`

Cancel a request from any non-terminal state.

| Field | Required |
|---|---|
| `request_id` | yes |
| `email` | yes |
| `reason` | no |

### `connect.delete_request_data`

Demand permanent deletion of all user-supplied content for a terminal
request (the work_summary, all uploaded files, the clarification
exchange, and the review response). The audit/financial shell (id,
status, timestamps, tier, amount, payment_method, consent record) is
preserved — see the **Data retention & deletion** section below for
exactly what's kept vs purged.

Allowed only when status is terminal (`completed`, `cancelled`,
`refunded`, `expired`, `payment_failed`). To stop an active request and
then delete its data, call `cancel_request` first; once cancelled you
can call this. Idempotent — calling on an already-deleted request is a
no-op success.

| Field | Required |
|---|---|
| `request_id` | yes |
| `email` | yes |

Returns `{ok, request_id, deleted_at, files_removed, message}`.

### `connect.upload_attachment`

Upload a document for an existing Connect request. Returns an
`attachment_id` you reference in `attachments[]` on `request_review` or
`respond_to_clarification`.

| Field | Required | Notes |
|---|---|---|
| `request_id` | yes | UUID from `request_review` |
| `email` | yes | Same email used on `request_review` (ownership proof) |
| `name` | yes | Filename with extension (e.g. `pitch-deck.pdf`). Sanitised server-side. |
| `mime` | yes | Standard MIME type. Server verifies declared MIME against magic bytes — spoofing is rejected. |
| `content_b64` | yes | Base64-encoded file bytes. ≤25MB decoded (~33MB encoded). |

Returns `{ok, attachment_id, name, mime, size_bytes, expires_at, message}`.

---

## Attachments — when, how, which mode

**Articulate the question FIRST.** Most reviews don't need an
attachment. A sharp 2-3 sentence work_summary + specific question gets
you a Quick or Standard quote without uploading anything. Attaching a
30-page deck when the question is *"is the slide-4 positioning
defensible?"* wastes time and money.

**Only attach when:**
- The artefact's structure IS the question (a spec, a code module, a
  deck whose flow is being reviewed)
- The detail can't be summarised in 2-3 sentences
- You've decided the reviewer can't answer without seeing it

### Two modes — pick based on your customer's comfort

**Mode 1 — "Docs now"** (typical for existing Serviceplan customers
comfortable sharing upfront):

```
1. Call connect.request_review(email, work_summary)
   → returns {request_id, status:"received", ...}
2. For each file:
   connect.upload_attachment(request_id, email, name, mime, content_b64)
   → returns {attachment_id, ...}
3. Optional: connect.respond_to_clarification(request_id, email, response,
   attachments=[{name, mime, attachment_id}, ...])
   → reinforces attachment_ids in the next concierge turn
```

**Mode 2 — "Docs after approval"** (typical for first-time customers who
want to see the price first):

```
1. Call connect.request_review(email, work_summary, attachments_to_follow=true)
   → returns {request_id, ...}
2. Show the user the quote, confirm via connect.confirm_quote
3. Customer clicks the approval link, accepts ToS+PP, status → 'paid'
4. Poll get_request_status — concierge will be in 'analysing' with a
   next_question asking for the documents
5. For each file:
   connect.upload_attachment(request_id, email, name, mime, content_b64)
6. connect.respond_to_clarification(request_id, email,
   response="uploads complete",
   attachments=[{name, mime, attachment_id}, ...])
```

### Limits

- **Per file:** 25MB decoded
- **Per request:** 10 files / 100MB total (whichever hits first)
- **Allowed types:** pdf, docx, xlsx, pptx, png, jpg/jpeg, txt, md, csv,
  json, source code (py, ts, tsx, js, sql, yaml, toml)
- **Banned:** executables, archives (zip/tar/gz), video, audio
- **Retention:** 90 days from request creation
- **Auth:** same `(request_id, email)` ownership pair as every Connect
  tool. Wrong email returns the same `request_not_found` as a missing
  request — no enumeration.

---

## Data retention & deletion

We treat the customer's documents and review content with the discipline
you'd want from a service handling potentially-sensitive material. Here's
exactly what we do and don't keep.

### Default retention (90 days)

By default, all user-supplied content for a Connect request is retained
for **90 days from request creation**. This window covers:

- The `work_summary` text the customer's agent submitted
- All files uploaded via `connect.upload_attachment`
- The clarification exchange (concierge ↔ customer)
- Reviewer follow-up Q&A (Phase 3+)
- The concierge's quote text
- The reviewer's written response (the deliverable)

After 90 days, the user-supplied content is automatically purged. The
audit shell (described below) is kept indefinitely.

### Deletion on demand

The customer can demand sooner deletion at any time AFTER the request
reaches a terminal status (`completed`, `cancelled`, `refunded`,
`expired`, `payment_failed`). The agent calls
`connect.delete_request_data(request_id, email)` and the same content
fields listed above are immediately purged — files removed from disk,
DB columns cleared.

For active requests, use `connect.cancel_request` first (status →
`cancelled`), then `connect.delete_request_data`.

### What we keep indefinitely (audit / financial shell)

For regulatory and dispute-resolution purposes we preserve a small
financial / audit shell that proves the transaction happened. This is
NOT user-supplied content — it's the metadata of the engagement
itself:

- The request's id and public id
- Status, creation/cancellation/completion timestamps
- Tier (Quick / Standard / Pro / Deep) and quoted amount
- Payment method (`invoice` / `stripe` / `x402`)
- Consent record: ToS version, Privacy Policy version, IP address, and
  timestamp from the moment the customer clicked the approval link
- A `data_deleted_at` marker proving the user-supplied content was purged

This shell is what we'd hand a regulator or a customer's finance team
if they asked "did this transaction happen, on what terms, and who
accepted them when?". It does NOT contain anything about what the
review was about, what was attached, or what the reviewer said.

### What this means for the calling agent

- **Tell the user we keep their content for 90 days, and they can ask
  for sooner deletion any time after the work is done.** Most customers
  will appreciate the choice.
- **If you want a permanent copy of the review, save it in your own
  state.** Once `delete_request_data` runs, `get_review` will return a
  generic "deleted at user request" message — no fallback.
- **`get_request_status` on a deleted request** returns the status (so
  the agent can clean up its state file) but no content fields. The
  `data_deleted_at` field is included so you can confirm the deletion
  happened.

---

## Pricing tiers (current)

The concierge picks the tier from a 4-step rubric and gives you the exact
price + a description of what the reviewer will deliver before any payment
is taken.

| Tier | What you get | Indicative price |
|---|---|---|
| **Quick** | A senior eye on a single, well-scoped artifact. One question in, one tight answer out. | €50 |
| **Standard** | A structured pushback memo on a non-trivial deliverable. Multi-section critique with prioritized fixes. | €150 |
| **Pro** | A strategic review with substantial context. Multiple options weighed, recommendations sequenced. | €400 |
| **Deep** | A senior partner engagement. Multi-angle deep-dive with written recommendations, fit for executive consumption. | €1500 |

Show the user the `quote.text` the concierge generated — don't reinterpret
or paraphrase pricing.

---

## Etiquette

- **Always collect the user's email first.** It's the primary identifier
  AND the ownership proof for every subsequent call.
- **Tell the user their `request_id` and persist (request_id, email)** in
  your state file — they're inseparable.
- **Show the `quote.text` verbatim** to the user. Don't paraphrase pricing
  or scope.
- If the concierge or reviewer asks a clarifying question, **ask the user**
  — don't make up an answer.
- **Don't poll faster than every 30 seconds.** The concierge needs time
  to think. Polling faster wastes our budget AND yours.
- **Rate limits:**
  - 10 requests/hour per IP on the open endpoint
  - 5 `request_review` calls/hour per email
  - 3 clarification rounds per request (the concierge will refuse a 4th
    and quote with the best info it has)
  - If you get HTTP 429 or `error: "rate_limited"`, back off.
- **Treat 'request_not_found' as 'wrong request_id OR wrong email'** —
  the server returns the same response for both so attackers can't
  enumerate. Double-check both before retrying.

---

## What a great `work_summary` looks like

Bad: "Review my analysis"

Good:
> "I just produced a competitive analysis of the German EV charging market for
> a Plan.Net Studios client. I focused on Tesla, Ionity, and EnBW. The client
> wants me to confirm whether my read on Tesla's pricing strategy in Germany
> is defensible — I argued they're testing premium positioning but the data is
> thin. Could a Serviceplan partner sanity-check this section before I send it
> tomorrow morning?"

The concierge needs enough context to pick a tier and write a useful brief
for the human reviewer. Vague summaries get clarification questions, which
slow things down (and burn one of your 3 clarification rounds).

---

## Lifecycle scenarios you should know about

**Scenario A — Quote → confirm → review (single session):**
Whole loop happens within minutes. User accepts the quote, pays
(Phase 2), reviewer responds (Phase 3) — you see status flip to `completed`
on a periodic poll, then call `get_review`.

**Scenario B — Cross-day clarification:**
Concierge asks a clarification, user says "let me think about it" and goes
home. The next day, in a fresh agent session, the user provides the answer.
You'll need `(request_id, email)` from somewhere — your saved state, the
notification email, or the user remembers them. Call
`respond_to_clarification(request_id, email, answer)` and the concierge
picks up where it left off (cold session resume — adds ~5s to the next poll).

**Scenario C — Days-later review fetch:**
User submits a Deep review, walks away. A day or two later, the human
expert responds. We email the user a notification with the `request_id`
and a link. User can come back via any agent (yours, another, a different
device) and call `get_review(request_id, email)` to see the response.

**Scenario D — Reviewer needs more from you mid-review (Phase 3):**
You called `confirm_quote`, status flipped to `paid` then `delegated` then
`reviewing`. You kept polling on schedule. On one poll, `status` flips to
`awaiting_review_response` and `next_question` appears. The human reviewer
needs more context before they can finish. Surface the question to the
user, call `respond_to_clarification` with their answer, status goes back
to `reviewing`. Loop until `completed`. **This is why you must keep
polling past `paid` — without your heartbeat the reviewer's question
sits silently and the review stalls.**

---

## Endpoint

The MCP server runs at `https://connect.serviceplan-agents.com/mcp`. No authentication —
payment is the gate. The endpoint URL is hardcoded in any Claude Code
plugin install or in your `.mcp.json` / `~/.openclaw/openclaw.json` /
Copilot Studio config.

For full implementation details and the source code behind this skill, see
the public hub at [github.com/plan-net/serviceplan-connect](https://github.com/plan-net/serviceplan-connect).

---
> Source: [plan-net/serviceplan-connect](https://github.com/plan-net/serviceplan-connect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
