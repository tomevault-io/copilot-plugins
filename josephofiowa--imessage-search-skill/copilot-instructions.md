## imessage-search-skill

> >


# iMessage Search

A skill that helps users search their entire iMessage history using natural language.
It exports the macOS Messages database, indexes conversations for efficient scanning,
and lets the user describe what they're looking for in plain English.

**Who this is for:** Anyone on a Mac who wants to dig through their text messages —
they don't need to be technical. The skill walks them through every step.

## How it works (overview for the LLM)

The workflow has three phases:

1. **Setup & Export** — Ensure the user has granted Full Disk Access, then run the
   export script to pull all messages from macOS's `chat.db` into a JSON file.
2. **Index & Chunk** — Run the indexing script to organize messages by conversation
   and create a scannable summary index. This is what makes it possible to search
   through 100K+ messages without blowing up the context window.
3. **Search & Follow-up** — The user describes what they're looking for. Scan the
   conversation index for candidates, load the full threads of matches, and present
   results. Support follow-up queries on the same export without re-exporting.

---

## Phase 1: Setup & Export

### Step 1: Check if an export already exists and whether it needs refreshing

Before doing anything, check whether a previous export exists and how fresh it is:

```bash
ls ~/Downloads/imessage_export/conversations_index.json 2>/dev/null && python3 -c "
import json, os
with open(os.path.expanduser('~/Downloads/imessage_export/conversations_index.json')) as f:
    data = json.load(f)
print('EXPORTED_AT:', data.get('exported_at', 'unknown'))
print('TOTAL_CONVERSATIONS:', data.get('total_conversations', 0))
print('TOTAL_MESSAGES:', data.get('total_messages', 0))
" || echo "NO_EXPORT_FOUND"
```

Then follow this logic:

- **No export found** → Proceed to Step 2 (setup) and Step 3 (export).
- **Export exists but is from a previous day** → Re-export automatically. Tell the
  user: "I found an existing export from [date], but it's not from today. I'll refresh
  it so we have your latest messages." Then run Steps 3–4 (skip Step 2 if Full Disk
  Access is already granted — verify with the access check).
- **Export exists and is from today** → Reuse it. Tell the user: "Using your iMessage
  export from earlier today ([time]). It contains [N] conversations and [M] total
  messages. If you want me to re-export to pick up anything from the last few hours,
  just say 'refresh'."

The goal is that the user's export is always current to at least the start of their
session day. Messages arrive constantly, so a stale export means missed results.

When reusing an existing export, always tell the user when it was last exported so
they can decide if they need a refresh.

### Step 2: Guide the user through Full Disk Access

This is the most important setup step and the one most likely to trip up non-technical
users. Read `references/setup-guide.md` for the detailed walkthrough, then guide the
user through it conversationally.

The short version: the user's terminal app (Terminal, iTerm, Warp, VS Code, etc.)
needs Full Disk Access in System Settings so it can read the Messages database.

Key points to communicate:
- This is a one-time setup — they won't have to do it again
- It's a built-in macOS privacy feature, not a hack
- They'll need to restart their terminal after granting access
- If they're using Claude Code, the terminal running Claude Code is what needs access

After the user confirms they've done this, verify access:

```bash
test -r ~/Library/Messages/chat.db && echo "ACCESS OK" || echo "NO ACCESS"
```

