## finupv2

> - ✅ PRD: `/docs/features/PRD_MOBILE_EXPERIENCE.md` (135+ páginas)

# Cursor Rules - Mobile Experience Project

## 📱 Projeto Atual: Mobile Experience V1.0

### Status: PRD + Style Guide Completos
- ✅ PRD: `/docs/features/PRD_MOBILE_EXPERIENCE.md` (135+ páginas)
- ✅ Style Guide: `/docs/features/MOBILE_STYLE_GUIDE.md` (45+ páginas)
- ✅ Index: `/docs/features/MOBILE_INDEX.md` (resumo executivo)
- ⏭️ Próximo: TECH_SPEC (Especificação Técnica)

---

## 🎨 Design System - Imagem "Trackers"

### Paleta de Cores Extraída (SEMPRE usar estas cores)
```typescript
const trackerColors = {
  casa: { bg: '#DDD6FE', icon: '#6B21A8', progress: '#9F7AEA' },      // Roxo
  alimentacao: { bg: '#DBEAFE', icon: '#1E40AF', progress: '#60A5FA' }, // Azul
  compras: { bg: '#FCE7F3', icon: '#BE185D', progress: '#F472B6' },   // Rosa
  transporte: { bg: '#E7E5E4', icon: '#78716C', progress: '#A8A29E' }, // Bege
  contas: { bg: '#FEF3C7', icon: '#D97706', progress: '#FCD34D' },    // Amarelo
  lazer: { bg: '#D1FAE5', icon: '#047857', progress: '#6EE7B7' },     // Verde
};
```

### Dimensões Críticas (SEMPRE usar estas medidas)
```typescript
const dimensions = {
  iconCircle: '48px',         // Ícones circulares
  progressHeight: '6px',      // Barras de progresso
  cardRadius: '16px',         // Border radius cards
  touchTarget: '44px',        // Mínimo WCAG 2.5.5
  screenPadding: '20px',      // Padding lateral
  cardGap: '16px',            // Gap entre cards
};
```

### Tipografia (SEMPRE usar estas classes Tailwind)
```typescript
const typography = {
  pageTitle: 'text-[34px] font-bold leading-tight text-black',
  categoryName: 'text-[17px] font-semibold leading-snug text-black',
  frequency: 'text-[13px] font-normal leading-relaxed text-gray-400',
  amountPrimary: 'text-[17px] font-semibold leading-snug text-black',
  amountSecondary: 'text-[13px] font-normal leading-relaxed text-gray-400',
};
```

---

## 🚨 REGRAS CRÍTICAS (Do copilot-instructions.md)

### 1. Sincronização Git
**SEMPRE:** Local → Git → Servidor  
**NUNCA:** Editar código diretamente no servidor

### 2. Estrutura de Pastas

**REGRA CRÍTICA:** Organização de Features/Projetos

**Estrutura OBRIGATÓRIA para novos projetos:**
```
/docs/features/[nome-projeto]/
├── README.md                    # START HERE - navegação rápida
├── 01-PRD/                      # Product Requirements Document
│   ├── PRD.md                   # Especificação principal
│   ├── STYLE_GUIDE.md          # Design System (se aplicável)
│   ├── FACTIBILIDADE.md        # Análise técnica
│   ├── ANALISE_STAKEHOLDER.md  # Respostas a perguntas específicas
│   ├── SUMMARY.md              # Resumo executivo
│   ├── CHECKLIST.md            # Checklist de implementação
│   └── INDEX.md                # Índice e quick reference
├── 02-TECH_SPEC/               # Especificação Técnica
│   ├── TECH_SPEC.md            # Arquitetura, endpoints, componentes
│   ├── API_SPEC.md             # Especificação de APIs (se necessário)
│   └── DATABASE_CHANGES.md     # Mudanças no banco (se necessário)
└── 03-DEPLOY/                  # Deploy e Produção
    ├── DEPLOY_PLAN.md          # Plano de deploy
    ├── ROLLBACK_PLAN.md        # Plano de rollback
    └── MONITORING.md           # Métricas e monitoramento
```

