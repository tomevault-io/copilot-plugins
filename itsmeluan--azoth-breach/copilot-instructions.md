## azoth-breach

> Este arquivo é contexto persistente para trabalho técnico em `/game` e `/tools`.

# AZOTH: BREACH — Contexto para Sessões de Implementação

Este arquivo é contexto persistente para trabalho técnico em `/game` e `/tools`.
Ele não é fonte de cânone — apenas orienta sessões futuras a encontrar rapidamente
a documentação que já manda em qualquer decisão de implementação.

## Precedência documental

`/docs` é a fonte de verdade e tem precedência sobre qualquer instrução de sessão
ou suposição feita aqui. Em caso de divergência, os documentos vencem. Se um
conflito exigir alterar `/docs`, siga as regras de versionamento/changelog do
Documento 00.3 e sinalize explicitamente — não resolva em silêncio.

Ordem de leitura obrigatória antes de qualquer trabalho técnico:

1. `docs/00_constituicao/AZOTH_00.3_Estrutura_Mestre_da_Documentacao_v2.40.md` — hierarquia de precedência e regras de versionamento
2. `docs/06_tecnologia/README.md` — papel da série de tecnologia
3. `docs/06_tecnologia/AZOTH_06.0_Estrategia_Tecnica_e_Stack_Inicial_v0.2.md` — stack e princípios técnicos obrigatórios
4. `docs/06_tecnologia/AZOTH_06.1_Arquitetura_Inicial_da_Build_e_Estrutura_de_Pastas_v0.1.md` — arquitetura da build e estrutura de pastas
5. `docs/06_tecnologia/AZOTH_06.2_Pacote_de_Arranque_do_M1_v0.2.md` — tarefas técnicas `TK-M1-xxx` e critérios de aceite
6. `docs/05_produto/AZOTH_05.11_Backlog_Executavel_da_Vertical_Slice_v0.1.md` — backlog `BL-xxx`
7. `docs/05_produto/AZOTH_05.12_Milestones_de_Implementacao_da_Vertical_Slice_v0.2.md` — milestones `M1`–`M6`

## Stack (fechada — não reabrir sem gatilho documental)

- **Engine:** Godot 4.x, ramo estável (instalado: 4.7.stable)
- **Linguagem:** GDScript
- **Formato:** 2D, retrato vertical
- **Plataforma prioritária:** iOS (Android é secundário/futuro)
- **Desktop:** apenas debug/iteração local
- **Execução:** local-first até o fechamento de `M3` — sem backend, conta ou sync em nuvem
- **Dados:** JSON (conteúdo e save local), JSONL (telemetria mínima local)

Reabrir a stack só é permitido pelos gatilhos listados na seção 7 do Documento 06.0.

## Estrutura de pastas

```text
/game
  project.godot
  icon.svg
  /assets
    /audio /sprites /ui /vfx /temp
  /data
    /codex /ets /loadouts /operations /regions /rewards /state_templates
  /scenes
    /app /hub /loadout /operations /reports /research /shared
  /scripts
    /core
    /features
      /codex /hub /loadout /operations /reports /research /rewards
    /services
    /state
  /tests
/tools
  /content
  /exports
```

Cada pasta reflete uma camada da arquitetura definida em `06.1`:
aplicação (`core`), estado (`state`), conteúdo (`data`), features (`scenes` +
`scripts/features`), serviços (`services`). Placeholders devem ser substituíveis
sem reescrever a arquitetura base (princípio 2.5 do Documento 06.1).

## Configuração de projeto atual

- `run/main_scene`: `res://scenes/app/app_shell.tscn` — App Shell real
  (`TK-M1-004`). Faz roteamento básico via um container `ScreenHost`.
  Qualquer tela filha pode emitir `navigate_to(scene_path, context)` para
  pedir navegação com contexto (ex.: `operation_id`) — o App Shell conecta
  esse sinal automaticamente e chama `set_context(context)` na tela seguinte
  antes dela entrar na árvore, se o método existir (padrão reaproveitável
  por Loadout, entrada em operação etc.).
- Hub (`TK-M1-007`) embute um `OperationBoard` (`scenes/hub/operation_board.tscn`)
  que lista as operações de `data/operations/*.json` via `OperationLoader`
  (`scripts/services/operation_loader.gd`), diferenciando disponível/bloqueada
  a partir de `SliceState.is_operation_unlocked`. Selecionar uma operação
  disponível navega para o Briefing repassando `operation_id`. Acesso a Codex
  e Mesa de Pesquisa ainda não existem no Hub — entram em tarefas de
  milestones posteriores (`M3`).
- Briefing (`TK-M1-008`, `scenes/operations/briefing_screen.tscn`) recebe o
  `operation_id` via `set_context`, busca a operação com `OperationLoader` e
  exibe nome, sublocal, objetivo, risco e recompensa prevista (`rewards_guaranteed`
  formatado). O botão "Seguir" navega para `scenes/loadout/loadout_screen.tscn`
  repassando o mesmo `operation_id`.
- Dados de ET e loadout (`TK-M1-009`) em `data/ets/*.json` (4 ETs-base,
  `role` transcrito de `05.3` §"Biblioteca Base de ETs") e `data/loadouts/*.json`
  (3 presets — Contenção/Agressão/Investigação, `ets` e `description_short`
  de `05.9` §6.3-6.4). `upgrade_path_basic` por ET é decisão de implementação:
  associei cada ET à "melhoria preferencial" do build onde ela aparece
  primeiro listada como central (Selagem Parcial → Contenção, Decomposição
  Dirigida → Agressão, Análise de Vestígio → Investigação); Cristalização
  Controlada não tem build primária nos documentos (é secundária em
  Contenção e Agressão) — tratada como utilidade cross-build. Loaders em
  `scripts/services/et_loader.gd` e `loadout_loader.gd`, ambos delegando a
  `json_directory_loader.gd` (extraído de `operation_loader.gd` para evitar
  triplicar a mesma leitura de diretório).
