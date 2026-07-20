## pje-ia

> Extensão Chrome (Manifest V3, JavaScript puro, **sem build step**) que adiciona um painel

# PJe IA — Extensão Chrome

Extensão Chrome (Manifest V3, JavaScript puro, **sem build step**) que adiciona um painel
de chat com Claude à tela de autos digitais do PJe. O usuário seleciona peças do
processo e conversa sobre elas; os PDFs são enviados diretamente à API da Anthropic.

## Arquitetura

**Multi-PJe (default-on)**: `content_scripts`, `host_permissions` e
`web_accessible_resources` cobrem `https://*.jus.br/*` — qualquer tribunal
funciona sem nenhuma ação do usuário (decisão de produto: zero fricção; o
aviso de permissão do Chrome fica mais amplo, aceito). Como o script roda em
TODA página jus.br (login SSO, portais…), o boot do painel em `content.js`
vive em `iniciar()`, chamada só quando `#divTimeLine` existe (ou surge — SPA
do PJe novo) — sem timeline, nada é injetado no DOM. O grau e o base path
variam por tribunal (`pje.tjce.jus.br/pje1grau`, `pje1g.trf5.jus.br/pje`…):
`pje.js` deriva o base path da URL (`getBase`). `DOMINIOS_JURIDICOS` ganha o
domínio-raiz do tribunal atual em runtime (busca de jurisprudência).

Content scripts injetados nesta ordem
(cada um é um IIFE que expõe um global — não há imports entre content scripts):

| Arquivo | Global | Papel |
|---|---|---|
| `src/pje.js` | `PJE` | Acesso ao PJe: lista peças da timeline (`#divTimeLine`), baixa cada uma pelo endpoint REST autenticado por cookie de sessão. |
| `src/panel.js` | `PjePanel` | Toda a UI (chat, seletor de peças, chips, popup `@`, card de progresso), isolada em **Shadow DOM**. CSS carregado de `src/panel.css` via `web_accessible_resources`. |
| `src/content.js` | — | Orquestração: downloads com concorrência 3, cache por peça, montagem dos blocos da API, conversa multi-turno, streaming via `Port`. |

O worker (`src/background.js` + `src/claude.js`, ES modules) guarda a chave da API e faz o
streaming SSE — **a chave nunca chega ao contexto da página**. Dois canais content↔worker:

- **Port** `chrome.runtime.connect({name:"claude"})` para os turnos (streaming). Tipos
  content→worker: `chat` e `gerarDoc`; worker→content: `delta`, `thinking`, `citation`,
  `tool`, `file`, `trunc`, `iter` (início de request físico — checkpoint da UI),
  `retry` (re-tentativa transitória — a UI reverte ao checkpoint para não duplicar
  texto/citações), `done {content, stopReason}`, `error`. **AUTO-RESUME**: se a porta
  cair SEM `done`/`error` (worker MV3 morto no meio do turno — acontece mesmo com
  keepalive), `stream()` em content.js reconecta e REENVIA o payload sozinho (até 2
  vezes; o turno é stateless e o prefixo está no cache de prompt). O handler
  `onReinicio` zera TODO o estado de UI do turno (o novo stream re-emite do zero).
  Não transformar esse reenvio em erro imediato — era a causa nº 1 de ".docx falha
  às vezes" no Haiku.
- **`chrome.runtime.sendMessage`** (request/response) para `caps` (capacidades do modelo),
  `upload` (Files API) e `countTokens` (pré-voo gratuito).

## Fluxo de um turno (protocolo v2)

`claude.js` acumula os **blocos completos** da resposta a partir do SSE (padrão dos SDKs:
`content_block_start/delta/stop`, incluindo `signature_delta` do thinking, `citations_delta`
e `input_json_delta`) e emite `{kind:"final", content, stopReason, containerId}`.
`background.js` resolve sozinho as continuações de **`pause_turn`** (reenvia
`messages + [{role:"assistant", content: parcial}]`, reutilizando `container.id` quando há
skills; máx. 8 iterações no chat e 16 na geração de documento — `payload.maxIter`) — o
content script enxerga um único turno lógico. **Erros transitórios re-tentam sozinhos**:
cada request físico ganha até 2 re-tentativas com backoff (429 espera 10 s) quando o
erro é 429/529/5xx ou queda de rede no meio do SSE (flag `retryable` posta pelo
`claude.js`; janela típica: os longos silêncios do code execution no docx). Se a
geração de documento terminar sem arquivo com `stop_reason` `pause_turn` (teto de
iterações) ou `max_tokens`, o worker LANÇA erro claro em vez de retornar em silêncio —
o Haiku precisa de mais rodadas de code execution que o Sonnet e era onde o docx
"falhava às vezes" sem explicação. `maxTokens` do docx é 32000 (16000 truncava o
código do Haiku no meio).

