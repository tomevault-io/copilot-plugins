## arkode-dev-toolkit

> Manter o front Lovable intacto e adotar refactors atômicos, guiados pelo contrato OpenAPI.

# Cursor Rules — Arkode

## Objetivo
Manter o front Lovable intacto e adotar refactors atômicos, guiados pelo contrato OpenAPI.

## Pode editar SOMENTE:
- apps/web/src/adapters/** (criar/alterar clients tipados)
- apps/web/src/pages/** (substituir fetch por adapters, um arquivo por vez)
- packages/contracts/openapi.yaml (depois rodar `pnpm run typegen`)
- apps/api/app/** (rotas e lógica, sem mover pastas)

## NÃO PODE:
- Mover/renomear/deletar pastas fora do escopo permitido
- Alterar configs de build (vite, turbo, docker) sem propor diff antes
- Tocar em components gerados (apps/web/src/components/ui/**)

## Procedimento por tarefa
1) Liste arquivos que pretende alterar
2) Mostre um diff simulado
3) Aplique mudanças somente após aprovação (ou cole o patch)
4) Se afetar >5 arquivos, quebre em passos

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GuilhermeCostaProenca) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