- Loadout real (`TK-M1-010`, `scenes/loadout/loadout_screen.tscn`) exibe o
  preset ativo (nome, papel/foco tático, ETs com nome+papel) e permite
  trocar entre os três presets via botões dinâmicos — a troca atualiza
  `SliceState.active_loadout` imediatamente (sem estado local paralelo,
  aplicando o princípio de Estado Explícito de `06.1` §2.3). "Confirmar e
  entrar em campo" navega para `scenes/operations/operation_field_screen.tscn`
  repassando `operation_id` e `loadout_id`.
- Campo de operação simples (`scenes/operations/operation_field_screen.tscn`,
  origem em M2 `BL-012`/`BL-013`/`BL-005`) implementa um loop **mínimo por
  rodadas** contra uma barra única de Instabilidade — sem grid, inimigos ou
  catálogo de estados. Serve só **Primeira Fissura** (`max_rounds=4`) —
  Varredura de Estabilização migrou para o grid tático (ver abaixo), então
  esta tela hoje tem uma única operação cliente; mantida como está porque
  nada indica que valha a pena colapsá-la na tela de grid. Cada tentativa
  passa por `scripts/services/et_resolution.gd` (qualidade
  fraca/normal/precisa/extraordinária, conceito simplificado de `04.2 §11.2`,
  sem "falha" dura, recebe `SliceState.et_upgrade_level` e desloca a
  rolagem pra cima) e opcionalmente soma `dual_objective_et`/`evidence_target`
  quando a operação define esses campos. Encerra em sucesso (Instabilidade
  chega a 0) ou estabilização parcial (rodadas esgotadas — não é derrota;
  `04.3 §10.3` recomenda vitória parcial). Navega para
  `scenes/reports/report_screen.tscn`.
- **Grid tático** (`scenes/operations/operation_grid_screen.tscn` +
  `scripts/services/grid_combat_model.gd`) serve **Vestígio Discrepante** e
  **Varredura de Estabilização** (`combat_mode: "grid"` nos dados de ambas)
  — reabertura de escopo pedida explicitamente pelo usuário depois de
  testar a build e sentir que a mecânica simples não passava sensação de
  combate tático. Varredura entrou depois, no mesmo pedido: como é a única
  operação `repetivel`, é o único jeito de re-testar o grid depois que as
  operações de campanha ficam `COMPLETED` e travam (`BL-020`). Grid 6x6
  (`04.3 §6.1`), turno por time (`04.3 §8.2`: mover e/ou 1 ET, depois fase
  de inimigo). Cada ET tem efeito de campo distinto usando o campo `role`
  de `data/ets/*.json` (existia desde `TK-M1-009`, nunca tinha sido usado
  de fato): Selagem Parcial reduz Instabilidade; Cristalização Controlada
  cria obstáculo/cobertura (reduz dano em 50% pra quem está adjacente);
  Decomposição Dirigida dana inimigo adjacente; Análise de Vestígio soma
  vestígio **só quando a operação define `dual_objective_et`** (só
  Vestígio Discrepante) — em operações sem objetivo duplo (Varredura),
  Análise de Vestígio reduz Instabilidade como Selagem Parcial
  (`GridCombatModel.apply_analise_instabilidade`), senão a ET ficaria sem
  nenhum efeito mecânico ali; o bônus da Lente de Vestígio (evita "fraca")
  se aplica igual nos dois casos. Derrotar inimigos não é obrigatório — o
  objetivo continua sendo Instabilidade ≤ 0, inimigos são pressão tática
  (`04.3 §4`). Simplificações explícitas: sem fog of war/linha de efeito,
  sem estados de campo além do obstáculo, sem telegraphing visual
  dedicado. Dois bugs reais de IA corrigidos durante a validação
  original: inimigo travava permanentemente atrás de obstáculo (só
  tentava os 2 passos "diretos"); a correção ingênua causava oscilação
  entre 2 células. Resolvido com busca gulosa entre todas as adjacentes
  livres + memória da célula anterior pra não retroceder. **Economia de
  ação por rodada** (1 movimento e/ou 1 ET por agente por rodada, botões
  desabilitados após o uso e reabilitados só em "Terminar rodada"): o
  design original documentava essa regra mas não a implementava — dava
  pra spammar ETs numa rodada só, zerando Instabilidade sem nunca
  enfrentar a fase de inimigo, contradizendo o motivo de existir o grid.
  Achado numa passada de regressão pedida pelo usuário e corrigido antes
  da expansão para múltiplos agentes descrita abaixo.
