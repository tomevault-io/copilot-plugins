## mcmv-search-online-gemini

> Localizar, baixar e catalogar, na internet aberta, documentos públicos (editais, listas, atas, resultados, anexos) do programa **Minha Casa Minha Vida (MCMV) — exclusivamente Faixa 1** que contenham **listas nominais de candidatos inscritos, sorteados, suplentes ou em cadastro reserva** em processos de seleção habitacional por sorteio público.

# Agente de Busca e Coleta de Editais MCMV Faixa 1

## 1. Objetivo

Localizar, baixar e catalogar, na internet aberta, documentos públicos (editais, listas, atas, resultados, anexos) do programa **Minha Casa Minha Vida (MCMV) — exclusivamente Faixa 1** que contenham **listas nominais de candidatos inscritos, sorteados, suplentes ou em cadastro reserva** em processos de seleção habitacional por sorteio público.

**Período de interesse:** 2009 a 2020 (programa MCMV original, Faixa 1, antes da transição para Casa Verde e Amarela).

**Finalidade:** alimentar um pipeline de pesquisa acadêmica que explora a aleatorização via sorteio para inferência causal sobre efeitos do programa. A **completude da lista** (idealmente com CPF, NIS OU nome completo) é crítica.

**Escopo restrito a Faixa 1** (renda familiar até aproximadamente R$ 1.600 no desenho original do programa, ajustada ao longo dos anos). Faixa 1 é onde o sorteio é juridicamente obrigatório e onde há listas nominais públicas. **Faixas 1.5, 2 e 3 estão fora do escopo** — descartar documentos que sejam exclusivamente dessas faixas.

---

## 2. Estrutura de pastas

O agente deve usar e manter a seguinte estrutura. As pastas de dados de referência já existem com os arquivos colocados pelo pesquisador; as demais devem ser criadas pelo agente conforme necessário.

```
mcmv_editais_search/
├── GEMINI.md                          # este arquivo
├── README.md                          # instruções de uso e setup
├── .gitignore                         # regras de exclusão (apenas CÓDIGO e documentos chave no GitHub)
├── reference_data/                    # dados de referência (LOCAL apenas, ignore no git)
│   ├── dados_abertos_filtrado.csv
│   └── takeup_far_dec17.xlsx
├── catalog/                           # catálogo e relatórios (LOCAL apenas, ignore no git)
│   ├── catalog.jsonl
│   ├── training_findings.md           # Descobertas e dicas acumuladas durante o treino
│   └── ...
├── downloads/                         # arquivos baixados (LOCAL apenas, ignore no git)
└── src/                               # CÓDIGO FONTE (Sincronizado no GitHub)
```

**Convenções de nomenclatura:**
- `municipio_slug`: nome do município em minúsculas, sem acentos, espaços convertidos em `_` (ex.: `sao_jose_do_rio_preto`).
- `hash8`: primeiros 8 caracteres do SHA-256 do arquivo baixado.
- `tipo`: `inscritos`, `sorteados`, `suplentes`, `cadastro_reserva`, `ata`, `misto`.

---

## 3. Dados de referência fornecidos

### 3.1 `reference_data/dados_abertos_filtrado.csv`
Contém nome e data de aprovação dos empreendimentos alvo.

### 3.2 `reference_data/takeup_far_dec17.xlsx`
Contém lista de contratantes (CPF, NIS, Nome). Usado para busca reversa e validação.

---

## 4. Definições e escopo

**Documento de interesse:** Listas de inscritos, sorteados, suplentes, cadastro reserva, atas de sorteio e editais de convocação vinculados ao MCMV Faixa 1.

**FORA DE ESCOPO:** 
- Editais de chamamento de construtoras, licitações, cartilhas.
- **Dumps integrais de Diários Oficiais (DOM):** Este agente NÃO deve baixar edições completas de DOMs. Outros agentes realizam essa tarefa. Este agente deve focar em links diretos para editais e listas.

---

## 5. Estratégia de busca

### 5.1 Ordem de prioridade
1. Rodada 1: Cidades no `takeup_far_dec17.xlsx`.
2. Rodada 2: Cidades em `dados_abertos_filtrado.csv` mas fora do takeup.

### 5.2 Variantes terminológicas acumuladas (Dicas de Treino)
- **Novos termos:** "beneficiários habilitados", "famílias sorteadas", "beneficiários titulares", "cadastro de interessados em moradia".
- **Padrões de Instituição:** "Companhia de Habitação Popular de [Cidade]", "Secretaria Municipal de Habitação".

### 5.3 Busca reversa (§5.3 do original mantido)
Utilizar identificadores do takeup para validar documentos.

---

## 6. Heurísticas e armadilhas (Atualizado)

- **Sorocaba:** Busca dirigida por número de edição do Diário Oficial (ex: "Jornal Município de Sorocaba 1572").
- **Campinas:** Uso frequente de termos como "A Prefeitura Municipal de Campinas convoca" ou "divulga a relação".
- **Multi-arquivo:** Atenção para listas separadas por grupos (Geral, Idosos, Deficientes) e sub-listas (Titulares vs Suplentes).
- **Take-up incompleto:** Editais de "ausência/reconvocação" são fontes críticas para identificar quem NÃO assinou o contrato.

---

## 13. Fase de Treino e Integração GitHub

### 13.1 Replicação de Achados
O agente deve processar as cidades em `data_train/` com o objetivo de:
1. Localizar os mesmos URLs/arquivos encontrados manualmente.
2. Identificar arquivos adicionais que foram perdidos na coleta manual.
3. Extrair padrões de busca que funcionaram e registrá-los em `catalog/training_findings.md`.

### 13.2 Integração GitHub
- **Repositório:** `mcmv_search_online_gemini`
- **Conteúdo:** Apenas CÓDIGO (`src/`, `scripts/`), `GEMINI.md`, `README.md` e `.gitignore`.
- **Exclusão:** Dados de referência, downloads, tmp e catálogos JSONL devem ser ignorados (`.gitignore`).

### 13.3 Procedimentos Úteis
- Sempre checar o Wayback Machine para URLs de prefeituras que dão 404.
- Para cidades com Diários Oficiais indexados, buscar pelo nome do empreendimento dentro do domínio do diário.
- Documentar em `training_findings.md` cada novo padrão de URL descoberto (ex: WordPress uploads, sistemas customizados de COHABs).

---
> Source: [gavrochet/mcmv_search_online_gemini](https://github.com/gavrochet/mcmv_search_online_gemini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
