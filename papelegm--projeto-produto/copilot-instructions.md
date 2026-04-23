## projeto-produto

> Automação de orçamentos do sistema **Caixa Escolar MG** (portal de compras das caixas escolares de Minas Gerais).

# Projeto CE

## Descrição

Automação de orçamentos do sistema **Caixa Escolar MG** (portal de compras das caixas escolares de Minas Gerais).

O projeto realiza três operações principais:
1. **Coleta de orçamentos** — consulta a API do Caixa Escolar para 67 municípios e grava os orçamentos do dia em uma planilha Google Sheets
2. **Coleta de itens** — lê os IDs da aba `resumo` e busca os itens detalhados de cada orçamento, gravando na aba `ITENS`
3. **Envio de propostas** — acessa o portal web e preenche/envia as propostas de preço para os orçamentos selecionados

A planilha Google Sheets serve como banco de dados intermediário, gerenciada via Google Apps Script (Web App).

---

## Estrutura do Projeto

```
Projeto CE/
├── resumohoje.spec.js        # Busca orçamentos via API e atualiza planilha (67 municípios)
├── coletar-itens.spec.js     # Busca itens dos orçamentos via API → aba ITENS
├── Enviar-sem-api.spec.js    # Envia propostas via interface web (sem API)
├── apps-script.gs            # Backend Google Apps Script da planilha
├── GitHubDisparo.gs          # Script GAS para disparar workflows do GitHub Actions
├── municipios.js             # Lista de municípios de MG com códigos
├── playwright.config.js      # Config do Playwright (1 worker, Chromium)
├── package.json              # Dependências Node.js
└── .github/
    └── workflows/
        └── playwright.yml    # CI/CD GitHub Actions
```

### Abas da planilha Google Sheets

| Aba | Conteúdo |
|-----|----------|
| `resumo` | Orçamentos coletados (cabeçalho com camelCase) |
| `ITENS` | Itens detalhados de cada orçamento |
| `PENDENTE ENVIO` | Orçamentos aguardando envio de proposta |
| `ENVIADO` | Orçamentos com proposta já enviada |

---

## Como Executar

### Pré-requisitos
```bash
npm install
npx playwright install chromium
```

### Rodar os scripts individualmente
```bash
npx playwright test resumohoje.spec.js
npx playwright test coletar-itens.spec.js
npx playwright test Enviar-sem-api.spec.js
```

### CI/CD
Os testes são disparados automaticamente via **GitHub Actions** a cada push/PR no branch `master` ou `main`.
O `GitHubDisparo.gs` também permite disparar o workflow manualmente a partir do Google Apps Script.

---

## Tecnologias

- **Node.js** + **Playwright** — automação web e requisições HTTP à API
- **Google Apps Script** — Web App que faz CRUD na planilha Google Sheets
- **Google Sheets** — banco de dados intermediário
- **GitHub Actions** — execução automatizada dos scripts em ambiente CI

---

## URLs e Endpoints

| Recurso | URL |
|---------|-----|
| Portal web | `https://caixaescolar.educacao.mg.gov.br` |
| API REST | `https://api.caixaescolar.educacao.mg.gov.br` |
| Web App GAS | `https://script.google.com/macros/s/AKfycbx.../exec` |

---

## Regras de Negócio

### Grupos bloqueados (não participar)
- Gás recarga
- Serviços Operacionais Contínuos
- Gêneros Alimentícios

### Status de fornecedor (API)
| Código | Significado |
|--------|-------------|
| `APRO` | Aprovada |
| `RECU` | Recusada |
| `ENVI` | Enviada |
| `NAEN` | Não Enviada |
| `FORA` | Prazo Encerrado |
| `CANC` | Cancelado |

---

## Notas

- O Playwright roda com **1 worker** e browser **Chromium** (sem paralelismo entre testes)
- Timeout global dos testes: **6.000.000 ms** (100 minutos) — necessário pela quantidade de municípios
- Datas da API vêm em ISO UTC; usar `parseDateLocal()` para evitar bug de fuso horário (UTC-3 Brasil)
- O campo `participar` na aba `resumo` é um checkbox booleano que controla se o script deve enviar proposta
- Melhorias de performance identificadas estão documentadas em `sugestoes-melhorias.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/papelegm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
