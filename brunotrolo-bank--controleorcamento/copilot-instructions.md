## controleorcamento

> Este arquivo é lido automaticamente por sessões do Claude Code neste

# CLAUDE.md — Contexto do Projeto para Claude Code

Este arquivo é lido automaticamente por sessões do Claude Code neste
repositório. Contém o conhecimento acumulado de meses de desenvolvimento
deste projeto — leia antes de mexer no código, especialmente as seções
de **armadilhas recorrentes**.

## O que é este projeto

**ControleOrcamento** (nome de exibição: "BudgetDev | Gestão Orçamentária")
é um web app de controle orçamentário/financeiro rodando inteiramente em
**Google Apps Script (GAS)**, com **Google Sheets** como banco de dados.
Não há backend externo, não há build step, não há framework de frontend —
é HTML/CSS/JS puro servido via `HtmlService`.

Domínio: controle de notas fiscais, fornecedores, prestadores de serviço
(consultorias/squads alocados em iniciativas de projeto), forecast
orçamentário por iniciativa, e dashboards de acompanhamento.

## Stack e não-stack

- **Backend**: arquivos `.gs` (Google Apps Script = JavaScript no runtime V8 do Google).
- **Frontend**: arquivos `.html` servidos via `HtmlService.createTemplateFromFile`,
  com `<script>`/`<style>` inline. Cada `Page_*.html` e `Card_*.html` é um
  módulo autocontido (IIFE) com seu próprio CSS, HTML e JS no mesmo arquivo.
- **Sem** bibliotecas externas de gráficos — todos os charts são SVG puro
  desenhado à mão em JS (ver `Page_Home.html`, funções tipo `_hxScale`).
- **Sem** build step, sem bundler, sem npm no runtime. `npm`/`clasp` existem
  só como ferramenta de deploy (ver `docs/DEPLOYMENT.md`).
- **Sem** testes automatizados. "Testar" = `clasp push` para um deployment
  de teste e clicar na UI manualmente.
- **Sem** framework — tudo é JS vanilla com padrão de módulo `var X = (function(){ ... })();`.

## Documentação relacionada

- `docs/ARCHITECTURE.md` — mapa de arquivos, camadas (Router/DAO), schema das
  abas do Sheets, convenções de frontend.
- `docs/DEPLOYMENT.md` — como configurar deploy automático via `clasp` +
  GitHub Actions (push para `main` → publica no Apps Script).

## Fluxo de trabalho de Git usado neste projeto

- **Merge direto na `main`** — não há branch `develop`/`staging`. Toda PR
  tem `main` como base.
- Uma feature/ajuste = uma branch curta = uma PR pequena e objetiva
  (o histórico deste repo tem dezenas de PRs pequenos, um por ajuste pedido).
- Depois que a PR é mergeada, se o deploy automático (`docs/DEPLOYMENT.md`)
  estiver configurado, o `push` para `main` dispara `clasp push` +
  `clasp deploy` automaticamente — não é necessário fazer deploy manual.
- Sem esse workflow configurado, deploy é manual: `clasp push` (ou colar o
  código no editor do Apps Script).

## Armadilhas recorrentes (leia antes de codar)

Estas são causas reais de bugs que já se repetiram várias vezes nesta base
de código. Preste atenção especial nelas.

### 1. Regex de acentos — NUNCA usar caractere combinante literal

Qualquer função de normalização de texto (remover acentos, comparar nomes)
deve usar a forma **ASCII-escapada**:

```js
// CERTO — o range Unicode de combining marks (U+0300 a U+036F) é
// referenciado só por código de escape, nunca digitando o caractere em si.
// Esta é a forma usada em DAO.gs (normalizeKey) — copie literalmente
// dali sempre que precisar de uma função parecida.
const semAcento = str.normalize('NFD').replace(new RegExp('[\\u0300-\\u036f]', 'g'), '');
```

**Errado**: um regex literal (`/[...]/`) contendo o caractere combinante
Unicode de verdade dentro dos colchetes, em vez do código de escape acima.
Costuma ser introduzido sem querer ao colar código de um editor ou ao
gerar código com ajuda de IA sem revisar o resultado. O arquivo parece
salvar normalmente e o bug só aparece depois, quando o pipeline de
edição corrompe o byte — manifestando como "o filtro de Iniciativa veio
vazio", bug que já se repetiu pelo menos 3 vezes nesta base. Regra prática:
**todo `replace(/[...]/g, ...)` que mexe com acentuação deve usar
`new RegExp('[\\u0300-\\u036f]', 'g')` com o range escrito em `\uXXXX`,
nunca um literal `/[...]VOGAL_ACENTUADA_COLADA.../`.**

Isso vale tanto em `.gs` (`normalizeKey` em `DAO.gs`) quanto em qualquer
`.html` que normalize nomes/colunas.

### 2. `_route()` acha o resultado na RAIZ da resposta, não em `res.data`

Em `Router.gs`, todo endpoint é envolvido por `_route(handler)`:

```js
function _route(handler) {
  const result = handler();
  return Object.assign({ success: true, metrics: {...} }, result);
}
```

