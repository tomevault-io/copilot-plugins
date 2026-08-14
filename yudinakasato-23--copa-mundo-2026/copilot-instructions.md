## copa-mundo-2026

> Este projeto é um Simulador e Acompanhamento em tempo real para a Copa do Mundo FIFA 2026 (React + Vite + Tailwind CSS v4 + Supabase).

# CLAUDE.md — Diretrizes do Projeto do Simulador da Copa do Mundo 2026

## 🎯 Contexto do Projeto
Este projeto é um Simulador e Acompanhamento em tempo real para a Copa do Mundo FIFA 2026 (React + Vite + Tailwind CSS v4 + Supabase).
Toda a documentação estratégica e os Procedimentos Operacionais Padrão (POPs) detalhados estão centralizados no Obsidian Vault do usuário.

---

## 🗺️ Integração com Obsidian Vault
Ao iniciar uma nova sessão ou trabalhar neste projeto, consulte a nota e o POP principal no Obsidian:
* **POP do Projeto:** [POP_Simulador-Copa-Mundo-2026.md](file:///mnt/c/Users/linco/OneDrive/Documents/Obsidian/Yudi%20Nakasato/03-RECURSOS/POPs/POP_Simulador-Copa-Mundo-2026.md)
* **Diretório do Vault:** `/mnt/c/Users/linco/OneDrive/Documents/Obsidian/Yudi Nakasato`

---

## 🛠️ Comandos de Desenvolvimento

### Instalação & Execução
* Instalar dependências: `npm install`
* Executar em desenvolvimento: `npm run dev`
* Compilar para produção (Vite build): `npm run build`
* Verificar linter (ESLint): `npm run lint`

### Scripts de Sincronização & Automação (Local Scraper Agent)
* **Atualizar Placares & Estatísticas (Gols/Cartões):**
  ```bash
  node scripts/update-db-scores.js scratch/temp_scores.json
  ```
  *(Insere gols/cartões em jsonb, recalcula classificação e propaga o chaveamento do mata-mata)*
* **Sincronizar Metadados de Partida (Datas/Estádios/Fusos do wiki):**
  ```bash
  node scripts/update-db-metadata.js
  ```
* **Verificar Status de Jogos no Supabase:**
  ```bash
  node scratch/check-matches.js
  ```

---

## 📜 Regras de Commit & Deploy
1. **Conventional Commits:** Escreva mensagens claras (ex: `feat: ...`, `chore: ...`).
2. **E-mail do Committer (CRÍTICO):** Configure sempre o e-mail local do git como `linconyudi@gmail.com` antes de commitar para evitar bloqueios de deploy no Vercel Hobby.
   ```bash
   git config user.email "linconyudi@gmail.com"
   git config user.name "Yudi Nakasato"
   ```

---
> Source: [yudinakasato-23/copa-mundo-2026](https://github.com/yudinakasato-23/copa-mundo-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