- **Múltiplos agentes jogáveis e variedade de inimigos** (reescrita de
  `grid_combat_model.gd`/`operation_grid_screen.gd`, extensão de escopo
  pós-M6 autorizada diretamente pelo usuário — ver "Pós-M6: transição
  para escopo de MVP"): pesquisa em `/docs` antes de codar confirmou que
  **não existem personagens jogáveis nomeados** (protagonista
  explicitamente "fora de escopo" em `02.5`, linhas 137/1466/1493/1510)
  — `04.3` §15 só recomenda "equipe de 3" por função tática (`04.3` §14),
  sem nomes. Decisão de design pra entregar "múltiplos personagens" sem
  inventar nada: **cada uma das 3 ETs do loadout ativo agora é um agente
  jogável separado no grid**, travado à própria ET (Build de Investigação
  → 3 agentes: "Análise de Vestígio", "Selagem Parcial", "Decomposição
  Dirigida"). HP por agente: `60` (`AGENT_MAX_HP`, era `100` num agente
  único). Cada agente seleciona-se implicitamente ao clicar no próprio
  botão de ET (não precisa de passo de seleção separado); movimento
  precisa de seleção explícita via novo `AgentButtons` (mostra
  `"<nome>: <hp>/<max_hp>"`, clicar habilita "Mover" pra aquele agente).
  Economia de ação por rodada migrou de bools no screen pra campos
  (`move_used_this_round`/`et_used_this_round`) dentro da própria classe
  `Agent` no model. `retirada forçada` agora exige `all_agents_defeated()`
  (os 3, não só 1). Inimigo mira sempre o **agente vivo mais próximo por
  distância Manhattan** (`_nearest_agent`) — regra nova, não existia com
  agente único.
  - **3 tipos de inimigo agora** (era 1, `Criatura Deslocada` repetida
    2x), 1 de cada, grounded em `04.3` §16 (nenhum valor numérico definido
    lá — HP/dano são decisão de implementação): **Criatura Deslocada**
    (§16.1, sem mudança de comportamento, só passa a mirar o agente
    certo); **Operador Hostil** (§16.3 "sabota contenção" — se não está
    adjacente a nenhum agente e há obstáculo adjacente a ele, remove o
    obstáculo em vez de se mover, tradução mecânica direta da sabotagem);
    **Fenômeno de Campo** (§16.4 "ameaça sistêmica mesmo sem vida" —
    estacionário, nunca ataca, soma Instabilidade a cada rodada em que
    segue vivo, só some via Decomposição Dirigida). `enemy_phase()` mudou
    de `-> void` pra `-> int` (retorna a pressão de Instabilidade gerada),
    mantendo o padrão já existente de "o model nunca decrementa
    `_instability` diretamente, sempre retorna delta pro screen aplicar".
    Composição de inimigos é igual nas duas operações de grid — variar
    por operação e adicionar Entidade Anômala/Guardião de Ruína (também
    em `04.3` §16, não implementados ainda) é iteração futura, não parte
    desta passada.
  - Relatório (`report_screen.gd`) ganhou campo `agents_hp` (lista por
    agente) — mudança aditiva, não reescrita: somar o HP dos 3 agentes
    num único número escondia informação real (um agente inteiro morto
    ainda mostraria "120/180", parecendo só "perdi 1/3"); Primeira
    Fissura (sem grid) continua usando o formato antigo de soma única
    via checagem condicional `_context.has("agents_hp")`.
  - Cuidado de tipagem confirmado na validação: `_agent_for_et()` não tem
    tipo de retorno declarado (evita expor `GridCombatModel.Agent` como
    tipo externo, mesmo padrão já usado pra `Enemy`) — todo call-site
    precisa usar `var agent = ...` (sem `:=`), porque `:=` com função sem
    tipo de retorno declarado falha a inferência estática em GDScript 4 e
    quebra o parse. Erro real encontrado e corrigido durante a validação
    desta feature.
  - Plano revisado por um segundo agente focado em riscos técnicos de
    GDScript antes da implementação (arquivado em
    `~/.claude/plans/piped-orbiting-narwhal.md`) — identificou a regra de
    alvo mais próximo (faltava no design original) e o risco de
    `.filter()`/`.map()` perder tipagem `Array[T]`, já historicamente
    real neste projeto.
- Relatório e recompensa (`scenes/reports/report_screen.tscn`, origem em
  M2 `BL-014`/`BL-006`, generalizado em M3 `BL-016`/`BL-018`): mostra
  resultado/rodadas/ETs usadas, evidência coletada vs. meta quando
  aplicável, e a lista de `rewards_guaranteed` da operação (sempre
  concedida, independente do desfecho — `BL-014`). Chama
  `SliceState.complete_operation(operation_id, report_data)`, que marca a
  operação como concluída, aplica `unlocks_on_complete`, concede recursos
  (`RESOURCE_GRANT_PER_OPERATION = 20`), desbloqueia `codex_unlocks` e
  registra `report_data` em `reports_resolved` — tudo lido de volta
  dinamicamente pelo Hub/Codex/Mesa de Pesquisa, sem estado duplicado. Se
  a operação tiver `recontextualization_notice` (só Vestígio Discrepante),
  o texto aparece no relatório — parafraseia diretamente `05.9 §7.2`
  ("vestígios incompatíveis com a leitura institucional inicial"), nenhuma
  lore nova.
- Mesa de Pesquisa (`BL-015`, `scenes/research/research_screen.tscn`,
  acessível pelo Hub) mostra `SliceState.resources` e as 4 ETs com nível
  atual; melhorar custa `ET_UPGRADE_COST = 20`, nível máximo `3`, efeito
  imediato na próxima resolução tática (ver acima). Codex (`BL-017`, sem
  cena própria — `scenes/` não reserva pasta `codex` em `06.1`, só
  `scripts/features/codex`) abre como `PopupPanel` a partir de um botão no
  Hub, listando `data/codex/*.json` filtradas por
  `SliceState.codex_entries_unlocked` via `codex_loader.gd`. Uma entrada
  por operação, desbloqueada junto com a recompensa.
- Autoload `SliceState` (`scripts/state/slice_state.gd`, `TK-M1-005`, save
  real desde `BL-019`) carrega `user://slice_state_save.json` no boot se
  existir; senão cai para `data/state_templates/slice_state_initial.json`
  (antes do `_ready()` do App Shell) e expõe os campos mínimos de `06.1`
  §4.4 + `resources` (extensão própria, não documentada em `06.1`,
  necessária para "consumo simples de recurso" de `BL-015`). Estado
  inicial: `op_primeira_fissura` desbloqueada; `op_vestigio_discrepante` e
  a repetível bloqueadas; `active_loadout = "loadout_contencao_inicial"`
  (build de Contenção escolhida como padrão por ser a primeira listada em
  `05.9`/`05.11` e tematicamente alinhada ao objetivo de estabilização de
  `BL-005` — não há padrão explícito nos documentos, decisão de
  implementação registrada aqui). IDs de operação e loadout (`op_*`,
  `loadout_*_inicial`) batem com os dados reais criados em
  `TK-M1-006`/`TK-M1-009`. Toda mutação relevante (`complete_operation`,
  `upgrade_et`, `set_active_loadout`) salva em disco automaticamente — sem
  botão de save manual, sem sync em nuvem (viola `06.0`).
- Estado de operação formalizado em `scripts/services/operation_state.gd`
  (`BL-020`): `LOCKED` / `AVAILABLE` / `COMPLETED` / `REPEATABLE`,
  computado a partir de `SliceState` + `type` da operação. Operações de
  campanha ficam desabilitadas (`COMPLETED`) depois de concluídas — evita
  farm de recursos repetindo Primeira Fissura/Vestígio Discrepante agora
  que a persistência é real. Só `op_varredura_estabilizacao` (`type:
  "repetivel"`) permanece sempre selecionável, rotulada "Repetível", com
  `max_rounds=3` (mais curta que as outras duas) e chance de `15%`
  (`SliceState.CHASE_LOOT_CHANCE`) de conceder a Lente de Vestígio por
  execução (sem re-rolar depois de obtida). O item, uma vez obtido, evita
  o pior resultado ("fraca") especificamente ao usar Análise de Vestígio
  (`operation_field_screen.gd`, efeito distinto da melhoria de ET) — regra
  vem de `05.9 §9.2-9.3`.
- Telemetria (`BL-022`) via autoload `TelemetryLogger`
  (`scripts/services/telemetry_logger.gd`), gravando JSONL local em
  `user://telemetry.jsonl` — decisão de stack de `06.0` §5.3/§8, implementada
  pela primeira vez aqui. `AppShell.go_to_scene()` loga `screen_opened` para
  toda transição (um ponto de instrumentação cobre todas as telas). Demais
  eventos do escopo mínimo (`operation_started`, `et_used`,
  `operation_completed`, `operation_abandoned`, `loadout_selected`,
  `reward_granted`, `report_viewed`) logados nos pontos naturais do fluxo.
- Cancelar/Abandonar (`BL-021`): Briefing e Loadout ganharam botão
  "Cancelar" e o Campo ganhou "Abandonar operação" — antes só dava pra
  desistir fechando o app inteiro. Abandonar em campo não chama
  `SliceState.complete_operation()`: sem recompensa, a operação continua
  `AVAILABLE`. Essa lacuna de UI só ficou visível ao tentar instrumentar
  "operação abandonada" para `BL-022` — abandono não era uma ação possível
  até então.
- `renderer/rendering_method`: `mobile` (compatível com Metal/iOS).
- Viewport base: `1080x1920`, `window/handheld/orientation = portrait`,
  stretch `canvas_items` / `expand`.
- Janela de desktop (debug): override `810x1440` (1.5× do viewport-base
  em proporção 9:16) para ficar legível durante iteração local. O valor
  original (`540x960`) ficava pequeno demais para enxergar texto/UI
  confortavelmente; ajustado após feedback direto testando no editor.
  É só conveniência de debug local — não afeta o build real em iOS
  (viewport/stretch continuam os mesmos).
- Presets de exportação (Team ID, bundle identifier, certificados) **não**
  foram criados — dados pessoais/de conta não estão documentados em `/docs`
  e não devem ser inventados. Configurar em Editor → Export quando necessário.

## M1 a M6: vertical slice completa

`M1 — Primeira Build Navegável` (`TK-M1-001` a `TK-M1-012`, `06.2` §5):
projeto abre localmente, hub funciona, quadro de operações funciona,
briefing funciona, loadout funciona, entrada em operação é alcançável,
dados carregados por arquivos estruturados, fluxo repetível sem vazamento
de nó.

`M2 — Primeiro Loop Completo` (`BL-012`, `BL-013`, `BL-005`, `BL-014`,
`BL-006`, `05.12` §5): a Primeira Fissura já pode ser jogada de ponta a
ponta (loop mínimo por rodadas), gera relatório e recompensa reais, e o
retorno ao hub já muda o estado do jogo de verdade.

`M3 — Progressão e Segunda Operação` (`BL-015`, `BL-010`, `BL-016`,
`BL-017`, `BL-007`, `BL-018`, `05.12` §6): Mesa de Pesquisa funcional com
efeito imediato na resolução tática; Vestígio Discrepante jogável com
objetivo duplo (estabilizar + coletar vestígio) e recontextualização;
Codex acessível do Hub com uma entrada por operação; builds com
diferenciação mecânica real (só Contenção/Investigação conseguem
perseguir o objetivo secundário de Vestígio Discrepante, por incluírem
Análise de Vestígio). **Decisão de escopo mantida consistente com M2:**
"combate tático de verdade" de `BL-007` continua sem grid/inimigos —
mesma leitura minimalista aplicada à Primeira Fissura, generalizando o
loop por rodadas em vez de construir um sistema novo. Se isso for
insuficiente quando a campanha completa (fora da vertical slice) exigir
o grid de `04.3`, essa é uma reabertura de arquitetura consciente, não
um esquecimento.

`M4 — Repetição, Estado e Persistência` (`BL-019`, `BL-020`, `BL-008`,
`05.12` §7): estado da slice sobrevive a fechar/reabrir o app de verdade
(`user://slice_state_save.json`, testado com 4 lançamentos separados do
processo Godot, não só validação em memória dentro da mesma execução
como nos milestones anteriores); operações têm estado formal
(`LOCKED`/`AVAILABLE`/`COMPLETED`/`REPEATABLE`) — campanha concluída não
pode mais ser refarmada por recursos; Varredura de Estabilização jogável
de verdade, mais curta (3 rodadas) que as outras operações, com chance
controlada de Lente de Vestígio e o efeito mecânico do item (favorece
Análise de Vestígio, não substitui melhoria de ET, per `05.9 §9`).

`M5 — Slice Validável` (`BL-021`, `BL-022`, `BL-023`, `05.12` §8):
telemetria JSONL local cobrindo os 9 eventos do escopo mínimo (validado
inspecionando um `telemetry.jsonl` real de sessão de teste); botões de
Cancelar/Abandonar em Briefing, Loadout e Campo — lacuna de UI que só
ficou visível ao tentar instrumentar "operação abandonada" (não dava
pra desistir de nada antes, só fechando o app); validação integrada
final rodando o fluxo canônico completo (Op01 → Mesa de Pesquisa → Op02
→ repetível → hub) como checagem de regressão depois de todas as
mudanças de M2–M5, sem quebra.

`M6 — Polimento Interno da Slice` (`05.12` §9, sem blocos de backlog
formais — "organizar correções, refinamentos e estabilização sobre os
blocos já existentes", sem reabrir arquitetura): novo
`scripts/services/text_format.gd` elimina duplicação de formatação de
slug espalhada por 3 telas; quadro de operações e recompensas
reformatados para legibilidade (lista com marcador em vez de string
por vírgula); relatório mostra total de recursos após a concessão, não
só o delta, e corrige as ETs usadas para mostrar nome de exibição em
vez do ID cru (bug real de revisão: "Et Selagem Parcial" vazando o
prefixo `et_`); progresso de vestígios (objetivo duplo) agora
permanente na tela, não só no log; Hub ganha resumo
"Operações concluídas: X/Y"; `risk_level` de "medio" para "médio" nos
dados — ficou visível de forma proeminente só depois que
`TextFormat.format_slug` passou a expor o valor bruto diretamente ao
jogador.

Todos os milestones documentados da vertical slice (`M1`–`M6`, `05.12`)
estão concluídos. A partir daqui, qualquer trabalho novo precisa de
autorização documental explícita — `05.12` não define escopo além de
`M6` para a fatia vertical; expandir a campanha completa (grid tático
de `04.3`, mais ETs, mais operações, Codex/Mesa de Pesquisa avançados)
exigiria voltar a `/docs` para uma decisão de produto, não é uma
extensão natural do que já existe aqui.

### Pós-M6: grid tático (reabertura de escopo autorizada diretamente)

Depois de testar a build completa, o usuário sentiu que a mecânica de
campo por rodadas não passava sensação de combate tático mesmo
descontando a apresentação em texto puro, e pediu explicitamente o
grid mínimo de `04.3` (6x6, movimento, cobertura, inimigos) para
Vestígio Discrepante — ver seção "Grid tático" acima. Diferente do
resto deste documento, essa decisão **não** veio de `/docs`: foi
autorização direta do usuário em sessão, contornando a regra "exigiria
voltar a `/docs`" logo acima. Registrado aqui para deixar explícito que
essa reabertura teve autorização humana e não foi uma decisão de
implementação unilateral.

### Pós-M6: transição para escopo de MVP (autorização direta do usuário)

Depois do grid tático, o usuário pediu avançar até uma demo mais
próxima de produto (arte placeholder, mapa, múltiplos personagens,
mais inimigos, loot) antes de validar em iOS de verdade. Antes de
codar, li `05.1` (MVP e Recorte de Produção) e `05.3` (Vertical Slice
e Protótipos) inteiros e respondi ao usuário com duas correções
importantes antes de prosseguir:

1. **O MVP (`05.1` §3.1/§20) já cobre exatamente o que foi pedido** —
   região inicial, 1-3 agentes jogáveis, 5-8 inimigos, sistema de loot
   por categorias, e arte "suficiente para provar atmosfera" (não arte
   final) são escopo MVP documentado, não invenção nova.
2. **A ordem documentada é o inverso do que fizemos.** `05.3` §4.2/§4.3
   define Protótipos → MVP → Vertical Slice polida (a slice deveria
   ser uma amostra polida *de dentro* do MVP). Construímos uma vertical
   slice completa (M1-M6) sem MVP prévio. Isso não se desfaz — o
   enquadramento correto daqui pra frente é **expandir o M1-M6 na
   direção do recorte completo do MVP** (`05.1` §21, por prioridade),
   não "fechar a slice e abrir uma fase separada".

O usuário confirmou esse enquadramento e pediu para seguir, priorizando
mapa/exploração primeiro.

### Mapa de Aletheia (primeira peça do recorte MVP)

Implementa a camada "Mapa de Aletheia" de `04.4` §4/§5.2 — explicitamente
**não** é mundo aberto navegável, é um mapa estratégico de regiões como
nós com estado, respondendo "onde vale a pena agir agora". Antes de
implementar, uma pesquisa dedicada em `/docs` confirmou que os 3
sublocais de operação já existentes (Estação Ferrínea de Ensaios
Estruturais, Câmara das Vigas Cantantes, Túneis de Calor Baixo) **já
eram canônicos** — `docs/02_lore/AZOTH_02.9` §4.3 "Ferrária e as Forjas
Subterrâneas" os lista literalmente como "Lugares iniciais associados";
`TK-M1-006` não inventou nada, só usou nomes já existentes no lore. O
próprio título do Hub (`"Cidade-Forja de Ferrária"`, definido desde
`TK-M1-004`) já colocava o jogo em Ferrária sem que isso tivesse sido
percebido como decisão regional até agora.

- `data/regions/regiao_ferraria.json` (novo tipo de conteúdo,
  `region_loader.gd` delegando a `json_directory_loader.gd`, mesmo
  padrão dos demais loaders): `name`, `subtitle` e `description` citam
  quase literalmente `02.9` §4.3; `academia` é "Academia Ferrínea de
  Matéria e Estruturas" (`02.4` §5.1). As 3 operações existentes
  ganharam `"region_id": "regiao_ferraria"`.
- **Eixos de estado regional** (`04.4` §7: Estabilidade, Pressão de
  Breach, Confiança Civil, Influência Acadêmica, Integridade de
  Infraestrutura, Pendências de Codex) existem em `/docs` só como
  conceito qualitativo — nenhum valor inicial, faixa numérica ou
  exemplo de UI é definido em lugar nenhum. Decisão de implementação:
  só implementei os 2 eixos com gancho mecânico real e honesto no
  estado já existente — **Estabilidade** (derivada de operações de
  campanha concluídas/total na região) e **Pendências de Codex**
  (derivada de `codex_unlocks` das operações da região vs
  `SliceState.codex_entries_unlocked`) — ambos recalculados
  dinamicamente a cada vez que o Mapa é aberto, sem estado duplicado.
  Os outros 4 eixos (Confiança Civil, Influência Acadêmica, Integridade
  de Infraestrutura) ficaram de fora deliberadamente: não têm gancho
  mecânico existente em `SliceState`, e inventar números/narrativa só
  pra preenchê-los seria inventar sistema novo sem grounding — mais
  fácil e honesto omitir do que fabricar.
- **7 outras regiões canônicas de Aletheia** (`02.9` cap. 4: Aurentum,
  Lumenport, Viridessa, Noctíria, Calx-Rama, Umbrafenda, Arquipélago de
  Nacre) aparecem bloqueadas no Mapa como "sinais de que o mundo é
  maior" (`05.1` §7.1, §19 lista "região no mapa ainda interditada"
  como placeholder aceitável). Nomes reais do lore, não inventados;
  hardcoded no script (não viraram tipo de conteúdo em `/data`) pela
  mesma lógica já aplicada aos inimigos fixos do grid — é lista de
  flavor sem conteúdo jogável por trás ainda, criar um formato de dados
  só pra isso seria abstração prematura.
- **Navegação original (superada, ver seção "Hub fundido com o Mapa" mais
  abaixo)**: nesta primeira versão, `hub_screen.tscn` ganhou um botão
  "Mapa de Aletheia" que levava a `region_map_screen.tscn` (tela
  separada, um clique a mais a partir do Hub). Entrar em Ferrária leva a
  `region_detail_screen.tscn`, que embute o mesmo `operation_board.tscn`
  reaproveitado sem alteração (ainda lista todas as operações sem
  filtrar por região — com só 1 região existindo isso é correto por
  coincidência; filtrar por `region_id` fica pra quando existir uma 2ª
  região de verdade, não antes). Isso implementou pela primeira vez a
  estrutura de 4 camadas de `04.4` §4 (Hub → Mapa de Aletheia →
  Estrutura de Campo → Confronto Tático).
- **Divergência sinalizada, não resolvida**: `05.1` §15.2 pede "1
  mentor ou superior acadêmico" no MVP. A mentora canônica da campanha
  principal é **Lys Aurel** (`02.5` linha 584-591, confirmada em
  `03.2` linha 328), mas sua Academia é a Auricária/Aurentum, não a
  Ferrínea/Ferrária. O único personagem nomeado de Ferrária ligado à
  Academia Ferrínea é **Cael Ferron**, categorizado como "Rival Técnico
  / Especialista de Campo" — não mentor. Não implementei nenhum NPC
  ainda (fora de escopo desta etapa, que era só o mapa), mas essa
  inconsistência regional precisa de decisão humana explícita antes de
  qualquer trabalho de NPC/mentor em Ferrária — não é algo que os
  documentos resolvem sozinhos.

### Catalisadores (primeira peça do sistema de loot/crafting)

Implementa o slot de "catalisador" da estrutura mínima de buildcraft do
MVP (`05.1` §10.3: "ET base + 1 modificador + 1 catalisador ou runa") —
só esse slot, não "modificador"/runa, pra não empacotar dois sistemas
novos de uma vez (fica pra iteração futura). Pesquisa em `/docs` antes de
codar confirmou que não existe lista canônica de catalisadores nomeados
em `01_alquimia` nem em `05.9` — só o conceito e a taxonomia formal
(`04.6` §7.2: 7 subtipos funcionais — Estabilidade, Intensidade,
Precisão, Propagação, Extração, Instáveis, Impossíveis). Os 4
catalisadores implementados (`data/catalysts/*.json`) usam esses
subtipos já canônicos como nome em vez de inventar nomenclatura nova:
Estabilidade (comum, bônus +1), Intensidade (incomum, +2), Precisão
(raro, +3), Instável (experimental/chase-tier, +4 — espelha "Instáveis"
de `04.6`: mais poder, sem garantia).

A pesquisa também achou um gancho pendurado nos dados desde `TK-M1-006`:
a string `"chance_de_runa_ou_catalisador"` já existia em
`rewards_guaranteed` de `op_varredura_estabilizacao.json` (vinda de
`05.9`), mas nunca teve efeito mecânico — só aparecia como texto
formatado genérico. Agora `SliceState.complete_operation()` rola contra
essa string (`CATALYST_DROP_CHANCE = 15`, mesma faixa do chase loot) e,
se acertar, sorteia 1 catalisador por peso de raridade
(`CATALYST_RARITY_WEIGHTS`, sem valor definido em `/docs` — decisão de
implementação) e adiciona ao inventário (`SliceState.catalysts_owned`).

Efeito mecânico: `bonus_level` do catalisador equipado soma direto ao
`upgrade_level` que já alimenta `ETResolution.resolve_attempt` — reaproveita
100% a infraestrutura existente de melhoria de ET, nenhuma mecânica nova.
Por isso `SliceState` ganhou dois métodos separados e não deve
confundi-los: `et_upgrade_level(et_id)` continua sendo o nível puro
comprado na Mesa de Pesquisa (usado só pra exibir "nível X/3"), e
`effective_resolution_level(et_id)` (novo) é `et_upgrade_level +
catalyst_bonus_for` — é esse último que os 3 call-sites reais de
resolução (`operation_field_screen.gd`, `operation_grid_screen.gd` ×2)
passam pra `ETResolution.resolve_attempt`. Misturar os dois faria a Mesa
de Pesquisa mostrar "nível 4/3", incoerente.

Equipar consome o catalisador do inventário permanentemente (sem
"desequipar e recuperar") — mantém o modelo simples e casa com a ideia
de catalisador como recurso consumível (`04.6` §7), não como equipamento
reutilizável. Mesa de Pesquisa ganhou um `OptionButton` por ET
(`research_screen.gd`) pra escolher entre "Nenhum" e os catalisadores
possuídos; selecionar chama `SliceState.equip_catalyst()` direto.

Validado headless: distribuição do sorteio de raridade em 1000 rolls
bateu com os pesos configurados; taxa de drop em 200 tentativas de
`complete_operation` ficou próxima dos 15% configurados; equipar consome
o inventário corretamente e não pode ser feito sem possuir o catalisador;
desequipar (`catalyst_id` vazio) limpa o slot sem devolver ao inventário;
fluxo completo via UI (selecionar no `OptionButton`) atualiza estado e
texto da linha (`"+X catalisador"`) corretamente; regressão do fluxo
canônico completo sem vazamento de nó.

### Hub fundido com o Mapa + HUD de menus persistente

Reestruturação de navegação pedida diretamente pelo usuário: "tela
inicial = mapa do mundo", com HUD de botões redondos nas laterais
(esquerda/direita) dando acesso a todos os menus de gestão do jogo, pra
montar a **casca completa de navegação** do MVP antes de qualquer
passada de arte e poder testar tudo de ponta a ponta num device iOS.
Cruzado com `05.1` §16.2 ("Telas Obrigatórias" do MVP): mapa, hub,
seleção de operação, preparo de equipe/loadout, combate, Codex,
inventário básico e configurações mínimas já eram pedidos — faltavam só
inventário/recursos e configurações como telas próprias; o resto já
existia, só precisava ficar alcançável do lugar certo.

- **`hub_screen.gd`/`.tscn` absorveu `region_map_screen.gd`/`.tscn`**
  (apagado depois da migração) — o Mapa de Aletheia agora é a própria
  tela inicial, sem clique extra a partir de um Hub separado. Escolha
  deliberada de manter o **mesmo caminho de arquivo**
  (`res://scenes/hub/hub_screen.tscn`) como destino da fusão: toda outra
  tela do jogo já usa esse caminho como `HUB_SCENE`/"Voltar ao Hub"
  (Briefing, Loadout, Campo, Grid, Relatório, Pesquisa) — fundir ali em
  vez de criar uma tela nova evitou precisar tocar em nenhuma dessas
  telas. Só `region_detail_screen.gd` mudou (botão "Voltar ao Mapa"
  agora aponta pra `HUB_SCENE` em vez do caminho apagado de
  `region_map_screen`).
- **Layout revisado logo em seguida, a pedido do usuário**: a primeira
  versão da fusão mostrava Ferrária como um card grande em destaque +
  lista separada de regiões bloqueadas abaixo — o usuário sentiu que
  isso não passava sensação de "mapa geral", parecia já abrir dentro da
  região. Redesenhado como `RegionsGrid` (`GridContainer`, 2 colunas), um
  grid uniforme com as 8 regiões (Ferrária + 7 bloqueadas) no mesmo peso
  visual — só o botão de Ferrária fica habilitado, os outros 7
  desabilitados com mesmo tamanho/estilo. Texto de Ferrária (nome,
  subtítulo, Academia, estabilidade, pendências de Codex, nota de
  repetível quando `SliceState.repeatable_unlocked`) foi todo pro
  próprio botão multi-linha, em vez de labels separadas — mantém tudo
  dentro do grid uniforme sem precisar de um painel de detalhe reativo à
  parte. A Varredura de Estabilização não virou uma "zona" separada no
  mapa (já pertence a Ferrária via `region_id`, inventar uma segunda
  região só pra ela contrariaria o grounding em `/docs`), só ficou
  visível via essa nota.
- **HUD de 5 botões redondos** (`LeftHudButtons`: Personagens,
  Habilidades, Recursos; `RightHudButtons`: Codex, Configurações),
  `custom_minimum_size` fixo + `StyleBoxFlat` com `corner_radius_*=50`
  compartilhado entre os 5 — redondos mesmo sem ícone/arte ainda
  (textura entra na passada de arte, usando o design system já
  produzido nesta sessão). O HUD aparece **só** nessa tela — não durante
  Briefing/Loadout/Combate/Relatório, que continuam telas de decisão
  única (`05.8` §8.3), então não ganharam HUD nenhum.
- **Personagens reaproveita `loadout_screen.gd`/`.tscn` sem tela nova** —
  ganhou um modo standalone quando chamado sem `operation_id` (a partir
  do HUD, não do Briefing): troca de build já salva na hora (como
  sempre), mas "Confirmar" vira "Salvar build" e só volta pro Hub, em
  vez de rotear pra uma cena de combate que não existiria contexto pra
  montar. Modo normal (a partir do Briefing) continua idêntico.
- **Recursos** (`resources_screen.gd`/`.tscn`, novo) mostra
  `SliceState.resources` + catalisadores possuídos (mesma lógica de
  `research_screen._build_owned_catalysts_text()`, sem duplicar) +
  catalisadores **equipados** por ET — essa segunda lista não existia em
  lugar nenhum antes, só dava pra inferir olhando ET por ET na Mesa de
  Pesquisa.
- **Configurações** (`settings_screen.gd`/`.tscn`, novo) é um stub
  consciente: "Nenhuma configuração disponível ainda." — não existe
  nada de fato configurável no jogo hoje (sem áudio, idioma ou
  acessibilidade implementados), inventar opções falsas seria escopo
  fantasma. O valor é só fechar a navegação: o botão existe e leva a
  algum lugar coerente.
- **Fora de escopo, registrado como lacuna consciente**: tutorial
  inicial (também "Tela Obrigatória" de `05.1` §16.2) não entrou —
  não foi pedido e não existe conteúdo/roteiro de tutorial definido em
  lugar nenhum ainda.
- **Sem trabalho novo de combate**: "um tipo de cada batalha, incluindo
  a repetível" já estava coberto (Primeira Fissura simples, Vestígio
  Discrepante grid, Varredura grid repetível) — confirmado no pedido,
  não gerou nenhuma mudança de código de combate.

