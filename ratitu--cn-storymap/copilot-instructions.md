## cn-storymap

> Guia para agentes que trabalham neste projeto. Leia antes de editar.

# AGENTS.md

Guia para agentes que trabalham neste projeto. Leia antes de editar.

## Visão geral

Storymap de rolagem (scroll-driven) em MapLibre GL JS que mostra as 6 camadas vetoriais de `merged.pmtiles`. Sem build step: HTML/CSS/JS estáticos + libs via CDN. Texto narrativo em português, baseado em documento externo (Google Docs).

## Stack

- **MapLibre GL JS** `4.7.1` (CDN unpkg)
- **pmtiles** `3.2.1` (pin exato, sem redirect) — protocolo `pmtiles://` (`pmtiles.Protocol` + `maplibregl.addProtocol`)
- Vanilla JS (IIFE, sem módulos/bundler), CSS puro

## Arquivos

- `index.html` — estrutura da página; referências CDN; ids: `app-loader`, `map`, `basemap-select`, `layers-btn`, `layer-control`, `legend`, `legend-content`, `dots`, `story`.
- `style.css` — tema escuro; cartões `.card`, `.chapter.active`, pontos `.dots`, painéis.
- `app.js` — toda a config e lógica (IIFE `(function(){ "use strict"; ... })()`).
- `merged.pmtiles` — fonte de dados (binário; não editar manualmente).

## Onde fica cada coisa (app.js)

- `BASEMAPS` — estilos de mapa base (`satellite` default, `liberty`, `osm`).
- `LAYERS` — cada camada: `key`, `title`, `source` (source-layer no pmtiles), `paint` (objetos `fill`/`line`) e `legend` (lista `{color, label}`). Camadas no mapa recebem ids `{key}-fill` e `{key}-line`.
- `CHAPTERS` — capítulos: `cover?`, `kicker`, `title`, `text` (HTML), `center` [lng,lat], `zoom`, `layers` (chaves de `LAYERS` visíveis). A ordem do array define a ordem do scroll e dos dots.
- `setActive(i)` — capítulo ativo: `map.easeTo` + `applyVisibility` + classes `.chapter.active`/dots.
- `addOverlayLayers()` — cria a source `pm` (`pmtiles://merged.pmtiles`) e adiciona os layers; re-utilizada ao trocar basemap.
- `window.__map = map` — hook de depuração no console (ex.: `__map.queryRenderedFeatures`).

## Convenções

- Não adicionar comentários ao código a menos que solicitado.
- Seguir a config declarativa: novos capítulos/camadas entram nos arrays `CHAPTERS`/`LAYERS`, não em código imperativo.
- Texto dos capítulos em português, HTML inline em `ch.text`.
- Mapas base usam tiles externos; overlay sempre adicionado por cima via `addOverlayLayers()`.

## Rodar localmente

```bash
npx http-server -c-1 .    # ou: npx serve .
```

**Importante:** o protocolo pmtiles exige servidor com **HTTP Range** (`206`). `python3 -m http.server` NÃO funciona (retorna `200` sem `206`) — verificado.

## Verificação

- Sintaxe JS: `node --check app.js`
- IDs do HTML referenciados em `app.js` devem existir em `index.html`.
- Teste funcional headless (opcional): `npx -y http-server -p 8090 -c-1 .` e script Puppeteer-core apontando para `/usr/bin/chromium`; conferir que (a) requests a `merged.pmtiles` retornam `206`, (b) `__map.getSource("pm").loaded()` é true, (c) `__map.queryRenderedFeatures({layers:["{key}-fill"]})` retorna features por capítulo, (d) classes `.chapter.active` alternam ao rolar.
- Sem lint/tests formais configurados.

## Armadilhas conhecidas

- `estrada` é **polígono** (ADA), não linha — renderizado como fill+line.
- `hids`/`pids` só têm dados em zoom ≥ 10 (tippecanoe `--minimum-zoom=10`).
- ZEU, bens CONDEPACC e ARIE Mata Santa Genebra **não** estão no pmtiles — aparecem apenas como narrativa em `CHAPTERS`.
- Ao trocar basemap, o style é recriado; os layers do pmtiles precisam ser readicionados (já tratado por `addOverlayLayers()` após `isStyleLoaded()`).
- `ERR_ABORTED` em requests de tiles durante `easeTo` é comportamento normal do MapLibre (cancela requisições em voo).

---
> Source: [ratitu/cn_storymap](https://github.com/ratitu/cn_storymap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
