## deploy-pm2

> Após mudanças de código que afectem a app CRM, faz build e reinicia PM2.


# Build e PM2 ao finalizar tarefas

No repositório **crm-telecom-energia**, quando alterares código que vá para produção (por exemplo `client/`, `server/`, `shared/`, `vite.config.ts`, `package.json`, `drizzle/`), executa **`pnpm run deploy:pm2`** na raiz do projecto antes de dar a resposta como concluída.

- O script faz `pnpm run build` (Vite + bundle do servidor) e **`pm2 restart crm`**.
- Não precisas de redepetir este passo apenas para edições puremente documentais (README, comentários que não mudam comportamento).

Se o comando falhar, reporta o erro e corrige antes de terminar.

---
> Source: [michellerikbrgo-cmyk/crm-telecom-energia](https://github.com/michellerikbrgo-cmyk/crm-telecom-energia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
