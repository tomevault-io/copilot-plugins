## project

> App mobile Expo — rotas, serviços e estilo React Native.


# `apps/mobile` — Expo

## Stack

- **Expo** (SDK do `package.json` do app) + **expo-router** onde aplicável.
- Formulários: **react-hook-form** + **zod** alinhados ao restante do monorepo.

## Padrões

- Telas e fluxos em `src/app/` ou estrutura já existente; serviços em `src/services/`.
- Estilo: **StyleSheet** / padrão RN do projeto; não assumir Tailwind neste app.

## Paridade

- Para features espelhadas no web-app, reutilizar **contratos de API** e nomes de campos; evitar divergência de DTOs sem motivo.

## Documentos

- Onde houver exportação `.docx`, o projeto já usa **`docx`** no mobile — manter o mesmo estilo de implementação ao estender.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/plural-plataforma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
