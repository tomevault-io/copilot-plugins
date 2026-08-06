## lecture-harvester

> Este documento serve como um guia de referência e regras de conduta para qualquer Agente de IA que atue neste repositório. Siga estas diretrizes estritamente.

# Diretrizes para Agentes de IA (AGENTS.md)

Este documento serve como um guia de referência e regras de conduta para qualquer Agente de IA que atue neste repositório. Siga estas diretrizes estritamente.

---

## 🚨 REGRA DE OURO: USO DE TOKENS E CHAMADAS DE IA

> [!IMPORTANT]
> **NUNCA execute chamadas de API de IA (OpenAI, Anthropic, Gemini) nem gaste tokens, a menos que o usuário peça explicitamente.**

* **Padrão Seguro:** Por padrão, o pipeline, enriquecimento de anexos e geração de markdown rodam **sem IA** (sem chamadas a APIs de LLMs).
  * Exemplo: `venv/bin/python main.py pipeline` (roda sem IA por padrão)
  * Exemplo: `venv/bin/python main.py markdown --all` (gera arquivos sem conteúdo de IA por padrão)
* **Não Executar Process / IA:** Nunca execute o subcomando `process` (`venv/bin/python main.py process --all`) ou use a flag `--use-ai` nos outros comandos a menos que o usuário peça com termos explícitos como *"processe com IA"*, *"gere os resumos da IA"*, ou *"enriqueça com IA"*.

---

## 📁 Estrutura de Diretórios do Projeto

* `config/`
  * `storage_state.json`: Estado de sessão do Playwright (cookies e localStorage). Se expirar, deve ser recriado executando o script de login interativo.
* `data/`
  * `raw/`: Cache local das aulas e materiais em formato JSON bruto.
  * `processed/`: JSONs processados pela IA (resumos executivos, conceitos, flashcards).
  * `markdown/`: Cache local dos arquivos Markdown gerados.
* `pipeline/`
  * `ai_summarizer.py`: Orquestrador de chamadas de LLM para gerar resumos estruturados e flashcards.
  * `clean_transcript.py`: Limpeza básica de transcrições antes do envio para a IA.
  * `cost_tracker.py` / `logs/cost_tracker.csv`: Monitoramento e auditoria de gastos com tokens de IA.
  * `enrich_attachments.py`: Downloads e resumos locais de Notion/GitHub/Drive.
  * `generate_markdown.py`: Construtor de arquivos Markdown no formato Obsidian.
  * `generate_module_index.py`: Atualiza os índices do Obsidian.
* `scraper/`
  * `login.py`: Script headed interativo para login inicial e salvamento dos cookies.
  * `crawl_course.py`: Sincronizador de índice do curso (`course_index.json`).
  * `extract_lesson.py`: Extrator individual de conteúdo de aulas (vídeos, resumos, anexos).
* `scratch/`: Scripts utilitários de diagnóstico, auditoria de notas e scripts de correção pontuais.

---

## ⚠️ Padrão de Nomenclatura e Tratamento de Acentos

> [!WARNING]
> **Nunca remova acentos de nomes de pastas e diretórios durante a extração e geração de arquivos.**

* **A Regra:** Todas as rotinas que criam ou buscam diretórios em `data/raw/`, `data/processed/` e no vault do Obsidian devem usar a mesma função `clean_filename` que preserva os acentos, apenas removendo caracteres inválidos de sistema operacional:
  ```python
  def clean_filename(name):
      import re
      return re.sub(r'[\\/*?:"<>|]', '', name).strip()
  ```
* **O Problema:** Caso use uma função que remova acentos (ex: substituindo `prático` por `prtico`), isso causará um descompasso com o índice do curso (`course_index.json`) e com o Obsidian, gerando diretórios duplicados e quebrando verificações de cache incremental.

---

## 🛠️ Comandos CLI Recorrentes

### Autenticação (Login)
Se a sessão expirar ou o arquivo `config/storage_state.json` não existir, execute o login em modo visual:
```bash
venv/bin/python scraper/login.py
```

### Sincronizar Estrutura do Curso
Para atualizar a listagem de aulas locais e gerar o `data/raw/course_index.json`:
```bash
venv/bin/python main.py sync
```

### Rodar o Pipeline (Seguro - Sem IA)
Para buscar novas aulas e gerar as notas sem gastar tokens:
```bash
venv/bin/python main.py pipeline
```

### Regerar Índices do Obsidian
Para atualizar a organização e o arquivo `00 - Índice Geral.md`:
```bash
venv/bin/python main.py index
```

### Reprocessar Notas Incompletas (Varredura do Vault)
Para varrer o vault do Obsidian buscando notas sem resumo ou transcrição, extrair os dados ausentes da plataforma e regenerar as notas (respeitando a regra de não consumir tokens de IA por padrão):
```bash
venv/bin/python main.py reprocess
```
* Adicione `--use-ai` apenas se explicitamente instruído a re-enriquecer as notas com a IA após obter os dados.

---
> Source: [leandromeira/lecture-harvester](https://github.com/leandromeira/lecture-harvester) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
