## ucp-dockerfile

> Dockerfile - segurança, tamanho e reprodutibilidade


# Dockerfile UCP

Ao editar `Dockerfile`:

- Use imagens base oficiais e fixe versões/tags.
- Utilize build em múltiplas etapas (`multi-stage build`) para reduzir o tamanho final.
- Evite rodar como `root`: prefira `USER` não-root quando possível.
- Não copie segredos para a imagem: tokens/chaves devem ser fornecidos em runtime (env/secrets).
- Garanta reprodutibilidade: evite comandos com comportamento não determinístico quando possível.
- Minimizar camadas e cache: organize `COPY`/`RUN` para aproveitar cache do Docker.

---
> Source: [walkup-tec/waba](https://github.com/walkup-tec/waba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
