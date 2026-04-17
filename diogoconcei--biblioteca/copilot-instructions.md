## biblioteca

> Aplicação desktop de leitura (Electron + React + TypeScript + Vite).

# GEMINI.md — Biblioteca (LegacyReader)

## Identidade do Projeto

Aplicação desktop de leitura (Electron + React + TypeScript + Vite).
Arquitetura Main/Renderer com comunicação via IPC (invoke/handle).
Stack: Electron, React, TypeScript, Zustand, MUI, SCSS Modules, pdfjs-dist, Sharp.

---

## Regras Gerais

- Não refatore código que não foi pedido. Se perceber algo errado fora do escopo
  da tarefa, sinalize em um comentário `// GEMINI NOTE:` e siga em frente.
- Não crie arquivos novos sem avisar explicitamente quais serão criados e por quê.
- Não instale dependências sem perguntar primeiro.
- Prefira alterações cirúrgicas. Se a tarefa é corrigir uma função, não reescreva
  o arquivo inteiro.
- O arquivo `KNOWN_ISSUES.md` NUNCA deve ser sobrescrito. Seu conteúdo deve ser
  sempre somado/acumulado.
- Sempre que terminar uma tarefa, atualize a seção "Contexto da próxima sessão"
  no TASKS.md.

---

## Processo Main (electron/)

### StorageManager

- Escritas no JSON store NUNCA devem ser paralelas.
- Toda operação de escrita deve ser enfileirada (fila de Promises serializada).
- O cache em memória deve ser a fonte de verdade durante a sessão. Leituras
  do disco só ocorrem se o dado não estiver no cache.
- Nunca quebre a interface pública dos métodos existentes sem avisar.

### IPC Handlers (ipc/)

- Handlers são finos. Lógica de negócio fica nos Services, não nos handlers.
- Todo handler deve ter tratamento de erro explícito com try/catch.
- Nomenclatura de canais: `domínio:ação` (ex: `chapter:get`, `series:patch`).
  Não invente novos padrões.

### Services (services/)

- Os Managers são singletons. Não instancie novos Managers dentro de handlers.
- Novos adapters de formato (PDF, EPUB) DEVEM implementar a interface
  MediaAdapter antes de qualquer implementação concreta:

```typescript
interface MediaAdapter {
  getPages(chapterPath: string): Promise<string[]>;
  getCover(seriesPath: string): Promise<string>;
}
```

- O PdfManager existente será refatorado para implementar essa interface.
  Não crie EpubAdapter.ts antes dessa refatoração estar feita.

### MediaServer

- O protocolo lib-media:// não deve ser modificado.
- Qualquer servidor HTTP para LAN sharing deve rodar em processo separado,
  paralelo ao protocolo existente — não substitui nem altera o lib-media://.
- O servidor LAN deve ter token de sessão obrigatório. Não implemente
  endpoints abertos na rede local.
- O servidor LAN deve utilizar mDNS (Bonjour) para descoberta facilitada
  (`biblioteca.local`) e QR Code para conexão automática no mobile.
- Todas as rotas de mídia LAN devem realizar a transformação de caminhos
  `lib-media://` para rotas HTTP internas de forma transparente.

---

## Processo Renderer (src/)

### Viewer.tsx

- O Viewer recebe apenas URLs de imagem. Ele não conhece o formato de origem.
- Novos modos de visualização (Webtoon, Double Page) são extensões do Viewer
  existente, não rewrites. Preserve a lógica de useChapter, useNavigation
  e useDrag.
- EPUB não é sequência de imagens. Não tente adaptar o Viewer atual para
  renderizar EPUB diretamente — isso requer decisão arquitetural separada.
  Se chegar nessa tarefa, pare e pergunte.

### Hooks (hooks/)

- Hooks abstraem IPC. Lógica de negócio não vai em hooks.
- Não acesse window.electron fora de hooks ou de arquivos explicitamente
  designados para isso.

### Store (store/)

- Zustand é para estado global entre páginas.
- Estado local de componente (ex: estado de UI, loading de um botão) fica
  em useState, não no store global.
- Não adicione lógica assíncrona complexa dentro das actions do Zustand.
  Async fica no hook, o store só recebe o resultado.

---

## Segurança e Performance

- **Validação de Caminhos:** O `MediaServer` deve validar todo `filePath` via `isPathSafe` antes de servir qualquer recurso através do protocolo `lib-media://`.
- **Virtualização de Caminhos (MAX_PATH):** Todo caminho físico gerado para capítulos ou capas (thumbnails/showcase) DEVE ser virtualizado via `FileManager.buildChapterPath` ou `FileManager.buildImagePath`. É terminantemente proibido utilizar o nome original da série ou capítulo diretamente no sistema de arquivos para evitar erros de estoiro de limite de caracteres (260 caracteres) no Windows, o que causa falhas silenciosas em bibliotecas nativas como `sharp`.
- **Gerenciamento de Memória (PDF):** Todo componente que utilize `pdfjsLib.getDocument` DEVE implementar uma função de limpeza que chame `pdfDoc.destroy()` para evitar vazamentos de memória em documentos grandes.
- **Sandboxing (EPUB):** O visualizador de EPUB deve utilizar um `iframe` com o atributo `sandbox="allow-scripts"` (sem `allow-same-origin`). O livro deve ser carregado via `ArrayBuffer` para permitir que a engine (`epub.js`) gerencie recursos internos sem precisar acessar o protocolo `lib-media://` como mesma origem, prevenindo ataques de escape de sandbox e acesso indevido ao sistema de arquivos.
- **Protocolo lib-media://:** O host `local` e `archive` são as únicas formas permitidas de acessar arquivos de mídia do disco. O uso de `file://` é terminantemente proibido.

---

## TypeScript

- Sem `any`. Se o tipo não é conhecido, use `unknown` e faça narrowing explícito.
- Sem `@ts-ignore`. Se precisar suprimir um erro, use `@ts-expect-error` com
  um comentário explicando o motivo.
- Interfaces de domínio (Series, Chapter, etc.) vivem em `types/`. Não
  redefina tipos inline em componentes ou handlers.
- Enums de status (`Pendente`, `Em andamento`, `Completo`) devem vir de
  `series.interfaces.ts`. O StorageManager consome esses enums,
  não define os seus próprios.

---

## O que nunca fazer

- Não desative webSecurity no Electron.
- Não use nodeIntegration: true.
- Não acesse o sistema de arquivos diretamente do Renderer.
- Não exponha APIs do Node.js inteiras via contextBridge — exponha apenas
  métodos específicos e tipados.
- Não use file:// para carregar mídia. Use lib-media://.
- Não faça writes paralelos no JSON store.
- Não crie EpubAdapter.ts antes da interface MediaAdapter estar definida
  e o PdfManager refatorado.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DiogoConcei) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
