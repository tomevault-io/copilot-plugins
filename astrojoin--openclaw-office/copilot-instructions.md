## openclaw-office

> Read, create, and edit Word, PowerPoint, Excel, and OneDrive files; send and read Outlook email; manage Outlook calendar — all via Microsoft Graph API.


# OpenClaw Office

Use this skill when the user needs to work with Microsoft 365 cloud files (OneDrive, Word, PowerPoint, Excel), Outlook email, or Outlook calendar.

## Prerequisites

1. **Azure app registered** with Device Code flow enabled. Client ID in `config.json`.
2. **Authenticated** — run `python3 scripts/auth.py login` first. Opens a device-code flow; user visits URL and enters code.
3. **Token valid** — tokens auto-refresh; check with `python3 scripts/auth.py status`.

If auth is missing or expired, tell the user to run `auth.py login` before continuing.
4. **Python packages** — install with `pip3 install --break-system-packages python-docx python-pptx msal requests pillow lxml`:
   - `python-docx` — Word (.docx) creation and editing
   - `python-pptx` — PowerPoint (.pptx) creation and editing
   - `msal` — Microsoft Authentication Library (OAuth2 device-code flow)
   - `requests` — HTTP client for Graph API calls
   - `pillow` — Image handling for `add_picture` in Word/PowerPoint
   - `lxml` — XML parsing (dependency of python-docx/python-pptx)

## Architecture

```
auth.py          → OAuth2 device-code flow + token refresh (MSAL)
graph_client.py  → Low-level Graph API HTTP client + Excel workbook/session
onedrive.py      → Cloud file operations (list, upload, download, move, copy, delete, search)
word.py          → Offline .docx CRUD (python-docx, bytes-in/bytes-out)
powerpoint.py    → Offline .pptx CRUD (python-pptx, bytes-in/bytes-out)
mail.py          → Outlook mail (list, search, read, send, reply, forward, move, delete)
outlook_calendar.py → Outlook calendar (list, create, update, delete, accept/decline)
```

Key principle: `word.py` and `powerpoint.py` are **purely offline** editors — they operate on bytes, no network calls. `onedrive.py` is the cloud bridge: it downloads bytes, passes them to word/pptx for editing, then uploads the result. Excel operations go through `graph_client.py` workbook/session endpoints (server-side editing).

## Capabilities

> ⚠️ **This section is API reference only.** Do NOT copy-paste these code blocks into `exec`. For actual execution, use the **Workflows** section below — it has the correct `exec` inline pattern with triggers.

### 1. Authentication

```bash
python3 scripts/auth.py login    # Device-code flow
python3 scripts/auth.py status   # Check token validity
python3 scripts/auth.py token    # Print current access token
python3 scripts/auth.py logout   # Delete stored tokens
```

Programmatic: `auth.get_access_token()` returns a valid token or `None`.

### 2. OneDrive Files

```python
from onedrive import OneDrive
od = OneDrive()

od.list("/Documents")            # List folder contents
od.info("/Documents/file.docx")  # File metadata
od.search("report")              # Search across OneDrive
od.download("/Documents/f.docx") # → bytes
od.upload("/Documents/new.docx", data_bytes)  # Upload (<4MB simple, >4MB session)
od.move("/f.docx", "/Archive")   # Move file
od.copy("/f.docx", "/Backup")    # Copy file
od.rename("/old.docx", "new.docx")
od.delete("/unwanted.docx")
od.create_folder("/Projects", "Q3")
```

### 3. Word (.docx)

```python
from onedrive import OneDrive
od = OneDrive()

# Read
text = od.docx_read("/Documents/report.docx", mode="plain")       # Plain text
text = od.docx_read("/Documents/report.docx", mode="structured")  # JSON with styles

# Create
od.docx_create("/Documents/new.docx", operations=[
    {"method": "add_heading", "args": ["Title", 1]},
    {"method": "add_paragraph", "args": ["Body text"]},
    {"method": "add_table", "kwargs": {"rows": 3, "cols": 2, "data": [["A","B"],["C","D"]], "style": "Table Grid"}},
])

# Edit (download → modify → upload)
od.docx_edit("/Documents/report.docx", operations=[
    {"method": "replace_text", "args": ["old text", "new text"]},
    {"method": "add_paragraph", "args": ["Appended paragraph"]},
    {"method": "add_heading", "args": ["Section 2", 2]},
    {"method": "add_page_break"},
])
```

