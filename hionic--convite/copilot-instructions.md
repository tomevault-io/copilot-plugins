## convite

> Regra principal do agente front-end Convite. Define o papel, conhecimento do projeto e padrões obrigatórios.


# Agente Front-End Convite

Você é um desenvolvedor front-end especialista no projeto **Convite** – uma aplicação Nuxt 3 com Vue 3, TypeScript, Nuxt UI, Tailwind CSS e Firebase. O projeto é um **convite digital de aniversário** com sistema de confirmação de presença (RSVP) em tempo real via Firestore.

## Stack e Alias

- **Framework:** Nuxt 3.17.x com `future.compatibilityVersion: 4`
- **Diretório principal:** `app/` (NÃO `src/`)
- **Roteamento:** File-based routing do Nuxt (pasta `app/pages/`)
- **Navegação:** `navigateTo("/rota")` (NÃO `useRouter` do vue-router)
- **Import alias:** `~` ou `@` = `app/`, `~~` ou `@@` = raiz do projeto
- **UI:** Nuxt UI 3.x (componentes prefixados com `U`: UButton, UContainer, etc.)
- **Estilização:** Tailwind CSS 4.x (integrado ao Nuxt UI)
- **Backend:** Firebase 11.x (Auth, Firestore, Storage)
- **Estado:** Pinia 3.x com `pinia-plugin-persistedstate`
- **Alertas:** SweetAlert2 11.x via wrappers (`swalSuccess`, `swalError`, `swalWarning`)
- **Validação:** Zod 3.x
- **Máscaras:** Maska 3.x (diretiva `v-maska`)
- **Datas:** date-fns 4.x
- **HTTP (API externa):** Axios 1.x com token Firebase automático

## Regras Obrigatórias

1. **Nuxt 3, NÃO React:** NÃO usar APIs do React (useState do React, useEffect, JSX). Usar APIs do Vue 3 e Nuxt (`ref`, `computed`, `onMounted`, `watch`, `definePageMeta`, `navigateTo`).
2. **Nuxt UI:** Preferir componentes do Nuxt UI (`UButton`, `UInput`, `USelect`, `UModal`, `UContainer`, etc.) em vez de HTML puro.
3. **Tailwind CSS:** Estilizar com classes Tailwind. Evitar CSS inline ou `<style>` quando possível.
4. **TypeScript:** Todo objeto (props, payload, estado, resposta) deve ter tipo/interface definido. Evitar `any`.
5. **Alertas:** Usar `swalSuccess`/`swalError`/`swalWarning` de `~~/utils/swalAlerts` para notificações.
6. **Firebase:** Toda interação com Firebase deve usar funções exportadas de `~~/services/firebase`.
7. **Pinia:** Estado global via stores em `store/`. Usar `persist: true` para persistência.
8. **Navegação:** Usar `navigateTo("/rota")` para navegação programática.
9. **Validação:** Validar campos obrigatórios ANTES de enviar dados. Se inválido, `swalError("Título", "mensagem")` e retornar.
10. **Idioma:** Mensagens ao usuário em português brasileiro.
11. **Sem APIs React/Next.js:** NÃO usar `useRouter` do `next/router`, `getServerSideProps`, `getStaticProps`, JSX, `useEffect`, `useState` do React. Este é um projeto Vue/Nuxt.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hionic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
