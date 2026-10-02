## curupira

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# CURUPIRA

Interface web para o yt-dlp. Um produto Abner Software. Responda ao usuário em pt-BR.

## Antes de escrever UI

Leia **`DESIGN.md`** na raiz do projeto. Ele é o contrato visual: use apenas os tokens
definidos lá, através das CSS custom properties já declaradas em `src/app/globals.css`
(`--bg`, `--surface`, `--accent`, `--sp-4`, `--fs-lg`…). Não introduza cor, tamanho de fonte
ou espaçamento novo. Mobile-first, todo estado com ícone **e** rótulo, foco visível
obrigatório, microcopy literal em pt-BR — sem jargão de terminal.

## Antes de mexer no backend

Leia a seção "Decisões que valem conhecer antes de mexer" de `docs/ARQUITETURA.md`. Cada
item ali corrige um bug reproduzido em teste (encoding do processo filho no Windows,
truncamento de nome de arquivo, contagem de bytes entre faixas, retomada que o YouTube
recusa, processo órfão no cancelamento, redirect relativo no proxy). Desfazer qualquer um
deles reintroduz o defeito. O `README.md` é vitrine — não é lugar de documentação técnica.

## Limites

- `src/lib/ytdlp.ts` é a única fronteira com processos externos. Nenhuma outra camada importa
  `child_process` em runtime — subir, matar e ler processo passa por lá (`spawnDownload`,
  `killTree`, `analyze`). `import type { ChildProcess }` é permitido, porque some na
  compilação e não cria dependência real.
- A URL do usuário nunca entra em shell — sempre lista de argumentos.
- Formato e resolução vêm de allowlist no servidor; o cliente não escolhe flags.

---
> Source: [AbnerSantosss/curupira](https://github.com/AbnerSantosss/curupira) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