**Word operations:** `add_paragraph`, `add_heading`, `add_table`, `add_page_break`, `add_picture`, `remove_paragraph`, `replace_text`. Paragraph-level ops use `paragraph_index` in kwargs. Table ops use `table_index`.

### 4. PowerPoint (.pptx)

```python
from onedrive import OneDrive
od = OneDrive()

# Read
text = od.pptx_read("/Documents/deck.pptx", mode="plain")
text = od.pptx_read("/Documents/deck.pptx", mode="structured")

# Create
od.pptx_create("/Documents/new.pptx", operations=[
    {"method": "add_slide", "kwargs": {"layout": "Title Slide", "title": "My Deck"}},
    {"method": "add_slide", "kwargs": {"layout": "Title and Content", "title": "Agenda", "body": "Item 1\nItem 2"}},
])

# Edit
od.pptx_edit("/Documents/deck.pptx", operations=[
    {"method": "add_textbox", "kwargs": {"slide_index": 0, "text": "Note", "left": 1, "top": 4}},
    {"method": "replace_text", "args": ["Draft", "Final"]},
    {"method": "add_notes", "kwargs": {"slide_index": 0, "text": "Speaker notes here"}},
    {"method": "add_table", "kwargs": {"slide_index": 1, "rows": 3, "cols": 3, "data": [["X","Y","Z"]]}},
    {"method": "add_picture", "kwargs": {"slide_index": 0, "path": "/tmp/chart.png", "left": 1, "top": 2}},
])
```

**PowerPoint operations:** `add_slide`, `remove_slide`, `add_textbox`, `add_table`, `set_table_cell`, `add_picture`, `replace_text`, `add_notes`. Layout names: Title Slide, Title and Content, Section Header, Two Content, Comparison, Title Only, Blank.

### 5. Excel (.xlsx)

All Excel operations use Graph API workbook sessions (server-side editing, no local file manipulation).

```python
from onedrive import OneDrive
od = OneDrive()

od.xlsx_list_worksheets("/Documents/data.xlsx")
od.xlsx_read_range("/Documents/data.xlsx", "Sheet1", "A1:D10")
od.xlsx_write_range("/Documents/data.xlsx", "Sheet1", "A1:B2", [["Name","Value"],["X","42"]])
od.xlsx_add_worksheet("/Documents/data.xlsx", "Summary")
od.xlsx_add_table("/Documents/data.xlsx", "Sheet1", "A1:D5", has_headers=True)
od.xlsx_add_formula("/Documents/data.xlsx", "Sheet1", "E2", "=SUM(B2:D2)")
od.xlsx_get_used_range("/Documents/data.xlsx", "Sheet1")
```

### 6. Outlook Mail

```python
from mail import Mail
m = Mail()

m.list_inbox(count=10)                           # Recent inbox
m.list_folder("sent", count=5)                    # Sent folder
m.search("project update")                        # Search all folders
m.read(message_id)                                # Full message
m.read_body(message_id)                           # Body text only
m.mark_read(message_id)                           # / mark_unread
m.move(message_id, "junkemail")                   # Move to folder
m.delete(message_id)

# Send
m.send(to=["user@example.com"], subject="Hello", body="Test")
m.send(to=["a@b.com"], subject="Report", body="<h1>Hi</h1>", body_type="html",
       cc=["c@d.com"], attachments=[{"name": "file.pdf", "path": "/tmp/file.pdf"}])

# Reply / Forward
m.reply(message_id, "Thanks!", reply_all=True)
m.forward(message_id, to=["other@example.com"], comment="FYI")
```

### 7. Outlook Calendar

```python
from outlook_calendar import Calendar
cal = Calendar()

cal.list_calendars()                              # All calendars
cal.list_events(days_ahead=7)                     # Upcoming events
cal.list_events(calendar_id="xxx", days_ahead=30)
cal.get_event(event_id)

# Create
cal.create(subject="Team standup",
           start="2026-05-21T09:00:00", end="2026-05-21T09:30:00",
           location="Room 3", attendees=["a@b.com", "c@d.com"])

# Update
cal.update(event_id, subject="Renamed event", start="2026-05-21T10:00:00", end="2026-05-21T11:00:00")

# Delete
cal.delete(event_id)

# Respond to invitations
cal.accept(event_id)
cal.decline(event_id)
cal.tentatively_accept(event_id)
```

## Workflows

Every operation starts with the same guard: **check auth first**.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 auth.py status
```

If expired or missing → tell the user to run `python3 auth.py login` and provide the device code.

All Python workflows below are executed as inline scripts via `exec`. The pattern is:

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
<code>
PYEOF
```

