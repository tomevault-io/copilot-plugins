## nuxt-enterprise-stack

> Quando receber este prompt @UltraThink, ative o modo de **análise profunda**:

# CLAUDE.md - Instruções Detalhadas para Claude

## 🧠 Modo de Operação: @UltraThink

Quando receber este prompt @UltraThink, ative o modo de **análise profunda**:

1. **PARE** e pense profundamente
2. **ANALISE** o problema por múltiplos ângulos
3. **CONSIDERE** edge cases e corner cases
4. **PLANEJE** a solução completa
5. **VERIFIQUE** todas as dependências
6. **EXECUTE** com precisão

### Processo @UltraThink:
```
1. ENTENDER o problema real
2. DECOMPOR em partes menores
3. IDENTIFICAR riscos e complexidade
4. PESQUISAR soluções disponíveis
5. DESIGN a arquitetura
6. IMPLEMENTAR gradualmente
7. TESTAR exaustivamente
8. DOCUMENTAR adequadamente
```

## 🏗️ Estrutura do Projeto Detalhada

### Apps (Aplicações)
```typescript
// apps/web - Frontend Nuxt 4
export interface WebConfig {
  ssr: true
  typescript: {
    strict: true
    typeCheck: true
  }
  modules: [
    '@nuxtjs/kit',
    '@nuxt/image',
    '@pinia/nuxt',
    '@vueuse/nuxt'
  ]
}

// apps/api - Backend Nitro
export interface ApiConfig {
  runtimeConfig: {
    database: DatabaseConfig
    auth: AuthConfig
    external: ExternalConfig
  }
  nitro: {
    experimental: {
      openAPI: true
    }
  }
}
```

### Packages (Bibliotecas)
```typescript
// packages/ui - Componentes Vue
// packages/types - Tipos TypeScript compartilhados
// packages/config - Configurações ESLint/Prettier
// packages/utils - Utilitários comuns
```

### Database (Drizzle ORM)
```typescript
// drizzle/schema/ - Definições de tabelas
// drizzle/migrations/ - Migrações versionadas
// drizzle/seed/ - Dados de teste
```

## 📦 Comandos pnpm Disponíveis

### Scripts Principais
```bash
# Desenvolvimento
pnpm dev                  # Inicia todos os apps em paralelo
pnpm dev:web             # Frontend (porta 3000)
pnpm dev:api             # Backend API (porta 3001)
pnpm dev:docs            # Documentação (porta 3002)

# Build e Produção
pnpm build               # Build all packages/apps
pnpm build:web          # Build frontend
pnpm build:api          # Build backend
pnpm preview            # Preview build

# Database (Drizzle)
pnpm db:generate        # Gerar migrations
pnpm db:migrate         # Aplicar migrations
pnpm db:push            # Push schema para DB
pnpm db:seed            # Popular database
pnpm db:studio          # Interface visual DB
pnpm db:reset           # Reset completo

# Testes
pnpm test               # Unit tests (Vitest)
pnpm test:e2e          # E2E tests (Playwright)
pnpm test:coverage     # Coverage report
pnpm test:ui           # Test UI

# Linting e Formatação
pnpm lint               # ESLint all
pnpm lint:fix           # ESLint fix
pnpm format             # Prettier format
pnpm typecheck          # TypeScript check

# Utilitários
pnpm clean              # Limpar dist/
pnpm clean:all          # Limpar tudo (node_modules, dist, .cache)
pnpm deps:update        # Update dependencies
```

### Scripts Monorepo
```bash
# Filtrar por package
pnpm --filter web build
pnpm --filter api dev
pnpm --filter ui test

# Parallel execution
pnpm dev --parallel
pnpm build --parallel

# Affected (Turbo)
pnpm build --filter=[origin/main]
```

## 🔄 Workflow de Desenvolvimento

### 1. Setup Inicial
```bash
# 1. Clone e install
git clone <repo>
pnpm install

# 2. Configure env
cp .env.example .env
# Editar .env com suas credenciais

# 3. Setup database
pnpm db:generate
pnpm db:migrate
pnpm db:seed

# 4. Start dev
pnpm dev
```

### 2. Desenvolvimento Diário
```bash
# Start dev (parallel)
pnpm dev

# Em outro terminal
pnpm test --watch        # Watch mode
pnpm lint --watch        # Watch lint

# Antes de commit
pnpm typecheck
pnpm test
pnpm lint:fix
pnpm format
```