**Exemplo (Mobile V1.0):**
```
/docs/features/mobile-v1/
├── README.md
├── 01-PRD/
│   ├── PRD.md
│   ├── STYLE_GUIDE.md
│   ├── FACTIBILIDADE.md
│   ├── ANALISE_STAKEHOLDER.md
│   ├── SUMMARY.md
│   ├── CHECKLIST.md
│   └── INDEX.md
├── 02-TECH_SPEC/ (a criar)
└── 03-DEPLOY/ (a criar)
```

**NUNCA:**
- ❌ Criar múltiplos arquivos soltos em `/docs/features/`
- ❌ Arquivos com prefixos como `MOBILE_*`, `PROJETO_*`, etc
- ❌ Duplicar informação em múltiplos arquivos
- ❌ Criar .md na raiz do projeto

**SEMPRE:**
- ✅ 1 pasta por projeto/feature
- ✅ Estrutura 01-PRD, 02-TECH_SPEC, 03-DEPLOY
- ✅ README.md como ponto de entrada
- ✅ Consolidar informações (máximo 7-8 arquivos por pasta PRD)

### 3. Banco de Dados — SEMPRE PostgreSQL via Docker
**SEMPRE usar PostgreSQL** (espelho da VM/produção). **NUNCA** SQLite em dev.
- **Local:** PostgreSQL via Docker — `docker compose up -d postgres` (antes de `quick_start.sh`)
- **Produção/VM:** PostgreSQL em `127.0.0.1:5432/finup_db`
- **URL:** `postgresql://finup_user:finup_password_dev_2026@localhost:5432/finup_db`
- **Como subir:** Docker Desktop aberto → `docker compose up -d postgres` (ou `quick_start.sh` faz isso)

### 4. Arquitetura
- **Backend:** DDD com domínios isolados (`app_dev/backend/app/domains/`)
- **Frontend:** Feature-based (`app_dev/frontend/src/features/`)
- **NUNCA:** Imports cruzados entre domínios ou features

### 5. Versionamento
**SEMPRE:** Usar `version_manager.py` para arquivos críticos  
**NUNCA:** Commitar versões `-dev` ou `-test`

### 6. Migrations
**SEMPRE:** Usar Alembic (`alembic revision --autogenerate`)  
**NUNCA:** Modificar schema SQL diretamente

### 7. Deploy
**FONTE ÚNICA DE VERDADE:** `deploy/` (pasta na raiz do projeto)

```
deploy/
├── README.md                         ← guia mestre
├── scripts/
│   ├── predeploy.sh                  ← RODAR ANTES DE TODO DEPLOY
│   ├── predeploy.py                  ← 22 testes auto + 13 UI (Playwright)
│   ├── deploy_docker_build_local.sh  ← DEPLOY PRINCIPAL
│   ├── deploy_docker_vm.sh           ← alternativo
│   └── validate_deploy.sh
├── validations/
│   └── ui_tests.py                   ← testes de UI headless
├── docs/                             ← GUIA_DEPLOY.md, TROUBLESHOOTING.md
└── history/                          ← checklists gerados + screenshots
```

**FLUXO:**
```bash
git add . && git commit -m "feat: ..." && git push origin <branch>
./deploy/scripts/predeploy.sh          # valida: Docker + API + DB + UI Playwright
./deploy/scripts/deploy_docker_build_local.sh  # só se 0 falhas
./deploy/scripts/validate_deploy.sh    # smoke test em prod
```

**NUNCA:** Deploy sem rodar `predeploy.sh` | Deploy com mudanças uncommitted  
**Doc:** `deploy/README.md` | `deploy/docs/GUIA_DEPLOY.md`

### 7.1 Deploy e branches (alterações grandes)
**SEMPRE:** Em alteração grande, criar branch **antes** de subir no servidor (ex.: `deploy/YYYY-MM-DD-nome` ou `feature/nome`). Subir e validar **nessa branch** no servidor. **Só depois** que der certo: merge na `main`. Rollback = descartar a branch.