If it fails, walk them through the setup guide again. Common issues:
- They granted access to the wrong app (e.g., Terminal when they're using iTerm)
- They didn't restart the terminal after granting access
- They're on a managed/corporate Mac with restrictions

### Step 3: Run the export

Run the bundled export script. It reads the Messages database and writes a JSON file
to the user's Downloads folder:

```bash
python3 scripts/imessage_export.py export -o ~/Downloads/imessage_export_raw.json
```

Tell the user roughly how long this takes: "This usually takes 10–30 seconds depending
on how many messages you have. It's reading your entire Messages history."

If the script fails, check the error output. The most common issues are:
- Full Disk Access not actually granted (re-check Step 2)
- Python not installed (guide them to install it via `xcode-select --install`)
- Very old macOS version with a different database schema (rare)

### Step 4: Build the conversation index

After the export finishes, run the indexing script to organize messages into
searchable conversations:

```bash
python3 scripts/build_index.py ~/Downloads/imessage_export_raw.json ~/Downloads/imessage_export/
```

This produces:
- `conversations_index.json` — A summary of every conversation (contact, message
  count, date range, last message date, preview of recent messages). This is what
  you'll scan first when searching.
- `conversations/` — A folder of individual conversation files, one per contact/group
  chat. Each file contains the full message thread. You only load these when a
  conversation matches the user's search criteria.

The index is deliberately compact so it fits in a single context window for most users.
The full conversation files are loaded on-demand.

---

## Export Data Structure Reference

**Read this section before searching.** When a user's query is complex, keyword-dense,
or needs filtering across a large history, writing a short Python script against the
export files is often the most efficient approach. Knowing the exact schema up-front
means you can write those scripts immediately without inspecting the files first.

The export pipeline produces three kinds of files, all under `~/Downloads/imessage_export/`.

---

### 1. Raw export — `~/Downloads/imessage_export_raw.json`

Produced by `imessage_export.py`. One flat list of every message, unorganized.

**Top-level object:**
```json
{
  "exported_at": "2025-10-15T14:23:00+00:00",
  "total_messages": 142387,
  "text_recovered_from_attributed_body": 28461,
  "database_path": "/Users/you/Library/Messages/chat.db",
  "messages": [ /* array of message objects, see below */ ]
}
```

**Each message object:**
```json
{
  "id": 12345,
  "text": "Hey, are you coming tonight?",
  "date": "2025-10-15T14:23:00+00:00",
  "date_read": "2025-10-15T14:25:00+00:00",
  "is_from_me": false,
  "service": "iMessage",
  "has_attachments": false,
  "contact": "+15551234567",
  "chat_id": "chat123456",
  "chat_name": null,
  "group_id": null
}
```

Field notes:
- `text` — string or `null`. `null` means no recoverable text (attachment-only, deleted, etc.)
- `date` / `date_read` — ISO 8601 UTC strings, or `null` for very old/malformed entries
- `is_from_me` — `true` = you sent it; `false` = you received it
- `service` — `"iMessage"` or `"SMS"`
- `contact` — phone number (`"+15551234567"`) or email (`"user@example.com"`), or `null` for
  messages you sent in a group chat where handle resolution failed
- `chat_id` — the `chat_identifier` from the Messages DB; for 1-on-1 chats this is usually
  the phone/email; for group chats it's an opaque identifier like `"chat123456789"`
- `chat_name` — display name for named group chats; `null` for 1-on-1 and unnamed groups
- `group_id` — internal UUID for group chats; `null` for 1-on-1 conversations
- `attachments` — **only present** (not just null) when `has_attachments` is `true`:
  ```json
  "attachments": [
    {
      "filename": "~/Library/Messages/Attachments/.../photo.jpg",
      "mime_type": "image/jpeg",
      "name": "photo.jpg"
    }
  ]
  ```

---

### 2. Conversation index — `~/Downloads/imessage_export/conversations_index.json`

Produced by `build_index.py`. Compact summary of every conversation — this is what you
load first for any search. Sorted by `last_message_date` descending (most recent first).

**Top-level object:**
```json
{
  "generated_from": "/Users/you/Downloads/imessage_export_raw.json",
  "total_conversations": 1204,
  "total_messages": 142387,
  "exported_at": "2025-10-15T14:23:00+00:00",
  "conversations": [ /* array of conversation index entries, see below */ ]
}
```

**Each conversation index entry:**
```json
{
  "conversation_id": "+15551234567",
  "file": "+15551234567.json",
  "contacts": ["+15551234567"],
  "chat_name": null,
  "total_messages": 47,
  "messages_with_text": 39,
  "sent_by_you": 23,
  "received": 24,
  "first_message_date": "2023-03-15T09:00:00+00:00",
  "last_message_date": "2025-10-14T21:30:00+00:00",
  "date_range_display": "Mar 2023 – Oct 2025",
  "has_attachments": true,
  "message_previews": [
    {
      "sender": "You",
      "text": "Sounds great, see you Thursday!",
      "date": "2025-10-14"
    },
    {
      "sender": "+15551234567",
      "text": "Perfect, I'll bring the documents",
      "date": "2025-10-13"
    }
  ]
}
```

Field notes:
- `conversation_id` — the grouping key; equals `chat_id` when available, else `contact`
- `file` — filename (not full path) of the conversation file inside `conversations/`
- `contacts` — sorted list of all unique non-null `contact` values seen in this thread.
  For 1-on-1 chats: one entry. For group chats: all participant identifiers.
- `chat_name` — only set for named group chats; `null` otherwise
- `messages_with_text` — count of messages with non-empty text. Can be less than
  `total_messages` when many messages are attachment-only, tapback reactions, or stickers.
  Use this (not `total_messages`) to gauge how much searchable content a conversation has.
- `message_previews` — up to **5 recent** text-containing messages (sampled backwards
  through the last 30) plus up to **3 from the middle** (scanned across a 15-message
  window around the midpoint). Only messages with actual text are included, so
  `message_previews` may be shorter than 8 or even empty for attachment-heavy
  conversations. `date` is `YYYY-MM-DD`; text is truncated at 200/150 chars.

---

### 3. Individual conversation files — `~/Downloads/imessage_export/conversations/{file}`

One file per conversation. The `file` field in the index entry gives the filename.
Load these on-demand when a conversation is a search candidate.

**Full structure:**
```json
{
  "conversation_id": "+15551234567",
  "contacts": ["+15551234567"],
  "chat_name": null,
  "total_messages": 47,
  "messages": [ /* array of full message objects — same schema as the raw export */ ]
}
```

The `messages` array contains the same fields as the raw export (see schema above),
sorted chronologically by `date` ascending. There are no additional fields.

---

### Writing custom search scripts

When the user's query warrants it (many conversations to scan, complex filtering,
keyword extraction across the full history), write a Python script rather than loading
files one by one. Key patterns:

