## shortcut

> Guia para agentes de IA (e humanos) trabalharem neste repositório.

# AGENTS.md

Guia para agentes de IA (e humanos) trabalharem neste repositório.

## O que é

Launcher estilo Spotlight para Linux (KDE): busca de aplicativos, calculadora e
histórico de área de transferência (texto, imagens e arquivos).

- **Frontend**: Solid 2 (RC) + Tailwind 4 + Vite
- **Backend**: Rust + Tauri 2 (webview via WebKitGTK)
- Instância única: rodar o binário de novo alterna a janela

## Estrutura

```
src/                    → frontend Solid 2
  App.tsx               → componente principal (busca, lista, configurações)
  main.tsx              → entry (render de @solidjs/web)
  app.css               → Tailwind 4 (@theme define a paleta)
src-tauri/
  tauri.conf.json       → janela 720x464 transparente, asset protocol
  capabilities/default.json
  src/
    lib.rs              → comandos Tauri + ciclo da janela + instância única
    apps.rs             → varredura de .desktop + resolução de ícones
    clipboard.rs        → watcher + histórico (texto/imagem/arquivo) + colar
    config.rs           → Config persistida (~/.config/shortcut/config.json)
    search.rs           → busca fuzzy + calculadora + tipos
```

## Comandos

```sh
npm install
npm run tauri dev          # dev (vite :1420 + cargo run)
npm run build              # só o frontend (vite build → dist/)
npm run build:release      # tauri build --bundles deb
```

Binário: `src-tauri/target/release/shortcut` · Pacote: `src-tauri/target/release/bundle/deb/`.

O AppImage falha no ambiente do usuário (linuxdeploy) — use sempre `--bundles deb`.

## Frontend — armadilhas do Solid 2 (RC)

Versões presas em tags `next`/RC (não "upgrade" sem testar):
`solid-js@2.0.0-rc.0`, `vite-plugin-solid@3.0.0-next.27`.

APIs mudadas em relação ao Solid 1.x (usar assim):

- `render` vem de **`@solidjs/web`**, não `solid-js/web`.
- **`onMount` não existe** — usar `onSettled` (retornar a função de cleanup
  como retorno, **não** chamar `onCleanup` dentro dele).
- **`createEffect` exige 2 argumentos**: `createEffect(() => valorTracked, (v) => { efeito })`.
  Single-arg com side-effect dentro causa `MISSING_EFFECT_FN`.
- Componentes de fluxo (`For`, `Show`) vêm de `solid-js`; `render`/`Portal` de `@solidjs/web`.
- JSX: `jsxImportSource: "solid-js"` no tsconfig.

Paleta no `@theme` do `app.css` (`--color-panel`, `--color-text-dim`, …) —
classes usadas: `bg-panel`, `text-text-main`, `text-text-dim`, `border-panel-border`,
`bg-item-active`, `bg-item-hover`, `text-accent`.

## Backend — convenções

- Comandos em `lib.rs` registrados em `invoke_handler!(generate_handler![...])`.
  Os nomes em JS são camelCase (ex: `search_cmd` → `invoke("search_cmd")`).
- `search_cmd` injeta um resultado `kind: "settings"` quando a busca casa com
  palavras-chave (config/atalho/histórico/etc.) — é assim que as configurações
  abrem (a frente só troca de modo ao executar `kind === "settings"`).
- `clipboard.rs`: histórico com `kind` em `"text" | "image" | "file"`. Imagens
  viram PNG em `~/.local/share/shortcut/images/` (+ thumbnail em `thumbs/`).
  `CONFIG` (config.rs) controla `max_history`/`save_images` — mudou a config,
  chame `HISTORY.lock().apply_limit()`.
- Ícones de apps: `resolve_icon_cached` (cache por nome). O frontend usa
  `convertFileSrc` para `file://` — o escopo está no `tauri.conf.json`
  (ícones do sistema + `images/`).

## Wayland — decisões importantes (NÃO reverter)

- **Visibilidade da janela é rastreada manualmente** (`WINDOW_VISIBLE: AtomicBool`
  em lib.rs). `window.is_visible()` do Tauri/wry é não-confiável no Wayland e
  quebra o toggle (mostra em vez de esconder). `toggle_main` usa o flag.
- **Atalho global** (`tauri-plugin-global-shortcut`, "Alt+Space") só é confiável
  no X11. No Wayland o usuário usa um atalho personalizado do KDE que executa o
  binário (instância única alterna). O handler do plugin responde apenas a
  `ShortcutState::Pressed` (senão release duplica o toggle).
- **Colar** (Ctrl+V simulado) exige `wtype` no Wayland; sem ele o item só é
  copiado. `maybe_paste()` em clipboard.rs.
- O `beforeDevCommand` roda vite na porta 1420 (strictPort).

## Verificação rápida

- `npm run build` deve passar sem erros (frontend).
- `cargo build` em src-tauri sem warnings.
- Teste funcional: `npm run tauri dev` (janela 720x464 centralizada) e rodar o
  binário de novo para alternar. No Wayland, checar estado da janela via KWin
  scripting (`workspace.windowList()` → `resourceClass === "shortcut"`).

---
> Source: [felipebrgs1/shortcut](https://github.com/felipebrgs1/shortcut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
