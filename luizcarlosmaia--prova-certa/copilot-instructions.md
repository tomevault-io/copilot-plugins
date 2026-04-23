## prova-certa

> Best practices for code generation and consistent Netlify builds.


- Priorize TypeScript (stack atual); use JavaScript apenas quando solicitado e mantenha tipos claros via JSDoc.
- Siga `docs/stack/stack-versions.md` e `docs/architecture/folder-structure.md` antes de criar/mover arquivos.
- Para gerar codigo, use o comando `codegen-js` e a skill `codegen-js`.
- Para build/deploy, use o comando `netlify-build-check` e a skill `netlify-build`.
- Codigo e identificadores em ingles; textos de UI em pt-br.
- Prefira funcoes pequenas, modulos coesos e separacao clara entre UI, dominio (regras de negocio) e IO (acoes/servicos).
- Valide entradas com Zod e trate erros com mensagens objetivas; nao confie em dados do cliente.
- Use ESM (`import`/`export`) e `async/await`; evite callbacks e efeitos colaterais em helpers.
- Evite novas dependencias externas sem necessidade e sem verificar impacto em performance/SEO.
- Para Netlify, `netlify.toml` e `README.md` sao a fonte de verdade do build e deploy.
- Mantenha `@netlify/plugin-nextjs`, `publish = ".next"` e o build via `npm run build:netlify`.
- Garanta `NODE_VERSION`/`NPM_VERSION` alinhados ao `netlify.toml` e docs relevantes.
- Variaveis de ambiente obrigatorias devem estar em `docs/v1/environment.md`; use `NEXT_PUBLIC_*` apenas no client.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Luizcarlosmaia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