### 3. Criando Novo Recurso
```bash
# 1. Criar branch
git checkout -b feature/novo-recurso

# 2. Gerar migration (se necessário)
pnpm db:generate --name add-novo-recurso

# 3. Desenvolver
pnpm dev

# 4. Testar
pnpm test
pnpm test:e2e

# 5. Commit
git add .
git commit -m "feat: add novo recurso"
```

### 4. Deploy
```bash
# Build all
pnpm build

# Preview
pnpm preview

# Deploy (exemplo Vercel)
vercel --prod
```

## 🎯 Mega-Prompts com Instruções

### Mega-Prompt 1: Novo Módulo CRUD
```markdown
"Crie um módulo CRUD completo para [ENTIDADE]:

INCLUIR:
✓ Schema Drizzle (drizzle/schema/)
✓ Migration (drizzle/migrations/)
✓ API endpoints (apps/api/server/api/)
✓ Tipos TypeScript (packages/types/)
✓ Página Vue (apps/web/pages/)
✓ Componentes UI (packages/ui/)
✓ Composables (apps/web/composables/)
✓ Validação Zod (apps/web/validation/)
✓ Testes unitários (test/)
✓ Testes E2E (e2e/)
✓ Documentação (docs/)

ARQUIVOS OBRIGATÓRIOS:
- Schema: drizzle/schema/entidade.ts
- Migration: drizzle/migrations/xxx_add_entidade.sql
- API: apps/api/server/api/entidade/index.get.ts
- API: apps/api/server/api/entidade/index.post.ts
- API: apps/api/server/api/entidade/[id].get.ts
- API: apps/api/server/api/entidade/[id].put.ts
- API: apps/api/server/api/entidade/[id].delete.ts
- Tipos: packages/types/entidade.ts
- Página: apps/web/pages/entidade/index.vue
- Form: apps/web/pages/entidade/form.vue
- Test: test/entidade.test.ts
- E2E: e2e/entidade.spec.ts
- Docs: docs/entidade.md"
```

### Mega-Prompt 2: Setup Projeto
```markdown
"Configure projeto completo Nuxt 4 enterprise:

CONFIGURAR:
✓ Nuxt 4 + TypeScript strict
✓ Drizzle ORM + PostgreSQL
✓ Lucia Auth
✓ Zod validation
✓ Vitest + Playwright
✓ ESLint + Prettier
✓ Pinia store
✓ Tailwind CSS
✓ Docker compose
✓ CI/CD (GitHub Actions)
✓ Documentação (nuxt docs)

ARQUIVOS:
- nuxt.config.ts (config completo)
- drizzle.config.ts
- package.json (scripts)
- tsconfig.json
- .eslintrc.js
- .prettierrc
- docker-compose.yml
- .github/workflows/ci.yml
- README.md completo
- ENV example"
```

### Mega-Prompt 3: Integração Externa
```markdown
"Integre [Odoo/n8n/Chatwoot] ao projeto:

IMPLEMENTAR:
✓ Client API (apps/api/clients/)
✓ Autenticação OAuth/API Key
✓ Models/Types (packages/types/)
✓ Configuração (.env)
✓ Error handling
✓ Rate limiting
✓ Cache strategy
✓ Webhooks (se aplicável)
✓ Testes integração
✓ Documentação API

SEGUIR:
- Padrão repository pattern
- Interface segregation
- Dependency injection
- Retry logic
- Logging estruturado"
```

### Mega-Prompt 4: Auth System
```markdown
"Implemente sistema auth completo Lucia:

INCLUIR:
✓ Config Lucia (apps/api/auth/)
✓ Providers (credentials, OAuth)
✓ Middleware auth (apps/web/middleware/)
✓ Composables auth (apps/web/composables/)
✓ Páginas login/register
✓ Protected routes
✓ Session management
✓ Password reset
✓ Email verification
✓ Testes auth
✓ Docs auth

SEGURANÇA:
- CSRF protection
- Rate limiting
- Password hashing (bcrypt)
- JWT secrets
- Secure cookies"
```

## ✅ Best Practices

### TypeScript
```typescript
// ✅ SEMPRE
interface User {
  id: string
  email: string
  createdAt: Date
}

// ❌ NUNCA
interface User {
  id: any
  email: any
}
```