Ou seja, se o handler retorna `{ rows, months }`, a resposta final do
`google.script.run` é `{ success, metrics, rows, months }` — **os campos
ficam na raiz**, não em `res.data.rows`. Isso já causou bugs de
"undefined" no frontend múltiplas vezes. Sempre confira o que o handler
específico retorna antes de acessar `res.data.X` — na maioria dos casos é
`res.X` diretamente.

### 3. V8 do Apps Script não aceita function declaration dentro de bloco

```js
// ERRADO — SyntaxError em runtime V8 do GAS dentro de try/if
if (cond) {
  function minhaFuncao() { ... }
}

// CERTO
if (cond) {
  const minhaFuncao = function() { ... };
}
```

### 4. `ALLOWED_SHEETS` é uma whitelist obrigatória

Qualquer endpoint genérico de CRUD (`routerGetSheet`, `routerInsertRow`,
`routerSaveComponent`, etc. em `Router.gs`/`ComponentHelper.gs`) valida a
aba contra `ALLOWED_SHEETS` via `_assertAllowed`. Se uma nova aba do
Sheets precisar ser acessada pelo frontend genérico, ela **tem que ser
adicionada em `ALLOWED_SHEETS`** (`Router.gs`) ou a chamada falha com
"Aba não autorizada".

### 5. Cache de cabeçalho de 6h — invalidar ao mudar colunas

`DAO.gs` cacheia o mapa de colunas (`getHeaderMap`) por 6 horas via
`CacheService`. Se uma coluna for renomeada/adicionada/removida numa aba
do Sheets, é preciso chamar `invalidateHeaderCache(sheetName)` (ou rodar
`limparCache()` em `codigo.gs`), senão o app continua lendo o mapa antigo
até o cache expirar.

### 6. Placeholders de matrícula na aba PRESTADOR

A aba `PRESTADOR` pode ter matrículas placeholder no formato `P0XXXXXX`
(prestadores sem matrícula real cadastrada ainda). O join entre
`PRESTADOR` e `NOTA_FISCAL_ANALITICO` (para achar o "realizado" de cada
profissional) usa `_joinKey(mat, nome)` em `Page_Home.html`: se a
matrícula for um placeholder (`/^p?0+$/i`), o join cai para nome
normalizado em vez de matrícula. **Nunca colapse múltiplos prestadores
placeholder pela matrícula** — são pessoas diferentes com a mesma
matrícula fictícia.

## Componente "Profissionais Alocados" (Page_Home.html)

O componente mais elaborado do dashboard. Duas vistas alternáveis
(`_profView`: `'lista' | 'squad'`):

- **Lista**: tabela plana, uma linha por prestador (aba `PRESTADOR`, sem
  colapsar), com filtros locais (Papel, Iniciativa, Fornecedor, Status,
  Busca) guardados em `_profFilters`.
- **Squad**: agrupa os mesmos prestadores por **iniciativa** em cards
  visuais. Cada card mostra: cabeçalho colorido com nome/código da
  iniciativa e estimativa de capacidade de entrega (User Stories/mês,
  calculada a partir da contagem de desenvolvedores no squad), faixa de
  composição por papel, tabela compacta de membros (matrícula, nome,
  papel, consultoria, rate card, est. mensal) e rodapé com totais por
  consultoria + total geral.

Fonte dos dados: `_prestadores` (todas as linhas da aba `PRESTADOR`,
sem colapsar) + `_realizadoAnalitico(data)` (agrega realizado por
prestador a partir de `NOTA_FISCAL_ANALITICO`, indexado por
`_joinKey`). Filtro de Papel/Fornecedor/Status respeita `PRESTADOR`;
filtro de Iniciativa respeita a aba analítica.

## Convenções de frontend em Page_Home.html (arquivo grande, ~110KB)

- Cada seção do dashboard é: 1 bloco HTML estático (`<div id="home-row-X">`)
  + 1 função `renderX(data)` que gera o HTML via concatenação de string
  (`h += '...'`) e injeta com `el.innerHTML = h`.
- `_attachHandlers()` é chamado uma única vez e liga todos os listeners
  (filtros locais, toggles, cliques de expandir card).
- O evento global `filterchange` (CustomEvent, disparado por
  `Card_Filter.html`) dispara o recálculo de `_filteredData` e re-render
  de todos os componentes do dashboard.
- Paleta de cores dos gráficos/squads: array `_CL` (10 cores fixas),
  indexado por posição/índice do item.
- Helpers de formatação: `BRL`/`BRL0` (moeda), `R()` (arredondamento para
  2 casas), `E()` (escape de HTML).

## Como validar uma mudança antes de dar como pronta

Não há testes automatizados nem lint configurado. Validação é:
1. Ler o diff com atenção a V8 quirks e ao padrão `_route`/`res.X`.
2. Se possível, `clasp push` para um ambiente de teste e clicar na
   funcionalidade alterada na UI real.
3. Nunca assumir que "compilou" = "funciona" — Apps Script só falha em
   runtime, não há checagem estática.

---
> Source: [brunotrolo-bank/ControleOrcamento](https://github.com/brunotrolo-bank/ControleOrcamento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
