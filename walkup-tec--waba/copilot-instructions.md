## sobe-para-o-servidor

> Gatilho "sobe para o servidor" — commit + push em master para disparar deploy FTP (GitHub Actions)


# Sobe para o servidor (Waba)

Quando o usuário disser **"sobe para o servidor"** ou equivalente (**publicar no servidor**, **deploy FTP**, **atualizar o que está no ar via Git**), execute **de ponta a ponta** sem só descrever passos: use o terminal no **root do repositório** (`E:\Waba` ou raiz do workspace).

## Objetivo

Garantir que o **remoto `origin/master`** receba as alterações do workspace para que o workflow **Deploy FTP (bundle)** (`.github/workflows/deploy-ftp.yml`) rode e o **ambiente publicado** fique alinhado ao Git.

## Sequência obrigatória

1. **`git status`** — ver branch e alterações. Deve estar em **`master`** (ou alinhar com o que o workflow dispara; hoje é `master`).
2. **Staging** — incluir alterações de código/config/docs que devem ir para produção (`src/`, `index.html`, `package.json`, `package-lock.json`, `.github/`, `scripts/`, `Dockerfile`, `.dockerignore`, `doc/`, etc.).
3. **Nunca commitar:** `.env`, ficheiros com segredos, `shortener-waba.zip` (e artefatos pesados desnecessários). `.env` está no `.gitignore`; não forçar `git add -f .env`.
4. **`dist/`** — o CI gera build no runner (`npm run bundle:ftp`). Pode commitar `dist/` só se o projeto **exigir** versionar build; caso contrário priorizar `src/` e deixar o Actions gerar o bundle.
5. Se **não houver nada para commitar** mas o usuário quiser **reenviar** o mesmo estado ao FTP: `git commit --allow-empty -m "chore: trigger deploy FTP"` e depois push.
6. **`git commit -m "..."`** — mensagem clara (`feat:`, `fix:`, `chore:`, etc.).
7. **`git push origin master`** — dispara o deploy. Exigir permissões `git_write` e `network` quando usar a ferramenta de terminal.
8. **Resposta ao usuário:** confirmar commit hash se útil, lembrar de verificar **GitHub → Actions → Deploy FTP (bundle)** e que **Node/.env no host** continuam necessários para o app responder na URL (FTP só envia ficheiros).

## Falhas comuns

- **"not a git repository"** — `cd` para a raiz do projeto, não `System32`.
- **`dubious ownership`** — `git config --global --add safe.directory E:/Waba` (ajustar path se diferente).
- **Push rejeitado** — `git pull --rebase origin master` (resolver conflitos) e repetir push.

## Documentação do projeto

Após alterações relevantes, cumprir também `ai-task-archive-doc.mdc` (LOG em `doc/` + `doc/memoria.md`), sem duplicar conteúdo desta regra.

---
> Source: [walkup-tec/waba](https://github.com/walkup-tec/waba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
