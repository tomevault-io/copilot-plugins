## imersao-jornada-databricks

> Contexto do projeto para o Claude Code. Leia também `aula-03-claude-code/PRD.md` antes de qualquer mudança no pipeline.

# CLAUDE.md

Contexto do projeto para o Claude Code. Leia também `aula-03-claude-code/PRD.md` antes de qualquer mudança no pipeline.

## O projeto

Imersão Jornada de Dados: e-commerce brasileiro no Databricks Free Edition, em 4 aulas. Pipeline medalhão no catálogo `ecommerce` (schemas `bronze`, `silver` e `gold`; os arquivos brutos ficam no volume `ecommerce.bronze.arquivos`), dashboards AI/BI e um Genie space. Tudo é implantado como Declarative Automation Bundle (`databricks.yml`).

## Estrutura

- `aula-01-sql-dashboard/`: notebooks da Aula 1 (os CSVs sobem pela interface direto em `ecommerce.bronze.*`) e dashboard sobre a bronze. A partir da Aula 2, o Job sobrescreve essas tabelas bronze, agora com metadados de ingestão.
- `aula-02-python-engenharia/`: `01_ingestao_bronze.py` → `02_silver.py` → `03_gold.sql`.
- `aula-03-claude-code/`: PRD, testes de qualidade (`testes/04_testes_qualidade.py`) e dashboard sobre `gold`.
- `aula-04-genie/`: comentários das tabelas gold e o Genie space (`genie/diretoria_ecommerce.geniespace.json`).
- `resources/`: Job, dashboards e Genie em YAML. Caminhos relativos começam com `../`.
- `dados/`: CSV e Parquet de origem.

## Comandos

Sempre passe o perfil da CLI (`-p <perfil>`); não assuma um perfil padrão.

```bash
databricks bundle validate --strict -t dev -p <perfil>
databricks bundle deploy -t dev -p <perfil>
databricks bundle run pipeline_ecommerce -t dev -p <perfil>
databricks experimental aitools tools query -p <perfil> -- "SELECT ..."
```

`dev` prefixa os recursos com `[dev usuario]` e pausa o agendamento. Faça deploy em `prod` só quando o usuário pedir.

## Convenções

- Nomes de tabelas e colunas em português, `snake_case`, sem acento.
- Notebooks no formato *source*: primeira linha `# Databricks notebook source` (Python) ou `-- Databricks notebook source` (SQL); células separadas por `COMMAND ----------`; texto em células `MAGIC %md`, em português e explicando o porquê.
- Python lê o catálogo do widget `catalogo`; notebooks SQL usam `USE CATALOG IDENTIFIER(:catalogo)` depois de criar o widget numa célula `%python`.
- Silver em PySpark, gold em SQL (ver justificativa no README da Aula 2).
- Gold é recriada com `CREATE OR REPLACE`, o que apaga comentários. Por isso o Job reaplica os comentários na última tarefa: coluna nova na gold exige comentário novo em `aula-04-genie/01_preparar_dados_para_ia.sql`.
- Toda tabela gold nova precisa de teste em `testes/04_testes_qualidade.py`.
- Na Free Edition, `CREATE CATALOG` funciona por SQL, mas não pela API REST (falta storage root). Não troque o SQL por chamada de API.

## Números de referência (para validar mudanças)

- Receita total: R$ 974.077,28 (3.020 vendas); deve ser igual na silver e nas golds de vendas e clientes.
- 20 vendas de produtos não cadastrados.
- Segmentos: 10 VIP, 25 TOP_TIER e 15 REGULAR.
- 35 produtos mais caros que todos os concorrentes.

---
> Source: [lvgalvao/Imersao-Jornada-Databricks](https://github.com/lvgalvao/Imersao-Jornada-Databricks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
