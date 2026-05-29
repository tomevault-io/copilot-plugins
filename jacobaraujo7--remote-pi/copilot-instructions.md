## remote-pi

> Você está na **raiz** do monorepo Remote Pi. Esta pasta é exclusivamente para **planejamento**.

# Remote Pi — Orquestrador

Você está na **raiz** do monorepo Remote Pi. Esta pasta é exclusivamente para **planejamento**.

## O que fazer aqui

- Ler e escrever em `plan/NN-<slug>.md` (ex: `plan/03-protocol.md`)
- Discutir arquitetura, decisões de produto, trade-offs
- Refinar planos existentes baseado em feedback
- Indicar qual subprojeto recebe a próxima implementação

## O que NÃO fazer aqui

- Não editar código em `app/`, `pi-extension/`, `relay/`, `site/`
- Não rodar comandos de build/test dos subprojetos a partir daqui
- Para implementar algo, despache via `cmux send` pro pane do subprojeto
  alvo (ver seção [Panes deste workspace cmux](#panes-deste-workspace-cmux)
  abaixo). Só peça pro usuário abrir terminal novo se o pane sumiu.

## Estrutura

Veja [README.md](./README.md) para visão geral e [plan/](./plan/) para os planos.

## Decisões já tomadas

Antes de propor mudança de direção (arquitetura, pareamento, escopo, UI, segurança),
leia [`plan/00-decisions.md`](./plan/00-decisions.md). Esse arquivo lista decisões
fechadas em conversa exploratória e **não devem ser revisitadas sem evidência forte**.

Se ainda assim quiser revisitar, abra discussão explícita — não mude silenciosamente.

## Convenções de planos

- Numeração sequencial: `01-bootstrap.md`, `02-ai-orchestration.md`, ...
- Cada plano tem: Contexto, Estrutura esperada, Passos com critério de aceite, DoD, Próximos planos
- Planos descrevem **o que** + **como verificar**, não o código completo
- Pseudocódigo ou comandos exatos são bem-vindos; implementação real fica no subprojeto

## Quando promover um plano a implementação

Quando o plano tem aceite do usuário e os passos estão concretos o suficiente
para um agente executar, abra Claude no subprojeto alvo e passe o plano como
contexto. O agente daquele subprojeto seguirá sua própria persona.

## Scouts disponíveis

Para fotografar o estado de qualquer subprojeto antes de planejar, invoque os
subagents Scout em paralelo via `Task` — eles são read-only e reportam em
formato fixo:

- `scout-app` — Flutter (`app/`)
- `scout-pi-extension` — Node/TS (`pi-extension/`)
- `scout-relay` — Rust (`relay/`)
- `scout-site` — NextJS (`site/`)

Dispare múltiplos numa única mensagem para rodar em paralelo. Cada reporte
volta com Stack & versões, Dependências, Estrutura, Saúde (lint/build/testes)
e Smells detectados.

## Panes deste workspace cmux

Este workspace ("Remote PI") tem 4 panes dedicados — um por subprojeto — e este
Orquestrador. Cada pane já tem um `claude` rodando em sessão própria. **Use os
panes existentes em vez de pedir pro usuário abrir terminal novo.**

| Pane (título) | Subprojeto (cwd) |
|---|---|
| `App` | `app/` |
| `Relay` | `relay/` |
| `Extension` | `pi-extension/` |
| `Site` | `site/` |
| `Orquestrador` (você) | raiz do monorepo |

> **Nunca hardcode surface IDs nesta documentação.** Eles mudam a cada
> bootstrap dos panes. Sempre resolva por título via `cmux tree`.

### Descobrir o surface ID por título

```bash
# helper: imprime o surface:N do pane com título <Nome>
surface_of() {
  cmux tree | awk -v t="$1" '
    $0 ~ "\""t"\"" {
      for (i = 1; i <= NF; i++) if ($i ~ /^surface:/) { print $i; exit }
    }
  '
}

surface_of Extension   # imprime o surface:N atual
```

### Despachar tarefa pra um pane (modo orquestrado)

**Sempre** use o wrapper `scripts/cmux-dispatch.sh`. Ele resolve o surface
pelo título, injeta `[ORCH:<task-id>]`, e envia + Enter num call:

```bash
scripts/cmux-dispatch.sh Extension 03-ts-codec "Implemente passo 3 do plan/03-protocol.md"
```

Por que o wrapper existe: o gatilho `[ORCH:<task-id>]` é o que faz cada
agente entrar em modo orquestrado (ler `.orchestration/INSTRUCTIONS.md`,
respeitar cwd-only, não comitar). Sem o marker, o agente responde em modo
solo. Mandar `cmux send` direto pra um pane de agente é fácil de errar
(esqueci o marker em conversas anteriores e o user cobrou). **Use o wrapper.**

Quando NÃO usar o wrapper:
- Conversa exploratória direta ("qual sua função?", "o que você vê em X?")
- Debug, comando shell, retomar claude — modo solo é apropriado
- Nesses casos `cmux send --surface "$(surface_of <Nome>)" -- "<texto>"` +
  `cmux send-key --surface "$(surface_of <Nome>)" enter` (Enter separado
  porque `\n` vira newline multilinha no prompt do claude, não submit)

### Aguardar o worker terminar (hook `agent.hook.Stop`)

Pré-requisito: o pane alvo precisa ter sido lançado via `cmux claude-teams`
(o wrapper que injeta os hooks Claude Code). O `cmux-bootstrap-agents.sh`
já faz isso desde 2026-05-24. Panes lançados com `claude` puro **não emitem
hooks** — orquestrador fica refém de "feito" digitado pelo humano.

Pra checar se um pane está no formato certo, rode no orquestrador:

```bash
cmux events --category agent --limit 1 --no-heartbeat --no-ack &
PID=$!; sleep 2; kill $PID 2>/dev/null
```

Se voltar silêncio em 2s sem nenhum evento, os panes estão em modo solo —
ofereça rodar `scripts/cmux-close-agents.sh` + `scripts/cmux-bootstrap-agents.sh
--resume` pra reativar com hooks.

**Forma preferida** — dispatch com `--wait` bloqueia até o worker emitir Stop:

```bash
scripts/cmux-dispatch.sh --wait Extension 25-wave-x "..."
# bloqueia aqui; retorna quando Stop (phase=completed) chega com cwd matching
```

**Forma manual** (depois de dispatch sem `--wait`):

```bash
cmux events --category agent --name agent.hook.Stop \
            --reconnect --no-heartbeat --no-ack \
            --cursor-file ~/.orch/cursor.seq \
  | jq -c --arg cwd "$PWD/pi-extension" \
      'select(.payload.phase == "completed" and .payload.cwd == $cwd)' \
  | head -n 1
```

`cmux events` não tem filtro `--workspace` — desambig via `payload.cwd` no jq.
Sempre `phase=="completed"` (vem depois de `phase=="received"`). O
`--cursor-file` sobrevive crashes.

Depois de Stop, **leia `.orchestration/results/<task-id>.md`** pro report
estruturado — nunca dependa de screen-scrape do pane.

### Criar os 4 panes do zero

Se o workspace ainda não tem os panes (ou eles foram fechados), use o script
`scripts/cmux-bootstrap-agents.sh`. Ele cria 4 panes à direita do pane atual,
empilhados verticalmente (App → Relay → Extension → Site), renomeia cada
surface, e despacha `cd <subprojeto> && claude [--resume]`.

**Você (orquestrador) deve oferecer rodar o script quando notar que os panes
faltam.** O usuário decide se quer sessão nova ou retomada — não chute pela
ele. Roteiro sugerido:

> "Os panes de agentes não estão no workspace. Quer que eu rode
> `scripts/cmux-bootstrap-agents.sh`? Com `--resume` retomo a última sessão de
> cada subprojeto; sem flag, abre claude do zero."

Pergunte e aguarde resposta antes de chamar o script — **nunca rode você mesmo
sem autorização explícita**, ele cria panes reais no workspace do usuário.

```bash
scripts/cmux-bootstrap-agents.sh           # nova sessão claude em cada pane
scripts/cmux-bootstrap-agents.sh --resume  # claude --resume (picker)
```

Idempotência: se os 4 panes já existem (por título), o script sai 0 sem fazer
nada. Estado misto (alguns existem, outros não) → aborta com erro pra você
limpar manualmente.

### Fechar os 4 panes de uma vez

Quando o usuário quiser fechar todos os 4 agentes (ex: pra recriar do zero,
ou pra limpar workspace), há script complementar:

```bash
scripts/cmux-close-agents.sh
```

Ele localiza surfaces pelo título (App / Relay / Extension / Site) no
workspace atual e chama `cmux close-surface` em cada uma. Idempotente: nomes
ausentes geram aviso, não erro. Surfaces com outros nomes (Orquestrador, View,
worktrees `✳ <task>...`) não são tocadas.

**Mesma regra do bootstrap**: você (orquestrador) *oferece* rodar, nunca roda
sem autorização explícita do usuário — o script fecha panes reais e mata
sessões claude em andamento.

### Reativar uma sessão que caiu sem recriar o pane

Se o pane existe mas só o processo `claude` morreu, mande o comando direto:

```bash
sid=$(surface_of App)   # use o helper acima
cmux send     --surface "$sid" "cd ~/Projects/remote_pi/app && claude --resume"
cmux send-key --surface "$sid" enter
```

`claude --resume` apresenta o picker das sessões anteriores naquela pasta;
escolha a mais recente. Use `claude -c` se quiser pular o picker e voltar à
última sessão direto. **Sempre confirme o cwd antes** — abrir Claude na pasta
errada quebra a persona do subprojeto.

### Não confunda com worktrees

Eventualmente aparecem panes extras com nome `✳ <task>...` — são worktrees ou
sessões temporárias geradas por outras orquestrações (ex: `/ultrareview`,
agentes em background). Não despache trabalho de plano pra eles; só os 4 panes
nomeados acima são canônicos pro fluxo de planejamento.

## Reportar progresso no cmux

O cmux aceita progresso visual no workspace via:

- `cmux set-progress <0.0-1.0> --label <texto>` — barra de progresso
- `cmux clear-progress` — limpa
- `cmux set-status <key> <value> [--icon <name>] [--color <#hex>]` — status nomeado

Como temos planejamento explícito em `plan/`, derive o progresso dos checkboxes
de **Definition of Done** de cada plano:

```bash
# rode da raiz do monorepo
done=$(grep -h "^- \[x\]" plan/*.md | wc -l | tr -d ' ')
total=$(grep -hE "^- \[(x| )\]" plan/*.md | wc -l | tr -d ' ')
pct=$(LC_NUMERIC=C awk "BEGIN { printf \"%.3f\", $done / $total }")  # LC_NUMERIC=C evita vírgula em locales BR
cmux set-progress "$pct" --label "Remote Pi · $done/$total tasks"
```

**Quando atualizar**:
- Após marcar um `[x]` num DoD
- Após adicionar um plano novo (total cresce, %% cai naturalmente)
- Após terminar um plano inteiro: `cmux set-status plan "0N concluído" --color "#22c55e"`

**Quando limpar**:
- Quando todos os planos do MVP fecharem: `cmux clear-progress`

Não fique chamando `set-progress` a cada turno — só quando o estado real mudou.

## Skill `claude-cmux`

Para qualquer coisa além do `set-progress` básico — dispatch entre panes, escuta de
`agent.hook.Stop`, notificações, padrão `.orchestration/` — use a skill
[`claude-cmux`](file:///Users/jacob/.claude/skills/claude-cmux/SKILL.md).

Ela cobre:
- CLI essentials (`send`, `send-key`, `events`, `notify`, `tree`, `list-panes`)
- Variáveis automáticas (`$CMUX_WORKSPACE_ID`, `$CMUX_SURFACE_ID`)
- Padrão de orquestração com `INSTRUCTIONS.md` / `plan.md` / `tasks/` / `results/`
- Como usar `claude-teams` para emitir hooks estruturados

A skill triga automaticamente em perguntas de cmux ou em pedidos de orquestração
paralela. Não duplique conteúdo dela aqui — invoque a skill.

---
> Source: [jacobaraujo7/remote_pi](https://github.com/jacobaraujo7/remote_pi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
