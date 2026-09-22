## data-safety

> Mandatory data safety when touching persistence, drafts, or entry content


# Data safety (strict)

## Required reading
- `.cursor/skills/wings-data-safety/SKILL.md`
- `.cursor/skills/wings-data-safety/incident-postmortem.md`

## Every save path MUST
1. Call `shouldBlockEmptySave(existingContent, nextMarkdown)` before `updateEntry`
2. Write `content` + `content_json` together
3. Have unit test in `editorContent.test.ts`

## Every load path MUST
1. Use `resolveInitialEditorContent` — never inline JSON preference
2. Use `shouldApplyDraft` before merging localStorage drafts

## Pending writes MUST
1. Replay only after `fetchEntries` (entries in state)
2. Use `shouldReplayPendingWrite`

## Before merge
- `bun run test -- src/lib/editorContent.test.ts`
- `bun run recover:entries` dry run
- Manual refresh test on page with existing content

---
> Source: [sabique-islam/wings](https://github.com/sabique-islam/wings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
