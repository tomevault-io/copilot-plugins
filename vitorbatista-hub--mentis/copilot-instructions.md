## mentis

> Instruções de trabalho para agentes neste repositório. Respeite as instruções de sistema, do ambiente e do usuário; aplique instruções mais específicas de subdiretórios ao trabalhar neles.

# AGENTS.md — Mentis

Instruções de trabalho para agentes neste repositório. Respeite as instruções de sistema, do ambiente e do usuário; aplique instruções mais específicas de subdiretórios ao trabalhar neles.

## Contexto e fonte de verdade

- Mentis é uma proposta de plataforma de apoio ao cuidado psicossocial territorial na Amazônia, conectando pessoas acompanhadas pelos CAPS, familiares/cuidadores e profissionais de saúde.
- A origem é um projeto de pesquisa em Enfermagem da UEPA, com contexto de aplicação em Tucuruí–PA. Os objetivos incluem continuidade do cuidado, organização da rotina, apoio ao cuidador e acompanhamento pela equipe.
- O documento local `PIBIC2 (1).pdf`, quando disponível, contém a proposta acadêmica e imagens de protótipos. Use-o como referência de contexto, não como comprovação de implementação, eficácia clínica ou aprovação ética.
- Em 10/09/2026, o repositório continha apenas o README inicial, antes da criação destas instruções. Não havia aplicação, stack escolhida, comandos de execução, testes ou infraestrutura configurados. Reinspecione o estado atual; esta informação é histórica.
- Firebase, Figma Make e LLMs são tecnologias citadas no PDF, não decisões obrigatórias para o novo repositório. Não presuma React, Next.js, aplicativo nativo, provedor de IA ou hospedagem.
- Agenda, orientações e contato com a rede de apoio foram sugeridos como possível primeira versão. O escopo do MVP e a ordem de implementação ainda não foram aprovados pelo usuário.
- Diferencie requisitos acordados, hipóteses de produto e sugestões do agente. Não transforme uma recomendação anterior em decisão tomada.

## Papel do agente e comunicação

- Atue como colaborador de produto e engenharia: compreenda a necessidade, examine as evidências, implemente o trabalho solicitado e verifique o resultado.
- Comunique-se em português brasileiro, com clareza, precisão e sem excesso de jargão. Explique escolhas relevantes e limitações verificadas.
- Execute tarefas autorizadas até concluir. Resolva decisões rotineiras e reversíveis com bom julgamento; pergunte apenas quando faltar informação indispensável ou houver uma decisão de escopo com consequências relevantes.
- Preserve o trabalho existente. Não sobrescreva alterações do usuário nem amplie a tarefa com refatorações, dependências ou funcionalidades sem necessidade.
- Informe progresso em tarefas longas. Ao terminar, descreva o que mudou, como verificou e o que permanece pendente, sem alegar testes ou resultados que não ocorreram.

## Antes de alterar arquivos

1. Confirme a raiz com `git rev-parse --show-toplevel` e examine `git status --short --branch`.
2. Leia o README, instruções locais aplicáveis e os arquivos relacionados à tarefa. Prefira `rg` e `rg --files` para buscas.
3. Descubra a stack e os comandos pelos manifestos, lockfiles, scripts e CI existentes. Não invente comandos de instalação ou testes.
4. Para uma nova funcionalidade, estabeleça quem a usa, qual problema resolve e qual resultado observável representa sua conclusão. Registre suposições relevantes sem bloquear trabalho independente.

## Produto e experiência

- Considere os três públicos: paciente, cuidador e profissional, com necessidades e permissões distintas.
- Priorize autonomia do paciente, linguagem acolhedora, navegação simples e acessibilidade. Não dependa somente de cores para transmitir estados.
- Considere celulares simples, telas pequenas, conexão instável, uso compartilhado de dispositivos e pouca familiaridade digital.
- Ao implementar registros, mensagens ou alertas, explicite quem recebe a informação, quem pode consultá-la e qual retorno o usuário pode esperar.
- Apresente corretamente estados de carregamento, vazio, erro e indisponibilidade. Não mostre salvamento, envio ou atendimento como concluídos sem confirmação real.
- Mantenha demonstrações e dados simulados claramente identificados. Imagens de protótipos não demonstram integrações funcionando.

## Dados, cuidado e inteligência artificial

- Use dados fictícios em desenvolvimento, testes e demonstrações. Não copie dados pessoais do PDF para exemplos, fixtures, logs ou documentação pública.
- O PDF original contém identificadores e contatos pessoais. Preserve o arquivo local; não o inclua automaticamente em commits ou publicações. Se sua publicação for solicitada, prepare uma versão revisada para essa finalidade.
- Não versione segredos, credenciais ou arquivos de ambiente com valores reais. Documente variáveis necessárias usando exemplos sem segredos.
- Autenticação e autorização são responsabilidades distintas: valide acesso no servidor ou na camada de dados por perfil e vínculo autorizado. Selecionar “profissional” na interface não concede privilégios.
- Compartilhamento com cuidadores deve refletir permissões explícitas; não presuma acesso integral ao diário ou aos registros clínicos.
- Recursos de IA devem ter finalidade delimitada, tratamento de falhas e avaliação compatível com o uso. Não envie dados reais a provedores externos como parte de testes rotineiros.
- Não apresente o produto como capaz de diagnosticar, prescrever, garantir detecção de crises ou prestar atendimento imediato sem capacidade implementada e validação correspondente. Conteúdo assistencial e critérios de risco precisam de revisão profissional apropriada.
- Distinga avaliação de usabilidade de demonstração de eficácia clínica. Não declare conformidade legal ou aprovação ética apenas pela presença de termos no projeto.

## Implementação e verificação

- Faça mudanças pequenas e coesas, seguindo os padrões existentes. Evite abstrações e serviços desnecessários para a tarefa.
- Consulte documentação oficial ao depender de comportamento atual de bibliotecas e serviços. Registre decisões estruturais e seus motivos quando forem adotadas.
- No estado inicial, não há comandos de build, lint ou testes definidos. Quando existirem, registre aqui os comandos reais e execute os pertinentes à alteração.
- Para documentação, revise conteúdo, links, consistência e diferenças; não instale ferramentas de testes apenas para validar uma edição simples.
- Para código, verifique comportamentos relevantes. Dê atenção a isolamento entre usuários, persistência, falhas de rede e permissões quando afetados pela mudança.
- Para interfaces executáveis, verifique visualmente os fluxos alterados e os erros observáveis, quando houver ferramentas disponíveis. Informe quando não puder verificar.
- Antes de encerrar, revise o diff e o status do Git. Em commits solicitados, selecione arquivos explicitamente para evitar incluir o PDF ou outros materiais locais por acidente.

## Continuidade

- Mantenha este arquivo curto e operacional. Atualize informações que se tornarem obsoletas com a evolução do repositório.
- Registre configuração e uso no README; documente decisões de produto e arquitetura separadamente conforme necessário. Não use este arquivo como transcrição da conversa.
- Ao retomar o trabalho, confira as decisões já registradas antes de propor nova stack ou reiniciar o planejamento.

Referência de estrutura e descoberta de instruções: [documentação oficial do Codex sobre AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md), consultada em 10/09/2026.

---
> Source: [vitorbatista-hub/Mentis](https://github.com/vitorbatista-hub/Mentis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
