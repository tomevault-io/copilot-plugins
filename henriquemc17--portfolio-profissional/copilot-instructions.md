## portfolio-profissional

> > **"Architecture is the art of making the invisible visible through high-level engineering."**

# 💎 GEMINI - Mission Control & Agent Handbook
## Sistema Operacional Executivo: Henrique Monteiro Cardoso (Portfólio Profissional)

> **"Architecture is the art of making the invisible visible through high-level engineering."**  
> Este documento é a **Fonte Única de Verdade (SSOT)** para agentes de IA e desenvolvedores sênior operando neste workspace. Ele define as diretrizes de governança, arquitetura e excelência técnica que regem cada linha de código deste ecossistema.

---

## 🏛️ 1. Arquitetura de Referência (Clean Architecture + FSD)

O projeto não é apenas um site; é um **Ativo Digital Escalável**. A estrutura de pastas reflete a separação rigorosa de preocupações, protegendo o domínio de detalhes de infraestrutura.

- **`src/core/domain`**: O coração do sistema. Contém interfaces puras, tipos de entidade e business rules. Proibido importar dependências de frameworks ou I/O aqui.
- **`src/core/application`**: Orquestração. Use cases, hooks complexos de lógica e transformadores de dados.
- **`src/app`**: Camada de entrega (Next.js App Router). Focada em roteamento, SEO e Server Components.
- **`src/modules`**: Divisão por funcionalidade (Feature Sliced Design). Agrupamento lógico de componentes, hooks e serviços que mudam juntos.
- **`src/integrations`**: Adaptadores para serviços externos (Convex, Analytics, MCPs).

---

## 🚀 2. Padrões de Qualidade (Senior Fullstack Standards)

### A. Coding Philosophy
- **SRP (Single Responsibility)**: Se você não consegue descrever o que a função faz sem usar a palavra "e", divida-a.
- **Fail Fast & Guard Clauses**: Valide entradas no início da execução. Erros devem ser barulhentos em desenvolvimento e resilientes em produção.
- **Explicit over Implicit**: Sem "mágica". Tipagem forte em TypeScript é obrigatória. Use `zod` para validação de esquemas em runtime.

### B. UI/UX "Pro-Max" Engine
- **Aesthetics First**: Designs ultra-premium. Uso de `framer-motion` para micro-interações, `lucide-react` para iconografia e `tailwind-merge` para gestão de classes.
- **Core Web Vitals**: LCP < 2.5s, CLS < 0.1, INP reativo. Imagens devem usar `next/image` com priority labels adequadas.

### C. Performance & Security
- **Hard-code Zero**: Segredos e Configs em `.env.local` ou Vaults. Nunca comite tokens.
- **Infinite Scalability**: Consultas ao Convex devem ser otimizadas via Índices. Operações pesadas são delegadas a Actions ou Background Tasks.

---

## 🛠️ 3. Governança de Git & Workflow

Seguimos rigorosamente o **Conventional Commits**:
- `feat`: Novas funcionalidades.
- `fix`: Correção de bugs.
- `refactor`: Melhoria de código sem mudança funcional.
- `chore`: Atualização de dependências ou build scripts.
- `docs`: Documentação técnica.

**Branching Strategy**: `main` é sagrada. Todo trabalho é feito em `feature/*` ou `bugfix/*` e passa por revisão atômica.

---

## 🤖 4. Guia para o Agente (Antigravity/Gemini)

Ao atuar como Agente Fullstack neste repositório:

1.  **Analise antes de agir**: Antes de modificar qualquer arquivo, use `grep` para verificar o impacto nos domínios dependentes.
2.  **Socratic Gate**: Se uma tarefa for complexa ou ambígua, pare e faça perguntas exploratórias sobre trade-offs arquiteturais.
3.  **Higiene do Workspace**: Não polue a raiz. Use `/docs` para especificações técnicas e `/scripts` para automações.
4.  **Refactor on the fly**: Se encontrar código "morto" ou padrões obsoletos enquanto executa uma tarefa, limpe-os (respeitando o escopo do PR).
5.  **Documentação Contínua**: Se a lógica de um Use Case mudar, atualize a tipagem e o comentário de domínio imediatamente.

---

## 📊 5. Stack Histórica do Projeto

| Camada | Tecnologia | Propósito |
| :--- | :--- | :--- |
| **Framework** | Next.js 15+ (App Router) | SSR, SSG e Performance |
| **Linguagem** | TypeScript | Type Safety & Developer Experience |
| **Estilização** | TailwindCSS + Framer Motion | Design Ultra-Premium & Interatividade |
| **Backend/DB** | Convex | Reatividade Serverless & Real-time |
| **Automação** | Playwright + Lighthouse | Testes E2E e Auditoria de Qualidade |
| **Integrations** | MCP / Firecrawl | Data Intelligence e Automação de Fluxo |

---

> [!IMPORTANT]
> **Defesa da Arquitetura**: Qualquer proposta que quebre a barreira entre `domain` e `infrastructure` deve ser rejeitada sumariamente. A manutenção da pureza do core é o que permite a evolução de longo prazo sem dívida técnica.

---
*Gerado por **Antigravity** sob governança do Sistema Operacional Executivo @HenriqueMC17.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HenriqueMC17) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