Validado headless: tela inicial já mostra o mapa sem clique extra; os 5
botões do HUD levam às telas certas e voltam sem vazamento de nó;
Personagens alterna corretamente entre texto/comportamento standalone
("Salvar build") e normal ("Confirmar e entrar em campo") dependendo de
como foi chamado; nota de repetível aparece só depois de
`repeatable_unlocked`; regressão do fluxo canônico completo (Primeira
Fissura → Vestígio Discrepante → Varredura) sem quebra.

### Primeiro tile de arte real no grid (via PixelLab MCP)

Primeira peça de arte de verdade a entrar no jogo, substituindo
placeholder de texto — o resto da apresentação (agentes, inimigos, ícones
de ET, UI) continua em texto/placeholder, isso cobre só chão/obstáculo do
grid. Fluxo: usuário instalou o servidor MCP do PixelLab via `claude mcp
add`, mas registrou no projeto errado (`Fragmentos_de_Alma`) e no escopo
do diretório pessoal — não em `AZOTH-Breach`, por isso não aparecia na
sessão. Corrigido registrando de novo com `--scope local` (padrão) a
partir do diretório do projeto, reaproveitando a mesma URL/token que já
estava salvos no config do usuário (sem pedir a chave de novo em chat).

- **Primeira tentativa (`create_topdown_tileset`, um Wang tileset de 16
  tiles com autotiling de canto) foi abandonada por feedback direto do
  usuário** — achou o resultado ruim. Preferência confirmada: gerar tiles
  individuais em vez de tileset, e nesse caso o usuário já tinha gerado 9
  tiles separados (via alguma ferramenta externa de tile/mapa, exportados
  como PNGs individuais 96×96 numa pasta local, não pelo MCP diretamente
  nesta sessão) e pediu pra eu usá-los em vez de gerar mais.