---

### Check or switch Microsoft account

**Trigger:** User asks to log in, log out, switch account, or auth status is invalid.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 auth.py logout
```
```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 auth.py login
```
Give user the URL + code, wait for confirmation.

### List files in OneDrive

**Trigger:** User asks "what files are in my OneDrive", "show me my documents", "list files", or needs to find a file before operating on it.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
items = od.list("/")
for i in items:
    icon = "\U0001f4c1" if i["type"] == "folder" else "\U0001f4c4"
    print(f'{icon} {i["name"]} ({i["size"]} bytes)')
PYEOF
```

### Read a Word document from OneDrive

**Trigger:** User asks to read, view, or extract text from a .docx file on OneDrive.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
text = od.docx_read("/Documents/report.docx", mode="plain")
print(text)
PYEOF
```

Use `mode="structured"` if you need paragraph styles, table data, or element positions for editing decisions.

### Create a new Word document on OneDrive

**Trigger:** User asks to create, write, or make a new .docx file on OneDrive (file does NOT exist yet).

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
od.docx_create("/Documents/new_report.docx", operations=[
    {"method": "add_heading", "args": ["Report Title", 1]},
    {"method": "add_paragraph", "args": ["Introduction text here."]},
    {"method": "add_table", "kwargs": {"rows": 3, "cols": 2, "data": [["Header A", "Header B"], ["val1", "val2"], ["val3", "val4"]], "style": "Table Grid"}},
])
PYEOF
```

Use `docx_create` when the file does **not** exist yet. Pass all content as `operations`.

### Edit an existing Word document on OneDrive

**Trigger:** User asks to edit, update, add to, modify, or append content to a .docx that already exists on OneDrive.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
od.docx_edit("/Documents/report.docx", operations=[
    {"method": "replace_text", "args": ["old text", "new text"]},
    {"method": "add_heading", "args": ["New Section", 2]},
    {"method": "add_paragraph", "args": ["Additional content."]},
])
PYEOF
```

It downloads → applies operations → uploads automatically. The file is never written to disk — all in memory.

### Create a PowerPoint presentation on OneDrive

**Trigger:** User asks to create, make, or build a new .pptx file on OneDrive.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
od.pptx_create("/Documents/deck.pptx", operations=[
    {"method": "add_slide", "kwargs": {"layout": "Title Slide", "title": "Presentation Title"}},
    {"method": "add_slide", "kwargs": {"layout": "Title and Content", "title": "Agenda", "body": "Point 1\nPoint 2\nPoint 3"}},
])
PYEOF
```

### Edit an existing PowerPoint on OneDrive

**Trigger:** User asks to edit, update, add slides or content to a .pptx that already exists on OneDrive.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
od.pptx_edit("/Documents/deck.pptx", operations=[
    {"method": "replace_text", "args": ["Draft", "Final"]},
    {"method": "add_textbox", "kwargs": {"slide_index": 0, "text": "Updated note", "left": 1, "top": 4}},
    {"method": "add_notes", "kwargs": {"slide_index": 0, "text": "Speaker notes"}},
])
PYEOF
```

### List worksheets in an Excel file

**Trigger:** User asks what sheets/tabs are in an Excel file, or you need to know worksheet names before reading/writing.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
sheets = od.xlsx_list_worksheets("/Documents/data.xlsx")
for s in sheets:
    print(s["name"], s.get("id"))
PYEOF
```

### Read data from an Excel range

**Trigger:** User asks to read, view, or extract data from an Excel file on OneDrive.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
data = od.xlsx_read_range("/Documents/data.xlsx", "Sheet1", "A1:D10")
print(data)
PYEOF
```

### Write data to an Excel range

**Trigger:** User asks to write, fill, or update data in an Excel file on OneDrive.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
od.xlsx_write_range("/Documents/data.xlsx", "Sheet1", "A1:B3", [
    ["Name", "Score"],
    ["Alice", 95],
    ["Bob", 87],
])
PYEOF
```

Excel edits are **server-side** via Graph API workbook sessions — no download/upload cycle.

### Add a worksheet to an Excel file

**Trigger:** User asks to add a new sheet/tab to an existing Excel file.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
od.xlsx_add_worksheet("/Documents/data.xlsx", "Summary")
PYEOF
```

### Add a table to an Excel worksheet

**Trigger:** User asks to format a range as a table in Excel, or add a named/structured table.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
od.xlsx_add_table("/Documents/data.xlsx", "Sheet1", "A1:D5", has_headers=True)
PYEOF
```

