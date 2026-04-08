---
name: box
description: Manage files and folders via Box API. Upload, download, and share content securely. Use when this capability is needed.
metadata:
  author: openclaw
---
# Box
Enterprise cloud storage.
## Environment
```bash
export BOX_ACCESS_TOKEN="xxxxxxxxxx"
```
## List Files in Folder
```bash
curl "https://api.box.com/2.0/folders/0/items" -H "Authorization: Bearer $BOX_ACCESS_TOKEN"
```
## Upload File
```bash
curl -X POST "https://upload.box.com/api/2.0/files/content" \
  -H "Authorization: Bearer $BOX_ACCESS_TOKEN" \
  -F "attributes={\"name\":\"file.txt\",\"parent\":{\"id\":\"0\"}}" \
  -F "file=@localfile.txt"
```
## Download File
```bash
curl "https://api.box.com/2.0/files/{fileId}/content" -H "Authorization: Bearer $BOX_ACCESS_TOKEN" -o file.txt
```
## Links
- Docs: https://developer.box.com

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openclaw)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/openclaw)
<!-- tomevault:4.0:skill_md:2026-04-08 -->