`MODEL_CAPS` em `background.js` governa por modelo: `contextTokens`, `maxPages` (600 nos
modelos de 1M; 100 no Haiku), versões de `web_search`/`web_fetch` (variantes `_20260209`
no Sonnet 5/Opus 4.8; básicas no Fable/Haiku), `thinking` (adaptive+summarized; omitido no
Haiku) e `effort` (não suportado no Haiku).

## Invariantes importantes

- **Assistant no histórico é SEMPRE array de blocos** (`response.content` completo), nunca
  string: a API exige thinking assinado intacto e os blocos de ferramenta/citações nos
  turnos seguintes. Em fallback (sem blocos), texto puro com os placeholders de citação
  removidos. **Citações NUNCA voltam à API**: a resposta traz campos que o request
  rejeita (`file_id` em `page_location` → 400 "Extra inputs are not permitted") e,
  pior, a API revalida os `document_index` contra o layout do request atual — com o
  anexo incremental essa revalidação falha (400 "Invalid citation indices: Document
  not found for placeholder citation", sempre na 2ª mensagem). Por isso o campo
  `citations` é REMOVIDO dos blocos de texto do assistant antes de qualquer reenvio:
  `sanearCitacoes` (content.js) ao gravar no histórico e `stripCitacoes`
  (background.js) nas continuações `pause_turn`. A UI mantém as citações
  renderizadas do turno; o modelo segue vendo o texto integral.
- **Dois tipos de request, nunca misturados**: *chat/busca* (documentos + citações +
  web tools quando o toggle "Jurisprudência" está ligado — e, uma vez usadas na
  conversa, as web tools seguem declaradas nos turnos seguintes mesmo com o toggle
  desligado (`buscaNaConversa`): trocar o conjunto de tools invalidaria o cache de
  prefixo e arriscaria rejeição do histórico com blocos de ferramenta) e *gerar
  documento* (skill
  `docx` + `code_execution_20260521` + betas `code-execution-2025-08-25`/
  `skills-2025-10-02`/`files-api-2025-04-14` — o `container` com skills exige a beta de
  code execution junto com a de skills).
  As versões `_20260209` dos web tools já embutem execução de código — **nunca** declare
  `code_execution` junto delas.
- **Peças vão por `file_id` (Files API)**: upload único pelo worker com cache em
  `chrome.storage.session` (chave `idProcesso:idPeca:tamanho`); beta
  `files-api-2025-04-14` em todos os requests de chat. Base64 inline é só fallback de
  upload (aí vale o teto `MAX_TOTAL_B64_CHARS` de 24 MB).
- **Guardas de processo grande**: contagem de páginas por heurística no binário do PDF
  (`pje.js`) bloqueia acima de `MODEL_CAPS.maxPages` ANTES do envio; `count_tokens`
  (gratuito) estima o contexto e bloqueia acima de 90% da janela — e recebe as
  MESMAS tools/betas do turno (histórico com blocos de ferramenta exige as tools
  declaradas também no count_tokens, senão o pré-voo falha mudo e o medidor some).
  Tratar também `stop_reason: model_context_window_exceeded`.
- **Citações**: `citations:{enabled:true}` em TODOS os blocos document (regra da API:
  tudo-ou-nada); peças HTML viram document com source text (citáveis por
  `char_location`). No stream, `citations_delta` gera marcadores por **placeholder PUA**
  (`\uE000<n>\uE001` — sempre como escapes ASCII no código, nunca o caractere cru) que
  atravessam o escape-first do `renderMd` e viram `<sup>` só DEPOIS do escape. PDFs
  escaneados sem camada de texto não são citáveis (degradação graciosa).

- **Fonte de verdade da seleção de peças**: os checkboxes de `.doclist` em `panel.js`.
  Chips da barra de contexto, contador `(x/y no contexto)`, popup `@` e mensagens são
  *projeções* desse estado — nunca guarde seleção em outro lugar.
- **Download do PJe é stateful**: o endpoint REST só libera peças já "abertas" na sessão
  JSF. Em 404 **ou corpo vazio com HTTP 200**, `pje.js` simula o clique na timeline (A4J)
  e faz poll com HEAD até liberar. As ativações são **serializadas** (`activationChain`) —
  o JSF não tolera dois submits simultâneos na mesma view. Cada download loga
  `[PJe IA] peça …` no console da página (F12) para diagnóstico.
- **Peças de encaminhamento são normais no PJe**: petições cujo conteúdo integral é algo
  como `<p>Em Anexo</p>` (o teor real está nos anexos "Documento de Comprovação"
  protocolados junto). Não é falha de download — o system prompt instrui o modelo a
  explicar isso e sugerir marcar os anexos.
- **Anexo incremental de peças** (`pecasNaConversa`): cada peça entra no histórico UMA
  única vez; a cada turno só o DELTA (peças ainda não enviadas) é anexado. Reanexar
  tudo a cada mudança de seleção duplicava páginas/tokens no request (os blocos já
  enviados fazem parte do prefixo cacheado) e estourava os limites já no segundo envio.
- **Desmarcar peça LIBERA contexto** (`prepararEnvio` em content.js): a API é
  stateless — o histórico inteiro é remontado a cada request —, então cada bloco
  `document` carrega o campo interno `__pecaId` e, no envio, `prepararEnvio(msgs,
  ativos)` filtra os blocos das peças desmarcadas e remove `__pecaId` (a API rejeita
  campos extras; o teste do scratchpad confirma que ele nunca vaza). Blocos do
  assistant (thinking assinado, ferramentas) NUNCA são tocados. `conversation` guarda
  o turno CRU (com `__pecaId`); re-marcar a peça faz os blocos voltarem sem reanexar
  (ela segue em `pecasNaConversa`). Custo aceito: mudar a seleção invalida o cache de
  prefixo daquele ponto em diante. As guardas de páginas/tokens contam o request que
  VAI de fato (só peças ativas + histórico filtrado).
- **Feedback de contexto em três camadas** (o usuário precisa saber quando encheu):
  (1) medidor `panel.setContexto` (tokens/páginas vs. limites), atualizado no envio e
  DINAMICAMENTE ao marcar/desmarcar peças — inclusive ANTES do primeiro envio, em
  DUAS sub-camadas, porque o clique não pode esperar download nem rede:
  (1a) estimativa LOCAL instantânea (0 ms, `estimativaLocalTokens`): PDF ≈ páginas ×
  2000 tokens, texto ≈ chars/3,5 sobre o que já está em `docsCache` (o tipo vem de
  `lerCorpo` em `pje.js`: content-type + assinatura `%PDF-` nos primeiros 1024
  bytes — PDF servido como octet-stream não pode cair no ramo de texto, que
  desperdiçaria ~17 mil tokens de lixo binário; HTML honra o charset do header
  ao decodificar); peças ainda sem download aparecem como
  "N peça(s) sem medir" (`pendentes` no gauge) — nunca fingir precisão;
  (1b) refinamento em segundo plano (debounce 900 ms): `baixarQuieto` (concorrência
  3, progresso peça a peça re-alimentando a estimativa local) → `subirPecas`
  (upload à Files API já na medição: count_tokens referencia por file_id, payload
  mínimo, e o envio reaproveia — prefetch completo) → count_tokens corrige o número.
  GUARDA de escala: acima de `LIMIAR_PREFETCH` (12) peças sem cache (ex.: "todas"
  marcadas), o refinamento NÃO dispara downloads — a ativação JSF do PJe é
  serializada e levaria minutos; fica a estimativa parcial e a medição completa
  acontece no envio. `estSeq` descarta respostas atrasadas e `ultimaChaveEst`
  (ids ordenados + tamanho da conversa) evita re-medir nos refreshs da timeline —
  a chave é limpa sempre que o alerta liga, para a próxima mudança re-medir.
  Durante um turno (`busy`) o handler de seleção retorna cedo: refreshs da
  timeline do PJe disparam `syncSelection` sem mudança real e sobrescreveriam
  a medição oficial do envio. Se o count_tokens do envio falhar (ex.: 429 —
  o motivo agora vai ao console), o fallback re-pinta a estimativa local com
  o cache já cheio (sem isso o medidor congelava no retrato do clique, "N
  peça(s) sem medir"). Após o turno, `atualizarGaugePosTurno` usa o
  `usageReq` (usage do ÚLTIMO request físico — a soma das iterações
  `pause_turn` serve para custo, mas duplicaria o tamanho do contexto) como
  medição EXATA, de graça, e memoriza `ultimaChaveEst`;
  (2) bloqueio a >90% da janela em `estimarContexto` (erro com flag `ctxCheio`);
  (3) barra de alerta persistente `panel.setAlerta` (`.alertbar`, `role="alert"`, com
  botão ⟲) ligada quando o envio é bloqueado ou em `model_context_window_exceeded` —
  diferente do `.status` (transitório), só some quando a conversa volta a caber
  (desmarcar peças re-estima e limpa sozinha) ou em "Nova conversa". Compaction
  server-side foi avaliada e descartada: resumiria as próprias peças, matando as
  citações por página — a saída certa aqui é tirar/incluir peças do request.
- **Custo por resposta** (`registrarCusto` em content.js + `.custo` no painel): a
  API não devolve valor monetário — só o `usage` (tokens por categoria). O
  acumulador SSE de `claude.js` captura o usage (entrada no `message_start`,
  saída no `message_delta`); `executarTurno` (background.js) SOMA o usage de
  todas as iterações `pause_turn` (um turno lógico = vários requests físicos) e
  calcula `custoUsd` pela tabela `MODEL_CAPS[model].preco` (US$/1M tokens; cache
  write 1,25× o input, cache read 0,1×; Sonnet 5 usa preço de tabela, não o
  promocional). O `done` leva `usage`+`custoUsd`; o content acumula
  `custoConversaUsd` (zera em "Nova conversa") e `panel.setCusto` mostra no
  rodapé ("nesta resposta • na conversa", tooltip com o detalhamento).
- **Prompt caching**: `montarBlocos()` marca o último bloco com
  `cache_control: {type: "ephemeral"}` e `stripOldCacheControl()` remove breakpoints
  antigos do histórico (a API aceita no máx. 4).
- **Limite de payload**: 24 MB de base64 (`MAX_TOTAL_B64_CHARS`) com folga sob o limite de
  32 MB da API. `montarBlocos()` lança erro amigável se exceder — por isso
  `panel.endPrep()` (confirmação "peças anexadas") só é chamado **depois** de montar os
  blocos.
- **Turnos desfeitos em erro**: em falha ou resposta vazia, `content.js` faz `pop()` do
  turno do usuário e remove as peças do turno de `pecasNaConversa`, para permitir nova
  tentativa limpa.
- **Keepalive do service worker (MV3)**: o Chrome mata o worker após ~30 s sem eventos
  de extensão — fatal na geração de .docx, cujo code execution roda no servidor com
  longos silêncios no SSE (sintoma: "conexão com o serviço interrompida"). Durante um
  turno, `background.js` chama `chrome.runtime.getPlatformInfo` a cada 20 s
  (`manterVivo`) e `content.js` manda `{type:"ping"}` pela porta; o handler do Port
  ignora tipos desconhecidos. Não remova nenhum dos dois lados.
- **Markdown seguro**: `renderMd()` em `panel.js` **escapa primeiro, formata depois**.
  Qualquer mudança ali precisa preservar essa ordem (a resposta do modelo pode conter
  conteúdo dos autos).
- **Blocos `document` levam `title`** (título da peça) para o modelo citar as peças pelo
  nome — exigência do system prompt.

## Busca de peças e orientações (panel.js)

- **Busca na lista de peças** (`.docsearch`/`filtrarDocs`): filtra por título sem
  acentos (`row.dataset.busca = norm(titulo)`), só esconde/mostra linhas (`row.hidden`
  — depende da regra global `[hidden]{display:none !important}` do panel.css); os
  checkboxes seguem sendo a fonte de verdade (peça marcada e filtrada continua
  marcada). "todas" respeita o filtro ativo (marca/desmarca só as visíveis). Esc
  limpa; `setDocs` re-aplica o filtro após re-renderizar a lista.
- **Orientações no estado vazio** (`showEmptyHint`): box `.guia` explica que NÃO é
  um agente autônomo (seleciona peças → envia solicitação), o limite de contexto
  (200 mil tokens no modelo padrão Haiku 4.5; modelos de 1M na configuração —
  medidor no rodapé) e cita o TecJustiça MCP (https://mcp.tecjustica.com/) e a
  demonstração com o PJe-CE (https://pjece.tecjustica.com/) como alternativa para
  autos volumosos com gerenciamento automático de contexto. Manter os DOIS links
  ao editar o hint.

## Modos de layout, preview no hover e "ver na timeline" (panel.js/pje.js)

- **Modos de layout** (classes no `.wrap`): flutuante → `expanded` (modal central com
  backdrop) → `expanded full` (tela cheia) e o modo `lateral` (sidebar colada à
  direita, página do PJe visível e CLICÁVEL ao lado — sem backdrop; `lateral` e
  `expanded` são mutuamente exclusivas). Transições centralizadas em `aplicarModo()`
  (não voltar aos handlers inline); a preferência persiste em
  `chrome.storage.local.layoutModo` (tela cheia é transitória: persiste "expandido")
  e é restaurada no `mount()`. Botão `.side` no header entre `.expand` e `.fs`.
- **"Ver na timeline"** (botão `.d-ver` em cada docrow, aparece no hover):
  `PJE.scrollAte(id)` rola a `#divTimeLine` até a peça com flash de ~2s — o estilo
  do flash é injetado no DOM da PÁGINA (`#pje-ia-flash-style`), pois o alvo vive
  fora do Shadow DOM. `scrollAte` NÃO clica no link (zero efeito A4J/JSF, não toca
  na `activationChain`) e retorna `false` quando a peça não está na timeline (o
  content mostra orientação no `.status`). No modal (expandido/cheia) o clique troca
  para o lateral ANTES de rolar — a página estava coberta. O handler é DELEGADO no
  `.doclist` e usa `preventDefault`+`stopPropagation`: a row é um `<label>`; sem
  isso o clique alternaria o checkbox (fonte de verdade da seleção) e dispararia o
  `change`. Callback: `panel.onVerNaTimeline(cb)`.
- **Preview de peça no hover** (só nos modos expandido/cheia/lateral): popover ÚNICO
  `.preview` no Shadow DOM, debounce de intenção de 400 ms, posicionado pela
  `getBoundingClientRect` da row (direita quando cabe; senão esquerda — caso do
  lateral). O conteúdo vem SEMPRE do `docsCache` via `panel.onPreview(cb)` (callback
  SÍNCRONO) — **o hover NUNCA baixa nada**: o download do PJe é serializado na
  sessão JSF (~5,6 s/peça + clique na timeline como efeito colateral) e passadas de
  mouse travariam a extensão. Cache-miss mostra aviso + botão "Baixar"
  (`panel.onPreviewBaixar` → `PJE.baixar`, bloqueado durante `busy`; alimenta o
  MESMO `docsCache` que o envio reaproveita — prefetch de graça). PDF: no máximo UM
  blob URL vivo, revogado em todo fechamento/re-render; acima de 15 MB não
  decodifica no hover (o `atob` travaria a UI) — só metadados + "Abrir em nova aba"
  (posse do URL transferida, revogação com 30 s de folga). Texto: `textContent`,
  nunca innerHTML (conteúdo dos autos). CSP hostil da página (embed de `blob:`
  barrado) é detectada pelo evento `securitypolicyviolation` no `document` → flag de
  sessão + fallback com metadados ("Abrir em nova aba" escapa: navegação de topo não
  é governada pela CSP da página). TODOS os listeners são delegados no `.doclist`
  (as rows são recriadas a cada `setDocs`, que chama `hidePreview()`; `filtrarDocs`,
  `aplicarModo`, scroll da lista e Esc também fecham — o Esc do preview faz
  `stopPropagation` para não cancelar o modo docx junto).

## Popup de menção `@` (panel.js)

Detecção por regex do token `@busca` antes do caret (`findMentionToken`); busca ignora
acentos via `norm()` (NFD + remoção de diacríticos). Ao selecionar, o token é removido do
texto e o checkbox correspondente é alternado. Detalhes fáceis de quebrar:

- As linhas do popup usam `mousedown` + `preventDefault()` (não `click`) para agir antes
  do `blur` do textarea.
- `Enter`/`Tab` com popup aberto selecionam; só com popup fechado o `Enter` envia.
- `updateMention()` é chamado em `input`, `click`, `keyup` (setas/Home/End) e em
  `setDocs()` — todos os caminhos que movem o caret ou mudam a lista.
- Cap de `MENTION_MAX` itens com linha "… e mais N peças" quando excede.

## Geração de .docx (skill oficial)

Botão "📄 Gerar .docx" no painel liga o **modo documento** (`docxMode` em `panel.js`):
a instrução padrão (editável) entra no campo, a faixa `.docxbar` explica o passo e o
botão Enviar vira "📄 Gerar" — Enviar/Enter disparam o request `gerarDoc` com as peças
selecionadas + a instrução do campo (vazia cai na padrão). ✕ na faixa, Esc (com o
popup `@` fechado) ou novo clique no botão cancelam; "Nova conversa" também desliga o
modo. Não reintroduzir o fluxo antigo de "dois cliques no mesmo botão" — o usuário
sempre aperta Enviar. O sufixo de formatação anexado à instrução em content.js é
PRESCRITIVO de propósito (tabelas nativas obrigatórias, autocorreção de código,
verificação do arquivo com python-docx antes de entregar): modelos menores (Haiku)
seguem instruções literalmente e, sem essas regras, às vezes entregavam o .docx sem
tabelas — não suavizar o texto. O worker extrai o
`file_id` dos blocos
`bash_code_execution_tool_result` (fica com o **último** `.docx` gerado), baixa via Files
API e repassa os bytes pelo Port; o content script dispara o download com Blob + âncora
(sem permissão `downloads`). Custo: code execution tem franquia de 1.550 h/mês por
organização (US$ 0,05/h depois), além dos tokens.

## Desenvolvimento e teste

- Não há bundler. Valide sintaxe com `node --check src/*.js`. Testes de unidade fora do
  navegador no scratchpad da sessão: `renderMd` (escape-first + citações) roda com
  `eval` do `panel.js`; o acumulador SSE de `claude.js` roda com `fetch` fake devolvendo
  um `ReadableStream` de eventos simulados (chat com citação, `pause_turn`, docx).
- **Testar a UI sem PJe**: criar um HTML que stub `window.chrome`
  (`runtime.getURL`, `storage.local.get`, `runtime.connect`) e carregue `src/panel.js`,
  servido por HTTP local (fetch do CSS falha em `file://`). Chamar `PjePanel.mount()`,
  `setConfigured(true)`, `setDocs([...])` com peças fictícias. As APIs `startPrep` /
  `setPrepState` / `endPrep` / `addMessage` permitem simular todo o fluxo visual.
- Para testar no PJe de verdade: recarregar a extensão em `chrome://extensions` e
  recarregar a aba do processo (o content script tem guard `window.__pjeIaLoaded`).

## Categorias de peças (destaque visual)

`CATEGORIAS` em `panel.js` classifica cada título por regex **sobre o texto normalizado
sem acentos** (`norm()`): decisões (dourado), audiências (verde), petições (azul),
provas (violeta), outros (neutro). A primeira regra que casar vence — cuidado com
sobreposições ("ata notarial" é prova, tratada com lookahead negativo na regra de
audiências). As cores vivem em variáveis `--cat-*` no `panel.css` e aparecem na lista
lateral (dot + peso da fonte), nos chips e no popup `@`; a legenda só é exibida no modo
expandido.

## Convenções

- Comentários e strings de UI em português do Brasil (com acentuação correta).
- Identidade visual: paleta do próprio PJe — azul-petróleo `#0078aa` (`--pje`, cor
  da barra do PJe/TJCE), escurecido `#005f88` (`--pje-2`, gradientes/hover/balão do
  usuário — texto branco sobre `#0078aa` puro passa AA por pouco, por isso texto
  longo usa o tom escuro), azul claro `#62a9c7` (`--pje-soft`, medidores), fundos
  frios `#f6f9fb`, títulos em Georgia serif. Variáveis CSS no topo de `panel.css`
  (`.wrap`) e espelhadas em `ui.css` (`:root`, popup/opções/ajuda — HTMLs têm
  referências inline a `var(--pje-2)`). Cores semânticas preservadas: categorias
  `--cat-*`, verde de sucesso, laranja da `.alertbar`/gauge crítico.
- Modelos da API: manter os IDs do `popup.html`/`options.html` alinhados aos aliases
  atuais da Anthropic (`claude-haiku-4-5` é o default em `background.js` — rápido e
  barato; todas as features funcionam nele, inclusive a skill docx com
  `code_execution_20260521`; a janela menor de 200 mil tokens/100 págs. é o custo
  aceito, com o Sonnet 5 de 1M oferecido para autos volumosos) e a tabela
  `MODEL_CAPS` sincronizada com os docs (limites, versões de tools, thinking/effort).
- Config no `chrome.storage.local`: `apiKey`, `model`, `effort` (baixo/médio/alto —
  `output_config.effort`; omitido nos modelos sem suporte).
- Alternar o toggle de busca ou trocar de modelo invalida o cache de prompt daquele ponto
  em diante (comportamento aceito). Arquivos enviados à Files API persistem na conta
  (100 GB por organização) — "limpar uploads" é melhoria futura registrada.

---
> Source: [marcosmarf27/pje-ia](https://github.com/marcosmarf27/pje-ia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
