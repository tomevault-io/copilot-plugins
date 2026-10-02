## openpages

> This repo is an open-source **Spaces / Pages** workspace with **SuperCompress** as the core context layer.

# OpenPages — agent notes

This repo is an open-source **Spaces / Pages** workspace with **SuperCompress** as the core context layer.

## Non-negotiable

Every chat / agent / MCP `workspace_context` path:

```
Workspace → Retrieval → SuperCompress → Model
```

Implement via `buildCompressedContext` (`src/lib/supercompress/pipeline.ts`). Do not send raw Space dumps to models.

## Key docs

- [README.md](./README.md) — product + SuperCompress pitch
- [ARCHITECTURE.md](./ARCHITECTURE.md) — module map
- SuperCompress: https://www.supercompress.dev · https://docs.supercompress.dev

## Next.js note

This Next.js version may differ from training data. Check `node_modules/next/dist/docs/` when unsure.

---
> Source: [arjunkshah12345-hash/openpages](https://github.com/arjunkshah12345-hash/openpages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
