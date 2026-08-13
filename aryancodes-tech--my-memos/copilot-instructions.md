## storage-invariants

> IndexedDB, codec, BlockDoc persistence, and settings tier rules


# Storage Invariants

## Two-tier model

| Tier | Technology | Contents |
|------|------------|----------|
| Heavy | IndexedDB `mymemos` | Pages (`doc_c` compressed), images |
| Light | chrome.storage / localStorage | theme, lastView, customThemes, collapsedDirs, productTour, demoWorkspaceSeeded |

## Block JSON only

- Persist Tiptap/ProseMirror `BlockDoc` - never HTML or markdown duplicates
- Field: `doc_c` (LZString compressed) in IndexedDB
- Decode failure → return `EMPTY_BLOCK_DOC`, `console.warn`, no UI throw

## Key files

| File | Role |
|------|------|
| `db.ts` | IDB schema, CRUD, `DB_VERSION` migrations |
| `codec.ts` | compress/decompress round-trip |
| `types.ts` | `Page`, `BlockDoc`, `BlockNode` |

## Schema changes

1. Increment `DB_VERSION` in `db.ts`
2. Add non-destructive `upgrade` handler
3. Add tests in `tests/extension/storage/db.test.ts` / `codec.test.ts`
4. Document migration in PR - user data must survive

## Page model essentials

```typescript
// parent_id empty string = root - check with len()
len(page.parent_id ?? "") === 0

// kind
"page" | "directory"

// section - workspace uses WORKSPACE_SECTION ("Pages")
```

## Images & attachments

- **Legacy images** may still use base64 `src` in block attrs.
- **New images and all audio** use OPFS via `extension/src/lib/attachments/`:
  - Paths like `images/img_*.png`, `audio/voice_*.webm` in block attrs
  - Files under OPFS root `mymemos-attachments/` (see `ATTACHMENT_OPFS_ROOT_DIR`)
- **Never** embed attachment binaries in `doc_c`.
- **Recording blocks** (`voiceNote` with `status: "recording"`) are ephemeral - `sanitizeBlockDocForPersistence()` removes them before IndexedDB write.
- **Page delete** must call `collectOrphanedAttachmentPaths()` and delete unreferenced OPFS files.
- **IndexedDB `images` store** and **`fs_handles` store** are legacy; new attachment flow uses OPFS only.

## Search

- **FlexSearch is never persisted**
- Rebuilt from pages in memory when palette opens
- Indexes `title`, body `text`, and `tags` - but there is **no UI** to edit tags today, so user-facing search is effectively title + body
- If adding indexed fields, update search builder only

## Schema vs product UI

- `exportWorkspace` / `importWorkspace` exist in `db.ts` and are tested - **not wired to any UI**
- `Page.archived` is filtered in selectors - **no archive action in UI**
- Do not document these as user features until exposed (see `AGENTS.md` §2.5)

## Platform note

IndexedDB is per-origin. Extension `chrome-extension://` ≠ `https://` demo - data is isolated by design.

---
> Source: [aryancodes-tech/my-memos](https://github.com/aryancodes-tech/my-memos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
