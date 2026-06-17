## openclaw-antigravity-bridge

> >


# Skill: Antigravity Bridge

Bridge entre OpenClaw Gateway e Google Antigravity via `antigravity-claude-proxy` (porta 8080).

## Setup Completo (instalação nova)

```powershell
powershell -ExecutionPolicy Bypass -File skills/antigravity/scripts/setup.ps1
```

O script verifica Node.js, instala o proxy globalmente, inicia o serviço e guia na configuração da primeira conta.

## Comandos do Proxy

```powershell
acc status              # Status do proxy
acc start               # Iniciar (serviço de fundo)
acc start --log         # Iniciar com logs
acc stop                # Parar
acc restart             # Reiniciar
acc ui                  # Dashboard web
```

## Gerenciar Contas

```powershell
acc accounts list           # Listar contas
acc accounts add            # Adicionar (abre navegador)
acc accounts add --no-browser  # Adicionar (headless)
acc accounts remove <email> # Remover
acc accounts verify         # Verificar saúde
```

## Estratégias de Balanceamento

```powershell
acc start --strategy=hybrid       # (padrão) health score + tempo ocioso
acc start --strategy=sticky       # Consistência de sessão
acc start --strategy=round-robin  # Alternância igual
```

## Importar Contas do OpenCode

```powershell
powershell -ExecutionPolicy Bypass -File skills/antigravity/scripts/import-opencode-accounts.ps1
```

## Configurar Provider no OpenClaw

Ver o arquivo de referência: `config/openclaw-provider.json`

Modelos disponíveis:
- `antigravity-proxy/claude-opus-4-6-thinking` (200K ctx)
- `antigravity-proxy/claude-sonnet-4-6` (200K ctx)
- `antigravity-proxy/gemini-3.1-pro-high` (1M ctx)
- `antigravity-proxy/gemini-3-flash` (1M ctx)

## Troubleshooting

- Proxy offline → `acc restart`
- Token expirado → `acc accounts add`
- Porta ocupada → `PORT=3000 acc start`
- Reset completo → `acc stop` + deletar `accounts.json` + `acc start` + `acc accounts add`

---
> Source: [lunalimadev-sketch/openclaw-antigravity-bridge](https://github.com/lunalimadev-sketch/openclaw-antigravity-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
