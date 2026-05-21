## project-management-events

> Quando gerenciar tarefas, documentar resultados ou organizar a estrutura do projeto

# Gerenciamento de Projeto (.fcai)

> **Nota**: Esta regra fornece orientações sobre quando e como gerenciar tarefas, documentar resultados e organizar a estrutura do projeto. A documentação completa está disponível em `.fcai/project/documentation/` e o estado atual do projeto em [state.md](mdc:.fcai/state.md).

## Quando iniciar uma nova tarefa
- Verifique a estrutura atual do projeto em [README.md](mdc:.fcai/README.md)
- Identifique o componente relacionado à tarefa
- Crie a tarefa na pasta apropriada seguindo as convenções do projeto
- **Atualize o arquivo [state.md](mdc:.fcai/state.md) para incluir a nova tarefa**
- Para detalhes completos, consulte [command-implementation.md](mdc:.fcai/project/documentation/command-implementation.md)

## Quando documentar resultados de uma tarefa
- Crie um arquivo de resultados com o sufixo `_result`
- Inclua resumo, desafios, soluções, resultados de testes e próximos passos
- Marque as tarefas como concluídas no arquivo original
- Para detalhes completos, consulte [command-implementation.md](mdc:.fcai/project/documentation/command-implementation.md)

## Quando concluir uma tarefa
- Mova a tarefa para a pasta `completed/`
- **Sempre valide a tarefa de forma prática antes de marcá-la como concluída**:
  - Execute todos os testes automatizados relacionados
  - Verifique o funcionamento no container
  - Confirme que todos os critérios de aceitação foram atendidos
- **Atualize o arquivo [state.md](mdc:.fcai/state.md)** para refletir a conclusão
- Para detalhes completos, consulte [command-implementation.md](mdc:.fcai/project/documentation/command-implementation.md)

## Quando fazer ajustes em tarefas já finalizadas
- Atualize o arquivo de resultados correspondente
- Documente claramente quais ajustes foram feitos e por quê
- **Atualize o arquivo [state.md](mdc:.fcai/state.md)** se os ajustes alterarem significativamente o estado do projeto

## Quando documentar aspectos do projeto
- Use as pastas apropriadas em `.fcai/project/` para diferentes tipos de documentação
- Nomeie os arquivos de forma descritiva, usando kebab-case
- Mantenha a documentação atualizada conforme o projeto evolui

## Quando revisar o progresso do projeto
- Consulte as pastas de tarefas concluídas, em andamento e backlog
- Use a documentação em `.fcai/project/` para entender o contexto geral
- **Atualize o arquivo [state.md](mdc:.fcai/state.md)** após a revisão
- Use os comandos do projeto para facilitar a visualização do estado

## Quando atualizar o estado do projeto
- **Sempre atualize o arquivo [state.md](mdc:.fcai/state.md)** quando houver mudanças significativas
- O arquivo deve conter informações sobre componentes, tarefas e estrutura
- Use os comandos do projeto para facilitar a atualização e visualização do estado
- Para detalhes completos, consulte [command-implementation.md](mdc:.fcai/project/documentation/command-implementation.md)

## Quando trabalhar com containers
- Utilize preferencialmente Docker neste projeto
- Todos os comandos devem ser executados dentro dos containers utilizando docker compose exec <service> <command>
- Para mais detalhes, consulte [container-events.mdc](mdc:.cursor/rules/container-events.mdc)

## Quando trabalhar com a arquitetura do projeto
- Siga os princípios de Domain Driven Design e Clean Architecture
- Mantenha o modelo de domínio separado das entidades do banco de dados
- Para mais detalhes, consulte [architecture-events.mdc](mdc:.cursor/rules/architecture-events.mdc)

## Referências

Para informações detalhadas, consulte:

- **Estrutura do projeto**: [README.md](mdc:.fcai/README.md)
- **Estado atual**: [state.md](mdc:.fcai/state.md)
- **Sistema de comandos**: `.fcai/commands.md` e `.fcai/project/documentation/command-implementation.md`
- **Mapeamento de migração**: `.fcai/project/documentation/migracao.md`
- **Regras do sistema de comandos**: `.cursor/rules/command-system-events.mdc` 

---
> Source: [devfullcycle/golangtechweek](https://github.com/devfullcycle/golangtechweek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
