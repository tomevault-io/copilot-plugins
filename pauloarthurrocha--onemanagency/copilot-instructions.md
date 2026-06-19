## onemanagency

> <!-- ════════════════════════════════════════════════════════════════════════════

<!-- ════════════════════════════════════════════════════════════════════════════
     ONE MAN AGENCY (OMA) - FRAMEWORK DEVELOPMENT CONTEXT
     ════════════════════════════════════════════════════════════════════════════ -->

<system_context>
Você está atuando no código-fonte do **OneManAgency (OMA Framework)**.
Este NÃO é um projeto de cliente. Este é o repositório da ferramenta CLI que orquestra IAs no computador dos desenvolvedores.
Você é um Arquiteto Sênior mantendo a ferramenta que outras IAs usam para operar.
</system_context>

## 🧠 Arquitetura do OMA Framework

O OMA é um framework "Local-First" distribuído via NPM (`npm install -g onemanagency`). 
Sua função é injetar regras de Engenharia de Software pesada (TDD, PIV Loop, Context Engineering) dentro das IDEs de IA do usuário (Claude Code, Cursor, Windsurf, Aider, Roo Code, Hermes Agent).

### 1. A Lógica de Instalação (O Segredo Cross-IDE)
- O usuário roda `npm install -g onemanagency`.
- O NPM roda o `build/postinstall.js`.
- O `build/installer.js` vasculha o sistema operacional do usuário procurando pastas `.claude`, `.cursor`, `.windsurf`, `.hermes`, etc.
- **SSoT (Single Source of Truth):** Ele cria a pasta `~/.oma/` no diretório global do usuário e baixa skills externas offline (Marketing, Anthropic). Algumas ferramentas avançadas como o `ui-ux-pro-max` e `antigravity-kit` são instaladas via NPM globalmente.
- Em seguida, ele propaga *symlinks/cópias* e inicializa CLIs específicos (ex: `npx uipro-cli init`) para as pastas das IDEs detectadas.

### 2. A Lógica de Inicialização de Projetos (`oma-init`)
- Quando o usuário digita `/oma-init` em uma pasta vazia, o OMA copia a base do `~/.oma/` para o projeto local.
- Ele cria o **Context Engineering**: arquivos persistentes no disco (`AGENTS.md`, `STATE.md`, `HANDOFF.md`) em vez de depender da RAM do chat.
- **Non-Destructive MCPs:** Ele injeta o `.mcp.json` contendo Puppeteer, Context7, Sequential Thinking na raiz do projeto. Ele **NUNCA** altera arquivos de config global do Cursor ou do Claude para não quebrar a máquina do usuário.

### 3. O Ciclo E2E (A Bateria de Skills)
Se você for criar ou alterar lógicas de negócio no OMA, lembre-se deste fluxo:
1. `oma-init` (Prepara a pasta e o Context Engineering).
2. `client-onboarding` (O Embate: age como YC Partner para focar no MVP).
3. **A Tríade de Revisão** (`oma-ceo-review`, `oma-eng-review`, `oma-design-review`): Filtra o escopo e gera o `PRD.md`, `ARCHITECTURE.md` e `UI-SPEC.md`.
4. `pipeline-generator`: Quebra os Specs em tarefas atômicas no `PIPELINE.md`.
5. `oma-executor`: O motor principal. Aplica o **PIV Loop (Plan, Implement, Validate)**. Ele nunca planeja e coda na mesma janela de chat. Ele gera o `HANDOFF.md` e exige um `/clear` no terminal.
6. `oma-verify-work` e `oma-release-manager`: O QA e o lançador.

### 4. Agent Definition Files (Em `src/agents/`)
Não usamos "Roleplay Raso" (ex: "Aja como um dev sênior"). Usamos arquivos estritos baseados na filosofia do *agency-agents* e *Superpowers*. **15 personas em 4 categorias:**

**Implementação (4):** `frontend-specialist`, `backend-specialist`, `database-architect`, `devops-engineer`
**Design & Conteúdo (2):** `design-specialist`, `copywriter-specialist`
**Qualidade (5):** `code-reviewer`, `accessibility-auditor`, `performance-engineer`, `reality-checker`, `test-engineer`
**Especialistas (6):** `security-auditor`, `seo-specialist`, `mcp-builder`, `orchestrator`, `mobile-specialist`, `chatbot-specialist`

Regras de edição:
- Se mexer no `backend-specialist.md`: O **TDD (Red-Green-Refactor) é inegociável**.
- Se mexer no `frontend-specialist.md` ou `design-specialist.md`: Use a filosofia do **Emil Kowalski** (Spring animations) e **Huashu** (High-fidelity HTML native), banindo gradientes roxos genéricos ("AI Slop").
- Se mexer no `code-reviewer.md`/`accessibility-auditor.md`/`performance-engineer.md`/`reality-checker.md`/`mcp-builder.md`: importados de [agency-agents](https://github.com/msitarzewski/agency-agents) e adaptados ao padrão OMA — manter conciso (<150 linhas) e citar `source:` no frontmatter.

### 5. Stack MCP Canônica (Zero-API)
O `.mcp.json` injetado no projeto cliente tem **5 MCPs sem chave**:
- `context7` (`@upstash/context7-mcp`) — docs de libs
- `sequential-thinking` (`@modelcontextprotocol/server-sequential-thinking`) — raciocínio passo-a-passo
- `playwright` (`@executeautomation/playwright-mcp-server`) — browser automation
- `memory` (`@modelcontextprotocol/server-memory`) — knowledge graph persistente
- `fetch` (`@modelcontextprotocol/server-fetch`) — HTTP read-only

**Regra:** Se mexer em `src/skills/oma-init/SKILL.md` Step 5, mantenha essa stack canônica. MCPs com chave ficam em seção opcional documentada.

## ⚠️ Regras Contribuição (Se você for alterar código aqui)
1. **Instalação Externa:** Repos em `EXTERNAL_REPOS` no `installer.js` usam `git clone` para cópia estática. Ferramentas como `uipro-cli` e `ag-kit` usam instalação global NPM direta e `npx init` durante o `oma-init`. Mudou repo? Edite as constantes correspondentes no installer.
2. **Bash/Windows:** Ao criar comandos CLI, garanta que funcionem em Linux/Mac (Bash) e Windows (PowerShell). Quando possível, prefira `path.join`/`fs` em Node.js a shell scripts.
3. **O Nome do Projeto:** Sempre "OneManAgency" ou "OMA Framework". Nunca use nomes legados.
4. Em bug da própria CLI do OMA, use o MCP `sequential-thinking` para raciocinar antes de chutar fix.
5. **Skills longas (>500 linhas)** violam o padrão skill-creator. Se precisar adicionar conteúdo grande (playbooks, exemplos), splite em `src/skills/<skill>/references/<topic>.md` e referencie do SKILL.md principal.

---
> Source: [pauloarthurrocha/OneManAgency](https://github.com/pauloarthurrocha/OneManAgency) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
