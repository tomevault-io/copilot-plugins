## pipeline-anvisa

> Este projeto contem dois pipelines integrados:

# AGENTS.md

## Objetivo do repositorio
Este projeto contem dois pipelines integrados:
1. Pipeline ANVISA (coleta, consolidacao e engenharia da base CMED).
2. Pipeline NFe (22 etapas de limpeza, matching, enriquecimento e saida para BI).

Ao portar logica de modulos Python para notebooks, preserve o comportamento original e a ordem de execucao do pipeline.

## Estrutura importante
- `pipelines/anvisa_base/`: pipeline ANVISA.
- `pipelines/anvisa_base/workflows/`: implementacao canonica das etapas 1.0 e 1.5 (download + consolidacao + engenharia inicial).
- `pipelines/anvisa_base/src/`: processamento avancado da base ANVISA e utilitarios.
- `pipelines/anvisa_base/src/modules/`: modulos-fonte com regras de negocio da base ANVISA (fonte principal para portabilidade).
- `pipelines/anvisa_base/scripts/`: wrappers/entradas de compatibilidade para etapas do pipeline ANVISA.
- `pipelines/anvisa_base/config_anvisa.py`: parametros de periodo, paralelismo e caminhos da base ANVISA.

- `pipelines/nfe/`: pipeline NFe.
- `pipelines/nfe/main.py`: orquestrador principal do pipeline NFe.
- `pipelines/nfe/src/`: etapas `nfe_etapa01` ate `nfe_etapa22` e utilitarios de caminho.
- `pipelines/nfe/scripts/`: scripts de execucao e validacao por etapa.
- `pipelines/nfe/support/`: arquivos de apoio de regras de negocio (json/csv/xlsx).
- `support/`: insumos compartilhados usados por etapas do NFe.

- `pipelines/common/`: utilitarios comuns entre pipelines (ex.: IO e validacoes).

- `data/raw/`: entradas brutas (downloads ANVISA e arquivos de origem).
- `data/processed/`: artefatos intermediarios das etapas.
- `data/external/`: artefatos auxiliares gerados/consumidos por etapas.
- `output/anvisa/`: saidas finais da base ANVISA.
- `QlikView/`: saidas finais consumidas pelo BI.
- `nfe/`: entrada principal da NFe (`nfe.csv`).

- `tests/unit/`: testes unitarios.
- `tests/smoke/`: testes de fumaca.
- `tests/manual/`: scripts de analise e diagnostico manual.

- `1_download_anvisa.py`, `2_processar_base_anvisa.py`, `2b_processar_dados_anvisa.py`, `3_pipeline_nfe.py`: wrappers de execucao na raiz.
- `pipeline_config.json` e `pipeline_config.py`: toggles de configuracao do pipeline.
- Notebooks atuais ficam na raiz (ex.: `1_download_anvisa.ipynb`, `2_processar_base_anvisa.ipynb`), nao em `notebooks/`.

## Regras obrigatorias
1. Antes de editar, estudar o modulo alvo e seus imports locais.
2. Nao transpor tudo de uma vez. Trabalhar em blocos pequenos.
3. Apos cada bloco, verificar equivalencia logica com o codigo original.
4. Nao inventar comportamento ausente no modulo-fonte.
5. Nao simplificar regra de negocio sem justificar.
6. Se houver ambiguidade, apontar explicitamente a duvida.
7. Se encontrar bug pre-existente, sinalizar separadamente.

## Modo de execucao obrigatorio
1. Primeiro, apenas ler o codigo e montar um plano antes de qualquer edicao.
2. No plano inicial, identificar obrigatoriamente:
   - funcoes e classes relevantes,
   - dependencias internas diretas e indiretas,
   - entradas e saidas de cada etapa,
   - efeitos colaterais,
   - pontos ambiguos ou arriscados.
3. Implementar a transposicao em blocos pequenos e logicos (nunca tudo de uma vez).
4. Ao final de cada bloco implementado, obrigatoriamente:
   - conferir equivalencia logica com o original,
   - checar divergencias de nomes, parametros, tipos e ordem de execucao,
   - revisar imports, variaveis de apoio e dependencias ocultas,
   - rodar validacoes possiveis,
   - explicar o que foi transposto e o que ainda falta.

## Restricoes adicionais
1. Preservar a logica original ao maximo.
2. Manter mudancas localizadas e rastreaveis.
3. Nao criar funcoes ficticias nem variaveis sem origem clara.
4. Nao assumir que o notebook ja possui contexto suficiente.
5. Sempre conferir se o trecho transposto depende de algo definido em outro modulo.
6. Se houver adaptacao ao formato notebook, documentar explicitamente a adaptacao.
7. REGRA CRITICA DE ENCODING: nunca introduzir corrupcao de caracteres (mojibake) em comentarios, strings, logs, docs e codigo.
8. Sempre salvar arquivos de texto em UTF-8 (preferencialmente sem BOM) e manter consistencia com o encoding ja usado no projeto.
9. Nunca converter acentuacao valida para sequencias corrompidas como `Módulo`, `descrição`, `Início` ou similares.
10. Nunca inserir ou manter caracteres de controle invisiveis em arquivos fonte (ex.: faixa C1, bytes quebrados de copy/paste).
11. Se detectar corrupcao de encoding pre-existente, corrigir de forma localizada e sinalizar claramente no resumo final.

## Validacao
1. Conferir imports e dependencias ocultas.
2. Conferir entradas e saidas de cada funcao ou etapa.
3. Conferir ordem de execucao.
4. Conferir nomes de colunas, parametros e variaveis criticas.
5. Rodar validacoes possiveis apos cada bloco.
6. Revisar o diff final e procurar regressos.
7. Revisar textos alterados procurando mojibake e caracteres estranhos antes de finalizar.
8. Quando editar arquivos `.py`, garantir que o arquivo compila apos mudancas (ex.: `python -m py_compile` ou equivalente).
9. Nao finalizar tarefa com warnings/erros de encoding pendentes.

## Formato esperado das respostas
1. Resumir o que o trecho original faz.
2. Listar dependencias necessarias.
3. Mostrar a transposicao feita.
4. Informar o que foi adaptado ao notebook.
5. Listar riscos e duvidas restantes.

## Done when
A tarefa so termina quando:
1. a logica relevante do modulo foi portada,
2. a sequencia do pipeline ficou coerente,
3. imports e dependencias foram resolvidos,
4. houve revisao final critica,
5. ficaram claros os pontos que ainda exigem validacao manual.
6. foi apresentado resumo final contendo:
   - o que foi transposto,
   - o que foi adaptado,
   - o que precisa de validacao manual,
   - possiveis divergencias em relacao ao modulo original.
7. foi realizada auto-revisao critica do notebook como auditoria final.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/luci18530)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/luci18530)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
