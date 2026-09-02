## orcaos

> Use these instructions whenever working on this repository.

# Copilot instructions — AFERIX

Use these instructions whenever working on this repository.

## 1. Visão do Produto
* **Produto**: Aferix.
* **Foco**: Orçamento, gestão financeira e operação para prestadores de serviço / autônomos.
* **Identidade Visual**: Dark Premium com amarelo/dourado como accent principal. Tema claro não existe. Sem teal/cyan como destaque.

## 2. Arquitetura Oficial
* **Fluxo Oficial**: UI → Hooks → Services → Repositories → Dexie (IndexedDB).
* **SSOT (Single Source of Truth)**: Dexie é a única fonte de verdade para orçamentos, clientes e ordens de serviço.
* **React e Storage**: O React **nunca** acessa o banco de dados/storage diretamente. Utilize Hooks que chamam Services.

## 3. Persistência e Migração
* **LocalStorage**: Considerado LEGADO. Utilizado apenas para migração (via `LegacyBudgetMigrationService.ts`) ou para pequenos estados não críticos (configurações de UI).
* **Proibido**: Escrita de orçamentos no localStorage. Bridges bidirecionais entre Dexie e LocalStorage. Sincronização silenciosa "fire-and-forget".

## 4. Componentes e Páginas Oficiais
* `src/pages/BudgetForm.tsx`: Formulário oficial de criação e edição de orçamentos.
* `src/pages/BudgetHistoryPage.tsx`: Lista e histórico oficial de orçamentos.
* `src/app/App.tsx`: Ponto de entrada principal do aplicativo.

## 5. Regras de Desenvolvimento
* **Língua**: Código em Inglês, interface em Português do Brasil (PT-BR).
* **Tipagem**: TypeScript estrito. Evite `any`.
* **Mobile-first**: O design deve ser focado em dispositivos móveis.
* **Simplicidade**: Evite overengineering (CQRS, DI complexa, factories gigantes).

## 6. Termos e Contextos Proibidos (Resíduos)
* **Nomes antigos**: OrcaOS, OrçaOS, ORSLS, WarSLS.
* **Contextos fora do ERP financeiro**: ENDAP, CLP, EasyCLP, bobinagem, calculadora técnica, automação residencial.
* **Cores antigas**: teal, cyan, verde neon, amber (como cor principal).

---
> Source: [mateusrossi-jpg/OrcaOS](https://github.com/mateusrossi-jpg/OrcaOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
