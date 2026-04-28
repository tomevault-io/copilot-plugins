## litecoin-com-chat

> Payload CMS standards: collections, webhook hooks, TypeScript types, and i18n


# CMS Payload

## Identity

Payload CMS is the content management system for the Litecoin Knowledge Hub. Content authors create and manage articles here; changes are automatically synced to the backend RAG pipeline via HMAC-signed webhooks.

## Stack

| Layer | Technology |
|-------|-----------|
| CMS | Payload CMS 3.x |
| Framework | Next.js (embedded) |
| Database | MongoDB via `@payloadcms/db-mongodb` |
| Editor | Lexical (`@payloadcms/richtext-lexical`) |
| Locales | en, es, fr |
| Config | `src/payload.config.ts` |

## Collections

| Collection | Slug | Purpose |
|------------|------|---------|
| Users | `users` | Auth and roles: admin, publisher, contributor, verified-translator |
| Media | `media` | File uploads |
| Article | `articles` | Knowledge base articles (title, content, category, status, author) |
| Category | `categories` | Hierarchical article categories |
| SuggestedQuestions | `suggested-questions` | Chat UI suggested questions (question, order, isActive) |
| KnowledgeBase | `knowledge-base` | Internal KB content (title, slug, status, content) |

## Critical: Webhook Sync Hooks

Content collections (`articles`, `knowledge-base`) have `afterChange` and `afterDelete` hooks that fire HMAC-signed webhooks to:

```
POST ${BACKEND_URL}/api/v1/sync/payload
```

These webhooks trigger the backend to re-chunk and re-index content in the vector store. **Any change to a collection schema must be validated against `PayloadWebhookDoc` in `backend/data_models.py`** to ensure the webhook payload structure remains compatible.

## Type Generation

Payload auto-generates TypeScript types in `payload-types.ts`. After schema changes, regenerate types:

```bash
npx payload generate:types
```

## Seed Data

Category seed script: `src/scripts/seed-categories.ts`

## CORS

Configured for: `chat.lite.space`, `cms.lite.space`, `litecoin.com` (see `payload.config.ts`).

## Key Rules

- Schema changes require checking `backend/data_models.py:PayloadWebhookDoc` compatibility
- After collection changes, run `reindex_vectors.py` or `reindex_with_faq.py` from `scripts/`
- Localized fields must support all 3 locales (en, es, fr)
- User roles gate content operations — respect the access control in collection configs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IndigoNakamoto) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
