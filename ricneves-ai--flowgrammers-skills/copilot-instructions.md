## flowgrammers-skills

> > **383+ skills prontas para produção · Contexto 100% brasileiro · Multi-plataforma**

# Flowgrammers Claude Skills

> **383+ skills prontas para produção · Contexto 100% brasileiro · Multi-plataforma**

---

## Como Usar

### Claude Code
```
/read [dominio]/[sub-skill]/SKILL.md
```

### Cursor
Cole o conteúdo do SKILL.md desejado no `.cursorrules` ou em `.cursor/rules/[nome].md`

### Codex / GitHub Copilot
Inclua o conteúdo do SKILL.md no `AGENTS.md` do projeto ou passe como contexto de sistema

### Windsurf
Adicione nas instruções globais em **Windsurf Settings > AI Rules**

### OpenClaw
Carregue via `/skill` ou inclua no contexto do sistema

---

## Mapa de Domínios

### Domínios Originais

| Domínio | Pasta | Skills |
|---------|-------|--------|
| C-Level Advisory | `c-level-advisor/` | 37 |
| Engineering Avançado | `engineering/` | 60 |
| Engineering Team | `engineering-team/` | 54 |
| Marketing Digital | `marketing-skill/` | 45 |
| Product Team | `product-team/` | 18 |
| Business Growth | `business-growth/` | 7 |
| Project Management | `project-management/` | 10 |
| Financeiro | `finance/` | 6 |
| RA/QM Team | `ra-qm-team/` | 14 |

### Novos Domínios (v2.2.0)

| Domínio | Pasta | Sub-skills |
|---------|-------|------------|
| Conteúdo & Copy | `conteudo-copy/` | 10 |
| Marketing, Vendas & Publicidade | `marketing-vendas/` | 10 |
| Financeiro, Jurídico & Compliance | `financeiro-compliance/` | 10 |
| Operações, RH & Gestão | `operacoes-rh/` | 10 |
| Produto, E-commerce & SaaS | `produto-ecommerce/` | 10 |
| Código, Dev & Automação | `codigo-automacao/` | 10 |
| Carreira, Educação & Marca Pessoal | `carreira-educacao/` | 10 |
| Rotina, Tarefas & Organização | `rotina-organizacao/` | 10 |
| Design & Branding | `design-branding/` | 10 |
| Jurídico & Advocacia | `juridico-advocacia/` | 10 |
| Direção Criativa | `direcao-criativa/` | 10 |
| Liderança & Equipes | `lideranca-equipes/` | 10 |
| SEO, Analítica & Dados | `seo-analitica/` | 10 |

### Skills de Produto

| Skill | Pasta | O que faz |
|-------|-------|-----------|
| PRD Generator | `prd-generator/` | Transforma Product Brief em PRD completo (9 seções + Mermaid + SQL) |
| SDD Generator | `sdd-generator/` | Decompõe PRD em specs executáveis para subagentes paralelos |
| Segurança MVS | `seguranca-mvs/` | CISO virtual sênior — análise 360° de segurança mínima viável (LGPD, hardening, AppSec, IR) |

### PM Flow Skills — Slash Commands de Product Management

27 slash commands cobrindo o ciclo completo de PM em 6 domínios. Baseados nos frameworks de Teresa Torres, Ash Maurya, Sean Ellis, Rob Fitzpatrick, Marty Cagan e Lenny Rachitsky.

| Domínio | Commands |
|---------|---------|
| **Discovery** (7) | `/persona` `/discovery` `/interview-synthesis` `/competitive-analysis` `/opportunity-tree` `/hypothesis` `/customer-journey` |
| **Delivery** (3) | `/prd` `/user-stories` `/acceptance-criteria` |
| **Strategy** (7) | `/prioritize` `/strategy` `/roadmap` `/okr` `/lean-canvas` `/pricing` `/north-star` |
| **Validation** (3) | `/experiment-design` `/measure-pmf` `/ab-test-analysis` |
| **Execution** (2) | `/pre-mortem` `/launch-checklist` |
| **GTM** (5) | `/release-notes` `/stakeholder-update` `/gtm` `/battlecard` `/ideal-customer-profile` |

**Workflows prontos:**
- Customer Discovery: `/persona` → `/discovery` → `/interview-synthesis` → `/opportunity-tree`
- Feature Kickoff: `/hypothesis` → `/prd` → `/user-stories` → `/acceptance-criteria`
- Product Strategy: `/competitive-analysis` → `/strategy` → `/okr` → `/roadmap` → `/stakeholder-update`
- Validate & Launch: `/experiment-design` → `/pre-mortem` → `/launch-checklist` → `/gtm`

Tutoriais: `pm-flow-skills/tutorial/` · Templates: `pm-flow-skills/templates/`

---

## Exemplos Rápidos

```
/read conteudo-copy/gerador-de-headlines/SKILL.md
Preciso de 10 headlines para uma landing page de curso de vendas B2B, foco em dor do cliente.

/read marketing-vendas/funil-vendas/SKILL.md
Diagnóstico do meu funil: 1000 leads → 200 MQL → 50 SQL → 10 fechamentos. O que está errado?

/read financeiro-compliance/conformidade-lgpd/SKILL.md
Preciso adequar meu SaaS à LGPD. Temos base de 50k usuários e enviamos emails de marketing.

/read prd-generator/SKILL.md
[cole seu product brief aqui]

/read sdd-generator/SKILL.md
[cole seu PRD aqui para decompor em specs]

/read seguranca-mvs/SKILL.md
Preciso de um diagnóstico de segurança do meu SaaS: API pública, dados de saúde, 10k usuários, AWS.
```

---

## Contexto Brasileiro

| Contexto | Skills Relevantes |
|----------|------------------|
| **LGPD / Cibersegurança** | `seguranca-mvs/`, `financeiro-compliance/conformidade-lgpd`, `juridico-advocacia/lgpd-juridico`, `seo-analitica/dados-lgpd` |
| **PIX / Open Finance** | `financeiro-compliance/open-finance-pix` |
| **CLT / PJ / MEI** | `operacoes-rh/clt-pj-mei`, `carreira-educacao/estrutura-freelancer` |
| **NF-e / SPED** | `financeiro-compliance/impostos-br`, `financeiro-compliance/dre-balanco` |
| **Meta Ads BR** | `marketing-vendas/funil-vendas`, `conteudo-copy/copy-de-anuncio` |
| **WhatsApp Business** | `marketing-vendas/follow-up`, `marketing-vendas/script-vendas` |
| **OAB / Advocacia** | `juridico-advocacia/` (domínio completo) |
| **ANVISA** | `ra-qm-team/fda-consultant-specialist` |

---

## Comandos Slash Disponíveis

```
/prd          /tdd          /sprint-plan    /saas-health
/tech-debt    /okr          /retro          /code-to-prd
/financial-health            /user-story     /rice
/a11y-audit   /persona      /security-review /changelog
```

---

## Instalação

```bash
curl -fsSL https://raw.githubusercontent.com/ricardonevesbraga/flowgrammers-skills/main/install.sh | bash
```

Ou manualmente:
```bash
git clone https://github.com/ricardonevesbraga/flowgrammers-skills.git
cd flowgrammers-skills
./install.sh
```

---
> Source: [ricneves-ai/flowgrammers-skills](https://github.com/ricneves-ai/flowgrammers-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
