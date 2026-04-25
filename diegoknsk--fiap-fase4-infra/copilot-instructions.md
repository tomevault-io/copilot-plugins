## fiap-fase4-infra

> Este documento define o padrão completo para criação de histórias técnicas de infraestrutura no repositório.


# Regras para Criação de Histórias (Stories)

Este documento define o padrão completo para criação de histórias técnicas de infraestrutura no repositório.

## 📁 Estrutura de Pastas

### Nomenclatura da Pasta da História

**Formato obrigatório:**
```
Storie-XX-Descricao_Breve_Da_Historia
```

**Regras:**
- Sempre começar com `Storie-` seguido do número com 2 dígitos (01, 02, 03...)
- Número deve ser sequencial e único
- Descrição deve ser curta (máximo 5-6 palavras)
- Usar underscore (`_`) para separar palavras na descrição
- Descrição deve ser clara e permitir identificar o conteúdo apenas pelo nome da pasta
- Primeira letra de cada palavra em maiúscula (PascalCase para palavras, snake_case para separação)

**Exemplos corretos:**
- `Storie-01-Backend_Terraform_S3`
- `Storie-05-Modulo_Cluster_EKS`
- `Storie-12-Security_Groups_EKS`
- `Storie-15-Padronizacao_Variaveis_Ambientes`

**Exemplos incorretos:**
- `Storie-1-Backend` (número sem zero à esquerda)
- `Storie-01-backend-terraform` (usar hífen em vez de underscore)
- `Storie-01-BackendTerraformS3` (sem separação, difícil de ler)

### Estrutura Interna da Pasta

```
Storie-XX-Descricao/
├── story.md                    # Arquivo principal da história
└── subtask/                    # Pasta contendo todas as subtasks
    ├── Subtask-01-Nome.md
    ├── Subtask-02-Nome.md
    └── Subtask-03-Nome.md
```

**Regras:**
- Sempre ter um arquivo `story.md` na raiz da pasta
- Sempre ter uma pasta `subtask/` (mesmo que vazia inicialmente)
- Nenhum outro arquivo na raiz da pasta da história

## 📝 Formato do Arquivo story.md

### Estrutura Obrigatória

```markdown
# Storie-XX: Título da História

## Status: Em Andamento

## Descrição
Como [papel], quero [ação], para [benefício].

## Objetivo
[Descrição clara e objetiva do que será entregue]

## Escopo Técnico
- Tecnologias: [Lista de tecnologias]
- Arquivos afetados: [Lista de arquivos/caminhos]
- Recursos AWS: [Lista de recursos AWS que serão criados]

## Subtasks

- [ ] [Subtask 01: Nome da subtask](./subtask/Subtask-01-Nome_da_subtask.md)
- [ ] [Subtask 02: Nome da subtask](./subtask/Subtask-02-Nome_da_subtask.md)
- [x] [Subtask 03: Nome da subtask](./subtask/Subtask-03-Nome_da_subtask.md)

## Critérios de Aceite da História

- [ ] Critério 1 (específico e mensurável)
- [ ] Critério 2 (específico e mensurável)
- [ ] Critério 3 (específico e mensurável)
```

### Regras de Escrita do story.md

1. **Título:**
   - Formato: `# Storie-XX: Título da História`
   - Título deve ser claro e descritivo
   - Máximo 10 palavras

2. **Descrição:**
   - Usar formato "Como... quero... para..."
   - Ser específico sobre o papel (ex: "engenheiro de plataforma")
   - Ação deve ser clara e objetiva
   - Benefício deve ser mensurável ou observável

3. **Objetivo:**
   - Uma única frase ou parágrafo curto
   - Deve explicar O QUE será entregue, não COMO
   - Focar no resultado final

4. **Escopo Técnico:**
   - Listar tecnologias específicas (ex: "Terraform, AWS EKS")
   - Listar arquivos/caminhos que serão criados/modificados
   - Listar recursos AWS que serão provisionados
   - Ser específico, não genérico

5. **Subtasks:**
   - Mínimo 3 subtasks, máximo 8 subtasks
   - Cada subtask deve ser uma ação específica e testável
   - Ordem deve fazer sentido lógico (dependências)
   - Links devem usar caminho relativo: `./subtask/Subtask-XX-Nome.md`
   - Usar formato de checklist: `- [ ]` para marcar subtasks como concluídas
   - Marcar como concluída: `- [x]` quando a subtask estiver finalizada
   - Uma história só é considerada concluída quando todas as subtasks estiverem marcadas como concluídas

