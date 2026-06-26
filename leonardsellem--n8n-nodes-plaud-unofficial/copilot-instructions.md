## n8n-nodes-plaud-unofficial

> > Agent instructions for this repo. Nearest-wins: this file governs all files in this directory tree.

# AGENTS.md — n8n-nodes-plaud-unofficial

> Agent instructions for this repo. Nearest-wins: this file governs all files in this directory tree.

## Project overview

n8n community node for the unofficial Plaud API. Reverse-engineered from `api-euc1.plaud.ai`.

## Directory map

```
├── credentials/          # n8n credential definitions
│   └── PlaudUnofficialApi.credentials.ts
├── nodes/PlaudUnofficial/
│   ├── PlaudUnofficial.node.ts    # Main node implementation
│   ├── PlaudUnofficial.node.json  # Node metadata (categories, docs)
│   ├── resources/                  # Operation implementations by resource
│   │   ├── file/      # File operations (getAll, get, download, getBatch)
│   │   ├── folder/    # Folder operations (getAll)
│   │   ├── ai/        # AI operations (getStatus, getNotes, getQuestions)
│   │   └── share/     # Share operations (getPrivate, getPublic)
│   └── *.svg          # Node icons (light/dark)
├── dist/               # Build output (gitignored)
└── .github/workflows/  # CI: lint + build
```

## Key commands

```bash
npm run build      # Compile TypeScript → dist/
npm run lint       # ESLint check
npm run lint:fix   # ESLint auto-fix
npm run dev        # Watch mode
```

## Quality gates

Before committing:
```bash
npm run lint && npm run build
```

CI runs these on every PR/push to main.

## API patterns

- Base URL: Configurable per-credential (EU: `api-euc1.plaud.ai`, US: `api.plaud.ai`, or custom)
- Auth: Bearer token in `Authorization` header
- All operations use `httpRequestWithAuthentication` helper
- Response data typically in `.data` or `.data_file_list` fields

## Adding new operations

1. Create operation file in appropriate `resources/<resource>/` folder
2. Export from `resources/<resource>/index.ts`
3. Add case in `PlaudUnofficial.node.ts` execute switch
4. Run lint + build

## Open questions

- Token expiration behavior unknown (no refresh mechanism implemented)
- API may have rate limits (not documented)

---
> Source: [leonardsellem/n8n-nodes-plaud-unofficial](https://github.com/leonardsellem/n8n-nodes-plaud-unofficial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
