## natralha

> leia o cursorrules e outros arquivos obrigatórios listados. Ao terminar, não me dê um resumo deles. Apenas confirme que está ciente, que compreendeu e está preparado para acatar as diretrizes

leia o cursorrules e outros arquivos obrigatórios listados. Ao terminar, não me dê um resumo deles. Apenas confirme que está ciente, que compreendeu e está preparado para acatar as diretrizes

vamos trabalhar agora de maneira extremamente metódica, sistemática e organizada. 
Lento é rápido. Isso significa que ir com calma e realizar ações assertivas, 1 por vez, nos leva ao resultado mais rápido que centenas de erros. 
Quero tudo analisado e pensado, e decisões acertadas com 100% de certeza da implementação atingido antes do resultado final. 
## Orientações Obrigatórias
- As orientações são para serem seguidas de forma direta e especifica.
- Quando eu fizer perguntas, pedir para traçar um plano ou questionar como algo deve ser feito, responda as perguntas. Me de as informações para que eu tome as decisões. Se eu perguntei "como fazer isso?" não comece a fazer imediatamente por conta própria
- Mantenha a atenção para a diferença entre uma pergunta, um comando, uma solicitação, uma orientação, e aja de acordo que o que foi solicitado.
- Não quero a criação de processos, passos e modificações de arquitetura sem aprovação
- A semantica e a ordem das orientações é relevante e deverão ser consideradas na compreensão das orientações
- Informações não solicitadas não deverão ser inventadas ou criadas
- Alterações críticas e mudanças de formato não deverão ser realizadas sem aprovação
- O objetivo é criar um portal online, funcional, confiável, estável, com conexões funcionais
- O uso de informações de fallback em hardcode deve ser evitado ao máximo para não atrapalhar o desenvolvimento das apis
- Layouts deverão seguir o padrão e orientações no layoutguidelines
- Caso um estilo não esteja descrito nesse arquivo deverá ser questionado sem alterações improvisadas
- A identidade visual da marca é importante e as cores, logos e fontes deverão ser respeitados
- Sempre que eu solicitar um plano, uma análise ou um diagnóstico, entregue o resultado pedido e espere por confirmação antes de tomar a iniciativa de realizar alterações. tudo precisa de autorização. 
- Não solicitar logs do git. esses logs não apresentam informações relevantes para o desenvolvimento e, adicionalmente, sempre acabam provocando um loop no qual você retorna com o log para tudo que tenta fazer no console posteriormente. 
- Insira alguma forma de identificação clara nos aqrquivos que compoe indicando a versao atual de cada um e criando um rastro de versao para cada arquivo. lembre-se de incrementar a versao de cada arquivo cada vez que uma alteração for considerada de sucesso. Siga o molde vX.Y.Z, onde X é a versao maior, Y é a versao menor e Z é a versao de correção.
- Sempre que houver um deploy real para GCP ou push real para GitHub, atualize o DEPLOY_LOG.md com as informações precisas: data/hora, tipo, versão, arquivos modificados e descrição. Use os templates específicos para GCP e GitHub. Não registre configurações ou setups como deploys.
- não realizar deploy do projeto, ou push no GitHub sem solicitar confirmação dessa ação. 
- o nome Index.html deve ser evitado para que parem de ocorrer erros de integrações e redirecionamentos das paginas criadas. os nomes deverão refletir os trechos trabalhados.
- Sempre certifique-se de estar fazendo o deploy ou o push para o destino correto.Pushes do git jamais deverão ser trocados de reporitório acidentalmente e deploys no vercel ou no GCP deverão ocorrer com precisão.
- Se a solicitação feita para uma tarefa ser excutada depende de mais informações, de confirmações de dados ou mais recursos, solicite essas informações antes de iniciar a tarefa.Mantenha o compliance com a diretiva nro 4 e não crie aleatóriamente dados adicionais para completar as tarefas.
- Ao final da leitura das diretrizes, leia a lista de arquivos de referencia obrigatoria para manter a padronização e ter uma visão geral do projeto e das alterações realizadas.

## 📋 ARQUIVOS OBRIGATÓRIOS PARA LEITURA

### **Arquivos de Referência Obrigatória:**
1. **DEPLOY_LOG.md** - Histórico de deploys e alterações realizadas
2. **CLEANUP_REPO.md** - Instruções de limpeza e organização do repositório
3. **LAYOUT_GUIDELINES.md** - Guia completo de layout, cores, tipografia e padrões visuais
4. **listagem de schema de coleções do mongoD.rb** - Schemas das coleções MongoDB
5. **README.md** - Documentação principal do projeto
6. **Diretrizes especificas do projeto** - Diretrizes especificas do projeto

### **Arquivos de Padrão Visual:**
- **src/styles/theme.js** - Tema Material-UI
- **src/styles/globals.css** - Estilos globais
- **LAYOUT_GUIDELINES.md** - Padrões visuais e de layout

**IMPORTANTE:** Sempre consulte estes arquivos antes de fazer alterações para manter a consistência e padronização do projeto.

## 🚨 DIRETRIZES CRÍTICAS - COMANDOS GIT

### **PROIBIÇÕES ABSOLUTAS:**
- **NUNCA** execute `git reset --hard` sem confirmação explícita
- **NUNCA** execute `git reset` de qualquer tipo sem confirmação
- **NUNCA** execute comandos que possam destruir trabalho (reset, rebase, force push)
- **NUNCA** execute comandos git destrutivos quando solicitado para fazer push

### **COMANDOS GIT SEGUROS:**
- **APENAS** execute comandos git seguros: `git add`, `git commit`, `git push`, `git status`, `git log`
- **SEMPRE** confirme o comando exato antes de executar
- **SEMPRE** leia cuidadosamente o que foi solicitado vs o que será executado

### **PROTOCOLO DE SEGURANÇA:**
1. **ANTES** de executar qualquer comando git, confirme:
   - O comando exato solicitado
   - O comando que será executado
   - Se há risco de perda de dados
2. **SE HOUVER DÚVIDA**, pergunte antes de executar
3. **SE O COMANDO FOR DESTRUTIVO**, solicite confirmação explícita
4. **SE ERRO FOR COMETIDO**, pare imediatamente e reporte

### **EXEMPLOS DE ERROS CRÍTICOS:**
- ❌ Solicitação: "fazer push" → Execução: `git reset --hard HEAD~1`
- ❌ Solicitação: "atualizar repositório" → Execução: `git reset --hard`
- ❌ Solicitação: "limpar mudanças" → Execução: comandos destrutivos

### **EXEMPLOS DE EXECUÇÃO CORRETA:**
- ✅ Solicitação: "fazer push" → Execução: `git push origin main`
- ✅ Solicitação: "adicionar arquivos" → Execução: `git add .`
- ✅ Solicitação: "fazer commit" → Execução: `git commit -m "mensagem"`

# Diretrizes de Trabalho
<!-- VERSION: v1.3.0 | DATE: 2024-12-19 | AUTHOR:Lucas Gravina - VeloHub Development Team

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joaosilva-source) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
