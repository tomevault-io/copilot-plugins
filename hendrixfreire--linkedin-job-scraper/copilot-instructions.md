## linkedin-job-scraper

> Pipeline local e auditável para candidaturas de emprego autônomas.

@AGENTS.md

## Delta técnico do Claude Code

### O que é
Pipeline local e auditável para candidaturas de emprego autônomas.

### Stack
- Python 3.14
- SQLite stdlib
- Playwright
- HTTP dashboard stdlib
- Hermes cron

### Estrutura
- `src/candidatura_agent/db.py`: schema e persistência
- `src/candidatura_agent/policy.py`: score, bloqueios e autoenvio
- `src/candidatura_agent/ingest.py`: importação do scraper
- `src/candidatura_agent/linkedin.py`: descoberta integrada no LinkedIn Guest Jobs
- `src/candidatura_agent/browser.py`: Playwright e adaptadores
- `src/candidatura_agent/dashboard.py`: painel HTTP, API de vagas, filtros e auditoria
- `src/candidatura_agent/report.py`: relatório diário

### Convenções técnicas
- Código e nomes técnicos em inglês; UI e relatórios em PT-BR.
- Toda transição de status gera evento auditável.
- Nenhum envio fora de domínio permitido.
- Nenhuma resposta factual inventada.
- TDD obrigatório para comportamento novo.

### Testes
```bash
./scripts/setup.sh
.venv/bin/python -m pytest -q
```

### Estado atual
Implementação inicial em `dry_run`; autoenvio bloqueado até calibração. GUI local em fatia vertical (fase 1): home, lista/filtros de vagas, detalhe somente leitura com bloqueios e trilha de auditoria — sem botões de envio.

### Referências técnicas
- `README.md`
- `docs/plans/2026-07-13-candidatura-autonoma.md`

---
> Source: [hendrixfreire/linkedin-job-scraper](https://github.com/hendrixfreire/linkedin-job-scraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
