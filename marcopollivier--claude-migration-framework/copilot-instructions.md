## claude-migration-framework

> Framework genérico para executar migrações em massa em repositórios .NET usando Claude Code.

# Migration Framework — Claude Code

Framework genérico para executar migrações em massa em repositórios .NET usando Claude Code.

## Estrutura

```
.claude/
├── skills/
│   ├── migrate/SKILL.md              # /migrate — orquestrador genérico
│   ├── migrate-owner/SKILL.md        # /migrate-owner — verifica e corrige CODEOWNERS
│   └── migrate-mediatr/
│       ├── SKILL.md                  # /migrate-mediatr — receita MediatR
│       └── detect.sh                 # detecta se o repo precisa da migração
├── agents/
│   └── dotnet-migrator.md            # Executor isolado de migrações .NET
├── rules/
│   └── dotnet.md                     # Padrões de código .NET (scoped *.cs)
├── settings.json                     # Hooks de guardrail
tracking/
├── repos.txt                         # Repos pendentes (owner/repo, um por linha)
├── done.txt                          # Repos migrados com sucesso
├── skipped.txt                       # Repos pulados (não-.NET ou sem necessidade)
└── owner-report.txt                  # Repos com owner inesperado no CODEOWNERS
migrate.sh                            # Orquestrador shell para execução em massa
```

## Como funciona

1. **Skills** definem O QUE fazer — cada tipo de migração é uma skill separada
2. **Agents** definem QUEM faz — executor isolado com ferramentas e modelo específicos
3. **Hooks** garantem QUALIDADE — build e testes são validados automaticamente antes de commit/push
4. **Rules** definem PADRÕES — convenções de código aplicadas por escopo de arquivo

## Fluxo de migração (11 passos)

```
git pull → build → test → [criar testes] → [rodar testes] → aplicar migração → build → test → commit → push → PR
           ▲               hooks garantem                                       ▲
           └── hook valida build antes de prosseguir                            └── hook valida build+test antes de commit
```

## Fluxo de triagem (antes de gastar créditos)

```
repos.txt
   │
   ├─── clone/pull repo
   │
   ├─── [1º] check CODEOWNERS owner:
   │         @neon/cards          → PR de fix (branch: fix/update-codeowners-owner)
   │         @neon/cards-engagement → ok
   │         outro / ausente      → owner-report.txt
   │
   ├─── tem .csproj? ──── NÃO ──→ skipped.txt  (# not-dotnet)
   │
   ├─── detect.sh passa? ─ NÃO ──→ skipped.txt  (# not-needed:<tipo>)
   │
   └─── migrar via Claude ──→ sucesso → done.txt
                              falha   → permanece em repos.txt (retry)
```

## Migrações disponíveis

| Skill | Descrição | Branch / PR |
|---|---|---|
| `/migrate-owner` | Verifica CODEOWNERS: `@neon/cards` → `@neon/cards-engagement` | `fix/update-codeowners-owner` |
| `/migrate-mediatr` | Remove MediatR → dispatcher nativo | `migration/remove-mediatr` |

A verificação de owner sempre roda primeiro, independente do tipo de migração.

Para adicionar uma nova migração, crie:

- `.claude/skills/migrate-{nome}/SKILL.md` — receita da migração
- `.claude/skills/migrate-{nome}/detect.sh` — retorna 0 se precisa migrar, 1 se já está ok

## Uso interativo

```bash
# Migrar um projeto específico
/migrate-mediatr ./caminho/do/projeto

# Orquestrar migração genérica
/migrate mediatr ./caminho/do/projeto
```

## Uso em massa (headless)

```bash
# Processa todos os repos de tracking/repos.txt
./migrate.sh mediatr

# Processa apenas 5 repos por vez (economiza créditos)
./migrate.sh mediatr tracking/repos.txt --batch-size 5

# Com paralelismo limitado
./migrate.sh mediatr tracking/repos.txt --batch-size 5 --max-parallel 2
```

Repos são removidos de `tracking/repos.txt` automaticamente após cada resultado:

- **Migrado** → `tracking/done.txt`
- **Pulado** (não-.NET ou sem necessidade) → `tracking/skipped.txt`
- **Owner inesperado** → `tracking/owner-report.txt`
- **Falhou** → permanece em `tracking/repos.txt` para reprocessamento

## Convenções

- Branches: `migration/{nome-da-migracao}`
- Commits: `refactor: {descrição da migração}`
- PRs: título curto, body com summary + test plan
- Co-author: `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcopollivier) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