- Os 9 tiles foram inspecionados visualmente (grade de ventilação, piso
  de pedra/cobblestone, painel de metal rebitado enferrujado, grelha de
  dreno, painel com listras de risco, metal com mancha escura, painel
  com janela, escada/barras enferrujadas numa parede, variante de piso
  em pedra) — escolhidos **`tile_1` (piso de pedra) como chão** e
  **`tile_2` (painel de metal enferrujado rebitado) como obstáculo**,
  copiados por cima de `floor.png`/`obstacle.png` em
  `game/assets/sprites/tiles/` (mesmos caminhos que o código já
  referenciava, nenhuma mudança de código necessária). Os outros 7 ficam
  em `variantes_nao_usadas/` pra uso futuro (variação de terreno, ícones,
  estados de campo per `05.8` §9.3) — decisão de qual usar como
  chão/obstáculo é subjetiva, o usuário pode trocar só apontando outro
  arquivo se não gostar.
- Os 9 tiles já vieram nativos em 96×96 (mesmo tamanho de célula do
  grid) — melhor que o Wang tileset anterior, que era 16×16 escalado 6×.
  Sem necessidade de reescalar.
- `tileset_ferraria_wang.png`/`.json` (da tentativa anterior) apagados —
  código morto, o jogo não referenciava mais depois da troca.