### 7.2 Testes de UI (Playwright)
```bash
source app_dev/venv/bin/activate
python3 deploy/validations/ui_tests.py           # headless (13 testes)
python3 deploy/validations/ui_tests.py --headed  # com janela (debug)
```
Credenciais de `.env.local` (gitignored). Screenshots de falhas em `deploy/history/screenshots/`.

### 8. Backup
**SEMPRE:** Rodar `backup_daily.sh` antes de modificações críticas  
**NUNCA:** Modificar banco sem backup

### 9. Virtual Environment
**SEMPRE:** Usar `app_dev/venv` (não `.venv` da raiz)  
**Scripts:** `./scripts/deploy/quick_start.sh` gerencia automaticamente

### 9.1 Docker — PostgreSQL Obrigatório
**SEMPRE:** PostgreSQL via Docker antes de rodar o backend. Alinha dev local com VM.
- **Subir:** `docker compose up -d postgres` (ou `quick_start.sh` tenta subir automaticamente)
- **Verificar:** `pg_isready -h localhost -p 5432` ou `docker ps | grep postgres`
- **Parar:** `docker compose stop postgres`

### 10. Segurança
**SEMPRE:** Secrets em `.env` (NUNCA hardcoded)  
**NUNCA:** Commitar `.env`, senhas, tokens, ou API keys

---

## 📱 Mobile Development - Regras Específicas

### Componentes Mobile
**SEMPRE criar em:** `app_dev/frontend/src/components/mobile/`  
**Padrão de nome:** `tracker-card.tsx`, `tracker-header.tsx`

### Rotas Mobile
**SEMPRE usar:** `/mobile/*` (ex: `/mobile/dashboard`, `/mobile/budget`)  
**Redirecionamento:** Detectar `window.innerWidth < 768px` → redirecionar

### Bottom Navigation
**5 tabs fixas:**
1. Dashboard (📊)
2. Transações (💳)
3. Metas (🎯)
4. Upload (📤)
5. Profile (👤)

### Componentes Base (Criar primeiro)
1. `TrackerCard` - Card de categoria com progress bar
2. `TrackerHeader` - Header com botões voltar/menu
3. `ProgressBar` - Barra de progresso standalone
4. `CategoryIcon` - Ícone circular colorido
5. `BottomNavigation` - Navegação inferior fixa

### Ícones (Lucide React)
```typescript
import { Home, UtensilsCrossed, ShoppingBag, Fuel, FileText, ShoppingCart } from 'lucide-react';
```

### Acessibilidade (WCAG 2.1 AA)
- **Touch targets:** Mínimo 44x44px
- **Contraste:** ≥4.5:1 para texto normal
- **ARIA labels:** Todos os botões de ícones
- **Screen reader:** Testar com VoiceOver (iOS) e TalkBack (Android)

---

## 🎯 5 Telas Principais - Status

1. **Dashboard Mobile** - Reutilizar `MetricCards` existente ✅
2. **Transações Mobile** - Base existe, melhorar (bottom sheet, swipe) ⚠️
3. **Metas (Budget) Mobile** - Criar do zero baseado em "Trackers" ❌
4. **Profile Mobile** - Adaptar desktop para mobile ⚠️
5. **Upload Mobile** - Criar versão mobile otimizada ❌

---

## 📖 Documentação - Sempre Consultar

### PRD (Product Requirements Document)
**Path:** `/docs/features/PRD_MOBILE_EXPERIENCE.md`  
**Conteúdo:** Requisitos completos, layouts ASCII, user stories, roadmap

### Style Guide (Guia de Estilo)
**Path:** `/docs/features/MOBILE_STYLE_GUIDE.md`  
**Conteúdo:** Cores, dimensões, tipografia, componentes prontos, exemplos

### Index (Resumo Executivo)
**Path:** `/docs/features/MOBILE_INDEX.md`  
**Conteúdo:** Visão geral, checklist, próximos passos

