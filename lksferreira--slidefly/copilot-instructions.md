## slidefly

> Slidefly é uma aplicação web (SPA) moderna que transforma arquivos CSV de demandas em apresentações profissionais, interativas e exportáveis (PDF/PPTX) diretamente no navegador, com foco em privacidade (100% client-side).

# Slidefly - Contexto do Projeto

Slidefly é uma aplicação web (SPA) moderna que transforma arquivos CSV de demandas em apresentações profissionais, interativas e exportáveis (PDF/PPTX) diretamente no navegador, com foco em privacidade (100% client-side).

## 🚀 Tecnologias Principais

- **Framework:** React 19 + Vite 6 + TypeScript
- **Estilização:** Tailwind CSS 4
- **Animações:** Framer Motion
- **Parsing CSV:** PapaParse
- **Exportação:** html2pdf.js (PDF) e PptxGenJS (PPTX)
- **Drag & Drop:** @hello-pangea/dnd (Ordem dos slides) e react-grid-layout (Layout Personalizado)
- **Infra de Dev:** Docker Compose

## 📁 Estrutura do Projeto

- `src/components/`: Componentes da UI (Landing, Configurator, Presentation, Slide).
- `src/utils/`: Lógica de processamento (CSV, Normalização, Exportação).
- `src/types/`: Definições de tipos TypeScript.
- `src/config/`: Regras de negócio e configurações de layout.
- `.agents/`: **Instruções Oficiais para IAs** (regras de workflow, git, docker, etc).
- `.metadocs/`: Documentação técnica, roadmap e histórico de decisões.
- `PRD.md`: Documento de requisitos do produto (fonte da verdade para funcionalidades).

## 🛠️ Comandos Úteis

### Ambiente Local (com Docker - Recomendado)
```bash
bash dev.sh          # Sobe o ambiente
bash dev.sh --build  # Rebuild das imagens
```

### Comandos npm (dentro do container ou local)
```bash
bash exec.sh run dev      # Inicia servidor de desenvolvimento (porta 3000)
bash exec.sh run build    # Gera build de produção
bash exec.sh run lint     # Checagem de tipos (tsc)
bash exec.sh run preview  # Preview do build local
```

## ⚖️ Fluxo de Trabalho Obrigatório (Regra de Ouro)

Este projeto exige um fluxo rigoroso de interação com o Agente, definido em `.agents/rules/workflow.md`:

1.  **Análise (Discovery):** Estudar contexto e impactos.
2.  **Discussão (Debate):** Apresentar análise e discutir trade-offs com o usuário.
3.  **Refinamento:** Ajustar proposta com base no feedback.
4.  **Aprovação:** Criar plano detalhado e **AGUARDAR aprovação explícita**.
5.  **Execução:** Implementar estritamente o que foi aprovado.

**NUNCA implemente código sem aprovação prévia do plano.**

## 📝 Convenções e Regras de Negócio

- **Privacidade:** Todo o processamento é client-side. Nenhum dado deve ser enviado para APIs externas.
- **Layouts:** Suporte oficial apenas para os layouts `Padrão` e `Personalizado`.
- **CSV:** Colunas obrigatórias são `ID` e `Title`. O parsing deve ignorar colunas extras e ocultar campos vazios.
- **Arquitetura:** Manter separação clara entre lógica de dados (`utils`) e renderização (`components`).
- **Commits:** Seguir as regras em `.agents/rules/git.md` (somente sob demanda explícita).

## 🔗 Referências Importantes
- [PRD.md](PRD.md): Requisitos detalhados.
- [AGENTS.md](AGENTS.md): Estrutura de instruções para IAs.
- [.metadocs/roadmap.md](.metadocs/roadmap.md): Próximos passos do projeto.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LKSFerreira)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LKSFerreira)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