### Add a formula to an Excel cell

**Trigger:** User asks to add a formula, calculation, or function to an Excel cell.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
od.xlsx_add_formula("/Documents/data.xlsx", "Sheet1", "E2", "=SUM(B2:D2)")
PYEOF
```

### Get the used range of a worksheet

**Trigger:** User asks how much data is in a sheet, or you need to discover the bounds before reading.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
used = od.xlsx_get_used_range("/Documents/data.xlsx", "Sheet1")
print(used)
PYEOF
```

### Send an email

**Trigger:** User asks to send, write, or compose an email via Outlook.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from mail import Mail
m = Mail()
m.send(to=["user@example.com"], subject="Hello", body="Message body")
PYEOF
```

### Read inbox and reply

**Trigger:** User asks to check email, read inbox, see new messages, or reply to an email.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from mail import Mail
m = Mail()
messages = m.list_inbox(count=10)
for msg in messages:
    print(f'{msg["subject"]} from {msg["from"]}')

# Read full message, then reply
msg = m.read(message_id)
m.reply(message_id, "Thanks for the update!")
PYEOF
```

### Check calendar and create events

**Trigger:** User asks about upcoming meetings, schedule, calendar events, or wants to create/modify an event.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from outlook_calendar import Calendar
cal = Calendar()
events = cal.list_events(days_ahead=7)
for e in events:
    print(f'{e["start"]} — {e["subject"]}')

cal.create(subject="Team standup",
           start="2026-05-21T09:00:00", end="2026-05-21T09:30:00",
           location="Room 3", attendees=["a@b.com"])
PYEOF
```

### Upload a local file to OneDrive

**Trigger:** User asks to upload, save, or move a local file to OneDrive.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
with open("/tmp/local_file.pdf", "rb") as f:
    data = f.read()
od.upload("/Documents/uploaded_file.pdf", data)
PYEOF
```

### Download a file from OneDrive to local disk

**Trigger:** User asks to download, save locally, or get a copy of a file from OneDrive.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
data = od.download("/Documents/report.docx")
with open("/tmp/report.docx", "wb") as f:
    f.write(data)
PYEOF
```

### Move, copy, rename, or delete files

**Trigger:** User asks to organize files — move to folder, make a copy, rename, or delete from OneDrive.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
od.move("/file.docx", "/Archive")              # Move to folder
od.copy("/file.docx", "/Backup", "backup.docx")  # Copy with new name
od.rename("/old.docx", "new.docx")               # Rename in place
od.delete("/unwanted.docx")                      # Delete permanently
od.create_folder("/", "Projects")               # New folder
PYEOF
```

### Search for files in OneDrive

**Trigger:** User asks to find, search, or locate a file by name or keyword on OneDrive.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
results = od.search("report")
for r in results:
    print(f'{r["name"]} — {r["webUrl"]}')
PYEOF
```

### Get file metadata from OneDrive

**Trigger:** User asks for file details, size, modified date, or needs the item ID before an Excel operation.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
info = od.info("/Documents/report.docx")
print(info)
PYEOF
```

### Read a PowerPoint file from OneDrive

**Trigger:** User asks to read, view, or extract text from a .pptx file on OneDrive.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from onedrive import OneDrive
od = OneDrive()
text = od.pptx_read("/Documents/deck.pptx", mode="plain")
print(text)
PYEOF
```

Use `mode="structured"` if you need slide details, layout names, or element positions.

### Read an email body only

**Trigger:** User asks to quickly see the content of a specific email without full headers.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from mail import Mail
m = Mail()
body = m.read_body(message_id)
print(body)
PYEOF
```

### Mark email as read or unread

**Trigger:** User asks to mark an email as read, unread, or flag a message.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from mail import Mail
m = Mail()
m.mark_read(message_id)    # or m.mark_unread(message_id)
PYEOF
```

### Reply to or forward an email

**Trigger:** User asks to reply, respond, or forward a specific email.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from mail import Mail
m = Mail()
m.reply(message_id, "Thanks for the update!", reply_all=True)
m.forward(message_id, to=["other@example.com"], comment="FYI")
PYEOF
```

### Update or delete a calendar event

