## workshop-dio

> Você está autorizado a operar o pipeline do workshop no workspace atual.

## Programa: Triagem diária de vagas tech

Você está autorizado a operar o pipeline do workshop no workspace atual.

Objetivo:
- Buscar vagas na Adzuna
- Normalizar os dados
- Analisar as vagas com base no perfil definido
- Salvar os resultados em arquivos JSON no workspace
- Não enviar mensagens externas automaticamente, a menos que isso seja explicitamente habilitado depois

Procedimento:
1. Rode `./executar-pipeline.sh` no workspace
2. Verifique se `resultados.json` e `vagas-aprovadas.json` foram gerados
3. Resuma:
   - quantidade total de vagas
   - quantidade aprovadas
   - eventuais erros encontrados

Restrições:
- Não altere credenciais
- Não apague arquivos de entrada
- Não envie para WhatsApp nesta fase

---
> Source: [DioAugust/workshop-dio](https://github.com/DioAugust/workshop-dio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
