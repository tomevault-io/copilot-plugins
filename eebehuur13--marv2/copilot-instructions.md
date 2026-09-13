## marv2

> This repo is optimized for multiline, multi-agent collaboration in the Cloudflare ecosystem. Keep these expectations in mind when adding automations or delegating tasks:

# Agents Guidance

This repo is optimized for multiline, multi-agent collaboration in the Cloudflare ecosystem. Keep these expectations in mind when adding automations or delegating tasks:

- **Runtime defaults**: The Worker expects Cloudflare bindings (`MARBLE_DB`, `MARBLE_FILES`, `MARBLE_VECTORS`) and Access headers on every request. Mock them explicitly in automated agents—see `tests/helpers/mock-env.ts` for a working example.
- **Embeddings & models**: Embedding dimensions must match the Vectorize index. When swapping OpenAI models, update both `wrangler.toml` and any automation that seeds data.
- **Chunking invariants**: Chunks must report inclusive line ranges. If an agent adjusts chunk size or overlap, keep the overlap ≥200 characters or update the README.
- **Frontend contract**: The SPA expects `/api` endpoints exactly as implemented (JSON shape noted in `frontend/src/lib/api.ts`). Agents modifying routes should sync that client file.
- **Tests as guardrails**: `tests/ingest.test.ts` and `tests/chat.test.ts` stub Cloudflare + OpenAI. Extend these when altering ingestion or retrieval flows to keep the mocks authoritative.
- **Data hygiene**: Folder IDs `public-root` and `private-root` are reserved; new automation should respect or extend the pattern `user:{base64url(id)}` for vectors and `users/{id}` for R2 keys.
- **Docs first**: Update `README.md` when changing deployment, auth, or infra expectations. Agents should fail the build if docs fall behind critical runtime behavior.

---
> Source: [eebehuur13/Marv2](https://github.com/eebehuur13/Marv2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
