## midiapraise

> > Central de Mídia ADSA Reimberg

# AGENTS.md — ADSA Reimberg Mídias

> Central de Mídia ADSA Reimberg

Este arquivo orienta futuros agentes (humanos ou IA) que venham a trabalhar no projeto ADSA Reimberg Mídias. Leia-o integralmente antes de qualquer alteração.

## 1. Objetivo

O ADSA Reimberg Mídias é a fonte oficial dos materiais de mídia da ADSA Reimberg. Substitui o uso informal do WhatsApp como repositório de artes e fotos. A aplicação deve ser:

- Persistente (dados e arquivos não somem após reload).
- Segura por padrão (permissões verificadas no backend).
- Mobile-first e acessível.
- Sem funções decorativas (todo botão faz algo real).

## 2. Stack

- **Framework**: Next.js 16 (App Router) + TypeScript estrito
- **Estilo**: Tailwind CSS 4 + shadcn/ui (variante New York)
- **Banco**: Prisma + SQLite (em `db/custom.db`)
- **Auth**: sessão própria com cookie JWT (jose) + senha com bcryptjs
- **Validação**: zod (disponível), validações manuais nas APIs
- **Outros**: lucide-react (ícones), sonner (toasts), date-fns

> ⚠️ Quando o ambiente Z.AI oferecer Supabase nativo, recomenda-se migrar auth/storage para lá (ver spec seção 17.3).

## 3. Comandos

```bash
# Desenvolvimento (já iniciado pelo sandbox)
bun run dev                # next dev -p 3000

# Banco
bun run db:push            # aplica schema.prisma no SQLite
bun run db:generate        # regenera client Prisma
bun run db:migrate         # cria migration
bun run db:reset           # reset completo (CUIDADO)

# Seed inicial
bun run scripts/seed.ts    # cria admin, settings, categorias e evento demo

# Qualidade
bun run lint               # ESLint
bun run build              # build produção (não usar em dev)
```

## 4. Estrutura do projeto

```
/home/z/my-project/
├── prisma/schema.prisma        # modelos do banco
├── db/custom.db                # SQLite (NÃO versionar conteúdo)
├── scripts/
│   ├── seed.ts                 # seed admin + categorias + settings
│   ├── check_events.ts         # utilitário de inspeção
│   └── parse_spec.py           # parser do PDF spec original
├── docs/                       # documentação do projeto
│   ├── product.md
│   ├── architecture.md
│   ├── design-system.md
│   ├── permissions.md
│   ├── storage.md
│   ├── privacy.md
│   ├── testing.md
│   ├── operations.md
│   └── decisions/
├── src/
│   ├── app/
│   │   ├── page.tsx                     # home pública
│   │   ├── layout.tsx                   # AuthProvider + Header + Footer
│   │   ├── login/page.tsx
│   │   ├── eventos/page.tsx             # lista pública
│   │   ├── eventos/[slug]/page.tsx      # detalhe público
│   │   ├── historico/page.tsx
│   │   ├── admin/
│   │   │   ├── page.tsx                 # redirect → /admin/dashboard
│   │   │   ├── dashboard/page.tsx
│   │   │   ├── eventos/page.tsx         # lista admin
│   │   │   ├── eventos/novo/page.tsx
│   │   │   ├── eventos/[id]/page.tsx    # editar
│   │   │   └── configuracoes/page.tsx
│   │   └── api/
│   │       ├── auth/{login,logout,me}/route.ts
│   │       ├── events/route.ts          # GET (lista) + POST (cria)
│   │       ├── events/[id]/route.ts     # GET + PATCH + DELETE
│   │       ├── categories/route.ts
│   │       ├── church-settings/route.ts # GET + PATCH
│   │       ├── audit/route.ts
│   │       └── dashboard/route.ts
│   ├── components/
│   │   ├── ui/                          # shadcn/ui (não editar diretamente)
│   │   └── praisehub/                   # componentes próprios
│   │       ├── header.tsx
│   │       ├── footer.tsx
│   │       ├── auth-provider.tsx
│   │       ├── admin-guard.tsx
│   │       ├── event-card.tsx
│   │       ├── event-form.tsx
│   │       ├── status-badge.tsx
│   │       ├── copy-link-button.tsx
│   │       ├── share-whatsapp-button.tsx
│   │       ├── dashboard-client.tsx
│   │       ├── admin-event-list.tsx
│   │       ├── church-settings-form.tsx
│   │       └── ano-filtro.tsx
│   ├── lib/
│   │   ├── db.ts              # Prisma client singleton
│   │   ├── session.ts         # JWT + cookies + helpers de permissão
│   │   ├── praise.ts          # slug, datas, status, próximo culto
│   │   └── utils.ts           # cn (tailwind-merge)
│   └── types/index.ts         # DTOs compartilhados
└── CHANGELOG.md, README.md, PROJECT_MEMORY.md
```

