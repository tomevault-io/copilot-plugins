## site

> O repositório é composto por dois projetos principais:

# Guia de Estrutura do Projeto

O repositório é composto por dois projetos principais:

- [backend/](mdc:backend): API RESTful em Python (Flask), responsável por fornecer dados dinâmicos para o frontend. Veja detalhes em [backend/README.md](mdc:backend/README.md).
- [frontend/](mdc:frontend): Aplicação Next.js que consome a API e exibe as informações do portfólio. Veja detalhes em [frontend/README.md](mdc:frontend/README.md).

A orquestração dos serviços é feita pelo arquivo [docker-compose.yaml](mdc:docker-compose.yaml), que integra ambos os projetos utilizando os respectivos Dockerfiles.

O arquivo principal de documentação do repositório é o [README.md](mdc:README.md), que traz instruções gerais de execução, contexto e links para os READMEs específicos de cada projeto.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ivanildobarauna-dev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ivanildobarauna-dev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