- `project.godot` mantém `textures/canvas_textures/default_texture_filter=0`
  (Nearest global, já configurado na tentativa anterior — continua
  necessário pra manter pixel art nítida). Célula do grid continua
  `96x96` (já estava certa desde a primeira tentativa).
- `operation_grid_screen.gd` não mudou nesta passada — já estava
  preparado pra qualquer conteúdo em `floor.png`/`obstacle.png` (ícone de
  terreno sempre presente, separado do texto de agente/inimigo).

Validado headless: import limpo dos PNGs (`.import` gerados
automaticamente pelo Godot), simulação confirmando 36 células
renderizadas, 2 com ícone de obstáculo, 34 com ícone de chão, 6 com
texto de agente/inimigo sobreposto (3 agentes + 3 inimigos da
configuração testada), tamanho de célula `96x96` aplicado.

### Tiles ferrários ligados ao grid e smoke test mínimo

Atualização posterior de consolidação: o grid tático deixou de depender
dos placeholders genéricos `floor.png`/`obstacle.png` e agora usa a
família real `ferraria-*` em `game/assets/sprites/tiles/`:

- `ferraria-base-1.png` / `ferraria-base-2.png` alternam o piso base por
  célula, sem alterar regras de combate;
- `ferraria-contamination-1.png` /
  `ferraria-contamination-2.png` aparecem como manchas visuais de
  Breach/contaminação, especialmente em torno do `Fenômeno de Campo`;
