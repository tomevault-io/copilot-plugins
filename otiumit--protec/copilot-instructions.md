## protec

> Workflow de push: build Lambda + commit + push (deploy API na AWS)


# Workflow de Push

Quando o usuário pedir para fazer **push** (ou "dar push", "commitar e dar push", etc.), seguir este fluxo:

## Comando único (recomendado)

```bash
pnpm run push "feat: descrição da alteração"
```

Isso executa automaticamente:
1. `pnpm run build:lambda` (gera `version.generated.ts`, `migrations-embedded.ts` e `apps/api/dist-lambda` — este último não vai para o Git)
2. `git add` (inclui `version.generated.ts` e `migrations-embedded.ts`)
3. `git commit -m "<mensagem>"`
4. `git push`

O workflow **Deploy API Lambda** no GitHub Actions roda `build:lambda` de novo no CI e faz **SAM deploy** para Lambda + API Gateway.

## Uso

- Substituir a mensagem entre aspas: `pnpm run push "fix: corrige validação de CPF"`
- A mensagem deve descrever as alterações (ex: `feat:`, `fix:`, `chore:`)

## Arquivos incluídos no commit (API)

- Código alterado
- `apps/api/src/version.generated.ts`
- `apps/api/src/db/migrations-embedded.ts`

## Nota: Vercel

A API **não** deve mais ser publicada na Vercel. Backend oficial: **AWS Lambda + API Gateway** (ver `docs/DEPLOY_LAMBDA.md`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OtiumIT)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/OtiumIT)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