### Copilot Instructions (Regras do Projeto)
**Path:** `/.github/copilot-instructions.md`  
**Conteúdo:** 2700+ linhas de regras críticas do projeto

---

## 🔄 Workflow de Desenvolvimento

### 1. Antes de Começar
```bash
# 1. Verificar sincronização
git status  # Deve estar limpo

# 2. Backup diário
./scripts/deploy/backup_daily.sh

# 3. Ligar servidores
./scripts/deploy/quick_start.sh

# Reiniciar (stop + start): ./scripts/deploy/quick_restart.sh
```

### 2. Durante Desenvolvimento
- **Arquivos críticos:** Usar `version_manager.py start/finish`
- **Imports:** Nunca cruzar domínios/features
- **Commits:** Usar prefixos (feat:, fix:, refactor:, docs:)

### 3. Antes de Commitar
```bash
# 1. Testar localmente
# 2. Verificar lints
# 3. Atualizar changelog (se necessário)
# 4. Commit com mensagem clara
git add .
git commit -m "feat(mobile): adiciona componente TrackerCard"
```

### 4. Antes de Deploy
```bash
# 1. Validar localmente
git status -uno          # Deve estar limpo
git push origin <branch> # Ex: feature/revisao-completa-do-app

# 2. Não deve existir middleware.ts (apenas proxy.ts)
#    Remover: app_dev/frontend/src/middleware.ts

# 3. Deploy (um comando)
./scripts/deploy/deploy.sh

# Se OOM no build: ./scripts/deploy/deploy_build_local.sh
```

**Scripts:** `deploy.sh` (padrão) | `deploy_build_local.sh` (alternativa OOM)  
**Doc:** `docs/deploy/DEPLOY_PROCESSO_CONSOLIDADO.md`

---

## 🚀 Próximos Passos (Ordem de Prioridade)

1. **TECH_SPEC** - Criar especificação técnica completa
2. **TrackerCard Component** - Implementar componente base
3. **TrackerHeader Component** - Implementar header mobile
4. **Bottom Navigation** - Implementar navegação inferior
5. **Metas Mobile** - Implementar tela de metas (baseada em "Trackers")

---

## 💡 Lembretes Importantes

### SEMPRE Fazer
- ✅ Consultar PRD e Style Guide antes de implementar
- ✅ Usar cores e dimensões da imagem "Trackers"
- ✅ Seguir estrutura de pastas do projeto
- ✅ Testar acessibilidade (contraste, touch targets)
- ✅ Backup antes de modificações críticas
- ✅ Usar domínios/features isolados

### NUNCA Fazer
- ❌ Editar código diretamente no servidor
- ❌ Criar arquivos na raiz (usar /docs/, /scripts/, /temp/)
- ❌ Hardcoded secrets/passwords
- ❌ Imports cruzados entre domínios
- ❌ Modificar schema SQL diretamente (usar Alembic)
- ❌ Deploy sem backup e validação

---

## 📞 Contatos e Recursos

### Documentação do Projeto
- **README:** `/Users/emangue/Documents/ProjetoVSCode/ProjetoFinancasV5/README.md`
- **Docs:** `/Users/emangue/Documents/ProjetoVSCode/ProjetoFinancasV5/docs/`

### Servidor de Produção
- **IP:** 148.230.78.91
- **SSH:** `ssh minha-vps-hostinger`
- **Path:** `/var/www/finup`

### Banco de Dados
- **Local:** SQLite em `app_dev/backend/database/financas_dev.db`
- **Produção:** PostgreSQL em `127.0.0.1:5432/finup_db`

---

**IMPORTANTE:** Este arquivo `.cursorrules` garante que todas as regras críticas sejam SEMPRE consideradas em futuras interações. SEMPRE consultar antes de implementar!

---

**Última atualização:** 31/01/2026  
**Status:** PRD + Style Guide completos ✅  
**Próximo:** TECH_SPEC 🚀

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emangue) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
