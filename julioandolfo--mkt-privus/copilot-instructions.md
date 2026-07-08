## laravel-conventions

> Convencoes Laravel para o projeto MKT Privus


# Convencoes Laravel

## Estrutura de Pastas
- `app/Models/` - Eloquent models
- `app/Services/` - Logica de negocio (AI, Social, Analytics, Blog, Content)
- `app/Jobs/` - Queue jobs (publicacao, sync analytics, geracao de conteudo)
- `app/Http/Controllers/` - Controllers Inertia (retornam Inertia::render)
- `app/Http/Requests/` - Form Requests para validacao
- `app/Actions/` - Acoes atomicas reutilizaveis
- `app/Enums/` - PHP Enums
- `app/DTOs/` - Data Transfer Objects

## Padroes de Codigo
- Usar PHP 8.3 features: typed properties, enums, match, named arguments, readonly
- Controllers devem ser enxutos - delegar logica para Services
- Sempre usar Form Requests para validacao
- Usar Eloquent scopes para queries reutilizaveis
- Usar Resources/DTOs para transformar dados para o frontend
- Jobs devem implementar `ShouldQueue` e ter `tries` e `backoff` definidos
- Usar `DB::transaction()` para operacoes criticas

## Multi-tenant por Marca
- A maioria das tabelas tem `brand_id` como foreign key
- Usar Global Scope ou middleware para filtrar por marca ativa
- Marca ativa e armazenada na sessao do usuario
- Verificar permissao do usuario na marca antes de qualquer operacao

## AI Gateway Pattern
```php
// Usar o AIGateway para abstrair modelos de IA
$gateway = app(AIGateway::class);
$response = $gateway->chat(
    model: AIModel::GPT4o,
    messages: $messages,
    brandContext: $brand->getAIContext()
);
```

## Naming Conventions
- Models: singular PascalCase (`Post`, `Brand`, `CustomMetric`)
- Controllers: plural PascalCase + Controller (`PostsController`, `BrandsController`)
- Services: PascalCase + Service (`OpenAIService`, `InstagramService`)
- Jobs: PascalCase + Job (`PublishPostJob`, `SyncAnalyticsJob`)
- Migrations: snake_case descritivo (`create_posts_table`, `add_brand_id_to_users`)
- Enums: PascalCase (`PostStatus`, `SocialPlatform`, `AIModel`)

---
> Source: [julioandolfo/mkt-privus](https://github.com/julioandolfo/mkt-privus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
