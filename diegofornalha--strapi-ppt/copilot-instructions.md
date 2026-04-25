## strapi-ppt

> SEMPRE RESPONDA EM PT BR

SEMPRE RESPONDA EM PT BR

# Projeto BIANCA-SANITY - Sistema Integrado de Gestão de Conteúdo e Apresentações

## MCP BiancaTools - Configuração

### Instalação Global do BiancaTools
Para usar o BiancaTools MCP em qualquer projeto, execute:

```bash
claude mcp add BiancaTools /Users/phiz/Desktop/BIANCA-SANITY/mcp-run-ts-tools/run-bianca-tools.sh --env GITHUB_TOKEN=SEU_TOKEN_AQUI -s user
```

### Ferramentas Disponíveis no BiancaTools
- **Puppeteer**: Automação web (navegação, screenshots, cliques, preenchimento de formulários)
- **GitHub**: Criar issues, PRs, repositórios, commits
- **Git**: Status, commit, push, pull
- **Mem0**: Gerenciamento de memória persistente (adicionar, buscar, listar, deletar memórias)
- **Claude**: Executar comandos Claude via CLI

### Estrutura do BiancaTools
```
/mcp-run-ts-tools/
├── src/                    # Código fonte TypeScript
│   ├── tools/             # Implementação das ferramentas
│   │   ├── claude/       # Integração com Claude CLI
│   │   ├── git/          # Comandos Git
│   │   ├── github/       # API do GitHub
│   │   ├── mem0/         # Sistema de memória persistente
│   │   └── puppeteer/    # Automação web
│   └── index.ts          # Entrada principal
├── build/                 # Código compilado
├── run-bianca-tools.sh   # Script wrapper para execução global
└── package.json          # Configuração do projeto
```

## Visão Geral
Projeto integrado que combina:
- **Sanity CMS**: Sistema de gestão de conteúdo (schemas prontos, Studio ainda não instalado)
- **Slidev**: Módulo de apresentações interativas (com cliente Sanity configurado)
- **Sistemas Auxiliares**: Cronogramas e gestão de clientes

### ⚠️ IMPORTANTE: Diferença entre Sanity Studio e Cliente Sanity

**Cliente Sanity** (`/bianca-slidev/sanity/client.ts`) ✅ JÁ CONFIGURADO
- Biblioteca para CONSUMIR dados do Sanity
- Usado no frontend para fazer queries GROQ
- Permite apenas LEITURA de dados

**Sanity Studio** ❌ AINDA NÃO INSTALADO
- Interface administrativa web para CRIAR/EDITAR conteúdo
- Painel onde editores gerenciam o CMS
- Necessário para adicionar dados ao sistema

## Estrutura Completa do Projeto

```
/BIANCA-SANITY/                    # Raiz do projeto
├── /schemas/                      # Schemas do Sanity Studio
│   ├── /documents/               
│   │   ├── cliente.js            # Schema de clientes
│   │   ├── evidenciaServico.js   # Schema de evidências
│   │   └── presentation.js       # Schema de apresentações
│   └── /objects/                 # Objetos reutilizáveis
│
├── /clientes/                     # Dados e configurações de clientes
│   ├── /follow-the-money/        
│   └── /nurnberg/                # Cliente Nurnberg
│       ├── nurnberg-site.json    
│       ├── playbook-replanejamento.json
│       └── workflow-playbook.md
│
├── /planilha/                     # Sistema de cronograma
│   ├── cronograma.db             # Banco de dados SQLite
│   ├── servidor_cronograma.py    # Servidor do cronograma
│   └── [arquivos HTML/JS]        # Interfaces do cronograma
│
├── /bianca-slidev/                # MÓDULO SLIDEV (Apresentações)
│   ├── /components/              # Componentes Vue
│   │   ├── SanityExample.vue    # Exemplo de integração Sanity
│   │   ├── EstrategiaNurnberg.vue
│   │   └── DrawflowEditor.vue   
│   ├── /composables/             
│   │   └── sanity.ts            # Funções para buscar dados do Sanity
│   ├── /sanity/                 
│   │   └── client.ts            # Cliente Sanity (NÃO é o Studio!)
│   ├── slides.md                # Slides principais
│   └── package.json             # Dependências do Slidev
│
├── /icones/                       # Ícones SVG compartilhados
│
└── [FALTA CRIAR] /studio/         # Sanity Studio (interface administrativa)
```

## Integração Sanity + Slidev

### 1. Configuração do Sanity no Slidev

O módulo Slidev já está preparado para integração com Sanity:

```typescript
// bianca-slidev/sanity/client.ts
import { createClient } from '@sanity/client'

export const client = createClient({
  projectId: process.env.VITE_SANITY_PROJECT_ID,
  dataset: process.env.VITE_SANITY_DATASET,
  useCdn: true,
  apiVersion: '2024-01-01'
})
```

### 2. Configurar Variáveis de Ambiente