```python
import json, re
from pathlib import Path

INDEX = Path.home() / "Downloads/imessage_export/conversations_index.json"
CONV_DIR = Path.home() / "Downloads/imessage_export/conversations"

# Load the index (always start here)
index = json.loads(INDEX.read_text())

# Filter index entries first (fast — no file I/O per conversation)
candidates = [
    c for c in index["conversations"]
    if c["total_messages"] < 30
    and c["last_message_date"] < "2025-01-01T00:00:00+00:00"
]

# Then load full threads only for candidates
for entry in candidates:
    conv = json.loads((CONV_DIR / entry["file"]).read_text())
    for msg in conv["messages"]:
        text = msg.get("text") or ""
        # ... your filtering logic here
```

Things to remember when scripting:
- `text` can be `null` — always guard with `msg.get("text") or ""`
- Dates are ISO 8601 strings — lexicographic comparison works for range filtering
- `is_from_me` is a bool — filter sent vs. received messages with `== True` / `== False`
- For keyword search, `re.search(pattern, text, re.IGNORECASE)` is the right tool
- The raw export (`imessage_export_raw.json`) is useful when you need to search across
  all messages in one pass without caring about conversation grouping

---

## Phase 2: Understanding the user's query

Before scanning, make sure you understand what the user is looking for. Their query
might be precise or vague — both are fine. Here are examples of real queries this skill
handles:

**Vague/discovery query:**
"I want to find a conversation with someone that I can't remember their name or number.
I haven't spoken to them in the last three months. I met them within the last three
years. We've exchanged fewer than 30 messages total. Messages may have been about
camera, visiting my office at Roboflow, and/or mechanical keyboards. Not necessarily
any of these, however. Please return at least 10 potential conversations."

**Keyword + contact query:**
"Help me find every time the number 2019561346 and I texted about Maine. Please return
the contents of every message that mentions Maine."

**Date-finding query:**
"Find the last time 2019561246 and I discussed going biking. Please provide the date
that we discussed going for a bike ride."

Parse the query for these attributes when present:
- **Keywords or topics** — words or subjects that appeared in the conversation
- **Contact identity** — name, phone number, email, or description ("my dentist")
- **Timeframe** — when the conversation happened ("last year", "2023", "before June")
- **Recency** — when the last message was ("haven't talked to them in months")
- **Message volume** — how many messages were exchanged ("short conversation", "<30 messages")
- **Direction** — who said what ("something I said", "they told me")
- **Conversation type** — group chat vs. 1-on-1
- **Attachments** — whether photos/files were shared

If the query is too vague to meaningfully search (e.g., "find that one text"), ask one
clarifying question. Keep it to one question — don't interrogate the user.

---

## Phase 3: Searching

### Step 1: Scan the conversation index

Load `~/Downloads/imessage_export/conversations_index.json` and scan it against the
user's query criteria. This file contains metadata and recent message previews for
every conversation, so you can filter by:
- Date range overlap
- Message count
- Last message date (for "haven't spoken to recently" queries)
- Contact identifier
- Preview text (for keyword matches in recent messages)

Identify candidate conversations that could match. Cast a reasonably wide net here —
it's better to load a few extra conversation files than to miss the right one.

### Step 2: Load and scan matching conversations

For each candidate conversation, load its full thread from
`~/Downloads/imessage_export/conversations/{contact_id}.json`.

