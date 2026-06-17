## copyman-cli

> Send and receive files via copyman session-based file transfer

## What I do
Send and receive files/notes using copyman CLI tool.

## When to use me
- User wants to push files to copyman ("push this file to copyman")
- User says they pushed files and wants me to receive them

## CLI Location
Binary: `copyman-cli` (must be in PATH)

## Session Management

Create temporary session:
```bash
copyman-cli create --temp
```

Create permanent session with password:
```bash
copyman-cli create --session-id <id> --password <password>
```

Login to existing session:
```bash
copyman-cli login --session-id <id> --password <password>
```

Logout:
```bash
copyman-cli logout
```

## Push Content

Push text:
```bash
copyman-cli push text "Your text content here"
```

Push file(s):
```bash
copyman-cli push file /path/to/file.png
copyman-cli push file file1.txt file2.pdf
```

## Receive Workflow

1. Login to session: `copyman-cli login --session-id <id> --password <password>`
2. List content: `copyman-cli list --json`
3. Get text: `copyman-cli get <content-id>`
4. Get file: `copyman-cli get <content-id> --output /path/to/save`

## List Content

JSON format (for parsing):
```bash
copyman-cli list --json
```

Returns array with: id, type (note|attachment), content, attachmentUrl, attachmentPath, timestamps.

## Delete Content

```bash
copyman-cli delete <content-id>
```

## Notes
- Sessions stored in `~/.config/copyman/config`
- One session active at a time
- Web interface at https://copyman.fr

---
> Source: [mathysIN/copyman-cli](https://github.com/mathysIN/copyman-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