6. **Critérios de Aceite:**
   - Mínimo 5 critérios, ideal 7-10
   - Cada critério deve ser:
     - Específico (não genérico)
     - Mensurável (pode ser validado)
     - Testável (pode ser verificado)
   - Usar formato de checklist: `- [ ]`
   - Incluir critérios de:
     - Funcionalidade (o que foi criado)
     - Qualidade (validações, testes)
     - Documentação (se aplicável)

7. **Status da História:**
   - Adicionar campo de status no início do arquivo (após o título)
   - Formato: `## Status: [Em Andamento | Concluída]`
   - Quando todas as subtasks e critérios de aceite estiverem concluídos, marcar como `## Status: Concluída`
   - Marcar subtasks como concluídas usando checkbox: `- [x]` na lista de Subtasks

## 📋 Formato do Arquivo de Subtask

### Nomenclatura da Subtask

**Formato obrigatório:**
```
Subtask-XX-Nome_Da_Subtask.md
```

**Regras:**
- Sempre começar com `Subtask-` seguido do número com 2 dígitos (01, 02, 03...)
- Número deve ser sequencial dentro da história
- Nome deve ser curto e descritivo (máximo 4-5 palavras)
- Usar underscore (`_`) para separar palavras
- Primeira letra de cada palavra em maiúscula

**Exemplos corretos:**
- `Subtask-01-Criar_modulo_Terraform_backend_S3.md`
- `Subtask-02-Configurar_backend_tf_principal.md`
- `Subtask-03-Adicionar_IAM_policies_minimas_backend.md`

### Estrutura Obrigatória da Subtask

```markdown
# Subtask XX: Nome da Subtask

## Status: [Em Andamento | Concluída]

## Descrição
[Uma ou duas frases explicando exatamente o que será feito nesta subtask]

## Passos de implementação
- [ ] [Passo 1 específico e acionável]
- [x] [Passo 2 específico e acionável]
- [ ] [Passo 3 específico e acionável]

## Como testar
- [Teste 1: Como validar este passo]
- [Teste 2: Comando ou ação de validação]
- [Teste 3: Verificação manual ou automatizada]

## Critérios de aceite
- [x] [Critério 1 específico e mensurável]
- [x] [Critério 2 específico e mensurável]
- [ ] [Critério 3 específico e mensurável]
```

### Regras de Escrita da Subtask

1. **Descrição:**
   - Máximo 2-3 frases
   - Deve ser clara sobre O QUE será feito
   - Não precisa explicar COMO (isso vai nos passos)

2. **Passos de implementação:**
   - Mínimo 3 passos, ideal 5-8 passos
   - Cada passo deve ser:
     - Específico (não genérico)
     - Acionável (pode ser executado)
     - Sequencial (ordem lógica)
   - Usar formato de checklist: `- [ ]` para permitir marcação de progresso
   - Marcar como concluído: `- [x]` quando o passo for finalizado
   - Incluir nomes de arquivos, recursos, ou comandos quando relevante
   - Exemplo: `- [ ] Criar arquivo terraform/modules/eks/main.tf`

3. **Como testar:**
   - Mínimo 3 formas de teste/validação
   - Deve incluir:
     - Comandos para executar (ex: `terraform validate`)
     - Verificações manuais (ex: "Verificar via AWS CLI")
     - Validações automatizadas (ex: "Teste unitário deve passar")
   - Cada teste deve ser executável e verificável
   - **IMPORTANTE:** Subtasks devem ser simples o suficiente para serem testadas individualmente

4. **Critérios de aceite:**
   - Mínimo 3 critérios, ideal 4-6 critérios
   - Cada critério deve ser:
     - Específico (ex: "Arquivo main.tf criado" não "Código criado")
     - Mensurável (pode ser verificado)
     - Testável (pode ser validado)
   - Usar formato de checklist: `- [ ]` para permitir marcação de conclusão
   - Marcar como concluído: `- [x]` quando o critério for atendido

5. **Status da Subtask:**
   - Adicionar campo de status no início do arquivo (após o título)
   - Formato: `## Status: [Em Andamento | Concluída]`
   - Marcar como `## Status: Concluída` quando:
     - Todos os passos de implementação estiverem concluídos
     - Todos os critérios de aceite estiverem marcados como concluídos
     - Testes foram executados e validados
   - Passos de implementação também podem usar checklist: `- [ ]` para marcar progresso

## ✅ Princípios de Qualidade

### Simplicidade das Subtasks

**Regra fundamental:** Cada subtask deve ser simples o suficiente para:
1. Ser implementada em 1-2 horas (ideal)
2. Ser testada de forma independente
3. Ter critérios de aceite claros e objetivos
4. Poder ser validada sem depender de outras subtasks (quando possível)

