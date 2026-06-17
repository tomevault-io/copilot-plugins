## qq-nt-chat-export

> Export and search local QQ NT chat records from copied, already-decrypted QQ NT SQLite databases. Use when the user asks to extract authorized local QQ/QQ NT chat logs, list QQ groups or group members, or export messages by group, person, QQ number, UID, nickname/card, keyword, or "my messages".


# QQ NT Chat Export

## Scope

Use this skill only for QQ data the user owns or is authorized to inspect. Work on copied databases, never on QQ's live database files.

This public version expects already-decrypted SQLite databases:

- `nt_msg.decrypt.db`
- `group_info.decrypt.db` (optional but recommended for member name/QQ lookup)
- `profile_info.decrypt.db` (optional)

The reusable script is `scripts/qq_nt_export.py`. It decodes QQ NT protobuf message bodies with `blackboxprotobuf` and exports JSON, CSV, and TXT.

## Quick Start

List groups:

```bash
python scripts/qq_nt_export.py --db-dir /path/to/decrypted-db-dir list-groups --query "group name"
```

List members in a group:

```bash
python scripts/qq_nt_export.py --db-dir /path/to/decrypted-db-dir list-members --group 123456789 --query "nickname"
```

Export a full group:

```bash
python scripts/qq_nt_export.py --db-dir /path/to/decrypted-db-dir export \
  --group 123456789 \
  --output-dir ./qq_exports
```

Export one person in a group:

```bash
python scripts/qq_nt_export.py --db-dir /path/to/decrypted-db-dir export \
  --group 123456789 \
  --person "nickname or QQ number or u_xxx_UID" \
  --output-dir ./qq_exports
```

Export a person across all known groups:

```bash
python scripts/qq_nt_export.py --db-dir /path/to/decrypted-db-dir export \
  --person "nickname or QQ number or u_xxx_UID" \
  --all-groups \
  --output-dir ./qq_exports
```

## Workflow

1. Confirm the user is working with their own or authorized local QQ data.
2. Locate a working directory containing decrypted DBs.
3. Run `list-groups` or `list-members` to resolve ambiguous group/person names before exporting.
4. Run `export`, then verify:
   - JSON `metadata.message_count`
   - CSV data row count
   - TXT first and last few records
5. Report absolute output paths and counts to the user.

## Useful Options

- `--group`: group number or fuzzy group name. Can repeat.
- `--person`: QQ number, UID, group card, or nickname. Can repeat.
- `--sender-qq` / `--sender-uid`: exact sender filters.
- `--mine --self "marker"`: export messages matching the user's marker, QQ number, UID, nickname, or card.
- `--keyword`: decoded content keyword. Can repeat.
- `--since` / `--until`: local date/time, e.g. `2026-01-01` or `2026-01-01 12:30:00`.
- `--all-groups`: allow person/keyword exports across all group chats.
- `--include-raw`: include decoded raw protobuf fields in JSON.

## Known QQ NT Fields

Core group message columns in `nt_msg.decrypt.db`:

- `[40001]` message id
- `[40002]` random
- `[40003]` sequence
- `[40020]` sender UID
- `[40021]` peer UID
- `[40027]` group number for group messages
- `[40033]` sender QQ
- `[40050]` Unix timestamp
- `[40090]` group card
- `[40093]` nickname
- `[40800]` protobuf message body
- `[40900]` referenced/replied message body

Group members in `group_info.decrypt.db`:

- `group_member3.[60001]` group number
- `group_member3.[1000]` UID
- `group_member3.[1002]` QQ number
- `group_member3.[64003]` group card
- `group_member3.[20002]` nickname

## Dependency Notes

Install the protobuf decoder dependency:

```bash
python -m pip install blackboxprotobuf
```

---
> Source: [Decent898/qq-nt-chat-export](https://github.com/Decent898/qq-nt-chat-export) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
