## shipyard

> Dashboard web local (localhost) para gerenciamento de projetos, tarefas, git, terminais e AI. Complementa o VS Code.

# Shipyard - Local Development Dashboard

Dashboard web local (localhost) para gerenciamento de projetos, tarefas, git, terminais e AI. Complementa o VS Code.

## Quick Start

```bash
pnpm dev          # client (5421) + server (5420)
shipyard.cmd      # Windows: batch file na raiz
./shipyard.sh     # Linux: server + browser
```

## Stack

| Camada | Tecnologia |
|--------|-----------|
| Frontend | React 18 + Vite + TypeScript + Tailwind CSS + shadcn/ui |
| Backend | Fastify 5 + TypeScript (via tsx) |
| Dados | Arquivos JSON em `data/` (sem banco de dados) |
| Monorepo | pnpm workspaces (client + server) |
| Desktop | Electron (opcional, `pnpm dist:win/mac/linux`) |

## Estrutura (resumo)

```
client/src/
  components/   # ui/ (shadcn), layout/, projects/, tasks/, git/, claude/,
                # terminals/, editor/, files/, sync/, mcp/, onboarding/
  hooks/        # useProjects, useTasks, useGit, useClaude, useTerminal,
                # useMilestones, useSheetSync, useFiles, useEditorTabs, useLogs, useMcp
  pages/        # Dashboard, Workspace, TasksPage, Settings, Help, LogsPage
  lib/          # api.ts (fetch wrapper), sync/ (provider pattern)

server/src/
  routes/       # projects, tasks, git, terminals, terminalWs, claude, mcp,
                # files, logs, sync, settings
  services/     # projectDiscovery, gitService, taskStore, terminalLauncher,
                # terminalService, claudeService, claudeContextBuilder,
                # claudeCliService, aiResolvePrompt, aiManagePrompt,
                # mcpServer, mcpAuth, logService, settingsStore, dataDir

data/           # Persistencia (auto-criado)
  projects.json, settings.json, claude.json, .claude-key,
  mcp-config.json, mcp-auth.json, server.log,
  tasks/{projectId}.json  # { milestones?: Milestone[], tasks: Task[] }

electron/       # main.ts, preload.ts (desktop wrapper)
```

## Modelos de Dados

```typescript
interface Task {
  id: string;               // nanoid(10)
  projectId: string;
  milestoneId?: string;     // undefined/'default' = milestone "General" (virtual)
  title: string;
  description: string;      // O QUE fazer (visao usuario/produto)
  prompt?: string;          // HOW/WHY tecnico (causas, arquivos, solucoes)
  priority: 'urgent' | 'high' | 'medium' | 'low';
  status: 'backlog' | 'todo' | 'in_progress' | 'done';
  order: number;
  createdAt: string;
  updatedAt: string;
  inboxAt?: string;         // quando entrou em backlog/todo
  inProgressAt?: string;    // quando moveu para in_progress
  doneAt?: string;          // quando foi concluida
}

interface Milestone {
  id: string;  projectId: string;  name: string;
  description?: string;  status: 'active' | 'closed';
  createdAt: string;  updatedAt: string;  order: number;
}

interface Project {
  id: string;  name: string;  path: string;  category: string;
  isGitRepo: boolean;  techStack: string[];  favorite: boolean;
  gitBranch?: string;  gitAhead?: number;  gitBehind?: number;
  gitStaged?: number;  gitUnstaged?: number;  gitUntracked?: number;
  externalLink?: string;  lastOpenedAt?: string;
  subRepos?: string[];   // Relative paths to sub-directories with their own .git
}
```

## Rotas da API (padrao: `/api/projects/:id/...`)

**Projetos**: GET /api/projects, PATCH /:id, POST scan/add/remove/refresh
**Milestones**: GET/POST /:id/milestones, PUT/DELETE /:id/milestones/:mid
**Tarefas**: GET /api/tasks/all, GET/POST /:id/tasks, PUT/DELETE /:id/tasks/:tid, POST /:id/tasks/reorder, POST /:id/tasks/replace
**Git**: GET /:id/git/status|diff|log|branches, POST /:id/git/stage|stage-all|unstage|commit|push|pull|discard|discard-all (all accept optional `subrepo` param for multi-repo projects)
**Files**: GET /:id/files/tree|content, PUT /:id/files/content, DELETE /:id/files, POST /:id/files/open-folder
**Terminais**: POST /api/terminals/launch|folder (nativos), GET/POST/DELETE /api/terminal/sessions (integrado), WS /ws/terminal/:id
**Claude AI**: GET /api/claude/status, POST config|config/test|chat(SSE)|analyze-task|summarize, DELETE config
**MCP**: POST /mcp (JSON-RPC), GET /mcp (SSE), OAuth em /register, /authorize, /token
**Sync**: POST /api/sync/proxy|test (proxy stateless para Google Apps Script)
**Logs**: GET /api/logs|logs/stats, DELETE /api/logs
**Sistema**: GET /api/settings, POST /api/browse