## 5. Modelo de dados

Modelos ativos (Fase 1+2):

| Modelo | Função |
|--------|--------|
| `Profile` | usuários (admin/editor/aprovador/fotógrafo/equipe_midia/visitante) |
| `ChurchSettings` | configurações da igreja (singleton id="singleton") |
| `EventCategory` | 20 categorias iniciais (Culto de Celebração, Santa Ceia, etc.) |
| `Event` | cultos/eventos |
| `Approval` | decisões de aprovação (preparado para Fase 3) |
| `AuditLog` | trilha de auditoria |
| `ProductFeedback` | sugestões/erros dos usuários (pendente de UI) |

Modelos planejados para próximas fases: `MediaAsset`, `MediaVersion`, `Album`, `AlbumPhoto`, `ContributionLink`, `ContributionSubmission`, `Download`.

## 6. Perfis e permissões

Definidos em `src/lib/session.ts`:

| Perfil | Pode gerenciar eventos | Pode aprovar | Pode editar settings | Acessa admin |
|--------|------------------------|--------------|----------------------|--------------|
| administrador | ✅ | ✅ | ✅ | ✅ |
| editor | ✅ | ❌ | ❌ | ✅ |
| aprovador | ❌ | ✅ | ❌ | ✅ |
| fotografo | ❌ | ❌ | ❌ | ✅ |
| equipe_midia | ❌ | ❌ | ❌ | ✅ |
| visitante | ❌ | ❌ | ❌ | ❌ |

## 7. Padrões de código

- Todo componente client-side deve começar com `"use client";`.
- Server Components por padrão; converter para client apenas quando necessário (estado, efeitos, handlers).
- Nunca passar funções (onClick, onChange) como props de Server → Client Component. Criar um Client Component wrapper.
- Usar `db` de `@/lib/db` (singleton). Nunca instanciar PrismaClient diretamente.
- Validações de permissão SEMPRE no backend (`getCurrentUser()` + `canManageEvents()` etc.).
- Não usar `console.log` em produção (use Prisma log `['warn','error']`).
- Em APIs, retornar `ApiResult<T>` consistente.

## 8. Segurança

- Senhas com bcrypt (12 rounds).
- Sessão JWT em cookie httpOnly (secret em `PRAISEHUB_SESSION_SECRET`).
- Variáveis de ambiente em `.env` (NÃO versionar o `.env` real).
- Nenhuma chave/token hardcoded no código.
- Validação de permissão em TODAS as rotas administrativas.
- Eventos só aparecem publicamente se `status=publicado` E `visibilidade=publico`.

## 9. Fluxo de banco de dados

- Alterou `prisma/schema.prisma`? Rode `bun run db:push`.
- Para reset completo (apaga tudo): `bun run db:reset` e depois `bun run scripts/seed.ts`.
- Em produção, usar `bun run db:migrate` (cria migration versionada).

## 10. Como atualizar documentação

Após cada alteração relevante:

1. Atualize `CHANGELOG.md` com data, tipo (added/changed/fixed/removed) e descrição.
2. Atualize `PROJECT_MEMORY.md` se houver decisão arquitetural.
3. Atualize `docs/decisions/<data>-<topico>.md` se for uma decisão técnica importante.
4. Verifique se este `AGENTS.md` ainda é verdadeiro.
5. Rode `bun run lint` para garantir que o código está limpo.

## 11. Arquivos que NÃO podem ser apagados

- `prisma/schema.prisma`
- `db/custom.db` (dados reais)
- `scripts/seed.ts`
- `.env`
- `src/lib/db.ts`, `src/lib/session.ts`, `src/lib/praise.ts`
- `CHANGELOG.md`, `README.md`, `PROJECT_MEMORY.md`, `AGENTS.md`
- `docs/` (toda a pasta)

## 12. Próximas fases (roadmap)

Conforme spec master:

- **Fase 3 — Artes**: upload real, versionamento, aprovação, download em ZIP, link público/privado.
- **Fase 4 — Galeria**: álbuns, upload em massa, miniaturas, capa, visibilidade.
- **Fase 5 — Colaboração**: link de contribuição, moderação, privacidade.
- **Fase 6 — Administração avançada**: dashboard completo, auditoria visual, backup, feedback.
- **Fase 7 — Qualidade**: testes E2E com Playwright, testes de acessibilidade, performance, segurança.

Cada fase deve terminar com uma aplicação utilizável.

---
> Source: [thicco271/midiapraise](https://github.com/thicco271/midiapraise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
