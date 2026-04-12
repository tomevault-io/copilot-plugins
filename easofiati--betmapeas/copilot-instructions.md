## betmapeas

> - **Backend:** sempre em **Python** (FastAPI por padrão)

# BetMapEAS Development Rules

## 1. Stack Tecnológica Imutável
- **Backend:** sempre em **Python** (FastAPI por padrão)
- **Banco de Dados:** **PostgreSQL** (versão 13+)
  - Scripts de migração gerenciados com Alembic
- **Infra:** cada camada em **containers** dedicados (Docker):
  1. `backend`
  2. `database`
  3. `frontend`
- Orquestração pelo **Docker Compose**
- Higiene de Imagens: sempre que uma nova imagem Docker for gerada e validada, remover a imagem anterior para evitar acúmulo de camadas obsoletas e consumo desnecessário de recursos.

## 2. Gerência de Configurações
- Qualquer dado sensível ou variável de ambiente **fica no arquivo** `.env`
- Nunca versionar `.env` em controle de versão; manter `.env.example` atualizado

## 3. Testes & Qualidade
1. **Testes unitários obrigatórios** para cada novo módulo
2. Só prosseguir se **100% dos testes atuais passam** (`pytest -q`)
3. Cobertura mínima de código: **90%** (cobertura falha quebra a *pipeline*)
4. Utilizar pre-commit hooks (black, isort, mypy) antes de qualquer *commit*

## 4. Fluxo de Tarefas e Git

| Ação | Regra |
|------|-------|
| **Conclusão de subtarefa** | 1. Executar testes<br>2. Commit se verde<br>3. Atualizar o **TaskMaster** |
| **Conclusão de tarefa** | 1. Garantir testes & lint OK<br>2. Fazer documentação<br>3. `git commit -m "feat: …"`<br>4. `git push` |
| **Branch naming** | `feat/<descr>`, `fix/<descr>`, `chore/<descr>` |
| **Commits** | Mensagens claras, em português, seguindo *Conventional Commits* |

## 5. Documentação
- **Centralizar** toda documentação na pasta `docs/` na raiz
- **Fazer documentação** apenas após testes serem realizados com sucesso
- Manter docs atualizadas a cada feature concluída

## 6. Mantras Rápidos
- "**Teste falhou → código falhou**" – corrija antes de seguir
- "**Container isolado → dependência controlada**"
- "**Atualize o TaskMaster → trace tudo**"
- "**Commit pequeno → revisão fácil**"
- "**Docs após testes → qualidade garantida**"

## Estrutura Mínima
```
projeto/
├── backend/        # Container Python
├── frontend/       # Container React
├── database/       # Container PostgreSQL
├── docs/           # Documentação centralizada
├── docker-compose.yml
├── .env
├── .env.example
└── taskmaster.md
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/easofiati)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/easofiati)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
