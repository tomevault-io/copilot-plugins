## markdown-provenance

> A Claude Code plugin that provides the `/markdown-provenance` slash command for uploading markdown files to Arweave permanent storage.

# Markdown Provenance

A Claude Code plugin that provides the `/markdown-provenance` slash command for uploading markdown files to Arweave permanent storage.

## Tech Stack

- TypeScript with ESM modules
- Node.js 18+
- ArDrive Turbo SDK for Arweave uploads
- Arbundles for data item signing
- Multiformats for IPFS CID generation

## Project Structure

```
.claude-plugin/
  plugin.json             # Plugin metadata for marketplace
commands/
  markdown-provenance.md  # Slash command definition
skills/
  markdown-provenance/
    SKILL.md              # Skill definition
scripts/
  upload-to-arweave.ts    # Main upload logic
plans/                    # Development planning docs
```

## Development Commands

```bash
npm install                    # Install dependencies
npm run upload <file.md>       # Upload a markdown file
```

## Environment Variables

Required:
- `MP_WALLET_PATH` - Path to Arweave wallet.json file

Optional:
- `MP_AUTHOR` - Author name added as metadata tag

## Key Implementation Details

- Files under 100KB upload free via Turbo
- Deduplication: checks local log and Arweave GraphQL before uploading
- Each upload generates an IPFS-compatible CID (CIDv1, SHA-256, raw codec)
- Transactions logged to `~/.markdown-provenance/transactions.jsonl`
- Tags applied: Content-Type, App-Name, App-Version, Type, IPFS-CID, Author (if set)

## Security Notes

- Never commit wallet.json to git
- The slash command only triggers on explicit `/markdown-provenance` invocation, never automatically

---
> Source: [rickmanelius/markdown-provenance](https://github.com/rickmanelius/markdown-provenance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