**Se uma subtask for muito complexa:**
- Quebre em múltiplas subtasks menores
- Cada subtask deve ter um objetivo único e claro
- Evite subtasks que fazem "muitas coisas ao mesmo tempo"

### Consistência

1. **Nomenclatura:**
   - Sempre seguir o padrão estabelecido
   - Manter consistência entre histórias
   - Usar termos técnicos corretos

2. **Formato:**
   - Sempre usar a estrutura obrigatória
   - Manter formatação consistente
   - Usar markdown corretamente

3. **Linguagem:**
   - Usar português brasileiro
   - Ser claro e objetivo
   - Evitar jargões desnecessários
   - Usar termos técnicos quando apropriado

### Rastreabilidade

1. **Links:**
   - Todos os links devem ser relativos
   - Links devem funcionar quando acessados
   - Formato: `./subtask/Subtask-XX-Nome.md`

2. **Referências:**
   - Se referenciar outras histórias, usar caminho relativo
   - Se referenciar arquivos do projeto, usar caminho relativo
   - Exemplo: `terraform/modules/backend/main.tf`

## 📊 Checklist de Criação de História

Antes de considerar uma história completa, verificar:

### Estrutura
- [ ] Pasta criada com nome no formato `Storie-XX-Descricao_Breve`
- [ ] Arquivo `story.md` criado na raiz da pasta
- [ ] Pasta `subtask/` criada
- [ ] Todas as subtasks criadas na pasta `subtask/`

### Conteúdo do story.md
- [ ] Título no formato correto
- [ ] Descrição no formato "Como... quero... para..."
- [ ] Objetivo claro e objetivo
- [ ] Escopo técnico completo (tecnologias, arquivos, recursos)
- [ ] Mínimo 3 subtasks listadas
- [ ] Links das subtasks funcionando
- [ ] Mínimo 5 critérios de aceite
- [ ] Todos os critérios são específicos e mensuráveis

### Conteúdo das Subtasks
- [ ] Cada subtask tem descrição clara
- [ ] Cada subtask tem mínimo 3 passos de implementação
- [ ] Cada subtask tem mínimo 3 formas de teste
- [ ] Cada subtask tem mínimo 3 critérios de aceite
- [ ] Subtasks são simples e testáveis individualmente
- [ ] Ordem das subtasks faz sentido lógico

### Qualidade
- [ ] Nomenclatura consistente
- [ ] Formatação markdown correta
- [ ] Sem erros de ortografia
- [ ] Linguagem clara e objetiva
- [ ] Termos técnicos usados corretamente

## 🎯 Exemplo Completo

### Estrutura de Pasta
```
stories/
└── Storie-19-Criar_Manifests_Kubernetes/
    ├── story.md
    └── subtask/
        ├── Subtask-01-Criar_namespace_orderhub.md
        ├── Subtask-02-Criar_deployment_orderhub.md
        ├── Subtask-03-Criar_service_orderhub.md
        └── Subtask-04-Criar_hpa_orderhub.md
```

### Exemplo de story.md
```markdown
# Storie-19: Criar Manifests Kubernetes para OrderHub

## Status: Em Andamento

## Descrição
Como engenheiro de plataforma, quero criar manifests Kubernetes (Deployment, Service, HPA) para o microsserviço OrderHub, para que ele possa ser implantado no cluster EKS com escalabilidade horizontal independente.

## Objetivo
Criar manifests Kubernetes completos para o OrderHub incluindo Deployment, Service tipo LoadBalancer, e HPA configurado para escalar baseado em CPU, garantindo alta disponibilidade e escalabilidade automática.

## Escopo Técnico
- Tecnologias: Kubernetes, YAML, EKS
- Arquivos afetados: `k8s/app/orderhub/deployment.yaml`, `k8s/app/orderhub/service.yaml`, `k8s/app/orderhub/hpa.yaml`
- Recursos Kubernetes: Deployment, Service (LoadBalancer), HorizontalPodAutoscaler

## Subtasks

- [x] [Subtask 01: Criar namespace orderhub](./subtask/Subtask-01-Criar_namespace_orderhub.md)
- [x] [Subtask 02: Criar deployment para OrderHub](./subtask/Subtask-02-Criar_deployment_orderhub.md)
- [x] [Subtask 03: Criar service LoadBalancer para OrderHub](./subtask/Subtask-03-Criar_service_orderhub.md)
- [ ] [Subtask 04: Criar HPA para OrderHub](./subtask/Subtask-04-Criar_hpa_orderhub.md)

## Critérios de Aceite da História

- [x] Namespace `orderhub` criado em `k8s/app/orderhub/namespace.yaml`
- [x] Deployment criado com imagem do ECR `orderhub-api`
- [x] Deployment configurado com resources (requests e limits)
- [x] Service tipo LoadBalancer criado com annotations para NLB
- [ ] HPA criado com minReplicas=2, maxReplicas=5
- [ ] HPA configurado para escalar baseado em CPU (targetCPUUtilization=70)
- [ ] Manifests validados com `kubectl apply --dry-run=client`
- [ ] Documentação de deploy criada
```

