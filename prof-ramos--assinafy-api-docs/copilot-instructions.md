## assinafy-api-docs

> Este projeto contém a documentação completa e estruturada da **API Assinafy (v1)**, extraída para facilitar a integração programática de fluxos de assinatura eletrônica.

# Assinafy Integration & Documentation

Este projeto contém a documentação completa e estruturada da **API Assinafy (v1)**, extraída para facilitar a integração programática de fluxos de assinatura eletrônica.

## Visão Geral do Projeto

O objetivo deste repositório é servir como base de conhecimento para o desenvolvimento de integrações com a plataforma Assinafy. A API é baseada em princípios **RESTful** e permite a gestão completa do ciclo de vida de documentos, signatários e notificações.

### Tecnologias da API
- **Arquitetura:** RESTful
- **Formatos:** JSON (padrão) e XML.
- **Ambientes:**
  - Sandbox: `https://sandbox.assinafy.com.br/v1`
  - Produção: `https://api.assinafy.com.br/v1`

## Estrutura de Documentação (`/docs`)

- `01_INTRODUCTION.md`: Conceitos básicos, verbos HTTP e ambientes.
- `02_AUTHENTICATION.md`: Métodos de autenticação (X-Api-Key e JWT).
- `03_DOCUMENTS.md`: Upload e gestão de documentos/templates.
- `04_SIGNERS.md`: Gestão de signatários e fluxos de aceite.
- `05_ASSIGNMENTS.md`: Pedidos de assinatura (Virtual e Collect) e custos de créditos.
- `06_WEBHOOKS.md`: Notificações em tempo real para automação.
- `07_FIELDS.md`: Definição de campos personalizados para coleta de dados.
- `API_REFERENCE.md`: Referência técnica completa e exaustiva.

## Uso e Integração

Para iniciar uma integração, siga as etapas descritas em `docs/01_INTRODUCTION.md` e `docs/02_AUTHENTICATION.md`. Os fluxos principais envolvem:
1. Upload de documento via `POST /accounts/{id}/documents`.
2. Criação de signatários via `POST /accounts/{id}/signers`.
3. Criação de pedido de assinatura via `POST /documents/{id}/assignments`.

### Convenções de Desenvolvimento
- **Segurança:** Nunca armazene a `X-Api-Key` diretamente no código fonte; utilize variáveis de ambiente.
- **Custos:** Esteja atento aos custos de créditos para notificações via WhatsApp (1 crédito por envio/reenvio).
- **Webhooks:** Recomenda-se o uso de webhooks para reagir ao evento `document_ready` em vez de fazer polling na API.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prof-ramos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/prof-ramos)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
