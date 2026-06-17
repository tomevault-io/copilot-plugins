## conversation-knowledge-mining-solution-accelerator

> - **Frontend:** React (Create React App), visualização dinâmica de insights (Chart.js), interface de chatbot, filtros dinâmicos.

# SKILL.md

## Conversation Knowledge Mining Solution Accelerator

### Arquitetura e Padrões
- **Frontend:** React (Create React App), visualização dinâmica de insights (Chart.js), interface de chatbot, filtros dinâmicos.
- **Backend:** FastAPI, modularização por rotas (API, histórico, helpers), integração com Azure OpenAI, Semantic Kernel, Azure AI Search, SQL e Cosmos DB.
- **Infraestrutura:** Bicep modular (infra/, modules/), alinhado ao Well-Architected Framework (WAF), suporte a sandbox/dev e produção, provisionamento via `azd up`.
- **Observabilidade:** Application Insights, Log Analytics, health check endpoint, monitoramento e diagnósticos habilitados por padrão.
- **Segurança:** Managed identities, RBAC, private endpoints, desabilitação de autenticação local, tags e locks em recursos críticos.
- **Escalabilidade:** Parâmetros para enableScalability, uso de Azure Container Apps, configuração de SKUs e recursos ajustáveis.
- **Dados:** Processamento de áudio/texto, extração de entidades, tópicos, sentimentos, armazenamento estruturado (JSON, SQL, vetores), histórico de conversas em Cosmos DB.

### Boas Práticas
- **Infraestrutura como código:** Use sempre os módulos Bicep e mantenha parâmetros separados para dev/prod.
- **Segurança:** Prefira endpoints privados, managed identities e RBAC mínimo necessário.
- **Observabilidade:** Habilite Application Insights e Log Analytics em todos os serviços.
- **Escalabilidade:** Ajuste SKUs e parâmetros conforme a carga esperada.
- **Modularização:** Separe lógica de API, processamento, helpers e testes.
- **Testes:** Utilize pytest, mantenha cobertura para endpoints críticos e helpers.
- **Documentação:** Atualize README, DeploymentGuide e TechnicalArchitecture ao evoluir a solução.

### Pontos de Atenção
- **Parâmetros sensíveis:** Nunca versionar secrets, use Key Vault e parâmetros seguros.
- **Deploy:** Para produção, sempre utilize `main.waf.parameters.json` para máxima segurança.
- **Dependências:** Mantenha requirements.txt e package-lock.json atualizados.
- **Atualizações Azure:** Verifique compatibilidade de módulos Bicep e SKUs periodicamente.

### Fluxo de Deploy
1. Configure variáveis e parâmetros em `infra/main.parameters.json` ou `main.waf.parameters.json`.
2. Execute `azd up` para provisionar toda a infraestrutura.
3. Faça build e deploy dos containers (App, funções, scripts).
4. Monitore via Application Insights e Log Analytics.
5. Use endpoints de health check para validação pós-deploy.

### Troubleshooting
- Consulte logs no Application Insights e Log Analytics.
- Use endpoints `/health` para checagem rápida.
- Para erros de provisionamento, revise parâmetros e dependências em Bicep.
- Veja a documentação em `/documents` e `/docs/workshop`.

---
> Source: [acmeleme/Conversation-Knowledge-Mining-Solution-Accelerator](https://github.com/acmeleme/Conversation-Knowledge-Mining-Solution-Accelerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
