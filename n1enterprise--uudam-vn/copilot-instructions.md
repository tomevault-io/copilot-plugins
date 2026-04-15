## uudam-vn

> File locations and naming conventions for backoffice modules


# Module File Structure

When creating a new module for entity `{Entity}` (e.g., `Product`, `Category`, `PostCategory`):

## Required Files and Paths

```
app/Models/{Entity}.php
app/Services/{Entity}Service.php
app/Repositories/{Entity}Repository.php
app/Repositories/Contracts/{Entity}RepositoryContract.php

app/Http/Controllers/Backoffice/{Entity}Controller.php
app/Http/Controllers/Backoffice/Api/{Entity}Controller.php

app/Http/Requests/Backoffice/Store{Entity}Request.php
app/Http/Requests/Backoffice/Update{Entity}Request.php
app/Contracts/Requests/Backoffice/Store{Entity}RequestContract.php
app/Contracts/Requests/Backoffice/Update{Entity}RequestContract.php

app/Http/Responses/Backoffice/List{Entity}Response.php
app/Http/Responses/Backoffice/Store{Entity}Response.php
app/Http/Responses/Backoffice/Update{Entity}Response.php
app/Contracts/Responses/Backoffice/List{Entity}ResponseContract.php
app/Contracts/Responses/Backoffice/Store{Entity}ResponseContract.php
app/Contracts/Responses/Backoffice/Update{Entity}ResponseContract.php

app/Http/Resources/Backoffice/{Entity}Resource.php

resources/views/backoffice/pages/{kebab-plural-entity}/index.blade.php
resources/views/backoffice/pages/{kebab-plural-entity}/create.blade.php
resources/views/backoffice/pages/{kebab-plural-entity}/edit.blade.php
```

## Optional Files (if module supports delete)

```
app/Http/Responses/Backoffice/Delete{Entity}Response.php
app/Contracts/Responses/Backoffice/Delete{Entity}ResponseContract.php
```

## Naming Conventions

- Model: PascalCase singular (e.g., `Category`, `PostCategory`)
- Route prefix: kebab-case plural (e.g., `categories`, `post-categories`)
- View directory: kebab-case plural (e.g., `categories`, `post-categories`)
- Route names: `bo.web.{kebab-plural}.{action}` and `bo.api.{kebab-plural}.{action}`
- Permission names: `{kebab-plural}.{action}` (e.g., `categories.index`, `categories.store`)

## Provider Registration Checklist

After creating module files, register bindings in:

1. `app/Providers/RepositoryServiceProvider.php` — add `{Entity}RepositoryContract => {Entity}Repository`
2. `app/Providers/BackofficeFormRequestServiceProvider.php` — add `Store{Entity}RequestContract => Store{Entity}Request` and `Update{Entity}RequestContract => Update{Entity}Request`
3. `app/Providers/BackofficeResponseServiceProvider.php` — add all response contract bindings
4. `routes/backoffice/web.php` — add web routes
5. `routes/backoffice/api.php` — add API routes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/N1Enterprise)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/N1Enterprise)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
