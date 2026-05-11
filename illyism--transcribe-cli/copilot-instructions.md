## cli-patterns

> const input = args.find(arg => !arg.startsWith('--')) || args[0]


# CLI Patterns

## Argument Parsing

```typescript
const input = args.find(arg => !arg.startsWith('--')) || args[0]
const useRaw = args.includes('--raw')
```

Always extract the file/URL first (non-flag argument), then check for flags.

## Help Text

Must include:
- Usage line with placeholder
- All flags and options
- Multiple examples (local file, YouTube, with flags)
- Current optimizations status
- Supported formats
- Configuration instructions

## Error Messages

Pattern: Always include helpful links and copy-paste commands:

```typescript
throw new Error(
  'OPENAI_API_KEY not found.\n\n' +
  '🔑 Get your API key: https://platform.openai.com/api-keys\n\n' +
  'Then set it using ONE of these methods:\n\n' +
  '1️⃣  Environment variable...\n' +
  '2️⃣  Config file...\n\n' +
  '📚 Full setup guide: https://github.com/...'
)
```

## Config Resolution

Priority order:
1. Environment variable (`OPENAI_API_KEY`)
2. Config file (`~/.transcribe/config.json`)

Always try both before throwing error.

## Output Format

```typescript
console.log(`\n✅ SRT file saved to: ${result.srtPath}`)
console.log(`\nTranscription preview:`)
console.log('─'.repeat(60))
console.log(result.text.substring(0, 500) + '...')
console.log('─'.repeat(60))
console.log(`\nLanguage: ${result.language}`)
console.log(`Duration: ${result.duration.toFixed(2)}s`)
```

Use emoji icons for progress steps: 🎬 🎥 📊 ⚡ 🎙️ ✅ ⏱️ 🧹

---
> Source: [Illyism/transcribe-cli](https://github.com/Illyism/transcribe-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