### Exemplo de Subtask
```markdown
# Subtask 01: Criar namespace orderhub

## Status: Concluída

## Descrição
Criar arquivo YAML para namespace Kubernetes que será usado pelo microsserviço OrderHub, garantindo isolamento de recursos.

## Passos de implementação
- [x] Criar diretório `k8s/app/orderhub/`
- [x] Criar arquivo `namespace.yaml` no diretório
- [x] Adicionar recurso `kind: Namespace` com `name: orderhub`
- [x] Adicionar labels padrão: `app: orderhub`, `env: dev`
- [x] Adicionar comentários explicativos no arquivo

## Como testar
- Executar `kubectl apply --dry-run=client -f k8s/app/orderhub/namespace.yaml` (deve passar sem erros)
- Validar sintaxe YAML com validador online ou `yamllint`
- Verificar que o arquivo segue padrão de indentação (2 espaços)
- Executar `kubectl get namespace orderhub` após aplicar (deve retornar o namespace)

## Critérios de aceite
- [x] Arquivo `k8s/app/orderhub/namespace.yaml` criado
- [x] Namespace tem nome `orderhub`
- [x] Labels padrão aplicadas (app, env)
- [x] `kubectl apply --dry-run=client` passa sem erros
- [x] Sintaxe YAML válida
```

## 🚫 Erros Comuns a Evitar

1. **Subtasks muito complexas:**
   - ❌ "Criar módulo EKS completo com todas as configurações"
   - ✅ "Criar estrutura base do módulo EKS" + "Implementar recurso aws_eks_cluster" + "Configurar logging"

2. **Critérios de aceite vagos:**
   - ❌ "Código funciona"
   - ✅ "terraform validate passa sem erros"

3. **Falta de testabilidade:**
   - ❌ "Verificar que está correto"
   - ✅ "Executar terraform plan e verificar que mostra criação de 1 recurso"

4. **Nomenclatura inconsistente:**
   - ❌ `Storie-1-Backend` (sem zero, sem descrição)
   - ✅ `Storie-01-Backend_Terraform_S3`

5. **Links quebrados:**
   - ❌ `./Subtask-01.md` (caminho errado)
   - ✅ `./subtask/Subtask-01-Criar_modulo.md`

## ✅ Marcação de Conclusão

### Como Marcar uma Subtask como Concluída

1. **No arquivo da subtask (`Subtask-XX-Nome.md`):**
   - Alterar `## Status: Em Andamento` para `## Status: Concluída`
   - Marcar todos os passos de implementação como concluídos: `- [x]`
   - Marcar todos os critérios de aceite como concluídos: `- [x]`
   - Garantir que todos os testes foram executados e validados

2. **No arquivo da história (`story.md`):**
   - Marcar a subtask na lista de Subtasks: `- [x] [Subtask XX: Nome da subtask](./subtask/Subtask-XX-Nome.md)`
   - Atualizar os critérios de aceite relacionados àquela subtask (se houver)

### Como Marcar uma História como Concluída

Uma história é considerada concluída quando:

1. **Todas as subtasks estão concluídas:**
   - Todas as subtasks na lista devem estar marcadas: `- [x]`
   - Cada subtask individual deve ter `## Status: Concluída`

2. **Todos os critérios de aceite da história estão atendidos:**
   - Todos os critérios devem estar marcados: `- [x]`
   - Cada critério deve ter sido validado/testado

3. **Atualizar o status da história:**
   - Alterar `## Status: Em Andamento` para `## Status: Concluída` no arquivo `story.md`

### Boas Práticas

- **Atualizar progresso regularmente:** Marque subtasks e critérios conforme são concluídos, não apenas no final
- **Validar antes de marcar:** Só marque como concluído após validar que realmente está funcionando/testado
- **Manter sincronizado:** Quando uma subtask é concluída, atualize tanto o arquivo da subtask quanto a lista na história
- **Documentar problemas:** Se uma subtask não puder ser concluída como planejado, documente o motivo e ajuste os critérios se necessário

## 📚 Referências

- Histórias existentes em `stories/` servem como exemplos
- Manter consistência com padrões já estabelecidos
- Quando em dúvida, seguir os exemplos das histórias 01-18

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/diegoknsk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