- `ferraria-obstacle-1.png` representa obstáculos/cobertura, inclusive
  os criados por `Cristalização Controlada`.

A pasta raiz `tiles/` foi removida porque era duplicata exata dos PNGs já
copiados para `game/assets/sprites/tiles/`. As referências visuais únicas
foram preservadas em `docs/99_arquivo/referencias_visuais/`, com README
próprio e sem precedência canônica.

Também foi criado `game/tests/smoke_validation.gd`, executável com:

```sh
/Applications/Godot.app/Contents/MacOS/Godot --headless --path game --script res://tests/smoke_validation.gd
```

O teste carrega as cenas principais, confere conteúdo estruturado mínimo
e instancia `operation_grid_screen.tscn` para garantir 36 células e uso de
tiles `ferraria-base-*`, `ferraria-contamination-*` e
`ferraria-obstacle-1.png`. O primeiro teste encontrou e corrigiu um parse
real de GDScript por inferência de tipo (`distance` em
`_is_near_field_phenomenon`), então essa checagem deve ser mantida antes
de fechar mudanças no grid.

### Bug de navegação corrigido em M2

`AppShell.go_to_scene()` chamava `child.queue_free()` sem `remove_child()`
antes — `queue_free()` só desanexa a criança no fim do frame, então por um
frame a tela antiga e a nova coexistiam como filhas de `ScreenHost`. Não
causava problema visível em uso manual, mas quebrava testes automatizados
que checam qual tela está ativa logo após a transição, e é uma fragilidade
real da primitiva de roteamento. Corrigido chamando `remove_child()`
explicitamente antes do `queue_free()`.