Criar arquivo `.env` em `/bianca-slidev/`:
```env
VITE_SANITY_PROJECT_ID=seu_project_id
VITE_SANITY_DATASET=production
```

### 3. Usar Dados do Sanity nas Apresentações

```vue
<!-- Em qualquer componente Vue -->
<script setup>
import { useSanity } from '../composables/sanity'

const { data: presentations, loading } = await useSanity().getPresentation('presentation-id')
</script>
```

## Como Instalar o Sanity Studio (NECESSÁRIO!)

### 1. Instalar Sanity CLI
```bash
npm install -g @sanity/cli
```

### 2. Criar o Sanity Studio na raiz do projeto
```bash
cd /Users/phiz/Desktop/BIANCA-SANITY
npm create sanity@latest -- --template clean --create-project "Bianca CMS" --dataset production

# Durante a instalação:
# - Escolha "Yes" para usar os schemas existentes
# - Aponte para a pasta /schemas quando perguntado
```

### 3. Mover os schemas existentes para o Studio
```bash
# Após criar o studio, copie seus schemas
cp -r schemas/* studio/schemas/
```

### 4. Comandos do Sanity Studio (após instalação)
```bash
cd studio
npm run dev          # Iniciar localmente
npm run deploy       # Deploy para sanity.studio
```

### Slidev Presentations
```bash
cd bianca-slidev

# Instalar dependências
npm install

# Desenvolvimento
npm run dev

# Build para produção
npm run build

# Exportar para PDF
npm run export
```

### Sistema de Cronograma
```bash
cd planilha

# Iniciar servidor
python servidor_cronograma.py

# Acessar em: http://localhost:8080
```

## Fluxo de Trabalho Integrado

### 1. Criar Conteúdo no Sanity
- Acessar Sanity Studio
- Criar/editar apresentações, clientes, evidências
- Publicar conteúdo

### 2. Consumir no Slidev
- Dados são buscados automaticamente via composables
- Componentes Vue renderizam conteúdo dinâmico
- Apresentações ficam sempre atualizadas

### 3. Verificar Visualmente (MCP Puppeteer)
```
1. Navegar → mcp__puppeteer-server__puppeteer_navigate
2. Screenshot → mcp__puppeteer-server__puppeteer_screenshot
3. Identificar problemas
4. Corrigir código
5. Repetir até perfeito
```

## Schemas Principais

### Schema de Apresentação
```javascript
// schemas/documents/presentation.js
export default {
  name: 'presentation',
  title: 'Apresentação',
  type: 'document',
  fields: [
    {
      name: 'titulo',
      title: 'Título',
      type: 'string',
      validation: Rule => Rule.required()
    },
    {
      name: 'cliente',
      title: 'Cliente',
      type: 'reference',
      to: [{type: 'cliente'}]
    },
    {
      name: 'slides',
      title: 'Slides',
      type: 'array',
      of: [{type: 'slide'}]
    }
  ]
}
```

## Checklist de Integração

### Passo 1: Instalar Sanity Studio
- [ ] Instalar Sanity CLI globalmente
- [ ] Criar novo projeto Sanity Studio em `/studio`
- [ ] Copiar schemas existentes para o Studio
- [ ] Configurar `sanity.config.js` com os schemas
- [ ] Testar Studio localmente com `npm run dev`

### Passo 2: Configurar Integração
- [ ] Obter `projectId` e `dataset` do Sanity
- [ ] Criar arquivo `.env` em `/bianca-slidev/`
- [ ] Adicionar credenciais ao `.env`
- [ ] Testar conexão cliente-Sanity

### Passo 3: Popular com Dados
- [ ] Acessar Sanity Studio
- [ ] Criar conteúdo de teste (clientes, apresentações)
- [ ] Testar queries GROQ no Vision plugin
- [ ] Verificar dados no Slidev

### Passo 4: Deploy
- [ ] Deploy do Sanity Studio
- [ ] Deploy das apresentações Slidev
- [ ] Configurar CORS no Sanity

## Próximos Passos

1. **Completar Integração Sanity**
   - Configurar projeto no Sanity.io
   - Adicionar credenciais ao .env
   - Testar busca de dados

2. **Expandir Componentes Slidev**
   - Criar mais componentes dinâmicos
   - Integrar com dados de clientes
   - Adicionar animações e transições

3. **Automatizar Workflows**
   - Scripts de backup do Sanity
   - CI/CD para deploy automático
   - Testes automatizados

## Recursos e Documentação

- [Sanity Docs](https://www.sanity.io/docs)
- [Slidev Guide](https://sli.dev/guide/)
- [Vue 3 Docs](https://vuejs.org/)
- [GROQ Cheat Sheet](https://www.sanity.io/docs/groq)

## Suporte

Para dúvidas sobre o projeto:
- Verificar esta documentação
- Consultar documentação oficial das ferramentas
- Abrir issue no repositório

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/diegofornalha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