### Vue 3
```typescript
// ✅ Composition API
<script setup lang="ts">
const props = defineProps<{
  user: User
}>()

const emit = defineEmits<{
  update: [user: User]
}>()

// Lógica aqui
</script>

// ❌ Options API
<script lang="ts">
export default {
  props: ['user'],
  methods: {}
}
</script>
```

### Drizzle ORM
```typescript
// ✅ Schema bem definido
import { pgTable, varchar, timestamp } from 'drizzle-orm/pg-core'

export const users = pgTable('users', {
  id: varchar('id').primaryKey(),
  email: varchar('email').notNull().unique(),
  createdAt: timestamp('created_at').defaultNow()
})

// ✅ Query tipada
const result = await db.select().from(users).where(eq(users.id, id))

// ❌ Query sem tipagem
const result = await db.query('SELECT * FROM users')
```

### API Endpoints
```typescript
// ✅ Nitro handler tipado
export default defineEventHandler(async (event) => {
  const id = getRouterParam(event, 'id')
  const user = await db.select().from(users).where(eq(users.id, id))

  if (!user) {
    throw createError({
      statusCode: 404,
      message: 'User not found'
    })
  }

  return user
})

// ✅ Com validação Zod
const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8)
})

const body = await readBody(event)
const data = schema.parse(body)
```

## 🚨 Troubleshooting Comum

### Problema: TypeScript errors
```bash
# Solução:
1. Verificar tsconfig.json
2. Regenerar tipos: pnpm typecheck
3. Verificar strict mode
4. Limpar cache: rm -rf node_modules/.cache
```

### Problema: Database connection
```bash
# Solução:
1. Verificar .env (DATABASE_URL)
2. Verificar se PostgreSQL está rodando
3. Testar conexão: pnpm db:studio
4. Regenerar migrations: pnpm db:generate
```

### Problema: Build fails
```bash
# Solução:
1. Verificar imports
2. pnpm lint:fix
3. pnpm typecheck
4. Verificar tree-shaking
```

### Problema: Tests failing
```bash
# Solução:
1. pnpm test --reporter=verbose
2. Verificar mocks
3. Atualizar snapshots
4. Verificar setup Vitest
```

### Problema: Performance
```bash
# Solução:
1. pnpm analyze (bundle)
2. Verificar lazy loading
3. Otimizar imagens (@nuxt/image)
4. Usar compression (gzip/brotli)
```

## 📚 Recursos Adicionais

### Documentação
- **Nuxt 4**: https://nuxt.com/docs
- **Vue 3**: https://vuejs.org/guide/
- **TypeScript**: https://www.typescriptlang.org/docs/
- **Drizzle**: https://orm.drizzle.team/docs
- **Lucia**: https://lucia-auth.com/
- **Zod**: https://zod.dev/
- **Vitest**: https://vitest.dev/
- **Playwright**: https://playwright.dev/

### Plugins Úteis
```bash
# Nuxt
@nuxt/image          # Image optimization
@pinia/nuxt          # State management
@vueuse/nuxt         # Composition utilities
@nuxtjs/tailwindcss  # Styling
@sidebase/nuxt-auth  # Auth helper

# Development
@typescript-eslint/eslint-plugin
eslint-plugin-vue
prettier
drizzle-kit
```

## 🎯 Comandos Específicos por Situação

### Debug
```bash
# Debug Node.js
node --inspect apps/api/.output/server/index.mjs

# Debug Vitest
pnpm test --inspect-brk

# Debug Playwright
pnpm test:e2e --debug
```

### Performance
```bash
# Lighthouse CI
pnpm lighthouse

# Bundle analyzer
pnpm analyze

# Memory usage
node --max-old-space-size=4096
```

### Maintenance
```bash
# Update dependencies
pnpm deps:update

# Audit security
pnpm audit

# Clean install
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

---

## 💡 Notas Importantes

1. **SEMPRE** leia AGENTS.md primeiro
2. **USE** @UltraThink para problemas complexos
3. **SIGA** os mega-prompts para features completas
4. **TESTE** antes de commit
5. **DOCUMENTE** APIs e componentes
6. **MANTENHA** TypeScript strict mode
7. **VALIDATE** inputs com Zod
8. **USE** Drizzle para DB operations

---

**Versão**: 3.0
**Última atualização**: 2025-12-08
**Compatibilidade**: Nuxt 4, TypeScript 5, Drizzle ORM

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neoand) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
