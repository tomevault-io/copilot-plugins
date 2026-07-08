## database-conventions

> Convencoes de banco de dados e models


# Convencoes de Banco de Dados

## Multi-tenant por Marca
- Maioria das tabelas possui `brand_id` (unsignedBigInteger, foreign key para `brands`)
- Indices compostos incluem `brand_id` como primeiro campo
- Soft deletes em tabelas criticas (posts, articles, brands)

## Naming
- Tabelas: plural snake_case (`posts`, `custom_metrics`, `chat_conversations`)
- Foreign keys: singular_snake_case_id (`brand_id`, `user_id`, `post_id`)
- Pivot tables: singular ordenado alfabeticamente (`brand_user`, `post_platform`)
- Timestamps: sempre incluir `created_at` e `updated_at`

## Models
- Sempre definir `$fillable` ou `$guarded`
- Usar casts para tipos complexos (JSON, enum, datetime)
- Definir relationships com return type hints
- Usar Enums do PHP para status e tipos

## Tabelas Principais
- `users` - Usuarios do sistema
- `brands` - Marcas/empresas gerenciadas
- `brand_user` - Pivot usuario-marca com role
- `posts` - Posts de redes sociais
- `post_media` - Midias dos posts
- `post_schedules` - Agendamentos
- `chat_conversations` - Conversas de chat
- `chat_messages` - Mensagens do chat
- `articles` - Artigos de blog
- `link_pages` - Paginas de bio link
- `analytics_connections` - Conexoes com plataformas
- `metrics_snapshots` - Dados coletados de analytics
- `custom_metrics` - Metricas customizadas
- `custom_metric_entries` - Entradas de metricas manuais
- `ai_usage_logs` - Log de uso de IA

---
> Source: [julioandolfo/mkt-privus](https://github.com/julioandolfo/mkt-privus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
