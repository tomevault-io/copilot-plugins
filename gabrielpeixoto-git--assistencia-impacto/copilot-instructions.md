## pythonstandardsmd

> <estrutura_de_arquivos>


# Padrões Python do Projeto

<estrutura_de_arquivos>
- Organize em: src/, tests/, docs/, scripts/
- Módulos: nomes em snake_case, sem abreviações confusas
- Classes: PascalCase | Funções/variáveis: snake_case | Constantes: UPPER_SNAKE_CASE
</estrutura_de_arquivos>

<tratamento_de_erros>
- Use exceções específicas — nunca `except Exception` sem log
- Crie exceções customizadas herdando de uma base do projeto: `class ProjectError(Exception)`
- Sempre feche recursos com context managers (`with`)
- Nunca silencie exceções com `pass` — ao menos logue
</tratamento_de_erros>

<testes>
- Use pytest com fixtures para setup/teardown
- Nome dos testes: `test_<o_que_testa>_<condicao>_<resultado_esperado>`
- Mocks apenas para dependências externas (HTTP, DB, filesystem)
- Cobertura mínima: 80% para código de produção
- Arquivo de teste espelha a estrutura: `src/utils/parser.py` → `tests/utils/test_parser.py`
</testes>

<commits>
- Formato Conventional Commits: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`
- Mensagem em inglês, imperativo: "Add validation to user input"
- Um commit por mudança lógica coesa
</commits>

---
> Source: [gabrielpeixoto-git/assistencia-impacto](https://github.com/gabrielpeixoto-git/assistencia-impacto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