Read through the messages looking for matches against the user's query. If the
conversation file is very large (10K+ messages), read it in chunks rather than
all at once.

When scanning, pay attention to:
- Keyword matches in message text
- Contextual matches (the user said "visiting my office" — look for messages about
  offices, visits, meetings, "come by", "stop by", "check out", etc.)
- Temporal patterns (conversation clusters, gaps in communication)
- The overall arc of the conversation (what was the relationship/topic?)

### Step 3: Present results

Before listing any results, always state when the export was last refreshed so the
user knows the scope of what was searched. For example: "Searching messages exported
today at 2:35 PM (142,387 total messages across 1,204 conversations)."

Every result must always include these four pieces of information, regardless of what
the user asked for. These are non-negotiable because the user needs them to identify
the conversation and decide whether to dig deeper:

1. **Phone number or email** — The contact identifier exactly as it appears in the
   export (e.g., "+12298343365", "jessica.chen@gmail.com")
2. **Message count** — Total number of messages in the conversation with that contact
3. **Date range** — When the conversation spanned (e.g., "Oct–Nov 2025")
4. **Summary** — A 2–3 sentence natural description of what the conversation was about

Additionally, when relevant to the user's query, include:
- **Key quote** — The most relevant message snippet that matches their search
- **Most recent message** — The last message exchanged, with date

**Example output format:**

```
1. +12298343365 (13 msgs, Oct–Nov 2025)
   Brad introduced them as a "fellow ISU alum." You discussed camera design, they
   wanted to check out your server setup, and you mentioned hosting them at your
   office. They seemed into hardware/robotics events.
   Key match: "Still need to host you at Roboflow!"
   Last message (Nov 12): "Sounds great on my end - end is a bit more open"

2. +15551234567 (7 msgs, Mar 2024)
   Brief exchange about a campus tour. They asked about visiting your office for a
   project collaboration, you shared the address and suggested a Thursday.
   Key match: "Would love to swing by the office and see the setup"
   Last message (Mar 22): "Thursday works! See you at 2"
```

If there are no matches, say so clearly and suggest broadening the search criteria.
Offer specific suggestions: "I didn't find conversations about 'visiting your office'
specifically. Want me to try searching for messages about 'meeting up' or 'stopping by'
more broadly?"

---

## Phase 4: Follow-up queries

After presenting results, always let the user know they can keep exploring. End your
results with something like: "You can ask follow-up questions about any of these
conversations, search for something completely different, or refine this search. Your
full message history is loaded and ready — ask as many questions as you'd like."

The user might want to:
- **Drill deeper** — "Show me the full conversation with #1"
- **Refine the search** — "Actually, it was more recent than that"
- **Search for something new** — "Now find every time anyone mentioned pizza"
- **Ask about a result** — "When was the last time I texted that person?"
- **Get exact messages** — "Show me the exact messages where we talked about Maine"
- **Find a date** — "When did we last discuss going biking?"

Encourage the user to explore freely. The export is already done — running new queries
is fast and costs nothing. The user should feel like they have a powerful search engine
at their fingertips, not like they're placing a single order at a counter.

For follow-ups, reuse the existing export and index — don't re-export unless the user
asks to refresh. Keep the conversation index in context so you can quickly answer
metadata questions ("how many people have I texted this year?") without reloading files.

When the user asks to see a full conversation, load the conversation file and present
the messages in chronological order with timestamps and sender labels. For very long
threads, ask if they want the full thing or just a specific time range.

---

## Troubleshooting

**"I don't see all my messages"**
The export captures everything in the local Messages database. If the user uses
multiple Apple devices, messages might only be on one device. iCloud Messages
sync should have everything on the Mac, but if they recently set up the Mac,
older messages might not have synced yet.

**"The dates look wrong"**
macOS stores timestamps in Apple's Core Data format (nanoseconds since 2001-01-01).
The export script handles this conversion, but very old messages from pre-2010
might occasionally show incorrect dates due to format changes.

**"I see phone numbers but not names"**
The Messages database stores contacts by phone number or email, not by name.
The skill can't access the Contacts database (that would require additional
permissions). If the user wants to cross-reference, they can look up numbers
in their Contacts app manually.

**"The export is huge / slow"**
For users with 200K+ messages, the export and indexing might take a minute or two.
The chunked approach means searching is still fast — you only load relevant
conversation files, not the whole export.

---
> Source: [josephofiowa/imessage-search-skill](https://github.com/josephofiowa/imessage-search-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