**Trigger:** User asks to modify, reschedule, or cancel an existing calendar event.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from outlook_calendar import Calendar
cal = Calendar()
cal.update(event_id, subject="Renamed", start="2026-05-21T10:00:00", end="2026-05-21T11:00:00")
cal.delete(event_id)
PYEOF
```

### Respond to a calendar invitation

**Trigger:** User asks to accept, decline, or tentatively accept a meeting invite.

```
exec: cd /sandbox/.openclaw/workspace/skills/openclaw-office/scripts && python3 << 'PYEOF'
from outlook_calendar import Calendar
cal = Calendar()
cal.accept(event_id)                # or cal.decline(event_id) or cal.tentatively_accept(event_id)
PYEOF
```

## Quick Reference

| Task | Script | Key method |
|---|---|---|
| Authenticate | `auth.py` | `login` / `get_access_token()` |
| List OneDrive | `onedrive.py` | `od.list(path)` |
| Search OneDrive | `onedrive.py` | `od.search(query)` |
| File metadata | `onedrive.py` | `od.info(path)` |
| Download file | `onedrive.py` | `od.download(path)` |
| Upload file | `onedrive.py` | `od.upload(path, data)` |
| Move file | `onedrive.py` | `od.move(path, dest)` |
| Copy file | `onedrive.py` | `od.copy(path, dest, name)` |
| Rename file | `onedrive.py` | `od.rename(path, name)` |
| Delete file | `onedrive.py` | `od.delete(path)` |
| Create folder | `onedrive.py` | `od.create_folder(path, name)` |
| Read .docx | `onedrive.py` | `od.docx_read(path, mode)` |
| Create .docx | `onedrive.py` | `od.docx_create(path, ops)` |
| Edit .docx | `onedrive.py` | `od.docx_edit(path, ops)` |
| Read .pptx | `onedrive.py` | `od.pptx_read(path, mode)` |
| Create .pptx | `onedrive.py` | `od.pptx_create(path, ops)` |
| Edit .pptx | `onedrive.py` | `od.pptx_edit(path, ops)` |
| List Excel worksheets | `onedrive.py` | `od.xlsx_list_worksheets(path)` |
| Read Excel range | `onedrive.py` | `od.xlsx_read_range(path, ws, range)` |
| Write Excel range | `onedrive.py` | `od.xlsx_write_range(path, ws, range, vals)` |
| Add Excel worksheet | `onedrive.py` | `od.xlsx_add_worksheet(path, name)` |
| Add Excel table | `onedrive.py` | `od.xlsx_add_table(path, ws, range, headers)` |
| Add Excel formula | `onedrive.py` | `od.xlsx_add_formula(path, ws, cell, formula)` |
| Get Excel used range | `onedrive.py` | `od.xlsx_get_used_range(path, ws)` |
| List inbox | `mail.py` | `m.list_inbox()` |
| Search email | `mail.py` | `m.search(query)` |
| Read email | `mail.py` | `m.read(id)` |
| Read email body | `mail.py` | `m.read_body(id)` |
| Send email | `mail.py` | `m.send(to, subject, body)` |
| Reply to email | `mail.py` | `m.reply(id, body, reply_all)` |
| Forward email | `mail.py` | `m.forward(id, to, comment)` |
| Mark read/unread | `mail.py` | `m.mark_read(id)` / `m.mark_unread(id)` |
| Move email | `mail.py` | `m.move(id, folder)` |
| Delete email | `mail.py` | `m.delete(id)` |
| List calendars | `outlook_calendar.py` | `cal.list_calendars()` |
| List events | `outlook_calendar.py` | `cal.list_events(days_ahead)` |
| Get event | `outlook_calendar.py` | `cal.get_event(id)` |
| Create event | `outlook_calendar.py` | `cal.create(subject, start, end)` |
| Update event | `outlook_calendar.py` | `cal.update(id, ...)` |
| Delete event | `outlook_calendar.py` | `cal.delete(id)` |
| Accept invite | `outlook_calendar.py` | `cal.accept(id)` |
| Decline invite | `outlook_calendar.py` | `cal.decline(id)` |
| Tentatively accept | `outlook_calendar.py` | `cal.tentatively_accept(id)` |
## Resources

### scripts/

- `auth.py` — OAuth2 device-code flow + token refresh
- `graph_client.py` — Graph API HTTP client + Excel workbook/session
- `onedrive.py` — OneDrive file operations + Word/PowerPoint/Excel cloud bridge
- `word.py` — Offline .docx editor (python-docx)
- `powerpoint.py` — Offline .pptx editor (python-pptx)
- `mail.py` — Outlook mail operations
- `outlook_calendar.py` — Outlook calendar operations

---
> Source: [Astrojoin/openclaw-office](https://github.com/Astrojoin/openclaw-office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