## Portas

- Backend: **5420** (dev), **5430** (Electron prod)
- Frontend Vite: **5421** (proxy /api → 5420)

## Convencoes e Regras

### Tarefas: description vs prompt
- **description**: O QUE fazer, visao usuario/produto, sem referencias a codigo
- **prompt**: Analise tecnica — causas, arquivos, solucoes, checklist de implementacao
- Para tarefas done: prompt contem resumo da implementacao

### Timestamps de Status (cascading)
Os timestamps sao cascading — etapas posteriores preenchem as anteriores automaticamente:
- `todo`/`backlog` → define `inboxAt`
- `in_progress` → define `inboxAt` + `inProgressAt`
- `done` → define `inboxAt` + `inProgressAt` + `doneAt`

**NUNCA remova timestamps existentes** ao editar tarefas. Ao mover entre colunas, adicione o novo sem apagar anteriores. Formato: ISO 8601 (`new Date().toISOString()`). Implementado via `buildCascadingTimestamps()` em taskStore.ts.

### Multi-repo Git (sub-repositorios)
- Projeto pode conter sub-pastas com `.git` proprio (ex: `client/` e `server/` dentro de `Sistema01/`)
- `detectSubRepos()` em projectDiscovery.ts escaneia 1 nivel de profundidade
- `subRepos` armazena caminhos relativos dos sub-repos encontrados
- Todas rotas git aceitam parametro opcional `subrepo` (query para GET, body para POST)
- GitPanel mostra tabs para selecionar sub-repo quando ha mais de um
- Query keys incluem `subrepo`: `['git-status', projectId, subrepo]`

### Cache e Invalidacao (react-query)
- refetchInterval: 15s (tasks), 30s (projects), 5s (git status)
- Mutations de tarefas DEVEM invalidar `['tasks', projectId]` E `['tasks', 'all']`

### Terminais (multiplataforma)
- **Windows**: `wt.exe` + `cmd.exe /k` (NAO bash — causa erro WSL)
- **Linux**: `gnome-terminal --title --working-directory` + `bash -c "cmd; exec bash"`
- **macOS**: `osascript` (Terminal.app)
- Terminal integrado: xterm.js + node-pty (optional dep) + WebSocket

### AI Task Management (Claude CLI)
- `claudeCliService.ts`: detecta e executa Claude CLI (`claude`) como subprocess
- `aiResolvePrompt.ts`: monta prompt para Claude resolver UMA tarefa (inclui contexto do projeto + task + MCP tools disponiveis)
- `aiManagePrompt.ts`: monta prompt para Claude gerenciar MULTIPLAS tarefas a partir de texto livre
- Auto-close: TerminalPanel detecta quando sessao AI termina e marca task como done se Claude nao o fez
- Prompt reforça que Claude DEVE atualizar status da task via MCP ao concluir

### Electron
- Server roda como child process via spawn (`ELECTRON_RUN_AS_NODE=1`)
- Data path: `SHIPYARD_DATA_DIR` env var → AppData em prod, ./data em dev
- Centralizado em `server/src/services/dataDir.ts`
- asar desabilitado, afterPack reinstala deps via npm (pnpm symlinks nao sobrevivem)

### Google Sheets Sync
- Config em **localStorage** apenas (`shipyard:sync:{projectId}`) — backend e proxy stateless
- URL validada: so permite `https://script.google.com/macros/s/...`
- Auto-push: debounce 2s apos mutations; auto-pull: polling 30s com merge bidirecional
- Anti-loop: `lastPushAt` guard impede pull nos 10s apos push

### Milestones
- "General" e virtual (nao armazenado) — tasks sem milestoneId pertencem a ele
- Deletar milestone move tasks para "General"
- Milestone ativo em localStorage: `shipyard:milestone:{projectId}`

## Regras para Contribuicao

1. **SEMPRE atualize este CLAUDE.md** quando mudar arquitetura, rotas, modelos ou convencoes
2. Dados persistem em JSON — nao introduza banco de dados sem discutir
3. Novos hooks seguem padrao de `useTasks.ts` (react-query + api.ts wrapper)
4. Componentes UI usam shadcn/ui (`npx shadcn@latest add <component>`)
5. Todas mutations de tarefas devem invalidar `['tasks', 'all']`
6. Terminal no Windows: SEMPRE cmd.exe, nunca bash
7. Novos services devem importar data path de `dataDir.ts`

---
> Source: [defremont/Shipyard](https://github.com/defremont/Shipyard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