### Regra de arquitetura descoberta em `TK-M1-012`

Scripts utilitários sem estado (loaders em `scripts/services/`) usam
`const X = preload("res://caminho/script.gd")` no arquivo que os consome,
**não** `class_name` global. `class_name` só fica resolvível depois de o
editor varrer o projeto pelo menos uma vez (cache de classes globais); rodar
`godot --headless` puro num clone novo, sem nunca ter aberto o editor, falha
com "Identifier not declared" para qualquer `class_name`. Autoloads
(`SliceState`) não têm esse problema — são registrados explicitamente em
`project.godot` `[autoload]`. Ao criar novo serviço/utilitário estático,
seguir o padrão `preload`, não `class_name`.

### Nota operacional (não é bug de código)

Durante `TK-M1-012` uma instância do editor gráfico do Godot ficou aberta em
paralelo às validações headless deste agente e ressuscitou em disco um
`scenes/app/main.tscn` já deletado desde `TK-M1-004` (aba antiga ainda
carregada na sessão gráfica). Se isso acontecer de novo: feche a aba/projeto
no editor gráfico antes de rodar validação headless, ou simplesmente delete
o arquivo de novo — ele não está no git e não faz parte da árvore atual.

## Rastreamento de trabalho

Até existir a primeira build jogável interna, o rastreamento oficial é:

- `05.11` para blocos `BL-xxx`;
- `05.12` para milestones `M1`–`M6`;
- `06.2` para tarefas técnicas `TK-M1-xxx` do milestone `M1`.

Ferramenta externa de backlog é opcional nesta fase.

## Regras que já causaram retrabalho se ignoradas

- Não introduzir backend, conta ou sync em nuvem antes do fechamento de `M3`.
- Não misturar linguagem de runtime além de GDScript antes do fechamento de `M3`.
- Não inventar sistemas, lore, nomes de tela ou escopo fora do que está em `/docs`.
- Conteúdo (operações, ETs, loadouts, recompensas, Codex) deve ser carregado por
  arquivos JSON em `/game/data`, nunca hardcoded em UI.

---
> Source: [itsmeluan/azoth-breach](https://github.com/itsmeluan/azoth-breach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
