## ovejitas-api

> domain: livestock/farm management

# CLAUDE.md

```yaml
project:
  name: ovejitas-api
  type: REST API
  domain: livestock/farm management
  runtime: node 18
  language: typescript 5 strict
  framework: fastify 5
  orm: sequelize 6
  database: postgresql 14
  entry: src/server.ts
  build_dir: build/
  src_dir: src/

scripts:
  dev: nodemon --config nodemon.json
  build: tsc
  start: node build/index.js
  migrate: npx sequelize-cli db:migrate
  seed: npx sequelize-cli db:seed:all
  undo_migrate: npx sequelize-cli db:migrate:undo

formatting:
  indent: tabs
  quotes: single
  semicolons: always
  trailing_commas: multiline
  brace_style: 1tbs
  object_curly_spacing: true
  eol: lf
  max_empty_lines: 1

naming:
  files: kebab-case
  variables: camelCase
  functions: camelCase
  classes: PascalCase
  interfaces: PascalCase
  types: PascalCase
  db_columns: snake_case
  db_tables: snake_case

typescript:
  target: ES2024
  module: NodeNext
  strict: true
  no_any: true, use unknown and narrow
  no_default_exports_except: resource index.ts and route files
  unused_vars: error, prefix _ to ignore args

architecture:
  pattern: plugin-based
  api_prefix: /api/v1
  autoload: src/resources/ via @fastify/autoload

plugin_rules:
  shared_plugins: wrap with fastify-plugin (breaks encapsulation, exposes decorators)
  resource_plugins: plain FastifyPluginAsync (stay encapsulated, autoloaded)
  shared_location: src/plugins/
  registration_order:
    - databasePlugin
    - fastifyCookie
    - customReplyPlugin
    - errorHandler
    - servicesPlugin (depends on database)
    - authenticationPlugin

resource_pattern:
  location: src/resources/{name}/
  files:
    - index.ts: plugin entry, registers routes with prefix
    - "{name}.routes.ts": route definitions with schema validation
    - "{name}.model.ts": sequelize model with TS interfaces
    - "{name}.schema.ts": typebox schemas for validation
    - "{name}.service.ts": business logic, extends BaseService
    - "{name}.serializer.ts": transforms models to API responses
  rules:
    - singular resource name
    - service is only layer touching database
    - routes never import models directly
    - serializer controls exposed fields
    - protected routes use preHandler fastify.authenticate

services:
  base_class: BaseService in src/services/base.service.ts
  registration: fastify.decorate in src/plugins/services.plugin.ts
  access: via fastify instance (fastify.animalService)
  utilities:
    - parseIncludes: relationship loading from query params
    - parseOrder: sorting from query params
    - parseFilters: filtering from query params
    - extractFilterParams: strips pagination params
    - filterTranslationsByLanguage: i18n support
    - combineWhereConditions: merges where clauses

schemas:
  library: "@sinclair/typebox"
  purpose: request params, body, querystring, response validation
  ajv_config:
    removeAdditional: false
    coerceTypes: true
    allErrors: true

responses:
  use: reply.success(), reply.error(), reply.successWithPagination()
  never: reply.send() directly
  format:
    status: success | error
    message: string
    data: optional
    meta: timestamp + optional pagination

auth:
  method: JWT
  storage: httpOnly cookies
  hook: preHandler fastify.authenticate
  password: bcryptjs
  id_obfuscation: hashids

database:
  migrations:
    language: javascript (not typescript)
    location: src/migrations/
    naming: "YYYYMMDDHHMMSS-description.js"
    must_have: up and down functions
  seeders:
    language: javascript
    location: src/seeders/
  pool:
    max: 5
    min: 0
    acquire: 30000
    idle: 10000
  errors: use handleSequelizeErrors utility
  no_raw_sql: prefer sequelize query interface

logging:
  use: fastify.log
  never: console.log

deployment:
  containerization: docker multi-stage
  compose: postgres + fastify
  production: render.com
  db_hosting: supabase
  env_file: .env (see .env.example)

cors:
  origins: ALLOWED_ORIGINS env var, comma separated
  credentials: true
  methods: [GET, POST, PUT, DELETE, OPTIONS]

docs:
  prd: docs/prd.md
  architecture: docs/architecture/
  coding_standards: docs/architecture/coding-standards/
  deployment: docs/production-deployment.md
  api_testing: little-sheep/ (Bruno collection)

test_framework: vitest

docker:
  rule: ALWAYS run commands inside Docker containers, never on host
  app_container: fastify-app
  build: docker compose exec app npm run build
  migrate: docker compose exec app npx sequelize-cli db:migrate
  dev: docker compose up (runs via entrypoint.sh)
  exec_pattern: docker compose exec app <command>

skill_evaluation:
  mandatory: true
  rule: |
    BEFORE writing ANY code, you MUST:
    1. List EVERY skill from the system-reminder's available skills section
    2. For each skill, write: [skill-name] → ACTIVATE / SKIP — [one-line reason]
    3. Call Skill(name) for every skill marked ACTIVATE
    4. Only THEN proceed to implementation
    If you skip this evaluation, your response is INCOMPLETE and WRONG.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aqwsxzzz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
