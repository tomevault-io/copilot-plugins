## sistema-marmelstein

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Contexto do Projeto

**Sistema Marmelstein** é um framework para construção de pipelines agênticos determinísticos voltados ao domínio jurídico. Combina servidores MCP de jurisprudência, skills de processamento de documentos e orquestradores para análise de processos.

**Autor:** George Marmelstein
**Licença:** Todos os direitos reservados

---

## Comandos de Desenvolvimento

### Instalação

```bash
pip install -r requirements.txt
```

### Testar MCPs

```bash
# Testar BNP API (Precedentes STF/STJ)
python .claude/mcp-servers/bnp-api/server.py

# Testar CJF Jurisprudência
python .claude/mcp-servers/cjf-jurisprudencia/server.py

# Testar JurisDF TJDFT
python .claude/mcp-servers/jurisdf-tjdft/server.py
```

### Verificar dependências

```bash
python -c "import mcp; import requests; import pdfplumber; print('OK')"
tesseract --version
tesseract --list-langs  # deve incluir 'por'
```

---

## Arquitetura

### Estrutura de Diretórios

```
sistema-marmelstein/
├── .claude/
│   ├── commands/           # Orquestradores (/pipeline-sentenca, /criar-*)
│   ├── agents/             # Subagentes por categoria (analise, extracao, revisao)
│   ├── skills/             # Skills com scripts Python
│   ├── mcp-servers/        # Servidores MCP de jurisprudência
│   │   ├── shared/         # Código compartilhado entre MCPs
│   │   ├── bnp-api/        # BNP (Precedentes STF/STJ)
│   │   ├── cjf-jurisprudencia/
│   │   └── jurisdf-tjdft/
│   └── spec/               # Framework de especificações
├── docs/                   # 11 guias consolidados (vocabulário → implementação)
├── slides/                 # Apresentações standalone por aula
└── requirements.txt
```

### Padrão Orquestrador Cego

O sistema usa arquitetura de duas camadas:

1. **Camada de Instruções** (`.claude/agents/*.md`): Define capacidades reutilizáveis
2. **Camada de Dados** (`workspace/`): Contém arquivos específicos de cada processo

O orquestrador **coordena** sem executar: calcula contexto, injeta caminhos e valida outputs. Subagentes recebem caminhos prontos, não variáveis.

### Servidores MCP

| MCP | Fonte | Sintaxe de Busca |
|-----|-------|------------------|
| `bnp-api` | Banco Nacional de Precedentes | `+termo -termo "frase exata"` |
| `cjf-jurisprudencia` | Portal CJF (STF, STJ, TRFs) | `termo E termo`, `termo OU termo`, `ADJ`, `PROX` |
| `jurisdf-tjdft` | JurisDF TJDFT | Texto livre ou filtros |

---

## Commands Disponíveis

| Comando | Descrição |
|---------|-----------|
| `/pipeline-sentenca <caminho>` | Pipeline completo de análise de sentença |
| `/criar-agente <nome>` | Cria novo agente |
| `/criar-orquestrador <nome>` | Cria novo orquestrador |
| `/criar-skill <nome>` | Cria nova skill |

---

## Skills Principais

### converter-pdf

Converte PDFs judiciais para TXT via OCR.

```bash
python .claude/skills/converter-pdf/scripts/pdf_para_txt.py \
  --input documento.pdf \
  --output ./textos
```

Flags: `--digital` (pdfplumber, mais rápido), `--limite N`

### pje-download

Baixa processos do PJE TRF5 via API REST.

```bash
# Extrair cookies de HAR
python .claude/skills/pje-download/scripts/extrair_cookies_har.py \
  --har sessao.har --output pje_session.json

# Listar processos
python .claude/skills/pje-download/scripts/listar_processos.py \
  --cookies pje_session.json --modo sentenca --limite 5

# Baixar PDFs
python .claude/skills/pje-download/scripts/baixar_pdfs.py \
  --cookies pje_session.json --processos processos.json --output data/
```

---

## Convenções

### Idioma
- **Documentação:** Português brasileiro COM acentos
- **Código:** Inglês (padrão internacional)

### Nomenclatura

| Tipo | Convenção | Exemplo |
|------|-----------|---------|
| Pasta | kebab-case | `mcp-servers/` |
| Arquivo Python | snake_case | `pdf_para_txt.py` |
| Arquivo Markdown | kebab-case | `pipeline-sentenca.md` |

### Output de Scripts

Formato padrão para economizar contexto:

```
[INICIO] X items -> destino
[OK] item1: resultado
[ERRO] item2: motivo
[FIM] X/Y OK
```

---

## Dependências de Sistema

### OCR (para converter-pdf)

- **Tesseract OCR** com pacote de idioma português
- **Poppler** (para pdf2image)

Windows: Adicionar ao PATH após instalação.

---

## Documentação

A pasta `docs/` contém 11 guias organizados em progressão:

1. **Vocabulário** (01-02): Glossários de termos
2. **Design** (03-04): Arquitetura e estrutura de projeto
3. **Artefatos** (05-07): Commands, Skills, Agents, Autonomia
4. **Arquitetura** (08): Padrão Orchestrator-Workers
5. **Implementação** (09-11): Prompts, Workflows, Pipelines

---

## Filosofia Central

> *"Pipelines trocam flexibilidade por previsibilidade."*

- **Aumentação, não Automação**: IA potencializa o humano
- **Filesystem-First**: Dados em hierarquia de diretórios, contexto sob demanda
- **Orquestrador Cego**: Coordena sem executar, valida via sinalizadores

---
> Source: [georgemarmelstein/sistema-marmelstein](https://github.com/georgemarmelstein/sistema-marmelstein) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
